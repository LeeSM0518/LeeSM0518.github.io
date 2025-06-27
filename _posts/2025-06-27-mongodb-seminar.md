---
title: MongoDB Seminar
date: 2025-06-27 10:00:00 +0900
categories: mongodb
tags:
  - mongodb
description: MongoDB Seminar 내용 정리
---

## 1. 개요
---

회사에서 진행하는 MongoDB 세미나를 청강하고 내용을 정리한다.

<br/>

## 2. PSS Read
---

- Replica Set
	- 데이터를 3중으로 저장한다.
	- Member 간 주기적으로 hearbeat 체크 (2초)
	- Primary의 데이터가 Secondary 로 복제
		- 시점에 따라 Primary와 Secondary의 데이터가 달라질 수 있다.
- Replication
	- 하나의 Primary에 변경사항을 저장한다.
	- Operation Log(Oplog)에 작성한다.
	- Secondary는 Oplog를 모니터링하다가 변화가 발생할 경우 데이터를 불러온다.
- MongoDB 8.0 : 고가용성과 성능 개선
	- 그룹화된 일괄 작업을 단일 oplog 항목으로 그룹화
	- 더 빨라진 majority acknowledgement
	- 복제 지연 시간 단축
	- w:majority 워크로드에 대한 처리량 증가
- Replication Lag (Primary와 Secondary의 데이터 시간 차이) & Flow Control
	- Primary와 Secondary의 리전이 다를 경우 Lag Time이 늘어날 수 있다.
	- flowControl 은 Lag가 지정된 시간으로 유지되도록 ticket을 조정 한다. (default: 10초)
		- Primary에서 쓰로틀링을 걸어서 데이터가 insert 되는 것을 막는 기능을 한다.
- PSS working
	- 한국과 미국에 클러스터 구성을 요청 할 경우
		- 서울에 네트워크(Primary 1, Secondary 2)가 구성된다.
		- 미국에 네트워크를 구성된다.
	- 다른 리전 간에 복제를 할 경우 Lag가 발생한다.
	- 내부망을 사용할 경우 피어링을 사용해서 연결해야 한다.
- 멀티 리전에 복제 해놨을 때의 Lag을 해결하기 위한 방법은?
	- 실시간으로 하는 것은 힘들다
	- 전파가 오래 걸리더라도 다 전파되고 저장을 완료하도록 함
- Write Concern
	- 중요한 데이터에 대해서 안정적으로 쓸 수 있도록 설정할 수 있다.
	- Replica Set 내에서 데이터를 복제에 대한 Tuning Option (default: majority)
	- Connection을 만들 때 정의할 수 있으며 Write 시에 이를 overwrite 할 수 있음
	- 쿼리에 대해서만 writeConcern을 지정해서 요청할 수 있음
	- Secondary의 저장 순서까지 보장하려면 Primary에서만 저장, 조회를 수행하도록 하면 된다.
- Read Preference & Concern
	- readPreference로 Secondary를 활용
	- Read Concern
		- local, available : 해당 인스턴스에 데이터 반환
		- majority : 읽은 데이터가 리플리카셋 과반수가 승인 했음을 보장
		- linearizable : 읽기 작업이 시작되기 전에 완료되고 과반수 이상이 확인한 데이터를 반환
- Read Node 연계
	- Read Concern : local, Write Concern : majority
		- 쓰기 작업시 과반수 노드에 데이터 적재
		- 읽기 작업시 local에 저장된 내용에서 수행 (최신 데이터를 가져오지 못 할 수 있음)
	- Read Concern : local, Write Concern : 1
		- 쓰기 작업에 대한 내구성이 부족
		- 읽기 작업시 순차 및 장애 사항에 대한 보장이 않됨
- Best Practice
	- 60초 넘는 트랜잭션을 롤백
	- 트랜잭션은 1000개 이상으로 묶지 않는 것이 좋다
	- 알아서 Retry를 수행한다
	- 삽입할 때 insertOne 보다는 insertMany로 하는 것이 좋다
	- 데이터의 특징마다 Write Concern을 조저ㄹ하면 좋다
	- Causal consistency : 순차적인 저장 및 조회 보장
		- 데이터 보장이 필요한 API만 primary에서 read & write를 수행하도록 설정하는 것이 좋다.

<br/>

## 3. Index
---

- 인덱스를 두 개의 필드를 만들 경우 MongoDB는 그 중 하나만 선택해서 조회하는 방식이다.
- 두 개의 필드가 A, B 일 경우 A와 A, B로 조회하면 인덱스를 타지만 B로 조회하면 인덱스를 타지 않는다.
- 복합 인덱스 상에서 조회한 후 정렬할 경우 오래 걸릴 수 있다.
- 배열에 인덱스를 걸어도 문제 없다.
- 특정 속성이 부합하는 것만 인덱스로 생성할 수 있도록 조건 인덱스를 만들 수 있다.

<br/>

## 4. Pattern & Lock
---

### 4.1. Pattern
---

 - `Extended Reference Pattern` : ex) 주문 내에 주문자(사용자) 정보
	 - 주소, 이름 정보를 주문에 같이 저장하는 방법
	 - 조인 횟수를 줄이기 위해 미리 넣어두는 것
	 - 데이터를 중복 저장해야 함
	 - 바뀌어도 상관없는 데이터를 넣는 것이 좋을 것 같다
		 - 트리거나 변경 스트림을 통해 관련 데이터를 변경시킬 수 있다.
 - `Subset Pattern` : ex) 가장 최신의 코멘트
	 - Problem
		 - 너무 많은 반복적인 조인
	 - Solution
		 - Lookup 하는 컬렉션 쪽에서 필드를 정의
		 - 해당 필드를 메인 오브젝트로 가져옴
 - `Bucket Pattern` : ex) 센싱 데이터
	 - 데이터를 그룹화해서 저장함
	 - 배열에 push
	 - time collection을 활용할 수도 있음
 - `Computed Pattern` : ex) 한 시간 동안의 평균 온도
	 - 미리 계산해두고 도큐먼트에 저장
	 - 주로 Bucket Pattern과 같이 사용됨
 - `Attribute Pattern` : Array 형태로 필드를 묶음
	 - `[{"k": "height", "v": 16}]`
	 - attribute에만 인덱스를 걸면 통합 인덱스로 활용할 수 있음
 - `Archive Pattern` : 오래된 문서를 아카이브 할 때 사용
	 - 오래된 문서를 더 저렴한 계층에 저장
	 - Archive는 Data Federation에서 조회할 수 있음
	 - DB를 나눌 수 있고, Archive 옵션을 사용할 수도 있음
 - `Schema Versioning Pattern` : 스키마 구조가 매번 바뀔 수 있을 경우
	 - 스키마 버전으로 스키마 구조를 다르게 구성

<br/>

### 4.2. Lock
---

- MongoDB Lock & WiredTiger Lock
- Collection 수준까지의 Lock 시스템
	- Multi-granularity Lock 시스템 : 
		- MongoDB Lock : Global, Database, Collection, 
		- WiredTiger Lock : Document
	- 잠금 종류 : Intent Shared, Intent Exclusive, Shard, Exclusive
	- updateOne 했을 경우 : Document에 대해 쓰기 잠금 (Exclusive Lock) 획득이 필요
		- Global, Database, Collection에 대해서 Intent Exclusive Lock이 발생
		- 다른 컬렉션의 Intent Shard, Intent Exclusive lock 은 허용
	- findOne 했을 경우 : 별도의 lock이 없음
		- Global, Database, Collection에 대해서 Intent Shard lock
		- 다른 컬렉션의 Intent Shared, Intent Exclusive, Shared 은 허용
- Document 수준의 Lock 시스템
	- Transaction (Read & Write) Concurrency
		- 읽기/쓰기 각 최대 128개 티켓
	- Document Level Concurrency
		- WiredTiger는 Document 레벨의 동시성 제어를 함
		- 다수의 클라이언트가 동시에 다른 문서를 수정할 수 있음
		- Optimistic concurrency control을 사용
		- Global, Database, Collection 수준의 intent lock만 사용
			- 두 작업 간의 충돌을 감지하면, 한 작업이 쓰기 충돌이 발생하고 해당 작업을 재시도
	- MVCC (MultiVersion Concurrency Control)
		- 데이터에 대한 특정 시점 스냅샷을 제공
		- 체크포인트(60s)으로 데이터를 디스크에 기록
		- 데이터가 수정되는 경우 기존 데이터가 유지되고 변경된 데이터가 추가된다
		- 트랜잭션 아이디로 요청 시점에 따라 이전 트랜잭션과 이후 트랜잭션의 데이터를 제공함