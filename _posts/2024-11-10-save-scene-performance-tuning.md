---
title: 위성 영상 저장 성능 개선
date: 2024-11-10 15:00:00 +0900
categories: troubleshooting
tags:
  - troubleshooting
  - performance-tuning
description: 데이터 동기화 방식과 배치 주기를 변경하여 저장 시간을 20분에서 1초로 개선
---

## 개요

회사에서 위성 영상 저장의 성능을 개선한 작업에 대해 기록하고 회고한다.

## 배경 및 목적

폐쇄망 환경의 **고객으로부터 위성 영상을 저장하면 약 40분 뒤에 조회된다는 피드백**을 받아 고객사로 출장을 가게 되었다. 고객사 서버에서 로그를 확인해보니, 영상과 함께 저장되는 영상 통계 테이블의 **INSERT 쿼리가 20분 소요**되는 것을 발견했다. 회사로 복귀하여 고객사에서 슬로우 쿼리가 발생하지 않도록 **INSERT 쿼리를 1분 이내로 개선하는 것을 목표로 작업을 진행**했다.

## 현상 재현

테스트 서버에서도 동일한 현상이 발생하는지 확인하기 위해 **영상 5만 개를 저장**하여 고객사와 **유사한 환경을 구성**했다. 테스트 결과, 동일하게 영상 통계 테이블의 INSERT 쿼리에서 **슬로우 쿼리가 발생하는 것을 확인**했다.

## 원인 분석

슬로우 쿼리가 발생하던 쿼리는 다음과 같으며, 영상 하나를 영상 통계 테이블에 입력하는 단순한 입력 쿼리였다.

```sql
INSERT INTO scene_label_view 
(scene_id, label_min_width, label_max_width, label_min_height, label_max_height)
SELECT s.name, 0, 0, 0, 0   
FROM scene s   
WHERE s.name NOT IN (SELECT scene_id FROM scene_label_view)   
ON CONFLICT (scene_id) DO NOTHING;
```

SELECT 문이 오래 걸리는 것인가 해서 따로 실행해봤으나, **평균적으로 200ms**로 완료되는 것을 확인했다. 이처럼 쿼리가 단순하고 조회 쿼리가 빠르게 완료되는 것을 보아 **쿼리 문제가 아닐 수도 있겠다**는 생각이 들었다.

그렇다면 DB에 문제가 있는 건가 해서 다음 쿼리를 통해 DB에 연결되어 있는 **세션 정보를 확인**해봤더니 **LWLock이 발생**한 것을 알 수 있었다.
```sql
SELECT * FROM pg_stat_activity;
```

> **LWLock(Lightweight Lock)** : PostgreSQL에서 공유 메모리 구조에 대한 동시 접근을 제어하기 위해 사용되는 잠금 메커니즘이다. 이는 데이터베이스의 다중 프로세스 아키텍처에서 일관된 읽기 및 쓰기를 보장하기 위해 필수적이다.[^percona]
{: .prompt-info }

<br/>

LWLock이 왜 발생하는지 찾아보던 중, "PostgreSQL Autovacuum 장애 대응기"[^29cm] 글을 보고 **Autovacuum에 의해 LWLock이 발생할 수 있음**을 알게 되었다.

> **Autovacuum** : 데이터베이스의 성능과 안정성을 유지하기 위해 자동으로 VACUUM과 ANALYZE 명령을 실행하는 기능이다. [^autovacuum]
{: .prompt-info }

> **VACUUM** : 테이블과 인덱스 내의 불필요한 데이터(죽은 튜플)를 제거하여 디스크 공간을 회수하고 데이터베이스 성능을 최적화하는 명령어이다. [^vacuum]
{: .prompt-info }

<br/>

Autovacuum이 어디서 발생하는지 확인하기 위해 코드를 분석해보니, 영상 통계 테이블을 배치로 업데이트하는 쿼리에 문제가 있음을 알 수 있었다. 배치 쿼리는 다음과 같으며 **10초마다 데이터를 제거, 수정, 삽입하여 Autovacuum이 발생**했음을 알 수 있다.


```sql
-- 삭제된 영상을 통계 테이블에서 제거
DELETE  
FROM scene_label_view  
WHERE scene_label_view.scene_id NOT IN  
      (SELECT name FROM scene);  

-- 영상에 탐지된 객체가 존재할 경우 영상들의 통계 정보를 업데이트
INSERT INTO scene_label_view  
(scene_id, label_min_width, label_max_width, label_min_height, label_max_height) 
SELECT scene_id,  
       MIN(ld.meter_width)  label_min_width,  
       MAX(ld.meter_width)  label_max_width,  
       MIN(ld.meter_height) label_min_height,  
       MAX(ld.meter_height) label_max_height  
FROM label_data ld  
GROUP BY ld.scene_id  
ON CONFLICT (scene_id)  
    DO UPDATE SET label_min_width  = EXCLUDED.label_min_width,  
                  label_max_width  = EXCLUDED.label_max_width,  
                  label_min_height = EXCLUDED.label_min_height,  
                  label_max_height = EXCLUDED.label_max_height;  

-- 새로 입력된 영상을 통계 테이블에 반영
INSERT INTO scene_label_view  
(scene_id, label_min_width, label_max_width, label_min_height, label_max_height) 
SELECT s.name, 0, 0, 0, 0  
FROM scene s  
WHERE s.name NOT IN (SELECT scene_id FROM scene_label_view)  
ON CONFLICT (scene_id) DO NOTHING;

```


## 성능 개선

Autovacuum이 자주 발생하지 않도록 영상 통계 테이블의 **데이터 동기화 방식을 다음과 같이 변경**했다.

1. 배치 처리로 생성과 삭제를 반영하는 것이 아닌, 해당 **작업이 발생했을 때 한 번만 수행하도록 수정**
2. 고객사가 사용하지 않는 시간대에 매일 한 번만 통계 테이블을 업데이트하도록 **배치 주기를 변경**

이렇게 수정한 이유는 영상 추가나 삭제가 **발생하지 않았음에도 배치 작업을 수행하는 것은 불필요한 연산**이며, 고객사의 서비스 사용 패턴을 살펴봤을 때 **통계 테이블이 빠르게 반영될 필요가 없을 것으로 판단**했기 때문이다.

## 성능 측정

위와 같이 작업을 마치고 코드를 빌드하여 테스트 서버에 배포한 뒤 정상 동작 여부와 성능을 확인했다. 영상 저장이 잘 되는 것이 확인됐으며, 영상 전처리 작업을 제외하고 **DB 저장 시간이 1초 내로 완료**되는 것을 볼 수 있었다. 이로써, 영상 저장의 소요 시간이 **20분에서 1초로 줄어들었으며 약 1,200배 개선**되었다.

## 정리 및 회고

영상 저장 시간이 20분이 소요되었으나, 통계 테이블의 데이터 동기화 방식과 배치 주기를 변경하여 1초 내로 줄여 **1,200배의 성능 개선**을 이루었다. 이번 작업을 통해 다음 두 가지를 배울 수 있었다.

1. 테이블에 많은 데이터 변경이 발생할 경우 **Autovacuum이 발생하여 쿼리에 영향을 줄 수 있다**.
2. **배치 작업의 시간대와 주기를 적절하게 설정**하는 것이 중요하다.

이를 바탕으로 앞으로는 **테이블에 변경 작업을 최소화하고 고객의 사용 패턴에 맞는 적절한 배치 처리를 구현**하도록 하자.

---

# Reference

[^percona]: [percona: lightweight locks](https://www.percona.com/blog/postgresql-locking-part-3-lightweight-locks/)
[^29cm]: [29CM: PostgreSQL Autovacuum 장애 대응기](https://medium.com/29cm/postgresql-autovacuum-%EC%9E%A5%EC%95%A0-%EB%8C%80%EC%9D%91%EA%B8%B0-1-8284955c0193)
[^autovacuum]: [우아한형제들: PostgreSQL Vacuum에 대한 거의 모든 것](https://techblog.woowahan.com/9478/)
[^vacuum]: [PostgreSQL: VACUUM](https://www.postgresql.org/docs/current/sql-vacuum.html)