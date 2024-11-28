---
title: OneSignal Guide
date: 2024-11-05 09:00:00 +0900
categories: onesignal
tags:
  - onesignal
  - notification
description: OneSignal으로 푸시 알림을 구현하기 위한 가이드 문서 정리
---

## 1. 빠른 시작 가이드

### 1.1. 조직 만들기

조직은 앱과 메시지 채널을 관리하기 위한 기반이 된다.

![onesignal1](/assets/img/onesignal1.png)

### 1.2. 앱 추가

조직을 설정한 후에 다음 단계로 앱을 추가한다.

![onesignal2](/assets/img/onesignal2.png)

### 1.3. 채널 설정

OneSignal은 여러 플랫폼에서 광범위한 메시징 채널을 지원한다.

![onesignal3](/assets/img/onesignal3.png)

#### 1.4. 사용자 및 구독

사용자가 앱과 상호 작용하고 메시징 채널을 구독하면 OneSignal ID(OneSignal의 사용자 ID)와 구독 ID(OneSignal의 고유 채널/기기 ID)가 할당된다. 각 사용자는 설정한 채널 수와 앱과 상호 작용하는 방식에 따라 여러 구독을 할 수 있다.

#### 1.5. 속성 및 이벤트 태그

태그는 이벤트(ex. 장바구니 업데이트, 마지막 주문, 온보딩 상태, 버튼 클릭 등) 및 특정 사용자 속성(ex. 이름, 위치, 선호도 등)을 추적하여 대상 고객에게 개인화되고 적절한 메시지를 보내는 데 사용된다.

#### 1.6. 세그먼트

대시보드에서 메시지 캠페인을 보낼 때는 Audience를 만들어야 한다. Audience는 세그먼트별로 그룹화된 사용자와 구독으로 구성된다. 예를 들어, 데이터 태그를 사용하면 해당 태그가 있는 사용자 세그먼트를 구성할 수 있다.

![onesignal4](/assets/img/onesignal4.png)

<br/>

## 2. 빠른 시작 API 가이드

### 2.1. 키와 식별자

#### 2.1.1. REST API Key

Dashboard > Settings > Keys & IDs 에서 REST API 키를 만든다.

![onesignal5](/assets/img/onesignal5.png)

#### 2.1.2. App ID & Key Safety

모든 요청에는 App ID가 필요하다. API 키를 안전하게 유지하는 것이 중요하다. App ID는 공개되지만 REST API 키를 사용하면 구독 관리, 사용자 데이터 수정, 사용자에게 메시지 전송을 포함하여 앱의 거의 모든 측면을 제어할 수 있다.

앱이 해킹되었을 수 있다고 의심되는 경우 가능한 한 빨리 새 REST API 키를 생성해야 한다. 새 키를 생성하려면 Dashbaord > Settings > Keys & IDs 로 이동한 다음 새 API 키 생성 버튼을 클릭하면 된다.

### 2.2. 인증

Authorization 헤더에 Basic 과 함께 REST API Key를 사용해라.

- API Key 예시 : `MjAyYzJlOTAtMTY4Mi00ODFlLTg2MDYtZTM2YzllM2ZlZTVi`
- Header 예시 : `Authorization: Basic MjAyYzJlOTAtMTY4Mi00ODFlLTg2MDYtZTM2YzllM2ZlZTVi`

**예시**
```http
POST /notifications HTTP/1.1
Content-Type: application/json
Authorization: Basic MjAyYzJlOTAtMTY4Mi00ODFlLTg2MDYtZTM2YzllM2ZlZTVi
Host: api.onesignal.com
Connection: close
Content-Length: 220

{
  "app_id":"202d4f61-1ca9-42df-9d36-bb17d8123abc",
  "contents":{"en":"Hello, World","es":"Hola Mundo","fr":"Bonjour le monde","zh-Hans":"\u4f60\u597d\u4e16\u754c"},
  "target_channel":"push",
  "included_segments":["All Subscribers"]
}
```

**오류 처리**

유효하지 않거나 만료된 키는 다음 오류 메시지를 응답한다.
- 잘못된 API Key 일 경우 : 401 Unauthorized
- Authorization 헤더가 존재하지 않을 경우 : 400 Bad Request

## 3. REST API 개요

### 3.1. 요구사항

OneSignal API는 표준 포트 443에서 TLS 1.2 이상을 지원하는 HTTPS 연결을 사용하여 액세스한다.

방화벽과 프록시는 포트 443에서 아웃바운드 HTTPS 트래픽이 REST API에 연결되도록 허용해야 한다.

모든 공개 IP 주소에 대한 HTTPS 트래픽을 허용 목록에 추가하거나 api.onesignal.com을 허용하는 것이 좋다.

### 3.2. 메시지 보내기

서버에서 푸시 알림, 이메일, SMS를 전송할 수 있다.

- 메시지는 채널에 따라 사용자 ID, 이메일 주소 또는 전화 번호를 사용하여 세그먼트, 데이터 필터 및 특정 장치에 전송할 수 있다.

- 콘텐츠 및 언어, 커스텀 데이터 및 이미지, 일정 옵션 등 사용 가능한 모든 매개변수가 지원된다.

- [예제 코드](https://documentation.onesignal.com/reference/create-message#example-code---create-notification)를 참고

**템플릿 생성, 수정, 조회, 삭제**

템플릿은 대시보드나 API를 통해 생성, 수정, 조회, 삭제가 가능한 재사용 가능한 푸시, 이메일, SMS 메시지이다.

### 3.3. 사용자 생성, 수정, 조회, 삭제

각 사용자는 고유한 OneSignal ID를 갖지만, 별칭 및 외부 ID로 조회할 수도 있다.

- 사용자 생성 : 구독 또는 별칭을 사용하여 새 사용자를 만든다.
- 사용자 조회 : 사용자의 속성, 별칭, 구독을 조회한다.
- 사용자 수정 : 태그와 같은 사용자의 속성을 수정한다. 구독 생성 또는 구독 수정을 사용하여 사용자에게 새 구독을 추가하거나 현재 구독을 업데이트 한다.
- 사용자 삭제 : 사용자와 모든 구독을 삭제한다.

**세그먼트 생성, 조회, 삭제**

세그먼트를 만들지 않거나 이러한 필터 조합에 많은 알림을 보낼 계획이 없는 경우 OneSignal을 사용하면 데이터 필터를 사용하여 장치를 직접 타겟팅할 수 있다.

### 3.4. 메시지 작성

푸시, 이메일, SMS를 통한 메시지 전송

#### 3.4.1. 메시지를 보내는 방법

메시지 작성
: 'Hello, world' 메시지를 현지화하여 전송할 수 있다.

```json
{
  "contents": {
    "en": "Hello, world",
    "es": "Hola Mundo"
  }
}
```

타겟팅 전략 선택
: 사용자를 타겟팅하기 위해 하나의 전략을 선택한다. "Subscribed Users" 세그먼트를 타겟하여 전송할 수 있다.

- Segments
- Filters
- Aliases

```json
{
  "contents": {
    "en": "Hello, world",
    "es": "Hola Mundo"
  },
  "included_segments": ["Subscribed Users"]
}
```


배송 채널 선택
: 채널을 통해 메시지를 전달할 수 있다. 푸시 알림으로 설정하여 전송할 수 있다.

- 모바일 및 웹 푸시 알림 ("push")
- 이메일 ("email")
- SMS ("sms")

```bash
curl -X "POST" "https://api.onesignal.com/notifications" \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Basic MjAyYzJlOTAtMTY4Mi00ODFlLTg2MDYtZTM2YzllM2ZlZTVi' \
     -d $'{
  "target_channel": "push",
  "included_segments": [
    "Subscribed Users"
  ],
  "app_id": "202d4f61-1ca9-42df-9d36-bb17d8123abc",
  "contents": {
    "en": "Hello, world",
    "es": "Hola mundo",
    "fr": "Bonjour le monde",
    "zh-Hans": "你好世界"
  }
}'
```

전송 일정 설정
: 전송 제한과 시간도 지정할 수 있다. 모든 구독자에게 메시지를 전송할 수 있다.

```http
POST /notifications HTTP/1.1
Content-Type: application/json
Authorization: Basic MjAyYzJlOTAtMTY4Mi00ODFlLTg2MDYtZTM2YzllM2ZlZTVi
Host: api.onesignal.com

{
  "app_id":"202d4f61-1ca9-42df-9d36-bb17d8123abc",
  "contents":{"en":"Hello, World","es":"Hola Mundo","fr":"Bonjour le monde","zh-Hans":"\u4f60\u597d\u4e16\u754c"},
  "target_channel":"push",
  "included_segments":["All Subscribers"]
}
```

**특정 사용자에게 알림 전송**

```bash
curl --request POST \
     --url 'https://api.onesignal.com/notifications' \
     --header 'Authorization: Basic YOUR_REST_API_KEY' \
     --header 'accept: application/json' \
     --header 'content-type: application/json' \
     --data '
{
  "app_id": "YOUR_APP_ID",
  "target_channel": "push",
  "headings": {"en": "English Title", "es": "Spanish Title"},
  "contents": {"en": "English Message", "es": "Spanish Message"},
  "include_aliases": {
    "external_id": [
      "USER_ID_1",
      "USER_ID_2",
      "USER_ID_3"
    ]
  }
}
'
```


## 4. 사용자 생성

새로운 사용자를 만들거나 기존 사용자와 관련된 구독을 수정한다.

이 엔드포인트를 사용하여 다음을 수행할 수 있다.

- **사용자 생성** : 별칭, 속성, 구독과 같은 사용자별 데이터를 제공하여 OneSignal 시스템에 새 사용자를 등록한다.

- **새 구독 만들기, 기존 구독 업데이트 또는 이전** : 새 구독을 추가하거나 기존 구독(ex. push, email, sms)을 새로 만든 사용자 또는 기존 사용자와 연결한다.

- **새 별칭 추가** : 사용자에게 사용자 지정 식별자(ex. external_id, facebook_id, 커스텀 식별자)를 할당하여 여러 플랫폼에서 사용자를 고유하게 식별한다.

```bash
curl -X "POST" "https://api.onesignal.com/apps/<APP_ID>/users" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "properties": {
    "country": "US",
    "tags": {
      "favorite_team": "Lakers"
    },
    "language": "EN"
  },
  "identity": {
    "external_id": "test"
  }
}'
```

**어플 사용자 등록**

```bash
curl -X "POST" "https://api.onesignal.com/apps/<APP_ID>/users" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "subscriptions": [
    {
      "type": "iOSPush",
      "token": "20bdb8fb3bdadc1bef037eefcaeb56ad6e57f3241c99e734062b6ee829271b71",
      "enabled": true,
      "notification_types": 1,
      "session_time": 98,
      "session_count": 6,
      "sdk": "",
      "device_model": "iPhone 14",
      "device_os": "18.0",
      "rooted": false,
      "test_type": 1,
      "app_version": "5.1.7",
      "web_auth": "",
      "web_p256": ""
    }
  ],
  "properties": {
    "country": "US",
    "tags": {
      "favoriteTeam": "Lakers"
    },
    "language": "EN"
  },
  "identity": {
    "external_id": "test"
  }
}'

```

<br/>

---

## Reference

<https://documentation.onesignal.com/docs/quickstart-guide>
<https://documentation.onesignal.com/reference/quick-start-api-guide>
