---
title: Stripe Docs
date: 2025-12-10 09:00:00 +0900
categories: stripe
tags:
  - stripe
  - docs
description: 
---

Stripe은 인터넷 비즈니스를 위한 **결제 인프라 플랫폼**입니다. 개발자가 복잡한 결제 시스템을 직접 구축하지 않고도 API 몇 줄로 신용카드 결제, 구독, 송금 등을 구현할 수 있게 해줍니다.

<br/>

## API 둘러보기
---

Stripe API 객체들이 어떻게 연결되는지 살펴보고, 이를 조합하는 모범 사례를 알아보세요.

<br/>

### 핵심 개념
---

#### 모든 것은 객체이다
---

Stripe 계정의 모든 것은 API로 생성하든 아니든 객체입니다. 잔액은 Balance 객체에 해당하고, 고객은 Customer 객체로 추적하며, 결제 정보는 PaymentMethod 객체에 저장합니다.

<br/>

#### 객체에는 생명주기가 있다.
---

Stripe 연동은 복잡한 프로세스를 처리합니다.

API는 각 프로세스를 추적하기 위해 단일 객체를 사용합니다. 프로세스 시작 시 객체를 생성하고, 매 단계 후에 `status` 를 확인하여 다음에 무엇이 필요한지 알 수 있습니다.

예를 들어, 결제를 완료하는 동안 고객이 여러 결제 수단을 시도할 수 있습니다. 하나의 결제 수단이 실패하면, `requires_payment_method` 상태를 통해 고객에게 다른 결제 수단을 요청해야 함을 알 수 있습니다.

<br/>

#### 연동은 협력하는 객체들로 구성된다.
---

결제를 받으려면 시스템이 여러 핵심 객체를 생성하고 여러 상태를 거쳐 관리해야 합니다.

Stripe 연동은 Stripe와 통신하여 이러한 생성과 관리를 처리하는 시스템입니다.

일부 연동은 그 이상의 작업을 수행합니다. 하지만 핵심 결제 기능은 여전히 동일한 객체와 단계에서 나오며, 그 핵심 주위에 더 많은 객체가 추가됩니다.

<br/>

### 결제 객체
---

Stripe는 결제를 처리하기 위해 다양한 관련 객체를 사용합니다. 특정 요구사항에 맞는 연동을 구축하려면 먼저 이러한 객체들이 어떻게 함께 작동하는지 익혀야 합니다.

Stripe 결제 객체는 결제 과정을 관리하고 추적하는 다양한 도구로, 낮은 수준의 'Charge', 'Payment Intent' 부터 높은 수준의 'Invoice', 'Subscription', 'Checkout Session', 'Payment Link' 까지 다양한 기능을 제공하며 서로 연동되어 작동합니다.

![stripe11](/assets/img/stripe11.png)

<br/>

#### Charges (요금, 저수준 객체)
---

정의
: Stripe 계정으로 돈을 이동시키려는 단일 시도를 나타냅니다.

포함 범위
: 고객으로부터 수입 결제(카드 결제, 은행 이체 등)뿐만 아니라, Stripe Connect를 사용할 때 다른 Stripe 계정으로부터 Stripe 계정으로 돈이 이동하는 경우도 포함합니다.

ID 접두사
: ID는 `ch_` 또는 `py_` 접두사로 시작합니다.

접근 방식
: 사용된 접두사에 관계없이 Charges API를 통해 접근합니다.

<br/>

#### Payment Intents (결제 의도, 저수준 객체)
---

정의
: 여러 시도나 연장된 기간에 걸쳐 특정 금액을 지불하려는 의도를 추적하도록 설계된 객체입니다.

**기능**

- 다양한 결제 지원 : 3D Secure 인증 유무에 따른 카드 결제, 은행 이체, 디지털 지갑 결제, 바우처 등 다양한 동기식 및 비동기식 결제 형태를 지원합니다.
- ID 접두사 : 객체 ID는 `pi_` 접두사를 가집니다.
- Charges 생성 : Payment Intent가 확인되거나 그 이후에 돈을 이동시키려고 시도할 때 Charges를 생성합니다.
- 다중 시도 : 결제에 여러 시도가 필요할 수 있으므로, 하나의 Payment Intent는 여러 개의 Charges를 생성할 수 있습니다.

특징
: Payment Intents와 Charges는 모두 저수준 결제 객체이며, 고수준 객체들의 강력한 기반을 제공합니다.

<br/>

#### Invoices (명세서, 고수준 객체)
---

정의
: 고객이 빚진 금액에 대한 상세 명세서 역할을 하며, 외부에서 접하는 일반적인 송장과 매우 유사합니다.

**기능**

- 명세서 역할 : 라인 항목 지정, 고객 정보, 세금 징수 등을 허용합니다.
- ID 접두사 : 객체 ID는 `in` 접두사를 가집니다.
- Payment Intent 생성 : 결제가 필요할 때, 청구할 총액을 징수하기 위해 Payment Intents를 생성합니다.

<br/>

#### Subscriptions (구독, 고수준 객체)
---

정의
: 반복적인 결제를 용이하게 하는 Invoice 생성 엔진 역할을 합니다.

**기능**

- 자동 청구 : Subscription 기간마다 Stripe가 고객에게 자동으로 Invoices를 생성하도록 설정할 수 있습니다.
- ID 접두사 : 객체 ID는 `sub_` 접두사를 가집니다.
- 관계 : Subscriptions와 Invoices는 반복적인 수익을 징수하기 위해 설계된 Billing 객체이지만, Subscription 없이 일회성 Invoices도 생성 가능합니다.

<br/>

#### Checkout Sessions (체크아웃 세션)
---

Stripe Checkout은 전환율에 최적화된 호스팅 결제 페이지이며, 이를 다루는 객체들이 존재합니다.

정의
: Checkout Sessions는 Stripe Checkout의 특정 인스턴스에 대한 구성 및 상호 작용을 처리합니다.

처리 내용
: 구매할 제품, 해당 제품의 가격, 배송지 주소, 배송지 주소 수집, 프로모션 코드, 세금 등을 포함합니다.

ID 접두사
: 객체 ID는 `cs_` 접두사를 가집니다.

**결제 흐름 구성**

- 일회성 결제 : 단일 일회성 결제를 위해 구성될 경우 Payment Intent를 생성합니다.
- 일회성 Invoice : 일회성 결제를 위해 Invoices를 생성하도록 구성할 수 있습니다.
- 반복 결제 : 반복 결제를 용이하게 하기 위해 Subscription을 생성하도록 구성할 수도 있습니다.

<br/>

#### Payment Links (결제 링크)
---

정의
: Payment Links는 방문 시 Checkout Sessions를 생성하는 공유 가능하고 재사용 가능한 Stripe 호스팅 URL을 관리합니다.

용도
: Stripe Checkout을 사용하여 동일한 항목을 여러 사람에게 반복적으로 판매하고 싶을 때 유용합니다.

ID 접두사
: 객체 ID는 `plink_` 접두사를 가집니다.

관계
: Payment Links는 Checkout Sessions를 생성합니다.

<br/>

#### 고수준 객체의 특징 및 권장 사항
---

Payment Links, Checkout Session, Subscriptions, Invoices는 모두 고수준 객체에 해당합니다.

**제공 기능**

- 세금 계산 및 징수, 배송 세부 정보 수집, 배송 요율 적용, 할인 계산, 고객 대상 프로모션 코드 지원 등 다양한 유용한 기능을 제공합니다.
- 상세한 라인 항목을 제공하며, Stripe의 Product 및 Price 객체를 활용합니다.
- 반복 결제와 같은 유용한 자동화 기능을 제공합니다.

**권장 사항**

- 가능한 한 고수준 객체 사용을 권장합니다.
- 세금, 배송, 할인이 필요한 경우, 지불해야 할 금액 계산에 Stripe의 도움을 받기 위해 고수준 객체를 사용하는 것이 유용합니다.

시작점
: 여기에 표시된 객체 중 어떤 것이든 통합의 시작점이 될 수 있습니다.

<br/>

#### 객체 간의 다양한 결제 흐름 예시
---

객체들은 유연하게 조합되어 다양한 결제 흐름을 만들 수 있습니다.

<br/>

1. Payment Link 시작 흐름 (반복 결제 예시) : Subscription을 생성하도록 구성하는 경우의 순서
    1. Payment Link가 방문되면 Checkout Session을 생성합니다.
    2. Checkout Session은 Subscription을 생성합니다.
    3. Subscription은 Invoice를 생성합니다.
    4. Invoice는 청구할 총액을 징수하기 위해 Payment Intent를 생성합니다.
    5. 최종적으로 Payment Intent는 총액을 징수하려고 시도할 때 Charge를 생성합니다.

**Payment Link -> Checkout Session -> Subscription  -> Invoice -> Payment Intent -> Charge**

<br/>

2. 다른 시작점 선택 가능성 : 반드시 Payment Link에서 시작할 필요는 없습니다.
    1. Checkout Session 직접 생성
    2. Subscription 직접 생성 : Checkout Sessions를 건너뛰고 Subscriptions를 직접 생성할 수 있습니다.
    3. Invoice 직접 생성 : Subscription 없이 Invoices를 직접 생성할 수 있습니다.
    4. 일회성 결제 흐름
        1. Subscription을 생성하지 않고 일회성 결제를 징수하도록 Payment Links를 구성할 수 있습니다.
        2. Checkout Sessions를 직접 생성하여 일회성 결제를 처리할 수 있습니다.
        3. Checkout이 일회성 결제를 위해 Invoices를 생성하도록 구성할 수도 있습니다.

<br/>

#### 저수준 객체 직접 사용 시나리오
---

모든 고수준 기능을 자체적으로 처리하고자 할 경우, 고수준 객체 없이 Payment Intents를 직접 생성할 수 있습니다.

1. Payment Intents 직접 생성
    1. 사용 지점 : 징수하고자 하는 정확한 금액을 이미 알고 있을 때 Payment Intents를 직접 생성해야 합니다.
    2. 제한 사항 : Payment Intents는 라인 항목이나 할인과 같은 고수준 기능을 가지고 있지 않지만, 사용자가 세금, 배송 또는 반복 결제 일정을 자체적으로 처리하고자 할 때 자체적인 고수준 추상화를 구축하기 위한 훌륭한 기반을 제공합니다.
2. Charges 직접 생성
    1. 권장하지 않음 : Charges를 직접 생성하는 것은 기술적으로 가능하지만, 현재는 사용지 중단되었으며 권장되지 않습니다.
    2. 이유 : Charges 직접 생성은 레거시 통합 경로이며, 다른 결제 객체들이 지원하는 결제 수단 및 결제 흐름의 일부 작은 하위 집합만 지원합니다.
3. 범위 : 이 개요는 가능한 모든 사용 사례를 포괄하는 포괄적인 목록은 아닙니다.

<br/>

#### 최종 권장 사항
---

결제 객체의 설계 목적과 상호 관계를 이해했으므로, 사용 사례에 가장 적합한 객체를 선택하여 요구사항에 맞는 통합을 구축할 수 있습니다.

1. 선택 가이드
    1. 저수준 기반 : 구축할 기반이 필요하다면 Payment Intents를 직접 생성할 수 있습니다.
    2. 최적의 활용 : 하지만 Stripe가 제공하는 기능을 최대한 활용하기 위해서는 가능한 한 고수준 객체와 기능을 사용하는 것을 권장합니다.

<br/>

### 결제까지의 경로
---

Stripe 연동에서 모든 결제는 PaymentIntent라는 객체를 사용합니다. 이름에서 알 수 있듯이, 결제를 수금하려는 의도를 나타냅니다. 이 객체는 그 의도를 실현하기 위해 거치는 단계들을 추적합니다.

예를 들어, 고객이 장바구니에 100 USD 상품을 담고 결제하기 버튼을 클릭한다고 가정해 봅시다. 아직 구매한 것은 아니며, 영영 구매하지 않을 수도 있습니다. 하지만 결제하기를 클릭한 것은 구매 의도를 나타냅니다. 이 시점에서 연동은 나머지 프로세스를 추적하기 위해 100 USD 금액의 `PaymentIntent` 객체를 생성합니다.

`PaymentIntent` 가 성공에 이르는 경로는 여러 상태를 거칩니다. 간략히 살펴보면 다음과 같습니다.

1. requires_payment_method
2. requires_confirmation
3. processing
    1. succeeded (성공)
    2. canceled (실패)

<br/>

#### 결제 수단 (Payment Methods)
---

PaymentIntent는 `requires_payment_method` 상태로 시작합니다. 앞으로 진행하려면 Stripe에 고객의 결제 수단에 대한 정보가 필요합니다. 카드 번호나 다른 결제 시스템의 자격 증명 등입니다.

연동에서는 이러한 정보를 PaymentMethod라는 API 객체로 나타냅니다. 일부 연동에서는 이 객체를 생성하고 PaymentIntent에 연결하는 코드를 직접 작성합니다. 다른 연동에서는 Stripe가 정보를 수집하고 대신 작업을 수행합니다. 또한 Setup Intents API를 사용하여 향후 PaymentIntent에서 사용할 결제 수단을 생성하고 저장할 수도 있습니다.

<br/>

#### 확인 (Confirmation)
---

다음 상태는 `requires_confirmation` 입니다. 대화형 결제 플로우에서 고객은 결제 의사가 있고, 제공한 수단으로 결제하겠다는 것을 확인해야 합니다. 일회성 온라인 결제에서는 보통 결제 버튼을 클릭할 대 이루어집니다.

고객이 결제를 클릭하거나 의도를 확인하면, 연동이 API 호출로 Stripe에 알립니다. 일부 연동에서는 이 호출을 하는 코드를 직접 작성합니다. Stripe는 유연한 커스텀 연동을 구축하면서도 이를 가능하게 하는 Stripe Elements라는 드롭인 UI 요소를 제공합니다. Stripe Checkout 이나 Payment Links 연동 같은 다른 연동에서는 Stripe가 호출을 하고 다음 단계를 처리합니다. Stripe를 연동하고 다양한 객체를 조합하여 사용 사례를 처리하는 방법은 여러 가지가 있습니다.

대부분의 경우 PaymentIntent가 확인되면 특정 자금 이동 시도를 나타내는 Charge가 생성됩니다. Charge는 성공하거나 실패할 수 있습니다. 실패하면 보통 새 결제 정보로 PaymentIntent를 다시 확인하여 결제를 재시도할 수 있습니다. 새 PaymentIntent를 생성할 필요 없이 즉시 재시도를 허용하면 전환율이 높아지는 경향이 있습니다.

<br/>

#### 처리 및 성공
---

인텐트의 상태는 이제 `processing` 이며, 이 시점에서 Stripe가 결제 처리를 시도합니다.

Stripe는 항상 이 부분을 대신 수행하며, 여러 단계가 있을 수 있습니다. 단계를 거치면서 결과에 따라 인텐트의 상태를 업데이트합니다: `succeeded` 이거나 실패하면 다시 `requires_payment_method` 입니다.

완료되면 마지막 객체가 등장합니다. `Event` 객체를 사용하여 활동을 나타냅니다. 이 경우 활동은 "과금이 성공했다" 또는 "과금이 실패했다" 일 수 있습니다. 일부 연동에서는 웹훅 엔드포인트를 사용하여 이벤트에 응답하는 커스텀 코드를 작성합니다. Checkout이나 Payment Links 연동 같은 다른 연동에서는 Stripe가 이벤트를 수신하고 미리 작성된 응답을 제공합니다.

<br/>

## Payment Intents API
---

Payment Intents API를 사용하여 PaymentIntent의 생명주기 동안 상태가 변하는 복잡한 결제 플로우를 처리할 수 있는 연동을 구축하세요. 이 API는 생성부터 결제 완료까지 결제를 추적하고, 필요할 때 추가 인증 단계를 트리거합니다.

Payment Intents API 사용의 장점
- 자동 인증 처리
- 이중 과금 방지
- 멱등성 키 문제 없음
- 강력한 고객 인증(SCA) 및 유사한 규제 변경 지원

<br/>

#### 완벽한 API 세트
---

Payment Intents API를 Setup Intents 및 Payment Methods API와 함께 사용하세요. 이 API들은 동적 결제(ex. 3D Secure)를 처리하고, 새로운 규제와 지역별 결제 수단을 지원하면서 다른 국가로 확장할 준비를 할 수 있게 해줍니다.

Payment Intents API로 연동을 구축하는 것은 두 가지 작업으로 구성됩니다: PaymentIntent 생성과 확인. 각 PaymentIntent는 일반적으로 애플리케이션의 단일 장바구니 또는 고객 세션에 해당합니다. PaymentIntent는 지원되는 결제 수단, 수금할 금액, 원하는 통화 등 거래에 대한 세부 정보를 캡슐화합니다.

<br/>

#### PaymentIntent 생성하기
---

서버에서 PaymentIntent를 생성하고 전체 PaymentIntent 객체 대신 클라이언트 시크릿(Stripe가 PaymentIntent의 일부로 반환하는 고유키, PaymentIntent의 상태, 금액, 통화에 접근할 수 있습니다)을 클라이언트에 전달하는 방법을 설명합니다.

PaymentIntent를 생성할 때 금액과 통화 같은 옵션을 지정할 수 있습니다.

```java
// 시크릿 키 설정
Stripe.apiKey = "***";

PaymentIntentCreateParams params =
  PaymentIntentCreateParams
    .builder()
    .setAmount(1099L)
    .setCurrency("usd")
    .build();
    
PaymentIntent paymentIntent = PaymentIntent.create(param);
```

<br/>

**모범 사례**

- 고객이 결제 프로세스를 시작할 때처럼 금액을 알게 되면 바로 PaymentIntent를 생성하는 것을 권장합니다. 구매 퍼널을 추적하는 데 도움이 됩니다. 금액이 변경되면 amount를 업데이트할 수 있습니다. 예를 들어, 고객이 결제 프로세스를 중단하고 장바구니에 새 상품을 추가하면, 다시 결제 프로세스를 시작할 때 금액을 업데이트해야 할 수 있습니다.
- 결제 프로세스가 중단되었거나 나중에 재개되면, 새로 생성하는 대신 동일한 PaymentIntent를 재사용하세요. 각 PaymentIntent에는 필요할 때 조회하는 데 사용할 수 있는 고유 ID가 있습니다. 애플리케이션의 데이터 모델에서 고객의 장바구니나 세션에 PaymentIntent의 ID를 저장하여 쉽게 조회할 수 있습니다. PaymentIntent를 재사용하면 객체 상태가 해당 장바구니나 세션에 대한 실패한 결제 시도를 추적하는 데 도움이 됩니다.
- 동일한 구매에 대한 중복 PaymentIntent 생성을 방지하려면 멱등성 키를 제공하세요. 이 키는 일반적으로 애플리케이션에서 장바구니나 고객 세션과 연결된 ID를 기반으로 합니다.

<br/>

#### 클라이언트 측에 클라이언트 시크릿 전달하기
---

PaymentIntent에는 개별 PaymentIntent에 고유한 키인 client secret이 포함되어 있습니다. 애플리케이션의 클라이언트 측에서 Stripe.js는 결제를 완료하기 위해 함수(stripe.confirmCardPayment 또는 stripe.handleCardAction 등)를 호출할 때 클라이언트 시크릿을 파라미터로 사용합니다.

<br/>

**클라이언트 시크릿 조회**

PaymentIntent에는 클라이언트 측이 결제 프로세스를 안전하게 완료하는 데 사용하는 클라이언트 시크릿이 포함되어 있습니다. 클라이언트 시크릿을 클라이언트 측에 전달하는 여러 가지 방법이 있습니다.

<br/> 

*싱글 페이지 애플리케이션*

브라우저의 `fetch` 함수를 사용하여 서버의 엔드포인트에서 클라이언트 시크릿을 조회합니다. 이 방식은 클라이언트 측이 React 프레임워크로 만들어진 싱글 페이지 애플리케이션일 때 가장 좋습니다.

서버 코드는 다음과 같습니다.

```java
Gson gson = new Gson();

get("/secret", (request, response) -> {
  PaymentIntent intent = // ... PaymentIntent 조회 또는 생성
  
  Map<String, String> map = new HashMap();
  map.put("client_secret", intent.getClientSecret());
  
  return map;
}, gson::toJson);
```

클라이언트 코드는 다음과 같습니다.

```js
(async () => {
  const response = await fetch('/secret');
  const {client_secret: clientSecret} = await response.json();
  // clientSecret을 사용하여 폼 렌더링
})();
```

<br/>

#### 결제 후
---

클라이언트가 결제를 확인한 후, 서버에서 웹훅을 모니터링하여 결제가 성공적으로 완료되거나 실패하는 시점을 감지하는 것이 모범 사례입니다.

`PaymentIntent` 에 여러 결제 시도가 있었다면 하나 이상의 Charge 객체가 연결될 수 있습니다. 예를 들어, 재시도로 여러 `Charge` 가 생성될 수 있습니다. 각 과금에 대해 결과와 사용된 결제 수단의 세부 정보를 검사할 수 있습니다.

<br/>

#### 향후 결제를 위한 결제 수단 최적화
---

setip_future_usage 파라미터는 향후 다시 사용할 결제 수단을 저장합니다. 카드의 경우, SCA 같은 지역 규정 및 네트워크 규칙에 따라 승인률도 최적화합니다. 어떤 값을 사용할지 결정하려면, 향후 이 결제 수단을 어떻게 사용할지 고려하세요.


| 결제 수단 사용 방식                                                  | setup_future_usage 값 |
| ------------------------------------------------------------ | -------------------- |
| 온세션(고객이 결제 플로우에 적극적으로 참여하여 결제 수단을 인증할 수 있는 상태에서 발생하는 결제) 결제만 | `on_session`         |
| 오프세션(이전에 수집한 결제 정보를 사용하여 고객의 직접적인 참여 없이 발생하는 결제) 결제만         | `off_session`        |
| 온세션과 오프세션 결제 모두                                              | `off_session`        |

```js
const paymentIntent = awiat stripe.paymentIntents.create({
  amount: 1099,
  currency: 'usd',
  setup_future_usage: 'off_session',
});
```

<br/>

#### 동적 명세서 설명자
---

기본적으로 고객의 카드를 결제할 때마다 Stripe 계정의 명세서 설명자가 고객 명세서에 표시됩니다. 결제별로 다른 설명을 제공하려면 `statement_descriptor` 파라미터를 포함하세요.

```js
const paymentIntent = await stripe.paymentIntents.create({
  amount: 1099,
  currency: 'usd',
  payment_method_types: ['card'],
  statement_descriptor: 'Custom descriptor',
});
```

<br/>

#### 메타데이터에 정보 저장하기
---

Stripe는 결제 처리와 같이 가장 일반적인 요청에 메타데이터를 추가할 수 있도록 지원합니다. 메타데이터는 고객에게 표시되지 않으며, 결제 거부 또는 사기 방지 시스템에 의한 차단 여부에 반영되지 않습니다.

포함한 메타데이터는 대시보드에서 볼 수 있고, 일반 리포트에서도 사용할 수 있습니다. 예를 들어, 해당 주문의 PaymentIntent에 스토어읭 주문 ID를 첨부할 수 있습니다. 이렇게 하면 Stripe의 결제와 시스템의 주문을 쉽게 대조할 수 있습니다.

```js
const paymentIntent = await stripe.paymentIntents.create({
  amount: 1099,
  currency: 'usd',
  payment_method_types: ['card'],
  metadata: {
    order_id: '6735',
  },
});
```

<br/>

### 결제 상태 업데이트
---

결제 상태를 모니터링하고 확인하여 성공 및 실패한 결제에 대응하세요.

PaymentIntent는 고객이나 결제 수단이 취하는 작업에 따라 업데이트됩니다. 연동에서 PaymentIntent를 검사하여 결제 프로세스의 상태를 확인할 수 있으므로, 비즈니스 조치를 취하거나 추가 개입이 필요한 상태에 대응할 수 있습니다.

Stripe 대시보드를 사용하여 성공한 결제와 같은 결제 상태에 대해 이메일을 받도록 계정을 설정할 수도 있습니다.

<br/>

#### 클라이언트에서 PaymentIntent 상태 확인
---

confirmCardPayment 함수로 클라이언트에서 결제를 완료할 때, 반환된 PaymentIntent를 검사하여 현재 상태를 확인할 수 있습니다.

```js
(async () => {
  cosnt {paymentIntent, error} = await stripe.confirmCardPayment(clientSecret);
  if (error) {
    // 여기서 오류 처리
  } else if (paymentIntent && paymentIntent.status === 'succeeded') {
    // 여기서 성공한 결제 처리
  }
})
```

<br/>

#### confirmCardPayment를 사용하지 않고 확인
---

`retrivePaymentIntent` 함수를 사용하고 클라이언트 시크릿(Stripe가 PaymentIntent의 일부로 반환하는 고유 키)을 전달하여 독립적으로 조회합니다.

```js
(async () => {
  const {paymentIntent} = await stripe.retrievePaymentIntent(clientSecret);
  if (paymentIntent && paymentIntent.status === 'succeeded') {
    // 여기서 성공한 결제 처리
  } else {
    // 여기서 실패, 처리 중, 취소된 결제 및 API 오류 처리
  }
})();
```
- `succeded` : 고객이 결제 페이지에서 결제를 완료함
- `requires_action` : 고객이 결제를 완료하지 않음
- `requires_payment_method` : 고객의 결제가 결제 페이지에서 실패함

<br/>

### 비동기 캡쳐
---

비동기 캡처를 사용하여 PaymentIntent 확인 속도를 높이세요.

비동기 캡처는 캡처 작업을 백그라운드에서 수행하여 PaymentIntent 확인의 지연 시간을 줄입니다. 캡처 요청을 보낸 후 연동은 성공 응답을 받고, Stripe는 백엔드에서 결제 캡처를 완료합니다. 더 빠른 PaymentIntent 캡처를 사용하려면 PaymentIntent를 확인할 때 `capture_method=automatic_async` 파라미터를 설정하세요.

<br/>

## Setup Intents API
---

결제 수단 저장을 위한 Setup Intents API에 대해 알아보세요.

Setip Intent API를 사용하여 향후 결제를 위한 결제 수단을 설정하세요. 결제와 유사하지만 과금이 생성되지 않습니다.

목표는 결제 자격 증명을 저장하고 향후 결제에 최적화하는 것입니다. 즉, 결제 수단이 모든 시나리오에 맞게 올바르게 구성되어야 합니다. 예를 들어 카드를 설정할 때 고객 인증이나 고객 은행에서 카드 유효성을 확인하는 것이 필요할 수 있습니다. Stripe는 이 프로세스 전반에 걸쳐 `SetupIntent` 객체를 업데이트 합니다.

<br/>

## PaymentIntent와 SetupIntent의 작동 방식
---

결제 플로우 내에서 PaymentIntent와 SetupIntent가 어떻게 작동하는지 알아보세요/

Payment Intents API와 Setup Intents API의 주요 차이점은 다음 목적에 있습니다.

- Payment Intents API : 결제를 수금하고 고객에게 즉시 청구하는 데 사용됩니다. 과금을 생성하고 거래를 처리하며 자금을 수금합니다.
- Setup Intents API : 과금을 생성하지 않고 향후 사용을 위해 결제 수단 정보를 수집하고 저장하는 데 사용됩니다. 결제를 처리하지 않고 결제 자격 증명을 설정합니다.

<br/>

## Payment Methods API
---

다양한 글로벌 결제 수단을 지원하는 API에 대해 알아보세요.

Payment Methods API를 사용하면 단일 API를 통해 다양한 결제 수단을 수락할 수 있습니다.

PaymentMethod 객체는 결제를 생성하기 위한 결제 수단 정보를 포함합니다. Payment Methods API를 사용하면 PaymentMethod를 다음과 결합할 수 있습니다.

- PaymentIntent와 결합하여 결제 수락
- SetupIntent 및 Customer(비즈니스의 고객을 나타냅니다. 결제 수단을 재사용하고 여러 결제를 추적할 수 있게 해줍니다)와 결합하여 나중을 위해 결제 정보 저장

<br/>

## Payment Records API
---

Stripe 내외의 모든 결제 기록을 통합하여 관리하세요.

Payment Records API를 사용하여 모든 결제의 원장을 관리하세요. Stripe를 통해 결제를 처리하거나 서드파티 프로세서와 연동하는 경우, 이 API를 사용하여 결제 내역을 통합 관리할 수 있습니다.

Payment Records API로 다음을 수행할 수 있습니다.

- 서드파티 프로세서로 결제를 처리하고, 결과를 Stripe에 보고하여 Subscriptions나 Radar와 같은 제품의 전체 기능 활용
- 각 캡처를 추적할 수 있는 복잡한 결제 플로우 생성
- Stripe 지시 카드 거래를 포함한 서드파티 및 파트너 시작 결제 추적

<br/>

## Product와 Price의 작동 방식
---

Stripe의 Product와 Price가 비즈니스를 어떻게 모델링하는지 알아보세요.

Product와 Price는 많은 Stripe 연동의 핵심 리소스입니다. Product는 비즈니스가 제공하는 것(상품이든 서비스든)을 정의합니다. Price는 제품에 대해 얼마나 자주, 얼마를 청구할지를 정의합니다.

Stripe에서 Product와 Price를 생성하거나 API를 통해 Stripe로 가져올 수 있습니다. Product와 Price를 생성한 후에는 Checkout Sessions, Payment Links, Invoices, Quotes 또는 커스텀 연동과 함께 사용하여 Subscriptions를 생성할 수 있습니다.

<br/>

### Product (제품)
---

Product는 고객에게 제공하는 특정 상품이나 서비스를 설명합니다. 사용 사례는 다음과 같습니다.

- E-커머스 : 온라인에서 옷을 판매한다면, 예를 들어 셔츠의 각 사이즈와 색상 조함에 대해 별도의 제품을 생성할 수 있습니다.
- SaaS 플랫폼 : 기본과 프리밍머 가격 티어를 제공할 수 있으며, 기본과 프리미엄은 별도의 제품입니다.
- 기부 플랫폼 : 여러 다른 명분에 대한 기부를 받을 수 있으며, 각 명분은 다른 제품입니다.

<br/>

#### Product ID
---

각 제품에는 고유한 ID가 있습니다. 대부분의 Stripe 리소스와 달리, 제품의 ID를 직접 선택할 수 있습니다. Stripe를 사용하는 다른 시스템과 쉽게 연동할 수 있는 ID를 선택하는 것이 좋습니다. 예를 들어 물리적 상품을 판매한다면, 자체 시스템의 내부 ID를 사용할 수 있습니다.

<br/>

#### Product 이름
---

Stripe에서 제품을 생성할 때 이름을 제공해야 합니다. 선택적으로 설명이나 이미지와 같은 다른 속성을 추가할 수 있습니다. Stripe Tax를 사용하는 경우, 펫 미용, 전자책, SaaS와 같은 각 제품의 세금 코드도 정의할 수 있습니다. Stripe Tax는 세금 코드를 사용하여 구매 중 판매세를 자동으로 계산하고 징수합니다.

<br/>

### Price (가격)
---

Stripe에서 Price는 세금 동작, 볼륨 티어, 구독의 반복 간격과 같은 정보를 포함합니다. 각 구매마다 세 가격을 생성할 필요가 없습니다. 하나의 가격에 제품을 판매한다면 하나의 Price만 생성하면 됩니다. 이 가격을 제품의 기본 가격으로 설정할 수도 있습니다.

<br/>

#### 일회성 및 반복 결제
---

Price는 일회성 또는 반복일 수 있습니다. 구독은 반복 가격을 사용하여 "월 1회" 와 같은 간격으로 고객에게 청구합니다. 동일한 서비스를 여러 다른 간격으로 판매하는 경우, 동일한 제품에 대해 여러 반복 가격을 생성하는 것이 좋습니다.

<br/>

#### 변동 가격
---

두 가지 유형의 변동 가격을 사용할 수 있습니다.

- 인라인 가격 : 구독, 인보이스, Checkout Session 또는 Payment Link를 생성할 때 고객의 가격을 정의합니다.
    - 경우에 따라 미리 설정되지 않은 커스텀 가격을 사용하고 싶을 수 있습니다. 예를 들어 Stripe 외부에서 제품 카탈로그를 관리할 때 인라인 가격을 사용할 수 있습니다. 인라인 가격은 API를 통해서만 생성할 수 있습니다.
- 원하는 만큼 지불 : 팁이나 기부와 같이 고객이 지불할 가격을 입력합니다. 반복 결제는 이 유형의 변동 가격을 지원하지 않습니다. 단일 결제에서 고객이 지풀 금액을 결정하도록 하는 방법을 참조하세요.

<br/>

#### 다중 통화
---

단일 Price가 여러 통화를 지원할 수 있습니다. 이는 국제적으로 판매할 때 현지화된 가격을 관리하는 데 도움이 됩니다. 예를 들어 미국에서 10 USD, 유럽에서 9 EUR, 일본에서 1300 JPY에 제품을 판매하는 경우, 동일한 `Price` 객체가 세 가지 통화를 모두 포함할 수 있습니다. 각 구매는 연동에서 가격을 사용하는 방법에 따라 가격에 대해 지원되는 통화 중 하나를 사용합니다.

<br/>

#### 다중 가격
---

제품은 여러 가격을 사용하여 다른 가격 옵션을 정의할 수 있습니다. 가격은 제품 설명을 공유하며, 고객의 영수증과 인보이스에서 동일하게 보입니다. 가격만 다릅니다. 제품에 여러 가격이 연결될 수 있으므로, Checkout Sessions, Payment Links, 인보이스, 견적 또는 구독을 생성할 때 사용할 가격을 지정해야 합니다.

<br/>

#### 단가
---

대부분의 가격은 고정된 `unit_amount` 를 정의하지만, 다른 티어나 사용량 기반 모델로 작동하도록 가격을 구성할 수도 있습니다.

<br/>

#### 세금 동작
---

Stripe Tax를 사용하는 경우, 세금이 이미 금액에 포함되어 있는지 또는 추가해야 하는지를 결정하기 위해 가격의 `tax_behavior` 를 지정할 수 있습니다.

<br/>

### Product와 Price 작업하기
---

#### Product와 Price 생성 또는 가져오기
---

Product와 Price를 시작하는 가장 빠른 방법은 Stripe 대시보드를 통해 생성하는 것입니다.

스프레드시트나 다른 소프트웨어를 사용하여 관리하는 대규모 제품 카탈로그가 있는 경우, Products 및 Prices API를 사용하여 프로그래밍 방식으로 제품 카탈로그를 가져오는 것이 좋습니다.

각 거래마다 다른 금액을 청구해야 하는 경우(ex. 사용자가 선택한 기부 금액), 제품은 생성하지만 가격은 생성하지 않을 수 있습니다. 대신 Checkout Sessions, Payment Links 또는 Subscriptions `price_data` 파라미터를 사용하여 특정 가격을 설정할 수 있습니다.

이 방법은 특정 Checkout Session, Payment Link 또는 Subscription에 관련된 `Price` 와 `Product` 객체를 생성합니다. `Price` 객체는 임시적이며 대시보드에서 보이지 않지만, 연결된 `Price` 객체는 항상 임시적인 것은 아닙니다. 예를 들어 `price_data` 로 생성된 `Price` 객체는 대시보드의 제품 검색이나 목록에 나타나지 않습니다. 대시보드에서 특정 URL을 구성하여 이러한 객체를 직접 볼 수 있지만, 메인 카탈로그에는 표시되지 않습니다.

<br/>

#### Product와 Price 사용하기
---

Stripe Checkout
: Checkout Session을 생성할 때 각 라인 항목에 대해 price `id` 를 지정합니다. Checkout Session은 가격을 사용하여 주문 총액을 계산합니다. 또한 가격과 연결된 제품을 조회한 다음, 제품의 이름과 이미지를 사용하여 결제 페이지를 렌더링합니다.

Payment Links
: 대시보드를 사용하여 가격을 기반으로 Payment Link를 생성합니다. Payment Link는 가격을 사용하여 주문 총액을 계산합니다. 또한 가격과 연결된 제품을 조회합니다. 제품의 이름과 이미지는 결제 페이지를 렌더링하는 데 사용됩니다.

Subscriptions
: Subscription을 생성할 때 price `id` 를 지정합니다. 가격은 반복이어야 합니다. 가격의 반복 간격은 구독의 청구 기간을 정의하고, 가격의 금액은 각 청구 기간에 구독이 청구하는 금액을 정의합니다. 동일한 반복 간격을 가진 경우 동일한 구독에 둘 이상의 가격을 추가할 수 있습니다.

Quotes
: Quote를 생성할 때 price `id` 를 지정합니다. Quote는 일회성 인보이스 또는 구독 비용을 모델링합니다. 고객이 견적을 수락하면 Stripe가 모든 관련 인보이스와 구독을 자동으로 생성합니다.

Invoices
: 인보이스를 생성할 때 각 라인 항목에 대해 price `id` 를 지정합니다. 인보이스는 가격을 사용하여 인보이스 총액을 계산합니다. 또한 가격과 연결된 제품을 조회하고, 인보이스를 웹 페이지나 PDF로 렌더링할 때 `product.name` 을 사용합니다.

|기능|Checkout|Payment Links|Subscriptions|Quotes|Invoices|
|---|---|---|---|---|---|
|**주요 목적**|웹사이트/앱 결제|링크 공유 결제|반복 결제|가격 견적|청구서 발송|
|**코드 필요**|✅ API 필요|❌ 불필요|❌/✅ 선택 가능|❌/✅ 선택 가능|❌ 불필요|
|**일회성 결제**|✅|✅|△ (추가 가능)*|✅|✅|
|**반복 결제**|✅|✅|✅|✅|✅|
|**특정 고객 대상**|△ (선택적)|❌|✅|✅|✅|
|**결제 조건 설정** (Net 30 등)|❌|❌|✅|✅|✅|
|**PDF 생성**|✅ (인보이스 연동 시)|✅ (인보이스 연동 시)|✅ (인보이스 자동 생성)|✅|✅|
|**커스터마이징**|중간|낮음|높음|높음|높음|
|**결제 수단**|40+|20+|40+|40+|40+|
|**Link (빠른 결제)**|✅|✅|✅|-|✅|
|**무료 체험**|✅|✅|✅|△ (구독 전환 후 설정)|❌|
|**세금 자동 계산**|✅|✅|✅|✅|✅|
|**적합 사용자**|개발자, E-commerce|소규모 비즈니스, 크리에이터|SaaS, 멤버십|B2B 영업팀|프리랜서, B2B|
|**주요 채널**|웹사이트/앱|SNS, 이메일, 문자, QR코드|웹사이트/앱|이메일|이메일|

<br/>

### 기존 Product와 Price 관리
---

대시보드나 API를 통해 제품 정보를 업데이트할 수 있습니다. 예를 들어 제품의 설명을 변경하거나 Checkout 페이지에서 사용할 새 제품 이미지를 추가할 수 있습니다.

더 이상 제품을 판매하지 않는 경우, 대시보드에서 Archive 버튼을 클릭하거나 API에서 `active` 를 `false` 로 설정하여 제품과 가격을 모두 보관할 수 있습니다. 과거 거래 기록을 유지하기 위해 보관된 제품과 가격 정보를 무기한으로 저장합니다.

일반적으로 제품이나 가격을 삭제할 수 없으며, 보관만 할 수 있습니다. 특정 경우에는 대시보드를 사용하여 한 번도 사용되지 않은 가격을 삭제하거나, 가격이 설정되지 않은 제품을 삭제할 수 있습니다.

제품 가격을 변경하려면 새 금액에 대한 새 가격을 생성한 다음, `active` 를 false로 설정하여 기존 가격을 보관합니다. 기존 가격의 `unit_amount` 를 변경하는 대신 새 가격을 생성해야 합니다. 이렇게 하면 기존 가격을 과거 거래의 불변 기록으로 유지할 수 있습니다.

제품의 기본 가격을 설정하여 고객에게 가장 일반적으로 표시할 가격을 지정할 수 있습니다. 나중에 제품 가격을 인상하는 경우와 같이 기본 가격을 다른 가격으로 변경할 수 있습니다.

<br/>

### Product와 Price 호환성 이해하기
---

제품과 가격의 모든 기능이 모든 Stripe API와 호환되는 것은 아닙니다. 호환성 정보는 다음 표를 참조하세요.

| 기능                    | Checkout | Payment Links | Quotes | Subscriptions | Invoices |
| --------------------- | -------- | ------------- | ------ | ------------- | -------- |
| Product 이미지           | ✓ 지원     | ✓ 지원          | (무시*)  | (무시*)         | (무시*)    |
| Product 설명            | ✓ 지원     | ✓ 지원          | ✓ 지원   | (무시*)         | (무시*)    |
| Product 세금 코드         | ✓ 지원     | ✓ 지원          | ✓ 지원   | ✓ 지원          | ✓ 지원     |
| Product 명세서 설명자       | ✓ 지원     | ✓ 지원          | (무시*)  | ✓ 지원          | ✓ 지원     |
| 반복 가격                 | ✓ 지원     | ✓ 지원          | ✓ 지원   | ✓ 지원          | ✓ 지원     |
| 다중 통화 가격              | ✓ 지원     | ✓ 지원          | (무시*)  | ✓ 지원          | (무시*)    |
| 티어별 가격                | ✓ 지원     | (불허*)         | ✓ 지원   | ✓ 지원          | ✓ 지원     |
| 소수점 금액 (예: 단위당 0.5센트) | ✓ 지원     | (불허*)         | ✓ 지원   | ✓ 지원          | ✓ 지원     |
| 사용량 기반 가격             | ✓ 지원     | (불허*)         | ✓ 지원   | ✓ 지원          | ✓ 지원     |
| 고객 가격 선택              | ✓ 지원     | ✓ 지원          | (불허*)  | (불허*)         | (불허*)    |

- 불허 : 해당 제품이나 가격을 이 Stripe API와 함께 사용할 수 없음을 나타냅니다.
- 무시 : Stripe API에서 효과가 없지만, 제품이나 가격을 평소대로 사용할 수 있음을 나타냅니다.

<br/>

### 제한사항 이해하기
---

Stripe 계정에서 생성할 수 있는 고객, 쿠폰, 제품, 가격 또는 대부분의 다른 객체 수에는 제한이 없습니다.

Subscriptions와 함께 반복 가격을 사용할 때
- Subscription의 모든 가격은 동일한 `recurring.internal` 과 `recurring.interval_count` 를 가져야 합니다.
- 가격의 최대 기간은 3년입니다.

<br/>

## Product와 Price 관리
---

제품과 가격을 관리하는 방법을 알아보세요.

대시보드나 API를 통해 제품과 가격을 생성하고 업데이트할 수 있습니다.

변동 가격 생성과 같은 일부 고급 사용 사례는 API를 사용해야 합니다. 제품과 가격이 많거나 Elements를 사용한 커스텀 연동을 구축하는 경우 API를 사용하세요.

- 대시보드 : 코드 작성을 피하고 싶거나 제품과 가격이 몇 개뿐인 경우 대시보드를 사용하세요. 샌드박스에서 가격 모델(가격 모델은 판매하는 제품/서비스, 가격, 결제 통화, 청구 기간(구독용)으로 구성됩니다)을 설정하고 제품 상세 페이지에서 Copy to live mode 버튼을 클릭하세요.
- API/CLI : API나 Stripe CLI를 사용하여 제품과 가격을 생성하고 관리하세요. 프로덕션 구현에 직접 사용하는 방법입니다.

<br/>

### Product 생성
---

#### 대시보드에서 제품 및 가격 생성
---

**제품 생성**

1. More > Product catalog로 이동
2. +Add product 클릭
3. 제품 이름 입력
4. (선택) 설명 추가 - 결제, 고객 포털, 견적에 표시됨
5. (선택) 제품 이미지 추가 - JPEG, PNG, WEBP (2MB 미만)
6. (선택) Stripe Tax 사용 시 세금 코드 선택
7. (선택) 명세서 설명자 입력 - 고객 은행 명세서에 표시됨
8. (선택) 단위 라벨 입력 - 예: "seat"을 입력하면 "per seat"로 표시

<br/>

**제품의 가격 생성**

대시보드에서 제품을 저장하려면 최소 하나의 가격을 추가해야 합니다.

제품 편집기는 기본적으로 정액 가격 모델을 표시합니다. 고급 가격 옵션을 사용하면 여러 가격을 생성하거나 다른 가격 모델을 사용할 수 있습니다.

가격 모델을 선택합니다.
1. 정액 (Flat-rate) : 단위당 동일한 가격을 청구합니다. 이 옵션을 사용하는 경우 일회성 또는 반복을 선택합니다. ex) 월 $10
2. 사용자당 (Per-seat) : 각 가격 단위가 한 명의 사용자를 나타냅니다. 예를 들어 기업이 직원용 소프트웨어를 구매하고 각 직원이 소프트웨어에 접근하려면 라이선스가 필요합니다.
3. 티어별 (Tiered) : 단가가 수량(볼륨 기반 가격)이나 사용량(단계별 가격)에 따라 변경됩니다.
4. 사용량 기반 (Usage-based) : 청구 기간 동안 서비스 사용량에 따라 고객에게 청구합니다. ex) API 호출 수 기준
5. 패키지 (Package) : 패키지 또는 단위 그룹으로 청구합니다. ex) 5개당 $25
6. 단계별 (Graduated) : 주문의 일부 단위에 다른 가격이 적용될 수 있는 가격 티어를 사용합니다. ex) 100개는 단위당 $10, 다음 50개는 단위당 $5
7. 볼륨 (Volume) : 판매된 총 단위 수에 따라 각 단위에 동일한 가격을 청구합니다. ex) 50개 구매 -> $10/개, 100개 구매 -> $7/개
8. 고객 선택 (Customer choose price) : 결제자가 제품, 서비스 또는 명분에 대해 지불할 금액을 결정하도록 합니다. 고객 선택 가격은 Checkout 및 Payment Links에서만 호환됩니다. ex) 기부, 팁

<br/>

#### API로 제품 및 가격 생성
---

단일 제품과 가격 생성하기

```java
// 비밀 키를 설정
Stripe.apiKey = "***";

ProductCreateParams params =
  ProductCreateParams
    .builder()
    .setName("Basic Dashboard")
    .setDefaultPriceData(
      ProductCreateParams.DefaultPriceData
        .builder()
        .setUnitAmount(1000L) // $10.00 (센트 단위)
        .setRecurring(
          ProuctCreateParams.DefaultPriceData.Recurring
            .builder()
            .setInterval(
              ProductCreateParams.DefaultPriceData.Recurring.Interval.MONTH
            )
            .build()
        )
        .build()
    )
    .addExpand("default_price")
    .build();
    
Product product = Product.create(params);
```

<br/>

설정 수수료 추가하기

```java
// 비밀 키를 설정
Stripe.apiKey = "***";

ProductCreateParams params =
  ProductCreateParams.builder()
    .setName("Starter Setup")
    .setDefaultPriceData(
      ProductCreateParams.DefaultPriceData.builder()
        .setUnitAmount(2000L)
        .setCurrency("usd")
        // recurring 없음 = 일회성 가격
        .build()
    )
    .addExpand("default_price")
    .build();
    
Product product = Product.create(params);
```

<br/>

### Product 편집
---

#### 대시보드에서 편집
---

1. More > Product catalog로 이동합니다.
2. 수정할 제품을 찾아 더보기 메뉴를 클릭한 다음 Edit product를 클릭합니다.
3. 제품을 변경합니다.
4. Save product를 클릭합니다.

<br/>

#### API로 편집
---

```java
Stripe.apiKey = "***";

// 기존 제품 조회
Product resource = Product.retrieve("prod_xxx");

// 업데이트할 파라미터 설정
ProductUpdateParams params =
  ProductUpdateParams.builder()
    .setName("Updated Product")
    .build();
    
// 제품 업데이트 실행
Product product = resource.update(params);
```

<br/>

### Product 보관
---

제품을 비활성화하여 새 인보이스나 구독에 추가할 수 없도록 하려면 보관할 수 있습니다. 제품을 보관해도 해당 제품을 사용하는 기존 구독은 취소될 때까지 활성 상태로 유지되며, 해당 제품을 사용하는 기존 결제 링크는 비활성화됩니다. 연결된 가격이 있는 제품은 삭제할 수 없지만 보관할 수 있습니다.

<br/>

#### 대시보드에서 보관
---

제품을 보관하려면

1. More > Product catalog로 이동합니다.
2. 수정할 제품을 찾아 더보기 메뉴를 클릭한 다음 Archive product를 클릭합니다.

제품 보관을 해제하려면

1. Product catalog > Overview 페이지의 Archived 탭으로 이동합니다.
2. 수정할 제품을 찾아 더보기 메뉴를 클릭한 다음 Unarchive product를 클릭합니다.

<br/>

#### API로 보관
---

비활성 상태로 새 제품 생성

```java
ProductCreateParams params =
  ProductCreateParams.builder()
    .setActive(false)
    .setName("My Product")
    .build();
    
Product product = Product.create(params);
```

<br/>

제품 보관을 해제

```java
// 기존 제품 조회
Product resource = Product.retrieve("prod_xxx");

// 활성 상태로 변경
ProductUpdateParams params =
  ProductUpdateParams.builder()
    .setActive(true)
    .build();
    
Product product = resource.update(params);
```

<br/>

### Product 삭제
---

연결된 가격이 없는 제품만 삭제할 수 있습니다.

<br/>

#### 대시보드에서 삭제
---

제품에 연결된 가격이 있는 경우, 제품을 삭제하기 전에 먼저 


<br/>

## Billing API
---

**정기 결제 (Billing)**

구독을 생성하고 관리하고, 사용량을 추적하고, 인보이스를 발행하세요.

Stripe Billing을 사용하면 구독과 인보이스를 손쉽게 관리할 수 있습니다. 정기 결제를 자동화하고, 원하는 대로 가격 플랜을 설계하며, 체험 기간이나 갱신 같은 청구 주기를 유연하게 처리할 수 있습니다.

<br/>

**주요 기능**

- **가격 모델** : 정액제, 사용자 수 기반, 사용량 기반, 구간별, 변동형, 다중 통화 등 다양한 가격 모델을 만들 수 있습니다.
- **사용량 기반 과금** : 일간, 주간, 월간, 분기별, 연간 주기로 고객의 실제 사용량에 따라 과금할 수 있습니다.
- **자동화된 맞춤 인보이스** : 인보이스를 자동 생성하고 발송하며, 로고, 맞춤 필드, 템플릿, 항목 그룹화로 브랜딩할 수 있습니다.
- **스마트 재시도** : 결제 재시도 일정을 최적화하여 수금률을 높일 수 있습니다.
- **고객 포털** : 고객이 직접 구독을 관리할 수 있는 셀프서비스 포털을 제공합니다.

구독(Subscription)은 인보이스(Invoice)와 결제 인텐트(PaymentIntent)를 자동으로 생성합니다. 구독은 다음과 같은 요소로 구성됩니다.

- **상품(Product)** : 비즈니스에서 판매하는 재화나 서비스를 나타냅니다.
- **가격(Price)** : 청구 금액과 주기를 정의합니다. 상품의 가격, 사용 통화, 구독의 경우 청구 간격을 포함합니다.
- **고객(Customer)** : 정기 결제에 사용되는 결제 수단(PaymentMethod)을 저장합니다.

![stripe1](/assets/img/stripe1.png)

```
고객 (Customer)
    └── 결제 수단 (PaymentMethod)
    └── 구독 (Subscription)
            └── 상품 (Product)
            │       └── 기능 (Feature) [ProductFeature로 연결]
            └── 가격 (Price)
            └── 인보이스 (Invoice)
                    └── 결제 인텐트 (PaymentIntent)
    └── 권한 (Entitlement) [구독 시 자동 생성]
```

<br/>

### API 객체 정의
---


| 리소스                        | 정의                                                                                                                                                                     |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Customer (고객)**          | 구독을 구매하는 고객을 나타냅니다. <br>구독에 연결된 Customer 객체를 사용하여 정기 결제를 처리하고 추적하며, <br>고객이 구독 중인 상품을 관리합니다.                                                                           |
| **Entitlement (권한)**       | 고객이 구독한 서비스 상품에 포함된 기능에 대한 접근 권한을 나타냅니다.<br>고객이 상품을 정기 구매하는 구독을 생성하면, 해당 상품에 연결된 <br>각 기능에 대한 활성  권한이 자동으로 생성됩니다.<br>고객이 서비스에 접근할 때, 활성 권한을 통해 구독에 포함된 기능을 <br>활성화합니다. |
| **Feature (기능)**           | 고객이 서비스 상품을 구독할 때 접근할 수 있는 기능이나 능력을 나타냅니다.<br>ProductFeature를 생성하여 상품에 기능을 포함시킬 수 있습니다.                                                                                |
| **Invoice (명세서)**          | 고객이 지불해야 할 금액 명세서로, 초안부터 결제 완료 또는 확정까지 <br>결제 상태를 추적합니다.<br>구독이 자동으로 인보이스를 생성합니다.                                                                                      |
| **PaymentIntent (결제 의도)**  | 동적인 결제 플로우를 구축하는 방법입니다.<br>고객 결제 흐름의 생명주기를 추적하고, 규제 요건, Redar 사기 방지 규칙, <br>리디렉션 기반 결제 수단 등에 따라 추가 인증 단계를 트리거합니다.<br>인보이가 자동으로 PaymentIntent를 생성합니다.                   |
| **PaymentMethod (결제 수단)**  | 고객이 상품 결제에 사용하는 결제 수단입니다.<br>예를 들어 Customer 객체에 신용카드를 저장하고 해당 고객의 <br>정기 결제에 사용할 수 있습니다.<br>주로 Payment Instents나 Setup Instents API와 함께 사용합니다.                         |
| **Price (가격)**             | 상품의 단가, 통화, 청구 주기를 정의합니다.                                                                                                                                              |
| **Product (상품)**           | 비즈니스에서 판매하는 재화나 서비스입니다.<br>서비스 상품은 하나 이상의 기능을 포함할 수 있습니다.                                                                                                              |
| **ProductFeature (상품 기능)** | 단일 상품에 단일 기능이 포함되어 있음을 나타냅니다.<br>각 상품은 포함된 기능에 대해 ProductFeature와 연결되고,<br>각 기능은 해당 기능을 포함하는 각 상품에 대해 ProductFeature와 연결됩니다.                                           |
| **Subscription (구독)**      | 고객의 예정된 정기 상품 구매를 나타냅니다.<br>구독을 사용하여 결제를 수금하고 상품을 반복 배송하거나<br>지속적인 접근 권한을 제공합니다.                                                                                       |

<br/>

### 상품, 기능, 권한이 함께 작동하는 예시
---

두 가지 등급의 정기 서비스를 설정한다고 가정해 보세요. 기본 기능을 제공하는 스탠다드 상품과 확장 기능이 추가된 어드밴스드 상품입니다.

<br/>

**설정 과정**

1. 두 개의 기능을 생성합니다: `basic_features` 와 `extended_features`
2. 두 개의 상품을 생성합니다: `standard_product` 와 `advanced_product`
3. 스탠다드 상품에는 `basic_feature` 를 `standard_product` 에 연결하는 ProductFeature 하나를 생성합니다.
4. 어드밴스드 상품에는 두 개의 ProductFeature를 생성합니다.
    1. `basic_features` 를 `advanced_product` 에 연결
    2. `extended_feature` 를 `advanced_product` 에 연결

<br/>

**고객 구독 시나리오**

첫 번째 고객 `first_customer` 가 스탠다드 상품을 구독합니다. 구독을 생성하면 Stripe가 자동으로 `first_customer` 와 `basic_features` 를 연결하는 권한(Entitlement)을 생성합니다.

두 번째 고객 `second_customer` 가 어드밴스드 상품을 구독합니다. 구독을 생성하면 Stripe가 자동으로 두 개의 권한을 생성합니다.
1. `second_customer` 와 `basic_features` 연결
2. `second_customer` 와 `extended_features` 연결

<br/>

**권한 확인 방법**

고객에게 어떤 기능을 제공할지 결정하려면 활성 권한을 조회하거나 Active Entitlement Summary 이벤트를 수신하면 됩니다. 구독, 상품, 기능을 일일이 조회할 필요가 없습니다.

<br/>

## 구독 (Subscriptions)
---

Stripe Billing API로 구독을 생성하고 관리하세요.

<br/>

### 구독의 작동 원리
---

정기 결제와 구독 생명주기를 관리하는 방법을 알아보세요.

Stripe는 구독 결제를 관리하는 데 유용한 다양한 기능을 제공합니다.
- 다양한 가격 모델 지원
- 구독 할인 관리
- 체험 기간 관리
- 구독 변경 시 비례 정산
- 고객 셀프 서비스 관리
- 결제 수금을 위한 인보이스 발행
- 자동 수익 복구
- 리포팅 및 분석

<br/>

### 구독 생명주기
---

구독을 생성하고 관리하느 과정에는 Customers, Invoices, PaymentIntents 등 핵심 Stripe API 리소스가 사용됩니다.

권장하는 구독 플로우는 다음과 같습니다.

<br/>

#### 구독 생성
---

대시보드나 API를 통해 새 구독을 생성합니다. 구독을 생성할 때마다 이벤트가 발생합니다. 웹훅 엔드포인트로 이 이벤트를 수신하고, 연동 시스템이 이를 적절히 처리하는지 확인하세요.

선택적으로 결제 없이 시작하는 체험판을 생성할 수 있습니다. 이 경우 상태는 `trialing` 이 됩니다. 체험 기간이 끝나면 구독이 `active` 로 전환되고 Stripe가 고객에게 과금합니다.

<br/>

**결제 동작 설정**

첫 결제 실패나 3DS(3D Secure, 카드 보안) 같은 복잡한 결제 플로우를 처리하려면 구독의 `payment_behavior` 를 default_incomplete로 설정하는 것을 권장합니다. 이렇게 하면 결제가 필요할 때 구독이 `incomplete` 상태로 생성되어, 이후에 결제 정보를 수집하고 확인하여 구독을 활성화할 수 있습니다.

- `payment_behavior` 설정값에 따른 동작
    - `allow_incomplete` : Stripe가 인보이스에 대해 즉시 결제를 시도합니다. 결제가 실패하면 구독 상태가 `incomplete` 로 변경됩니다.
    - `error_if_incomplete` : 결제가 실패하면 구독 생성 자체가 실패합니다.

대시보드에서 생성하는 구독은 결제 수단에 따라 적절한 결제 동작이 기본 설정됩니다.

<br/>

#### 인보이스 처리
---

구독을 생성하면 Stripe가 자동으로 `open` 상태의 인보이스를 생성합니다. 고객은 약 23시간 내에 결제를 완료해야 합니다. 이 시간 동안 구독 상태는 `incomplete` 이고 인보이스는 `open` 을 유지합니다.

23시간 제한이 있는 이유는 보통 고객이 첫 결제를 온세션(고객이 결제 플로우에 직접 참여하여 결제 수단을 인증할 수 있는 상태) 상태에서 진행하기 때문입니다. 23시간이 지난 후 고객이 다시 방문하면 새 구독을 생성해야 합니다.

<br/>

#### 고객 결제
---

고객이 인보이스를 결제하면 구독이 `active` 로, 인보이스가 `paid` 로 업데이트됩니다. 결제하지 않으면 구독은 `incomplete_expired` 로, 인보이스는 `void` 가 됩니다.

- 인보이스 결제 여부를 확인하는 방법
    - 웹훅 엔드포인트나 다른 이벤트 수신 방법을 설정하고 `invocie.paid` 이벤트를 수신합니다.
    - 구독 객체에서 `subscription.status=active` 인지 직접 확인합니다. 자동 과금이든 고객의 수동 결제든 인보이스가 결제되면 상태가 `active` 가 됩니다.

<br/>

#### 상품 접근 권한 부여
---

고객을 위해 구독을 생성하면, 해당 상품에 연결된 각 기능에 대해 활성 권한(Entitlement)이 생성됩니다. 고객이 서비스에 접근할 때, 활성 권한을 확인하여 구독에 포함된 기능에 대한 접근을 허용하세요.

또는 웹훅 이벤트로 활성 구독을 추적하고 해당 활동을 기반으로 상품을 제공할 수도 있습니다.

<br/>

#### 구독 업데이트
---

기존 구독을 취소하고 다시 만들 필요 없이 필요에 따라 수정할 수 있습니다. 주요 변경 사항으로는 구독 가격 업그레이드 또는 다운그레이드, 활성 구독의 결제 수금 일시 중지 등이 있습니다.

웹훅 엔드포인트로 구독 이벤트를 수신하여 구독 변경을 감지할 수 있습니다. 예를 들어 결제 실패 시 고객에게 이메일을 보내거나, 구독 취소 시 접근 권한을 회수할 수 있습니다.

Stripe Checkout 연동의 경우, 세션의 구독이 `incomplete` 상태이면 구독이나 인보이스를 업데이트할 수 없습니다. 세션 완료 후 업데이트하려면 `checkout.session.completed` 이벤트를 수신하세요. 세션의 구독을 취소하거나, 인보이스를 무효화하거나, 수금 불가로 표시하려면 세션을 만료시킬 수도 있습니다.

<br/>

#### 첫 번째 인보이스 업데이트

collection_method가 `send_invoice` 인 경우 구독의 첫 인보이스를 업데이트할 수 있습니다. 인보이스 생성 후 1시간 이내에 금액, 항목, 설명, 메타데이터 등을 수정할 수 있습니다. 인보이스가 확정되어 고객에게 결제 요청이 전송된 후에는 수정할 수 없습니다.

`collection_method` 가 `charge_automatically` 인 경우 첫 인보이스는 즉시 확정되어 과금됩니다. 첫 인보이스는 확정 전에 수정할 수 없지만, 이후 갱신 인보이스는 수정할 수 있습니다.

구독 일정(subscription schedules)의 첫 인보이스는 수금 방식에 관계없이 항상 열린 상태이며 수정할 수 없습니다.

<br/>

#### 미결제 구독 처리
---

미결제 인보이스가 있는 구독의 경우, 미결제 인보이스는 열린 상태로 유지되지만 추가 결제 시도는 중단됩니다. 구독은 매 청구 주기마다 인보이스를 계속 생성하며, 이 인보이스들은 `draft` 상태로 유지됩니다. 구독을 다시 활성화하려면:

1. 필요시 새 결제 정보를 수집합니다.
2. draft 인보이스에서 auto_advance 를 `true` 로 설정하여 자동 수금을 활성화합니다.
3. open 인보이스를 확정하고 결제합니다. 만기일 전에 가장 최근의 무효화되지 않은 인보이스를 결제하면 구독 상태가 `active` 로 업데이트됩니다.

수금 불가로 표시된 인보이스는 기본 구독을 `active` 상태로 유지합니다. Stripe는 구독 상태를 결정할 때 무효화된 인보이스를 무시하고 가장 최근의 무효화되지 않은 인보이스를 사용합니다.

미결제 구독의 상태는 대시보드의 결제 실패 설정에 따라 결정됩니다.

<br/>

#### 구독 취소
---

구독은 언제든지 취소할 수 있으며, 청구 주기 말이나 지정된 청구 주기 후에 취소되도록 설정할 수도 있습니다.

기본적으로 구독을 취소하면 새 인보이스 생성이 비활성화되고 해당 구독의 모든 미결제 인보이스에 대한 자동 수금이 중단됩니다(`auto_advance=false`). 구독이 삭제되어 더 이상 구독이나 메타데이터를 업데이트할 수 없습니다. 고객이 재구독하려면 새 결제 정보를 수집하고 새 구독을 생성해야 합니다.

<br/>

### 구독 상태
---


| **상태**               | **설명**                                                                                                                                                                                                                                |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `trialing`           | 현재 체험 기간중.<br>고객에게 상품을 안전하게 제공할 수 있습니다.<br>고객이 첫 결제를 하면 자동으로 `active` 로 전환됩니다.                                                                                                                                                        |
| `active`             | 구독이 정상 상태입니다.<br>`past_due` 구독의 경우 최근 인보이스를 결제하거나 수금 불가로 표시하면 <br>`active` 로 전환됩니다.<br>참고: `active` 가 해당 구독의 모든 미결제 인보이스가 결제되었음을 의미하지는 않습니다.                                                                                          |
| `incomplete`         | 고객이 23시간 내에 결제를 완료해야 구독이 활성화됩니다.<br>또는 결제에 고객 인증 같은 추가 조치가 필요합니다.<br>PaymentIntent 상태가 `processing` 인 보류 중인 결제가 있을 때도 <br>`incomplete` 가 될 수 있습니다.                                                                                    |
| `incomplete_expired` | 구독의 최초 결제가 실패했고 고객이 생성 후 23시간 내에 결제를 완료하지 못했습니다.<br>이 구독은 고객에게 과금하지 않습니다.<br>구독 활성화에 실패한 고객을 추적하기 위한 상태입니다.                                                                                                                           |
| `past_due`           | 가장 최근 확정된 인보이스의 결제가 실패했거나 시도되지 않았습니다.<br>구독은 계속 결제 인보이스를 생성합니다.<br>대시보드의 구독 설정에 따라 다음 상태가 결정됩니다.<br>모든 스마트 재시도 후에도 인보이스가 미결제면 구독을 `canceled` , `unpaid` 로 이동하거나<br>`past_due` 로 유지하도록 설정할 수 있습니다.<br>고객이 최근 인보이스를 결제하면 구독이 다시 활성화됩니다. |
| `canceled`           | 구독이 취소되었습니다.<br>취소 시 모든 미결제 인보이스의 자동 수금이 비활성화됩니다(`auto_advance=false`).<br>이는 업데이트할 수 없는 최종 상태입니다.                                                                                                                                    |
| `unpaid`             | 최근 인보이스가 결제되지 않았지만 구독은 유지됩니다.<br>최근 인보이스는 열린 상태로 유지되고 인보이스 생성은 계속되지만<br>결제 시도는 하지 않습니다.<br>`past_due` 동안 이미 결제 시도와 재시도가 이루어졌으므로 `unpaid` 상태일 때는<br>상품 접근을 차단하세요.<br>만기일 전에 최근 인보이스를 결제하면 구독이 `active` 로 전환됩니다.                        |
| `paused`             | 구독이 기본 결제 수단 없이 체험 기간을 종료했고<br>`trial_settings.end_behavior.missing_payument_method` 가 `pause` 로 설정되어 있습니다.<br>더 이상 인보이스가 생성되지 않습니다. 고객에게 기본 결제 수단을 연결한 후<br>구독을 재개할 수 있습니다.                                                          |

<br/>

### 결제 상태
---

PaymentIntent는 모든 결제의 생명주기를 추적합니다. 구독에 결제가 필요할 때마다 Stripe가 인보이스와 PaymentIntent를 생성합니다. PaymentIntent ID는 인보이스에 첨부되며 Invoice와 Subscription 객체에서 접근할 수 있습니다.

PaymentIntent의 상태는 인보이스와 구독의 상태에 영향을 미칩니다. 결제 결과에 따른 상태 매핑은 다음과 같습니다.


| **결제 결과** | **PaymentIntent 상태**      | **인보이스 상태** | **구독 상태**    |
| --------- | ------------------------- | ----------- | ------------ |
| 성공        | `succeeded`               | `paid`      | `active`     |
| 카드 오류로 실패 | `requires_payment_method` | `open`      | `incomplete` |
| 인증 필요로 실패 | `requires_action`         | `open`      | `incomplete` |
- 참고: ACH Direct Debit 같은 비동기 결제 수단은 즉시 결제 수단과 다르게 구독 상태 전환을 처리합니다. 비동기 수단을 사용하면 구독이 다른 결제 유형과 관련된 `incomplete` 상태를 거치지 않고 생성 후 바로 `active` 상태로 전환됩니다. 비동기 결제가 나중에 실패하면 관련 인보이스가 무효화되지만 구독은 `active` 상태로 유지합니다. 이는 실패한 결제가 종종 `incomplete` 나 `past_due` 상태로 이어지는 즉시 결제 수단과 대조됩니다. 이 차이를 인식하고 구독 상태, 접근 제어, 결제 재시도 메커니즘을 관리하는 적절한 로직을 구현하세요.

<br/>

#### 결제 성공
---

고객의 결제가 성공하면:
- PaymentIntent의 상태가 `succeeded` 로 변경됩니다.
- 구독 상태가 `active` 가 됩니다.
- 인보이스 상태가 `paid` 가 됩니다.
- Stripe가 설정된 웹훅 엔드포인트로 `invoice.paid` 이벤트를 전송합니다.

처리 기간이 긴 결제 수단의 경우 구독이 즉시 활성화됩니다. 이 경우 결제가 성공할 때까지 `active` 구독에 대한 PaymentIntent 상태가 `processing` 일 수 있습니다.

<br/>

#### 결제 수단 필요
---

거절(decline) 같은 카드 오류로 결제가 실패하면:
- PaymentIntent 상태가 `requires_payment_method` 입니다.
- 구독 상태가 `incomplete` 입니다.
- 인보이스 상태가 `open` 입니다.

이 상황을 처리하려면:
- 고객에게 알립니다.
- 새 결제 정보를 수집하고 PaymentIntent를 확인합니다.
- 구독의 기본 결제 수단을 업데이트합니다.
- Stripe가 스마트 재시도를 사용하거나 사용자 정의 재시도 규칙에 따라 결제를 다시 시도합니다.
- `invoice.payment_failed` 이벤트를 사용하여 구독 결제 실패 이벤트와 재시도 업데이트를 모니터링합니다.

<br/>

#### 추가 조치 필요
---

일부 결제 수단은 결제 프로세스를 완료하기 위해 3DS(3D Secure) 고객 인증이 필요합니다. 결제 수단에 인증이 필요한지는 Radar 규칙과 카드 발급 은행에 따라 다릅니다.

고객이 결제를 인증해야 해서 결제가 실패하면.
- PaymentIntent 상태가 `requires_action` 입니다.
- 구독 상태가 `incomplete` 입니다.
- 인보이스 상태가 `open` 입니다.

이 상황을 처리하려면:
- 웹훅 엔드포인트로 `invoice.payment_action_required` 이벤트 알림을 모니터링합니다. 인증이 필요함을 나타냅니다.
- 고객에게 인증이 필요하다고 알립니다.
- PaymentIntent의 클라이언트 시크릿을 가져와서 `stripe.ConfirmCardPayment` 호출에 전달합니다. 인증 모달이 표시되고, 결제를 시도한 후 모달이 닫히며 애플리케이션으로 컨텍스트가 반환됩니다.
- 이벤트 수신 대상에서 `invoice.paid` 이벤트를 모니터링하여 결제가 성공했는지 확인합니다. 사용자가 `confirmCardPayment()` 완료 전에 애플리케이션을 떠날 수 있으므로, 결제 성공 여부를 확인하면 상품을 올바르게 제공할 수 있습니다.

<br/>

### 한눈에 보기
---

**구독 생명주기 요약**

```
구독 생성 → 인보이스 생성 (open) → 고객 결제
    ↓                                    ↓
체험 기간 (trialing)              결제 성공 → active
    ↓                                    ↓
23시간 내 미결제 → incomplete_expired   결제 실패 → incomplete
                                         ↓
                              재시도/인증 → active 또는 past_due
                                         ↓
                              최종 실패 → canceled/unpaid
```

<br/>

**주요 상태 전환**

| **이벤트**       | **이전 상태**                 | **이후 상태**             |
| ------------- | ------------------------- | --------------------- |
| 구독 생성 + 결제 대기 | -                         | `incomplete`          |
| 체험 기간 시작      | -                         | `trialing`            |
| 첫 결제 성공       | `incomplete` / `trialing` | `active`              |
| 23시간 내 미결제    | `incomplete`              | `incomplete_expired`  |
| 갱신 결제 실패      | `active`                  | `past_due`            |
| 재시도 모두 실패     | `past_due`                | `canceled` / `unpaid` |
| 구독 취소         | 모든 상태                     | `canceled`            |

<br/>

### 구독 시스템 빠르게 구축하기
---

#### Stripe Checkout으로 구독 페이지 만들기
---

Stripe Billing과 Stripe Checkout을 사용하여 완전히 작동하는 구독 연동을 구축해 보세요.

Stripe Billing API가 구독, 인보이스, 정기 결제를 생성하고 관리하며, Checkout이 결제 정보를 수집하는 안전한 Stripe 호스팅 UI를 제공합니다.

<br/>

#### (1단계) 상품, 가격 및 결제 방법을 설정
---

- 새로운 상품(Product)과 가격(Price)을 생성
- 상품에 기능을 추가

<br/>

#### (2단계) 구독 페이지 구성
---

- 가격 미리보기 페이지 추가
	- 제품 정보를 표시하고 구독할 수 있는 페이지 추가
- 체크아웃 버튼 추가
	- 버튼을 클릭하면 고객이 Stripe에서 호스팅하는 결제 페이지로 이동
	- 제품의 `lookup_key` 를 사용하여 서버에서 `price_id` 를 가져온다.
- 성공 페이지 추가
	- 이 페이지를 결제 세션의 `success_url` 과 연결
	- 고객이 결제를 완료한 후 Stripe가 해당 페이지로 리디렉션
- 고객 포털 버튼 추가
	- 고객이 구독을 관리할 수 있도록 고객 포털로 이동하는 버튼
	- 버튼을 클릭하면 Stripe에서 호스팅하는 고객 포털 페이지로 이동
- 고객 포털 세션으로 리디렉션
	- 새 고객 포털 세션으로 리디렉션 하기 위해 서버의 엔드포인트에 요청을 수행하십시오.
	- 이 예시는 결제 세션의 `session_id` 를 사용하여 `customer_id` 를 가져오는 방법을 보여줍니다.
	- 실제 운영 환경에서는 이 값을 인증된 사용자와 함께 데이터베이스에 저장할 것을 권장합니다.

<br/>

#### (3단계) Stripe API 호출
---

Stripe Java 라이브러리 설치 : `com.stripe.stripe-java`

기본 설정

```java
// 서버 포트
port(4242);

// Stripe 테스트용 비밀 API 키
String apiKey = "sk_test_...";

// 웹훅 엔드포인트 시크릿 (서명 검증용)
String endpointSecret = "whsec_12345";

final String YOUR_DOMAIN = "http://localhost:4242";
```

<br/>

결제 세션(Checkout Session) 생성
- 결제 세션은 고객이 Stripe 호스팅 결제 페이지에서 보는 항목을 제어한다.

```java
SessionCreateParams params = 
  SessionCreateParams
    .builder()
    .addLineItem(
      SessionCreateParams
        .LineItem
        .builder()
          .setPrice(
            prices
              .getData()
              .get(0)
              .getId()
          )
          .setQuantity(1L)
          .build()
    )
    .setMode(SessionCreateParams.Mode.SUBSCRIPTION)
    .setSuccessUrl(YOUR_DOMAIN + "?success=true&session_id={CHECKOUT_SESSION_ID}")
    .build();

Session session = Session.create(params);

response.redirect(session.getUrl(), 303);
```
- `addLineItem` : 제품 제고에 관한 가격 및 재고 상태와 같은 민감한 정보는 항상 서버에 보관하여 클라이언트 측의 고객 조작을 방지하십시오.
- `setMode` : 모드를 `subscription` 으로 설정해라. 결제 시에는 비정기 결제를 위한 결제 및 설정 모드로 지원한다.
- `setSuccessUrl` : Stripe가 결제 성공 후 고객을 리디렉션할 수 있는 공개적으로 접근 가능한 URL을 지정해라. URL 끝에 `session_id` 쿼리 매개변수를 추가하면 나중에 고객을 조회할 수 있으며 Stripe가 고객의 호스팅된 대시보드를 생성할 수 있다.
- `redirect` : 세션을 생성한 후, 응답에서 반환된 URL로 고객을 리디렉션해라.

<br/>

lookup key로 가격 조회
- 주문에 해당 상품의 가격을 적용하려면 가격 엔드포인트에서 상품에 대한 정의한 조회 키를 전달해라.

```java
PriceListParams priceParams =
  PriceListParams
    .builder()
    .addLookupKeys(
      request
        .queryParams("lookup_key")
        .build();
    )
    .build();
    
PriceCollection prices = Price.list(priceParams);
```

<br/>

고객 포털 세션 생성
- 고객이 구독 및 결제 정보를 관리할 수 있도록 Stripe에서 호스팅하는 안전한 고객 포털 세션을 시작해라.

```java
SessionCreateParams = 
  new SessionCreateParams
    .Builder()
    .setCustomer(checkoutSession.getCustomer())
    .setReturnUrl(YOUR_DOMAIN).build();
    
Session portalSession = Session.create(params):
```

<br/>

고객 포털로 리디렉션

- 포털 세션을 생성한 후, 응답에서 반환된 URL로 고객을 리디렉션하십시오.

```java
response.redirect(portalSession.getUrl(), 303);
```

<br/>

구독을 완료하세요
- 구독 활동 관련 이벤트를 수신하려면 Workbench의 웹훅 탭에서 /webhook 엔드포인트를 생성하고 웹훅 비밀 키를 획득하세요.
- 결제 성공 후 성공 페이지로 리디렉션되면 구독 상태가 활성인지 확인하고 고객이 제품 및 기능에 대한 접근 권한을 부여하세요.

```java
post("/webhook", (request, response) -> {
  String payload = request.body();
  Event event = null;
  
  // 1단계: JSON 파싱
  try {
    event = ApiResource.GSON.fromJson(payload, Event.class);
  } catch (JsonSyntaxException e) {
    // 잘못된 페이로드
    System.out.println("기본 요청 파싱 중 웹훅 오류 발생")
    response.status(400);
    return "";
  }
  
  // 2단계: 서명 검증 (보안)
  String sigHeader = request.headers("Stripe-Signature");
  if (endpointSecret != null && sigHeader != null) {
    try {
      event = Webhook.constructEvent(payload, sigHeader, endpointSecret);
    } catch (SignatureVerificationException e) {
      // 잘못된 서명
      System.out.println("서명 검증 중 웹훅 오류 발생");
      response.status(400);
      return "";
    }
  }
  
  // 3단계: 이벤트 내부의 객체 역직렬화
  EventDataObjectDeserializer dataObjectDeserializer = 
    event.getDataObjectDeserializer();
  StripeObject stripeObject = null;
  if (dataObjectDeserializer.getObject().isPresent()) {
    stripeObject = dataObjectDeserializer.getObject().get();
  } else {
    // 역직렬화 실패 - API 버전 불일치 가능성
    // EventDataObjectDeserializer Javadoc을 참고하여 처리하거나
    // 여기서 에러를 반환하세요
  }
  
  // 4단계: 이벤트 유형별 처리
  Subscription subscription = null;
  switch (event.getType()) {
    case "customer.subscription.deleted":
      // 구독이 삭제됨
      subscription = (Subscription) stripeObject;
      // handleSubscriptionDeleted(subscription); 같은 함수 호출
	  
	case "customer.subscription.trial_will_end":
	  // 구독 체험 기간이 곧 종료됨
	  subscription = (Subscription) stripeObject;
	  // handleSubscriptionTrialEnding(subscription);
	  
    case "customer.subscription.created":
      // 새 구독이 생성됨
      subscription = (Subscription) stripeObject;
      // handleSubscriptionCreated(subscription);
    
    case "customer.subscription.updated":
      // 구독이 업데이트됨
      subscription = (Subscription) stripeObject;
      // handleSubscriptionUpdated(subscription);
    
    case "entitlements.active_entitlement_summary.updated":
      // 활성 권한 요약이 업데이트됨
      subscription = (SubscriptionObject);
      // handleEntitlementUpdated(subscription);
      
    default:
      System.out.println("처리되지 않은 이벤트 유형: " + event.getType());
  }
  
  response.status(200);
  return "";
})
```

<br/>

#### 전체 흐름 요약
---

1. 고객이 구독 버튼 클릭
	1. /create-checkout-session 호출
	2. Stripe 결제 페이지로 이동
2. 결제 완료
	1. Stripe가 /webhook으로 이벤트 전송
	2. `customer.subscription.created` 이벤트 처리
3. 고객이 구독 관리 원함
	1. /create-portal-session 호출
	2. Stripe 고객 포털로 이동
4. 구독 변경/취소 시
	1. Stripe가 /webhook으로 이벤트 전송
	2. 해당 이벤트 처리

<br/>

#### 전체 코드 정리
---

```java
public class Server {
  // JSON 직렬화/역직렬화를 위한 Gson 인스턴스
  private static Gson gson = new Gson();
  
  public static void main(String[] args) {
    // 서버 포트 설정
    port(4242);
    
    // Stripe 테스트용 비밀 API 키 설정
    // 실제 운영 환경에서는 환경 변수나 설정 파일에서 불러와야 함
    String apiKey = "****";
    
    // 웹훅 서명 검증을 위한 엔드포인트 시크릿
    String endpointSecret = "whsec_12345";
    
    // 서비스 도메인 URL (리다이렉트에 사용)
    final String YOUR_DOMAIN = "http://localhost:4242";
    
    // 엔드포인트 1: 체크아웃 세션 생성
    // 고객이 구독을 시작할 때 호출됨
    post("/create-checkout-session", (request, response) -> {
      // 요청에서 lookup_key를 가져와서 해당하는 가격(Price) 정보 조회
      PriceListParams priceParams = 
        PriceListParams
          .builder()
          .addLookupKeys(request.queryParams("lookup_key"))
          .build();
	  PriceCollection prices = Price.list(priceParams);
	  
	  // 체크아웃 세션 파라미터 설정
	  SessionCreateParams params = 
	    SessionCreateParams
	      .builder()
	      // 구매 항목 추가 (가격 ID와 수량 설정)
	      .addLineItem(
	        SessionCreateParams
	          .LineItem
	          .builder()
	          // 조회한 가격의 ID
	          .setPrice(prices.getData().get(0).getId())
	          // 수량 1개
	          .setQuantity(1L)
	          .build()
	      )
	      // 결제 모드를 구독(SUBSCRIPTION)으로 설정
	      .setMode(SessionCreateParams.Mode.SUBSCRIPTION)
	      // 결제 성공 시 리다이렉트될 URL (세션 ID 포함)
	      .setSuccessUrl(YOUR_DOMAIN + "?success=true&session_id={CHECKOUT_SESSION_ID}")
	      // 구독에 추가 데이터 설정
	      .setSubscriptionData(
	        SessionCreateParams
	          .SubscriptionData
	          .builder()
	          // 7일 무료 체험 기간 설정
	          .setTrialPeriodDays(7L)
	          // 결제 주기 기준일 설정 (Unix 타임스탬프)
	          .setBillingCycleAnchor(1672531200)
	          // 자동 세금 계산 활성화 (고객 위치 기반)
	          .setAutomaticTax(
	            SessionCreateParams
	              .AutomaticTax
	              .builder()
	              .setEnabled(true)
	              .build()
	          )
	          .build();
	      )
	      .build();
	      
	    // Stripe API를 호출하여 체크아웃 세션 생성
	    Session session = Session.create(params);
	    
	    // 생성된 세션의 URL(Stripe 결제 페이지)로 리다이렉트
	    response.redirect(session.getUrl(), 303);
	    return "";
    });
    
    // 엔드포인트 2: 고객 포털 세션 생성
    // 고객이 구독을 관리(변경, 취소 등)할 때 호출됨
    post("/create-portal-session", (request, response) -> {
      // 체크아웃 세션 ID로 세션 정보 조회하여 고객 ID 획득
      // 실제 서비스에서는 DB에 저장된 고객 ID를 사용하는 것이 일반적
      Session checkoutSession = Session.retrieve(request.queryParams("session_id"))
      
      // 고개 포털 세션 파라미터 설정
      SessionCreateParams params =
        new SessionCreateParams
          .Builder()
          // 고객 ID 설정
          .setCustomer(checkoutSession.getCustomer())
          // 포털에서 돌아올 URL
          .setReturnUrl(YOUR_DOMAIN)
          .build();
    
    
      // Stripe API를 호출하여 고객 포털 세션 생성
      Session portalSession = Session.create(params);
      
      // 생성된 포털 URL로 리다이렉트
      response.redirect(portalSession.getUrl(), 303);
      return "";
    });
    
    // 엔드포인트 3: 웹훅 처리
    // Stripe에서 발생하는 이벤트를 실시간으로 수신
    post("/webhook", (request, response) -> {
      // 요청 본문(페이로드) 추출
      String payload = request.body();
      Event event = null;
      
      // 1단계: JSON 파싱 시도
      try {
        event = ApiResource.GSON.fromJson(payload, Event.class);
      } catch (JsonSyntaxException e) {
        // JSON 형식이 잘못된 경우
        System.out.println("요청을 파싱하는 동안 웹훅 에러 발생")
        response.status(400);
        return "";
      }
      
      // 2단계: 서명 검증 (보안)
      // Stripe-Signature 헤더를 사용하여 요청이 실제 Stripe에서 온 것인지 확인
      String sigHeader = request.headers("Stripe-Signature");
      if (endpointSecret != null && sigHeader != null) {
        try {
          // 서명 검증과 함께 이벤트 재구성
          event = Webhook.constructEvent(payload, sigHeader, enpointSecret);
        } catch (SignatureVerificationException e) {
          // 서명이 유효하지 않은 경우
          System.out.println("서명 검증하는 동안 에러 발생");
          response.status(400);
          return "";
        }
      }
      
      // 3단계: 이벤트 데이터 객체 역직렬화
      EventDataObjectDeserializer dataObjectDeserializer =
        event.getDataObjectDeserializer();
        StripeObject stripeObject = null;
        if (dataObjectDeserializer.getObject().isPresent()) {
          // 역직렬화 성공
          stripeObject = dataObjectDeserializer.getObejct().get();
        } else {
          // 역직렬화 실패 - API 버전 불일치 가능성
        }
        
      // 4단계: 이벤트 유형별 처리
      Subscription subscription = null;
      switch (event.getType()) {
        case "customer.subscription.deleted":
          // 구독이 삭제/취소된 경우
          subscription = (Subscription) stripeObject;
          // handleSubscriptionDeleted(subscription); 호출하여 처리
          
        case "customer.subscription.trial_will_end":
          // 구독 체험 기간이 곧 종료되는 경우 (보통 3일전 발송)
          subscription = (Subscription) stripeObject;
          // handleSubscriptionTrialEnding(subscription); 호출하여 처리
          
	    case "customer.subscription.created":
	      // 새로운 구독이 생성된 경우
	      subscription = (Subscription) stripeObject;
	      // handleSubscriptionCreated(subscription); 호출하여 처리
	      
	    case "customer.subscription.updated":
	      // 구독 정보가 업데이트된 경우 (플랜 변경, 결제 수단 변경 등)
	      subscription = (Subscription) stripeObject;
	      // handleSubscriptionUpdated(subscription); 호추라여 처리
	      
	    case "entitlements.active_entitlement_summary.updated":
	      // 활성 권한(entitlement) 요약이 업데이트된 경우
	      subscription = (Subscription) stripeObject;
	      // handleEntitlementUpdated(subscription); 호출하여 처리
	      
	    default:
	      // 위에서 처리하지 않은 이벤트 유형
	      System.out.println("처리되지 않은 유형: " + event.getType());
      }
      
      // 웹훅 수신 성공 응답 (200 OK)
      response.status(200);
      return "";
    });
  }
}
```

<br/>

## 구독 연동 설계하기
---

구독을 연동할 때 다음 사항들을 고려해야 한다.

- 고객에게 어떻게 과금할 것인가
- 고객이 어떻게 결제하고 결제 정보를 제공할 것인가
- 고객이 언제 구독료를 결제할 것인가

<br/>

### 가격 모델 결정
---

다음 가격 모델들을 비교하고 상품이나 서비스 구독에 대해 고객에게 어떻게 과금할지 결정하세요:

- **정액제(Flat rate)** : 고객이 선택한 서비스 등급에 대해 고정 금액을 청구한다.
- **사용자당 과금(Per-seat)** : 한 명의 사용자 또는 좌석을 나타내는 가격 단위별로 청구한다.
- **구간별 과금(Tiered)** : 수량이나 사용량에 따라 가격 단위(사용자나 좌석 등)당 다른 금액을 청구한다.
- **사용량 기반(Usage-based)** : 상품이나 서비스 사용량에 따라 청구한다.

<br/>

**정액제(Flat rate)**

아래 예시에서는 Basic, Starter, Enterprise 세 가지 서비스 등급을 제공한다. 각 등급에 대해 월간 및 연간 가격을 지정한다.

![stripe2](/assets/img/stripe2.png)

<br/>

**사용자당 과금(Per-seat)**

아래 예시에서는 소프트웨어 라이선스에 대한 사용자당 플랜을 제공합니다. 각 사용자의 라이선스에 대해 특정 금액을 청구합니다.

![stripe3](/assets/img/stripe3.png)

<br/>

**구간별 과금(Tiered)**

**볼륨 기반 가격(Volume-based pricing)**: 기간 말 총 사용량이 해당하는 구간의 단가를 전체 수량에 곱합니다. 즉, 최종 사용량이 속한 구간의 단가 하나만 적용됩니다.

**누진 가격(Graduated pricing)** : 기간 동안 각 구간별로 사용된 수량에 해당 구간의 단가를 곱한 후, 모든 구간의 금액을 합산합니다. 볼륨 기반 가격과 달리, 여러 구간의 단가가 단계별로 적용된다.

*예시로 이해하기*

| 구간  | 프로젝트 수 | 단가   |
| --- | ------ | ---- |
| 1구간 | 1-5개   | $7   |
| 2구간 | 6-10개  | $6.5 |
| 3구간 | 11개 이상 | $6   |

*고객이 12 프로젝트를 사용한 경우*

- 볼륨 기반 : 12 x $6 = $72
- 누진 : (5 x $7) + (5 x $6.5) + (2 x $6) = 79.5

<br/>

**사용량 기반**

- 고정 요금 + 초과분 가격
	- 상품이나 서비스에 대해 월 고정 요금을 청구합니다.
	- 고정 요금에는 일정량의 사용 권한이 포함되어 있고, 추가 사용량(초과분)은 기간 말에 청구합니다.
- 종량제 가격
	- 특정 기간 동안 추적된 사용량에 대해 청구합니다.
	- 단위당, 패키지당, 볼륨 기반, 누진 등 다양한 가격 책정 방식을 사용할 수 있습니다.
- 크레딧 소진 가격
	- 사용량 기반 상품이나 서비스에 대해 선불로 결제받고, 고객이 사용하면 결제 크레딧을 적용할 수 있게 합니다.

*과금 예시*

| 구간      | 시작 단위   | 마지막 단위  | 단위당    | 정액    |
| ------- | ------- | ------- | ------ | ----- |
| 첫 번째 구간 | 0       | 100,000 | $0.001 | $1.00 |
| 두 번째 구간 | 100,001 | 무한      | $0.002 | $2.00 |

<br/>

### 결제 방식 결정
---

다음 결제 인터페이스들을 비교하고 고객이 구독을 위해 결제 정보를 어떻게 제공할지 결정하세요.

<br/>

#### Stripe 호스팅 페이지
---

Stripe가 미리 만들어 호스팅하는 결제 페이지를 사용합니다.

Stripe가 결제 수단 수집과 검증을 처리하고, 구독 프로세스를 자동으로 시작합니다.

![stripe4](/assets/img/stripe4.png)

<br/>

#### 임베디드 결제 양식
---

Stripe가 미리 만들어 호스팅하는 결제 양식을 사이트에 직접 삽입합니다.

Stripe가 결제 수단 수집과 검증을 처리하고, 구독 프로세스를 자동으로 시작합니다.

![stripe5](/assets/img/stripe5.png)

<br/>

#### 커스텀 결제 양식
---

웹사이트에 통합할 수 있는 UI 컴포넌트를 사용하여 커스텀 결제 양식을 만듭니다.

Stripe Elements를 웹/앱 프론트엔드와 결합하고, 결제 플로우에 맞게 Payment Element 레이아웃을 커스터마이징할 수 있습니다.

![stripe6](/assets/img/stripe6.png)

<br/>

#### 가격표
---

웹 사이트에 가격표를 삽입하여 구독 가격 정보를 표시합니다.

다양한 가격 옵션을 표시하고, 결제 플로우를 위해 Stripe 호스팅 결제 페이지로 리디액션한다.

![stripe7](/assets/img/stripe7.png)

<br/>

#### 원클릭 결제 버튼
---

다양한 결제 수단을 위한 원클릭 결제 버튼으로 결제를 받습니다.

![stripe8](/assets/img/stripe8.png)

<br/>

#### 결제 링크
---

고객에게 직접 공유할 수 있는 결제 링크를 생성합니다. 고객이 링크를 클릭하면 Stripe 호스팅 결제 페이지로 이동합니다.

![stripe9](/assets/img/stripe9.png)

<br/>

#### 모바일 앱
---

![stripe10](/assets/img/stripe10.png)

<br/>

### 결제 시점 결정
---

다음 모델들을 비교하고 고객이 구독료를 언제 결제할지 결정하세요.

<br/>

#### 선불(Pay up front) 결제
---

상품이나 서비스에 대한 접근 권한을 제공하기 전에 고객이 먼저 결제하도록 요구합니다.

1. 고객이 구독 플랜을 선택합니다.
2. 결제 정보를 수집합니다.
3. 상품이나 서비스에 대한 접근 권한을 부여합니다.
4. 구독 생명주기 동안 고객에게 지속적으로 접근 권한을 제공합니다.
5. 최초 과금 후, 동일한 서비스에 대해 동일한 고정 가격으로 정기적으로 계속 청구합니다.

<br/>

#### 무료 체험(Free trial) 플로우
---

과금하기 전에 고객에게 상품이나 서비스의 무료 체험 기간을 제공합니다.

1. 고객이 구독 플랜을 선택합니다.
2. 결제 정보를 수집하지만, 고객에게 청구하지 않습니다.
3. 제한된 시간 동안 상품이나 서비스에 대한 접근 권한을 부여합니다.
4. 체험 기간이 끝나면 새 청구 기간이 시작됩니다.
5. Stripe가 서비스에 정의된 가격으로 인보이스를 생성합니다.

<br/>

#### 프리미엄(Freemium) 플로우
---

결제 정보를 요청하지 않고 고객이 상품이나 서비스에 접근할 수 있게 합니다.

1. 고객이 구독 플랜을 선택합니다.
2. 제한된 시간 동안 상품이나 서비스에 대한 접근 권한을 부여합니다.
3. 체험 기간이 끝나기 전에 결제 정보를 수집합니다.
4. 체험 기간이 끝나면 새 청구 기간이 시작됩니다.
5. Stripe가 서비스에 정의된 가격으로 인보이스를 생성합니다.

<br/>

### 구독 연동 구축하기
---


| 가격 모델          | 결제 인터페이스                            | 결제 모델         | 사용 사례                                                                                                                                    | 연동 방법                                                         |
| -------------- | ----------------------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| 정액제            | 1. 결제 페이지<br>2. 임배디드 양식             | Free<br>trial | 구독에 무료 체험 기간을 제공하고, 체험 종료 후 과금할 결제 수단을 미리 수집하고 싶을 때 사용<br><br>Stripe 호스팅 양식 또는 직접 만든 커스텀 양식 중에 선택                                        | 1. Stripe 호스팅 페이지<br>2. 임베디드 결제 양식<br>3. 커스텀 결제 양식으로 무료 체험 시작 |
| 사용량 기반         | 1. 결제 페이지<br>2. 임베디드 양식<br>3. 모바일 앱 | -             | 고객의 상품/서비스 사용량에 따라 과금하고 싶을 때 사용<br><br>Stripe 호스팅 페이지, 결제 플로우에 삽입되는 Stripe 호스팅 양식, 직접 만든 커스텀 양식, 또는 모바일 앱의 결제 양식으로 결제 정보를 수집할 수 있습니다.    | 사용량 기반 과금 설정하기                                                |
| 정액제, 사용자당, 구간별 | 가격표                                 | Free trial    | 웹사이트에 삽입된 가격표로 다양한 구독 요금제를 보여주고 싶을 때 사용<br><br>정액제, 사용자당 과금, 구간별 과금, 무료 체험 등을 제공할 수 있습니다.<br>고객이 요금제를 선택하면 미리 만드어진 결제 양식에서 결제 정보를 입력합니다. | 웹사이트에 가격표 생성 및 삽입하기                                           |
| 정액제            | 결제 링크                               | 선불            | 정액 구독을 판매하고, 고객에게 공유할 수 있는 결제 링크로 결제 정보를 수집하고 싶을 때 사용<br><br>결제 링크를 클릭하면 Stripe 호스팅 결제 페이지로 이동합니다.                                       | 구독 생성 후 구독용 결제 링크 만들기                                         |
| 정액제            | 모바일 앱                               | 선불            | 정액 구독을 판매하고, 모바일 앱에 삽입된 커스텀 결제 양식으로 결제 정보를 수집하고 싶을 때 사용                                                                                  | iOS 앱, Androdi 앱 또는 React Native 앱에결제 양식 생성 및 삽입하기            |
| 정액제            | 원클릭 결제 버튼                           | 선불            | 정액 구독을 판매하고, 원클릭 결제 버튼으로 결제 정보를 수집하고 싶을 때 사용<br><br>Stripe 호스팅 결제 페이지, 결제 플로우에 삽입하는 Stripe 호스팅 양식, 또는 직접 만든 커스텀 양식에서 사용할 수 있습니다.         | 결제 페이지에 완료될 결제 버튼 추가하기                                        |

<br/>

## 구독 연동 구축하기
---

정기 결제를 받기 위한 구독을 생성하고 관리하는 방법을 알아보자.

<br/>

### Stripe 호스팅 페이지 방식
---

미리 만들어진 호스팅 페이지를 사용하여 결제를 수집하고 구독을 관리합니다.

<br/>

### 이 가이드에서 만들 것
---

이 가이드는 Stripe Checkout을 사용하여 고정 가격 월간 구독을 판매하는 방법을 설명합니다.

배우게 될 내용:
- 상품 카탈로그를 만들어 비즈니스 모델링하기
- 사이트에 Checkout 세션 추가하기 (버튼, 성공/취소 페이지 포함)
- 구독 이벤트를 모니터링하고 서비스 접근 권한 부여하기
- 고객 포털 설정하기
- 사이트에 고객 포털 세션 추가하기 (버튼, 리디렉션 포함)
- 고객이 포털을 통해 구독을 관리하도록 허용하기
- 유연한 과금 모드를 사용하여 향상된 과금 동작과 추가 기능에 접근하는 방법

연동을 완료한 후 다음 기능을 확장할 수 있습니다:
- 세금 표시
- 할인 적용
- 고객에게 무료 체험 기간 제공
- 결제 수단 추가
- 호스팅 인보이스 페이지 연동
- 설정 모드에서 Checkout 사용
- 사용량 기반 과금, 가격 구간 설정
- 비례 정산 관리
- 고객이 여러 상품을 구독할 수 있게 하기
- 권한(entitlement)을 연동하여 상품 기능 접근 관리

<br/>

### 1단계: Stripe 설정
---

Stripe 클라이언트를 설치합니다.

```kotlin
implementation("com.stripe:stripe-java:31.0.0")
```

선택적으로 Stripe CLI를 설치하세요. CLI는 웹훅 테스트를 제공하고, 상품과 가격을 생성하는 데 사용할 수 있습니다.

```
brew install stripe/stripe-cli/stripe

stripe login
```

<br/>

### 2단계: 가격 모델 생성 (Dashboard 나 Stripe CLI)
---

정기 가격 모델은 판매하는 상품/서비스, 비용, 결제 통화, 구독 서비스 기간을 나타냅니다. 가격 모델을 구성하려면 상품(판매 항목)과 가격(금액과 청구 주기)을 생성합니다.

<br/>

### 3단계: Checkout Session 생성 (Client 와 Server)
---

웹사이트에 Checkout Session을 생성하는 서버 엔드포인트를 호출하는 결제 버튼을 추가한다.

```html
<html>
  <head>
    <title>Checkout</title>
  </head>
  <body>
    <form action="/create-checkout-session" method="POST">
      <input type="hidden" name="priceId" value="price_G0FvDp6vZvdwRZ" />
      <button type="submit">결제하기</button>
    </form>
  </body>
</html>
```

- 서버에서 다음과 같은 값들로 세션을 생성하는 엔드포인트를 정의합니다.
	- 고객이 가입하는 구독의 PriceId(프론트엔드에서 전달)
	- `success_url` : 고객이 결제를 완료한 후 돌아올 페이지 URL
- 선택적으로 설정
	- 구독에 청구 주기 설정
	- 커스텀 텍스트를 사용하여 구독 및 취소 약관 포함

<br/>

2단계에서 일회성 가격을 생성했다면, 해당 가격 ID도 함께 전달하세요. Checkout 세션을 생성한 후, 응답에서 반환된 URL로 고객을 리디렉션합니다.

Checkout 세션을 생성할 때 과금 모드 타입을 `flexible` 로 설정하면 더 정확하고 예측 가능한 구독 동작을 활성화할 수 있습니다.

```java
// 시크릿 키를 설정합니다. 운영 환경에서는 라이브 시크릿 키로 전환하세요.
Stripe.apiKey = "***"

// 클라이언트에서 전달된 가격 ID
//   String priceId = request.queryParams("priceId");
String priceId = "{{PRICE_ID}}";

SessionCreateParams params = new SessionCreateParams.Builder()
  .setSuccessUrl("https://example.com/success.html?session_id={CHECKOUT_SESSION_ID}")
  .setMode(SessionCreateParams.Mode.SUBSCRIPTION)
  .addLineItem(new SessionCreateParams.LineItem.Builder()
    // 사용량 기반 과금의 경우 quantity를 전달하지 마세요
    .setQuantity(1L)
    .setPrice(priceId)
    .build()
  )
  .setSubscriptionData(
    new SessionCreateParams.SubscriptionData.Builder()
      .setBillingMode(SessionCreateParams.SubscriptionData.BillingMode.FLEXIBLE)
      .build()
  )
  .build();
  
Session session = Session.create(params);

// Checkout 세션에서 반환된 URL로 리디렉션
```

- 이 예제에서는 세션ID를 추가하여 `success_url` 을 커스터마이징합니다. 성공 페이지 커스터마이징에서 더 자세히 알아보세요.
- 대시보드에서 고객에게 받고 싶은 결제 수단을 활성화하세요. Checkout은 다양한 결제  수단을 지원합니다.

<br/>

### 4단계: 구독 프로비저닝 및 모니터링 (Server)
---

구독 가입이 성공하면 고객이 `success_url` 에서 웹사이트로 돌아오고, `checkout.session.completed` 웹훅(HTTPS 요청을 통해 JSON 페이로드로 애플리케이션에 전송되는 실시간 푸시 알림)이 시작됩니다.

`checkout.session.completed` 이벤트를 받으면 권한(entitlements)을 사용하여 구독을 프로비저닝합니다. 매월(월간 과금의 경우) `invoice.paid` 이벤트를 받을 때마다 프로비저닝을 계속합니다. `invoice.payment_failed` 이벤트를 받으면 고객에게 알리고 고객 포털로 보내 결제 수단을 업데이트하도록 합니다.

```java
// 시크릿 키를 설정합니다.
Stripe.apiKey = "***";

post("/webhook", (request, response) -> {
  String payload = request.body();
  String sigHeader = request.headers("Stripe-Signature");
  String endpointSecret = "{{STRIPE_WEBHOOK_SECRET}}"
  
  Event event = null;
  
  try {
    event = Webhook.constructEvent(payload, sigHeader, endpointSecret);
  } catch (SignatureVerificationException e) {
    // 서명이 유효하지 않음
    response.status(400);
    return "";
  }
  
  switch (event.getType()) {
    case "checkout.session.completed":
      // 결제가 성공하고 구독이 생성됨
      // 구독을 프로비저닝하고 고객 ID를 데이터베이스에 저장해야 함
      break;
    case "invoice.paid":
      // 결제가 계속될 때 구독 프로비저닝 유지
      // 상태를 데이터베이스에 저장하고 사용자가 서비스에 접근할 때 확인
      // 이 방식은 API 속도 제한에 걸리는 것을 방지하는 데 도움됨
      break;
    case "invoice.payment_failed":
      // 결제 실패 또는 유효한 결제 수단이 없음
      // 구독이 past_due 상태가 됨. 고객에게 알리고 고객 포털로 보내
      // 결제 정보를 업데이트하도록 함
      break;
    default:
      // 처리되지 않은 이벤트 유형 로깅
  }
  
  response.status(200);
  return "";
});
```

- `STRIPE_WEBHOOK_SECRET` : Workbench의 Webhooks 탭에서 대상 상세 보기로 이동해서 확인할 수 있다.
- 모니터링할 최소 이벤트 유형
	- `checkout.session.completed` : 고객이 Checkout Session을 성공적으로 완료했을 때 전송됩니다. 새 구매를 알려줍니다.
	- `invoice.paid` : 결제가 성공할 때마다 각 청구 기간에 전송됩니다.
	- `invoice.payment_failed` : 고객의 결제 수단에 문제가 있을 때 각 청구 기간에 전송됩니다.

<br/>

### 5단계: 고객 포털 설정 (Dashboard)
---

고객 포털을 사용하면 고객이 기존 구독과 인보이스를 직접 관리할 수 있습니다.

대시보드에서 포털을 설정합니다. 최소한 고객이 결제 수단을 업데이트할 수 있도록 포털을 설정하세요.

<br/>

### 6단계: 포털 세션 생성 (Server)
---

프론트엔드가 호출할 고객 포털 세션을 생성하는 엔드포인트를 정의합니다. `CUSTOMER_ID` 는 `checkout.session.completed` 이벤트를 처리하는 동안 저장한 Checkout 세션이 생성한 고객 ID입니다.

```java
// 시크릿 키를 설정
Stripe.apiKey = "***"

// 사용자가 포털에서 구독 관리를 마친 후 리디렉션될 URL
String domainUrl = "{{DOMAIN_URL}}"
String customer = "{{CUSTOMER_ID}}"

SessionCreateParams params =
  new SessionCreateParams.Builder()
    .setReturnUrl(domainUrl)
    .setCustomer(customer)
    .build();
    
Session portalSession = Session.create(params)

// 세션 URL로 리디렉션
//   response.redirect(portalSession.getUrl(), 303);
//   return "";
```

<br/>

### 7단계: 고객을 고객 포털로 보내기 (Client)
---

프론트엔드의 `success_url` 페이지에 고객 포털로 연결되는 버튼을 추가합니다.

```html
<html>  
<head>  
  <title>Manage Billing</title>  
</head>  
<body>  
<form action="/customer-portal" method="POST">  
  <!-- Note: If using PHP set the action to /customer-portal.php -->  
  <button type="submit">Manage Billing</button>  
</form>  
</body>  
</html>
```

- 고객 포털을 나간 후, 고객은 `return_url` 에서 웹사이트로 돌아옵니다.
- 고객의 구독 상태를 추적하기 위해 구독 이벤트를 모니터링해라.
- 구독 취소 같은 작업을 허용하도록 고객 포털을 설정하면 관련 이벤트를 모니터링해라.

<br/>

### 8단계: 연동 테스트
---

다음 표를 사용하여 다양한 결제 수단과 시나리오를 테스트하세요.


| 결제 수단             | 시나리오                         | 테스트 방법                                             |
| ----------------- | ---------------------------- | -------------------------------------------------- |
| 신용카드              | 결제 성공, 인증 불필요                | 카드 번호 `4242 4242 4242 4242` 와 아무 만료일, CVC, 우편변호 사용 |
| 신용카드              | 결제에 인증(3DS) 필요               | 카드 번호 `4000 0025 0000 3155` 사용                     |
| 신용카드              | 카드 거절 (`insufficient_funds`) | 카드 번호 `4000 0000 0000 9995` 사용                     |
| BECS Direct Debit | 결제 성공                        | 계좌 번호 `900123456`, BSB `000000` 사용                 |
| BECS Direct Debit | `account_closed` 오류          | 계좌 번호 `111111113`, BSB `000000` 사용                 |
| SEPA Direct Debit | 결제 성공                        | 계좌 번호 `AT321904300235473204` 사용                    |
| SEPA Direct Debit | 결제 실패                        | 계좌 번호 `AT861904300235473202` 사용                    |

<br/>

### 한눈에 보기: 구독 연동 플로우
---

```
1. Stripe 설정  
   └── SDK 설치  
  
2. 가격 모델 생성  
   ├── 상품 생성 (Basic, Premium)  
   └── 가격 생성 (월간 정기)  
  
3. Checkout 세션 생성  
   ├── 클라이언트: 결제 버튼  
   └── 서버: 세션 생성 엔드포인트  
  
4. 웹훅으로 이벤트 모니터링  
   ├── checkout.session.completed → 구독 프로비저닝  
   ├── invoice.paid → 접근 권한 유지  
   └── invoice.payment_failed → 고객에게 알림  
  
5. 고객 포털 설정  
   ├── 대시보드에서 포털 설정  
   └── 포털 세션 생성 엔드포인트  
  
6. 테스트  
   └── 테스트 카드로 결제 시뮬레이션
```

<br/>

## 정기 결제 가격 모델 구축하기
---

가격 모델은 Stripe에서 비즈니스를 나타내는 패턴으로, 판매하는 상품이나 서비스, 비용, 결제에 사용할 통화, 구독의 서비스 기간으로 구성됩니다. 가격 모델을 구축하려면 상품(Product)과 가격(Price)을 사용합니다.


| 가격 모델                | 설명                                        |
| -------------------- | ----------------------------------------- |
| 정액제 (Flat rate)      | 고객이 서비스 등급을 선택하고 해당 등급에 대해 고정 금액을 지불합니다.  |
| 사용자당 과금 (Per-seat)   | 각 가격 단위가 한 명의 사용자를 나타냅니다.                 |
| 구간별 과금 (Tiered)      | 수량(볼륨 기반 가격) 또는 사용량(누진 가격)에 따라 단가가 변동합니다. |
| 사용량 기반 (Usage-based) | 고객의 상품이나 서비스 사용량에 따라 과금합니다.               |

<br/>

### 정액제(Flat rate) 가격 설정하기
---


<br/>

## Reference
---

<https://docs.stripe.com/revenue>

<https://youtu.be/CUAY6IQcVQM>