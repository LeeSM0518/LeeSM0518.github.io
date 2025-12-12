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

<br/>

## Billing API 소개
---

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
| **Invoice (청구서)**          | 고객이 지불해야 할 금액 명세서로, 초안부터 결제 완료 또는 확정까지 <br>결제 상태를 추적합니다.<br>구독이 자동으로 인보이스를 생성합니다.                                                                                      |
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
