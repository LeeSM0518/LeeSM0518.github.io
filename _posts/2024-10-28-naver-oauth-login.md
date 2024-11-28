---
title: Naver OAuth Login
date: 2024-10-28 09:00:00 +0900
categories: oauth
tags:
  - oauth
  - docs
description: 네이버 OAuth 로그인을 연동하기 위해 가이드 문서 정리
mermaid: "true"
---

## 1. 개요
---

- 동의한 사용자 정보 조회
    - 이름, 이메일, 별명, 생일, 연령대, 성별 등의 사용자 정보 조회 가능
- 간편한 운영
    - 환경별 로그인, 누적 사용자 통계 제공

<br/>

## 2. 네이버 로그인 서비스 소개
---

### 2.1. 제공하는 기능
---
- 소셜 로그인
- 로그인 연동 회원 프로필 조회
    - 이름, 이메일, 별명, 프로필 사진, 생일, 연령대, 성별 정보
- 네이버의 로그인 오픈 API 이용 (네이버 서비스 이용)
- 서비스 이용 통계 제공
- 네이버 로그인 뱃지

<br/>

## 3. 네이버 로그인 적용
---

### 3.1. 네이버 로그인 적용 전 점검 사항
---

#### 3.1.1. 애플리케이션 등록
---

먼저 ‘Application-애플리케이션 등록’ 메뉴에서 애플리케이션을 등록해야 한다.

[_→ 애플리케이션 등록 가이드_](https://developers.naver.com/docs/common/openapiguide/appregister.md)

#### 3.1.2. 필수 항목 확인
---

1. 애플리케이션 이름은 명확하고 간결하게 작성
2. 애플리케이션 이름은 “의미를 알수 없는 문자” 또는 “서비스와 관련없는 이름” 사용하지 않는다.
3. 로고 이미지는 규격을 준수하며 서비스를 대표할 수 있는 이미지로 설정
4. 사용 API에 “네이버 로그인”을 반드시 선택해야 한다. 로그인 오픈 API 서비스 환경은 반드시 1개 이상 선택한다.
5. 웹 서비스 환경에서는 서비스의 대표 URL이 입력되어야 한다.
6. 애플리케이션 환경에서는 AppScheme와 package name을 정확하게 입력이 되어야 한다.

<br/>

#### 3.1.3. 사전 검수 요청
---

실제 서비스에 적용하고자 한다면 애플리케이션 검수 요청을 등록해야 한다.

[_→ 검수 가이드_](https://developers.naver.com/docs/login/verify/verify.md)

<br/>

### 3.2. 네이버 로그인 버튼 적용 가이드
---

- 네이버 로그인 SDK를 이용해 개발
- 네이버 로그인 버튼 이미지
    [_→ 로그인 버튼 사용 가이드_](https://developers.naver.com/docs/login/bi/bi.md)

<br/>

### 3.3. 회원 정보 확인 및 기존 회원과의 연동
---

#### 3.3.1. 네이버 로그인 회원의 프로필 정보
---

네이버 로그인 성공시 접근 토큰을 얻는다.

접근 토큰을 이용하여 사용자 정보를 조회할 수 있다.

- 사용자 정보
    - 이용자 식별자 : 64자 이내로 구성된 BASE64 형식의 문자열
        - (2021년 5월 1일 이후 생성된 애플리케이션부터 적용. 기존 INT64 규격의 숫자)
    - 이름 : 10자 이내로 구성된 문자열
    - 닉네임 : 20자 이내로 구성된 문자열
    - 프로필 이미지 : 255자 이내로 구성된 URL 형태의 문자열
    - 연락처 이메일 주소 : 이메일 규격의 문자열
    - 생일 : 월-일 (MM-DD) 형태의 문자열
    - 연령대 : 연령 구간에 따라 0-9 / 10-19 / 20-29 / 30-39 / 40-49 / 50-59 / 60- 으로 표현된 문자열
    - 성별 : M/F (남성/여성) 으로 표된 문자
    - 출생연도 : 연(YYYY) 형태의 문자열
    - 휴대전화번호 : 대쉬(-)를 포함한 휴대전화번호 문자열

각 항목별로 필수 제공 항목과 추가 제공 항목 중에 선택할 수 있다.

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252FNYNJPpJUek8Ozr3HD4xj%252Fimage.png%3Falt%3Dmedia%26token%3Dbce06c73-18a6-468b-b606-8ec330635ea4&width=768&dpr=4&quality=100&sign=e905a042&sv=1)

<br/>

### 3.4. 네이버 로그인 연동 개발하기
---

#### 3.4.1. 네이버 로그인 연동을 개발하기에 앞서
---

네이버 개발자센터를 통해 애플리케이션을 등록해야 한다.

등록한 후에 애플리케이션의 Client ID와 Client Secret 값을 확인할 수 있다.

> _**ClientID와 ClientSecret에 대하여**_
> 
> ClientID와 ClientSecret은 애플리케이션을 구분해주는 정보이다.
- ClientID : 알파뱃 대소문자, 숫자가 조합된 40자리 이하의 문자열
- ClientSecret : 알파뱃 대소문자, 숫자가 조합된 40자리 이하의 문자열
{: .prompt-info }

<br/>

#### 3.4.2. 네이버 로그인 연동 URL 생성하기
---

‘네이버 로그인’ URL을 먼저 생성해야 한다.

사용자가 로그인 연동에 동의하였을 경우 동의 정보를 포함하여 Callback URL로 전송된다.

- 요청 URL 정보
    - 메서드 : GET / POST
    - 요청 URL : [https://nid.naver.com/oauth2.0/authorize](https://nid.naver.com/oauth2.0/authorize)
    - 출력 포맷 : URL 리다이렉트
    - 설명 : 네이버 로그인 인증 요청

<br/>

##### 네이버 로그인 연동 API

_**요청 URL 정보**_

|메서드|요청 URL|출력 포맷|설명|
|---|---|---|---|
|GET / POST|[https://nid.naver.com/oauth2.0/authorize](https://nid.naver.com/oauth2.0/authorize)|URL 리다이렉트|네이버 로그인 인증 요청|

_**요청 변수 정보**_

|요청 변수명|타입|필수 여부|기본값|설명|
|---|---|---|---|---|
|response_type|string|Y|code|인증 과정에 대한 내부 구분값으로 ‘code’로 전송해야 함|
|client_id|string|Y|-|애플리케이션 등록 시 발급받은 Client ID 값|
|redirect_uri|string|Y|-|애플리케이션을 등록 시 입력한 Callback URL 값으로 URL 인코딩을 적용한 값|
|state|string|Y|-|사이트 간 요청 위조 (cross-site request forgery) 공격을 방지하기 위해 애플리케이션에서 생성한 토큰값으로 URL 인코딩을 적용한 값을 사용|

_**요청문 샘플**_

```
<https://nid.naver.com/oauth2.0/authorize?response_type=code&client_id=CLIENT_ID&state=STATE_STRING&redirect_uri=CALLBACK_URL>
```

<br/>

#### 3.4.3. 네이버 로그인 연동 결과 Callback 정보
---

네이버 로그인 인증 요청 API를 호출했을 때 사용자가 네이버로 로그인하지 않은 상태이면 네이버 로그인 화면으로 이동하고, 사용자가 로그인한 상태이면 기본 정보 제공 동의 확인 화면으로 이동한다.

정보 제공 동의 과정이 완료되면 콜백 URL에 code 값과 state 값이 URL 문자열로 전송된다.

code 값은 접근 토큰 발급 요청에 사용한다.

API 요청 실패시에는 에러 코드와 에러 메시지가 전송된다.

_**Callback 응답 정보**_

- API 요청 성공시 : [http://콜백URL/redirect?code={code값}&state={state값}](http://xn--url-l50nw08e/redirect?code=%7Bcode%EA%B0%92%7D&state=%7Bstate%EA%B0%92%7D)
- API 요청 실패시 : [http://콜백URL/redirect?state={state값}&error={에러코드값}&error_description={에러메시지}](http://xn--url-l50nw08e/redirect?state=%7Bstate%EA%B0%92%7D&error=%7B%EC%97%90%EB%9F%AC%EC%BD%94%EB%93%9C%EA%B0%92%7D&error_description=%7B%EC%97%90%EB%9F%AC%EB%A9%94%EC%8B%9C%EC%A7%80%7D)

|필드|타입|설명|
|---|---|---|
|code|string|네이버 로그인 인증에 성공하면 반환받는 인증 코드, 접근 토큰(access token) 발급에 사용|
|state|string|사이트 간 요청 위조 공격을 방지하기 위해 애플리케이션에서 생성한 토큰값으로 URL 인코딩을 적용한 값|
|error|string|네이버 로그인 인증에 실패하면 반환받는 에러 코드|
|error_description|string|네이버 로그인 인증에 실패하면 반환받는 에러 메시지|

<br/>

#### 3.4.4. 접근 토큰 발급 요청
---

Callback으로 전달받은 정보를 이용하여 접근 토큰을 발급받을 수 있다.

접근 토큰은 사용자가 인증을 완료했다는 것을 보장할 수 있는 인증 정보이다.

‘code’ 값을 이용하여 ‘접근토큰발급API’를 호출하게 되면 API 응답으로 접근토큰에 대한 정보를 받을 수 있다.

‘code’ 값을 이용한 API 호출은 최초 1번만 수행할 수 있으며 접근 토큰 발급이 완료되면 사용된 ‘code’는 더 이상 재사용할 수 없다.

<br/>

##### 접근 토큰 발급 API
---

_**요청 URL 정보**_

|메서드|요청 URL|출력 포맷|설명|
|---|---|---|---|
|GET / POST|[https://nid.naver.com/oauth2.0/token](https://nid.naver.com/oauth2.0/token)|json|접근토큰 발급 요청|

_**요청 변수 정보**_

|요청 변수 정보|타입|필수 여부|기본값|설명|
|---|---|---|---|---|
|grant_type|string|Y|-|인증 과정에 대한 구분값 1) 발급: ‘authorization_code’ 2) 갱신: ‘refresh_token’ 3) 삭제: ‘delete’|
|client_id|string|Y|-|애플리케이션 등록 시 발급받은 Client ID 값|
|client_secret|string|Y|-|애플리케이션 등록시 발급받은 Client secret 값|
|code|string|발급 때 필수|-|로그인 인증 요청 API 호출에 성공하고 리턴받은 인증코드 값 (authorization code)|
|state|string|발급 때 필수|-|애플리케이션에서 생성한 상태 토큰값으로 URL 인코딩을 적용한 값을 사용|
|refresh_token|string|갱신 때 필수|-|네이버 사용자 인증에 성공하고 발급받은 갱신 토큰 (refresh token)|
|access_token|string|삭제 때 필수|-|기 발급받은 접근 토큰으로 URL 인코딩을 적용한 값을 사용|
|service_provider|string|삭제 때 필수|‘NAVER’|인증 제공자 이름으로 ‘NAVER’로 세팅해 전송|

_**요청문 샘플**_

```
<https://nid.naver.com/oauth2.0/token?grant_type=authorization_code&client_id=jyvqXeaVOVmV&client_secret=527300A0_COq1_XV33cf&code=EIc5bFrl4RibFls1&state=9kgsGTfH4j7IyAkg>
```

_**응답 정보**_

|필드|타입|설명|
|---|---|---|
|access_token|string|접근 토큰, 발급 후 expires_in 파라미터에 설정된 시간(초)이 지나면 만료됨|
|refresh_token|string|갱신 토큰, 접근 토큰이 만료될 경우 접근 토큰을 다시 발급받을 때 사용|
|token_type|string|접근 토큰의 타입으로 Bearer와 MAC의 두 가지 지원|
|expires_in|integer|접근 토큰의 유효 기간(초 단위)|
|error|string|에러 코드|
|error_description|string|에러 메시지|

<br/>

#### 3.4.5. 접근 토큰을 이용하여 프로필 API 호출하기
---

접근 토큰을 이용하면 **프로필 정보 조회 API를 호출**하거나 오픈 API를 호출하는 것이 가능하다.

##### 프로필 정보 조회 API

_**요청 URL 정보**_

|메서드|인증|요청 URL|출력 포맷|설명|
|---|---|---|---|---|
|GET / POST|OAuth2.0|[https://openapi.naver.com/v1/nid/me](https://openapi.naver.com/v1/nid/me)|json|프로필 정보 조회|

_**요청 변수 정보**_

요청 변수는 별도로 없으며, 요청 URL로 호출할 때 아래와 같이 요청 헤더에 접근 토큰 값을 전달하면 된다.

_**요청 헤더**_

|요청 헤더명|설명|
|---|---|
|Authorization|접근 토큰(access token)을 전달하는 헤더 다음과 같은 형식으로 헤더 값에 접근 토큰(access token)을 포함한다. 토큰 타입은 “Bearer”로 값이 고정되어 있다. Authorization: {토큰 타입} {접근 토큰}|

_**요청문 예시**_

```
curl -XGET "<https://openapi.naver.com/v1/nid/me>" \\
     -H "Authorization: Bearer AAAAPIuf0L+qfDkMABQ3IJ8heq2mlw71DojBj3oc2Z6OxMQESVSrtR0dbvsiQbPbP1/cxva23n7mQShtfK4pchdk/rc="
```

_**출력 결과**_

|필드|타입|필수 여부|설명|
|---|---|---|---|
|resultcode|string|Y|API 호출 결과 코드|
|message|string|Y|호출 결과 메시지|
|response/id|string|Y|동일인 식별 정보|
|response/nickname|string|Y|사용자 별명|
|response/name|string|Y|사용자 이름|
|response/email|string|Y|사용자 메일 주소|
|response/gender|string|Y|성별 (F: 여자, M: 남자, U: 확인불가)|
|response/age|string|Y|사용자 연령대|
|response/birthday|string|Y|사용자 생일(MM-DD 형식)|
|response/profile_image|string|Y|사용자 프로필 사진 URL|
|response/birthyear|string|Y|출생연도|
|response/mobile|string|Y|휴대전화번호|

<br/>

#### 3.4.6. 접근 토큰을 이용하여 사용자 허용 프로필 권한 확인하기
---

사용자가 제공을 허용한 프로필의 항목을 확인

_**요청 URL 정보**_

|메서드|인증|요청 URL|출력 포맷|설명|
|---|---|---|---|---|
|GET / POST|OAuth2.0|[https://openapi.naver.com/v1/nid/verify](https://openapi.naver.com/v1/nid/verify)|json|접근 토큰 검증 및 권한 확인|

_**요청 변수 정보**_

|요청 변수 정보|타입|필수 여부|기본값|설명|
|---|---|---|---|---|
|info|boolean|N|false|true일 경우 권한 설정정보 응답|

_**요청 헤더**_

|요청 헤더명|설명|
|---|---|
|Authorization|접근 토큰(access token)을 전달하는 헤더 다음과 같은 형식으로 헤더 값에 접근 토큰(access token)을 포함한다. 토큰 타입은 “Bearer”로 값이 고정되어 있다. Authorization: {토큰 타입} {접근 토큰}|

<br/>

### 3.5. 네이버의 로그인오픈 API의 이용
---

- [카페 오픈 API](https://developers.naver.com/products/login/cafe/cafe.md)
- [캘린더 오픈 API](https://developers.naver.com/products/login/calendar/calendar.md)

<br/>

## 4. 시스템에 네이버 로그인 정보 유지
---

### 4.1. 기존 로그인 시스템에 네이버 로그인 적용하기
---

#### 4.1.1. Database의 구성
---

_**일반적인 회원 데이터베이스 구조**_

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252FC0NOnstVVKAiTZmZ0FMq%252Fimage.png%3Falt%3Dmedia%26token%3D17c93171-0197-440f-9452-9c30eba5cdc2&width=768&dpr=4&quality=100&sign=bf0ba34f&sv=1)

_**테이블 구조 및 테이블 생성 SQL 예시**_

```sql
create table users (
    id int not null auto_increment,
    username varchar(50) null,
    password varchar(50) null,
    email varchar(100) null,
    nickname varchar(50) null,
    mobile varchar(20) null,
    create_date datetime null,
    modify_date datetime null,
    primary key (id),
    index idx1_username (username asc),
    index idx2_email (email, asc)
);
```

_**네이버 로그인 연동을 위한 테이블 구성**_

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252F20nWVBiA5f9MqB0MDK0J%252Fimage.png%3Falt%3Dmedia%26token%3D9d42e4d3-3cf3-44cf-9cbc-2cbce65ccfb6&width=768&dpr=4&quality=100&sign=ef07b0de&sv=1)

_**네이버 로그인 사용자 정보를 저장하기 위한 테이블 구조 및 생성 SQL 예시**_

```sql
create table sns_info (
    id int(11) not null,
    sns_id varchar(255) not null,
    sns_type varchar(10 null,
    sns_name varchar(255) null,
    sns_profile varchar(255) null,
    sns_connect_date datetime null,
    key idx01_id (id),
    key idx02_sns_id (sns_id),
    constraint id foreign key (id) references users (id)
);
```

<br/>

#### 4.1.2. 사용자 연동 처리 및 로그인 처리
---

사용자 유니크 ID(sns_id)를 사용하여 사용자 정보를 조회할 수 있다.

```sql
select a.username, a.email, a.nickname, a.create_time, a.modify_date, 
	b.sns_id, b.sns_type, b.sns_name, b.sns_profile, b.sns_connect_date
from users a, sns_info b
where a.id = b.id and b.sns_id = :snsId
```

- 사용자에 대한 조회가 완료되었을 경우 세션에 로그인 정보를 발행하거나 쿠키로 로그인 정보를 발행하여 로그인 상태로 만들 수 있다.

<br/>

### 4.2. 신규 시스템에 네이버 로그인 적용하기
---

#### 4.2.1. Database의 구성
---

_**네이버 로그인 연동을 위한 테이블 구성**_

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252FTy8HsoAwnwkBBYIsPKp2%252Fimage.png%3Falt%3Dmedia%26token%3Da5a1cb51-847f-42c4-959a-319adf5405ef&width=768&dpr=4&quality=100&sign=6cd485ec&sv=1)

_**네이버 로그인 사용자 정보를 저장하기 위한 테이블 구조 및 생성 SQL 예시 (MYSQL)**_

```sql
create table users (
	id int not null auto_increment,
	username varchar(50) null,
	email varchar(100) null,
	nickname varchar(50) null,
	sns_id varchar(255) null,
	sns_type varchar(10) null,
	sns_profile varchar(255) null,
	create_date datetime null,
	modify_date datetime null,
	primary key (id),
	index idx1_username (username, asc),
	index idx2_email (email, asc),
	index idx3_sns_id (sns_id, asc)
);
```

<br/>

#### 4.2.2. 네이버 로그인을 통한 회원 가입
---

네이버 로그인을 통하여 얻은 사용자 유니크 ID 정보를 이용하여 사용자를 식별할 수 있다.

이 식별자는 이미 가입한 사용자인지 신규 연동 사용자인지 확인하는 키로 사용할 수 있다.

```sql
insert into sns_info(id, sns_id, sns_type, sns_name, sns_profile, sns_connect_date)
values (:id, :snsId, :snsType, :snsName, :snsProfile, now())
```

<br/>

#### 4.2.3. 네이버 로그인을 통한 로그인/로그아웃의 구현
---

사용자 유니크 ID 정보를 통해 사용자 정보를 조회할 수 있다.

```sql
select a.username, a.email, a.nickname, a.create_date, a.modify_date, 
	b.sns_id, b.sns_type, b.sns_name, b.sns_profile, b.sns_connect_date
from users a, sns_info b
where a.id = b.id and b.sns_id = :snsId
```

<br/>

## 5. 네이버 로그인 사용자 프로필 갱신 및 재인증
---

### 5.1. 네이버 로그인 사용자의 프로필 갱신
---

#### 5.1.1. 접근 토큰에 대하여
---

접근 토큰 발급 API를 통하여 접근 토큰 및 갱신 토큰을 발급받을 수 있다.

_**접근 토큰 API 응답형태**_

```json
{
    "access_token" : "접근토큰(Access Token)",
    "refresh_token" : "갱신토큰(Refresh Token)",
    "token_type" : "접근토큰 타입(bearer)",
    "expires_in" : "유효시간(초)"
}
```

_**접근 토큰 규격**_

- access_token : 알파벳 대소문자, 숫자, 특수문자 (+/=)가 조합된 256자리 이하의 문자열
    
- refresh_token : 알파벳 대소문자, 숫자가 조합된 256자리 이하의 문자열
    
- expires_in : 숫자, 발급 시점부터 expires_in(초) 후 까지 유효
    

_**접근 토큰의 용도**_

접근 토큰은 사용자 프로필 조회 API를 호출하거나 네이버에서 제공하는 로그인 OpenAPI를 이용할 때 사용자 인증값으로 이용된다.

_**접근 토큰 사용 방법**_

접근 토큰을 이용하여 API를 호출하는 경우 다음과 같이 요청 헤더에 접근 토큰 값을 포함한다.

- 요청 헤더명 : Authorization
- 요청 헤더값 형식 : TOKEN_TYPE ACCESS_TOKEN
    

_**접근 토큰을 포함한 응답헤더 예시**_

```
Authorization: Bearer ACCESS_TOKEN
```

접근 토큰을 이용한 API 호출 예시

```bash
curl -XGET "<https://openapi.naver.com/vi/nid/me>" \\
     -H "Authorization: Bearer ACCESS_TOKEN"

GET /v1/nid/me HTTP/1.1
Host: openapi.naver.com
User-Agent: curl/7.43.0
Accept: */*
Authorization: Bearer ACCESS_TOKEN
```

<br/>

#### 5.1.2. 갱신 토큰에 대하여
---

접근 토큰은 발급 후 expires_in(기본 3600초 / 1시간) 이내에만 사용이 가능하며 expires_in(초) 만큼의 시간이 지나면 접근 토큰은 더 이상 사용할 수 없다.

접근 토큰이 만료된 경우 갱신 토큰(refresh token)을 이용하여 유효한 접근토큰을 재발급 받을 수 있다.

갱신 토큰은 접근 토큰이 만료될 것을 대비하여 데이터베이스에 별도로 저장하고 이후 필요에 따라 갱신 토큰을 사용하면 된다.

<br/>

##### 접근 토큰 갱신 API

_**요청 URL 정보**_

|메서드|요청 URL|출력 포맷|설명|
|---|---|---|---|
|GET / POST|[https://nid.naver.com/oauth2.0/token](https://nid.naver.com/oauth2.0/token)|json|갱신토큰을 이용한 접근토큰 재발급 요청|

_**요청 변수 정보**_

|요청 변수 정보|타입|필수 여부|기본값|설명|
|---|---|---|---|---|
|client_id|string|Y|-|애플리케이션 등록 시 발급받은 Client ID 값|
|client_secret|string|Y|-|애플리케이션 등록시 발급받은 Client secret 값|
|refresh_token|string|Y|-|접근토큰 발급 API를 통하여 발급받은 갱신토큰 값|
|grant_type|string|Y|-|요청 타입. refresh_token으로 설정|

_**요청문 샘플**_

```
<https://nid.naver.com/oauth2.0/token?grant_type=refresh_token&client_id=CLIENT_ID&client_secret=CLIENT_SECRET&refresh_token=REFRESH_TOKEN>
```

_**응답 정보**_

|필드|타입|설명|
|---|---|---|
|access_token|string|재발급 받은 접근토큰|
|token_type|string|토큰 타입 (bearer)|
|expires_in|integer|접근 토큰의 유효 기간(초 단위)|

<br/>

#### 5.1.3. 접근 토큰 만료와 갱신 주기. 프로필 정보의 갱신
---

접근 토큰이 유효하지 않은 경우에는 갱신토큰을 이용하여 유효한 접근 토큰을 재발급 받거나 네이버 로그인 인증을 다시한번 수행하는것으로 유효한 접근 토큰을 받을 수 있다.

접근 토큰의 유효성을 판단하기 위해서는 다음과 같은 방법을 이용할 수 있다.

- 프로필 정보 조회 API 호출 시 응답이 정상적으로 전달될 경우 접근 토큰은 유효하다고 할 수 있다.
- 접근 토큰 유효성 체크 API 호출을 통해 현재 접근 토큰이 유효한지 판단할 수 있다.

<br/>

##### 접근 토큰 유효성 체크 API

_**요청 URL 정보**_

|메서드|요청 URL|출력 포맷|설명|
|---|---|---|---|
|GET / POST|[https://openapi.naver.com/v1/nid/me](https://openapi.naver.com/v1/nid/me)|json|접근 토큰 유효성 체크|

_**요청 변수 정보**_

요청 변수는 별도로 없으며, 요청 URL로 호출할 때 아래와 같이 요청 헤더에 접근 토큰 값을 전달하면 된다.

_**요청 헤더**_

|요청 헤더명|설명|
|---|---|
|Authorization|접근 토큰(access token)을 전달하는 헤더 다음과 같은 형식으로 헤더 값에 접근 토큰(access token)을 포함한다. 토큰 타입은 “Bearer”로 값이 고정되어 있다. Authorization: {토큰 타입} {접근 토큰}|

_**요청문 샘플**_

```bash
curl -XGET "<https://openapi.naver.com/v1/nid/verify>" \\
		 -H "Authorization: Bearer ..."
```

_**응답 정보**_

|필드|타입|설명|
|---|---|---|
|resultcode|string|API 호출 결과 코드|
|message|string|접근토큰 유효성 체크 결과 메시지|

<br/>

#### 5.1.4. 프로필의 갱신
---

사용자는 다음 정보를 변경할 수 있다.

- 이름, 닉네임, 프로필 이미지, 연락처 이메일 주소, 생일, 성별, 연령대, 출생연도, 휴대전화번호
    

따라서 주기적으로 또는 사용자 로그인이 발생할 때마다 프로필 정보를 조회하여 갱신하는 것을 권장하고 있다.

<br/>

#### 5.1.5. 사용자가 거부한 프로필 권한에 대하여 다시 동의를 수행하는 경우
---

제공이 거부된 프로필 항목이 서비스 이용에 반드시 필요한 항목일 경우에는 사용자로 하여금 재동의를 수행해야 한다.

<br/>

##### 네이버 로그인 재동의 API
---

_**요청 URL 정보**_

|메서드|요청 URL|출력 포맷|설명|
|---|---|---|---|
|GET / POST|[https://nid.naver.com/oauth2.0/authorize](https://nid.naver.com/oauth2.0/authorize)|URL 리다이렉트|네이버 로그인 인증 요청|

_**요청 변수 정보**_

|요청 변수 정보|타입|필수 여부|기본값|설명|
|---|---|---|---|---|
|response_type|string|Y|code|인증 과정에 대한 내부 구분값으로 ‘code’로 전송해야 함|
|client_id|string|Y|-|애플리케이션 등록시 발급받은 Client ID 값|
|redirect_uri|string|Y|-|애플리케이션을 등록 시 입력한 Callback URL 값으로 URL 인코딩을 적용한 값|
|state|string|Y|-|사이트 간 요청 위조(cross-site request forgery) 공격을 방지하기 위해 애플리케이션에서 생성한 상태 토큰값으로 URL 인코딩을 적용한 값을 사용|
|auth_type|string|Y|-|재동의 요청의 경우 ‘reprompt’로 전송해야 함|

_**요청문 샘플**_

```
<https://nid.naver.com/oauth2.0/authorize?response_type=code&client_id=CLIENT_ID&state=STATE_STRING&redirect_uri=CALLBACK_URI&auth_type=reprompt>
```

<br/>

### 5.2. 재인증
---

#### 5.2.1. 사용자 재인증이 필요한 경우
---

접근 토큰이 유효하더라도 네이버 로그인 재인증을 통해서 네이버 사용자 인증을 수행할 수 있다.

일반적으로 다음과 같은 상황에서 사용된다.

- 개인정보 조회 또는 변경 페이지에 접근
- 서비스 해지 또는 탈퇴
- 계정 도용이 의심되는 사용자

네이버 로그인 재인증은 다음 절차로 수행된다.

1. 네이버 로그인 재인증 요청
2. 현재 로그인 상태와 관계없이 네이버 로그인 절차 요구
3. ID/PW 입력
4. 인증 완료

<br/>

##### 네이버 로그인 재인증 API

_**요청 URL 정보**_

|메서드|요청 URL|출력 포맷|설명|
|---|---|---|---|
|GET / POST|[https://nid.naver.com/oauth2.0/authorize](https://nid.naver.com/oauth2.0/authorize)|URL 리다이렉트|네이버 로그인 인증 요청|

_**요청 변수 정보**_

|요청 변수 정보|타입|필수 여부|기본값|설명|
|---|---|---|---|---|
|response_type|string|Y|code|인증 과정에 대한 내부 구분값으로 ‘code’로 전송해야 함|
|client_id|string|Y|-|애플리케이션 등록시 발급받은 Client ID 값|
|redirect_uri|string|Y|-|애플리케이션을 등록 시 입력한 Callback URL 값으로 URL 인코딩을 적용한 값|
|state|string|Y|-|사이트 간 요청 위조(cross-site request forgery) 공격을 방지하기 위해 애플리케이션에서 생성한 상태 토큰값으로 URL 인코딩을 적용한 값을 사용|
|auth_type|string|Y|-|재동의 요청의 경우 ‘reauthenticate’로 전송해야 함|

_**요청문 샘플**_

```
<https://nid.naver.com/oauth2.0/authorize?response_type=code&client_id=CLIENT_ID&state=STATE_STRING&redirect_uri=CALLBACK_URI&auth_type=reauthenticate>
```

<br/>

### 5.3. 네이버 로그인 연동 해제
---

#### 5.3.1. 네이버 로그인 연동 해제가 필요한 경우
---

사용자가 서비스를 더이상 이용하지 않을 경우 네이버 로그인 연동 해제 API를 통해 연결 관계를 끊을 수 있다.

연동 해제 API를 통해 성공적으로 연동이 해제되면 다음과 같이 변경사항이 적용된다.

- 접근토큰과 갱신토큰 만료
- 네이버 로그인 연동 목록에서 항목 제거
- 새로 연동할 경우 사용자 동의를 다시 받음

<br/>

##### 네이버 로그인 연동 해제 API

_**요청 URL 정보**_

|메서드|요청 URL|출력 포맷|설명|
|---|---|---|---|
|GET / POST|[https://nid.naver.com/oauth2.0/token](https://nid.naver.com/oauth2.0/token)|URL 리다이렉트|접근토큰을 이용한 연결해제 요청|

_**요청 변수 정보**_

|요청 변수 정보|타입|필수 여부|기본값|설명|
|---|---|---|---|---|
|client_id|string|Y|-|애플리케이션 등록시 발급받은 Client ID 값|
|client_secret|string|Y|-|애플리케이션 등록시 발급받은 Client Secret 값|
|access_token|string|Y|-|유효한 접근토큰 값|
|grant_type|string|Y|-|요청 타입. delete 로 설정|

_**요청문 샘플**_

```
<https://nid.naver.com/oauth2.0/token?grant_type=delete&client_id=CLIENT_ID&client_secret=CLIENT_SECERT&access_token=ACCESS_TOKEN>
```

_**응답 정보**_

|필드|타입|설명|
|---|---|---|
|access_token|string|삭제처리된 접근토큰|
|result|string|처리결과 (success)|

- 연동 해제를 수행하기 전에 접근 토큰의 유효성을 점검하고 접근토큰을 갱신한 후에 수행하는 것을 권장한다.

<br/>

## 6. 네이버 로그인 부가 기능
---

### 6.1. 네이버앱에서 서비스 자동로그인 처리
---

#### 6.1.1. 서비스 자동로그인이란
---

사용자의 이용편의를 위하여 서비스에 자동으로 로그인된 상태로 전환하는 기능

- 네이버앱에서 검색을 통해 서비스에 접근하는 경우
- 네이버앱 즐겨찾기를 통해 서비스에 접근하는 경우
- 톡톡, 메일 등으로 전달된 링크를 통해 서비스에 접근하는 경우

자동 로그인 절차

1. 자동 로그인 처리 가능 환경에 대한 체크
2. 네이버 로그인 연동 URL에 대한 처리
3. Callback 페이지에서 연동 처리 또는 오류 사항에 대한 처리
4. 로그인 완료

<br/>

#### 6.1.2. 제약사항
---

본 기능은 “네이버 앱”에서 서비스의 웹페이즈를 접근하는 경우에만 수행이 가능하다.

[_**관련 링크**_](https://developers.naver.com/docs/login/devguide/devguide.md#6--%EB%84%A4%EC%9D%B4%EB%B2%84-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EB%B6%80%EA%B0%80-%EA%B8%B0%EB%8A%A5)

<br/>

### 6.2. 네이버 로그인 플러스
---

약관 동의 대행 기능과 회원에게 마케팅 메시지 발송을 위한 기능 제공

[_**관련 링크**_](https://developers.naver.com/docs/login/devguide/devguide.md#6-2-%EB%84%A4%EC%9D%B4%EB%B2%84-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%ED%94%8C%EB%9F%AC%EC%8A%A4)

<br/>

## 7. 튜토리얼
---

먼저 [애플리케이션 등록](https://developers.naver.com/apps/#/register?api=nvlogin)을 수행한다.

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252Frihv9Y8odIj4lnAdrJWP%252Fimage.png%3Falt%3Dmedia%26token%3Dd74c3640-d437-4999-bda5-26e8e4a927ae&width=768&dpr=4&quality=100&sign=96ae8307&sv=1)

- 등록한 후에 Client ID와 Client Secret 정보를 복사하여 사용한다.

<br/>

### 7.1. 프로세스 설계
---

#### 7.1.1. 회원가입
---

```mermaid

sequenceDiagram
participant F as frontend
participant OA as oauth-authorization-server
participant OR as oauth-resource-server
participant B as backend
note over F: 로그인 버튼 클릭
F->>OA: 로그인 페이지 요청
OA-->>F: 로그인 페이지 응답
F->>OA: 로그인 요청
OA-->>F: Authorization code 발급
F->>B: Authorization code로 로그인 요청
B->>OA: Authorization code로 Access Token 요청
OA-->>B: Access Token 발급
B->>OR: Access Token으로 회원 정보 요청
OR-->>B: 회원 정보 응답
note over B: 회원 정보가 존재하는지 확인
B-->>F: 회원 정보가 존재하지 않음을 응답
note over F: 회원가입에 필요한 정보 입력
F->>B: 회원가입 요청
B-->>F: 회원가입 응답
```

#### 7.1.2. 로그인
---

```mermaid
sequenceDiagram

participant F as frontend
participant OA as oauth-authorization-server
participant OR as oauth-resource-server
participant B as backend
note over F: 로그인 버튼 클릭
F->>OA: 로그인 페이지 요청
OA-->>F: 로그인 페이지 응답
F->>OA: 로그인 요청
OA-->>F: Authorization code 발급
F->>B: Authorization code로 로그인 요청
B->>OA: Authorization code로 Access Token 요청
OA-->>B: Access Token 발급
B->>OR: Access Token으로 회원 정보 요청
OR-->>B: 회원 정보 응답
note over B: 회원 정보가 존재하는지 확인
B-->>F: Access Token과 Refresh Token 응답
```

<br/>

### 7.2. 예제 코드
---

[GitHub: LeeSM0518/oauth-tutorial](https://github.com/LeeSM0518/oauth-tutorial)

<br/>

## Reference
---

[Naver: 로그인 개발가이드](https://developers.naver.com/docs/login/devguide/devguide.md)
