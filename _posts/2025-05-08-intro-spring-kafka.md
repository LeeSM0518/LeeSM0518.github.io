---
title: Intro to Apache Kafka with Spring
date: 2025-05-08 19:00:00 +0900
categories: spring
tags:
  - spring
  - kafka
description: Kafka에 대한 Spring의 지원과, 네이티브 Kafka Java 클라이언트 API 위에 제공되는 추상화 수준을 다룬다.
---

## 1. Overview
---

Apache Kafka는 분산형이며 장애 허용(fault-tolerant)을 갖춘 스트림 처리 시스템이다.

Spring Kafka는 `KafkaTemplate` 을 통한 일반적인 Spring 템플릿 프로그래밍 모델과, `@KafkaListener` 애노테이션을 사용한 메시지 기반 POJO 구성 방식을 제공한다.

<br/>

## 2. What Is a Listener Container in Spring for Apache Kafka?
---

빈(Bean)은 Spring IoC 컨테이너에 의해 인스턴스화되고 구성 및 관리되는 객체이며, 컨테이너는 이러한 빈의 생성, 설정, 조합을 담당하는 애플리케이션 컨텍스트이다.

<br/>

Apache Kafka에서의 리스너 컨테이너는 Kafka 메시지를 소비하는 소비자(Consumer)를 포함하는 컨테이너이다. Spring for Apache Kafka는 이 리스너 컨테이너를 생성하기 위해 컨테이너 팩토리를 사용한다. `@KafkaListener` 애노테이션을 사용하여 메서드를 Kafka 메시지 리스너로 지정할 수 있으며, 이 메서드들을 기반으로 컨테이너 팩토리가 리스너 컨테이너를 생성한다.

<br/>

## 3. Installation and Setup
---

다음 의존을 추가한다.

```kotlin
implementation("org.springframework.kafka:spring-kafka")
```

<br/>

## 4. Producing Messages
---



<br/>

## Reference
---

[Baeldung: Intro to Apache Kafka with Spring](https://www.baeldung.com/spring-kafka)
