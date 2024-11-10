---
title: 검수 저장 성능 개선
date: 2024-11-10 08:00:00 +0900
categories: troubleshooting
tags:
  - troubleshooting
  - performance-tuning
description: 데이터 조회, 수정, 저장 방식을 변경하여 약 300배(1m35s->0.3s) 성능 개선
---

> **검수** : AI가 객체를 올바르게 탐지했는지 사람이 확인하는 작업
{: .prompt-info }

## 개요

2024년 6월 4일에 회사에서 검수 저장 API의 성능을 개선한 작업에 대해 기록하고 회고한다.

## 배경 및 목적

폐쇄망 환경의 **고객으로부터 검수 저장 시 에러가 발생한다는 피드백**을 받았다. 내부에서도 동일한 문제가 발생하는지 확인하기 위해 버그를 재현해보았다. 임의로 **10,000개 이상의 객체를 생성하여 검수 저장이 실패하는지 확인**한 결과, 검수 저장 요청 시 1분 이상 소요되어 **게이트웨이에서 타임아웃(504 에러)이 발생**하는 것을 발견했다. 어디에서 병목이 발생하여 에러가 발생하는지 확인하고, **성능을 1분 미만으로 개선하는 것을 목표**로 작업을 진행했다.

## 개선 전 성능 측정

10,000개의 검수 정보를 저장하는 통합 테스트 코드를 구현하여 소요 시간을 확인했다. 아래 이미지는 통합 테스트의 실행 결과이며, 병목이 발생하는 부분을 확인하기 위해 IntelliJ Profiler 도구를 사용했다. 이미지를 보면 **10,000개의 검수 정보를 저장하는데 약 1분 35초(94,727ms)가 소요**되는 것을 확인할 수 있다.

![2024-11-10-save-review-1](/assets/img/2024-11-10-save-review-1.png)
_IntelliJ Profiler로 확인한 통합 테스트 결과 (94,727ms 소요)_

## 1차 원인 분석

Profiler 결과를 기반으로 오래 걸리는 로직을 검토한 결과, 다음 로직들에서 **병목이 발생**하는 것을 확인했다.

1. **Label 엔티티 목록 조회** : 약 27초 (26,561ms)
2. **썸네일 이미지 생성** : 약 10초 (9,619ms)
3. **LabelData 엔티티 목록 수정 및 저장** : 약 35초 (35,241ms)
4. **LabelReview 엔티티 목록 저장** : 약 7초 (7,383 ms)
5. **수정 이벤트 발행** : 약 1초 (560ms)
6. **응답을 위한 관련 데이터 조회** : 약 15초 (14,822ms)

성능을 개선하기 위해 코드를 분석한 후, **사용자가 수정한 것과 수정하지 않은 데이터를 나눠서 처리**하도록 코드를 수정하기로 결정했다. 이는 사용자가 수정하지 않은 데이터에 대해서는 확인 상태만 변경하면 되므로 **조회 및 수정할 데이터가 적기 때문**이다. 또한, 서비스 정책상 5분마다 자동 저장이 되므로 **사용자가 변경한 값이 많지 않을 것이라고 판단**했다.

## 1차 성능 개선

기존 코드에서 **변경된 것과 변경되지 않은 라벨을 나눠서 처리**하도록 다음과 같이 수정했다.

```kotlin
val (notChangedLabels, changedLabels) =
  params.partition { 
    it.status == LabelStatus.TP && 
      it.reviewContent == ReviewContent.CHECK 
  }
updateChangedLabels(changedLabels, pscene, updateTime, reviewInfo)
updateNotChangedLabels(notChangedLabels, reviewInfo)
```

엔티티를 조회하던 기존 방식에서 **변경할 라벨의 식별자 정보와 필요한 정보만 네이티브 쿼리로 조회**하도록 변경했다.

```sql
SELECT ld.data_id, ld.scene_id  
FROM label_data ld,  
     label l WHERE ld.data_id = l.data_id  
  AND l.label_id IN (:labelIdList);
```

데이터를 수정할 때도 엔티티 목록을 전달하여 수정하는 대신, **네이티브 쿼리로 특정 정보만 수정**하도록 변경했다.

```sql
UPDATE label_data  
SET status     = LabelStatus.TP.ordinal,  
    is_checked = TRUE
WHERE data_id IN (:dataIdList);
```

LabelReview 엔티티 목록을 순회하며 저장하던 로직을 **엔티티 목록을 한 번에 저장**하도록 수정했다.

```kotlin
notChangedLabelDataList.map { labelData ->
    LabelReview(
        labelDataId = labelData.dataId,
        reviewProjectId = reviewInfo.projectId,
        reviewer = reviewInfo.reviewer,
        reviewContent = ReviewContent.CHECK,
        sceneId = labelData.sceneId
    )
}.let { reviewRepository.save(it) }
```

동기로 처리하지 않아도 되는 **이벤트 발행은 코루틴을 활용하여 비동기로 처리**했다.

```kotlin
CoroutineScope(Dispatchers.IO).launch {
    val labelDataIdList = labelDataList.map { it.dataId }
    labelDataRepository
        .findAll(labelDataIdList)
        .forEach { labelData ->
            LabelDataMessagePublisher.labelDataUpdated(labelData, labelData)
        }
}
```

이러한 수정으로 **Label 엔티티 목록 조회, 썸네일 이미지 생성, LabelData 엔티티 목록 수정 및 저장, LabelReview 엔티티 목록 저장**의 성능을 모두 개선할 수 있었다.

## 1차 성능 측정

1차 성능 개선 후 성능을 측정한 결과, 약 15초로 개선된 것을 확인했다. 즉, **1분 30초에서 15초로 약 6배 개선**하여 **버그 수정 작업을 완료했다.** 그러나 15초도 느리다고 판단되어, 아직 개선하지 못한 부분을 분석하여 추가 작업을 진행했다.

## 2차 원인 분석

아직 개선하지 못한 병목 로직은 **응답을 위한 데이터 조회**이다. 코드를 분석해보니 Label 테이블과 LabelData 테이블을 조인하여 조회하고 사용자에게 데이터를 응답하는 로직이었다. 쿼리의 병목 지점을 확인하기 위해 실행 계획을 살펴보았다.

![2024-11-10-save-review-3](/assets/img/2024-11-10-save-review-3.png)
_실행 계획 (63.472ms 소요)_

쿼리가 63ms 소요되는 것을 확인했다. 즉, **쿼리 자체의 문제가 아닌 조회된 결과를 엔티티로 변환하는 과정에서 시간이 소요**되고 있었다.

<br/>

IntelliJ Profiler를 통해 병목 지점을 확인한 결과, 데이터 조회에 19초가 소요되었으며, org.hibernate.engine.internal.TwoPhaseLoad.initializeEntityEntryLoadedState 메서드가 약 18초 소요된 것을 확인했다.

![2024-11-10-save-review-4](/assets/img/2024-11-10-save-review-4.png)
_IntelliJ Profiler로 조회 소요 시간을 확인한 결과 (19,086ms 소요)_

![2024-11-10-save-review-5](/assets/img/2024-11-10-save-review-5.png)
_IntelliJ Profiler로 initializeEntityEntryLoadedState 메서드 소요 시간을 확인한 결과 (17,738ms 소요)_

`initializeEntityEntryLoadedState` 메서드는 Hibernate에서 엔티티를 로드하는 과정 중 **엔티티의 상태를 초기화하고, 데이터베이스에 로드된 시점의 스냅샷을 설정**하는 역할을 한다.

즉, **데이터 조회에는 100ms도 소요되지 않지만, 영속성 컨텍스트에 엔티티를 초기화하는 데 약 18초가 소요**된 것이다.

## 2차 문제 해결

이러한 분석을 바탕으로, 엔티티로 변환하여 조회하는 대신 **단순 DTO로 변환하여 조회**하도록 코드를 수정하기로 결정했다. 해당 로직은 단순히 사용자에게 응답하기 위한 데이터 조회이므로 **영속성 컨텍스트로 초기화할 필요 없이 DTO로 처리해도 문제없다고 판단**했다. 기존 로직을 DTO로 조회하도록 변경한 후 테스트를 진행했다.

## 2차 성능 측정

테스트 결과, 성공적으로 완료되었으며 **314ms가 소요**된 것을 확인했다. 즉, **API 성능이 약 1분 35초(94,727ms)에서 약 0.3초(314ms)로 약 300배 개선**되었다.

![2024-11-10-save-review-2](/assets/img/2024-11-10-save-review-2.png)

## 정리 및 회고

검수 정보 10,000개를 저장할 때 1분 35초가 소요되었으나, 데이터 처리 방식을 변경하고 엔티티 조회를 DTO 조회로 변경하여 **1초 이하로 줄여 300배 이상의 성능 개선**을 이루었다. 이번 버그 수정 작업을 통해 다음과 같은 네 가지를 배울 수 있었다.

1. 쿼리 튜닝이 아닌 **처리 방식의 변경으로도 성능을 개선**할 수 있다.
2. 필요한 데이터만 조회하고 변경해야 할 데이터만 수정하는 **네이티브 쿼리로 성능을 향상**시킬 수 있다.
3. 목록을 순회하며 저장하는 것보다 **목록 자체를 한 번에 저장하는 것이 성능에 유리**하다.
4. 조회 결과를 **단순 DTO로 조회하는 것이 엔티티로 조회하는 것보다 빠르다**.

앞으로는 데이터 유형에 **적합한 처리 방식을 선택**하고, 데이터를 조회할 때 **엔티티로 조회해야 하는지 DTO로 조회해야 하는지를 신중히 판단**하여 구현하도록 하자.
