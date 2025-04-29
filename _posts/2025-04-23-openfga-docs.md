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

`user:anne` 은 타입 정의에서 허용하고 관계를 만족하는 타입 튜플이 존재하는 경우 `viewer` 로서 `document:new-roadmap` 과 관계를 갖는다.

<br/>

예를 들어, 다음과 같은 타입 정의를 가정해보자.

```
type document
  relations
    define viewer: [user] or editor
    define editor: [user]
```

<br/>

시스템에 다음 관계 튜플이 존재하는 것을 가정해보자.

```
[{
  "user" : "user:anne",
  "relation" : "editor",
  "object" : "document:new-roadmap"
}]
```

<br/>

이러한 경우에 `viewer` 로서의 `user:anne` 와 `document:new-roadmap` 간의 관계는 `ㄱ` 이 동일한 문서와 맺는 직접적인 `editor` 관계에서 암시된다. 그러므로, 다음과 같은 `user:anne` 와 `document:new-roadmap` 간의 조회 권한이 존재함을 확인하는 `check` 요청은 `true` 를 반환한다.

```java
var options = new ClientCheckOptions()
        .authorizationModelId("01HVMMBCMGZNT3SED4Z17ECXCA");

var body = new ClientCheckRequest()
        .user("user:anne")
        .relation("viewer")
        ._object("document:new-roadmap");

var response = fgaClient.check(body, options).get();

// response.getAllowed() = true 
```

<br/>

### Check Request
---

check request는 사용자가 객체와 특정 관계를 맺고 있는지 여부를 반환하는 OpenFGA check endpoint에 대한 호출이다.

<br/>

check request는 curl을 사용하여 수동으로 확인 엔드포인트를 호출하거나 코드에서 OpenFGA SDK의 `check` 메서드를 사용한다. 이러한 check endpoint는 관계가 존재하면 `{"allowed" : true}` 를, 관계가 존재하지 않으면 `{"allowed" : false}` 를 응답한다.

<br/>

예를 들어, 다음은 user 타입의 `anne` 가 `document:new-roadmap` 에 `viewer` 관계를 갖고 있는지 확인한다.

```kotlin
val options = ClientCheckOptions().authorizationModelId("{modelId}")

val body = ClientCheckRequest()
            .user("user:anne")
            .relation("viewer")
            ._object("document:new-roadmap")

val response = fgaClient.check(body, options).get()

// response.allowed == true
```

<br/>

### List Objects Request
---

list objects request는 사용자가 지정된 관계를 맺고 있는 지정된 타입의 모든 객체를 반환하는 OpenFGA 목록 객체 엔드포인트에 대한 호출이다.

<br/>

list objects 요청은 `listobjects` 메서드를 사용하면 된다. list objects 엔드포인트는 사용자가 특정 관계를 갖는 타입의 객체 목록을 응답한다. 예를 들어, 다음과 같이 `viewer` 관계를 갖는 사용자인 `anne` 의 document 유형의 모든 객체들을 응답한다.

```java
var options = new ClientListObjectsOptions()
        .authorizationModelId("01HVMMBCMGZNT3SED4Z17ECXCA");

var body = new ClientListObjectsRequest()
        .user("user:anne")
        .relation("viewer")
        .type("document");

var response = fgaClient.listObjects(body, options).get();

// response.getObjects() = ["document:otherdoc", "document:planning"]
```

<br/>

### List Users Request
---

list users request는 객체에 대한 특정 관계를 갖는 타입이 주어진 모든 사용자들을 반환하는 OpenFGA list users endpoint 요청이다. 이는 `ListUsers` 메서드를 사용하면 된다. 이는 객체에 대한 특정 관계가 있는 지정된 유형의 모든 사용자들을 반환한다.

<br/>

예를 들어, 다음과 같이 `document:planning` 에 대해 `viewer` 관계를 갖는 사용자 타입의 모든 사용자들을 반환한다.

```java
var options = new ClientListUsersOptions().authorizationModelId("{modelId}");

var userFilters = new ArrayList<UserTypeFilter>() {
    {
        add(new UserTypeFilter().type("user"));
    }
};

var body = new ClientListUsersRequest()
                ._object(new FgaObject().type("document").id("planning"))
                .relation("viewer")
                .userFilters(userFilters);

var response = fgaClient.listUsers(body, options).get();

// response.getUsers() = [{"objects" : {"type" : "user", "id" : "anne"}}]
```

<br/>

### Contextual Tuples
---

contextual tuple는 Check 요청, ListObjects 요청, ListUsers 요청, Expand 요청에 추가할 수 있는 튜플이다. 해당 특정 요청의 컨텍스트 내에서만 존재하며 데이터스토어에 유지되지 않는다. 

<br/>

관계 튜플과 같이 contextual tuple은 사용자, 관계, 객체로 구성된다. 관계 튜플과 다르게 스토어에 저장되지 않는다. 그러나 특정 check 요청의 컨텍스트에서 contextual tuple이 check 요청과 함께 전송되는 경우, 스토어에 저장된 것처럼 처리된다.

<br/>

### Type Bound Public Access
---

OpenFGA에서 type bound public access(`<type>:*` 와 같은)는 관계 튜플의 사용자로 호출될 때 "\[type]의 모든 객체" 를 의미하는 특수 OpenFGA 문법이다. 예를 들어, `user:*` 는 현재 시스템에 존재하지 않는 객체를 포함하여 `user` 타입의 모든 객체를 나타낸다.

<br/>

`document:new-roadmap` 이 공개적으로 쓰기 가능함을 나타내려면(즉, `user` 타입이 모두 편집자임을 나타내려면) 다음 관계 튜플을 추가하면 된다.

```json
[{
    "user" : "user:*",
    "relation" : "editor",
    "object" : "document:new-roadmap"
}]
```

- `<type>:*` 은 `relation` 이나 `object` 속성으로 사용될 수 없다.
- 또한 `<type>:*` 은 튜플의 사용자 속성에서 사용자 집합의 일부로 사용할 수 없다.

<br/>

## Configuration Language
---

OpenFGA의 구성 언어는 시스템의 권한 모델을 표현하며, 이 표현은시스템에 존재하는 객체 타입과 그들 간의 관계를 OpenFGA에 알려준다. 구성 언어는 특정 타입의 객체에 대해 성립할 수 있는 관계를 설명하고, 그 객체와 관계를 맺게 되는 조건들을 나열한다.

구성 언어는 DSL 이나 JSON 문법으로 표현될 수 있다. JSON 문법은 API에서 그대로 사용할 수 있으며, Zanzibar 논문에 나온 언어를 거의 그대로 따른다. DSL은 사용 편의를 위해 JSON 위에 문법적 편의를 추가하지만, OpenFGA의 API로 전송되기 전에 JSON으로 변환된다. JSON 문법은 직접 API를 호출하거나 SDK에 사용되며, DSL은 Playground, CLI, IDE의 확장 프로그램으로 OpenFGA와 상호작용 할 때 사용된다.

<br/>

### Configuration Language Guide
---

다음은 권한 모델 예시이다. 

```
model  
  schema 1.1   
  
type user  
  
type domain  
  relations  
    define member: [user]  
  
type folder  
  relations  
    define can_share: writer  
    define owner: [user, domain#member] or owner from parent_folder  
    define parent_folder: [folder]  
    define viewer: [user, domain#member] or writer or viewer from parent_folder  
    define writer: [user, domain#member] or owner or writer from parent_folder  
      
type document  
  relations  
    define can_share: writer  
    define owner: [user, domain#member] or owner from parent_folder  
    define parent_folder: [folder]  
    define viewer: [user, domain#member] or writer or viewer from parent_folder  
    define writer: [user, domain#member] or owner or writer from parent_folder
```
- 해당 인가 모델은 `user` , `domain` , `folder` , `document` 객체들의 타입을 나타낸다.
- `domain` 은 `member` 에 대한 직접적인 관계만 허용하는 단일 관계를 갖는다.
- `folder` 와 `document` 는 `parent_folder` , `owner` , `writer` , `viewer` , `can_share` 관계들을 갖는다.

<br/>

### Direct Relationship Type Restrictions
---

관계 정의의 `[<string>, <string>, ...]` 은 지정된 타입의 객체에 의한 직접적인 관계를 허용한다. `<string>` 은 다음 세 개의 형태를 갖는다.

- `<type>` : 해당 타입의 객체를 사용자로 연결하는 튜플을 나타낸다. 예를 들어, `group` 이 타입 제한되어 있는 경우 `group:marketing` 을 추가할 수 있다.
- `<type:*>` : 해당 타입의 모든 객체를 연결하는 튜플을 나타낸다. 예를 들어, `user:*` 이 타입 제한되어 있는 경우 `user:*` 추가할 수 있다.
- `<type>#<relation>` : 해당 특정 관계에 의해 해당 타입의 객체와 관련된 사용자 집합이 있는 튜플을 나타낸다. 예를 들어, `group#member` 가 타입 제한된 경우 `group:marketing#member` 를 추가할 수 있다.

<br/>

직접적인 관계 유형 제한이 지정되지 않으면 직접적인 관계가 허용되지 않으며, 이 특정 관계의 다른 객체를 이 타입의 객체와 관련시키는 튜플을 작성할 수 없다.

<br/>

다음은 `team` 타입의 구성이다.

```
type team
  relations
    define member: [user, uesr:*, team#member]
```
- 사용자가 `team` 타입의 객체와 가질 수 있는 모든 관계를 정의한다.
- `[user, team#member]` 와 같이 직접적인 관계가 제한되었기 때문에 시스템 내 사용자는 `team` 타입의 `member` 로써 직접적 관계를 갖을 수 있다.
	- `user` 타입
	- 모든 접근에 대한 `user` 타입 (`user:*`)
	- `team` 타입과 `member` 관계의 사용자 집합 (e.g. `team:product#member`)

<br/>

`anne` 은 다음 관계 튜플 집합 중 하나라도 존재하는 경우 `team:product` 의 `member` 이다.

<br/>

```json
[
  {
    "user" : "user:anne",
    "relation" : "member",
    "object" : "team:product",
    "_description" : "Anne is directly related to teh product team as a member"
  }
]
```

```json
[
  {
    "user" : "user:*",
    "relation" : "member",
    "object" : "team:product",
    "_description" : "Everyone (`*`) is directly related to the product team as a member"
  }
]
```

```json
[
  {
    "user" : "team:contoso#member",
    "relation" : "member",
    "obejct" : "team:product",
    "_description" : "Members of the contoso team are members of the prduct team"
  },
  {
    "user" : "user:anne",
    "relation" : "member",
    "object" : "team:contoso",
    "_description" : "Anne is a member of the contoso team"
  }
]
```

<br/>

### Referencing Other Relations On The Same Object
---

같은 객체들이 다른 관계를 참조할 수도 있다. 다음은 간단한 `document` 타입 정의이다.

```
type document
  relations
    define editor: [user]
    define viewer: [user] or editor
    define can_rename: editor
```

- 사용자가 가질 수 있는 `document` 관련한 모든 관계들을 정의한다.
- `viewer` 와 `can_rename` 관계 정의는 모두 동일한 타입의 또 다른 관계인 `editor` 를 참조한다.
- `can_rename` 은 직접 관계 타입 제한을 참조하지 않는다. 즉, 사용자에게 이 관계를 직접 할당할 수 없으며 `editor` 관계가 할당될 때 상속되어야 한다.
- `viewer` 관계는 직접적이거나 직접적이지 않는 관계 둘다 허용한다.

<br/>

다음 튜플 집합 중에 하나라도 존재할 경우, `anne` 은 `document:new-roadmap` 의 `viewer` 이다.

```json
[
  {
    "user" : "user:anne",
    "relation" : "editor",
    "object" : "document:new-roadmap",
    "_description" : "Anne is an editor of the new-roadmap document"
  }
]
```

```json
[
  {
    "user" : "user:anne",
    "relation" : "viewer",
    "object" : "document:new-roadmap",
    "_description" : "Anne is a viewer of the new-roadmap document"
  }
]
```

<br/>

`anne` 은 document에 대해 `editor` 관계를 갖고 있는 경우에만 `document:new-roadmap` 의 `can_rename` 관계를 갖는다.

```json
[
  {
    "user" : "user:anne",
    "relation" : "editor",
    "object" : "document:new-roadmap",
    "_description" : "Anne is an editor of the new-roadmap document"    
  }
]
```

<br/>

### Referencing Relations On Related Objects
---

간접적인 관계의 또 다른 집합은 다른 객체와의 관계를 참조함으로써 가능해진다.

문법은 `X from Y` 이며, 다음이 요구된다.
- 다른 객체는 현재 객체와 `Y` 로 관련된다.
- 사용자는 `X` 로 다른 객체와 관련되어 있다.

아래 인가 모델을 참조해라.

```
model
  schema 1.1

type user

type folder
  relations
    define viewer: [user, folder#viewer]

type document
  relations
    define parent_folder: [folder]
    define viewer: [user] or viewer from parent_folder
```

<br/>

아래 코드는 `viewer` 관계에 직접 할당된 모든 사용자와 `document` 의 `parent_folder` 를 볼 수 있는 모든 사용자라는 것을 나타낸다.

```
type document
  relations
    define viewer: [user] or viewer from parent_folder
```

<br/>

다음과 같은 관계 튜플 중에 하나라도 존재할 경우, `user:anne` 은 `document:new-roadmap` 의 `viewer` 이다.

```json
[
  {
    "user" : "folder:planning",
    "relation" : "parent_folder",
    "object" : "document:new-roadmap",
    "_description" : "planning folder is the parent folder of the new-roadmap document"
  },
  {
    "user" : "user:anne",
    "relation" : "viewer",
    "object" : "folder:planning",
    "_description" : "anne is a viewer of the planning folder"
  }
]
```

<br/>

관련 객체에 대한 참조 관계는 전이적인 암시적 관계를 정의한다. 사용자 A가 viewer 로서의 객체 B가 관련 되어 있고 객체 B가 parent 로서 객체 C와 관련되어 있으면, 사용자 A는 viewer로서 객체 C가 관련 되어 있다. 이는 폴더를 보는 사람이 폴더의 모든 문서를 보는 사람이라는 것을 나타낼 수 있다.

<br/>

OpenFGA는 참조된 관계가 다른 관계를 참조(`from`)하는 것을 허용하지 않으며 타입 제한에서 비구체적 타입(`<object_type>:*`) 또는 사용자 집합(`<object_type>#<relation>`)을 허용하지 않는다.

<br/>

### Union Operator
---

DSL에서 `or` 나 JSON에서의 `union` 은 사용자가 사용자 집합 중에 하나에 속하는 경우 관계가 존재함을 나타낸다.

```
type document
  relations
    define viewer: [user] or editor
```

<br/>

`user:anne` 는 `document:new-roadmap` 의 `viewer` 이다.

```json
[{
  "user" : "user:anne",
  "relation" : "editor",
  "object" : "document:new-roadmap"
}]
```

```json
[{
  "user" : "user:anne",
  "relation" : "viewer",
  "object" : "document:new-roadmap"
}]
```

<br/>

### Intersection Operator
---

DSL의 `and` 나 JSON의 `intersection` 은 사용자가 모든 사용자 집합에 포함되어 있는 경우 관계까 존재함을 나타낸다.

```
type document
  relations
    define viewer: authorized_user and editor
```

<br/>

다음을 모두 만족할 경우 `user:anne` 은 `document-new-roadmap` 의 `viewer`  이다.

```json
[{
  "user" : "user:anne",
  "relation" : "editor",
  "object" : "document:new_roadmap"
}]
```

```json
[{
  "user" : "user:anne",
  "relation" : "authorized_user",
  "object" : "document:new-roadmap"
}]
```

<br/>

### Exclusion Operator
---

DSL에서 `but not` 이나 JSON에서 `difference` 는 사용자가 기본 사용자 집합에 있지만 제외도니 사용자 집합에 없는 경우 관계까 존재함을 나타낸다. 이 연산자는 제외 또는 차단 목록을 모델링할 때 특히 유용하다.

```
type document
  relations
    define viewer: [user] but not blocked
```

<br/>

다음을 만족할 경우에만 `user:anne` 은 `document:new-roadmap` 의 `viewer` 이다.

```json
[{
  "user" : "user:anne",
  "relation" : "viewer",
  "object" : "document:new-roadmap"
}]
```

AND 아래 튜플이 존재하지 않을 경우

```json
[{
  "user" : "user:anne",
  "relation" : "blocked",
  "object" : "document:new-roadmap"
}]
```

<br/>

### Grouping and nesting operators
---

괄호를 사용하여 연산자를 그룹화하고 중첩하면 복잡한 조건을 정의할 수 있다. 직접적인 관계는 괄호를 사용하여 표현식에 포함될 수 있다.

```
type user

type organization
  relations
    define member: [user]

type folder
  relations
    define organization: [organization]
    define parent: [folder]
    define viewer: ([user] or viewer from parent) and member from organization
```

<br/>

## Modeling Guildes
---

### Introduction To Modeling
---

OpenFGA는 빠르고 안정적으로 권한 확인을 수행하도록 설계되었다. 즉, 사용자 U가 객체 O에 대해 작업 A를 수행할 수 있는가? 라는 요청에 응답을 보낸다.

ReBAC 시스템은 사용자와 객체의 관계에 따른 액세스를 결정한다.
- 일반적인 인가 확인 : U가 O에 대해 A(Action)를 할 수 있는가?
- OpenFGA 인가 확인 : U가 O에 대한 R(Relation)를 갖고 있는가?

<br/>

작업 A에 대한 권한을 의마흔 관계 R을 정의해야 한다.
- 일반 : Jane 사용자는 Project SandCastle 객체를 볼 수 있는가?
- OpenFGA : Jane 사용자는 Project SandCastle 객체를 볼 관계를 갖는가?

<br/>

### A Process For Defining Authorization Models
---

인가 모델을 정의하려면 시스템의 모든 사용 사례 또는 작업에 대해 "사용자 U가 객체 O에 대해 작업 A를 수행할 수 있는 이유는 무엇인가?" 라는 질문에 대한 답변을 체계화해야 한다.

Google Drive와 같은 시스템을 예시로 인가 모델을 정의하는 과정에 대해 살펴보자.

인가 모델을 정의하는 과정은 다음과 같다.

1. 중요 기능을 선택한다.
2. 객체 타입들을 나열한다.
3. 해당 타입에 대한 관계를 나열한다.
4. 관계들을 정의한다.
5. 모델을 검증한다.
6. 반복한다.

![openfga1](/assets/img/openfga-process1.png)

<br/>

#### 1. Pick The Most Import Feature
---

![openfga2](/assets/img/openfga-process2.png)

가장 중요한 기능을 선택하고 기능을 인가 모델로 묘사할 수 있도록 기능 설명을 정리한다. 기능 설명은 다음과 같이 시스템의 객체, 사용자, 사용자들의 집합을 포함해야 한다.

```
만약 {conditions} 할 경우에 사용자{user}는 {object types}에 대해 작업{action}을 수행할 수 있다.
```
- e.g. A user can create a document in a drive if they are the owner of the drive.

<br/>

#### 2. List The Object Types
---

![openfga3](/assets/img/openfga-process3.png)

다음으로 시스템에 있는 객체 타입 목록을 작성한 후 정리된 기능 설명에서 해당 객체들을 찾는다.

```
A user {user} can perform action {action} to/on/in {object type} ... IF {conditions}
```
- e.g. Document, Folder, Organization
- e.g. A user can create a `document` in a drive if they are the owner of the drive.

<br/>

그 다음으로 conditions에 나타나는 명사를 찾는다.
- e.g. A user can create a `document` in a drive if they are the `owner of the drive`.

객체 목록 유형에 없는 명사를 찾아서 추가해준다.
- e.g. Document, Folder, Organization, User, Drive

<br/>

이제 객체 타입 목록이 있으므로 OpenFGA에 정의할 수 있다.

```
model
  schema 1.1

type user

type document

type folder

type organization

type drive
```

<br/>

#### 3. List Relations For Those Types
---

![openfga4](/assets/img/openfga-process4.png)

{type}에 대한 관계는 다음과 같다.

- 조건식에 존재하는 명사는 데이터베이스의 외래 키이다.
- `~ 할 수 있는` 것은 타입에 대한 권한이다.
- e.g. A user `can create a document in a drive` if they are the `owner of the drive`.
	- Document
		- parent, can_share, owner, editor, can_write, can_view, viewer, can_change_owner
	- Folder
		- can_create_document, owner, can_create_folder, can_view, viewer, parent
	- Organization
		- member
	- Drive
		- can_create_document
		- owner
		- can_create_folder

<br/>

이는 다음과 같이 OpenFGA 언어로 표현할 수 있다.

```
model
  schema 1.1

type user

type document
  relations
    define parent:
    define owner:
    define editor:
    define viewer:
    define can_share:
    define can_view:
    define can_write:
    define can_change_onwer:

type folder
  relations
    define owner:
    define parent:
    define viewer:
    define can_create_folder:
    define can_create_document:
    define can_view:

type organization
  relations
    define member:

type drive
  relations
    define owner:
    define can_create_document:
    define can_create_folder:
```

<br/>

#### 4. Define Relations
---

![openfga5](/assets/img/openfga-process5.png)

각 관계에 대해 정의해보자.

<br/>

**Type: Organization**

Relation: Member

```
type organization
  relations
    define member: [user, organization#member]
```
- 조직에는 회원이 있으므로 `user` 가 포함된다.
	- `{ user: "user:{user-id}", relation: "member", object: "organization:{id}" }`
- 다음과 같은 관계를 표현할 수 있다.
	- `{ user: "organization:A#member", relation: "member", object: "organization:B" }`

<br/>

**Type: Document**

Relation: Owner

```
type document
  relations
    define owner: [user, organization#member]
```
- document는 한 명이나 그 이상의 owner 가 있을 수 있다.
- document의 owner는 다음 형태의 튜플 생성을 통해 할당할 수 있다.
	- `{ user: "{user_id}", relation: "owner", object: "document:{id}" }

<br/>

Relation: Editor

```
type document
  relations
    define editor: [user, organization#member]
```

- document는 editor가 있을 수 있다.
- document의 editor는 다음 형태의 튜플 생성을 통해 할당할 수 있다.
	- `{ user: "{user-id}", relation: "editor", object: "document:{id}" }`

<br/>

Relation: Viewer

```
type document
  relations
    define viewer: [user, organization#member]
```

<br/>

Relation: Parent

```
type document
  relations
    define parent: [folder, drive]
```

- document는 parent가 있을 수 있다.
	- `{ user: "forlder:{id}", relation: "parent", object: "document:{id}" }`
	- `{ user: "drive:{id}", relation: "parent", object: "document:{id}" ]`

<br/>

Relation : can_share

```
type document
  relations
    define can_share: owner or editor or owner from parent
```

- 직접 할당할 수 없는 권한을 나타낸다.

<br/>

Relation : can_view

```
type document
  relations
    define can_view: viewer or editor or owner or viwer from parent or owner from parent
```

<br/>

Relation : can_write

```
type document
  relations
    define can_write: editor or owner or owner from parent
```

<br/>

Relation : can_change_owner

```
type document
  relations
    define can_change_owner: owner
```

<br/>

**Complete Type Definition**

```
model
  schema 1.1

type user

type organization
  relations
    define member: [user, organization#member]

type document
  relations
    define owner: [user, organization#member]
    define editor: [user, organization#member]
    define viewer: [user, organization#member]
    define parent: [folder]
    define can_share: owner or editor or owner from parent
    define can_view: viewer or editor or owner from parent or editor from parent or owner from parent
    define can_write: editor or owner or owner from parent
    define can_change_owner: owner
```

<br/>

#### 5. Test The Model
---

![openfga6](/assets/img/openfga-process6.png)

1. 튜플을 추가한다.
2. Assertions를 통해 모델이 잘 동작하는지 검증한다.

<br/>

### Direct Access
---

**Direct Access 모델링은 언제 사용하는지?**

관계 튜플을 사용한 액세스 권한 부여는 OpenFGA의 핵심 부분이다. 관계 튜플이 존재하지 않으면 어떤 `checks` 요청이든 실패할 것이다. 다음과 같이 사용해야 한다.

- 인가 모델을 사용하여 시스템에서 사용자와 객체 간에 가능한 관계를 나타낸다.
- 관계 튜플을 사용하여 시스템에서 사용자와 객체 간의 관계에 대한 사실을 표현한다.

<br/>

#### 1. Create A Relationship Tuple
---

다음과 같이 `bob` 이 `document:meeting_notes.doc` 에 대한 `editor` 권한을 갖는다는 관계 튜플을 추가해보자.

```java
var options = new ClientWriteOptions()
  .authorizationModelId("{modelId}");

var body = new ClientWriteRequest()
  .writes(List.of(
    new ClientTupleKey()
      .user("user:bob")
      .relation("editor")
      ._object("document:meeting_notes.doc")
  ));

var response = fgaClient.write(body, options).get();
```

<br/>

#### 2. Check That The Relationship Exists
---

위와 같이 튜플을 추가한 후, `bob` 이 `document:meeting_notes.doc` 에 `editor` 인지에 대한 관계를 확인함으로써 관계가 유효한지 확인할 수 있다.

```java
var options = new ClientCheckOptions()
  .authorizationModelId("{modelId}");

var body = new ClientCheckRequest()
  .user("user:bob")
  .relation("editor")
  ._object("document:meeting_notest.doc");

var response = fgaClient.check(body, options).get();

// response.getAllowed() = true
```

<br/>

`bob` 이 `doucment:meeting_notest.doc` 의 `viewer` 가 인지 확인할 경우 `false` 가 응답된다. OpenFGA는 아직 해당 튜플을 알지 못 하기 때문이다.

```java
var options = new ClientCheckOptions()
  .authorizationModelId("{modelId}")

var body = new ClientCheckRequest()
  .user("user:bob")
  .relation("viewer")
  ._object("document:meeting_notes.doc");

var response = fgaClient.check(body, options).get();
```

OpenFGA에서 관계 튜플을 생성할 때, 각 객체와 사용자의 식별자는 고유한 것을 사용해라. 위에서 사용하는 이름들과 식별자는 쉬운 예시를 위한 것이다.

<br/>

### User Groups
---

User Groups 모델링이란 사용자를 그룹에 추가하고 객체에 그룹 접근 권한을 부여하는 방법이다.

<br/>

**User Groups 모델링을 언제 사용하는지?**

관계 튜플들은 객체와 관계를 갖는 모든 그룹이 명시될 수 있으므로, 객체와 동일한 관계를 가진 사용자 집합을 포함하고자 할 때 유용하다. 예시는 다음과 같다.

- `roadmap.doc` 에서 `engineers` 그룹에  `viewer` 접근 권한 부여
- `document` 에 접근할 수 없는 `members` 의 `block_list` 생성
- `team` 과 `document` 를 공유
- `followers` 에게만 `photo` 에 대한 `viewer` 접근 권한 부여
- `organization` 에 속한 모든 `users` 가 `file` 을 볼 수 있도록 함
- 특정 `locale` 의 `user` 에 대한 접근 제한

<br/>

#### 01. Introduce the concept of a team to the authorization model
---

먼저 인가 모델에 `team` 객체를 정의한다. `team` 은 `member` 들을 가질 수 있으며, 다음과 같이 인가 모델을 작성하면 된다.

```
model
  schema 1.1

type user

type document
  relations
    define editor: [team#member]

type team
  relations
    define member: [user]
```

<br/>

#### 02. Add users as members to the team
---

이제 `team` 의 `member` 인 user를 할당할 수 있다. `user:alice` 가 `team:writers` 의 member 임을 나타내는 관계 튜플을 생성해보자.

```java
var options = new ClientWriteOptions()
        .authorizationModelId("{modelId}");

var body = new ClientWriteRequest()
        .writes(List.of(
            new ClientTupleKey()
                    .user("user:alice")
                    .relation("member")
                    ._object("team:writers")
        ));

var response = fgaClient.write(body, options).get();
```

<br/>

#### 03. Assign the team members a relation to an object
---

그룹을 표현하려면 `type:object_id#relation` 포맷을 사용하면 된다. 이는 사용자 집합이 `type:object_id` 에 관계가 있음을 나타낸다. 예를 들어, `team:writers#members` 는 `members` 로써 `team:writers` 객체에 관계가 있는 사용자 집합을 나타낸다.

`team` 의 `members` 에 `document` 에 대한 관계를 할당하기 위해, 다음과 같이 `team:writers` 의 members가 `document:meeting_notes.doc` 의 `editor` 임을 나타내는 관계 튜플을 생성해라.

```java
// options 생략

var body = new ClientWriteRequest()
        .writes(List.of(
                new ClientTupleKey()
                        .user("team:writers#member")
                        .relation("editor")
                        ._object("document:meeting_notes.doc")
        ));

// response 생략
```

- `alice` 는 `team:writers` 의 `member` 이다.
- `team:writers` 의 `members` 는 `document:meeting_notes` 의 `editor` 이다.
- 그러므로, `alice` 는 `document:meeting_notes` 의 `editor` 이다.

<br/>

### Roles and Permissions
---

- 역할은 사용자나 사용자 그룹으로 할당된다. 사용자는 `editor` 나 `owner` 와 같은 하나 이상의 역할을 가질 수 있다.
- 권한은 사용자가 `device_renamer` 나 `channel_archiver` 같은 특정 역할에 따라 특정 객체들에 접근할 수 있도록 허가한다.

예를 들어, `trip` 의 `viewer` 역할은 예약을 볼 수 있는 권한이 있고 `owners` 역할은 예약을 추가하거나 볼 수 있는 권한이 있을 수 있다.

<br/>

**Roles and Permissions model은 언제 사용하는가?**

이 모델은 직접 사용자에게 역할을 부여하거나 사용자가 다른 관계에서 다운스트림으로 받는 관계를 통해 권한을 할당할 수 있다. 예시는 다음과 같다.

- `document` 를 수정하고 읽을 수 있는 `admin` 역할을 누군가에게 부여한다.
- `device` 에서 `live_video_viewer`  할 수 있는 `security_guard` 역할을 누군가에게 부여한다.
- `shop` 에서 `view_products` 할 수 있는 `viewer` 역할을 누군가에게 부여한다.

역할 및 권한 모델을 구현하면 기존 역할에 더 세분화된 권한을 부여할 수 있으므로 애플리케이션에서 특정 사용자 역할을 명시적으로 확인하지 않고도 특정 객체에 대한 접근 권한이 있는지 여부를 확인할 수 있다. 또한, 애플리케이션 동작에 영향을 주지 않고 새 역할/권한을 추가하거나 역할을 통합할 수 있다. 예를 들어, 앱의 `check` 가 `check('bob', 'owner', 'trip:Europe')` 대신 `check('bob', 'booking_addr', 'trip:Europe')` 과 같이 세부 권한에 대한 것인데 나중에 `onwers` 가 더 이상 `trip` 에 예약을 추가할 수 없다고 결정하면, 앱에서 코드 수정 없이 `trip` 에 대한 관계를 제거할 경우 모든 권한이 자동으로 변경 사항을 반영하게 된다.

<br/>

아래의 Roles and Permission 예시는 `owners` 및/또는 `viewer` 가 있는 여행 예약 시스템으로, 둘 다 여행에 예약을 추가하거나 여행의 예약을 조회하는 등 보다 세분화된 권한을 가질 수 있다. 이를 OpenFGA 환경에 구성하려면 다음과 같이 수행해야 한다.

1. 여행 예약 시스템에서 역할이 직접적인 관계와 어떻게 연관되어 있는지 이해한다.
2. 기존 인가 모델에 암시적 관계를 추가하여 예약에 대한 권한을 정의한다.
3. 직접적이고 암시적인 관계에 대해 관계 튜플을 기반으로 사용자 역할과 권한을 확인한다.

<br/>

#### 01. Understand how roles work within the trip booking system
---

역할은 사용자에게 직접적으로 할당된 관계이다. 아래에서 특정 사용자에게 할당할 수 있는 명시된 역할은 `owner` 와 `viewer` 이다.

```
model
  schema 1.1

type user

type trip
  relations
    define owner: [user]
    define viewer: [user]
```

<br/>

#### 02. Add permission for bookings
---

권한은 사용자가 다른 관계를 통해 얻을 수 있는 관계이다. 권한을 표현할 때 인가 모델의 관계에 직접적인 관계 타입 제한을 추가하는 것을 피하기 위해, 대신 다른 관계를 통해 해당 관계를 정의한다. 이는 다른 관계로부터 부여되고 암시되는 권한임을 나타낸다.

예약과 관련된 권한을 추가하려면, `trip` 객체 타입에 사용자가 `trips` 에 대해 수행할 수 있는 다양한 동작(ex. 조회, 수정, 삭제, 이름 변경)을 나타내는 새로운 관계를 추가해라.

예약을 조회할 권한을 가진 `trip` 의 `viewer` 와 예약을 추가/조회할 권한을 가진 `owners` 를 허가하려면 다음 타입을 변경해라.

`trip` 의 `viewer` 가 예약을 조회할 수 있는 권한을 갖고, `owner` 가 예약을 추가하고 조회할 수 있는 권한을 갖도록 하려면, 타입을 수정해야 한다.

<br/>

```
model
  schema 1.1

type user

type trip
  relations
    define owner: [user]
    define viewer: [user]
    define booking_adder: owner
    define booking_viewer: viewer or owner
```

- `booking_viewer` 와 `booking_adder` 둘 다 직접적인 관계 타입 제한이 없으며, 이로 인해 해당 관계는 역할을 통해서만 할당되고 직접 할당될 수 없도록 보장된다.

<br/>

#### 03. Check user roles and their permissions
---

타입 정의는 예약을 어떻게 조회하거나 추가할 수 있는지에 대한 역할과 권한을 반영한다. 따라서 관계 튜플을 생성하여 사용자에게 역할을 할당하고, 사용자가 적절한 권한을 가지고 있는지 확인할 수 있다.

<br/>

두 관계 튜플을 생성하라
1. `bob` 에게 유렵에 대한 `trip` 의 `viewer` 역할을 준다.
2. `alice` 에게 유럽에 대한 `trip` 의 `owner` 역할을 준다.

```java
// options 생략

var body = new ClientWriteRequest()
        .writes(List.of(
                new ClientTupleKey()
                        .user("user:bob")
                        .relation("viewer")
                        ._object("trip:Europe"),
                new ClientTupleKey()
                        .user("user:alice")
                        .relation("owner")
                        ._object("trip:Europe")
        ));

// response 생략
```

<br/>

`bob`이 유럽에 대한 `trip` 예약을 조회할 수 있는지 확인해라.

```java
// options 생략

var body = new ClientCheckRequest()
        .user("user:bob")
        .relation("booking_viewer")
        ._object("trip:Europe");

var response = fgaClient.check(body, options).get();

// response.getAllowed() = true
```

`bob` 은 `booking_viewer` 이며, 이유는 다음과 같다.

1. `bob` 은 유럽에 대한 `trip` 의 `viewer` 이다.
2. `trip:Europe` 객체에 `viewer` 로 연결된 모든 사용자는 `booking_viewer` 로도 연결된다.
3. 그러므로, 특정 `trip`  모든 `viewer` 는 `booking_viewers` 가 된다.

<br/>

`bob` 이 `trip:Europe` 을 추가할 수 없는지 확인하려면 다음과 같이 요청하면 된다.

```java
// option 생략

var body = new ClientCheckRequest()
        .user("user:bob")
        .relation("booking_adder")
        ._object("trip:Europe");

var response = fgaClient.check(body, options).get();

// response.getAllowed() = false
```

<br/>

`alice` 가 예약을 조회하고 추가할 수 있는지 확인하려면 다음과 같이 요청하면 된다.

```java
// options 생략

var body = new ClientCheckRequest()
        .user("user:alice")
        .relation("booking_viewer")
        ._object("trip:Europe");

var response = fgaClient.check(body, options).get();

// response.getAllowed() = true
```

```java
// options 생략

var body = new ClientCheckRequest()
        .user("user:alice")
        .relation("booking_adder")
        ._object("trip:Europe");

var response = fgaClient.check(body, options).get();

// response.getAllowed() = true
```

<br/>

`alice` 는 `booking_viewer` 이고 `booking_adder` 이다. 이유는 다음과 같다.

1. `alice` 는 `trip:Europe` 에 대한 `owner` 이다.
2. `trip:Europe` 객체에 `onwer` 로 연결된 모든 사용자는 `booking_viewer` 로도 연결된다.
3. `trip:Europe` 객체에 `onwer` 로 연결된 모든 사용자는 `booking_adder` 로도 연결된다.
4. 그러므로, 특정 `trip` 에 대한 모든 `owners` 는 해당 `trip` 에 대한 `booking_viewers` 이면서 `booking_adders` 이다.

<br/>

### Parent-Child Objects
---

사용자가 하나의 객체와 맺는 관계가 다른 객체와읭 관계에도 영향을 미칠 수 있다. 예를 들어, `folder` 의 `editor` 는 해당 `folder` 가 `parent` 인 모든 `documents` 의 `editor` 가 될 수 있다.

<br/>

**Parent-Child 모델링은 언제 사용하는가?**

객체 간의 관계는 구성된 권한 모델과 결합되어, 사용자가 하나의 객체와 맺은 관계가 다른 객체와의 관계에도 영향을 미칠 수 있음을 나타낼 수 있다. 또한, 사용자 그룹을 이용해 객체 간의 관계를 수정할 필요성을 없앨 수도 있다.

다음은 객체 간의 관계에 대한 간단한 예시이다.

- `employee` 의 `managers` 는 `employee` 가 제출한 요청들을 `approve` 할 수 있는 권한을 갖는다.
- 조직 내에서 저장소 관리자 역할(`repo_admin`)을 가진 사용자는 그 조직에 있는 모든 저장소들에 대해 자동으로 `admin` 접근 권한을 갖는다.
- `plan` 을 `subscribed` 한 사용자는 `plan` 의 모든 `features` 에 대해 접근 권한을 갖는다.

<br/>

다음 예시에서는 문서들을 포함하는 폴더(a)와 특정 폴더에 접근 권한을 가진 사용자(b)가 해당 폴더 내 모든 문서에 대해서도 편집자 접근 권한을 갖는 경우를 모델링한다.

`folder` 의 `editor` 가 포함된 `document` 의 `editors` 가 되도록 하려면, 다음 작업을 수행해야 한다.

1. `folder` 와 `document` 사이의 `parent` 관계를 허용하도록 인가 모델을 수정해라.
2. `folder` 로부터 권한이 연속되도록 `document` 타입 정의의 `editor` 관계를 수정해라.

<br/>

다음 세 단계는 `bob` 이 `folder:notes` 의 `editor` 이므로 `bob` 이 `document:meeting_notes.doc` 의 `editor` 이라는 것을 나타내고 검증한다.

3. `bob` 이 `folder:notes` 의 `editor` 를 나타내는 관계 튜플을 생성한다.
4. `folder:nots` 가 `document:meeting_notes.doc` 의 `parent` 인 것을 나타내는 관계 튜플을 생성한다.
5. `bob` 이 `document:meeting_notes.doc` 의 `editor` 인지 확인한다.

<br/>

#### 1. Update the Authorization Model to allow a parent relationship between folder and document
---

다음 권한 모델 업데이트는 `folder` 와 `document` 간의 `parent` 관계를 허용한다.

```
model
  schema 1.1

type user

type folder
  relations
    define editor: [user]

type document
  relations
    define parent: [folder]
    define editor: [user]
```

- `document` 가 `parent` 관계를 갖는 것은 다른 객체들이 `document` 의 `parent` 가 될 수 있음을 나타낸다.

<br/>

#### 2. Update the editor relation in the document type definition to support cascading from folder
---

`folder` 와 `document` 간의 연속적인 관계를 허용하도록 인가 모델을 수정해라.

```
model
  schema 1.1

type user

type folder
  relations
    define editor: [user]

type document
  relations
    define parent: [folder]
    define editor: [user] or editor from parent
```

- `document` 의 `editor` 는 다음과 같이 될 수 있다.
    - 직접 `editor` 로 할당된 사용자들
    - `document` 의 `parent` 객체 중 하나에 `editor` 로 연결된 사용자들

이처럼 변경한 후, `editor` 로서 `document` 의 `parent` 인 `folder` 와 연관된 것은 `document` 의 `editor` 이다.

<br/>

#### 3. Create a new relationship tuple to indicate that `bob` is an `editor` of `folder:notes`
---

새로운 연속 관계를 활용하기 위해, `bob` 이 `folder:notes` 의 `editor` 를 나타내는 관계 튜플을 생성해라.

```java
// option 생략

var body = new ClientWriteRequest()
        .writes(List.of(
                new ClientTupleKey()
                        .user("user:bob")
                        .relation("editor")
                        ._object("folder:notes")
        ));

var response = fgaClient.write(body, options).get();
```

<br/>

#### 4. Create a new relationship tuple to indicate that `folder:notes` is a `parent` of `document:meeting_notes.doc`
---

현재는 `bob` 이 `folder:notes` 의 `editor` 이며, `folder:notes` 가 `document:meeting_notes.doc` 의 `parent` 를 나타내도록 해보자.

```java
// options 생략

var body = new ClientWriteRequest()
        .writes(List.of(
                new ClientTupleKey()
                        .user("folder:notes")
                        .relation("parent")
                        ._object("document:meeting_notes.doc")
        ));

var response = fgaClient.write(body, options).get();
```

<br/>

#### 5. Check if `bob` is an `editor` of `document:meeting_notes.doc`
---

인가 모델을 변경하고 두 관계 튜플을 추가한 이후, 다음과 같이 `bob` 이 `document:meeting_notes.doc` 의 `editor` 인지 확인하여 설정한 것이 올바른지 검증해라.

```java
// options 생략

var body = new ClientCheckRequest()
        .user("user:bob")
        .relation("editor")
        ._object("document:meeting_notest.doc");

var response = fgaClient.check(body, options).get();

// response.getAllowed() = true
```


<br/>

## Reference
---

<https://openfga.dev/docs/fga>