---
title: Java on Azure Day Seoul
date: 2025-04-23 19:00:00 +0900
categories: azure
tags:
  - java
  - azure
  - microsoft
description: Java On Azure Day Seoul 행사
---

![seminar1](/assets/img/java-on-azure-day-seoul.png)

## Empower Your Java Applications with Latest Microsoft Azure Technologies
---

### Github Copilot
---

- 프로젝트 분석을 요청하여 클래스 다이어그램이나 아키텍처 플로우 등을 출력시킬 수 있다.


<br/>

### Java SDK & Libraries for AI App Dev
---

- LangChain4J, spring-ai 를 활용하여 AI 서비스와 쉽게 연동할 수 있다.
- [Java on Azure Microsoft Learn Docs](https://learn.microsoft.com/en-us/azure/developer/java/)

<br/>

## 모던 자바와 데이터 지향 프로그래밍
---

- 도서
    - 개발자 원칙
    - Release의 모든 것
    - Data-Oriented Programming

<br/>

### 데이터 지향 프로그래밍
---

데이터 지향 프로그래밍은 비본질적 복잡도를 낮추는 프로그래밍 패러다임이다. 복잡도는 시스템 이해에 들어가는 인지 부하이다. 코드는 데이터를 조작하는 일이 대부분이라는 핵심에 집중한다.

<br/>

#### 데이터 지향 프로그래밍 원칙
---

데이터와 코드를 나누자. 데이터와 데이터 구조를 설명하는 것을 분리하라. 데이터 자료구조를 사용하고 불변하도록 구성하라.

<br/>

#### 1. 코드와 데이터를 분리하라
---

행위와 데이터 때문에 발생하는 의존이 존재한다. 클래스 간에 의존성은 데이터와 행위 때문에 의존이 발생하여 복잡해진다. 그러므로 데이터와 행위에 대한 의존을 나눠야 한다.

<br/>

#### 2. 데이터를 범용 자료구조로 표현하라
---

디버깅, 직렬화, 범용 함수를 사용한 조직, 정보 경로 활용에 유리하다.

한 가지 자료 구조를 다루는 함수 100개가 10가지 자료 구조를 다루는 함수 10개보다 낫다.

<br/>

#### 3. 데이터는 불변이다.

- 값과 불변 자료구조 중심
- 쓰레드 안전
- 부수 효과 회피
- 무상태인 연산 단계, 신규 버전 데이터 생성
- 상태를 다루는 반영 단계, 상태 전이
- 반영 단계에서 낙관적 동시성 제어와 충돌 조정
- 구조적 공유와 영속 자료 구조로 성능 문제 해결

<br/>

#### 4. 데이터 표현과 스키마는 분리하라
---

- 이종 맵의 단점 보완
- JSON 스키마를 사용한 유효성 검사
- 시스템 경계에서 데이터가 유효하다면 내부의 데이터 유효성은 별 문제가 안된다.

<br/>

### 모던 자바
---

- 람다 프로젝트 : 1급 함수 도입
    - 람다식, 함수 인터페이스, 메서드 참조, 스트림, 값 기반 클래스
- 앰버 프로젝트 : "쉬운 일은 쉽게 처리하게 만들자"
    - ADT(Algebraic data type) : 대수 자료형
        - 스위치식, 패턴 매칭, 레코드, 실드 클래스, 레코드 패턴 등

<br/>

### 자바 데이터 지향 프로그래밍
---

[Data Oriented Programming in Java](https://www.infoq.com/articles/data-oriented-programming-java/)

<br/>

1. 데이터를 변하지 않고 투명하게 모델링하라
    - 불변성 : 코드의 예측 가능성을 높이고 오류를 줄임
    - 투명성 : 객체가 상태를 숨기지 않고 그대로 드러냄
    - 명칭 있는 튜플, 레코드가 불변성과 투명성을 제공
2. 데이터를 다른 무엇이 아닌 데이터 그 자체로 모델링한다
    - 레코드와 밀봉 타입으로 ADT로 자유로운 데이터 표현
3. 규칙을 위반하는 상태는 표현할 수 없게 한다
4. 데이터와 행위를 분리하라

<br/>

자바빈은 데이터 객체가 아니다. 

<br/>

## Java AI 앱 개발 및 운영을 위한 Azure Container Apps 핸즈온 워크샵
---

### Java on Azure Container Apps
---

- kubernatest와 함께 사용되는 기술들
    - dapr : build apps using any language with any framework
        - MSA를 편리하게 구성할 수 있도록 하는 k8s 프레임워크
    - keda : event-driven autoscaling
        - http 수신량, cpu, memory 사용량들을 기반으로 오토스케일링
    - envoy : ingress