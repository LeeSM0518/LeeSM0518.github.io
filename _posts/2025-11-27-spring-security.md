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

#### HTTP 요청
---

정적 리소스를 포함한 모든 HTTP 기반 통신은 TLS를 사용하여 보호되어야 한다. Spring Security는 클라이언트가 HTTP를 사용할 때 HTTPS로 리디렉션하도록 구성할 수 있다.

<br/>

## Architecture (Servlet Applications)
---

서블릿 기반 애플리케이션 내에서 Spring Security의 고수준 아키텍처에 대해 설명한다.

<br/>

### FilterChain
---

Spring Security의 서블릿 지원은 서블릿 필터를 기반으로 하므로, 먼저 필터의 일반적인 역할을 살펴보는 것이 좋다. 다음 이미지는 단일 HTTP 요청에 대한 핸들러의 일반적인 계층화를 보여준다.

![spring-security1](/assets/img/spring-security1.png)

<br/>

클라이언트가 애플리케이션에 요청을 보내면, 컨테이너는 요청 URI의 경로를 기반으로 `HttpServletRequest` 를 처리해야 하는 `Filter` 인스턴스와 `Servlet` 을 포함하는 `FilterChain` 을 생성한다.

Spring MVC 애플리케이션에서 `Servlet` 은 `DispatcherServlet` 의 인스턴스이다. 하나의 `Servlet` 은 최대 하나의 `HttpServletRequest` 와 `HttpServletResponse` 를 처리할 수 있다. 그러나 둘 이상의 `Filter` 를 사용하여 다음을 수행할 수 있다.

- 다운스트림 `Filter` 인스턴스나 `Servlet` 이 호출되지 않도록 방지한다. 이 경우 `Filter` 는 일반적으로 `HttpServletResponse` 를 작성한다.
- 다운스트림 `Filter` 인스턴스와 `Servlet` 이 사용하는 `HttpServletRequest` 또는 `HttpServletResponse` 를 수정한다.

<br/>

**FilterChain 사용 예제**

```kotlin
@Throws(IOException::class, ServletException::class)
override fun doFilter(
	request: ServletRequest?,
	response: ServletResponse?,
	chain: FilterChain
) {
  // 애플리케이션의 나머지 부분 전에 무언가를 수행
  chain.doFilter(request, response) // 애플리케이션의 나머지 부분 호출
  // 애플리케이션의 나머지 부분 후에 무언가를 수행
}
```

- `Filter` 는 다운스트림 `Filter` 인스턴스와 `Servlet` 에만 영향을 미치므로, 각 `Filter` 가 호출되는 순서가 매우 중요하다.

<br/>

### DelegatingFilterProxy
---

Spring은 서블릿 컨테이너의 생명주기와 Spring의 `ApplicationContext` 사이를 연결할 수 있게 해주는 `DelegatingFilterProxy` 라는 `Filter` 구현을 제공한다.

서블릿 컨테이너는 자체 표준을 사용하여 `Filter` 인스턴스를 등록할 수 있지만, Spring에서 정의한 빈은 인식하지 못한다. 표준 서블릿 컨테이너 메커니즘을 통해 `DelegatingFilterProxy` 를 등록하되, 모든 작업을 `Filter` 를 구현하는 Spring 빈을 위임할 수 있다.

![spring-security2](/assets/img/spring-security2.png)

<br/>

`DelegatingFilterProxy` 는 `ApplicationContext` 에서 Bean Filter(0)을 조회한 다음 Bean Filter(0)을 호출한다. 다음은 `DelegatingFilterProxy` 의 의사 코드를 보여준다.

```kotlin
fun doFilter(
	request: ServletRequest,
	response: ServletResponse,
	chain: FilterChain
) {
	val delegate: Filter = getFilterBean(someBeanName) // (1)
	delegate.doFilter(request, response) // (2)
}
```

1. Spring 빈으로 등록된 Filter를 지연 로딩한다.
2. Spring 빈에 작업을 위임한다.

<br/>

### FilterChainProxy
---

Spring Security의 서블릿 지원은 `FilterChainProxy` 내에 포함되어 있다. `FilterChainProxy` 는 Spring Security가 제공하는 특별한 `Filter` 로, SecurityFilterChain을 통해 많은 `Filter` 인스턴스에 위임할 수 있게 한다.

`FilterChainProxy` 는 빈이므로 일반적으로 `DelegatingFilterProxy` 로 래핑된다. 

![spring-security3](/assets/img/spring-security3.png)

<br/>

### SecurityFilterChain
---

`SecurityFilterChain` 은 `FilterChainProxy` 가 현재 요청에 대해 호출해야 하는 Spring Security `Filter` 인스턴스를 결정하는 데 사용된다.

![spring-security4](/assets/img/spring-security4.png)

<br/>

`SecurityFilterChain` 의 보안 필터는 일반적으로 빈이지만, DelegatingFilterProxy 대신 `FilterChainProxy` 에 등록된다.

`FilterChainProxy` 는 서블릿 컨테이너나 DelegatingFilterProxy에 직접 등록하는 것에 비해 여러 가지 이점을 제공한다.

첫째, Spring Security의 모든 서블릿 지원에 대한 시작점을 제공한다. 그런 이유로, Spring Security의 서블릿 지원 문제를 해결하려고 할 때 `FilterChainProxy` 에 디버그 포인트를 추가하는 것이 좋은 시작점이다.

둘째, `FilterChainProxy` 는 Spring Security 사용의 중심이므로 선택 사항으로 보이지 않는 작업을 수행할 수 있다. 

또한, `SecurityFilterChain` 이 언제 호출되어야 하는지 결정하는 데 많은 유연성을 제공한다. 서블릿 컨테이너에서 `Filter` 인스턴스는 URL만을 기반으로 호출된다. 그러나 `FilterChainProxy` 는 `RequestMatcher` 인터페이스를 사용하여 `HttpServletRequest` 의 모든 것을 기반으로 호출을 결정할 수 있다.

![spring-security5](/assets/img/spring-security5.png)

 - `FilterChainProxy` 는 어떤 `SecuriyFilterChain` 을 사용해야 하는지 결정한다. 

<br/>

### Security Filters
---

보안 필터는 `SecurityFilterChain` API를 사용하여 `FilterChainProxy` 에 삽입된다. 이러한 필터는 취약점 공격 방어, 인증, 인가 등 다양한 목적으로 사용될 수 있다.

필터는 적절한 시점에 호출되도록 호출되도록 보장하기 위해 특정 순서로 실행된다. 예를 들어 인증을 수행하는 필터는 인가를 수행하는 필터보다 먼저 호출되어야 한다. 

<br/>

보안 필터는 대부분 `HttpSecurity` 인스턴스를 사용하여 선언된다.

```kotlin
import org.springframework.security.config.web.servlet.invoke

@Configuration
@EnableWebSecurity
class SecurityConfig {
  
  @Bean
  fun filterChain(http: HttpSecurity): SecurityFilterChain {
    http {
      csrf { }
      httpBasic { }
      formLogin { }
      authorizeHttpRequests {
        authorize(anyRequest, authenticated)
      }
    }
    return http.build()
  }
}
```

위 구성은 다음과 같은 `Filter` 순서를 생성한다.
1. `CsrfFilter` 가 CSRF 공격으로부터 보호하기 위해 호출된다.
2. `AuthenticationFilter` 가 요청을 인증하기 위해 호출된다.
3. `AuthorizationFilter` 가 요청을 인가하기 위해 호출된다.

<br/>

**필터 체인에 필터 추가하기**

`SecurityFilterChain` 에 사용자 정의 `Filter` 를 추가하고 싶을 수 있습니다.

`HttpSecurity` 는 필터를 추가하기 위한 세 가지 메서드를 제공한다.

- `#addFilterBefore(Filter, Class<?>)` : 다른 필터 앞에 필터 추가
- `#addFilterAfter(Filter, Class<?>)` : 다른 필터 뒤에 추가
- `#addFilterAt(Filter, Class<?>)` : 다른 필터를 자신의 필터로 교체

<br/>

**사용자 정의 필터 추가**

자체 필터를 만드는 경우, 필터 체인에서 해당 위치를 결정해야 한다. 필터 체인에서 발생하는 주요 이벤트는 다음과 같다.

1. `SecurityContext` 가 세션에서 로드된다.
2. 요청이 일반적인 취약점 공격으로부터 보호된다.
3. 요청이 인증된다.
4. 요청이 인가된다.

필터를 배치하기 위해 어떤 이벤트가 발생해야 하는지 고려해라. 다음은 경험 법칙이다.

| 필터가 다음인 경우   | 다음 뒤에 배치                        | 이러한 이벤트가 이미 발생했으므로 |
| ------------ | ------------------------------- | ------------------ |
| 취약점 공격 방어 필터 | `SecurityContextHolderFilter`   | 1                  |
| 인증 필터        | `LogoutFilter`                  | 1, 2               |
| 인가 필터        | `AnonymousAuthenticationFilter` | 1, 2, 3            |

<br/>

예를 들어, 테넌트 ID 헤더를 가져오고 현재 사용자가 해당 테넌트에 접근할 수 있는지 확인하는 `Filter` 를 추가하고 싶다고 가정해 보자.

먼저 `Filter` 를 만들어 보자.

```java
public class TenantFilter implements Filter {
  
  @Override
  public void doFilter(
    ServletRequest servletRequest,
    ServletResponse servletResponse,
    FilterChain filterChain
  ) throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest) servletRequest;
    HttpServletResponse response = (HttpServletResponse) servletResponse;
    
    String tenantId = request.getHeader("X-Tenant-Id");
    boolean hasAccess = isUserAllowed(tenantId);
    if (hasAccess) {
      filterChain.doFilter(request, response);
      return;
    }
    throw new AccessDeniedException("Access denied");
  }
}
```

<br/>

이제 필터를 `SecurityFilterChain` 에 추가해야 한다.

사용자를 알아야 하므로 인증 필터 뒤에 추가해야 한다. 경험법칙에 따라 체인에서 마지막 인증 필터인 `AnonymousAuthenticationFilter` 뒤에 다음과 같이 추가한다.

```kotlin
@Bean
fun filterChain(http: HttpSecurity): SecurityFilterChain {
  http
    // ...
    .addFilterAfter(TenantFilter(), AnonymousAuthenticationFilter::class.java)
    
  return http.build()
}
```

- `HttpSecurity#addFilterAfter` 를 사용하여 `AnonymousAuthenticationFilter` 뒤에 `TenantFilter` 를 추가

<br/>

**필터를 빈으로 선언하기**

필터를 `@Component` 로 어노테이션하거나 구성에서 빈으로 선언하여 Spring 빈으로 선언하면, Spring Boot가 자동으로 내장 컨테이너에 등록한다. 이로 인해 필터가 두 번 호출될 수 있다.

<br/>

**Spring Security 필터 사용자 정의**

Spring Security 필터를 직접 구성하려는 경우 다음과 같이 `addFilterAt` 를 사용하여 `DSL` 에서 지정할 수 있다.

```kotlin
@Bean
fun filterChain(http: HttpSecurity): SecurityFilterChain {
  val basic = BasicAuthenticationFilter()
  
  http
    // ...
    .addFilterAt(basic, BasicAuthenticationFilter::class.java)
    
  return http.build()
}
```

- 해당 필터가 이미 추가된 경우 Spring Security는 예외를 던진다.

<br/>

### 보안 예외 처리
---

`ExceptionTranslationFilter` 는 `AccessDeniedException` 및 `AuthenticationException` 을 HTTP 응답으로 변환할 수 있게 한다.

`ExceptionTranslationFilter` 는 보안 필터 중 하나로 `FilterChainProxy` 에 삽입된다.

<br/>

다음 이미지는 `ExceptionTranslationFilter` 와 다른 구성요소와의 관계를 나타낸다.

![spring-security6](/assets/img/spring-security6.png)

1. 먼저, `ExceptionTranslationFilter` 는 `FilterChain.doFilter(request, response)` 를 호출하여 애플리케이션의 나머지 부분을 호출한다.
2. 사용자가 인증되지 않았거나 `AuthenticationException` 인 경우, 인증을 시작한다.
	1. `SecurityContextHolder` 가 지워진다.
	2. `HttpServletRequest` 가 저장되어 인증을 성공하면 원래 요청을 재생하는 데 사용할 수 있다.
	3. `AuthenticationEntryPoint` 는 클라이언트에서 자격 증명을 요청하는데 사용된다.
3. `AccessDeniedException` 인 경우, 접근 거부된다. `AccessDeniedHanlder` 가 접근 거부를 처리하기 위해 호출된다.

<br/>

`ExceptionTranslationFilter` 의 의사 코드는 다음과 같다.

```java
try {
  filterChain.doFilter(request, response); // (1)
} catch (AccessDeniedException | AuthenticationException ex) {
  if (!authenticated || ex instanceof AuthenticationException) {
    startAuthentication(); // (2)
  } else {
    accessDenied();  // (3)
  }
}
```

1. 애플리케이션의 나머지 부분을 호출한다. 애플리케이션의 다른 부분이 `AuthenticationException` 또는 `AccessDeniedException` 을 던지면 여기서 포착되어 처리된다는 것을 의미한다.
2. 사용자가 인증되지 않았거나 `AuthenticationException` 인 경우, 인증을 시작한다.
3. 그렇지 않으면, 접근 거부된다.

<br/>

### 인증 간 요청 저장
---

보안 예외 처리에서 설명한 대로, 요청에 인증이 없고 인증이 필요한 리소스에 대한 경우, 인증을 성공한 후 요청할 리소스에 대한 요청을 저장해야 한다.

Spring Security에서는 `RequestCache` 구현을 사용하여 `HttpServletRequest` 를 저장함으로써 이를 수행한다.

<br/>

#### RequestCache
---

`HttpServletRequest` 는 `RequestCache` 에 저장된다. 사용자가 성공적으로 인증하면 `RequestCache` 가 원래 요청을 재생하는 데 사용된다.

`RequestCacheAwareFilter` 는 사용자가 인증한 후 저장된 `HttpServletRequest` 를 가져오기 위해 `RequestCache` 를 사용하고, `ExceptionTranslationFilter` 는 `AuthenticationException` 을 감지한 후 사용자를 로그인 엔드포인트로 리디렉션하기 전에 `HttpServletRequest` 를 저장하기 위해 `RequestCache` 를 사용한다.

기본적으로 `HttpSessionRequestCache` 가 사용된다. 아래 코드는 `continue` 라는 매개변수가 있는 경우 저장된 요청에 대해 `HttpSession` 을 확인하는 데 사용되는 `RequestCache` 구현을 사용자 정의하는 방법을 보여준다.

```kotlin
@Bean
open fun springSecurity(http: HttpSecurity): SecurityFilterChain {
  val httpRequestCache = HttpSessionRequestCache()
  httpRequestCache.setMatchingRequestParameterName("continue")
  http {
    requestCache {
      requestCache = httpRequestCache
    }
  }
  
  return http.build()
}
```

<br/>

#### 요청 저장 방지
---

로그인 전에 방문하려고 했던 페이지 대신 항상 사용자를 홈페이지로 리디렉션하려는 경우 이 기능을 끄고 싶을 수 있다.

이를 수행하려면 `NullRequestCache` 구현을 사용할 수 있다.

```kotlin
@Bean
open fun springSecurity(http: HttpSecurity): SecurityFilterChain {
  val nullRequestCache = NullRequestCache()
  http {
    requestCache {
      requestCache = nullRequestCache
    }
  }
  return http.build()
}
```

<br/>

### 로깅
---

Spring Security는 DEBUG 및 TRACE 수준에서 모든 보안 관련 이벤트에 대한 포괄적인 로깅을 제공한다. 이는 보안 조치를 위해 Spring Security가 요청이 거부된 이유에 대한 세부 정보를 응답 본문에 추가하지 않기 때문에 애플리케이션을 디버깅할 때 매우 유용할 수 있다.

모든 보안 이벤트를 기록하도록 애플리케이션을 구성하려면 다음을 애플리케이션에 추가할 수 있다.

```
logging.level.org.springframework.security=TRACE
```
{: file='application.properties'}

{% raw %}
```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- ... -->
  </appendar>
  <!-- ... -->
  <logger name="org.springframework.security" level="trace" additivity="false">
    <appender-ref ref="Console" />
  </logger>
</configuration>
```
{: file='logback.xml'}
{% endraw %}

<br/>

## Reference
---

[Spring Security Docs](https://docs.spring.io/spring-security/reference/index.html)
