---
title: Code Smells
date: 2025-12-08 09:00:00 +0900
categories: refactoring
tags:
  - refactoring
description:
---

다음과 같은 내용들로 인해 코드에 악취가 날 수 있다.

<br/>

## 비대해진 것들 (Bloaters)
---

비대해진 것들은 너무 거대한 비율로 증가하여 작업하기 어려운 코드, 메서드 및 클래스이다. 일반적으로 이러한 악취는 바로 나타나지 않고, 프로그램이 발전함에 따라 시간이 지나면서 축적된다.

- 긴 메서드 (Long Method)
- 거대한 클래스 (Large Class)
- 기본형 집착 (Primitive Obsession)
- 긴 매개변수 목록 (Long Parameter List)
- 데이터 덩어리 (Data Clumps)

<br/>

## 객체 지향 남용 (Object-Orientation Abusers)
---

이 모든 악취는 객체 지향 프로그래밍 원칙의 불완전하거나 잘못된 적용이다.

- 서로 다른 인터페이스를 가진 대안 클래스 (Alternative Classes with Different Interfaces)
- 거부된 유산 (Refused Bequest)
- Switch 문 (Switch Statements)
- 임시 필드 (Temporary Field)

<br/>

## 변경 방해자 (Change Preventers)
---

이러한 악취는 코드의 한 곳에서 무언가를 변경해야 하면 다른 여러 곳에서도 많은 변경을 해야 한다는 것을 의미한다. 그 결과 프로그램 개발이 훨씬 더 복잡하고 비용이 많이 든다.

<br/>

## 불필요한 것들 (Dispensables)
---

불필요한 것은 무의미하고 불필요한 것으로, 그것이 없으면 코드가 더 깨끗하고 효율적이며 이해하기 쉬워진다.

- 주석 (Comments)
- 중복 코드 (Duplicate Code)
- 데이터 클래스 (Data Class)
- 죽은 코드 (Dead Code)
- 게으른 클래스 (Lazy Class)
- 추측성 일반화 (Speculative Generality)

<br/>

## 결합자 (Couplers)
---

이 그룹의 모든 악취는 클래스 간의 과도한 결합에 기여하거나 결합이 과도한 위임으로 대체될 때 어떤 일이 발생하는지 보여준다.

- 기능 욕심 (Feature Envy)
- 부적절한 친밀함 (Inappropriate Intimacy)
- 불완전한 라이프러리 클래스 (Incomplete Library Class)
- 메시지 체인 (Message Chains)
- 중개자 (Middle Man)

<br/>

## Reference
---

https://refactoring.guru/refactoring/smells