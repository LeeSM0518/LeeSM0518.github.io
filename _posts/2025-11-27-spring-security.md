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

## Reference
---

[Spring Security Docs](https://docs.spring.io/spring-security/reference/index.html)
