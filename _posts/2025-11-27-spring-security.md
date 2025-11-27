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

## Reference
---

[Spring Security Docs](https://docs.spring.io/spring-security/reference/index.html)
