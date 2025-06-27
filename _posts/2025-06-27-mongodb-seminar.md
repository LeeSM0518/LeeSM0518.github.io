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

