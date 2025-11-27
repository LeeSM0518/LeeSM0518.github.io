---
title: Spring Security (7.0.0)
date: 2025-11-27 09:00:00 +0900
categories: spring
tags:
  - spring
  - security
description:
---

Spring Security는 인증, 인가를 지원하고 주요 공격으로부터 애플리케이션을 보호해주는 프레임워크다.

<br/>

## Spring Security 시작하기
---

Spring Boot는 Spring Security 관련 의존성을 집약하는 `spring-boot-starter-security` 스타터를 제공한다.

```groovy
dependencies { 
  implementation "org.springframework.boot:spring-boot-starter-security" 
}
```

<br/>

Spring Security 버전을 재정의하려면 아래와 같이 빌드 속성을 사용할 수 있다.

```groovy
ext['spring-security.version']='6.5.6'
```

<br/>

Spring Boot 없이 Spring Security를 사용할 때 권장되는 방법은 Spring Security의 BOM을 사용하여 전체 프로젝트에서 일관된 버전의 Spring Security가 사용되도록 하는 것이다.

```groovy
plugins {
  id "io.spring.dependency-management" version "1.0.6.RELEASE"
}

dependencyManagement {
  imports {
    mavenBom 'org.springframework.security:spring-security-bom:6.5.6'
  }
}
```

<br/>

최소한의 Spring Security Maven 의존성 세트는 다음과 같다.

```groovy
dependencies {
  implementation "org.springframework.security:spring-security-web"
  implementation "org.springframework.security:spring-security-config"
}
```

<br/>

## Spring Security Features
---

### 인증 (Authentication)
---

인증은 특정 리소스에 접근하려는 사람의 신원을 확인하는 방법이다. 인증이 수행되면 신원을 알 수 있으며 권한 부여(인가)를 수행할 수 있다.

<br/>

### 인가 (Authorization)
---

인가는 누가 특정 리소스에 접근할 수 있는지를 결정하는 것이다. 요청 기반 인가와 메서드 기반 인가를 통해 심층 방어를 제공한다.

<br/>

### 취약점 공격 방어 (Protection Against Exploits)
---

CSRF나 Header, Request를 통한 일반적인 취약점 공격에 대한 보호를 제공한다.

<br/>

#### 크로스 사이트 요청 위조 (Cross Site Request Forgery, CSRF)
---

CSRF 공격은 피해자의 웹사이트에서 오는 HTTP 요청과 공격자의 웹사이트에서 오는 요청이 정확히 동일하기 때문에 발생하는 것이다. CSRF 공격으로부터 보호하려면, 악성 사이트가 제공할 수 없는 무언가가 요청에 있어야 두 요청을 구별할 수 있다.

<br/>

Spring은 CSRF 공격으로부터 보호하기 위해 두 가지 메커니즘을 제공한다.

- 동기화 토큰 패턴 사용하기
- 세션 쿠키에 SameSite 속성을 지정하기

<br/>

**동기화 토큰 패턴 (Synchronizer Token Pattern)**

CSRF 공격으로부터 보호하는 가장 일반적이고 포괄적인 방법은 동기화 토큰 패턴을 사용하는 것이다.

HTTP 요청이 제출되면, 서버는 예상되는 CSRF 토큰을 조회하고 HTTP 요청의 실제 CSRF 토큰과 비교해야 한다. 값이 일치하지 않으면 HTTP 요청은 거부되어야 한다.

<br/>

**SameSite 속성**

CSRF 공격으로부터 보호하는 다른 방법은 쿠키에 SameSite 속성을 지정하는 것이다.

서버는 쿠키를 설정할 때 `SameSite` 속성을 지정하여 외부 사이트에서 올 때 쿠키가 전송되지 않아야 함을 나타낼 수 있다.

`SameSite` 속성이 있는 HTTP 응답 헤더의 예는 다음과 같다.
```
Set-Cookie: JSESSIONID=randomid; Domain=bank.example.com; Secure; HttpOnly; SameSite=Lax
```

<br/>

`SameSite` 속성의 유효한 값은 다음과 같다.
- `Strict` : 동일 사이트에서 오는 모든 요청에 쿠키가 포함된다. 그렇지 않으면 쿠키가 HTTP 요청에 포함되지 않는다.
- `Lax` : 동일 사이트에서 오거나 최상위 탐색에서 요청이 오고 메서드가 읽기 전용인 경우 쿠키가 전송된다. 그렇지 않으면 쿠키가 HTTP 요청에 포함되지 않는다.

<br/>

`SameSite` 를 `Strict` 로 설정하면 강력한 방어를 제공하지만 사용자를 혼란스럽게 할 수 있다. 만약 `social.example.com` 에 로그인 한 사용자가 `email.example.org` 에 접속할 경우 `SameSite` 가 `Strict` 이면 쿠키가 전송되지 않으므로 사용자가 인증되지 않는다.

<br/>

#### 보안 HTTP 응답 헤더
---

웹 애플리케이션의 보안을 강화하기 위해 HTTP 응답 헤더를 다양한 방식으로 사용할 수 있다.

<br/>

##### 기본 보안 헤더

Spring Security는 보안 기본값을 제공하기 위해 보안 관련 HTTP 응답 헤더의 기본 세트를 제공한다.

```
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: DENY
X-XSS-Protection: 0
```
- `Strict-Transport-Security` 는 HTTPS 요청에서만 추가된다.

<br/>

##### Cache Control

Spring Security의 기본값은 사용자의 콘텐츠를 보호하기 위해 캐싱을 비활성화하는 것이다. 기본적으로 전송되는 캐시 제어 헤더는 다음과 같다.

```
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
```

<br/>

##### Content Type Options

콘텐츠 스니핑을 통해 악의적인 사용자가 XSS 공격을 수행할 수 있다. 예를 들어, 악의적인 사용자는 유효한 JavaScript 파일이기도 한 포스트스크립트 문서를 만들어 XSS 공격을 수행할 수 있다.

- Content Sniffing : 브라우저가 파일의 타입을 추측해서 동작시키는 것
- XSS (Cross-Site Scription) : 공격자가 다른 사람의 브라우저에서 악성 스크립트를 실행시키는 공격

<br/>

위와 같은 공격을 막기 위해 Spring Security는 HTTP 응답에 다음 헤더를 추가하여 콘텐츠 스니핑을 비활성화한다.

```
X-Content-Type-Options: nosniff
```

<br/>

##### HTTP Strict Transport Security (HSTS)

`mybank.example.com` 과 같이 `https` 프로토콜을 생략해서 웹사이트를 입력할 때, 초기 HTTP 요청을 가로채서 `https://mibank.example.com` 으로 리디렉션 할 수 있다.

이처럼 많은 사용자가 `https` 프로토콜을 생략하기 때문에 **HTTP Strict Transport Security (HSTS)** 가 만들어졌다.

이를 막기 위해 Spring Security 에서는 브라우저에 도메인을 1년 동안 HSTS 호스트로 취급하도록 지시하는 다음 헤더를 추가한다.

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

<br/>

##### X-Frame-Options

웹사이트가 프레임에 추가되도록 허용하는 것은 보안 문제가 될 수 있다. 예를 들어, 교묘한 CSS 스타일링을 사용하여 사용자가 의도하지 않은 것을 클릭하도록 유도할 수 있다. 이러한 공격을 클릭재킹이라고 한다.

클릭재킹을 해결하는 방법은 `X-Frame-Options` 헤더를 사용하는 것이다. 기본적으로 Spring Security는 다음 헤더를 사용하여 iframe 내에서 페이지 렌더링을 비활성화한다.
```
X-Frame-Options: DENY
```

<br/>

##### Content Security Policy (CSP)

콘텐츠 보안 정책(CSP)은 웹 애플리케이션이 크로스 사이트 스크립팅(XSS)과 같은 콘텐츠 삽입 취약점을 완화하는 데 사용할 수 있는 메커니즘이다. CSP는 웹 애플리케이션 작성자가 웹 애플리케이션이 리소스를 로드할 것으로 예상하는 소스에 대해 클라이언트에게 선언하고 궁극적으로 알리는 기능을 제공하는 선언적 정책이다.

예를 들어, 웹 애플리케이션은 응답에 다음 헤더를 포함하여 특정 신뢰할 수 있는 소스에서 스크립트를 로드할 것으로 예상한다고 선언할 수 있다.
```
Content-Security-Policy: script-src https://trustedscripts.example.com
```

<br/>

## Reference
---

[Spring Security Docs](https://docs.spring.io/spring-security/reference/index.html)
