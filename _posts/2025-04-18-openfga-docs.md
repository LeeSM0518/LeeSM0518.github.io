---
title: OpenFGA Docs
date: 2025-04-18 00:00:00 +0900
categories: openfga
tags:
  - openfga
  - permission
  - authorization
description: OpenFGA 공식 문서 정리
---

## OpenFGA 소개
---

OpenFGA는 개발자를 위한 확장 가능한 오픈 소스 인증 시스템으로, 모든 종류의 애플리케이션에 대한 인증을 구현할 수 있으며 시간이 지나면서 복잡성이 증가함에 따라 원활하게 발전시킬 수 있다.

Google의 내부 승인 시스템인 Zanzibar에서 영감을 받아, OpenFGA는 관계 기반 액세스 제어를 사용하는데, 이를 통해 개발자는 역할 기반 액세스 제어를 쉽게 구현할 수 있으며 속성 기반 액세스 제어를 구현하기 위한 추가 기능을 제공한다.

<br/>

### OpenFGA 이점
---

- 권한 부여 로직을 애플리케이션 코드 외부로 옮겨서 작성, 변경, 심사를 더 쉽게 만든다.
- 단일 승인 솔루션을 표준화하여 속도를 높인다.
- 보안 및 규정 준수 요구 사항을 준수하는 것이 더 쉬워지도록 권한 부여 결정과 감사 로그를 중앙에서 관리한다.
- 권한 부여 정책을 발전시키는 것이 더 간단해지면서 제품이 더 빨리 발전할 수 있다.

<br/>

### OpenFGA 기능들
---

- 다른 환경들(prod/testing/dev)과 유스케이스(내부 앱, 외부 앱, 인프라)의 권한 부여 관리를 위한 다중 스토어를 지원한다.
- 상황적 튜플과 조건적 관계 튜플을 사용한 일부 ABAC(속성 기반) 시나리오 지원
- Java, .NET, Javascript, Go, Python 지원
- HTTP와 gRPC 제공
- Postgres, MySQL, SQLite를 프로덕션 데이터 저장소로 사용할 수 있도록 지원하고, 비프로덕션 사용을 위한 메모리 내 데이터 저장소도 지원한다.
- OpenFGA 스토어나, 테스트 모델, 가져오기/내보내기 모델들, 데이터를 관리하기 위한 CLI

<br/>

## Authorization Concepts
---

### 인증(Authentication)과 인가(Ahorization)
---

인증은 사용자의 신원을 보장한다. 인가는 사용자가 특정 리소스에 대해 특정 작업을 수행할 수 있는지 여부를 결정한다.

<br/>

### 세분화된 인가 (FGA, Fine-Grained Authorization)
---

FGA는 특정 사용자에게 특정 리소스에서 특정 작업을 수행할 수 있는 권한을 부여하는 기능을 의미한다.

잘 설계된 FGA 시스템은 수백만개의 객체들과 사용자들의 권한을 관리할 수 있다. 이러한 권한은 시스템이 지속적으로 객체를 추가하고 액세스 권한을 업데이트함에 따라 빠르게 변경될 수 있다.

FGA의 대표적인 예로는 Google Drive가 있다. 접근 권한은 문서나 폴더에 부여할 수 있으며, 개별 사용자나 그룹 사용자에게도 부여할 수 있다. 또한, 새로운 문서가 생성되어 특정 사용자나 그룹과 공유되면 접근 권한은 정기적으로 변경된다.

<br/>

### 역할 기반 접근 제어 (RBAC, Role-Based Access Control)
---
RBAC에서는 시스템 내에서의 역할에 따라 사용자에게 권한이 할당된다. 예를 들어, 사용자가 내용을 수정하기 위해서는 `editor` 역할이 필요하다.

RBAC 시스템을 사용하면 사용자, 그룹, 역할 및 권한을 정의하고 중앙화된 위치에 저장할 수 있다. 애플리케이션은 해당 정보에 접근하여 권한 부여 결정을 내린다.

<br/>

### 속성 기반 접근 제어 (ABAC, Attribute-Based Access Control)
---

ABAC에서는 권한은 사용자나 리소스가 소유한 속성 집합을 기반으로 부여된다. 예를 들어, `marketing` 과 `manager` 속성이 할당된 사용자는 `marketing` 속성이 있는 게시물을 삭제할 수 있는 권한이 있다. 

ABAC를 구현하는 애플리케이션은 권한 부여 결정을 내리기 위해 RBAC 서비스, 사용자 디렉토리, 애플리케이션별 데이터 소스 등 여러 데이터 소스에 저장된 정보를 검색해야 한다.

<br/>

### 정책 기반 접근 제어 (PBAC, Policy-Based Access Control)
---

PBAC는 애플리케이션 코드 외부에서 중앙 집중식으로 권한 부여 정책을 관리하는 기능이다. ABAC의 대부분 구현은 PBAC이기도 하다.

<br/>

### 관계 기반 접근 제어 (ReBAC, Relationship-Based Access Control)
---

ReBAC를 사용하면 사용자 액세스 규칙이 특정 사용자와 특정 객체 간의 관계 및 해당 객체와 다른 객체 간의 관계에 따라 달라질 수 있다. 예를 들어, 특정 사용자는 해당 문서의 상위 폴더에 대한 접근 권한이 있는 경우에만 해당 문서를 볼 수 있다.

ReBAC는 RBAC의 상위 집합이다. ReBAC을 사용하여 RBAC를 구현할 수 있다. ReBAC는을 사용하면 속성을 관계의 형태로 표현할 수 있는 경우에도 기본적으로 ABAC를 해결할 수 있다. 예를 들어, '사용자 관리자', '상위 폴더', '문서 소유자', '사용자 부서' 등을 관계로 정의할 수 있다.

OpenFGA는 조건이나 상황적 튜플을 사용하여 추가적인 ABAC 시나리오를 표현하는 것을 더 간편하게 만들어 ReBAC를 확장한다.

ReBAC는 권한 부여 정책이 중장 집중화되어 있으므로 PBAC로 간주될 수도 있다.

<br/>

### Zanzibar
---

Zanzibar는 구글의 글로벌 권한 부여 시스템이다. 이는 ReBAC을 기반으로 하며 객체-관계-사용자 튜플을 사용하여 관계 데이터를 저장한 다음 사용자와 객체 간의 일치 여부를 확인하기 위해 해당 관계를 검사한다. Zanzibar 기반의 ReBAC 시스템은 권한 부여 결정을 내리는데 필요한 데이터를 중앙 데이터베이스에 저장한다. 애플리케이션은 API를 호출하기만 하면 권한 부여 결정을 내릴 수 있다.

OpenFGA는 Zanzibar 기반 인증 시스템의 예시이다.

<br/>

## OpenFGA Concepts
---

OpenFGA 서비스는 객체와 사용자 사이에 관계가 있는지 여부를 판단하여 인가 확인에 응답한다. 검사는 인가 권한을 위해 관계 튜플에 대한 인가 모델을 참조한다. 아래에는 타입 및 인가 모델과 같은 기본 FGA 개념에 대한 설명과 지식을 테스트할 수 있는 playground를 설명한다.

<br/>

### Type

Type은 문자열이다. 이는 유사한 특성을 가진 객체의 클래스를 정의한다. 

ex. `workspace` , `repository` , `organization` , `document` 

<br/>

### Type Definition

Type Definition 사용자 또는 다른 객체가 이 type과 관련하여 가질 수 있는 모든 관계를 가진다.

Type Definition 예시는 다음과 같다.

```
type document
  relations
    define viewer: [user]
    define commenter: [user]
    define editor: [user]
    define owner: [user]
```

<br/>

### Authorization Model

Authorization Model은 하나 이상의 type definitions를 결합한 것이다. 시스템의 권한 모델을 정의하는 데 사용된다.

Authorization Model 예시는 다음과 같다.

```
model
  schema 1.1

type document
  relations
    define viewer: [domain#member, user]
    define commenter: [domain#member, user]
    define editor: [domain#member, user]
    define owner: [domain#member, user]

type domain
  relations
    define member: [user]

type user
```
- 인가 모델은 관계 튜플과 함께 사용자와 객체 간의 관계가 있는지 여부를 판별한다.
- OpenFGA는 인가 모델을 정의하기 위해 JSON 이나 DSL을 사용한다.

<br/>

### Store

Store는 인가 확인 데이터로 구성되는 OpenFGA 엔티티이다.

각 저장소는 여러 버전의 인가 모델을 포함하고 다양한 관계 튜플을 포함할 수 있다. 저장소 데이터는 저장소끼리 공유할 수 없다. 관련되거나 인가 결과에 영향이 있는 모든 데이터들은 하나의 저장소에 저장하는 것을 권장한다. 별도의 인가 요구 사항이나 격리된 환경에 대해 별도의 저장소를 생성할 수도 있다.

<br/>

### Object

Object는 시스템 내의 엔티티를 나타낸다. 사용자와 서비스의 관계는 관계 튜플과 인가 모델을 통해 정의된다.

Object는 type과 id의 조합이며, 예시는 다음과 같다.
- `workspace:fb83c013-3060-41f4-9590-d3233a67938f`
- `repository:auth0/express-jwt`
- `organization:org_ajUc9kJ`
- `document:new-roadmap`

user, relation, object는 relationship tuples의 구성 요소이다.

<br/>

### User
---

user는 object와 관련될 수 있는 시스템 내의 entity이다.

user는 type, id, relation의 결합이며, 예시는 다음과 같다.
- identifier 예시 : `user:anne` , `user:{UUID}`
- object 예시 : `workspace:{UUID}` , `repository:auth0/express-jwt` , `organization:org_ajUc9kJ`
- user의 그룹 예시 : `organization:org_ajUc9kJ#members` (org_ajUc9kJ와 관련된 member)
- 모든 : `*`

<br/>

### Relation
---

relation은 authorization model의 type definition으로 정의된 문자열이다. relations는 시스템의 object와 user 사이의 접근 가능한 관계를 정의한다. relation의 예시는 다음과 같다.
- 사용자는 문서의 `reader` 가 될 수 있다.
- 팀은 저장소의 `administer` 가 될 수 있다.
- 사용자는 팀의 `member` 가 될 수 있다.

<br/>

### Relation Definition
---

relation definition은 관계가 가능한 조건 또는 요구사항이 나열된다.

예시는 다음과 같다.

- `document` 타입에서 사용자와 객체 간의 가능한 관계를 설명하는 `editor` 는 다음을 허용한다.
    - **사용자 식별자와 객체 간의 관계** : 사용자 타입의 사용자 ID `anne`  는  `document:roadmap` 과 `editor` 로 연관된다.
    - **객체와 객체 간의 관계** : 객체 `application:iffft` 는 객체 `document:roadmap` 과 `editor` 로 연관된다.
    - **사용자 집합과 객체 간의 관계** : 사용자 집합 `organization:auth0.com#member` 는 `document:roadmap` 과 `editor` 로 연관된다.
        - `organization:auth0.com` 과 `member` 로 관련된 사용자들의 집합이 `document:roadmap` 과 `editor` 로 관련되어 있음을 나타낸다.
        - 회사나 팀의 모든 멤버들이 문서를 공유하는 것과 같은 사용 사례에 대한 해결책을 제공한다.
    - **모든 것과 객체 간의 관계** : 모든 것(`*`)은 `document:roadmap` 과 `editor` 와 연관된다.
        - 이는 공개적으로 편집 가능한 문서를 모델링하는 방법이다.

<br/>

인가 모델은 다음과 같이 정의할 수 있다.

```
type document
  relations
    define viewer: [user]
    define commenter: [user]
    define editor: [user]
    define owner: [user]

type user
```
- `document` 타입 구성에 `viewer` , `commenter` , `editor` , `owner` 관계가 존재

<br/>

### Directly Related User Type
---

directly related user type은 관계에 직접적으로 연관된 사용자의 타입들을 정의한 배열이다.

<br/>

다음 모델의 경우 사용자 유형이 `user` 인 관계 튜플만 문서에 할당할 수 있다.

```
type document
  relations
    define viewer: [user]
```

<br/>

사용자  `user:anne` 나 `user:3f7768e0-4fa7-4e93-8417-4da68ce1846c` 관계 튜플은 타입 `document` 와 관계 `viewer` 로 구성된 객체로 쓰일 수도 있으며, 다음과 같이 작성하면 된다. 

```json
{
  "user" : "user:anne",
  "relation" : "viewer",
  "object" : "document:roadmap"
}
```

<br/>

`document` 타입의 객체에 대한 `viewer` 관계에 대해 허용되지 않는 사용자 타입이 있는 관계 튜플이다. 예를 들어, `workspace:auth0` 나 `folder:planning#editor` 는 작성을 실패한다. 

```json
{
  "user" : "folder:product",
  "relation" : "viewer",
  "object" : "document:roadmap"
}
```
- 입력 시 실패하는 데이터

<br/>

### Condition
---

condition은 하나 이상의 파라미터와 표현식들로 구성된 함수이다. 모든 condition은 boolean을 반환하며, 표현식은 구글의 Common Expression Language(CEL)로 정의된다.

<br/>

아래 보이는 `less_than_hundred` 는 boolean을 반환하는 condition을 정의한다. integer 타입의 `x` 파라미터는 `x < 100` boolean 표현식에 사용된다. condition은 true나 false를 반환한다.

```
condition less_than_hundred(x: int) {
  x < 100
}
```

<br/>

### Relationship Tuple
---

relationship tuple은 사용자, 관계, 객체로 구성된 기본 튜플/트리플렛 이다. 튜플은 Conditional Relationship Tuple과 같이 선택적 조건을 추가할 수 있다. relationship tuple은 OpenFGA에 작성되어 저장된다. 

<br/>

relationship tuple은 다음과 같이 구성된다.
- `user` : `user:anne` , `workspace:auth0` , `folder:planning#editor`
- `relation` : `editor` , `member` , `parent_workspace`
- `object` : `repo:auth0/express_jwt` , `domain:auth0.com` , `channel:marketing`
- `condition` (optional) : `{"condition" : "in_allowed_ip_range", "context" : {...}}`

<br/>

authorization model은 relationship tuple와 함께 user와 object 간의 관계가 있는지 여부를 결정한다. relationship tuple은 다음과 같이 작성된다.

```json
[{
  "user" : "user:anne",
  "relation" : "editor",
  "object" : "document:new-roadmap"
}]
```

<br/>

### Conditional Relationship Tuple
---

conditional relationship tuple은 condition의 결과에 따라 조건이 결정되는 relationship을 나타내는 relationship tuple 이다.

relationship tuple 조건이 있는 경우, relationship tuple이 허용되려면 해당 조건이 true이여만 한다.

<br/>

다음 relationship tuple은 `less_than_hundred` 에 따라 조건이 정해지므로 conditional relationship tuple이다. `less_than_hundred` 표현식이 `x < 100` 으로 정의되면, relationship은 20 < 100 과 같이 true 값을 반환해야 허가된다.

```json
[{
  "user" : "user:anne",
  "relation" : "editor",
  "object" : "document:new-roadmap",
  "condition" : {
    "name" : "less_than_hundred",
    "content" : {
      "x" : 20
    }
  }
}]
```

<br/>

### Relationship
---

relationship은 사용자와 객체 간의 관계가 실현되는 것을 말한다.

<br/>

authorization model은 relationship tuples와 함께 사용자와 객체 간의 relationship을 결정한다. relationship은 직접적이거나 묵시적일 수 있다.

<br/>

### Direct And Implied Relationships
---

user X와 object Y 간의 direct relationship은 relationship tuple (user=X, relation=R, object=Y)이 존재하, 해당 관계에 대한 OpenFGA authorization model은 direct relationship type 제한으로 인해 직접 관계를 허용한다는 의미이다.

user X와 object Y 사이의 relationship(R)이 존재한다는 것은 사용자 X가 객체 Y와 직접적 또는 묵시적 관계에 있는 객체 Z와 관련되어 있고 OpenFGA 권한 부여 모델이 이를 허용하는 경우이다.

user X가 object Y와 직접적 또는 암시적 관계에 있는 객체 Z와 관련이 있고 OpenFGA authorization model이 이를 허용하는 경우 user X와 object Y 사이에 암시적(또는 계산된) 관계(R)가 존재한다.

<br/>

`user:anne` 은 type definition에서 direct relationship type 제한으로 허용하고 다음 relationship tuple 중 하나가 존재하는 경우 `document:new-roadmap` 과 `viewer` 로서 직접 관계를 갖는다.

<br/>


```json
[
  {
    "_description" : "Anne of type user is directly related to the document",
    "user" : "user:anne",
    "relation" : "viewer",
    "object" : "document:new-roadmap"
  }
]
```

```json
[
  {
    "_description" : "Everyone (`*`) of type user is directly related to the document",
    "user" : "user:*",
    "relation" : "viewer",
    "object" : "document:new-roadmap"
  }
]
```

```json
[
  {
    "_description" : "The userset is directly related to the document",
    "user" : "team:product#member",
    "relation" : "viewer",
    "object" : "document:new-roadmap"
  },
  {
    "_description" : "AND Anne of type user is a member of the userset team:product#member",
    "user" : "user:anne",
    "relation" : "member",
    "object" : "team:product#member"
  }
]
```

<br/>

`user:anne` 는 

<br/>

## Reference
---

<https://openfga.dev/docs/fga>