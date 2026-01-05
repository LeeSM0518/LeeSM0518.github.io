---
title: Code Smells
date: 2025-12-08 09:00:00 +0900
categories: refactoring
tags:
  - refactoring
description:
---

코드에 악취가 날 수 있는 상황들과 해결 방법들에 대해서 알아보자.

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

### 긴 메서드 (Long Method)
---

#### 징후와 증상
---

메서드가 10줄 이상의 코드를 포함한다.

<br/>

#### 문제의 원인
---

기존 메서드에 추가하는 것보다 새 메서드를 만드는 것이 더 어려우므로 기존 메서드에 코드가 계속해서 추가되어 스파게티 코드가 된다.

<br/>

#### 해결 방법
---

메서드에 대해 설명이 필요해질 경우 새 메서드로 분리해야 한다. 그리고 메서드에 설명적인 이름이 있으면, 해당 메서드가 무엇을 하는지 코드를 보지 않아도 된다.

<br/>

메서드 길이를 줄이려면 메서드 추출을 사용해라.

```java
// 문제
void printOwing() {
  printBanner();

  // Print details.
  System.out.println("name: " + name);
  System.out.println("amount: " + getOutstanding());
}

// 해결
void printOwing() {
  printBanner();
  printDetails(getOutstanding());
}

void printDetails(double outstanding) {
  System.out.println("name: " + name);
  System.out.println("amount: " + outstanding);
}
```

<br/>

지역 변수나 매개변수 때문에 메서드를 추출하기 어렵다면, 임시 변수를 질의로 바꾸기, 매개변수 객체 만들기, 객체 통째로 넘기기를 시도해라.

<br/>

임시 변수를 질의로 바꾸기 예시

```java
// 문제
double calculateTotal() {
  double basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  else {
    return basePrice * 0.98;
  }
}

// 해결
double calculateTotal() {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  else {
    return basePrice() * 0.98;
  }
}
double basePrice() {
  return quantity * itemPrice;
}
```

<br/>

매개변수 객체 만들기 예시

```java
// 문제
Int amountInvoicedIn(start: Date, end: Date)

// 해결
Int amountInvoicedIn(date: DateRange)
```

<br/>

객체 통째로 넘기기 예시

```java
// 문제
int low = daysTempRange.getLow();
int high = daysTempRange.getHigh();
boolean withinPlan = plan.withinRange(low, high);

// 해결
boolean withinPlan = plan.withinRange(daysTempRange);
```

<br/>

위 방법들이 모두 통하지 않는다면, 메서드를 메서드 객체로 바꾸기로 메서드 전체를 별도 객체로 옮기는 것도 방법이다.

<br/>

메서드를 메서드 객체로 바꾸기 예시

```java
// 문제
class Order {
  // ...
  public double price() {
    double primaryBasePrice;
    double secondaryBasePrice;
    double tertiaryBasePrice;
    // Perform logn computation.
  }
}

// 해결
class Order {
  // ...
  public double price() {
    return new PriceCalculator(this).compute();
  }
}

class PriceCalculator {
  private double primaryBasePrice;
  private double secondaryBasePrice;
  private double tertiaryBasePrice;
  
  public PriceCalculator(Order order) {
    // Copy relevant information from the
    // order object.
  }
  
  public double compute() {
    // Perform long computation.
  }
}
```

<br/>

조건문이나 반복문은 분리할 수 있다는 좋은 신호이다. 조건문에는 조건문 분해하기를, 반복문에는 메서드 추출을 적용해보자.

<br/>

조건문 분해하기 예시

```java
// 문제
if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
  charge = quantity * winterRate + winterServiceCharge;
}
else {
  charge = quantity * summerRate;
}

// 해결
if (isSummer(date)) {
  charge = summerCharge(quantity);
}
else {
  charge = winterCharge(quantity);
}
```

<br/>

#### 장점
---

코드 이해와 유지보수가 쉬워지고, 숨어있던 중복 코드를 찾기 쉽다.

<br/>

### 긴 매개변수 목록 (Long Parameter List)
---

#### 징후와 증상
---

메서드에 3~4개 이상의 매개변수가 있다.

<br/>

#### 문제의 원인
---

긴 매개변수 목록은 여러 유형의 알고리즘이 단일 메서드에 존재하는 경우에 발생할 수 있다.

이러한 목록은 이해하기 어렵고, 길어질수록 모순되고 사용하기 어려워집니다. 긴 매개변수 목록 대신 메서드는 자체 객체의 데이터를 사용할 수 있습니다.

<br/>

#### 해결 방법
---

매개변수를 메서드 호출로 대체
: 매개변수에 전달되는 값이 다른 객체의 메서드 호출 결과인 경우, 이 객체를 자체 클래스의 필드에 배치하거나 메서드 매개변수로 전달

전체 객체 보존
: 다른 객체에서 받은 데이터 그룹을 매개변수로 전달하는 대신 객체 자체를 메서드에 전달

매개변수 객체 도입
: 이러한 매개변수가 서로 다른 소스에서 오는 경우 단일 매개변수 객체로 전달

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

### 중복 코드 (Duplicate Code)
---

#### 징후와 증상
---

두 코드 조각이 거의 동일하게 작성된다.

<br/>

#### 문제의 원인
---

- 동시 개발
	- 여러 프로그래머가 동일한 프로그램의 다른 부분에서 동시에 작업할 때 발생
- 미묘한 중복
	- 코드의 특정 부분이 다르게 보이지만 실제로는 동일한 작업을 수행하는 경우
- 의도적 중복
	- 마감일을 맞추기 위해 서두르고 기존 코드가 작업에 "거의 맞을" 때 복사 붙여넣기의 유혹을 이기지 못할 수 있음
- 게으름
	- 일부 경우 프로그래머가 단순히 정리하기에 너무 게으름

<br/>

#### 해결 방법
---

- 같은 클래스 내 두 개 이상의 메서드 발견
	- 메서드 추출로 해결
- 같은 수준의 두 서브 클래스에서 발견
	- 일반 중복 코드일 경우 메서드 추출, 상위 클래스로 이동
	- 생성자 내 중복일 경우 상위 클래스 생성자 활용
	- 유사하지만 완전히 동일하지 않은 중복일 경우 템플릿 메서드 패턴 활용
	- 같은 작업을 다른 알고리즘 수행인 경우 최선의 알고리즘 선택 후 변경
- 서로 다른 두 클래스에서 발견
	- 계층 구조에 속하지 않는 클래스일 경우 슈퍼클래스로 추출하여 이전의 모든 기능을 유지하는 단일 상위 클래스 생성
	- 상위 클래스 생성이 어렵거나 불가능한 경우 한 클래스에서 클래스 추출을 사용하고 다른 클래스에서 새 컴포넌트를 사용
- 조건부 표현식에서 발견
	- 많은 조건 표현식이 같은 코드를 수행하는 경우 조건식 통합을 통해 연산자를 단일 조건으로 병합하거나 메서드 추출로 이해하기 쉬운 이름의 별도 메서드에 조건 배치
	- 조건 표현식의 모든 분기에서 같은 코드 수행할 경우 조건부 조각 통합을 통해 동일 코드를 조건 트리 외부에 배치


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