---
title: Logging in Spring Boot
date: 2024-11-19 02:00:00 +0900
categories: spring
tags:
  - spring
  - logging
description: Spring Boot 로깅 가이드
---
## 1. Overview

Spring Boot에서 사용할 수 있는 주요 로깅 옵션에 대해 살펴보자.

Logback에 대한 더 자세한 정보는 [A Guide to Logback](https://www.baeldung.com/logback) 에서 확인할 수 있으며, Log4j2에 대한 소개는 [Intro to Log4j2 – Appenders, Layouts and Filters](https://www.baeldung.com/log4j2-appenders-layouts-filters) 에서 확인할 수 있다.

<br/>

## 2. Initial Setup

간단하게 로깅할 수 있는 컨트롤러와 테스트 코드를 작성한다.

```kotlin
@RestController  
class LoggingController {  
  
    private val logger = LoggerFactory.getLogger(LoggingController::class.java)  
  
    @RequestMapping("/logging")  
    fun index(): String {  
        logger.trace("A TRACE Message")  
        logger.debug("A DEBUG Message")  
        logger.info("An INFO Message")  
        logger.warn("A WARN Message")  
        logger.error("An ERROR Message")  
  
        return "Check out the Logs to see the output"  
    }  
  
}
```
{: file='LoggingController.kt'}

```kotlin
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)  
class LoggingControllerTest @Autowired constructor(  
    private val webTestClient: WebTestClient,  
) {  
  
    @Test  
    fun loggingTest() {  
        webTestClient  
            .get()  
            .uri("/logging")  
            .exchange()  
            .expectStatus().isOk  
    }  
  
}
```
{: file='LoggingControllerTest.kt'}

![spring-logger-setup](/assets/img/spring-logger-setup.png)
_실행 결과_

<br/>

## 3. Zero Configuration Logging

`Spring 5(Spring Boot 2.x)` 에서는 Spring Framework의 **spring-jcl 모듈이 제공**되므로 `Spring 4.x(Spring Boot 1.x)` 를 사용할 때만 가져와야 한다.

Spring Boot Starter를 사용하면 대부분의 Starter(ex. spring-boot-starter-web)가 **spring-boot-starter-logging 의존을 포함**하고 있으므로 spring-jcl 를 따로 포함할 필요가 없다.

### 3.1. Default Logback Logging

starter를 사용할 경우 기본적으로 **Logback**을 사용하여 로깅을 수행한다.

![spring-boot-starter-logback](/assets/img/spring-boot-starter-logback.png)
_테스트 실행시 출력되는 로그_

기본 로깅 레벨은 `INFO` 로 설정된다. 즉, `TRACE` 와 `DEBUG` 메시지는 확인할 수 없다. 보이지 않는 로그 레벨을 설정 없이 활성화하기 위해서는 java 실행 명령어에 `-debug` 나 `-trace` 옵션을 추가하면 된다.

```sh
$ java -jar target/spring-boot-logging-0.0.1-SNAPSHOT.jar --trace
```

<br/>

### 3.2. Log Levels

Spring Boot는 환경 변수를 통해 보다 세부적인 로그 수준 설정 기능을 제공한다.

1. VM 옵션을 사용해서 로깅 레벨을 설정할 수 있다.

```
-Dlogging.level.org.springframework=TRACE
-Dlogging.level.com.baeldung=TRACE
```

2. Gradle을 사용할 경우 명령어로 설정할 수 있다.

```sh
./gradlew bootRun -Pargs=--loging.level.org.springframework=TRACE,--logging.level.com.baeldung=TRACE
```

3. application.properties 파일로 설정할 수 있다.

```
logging.level.root=WARN
```

 4. 로깅 구성 파일로 설정할 수 있다.

```xml
<logger name="org.springframework" level="INFO" />
<logger name="com.baeldung" level="INFO" />
```

<br/>

> 동일한 패키지에 대해 서로 다른 로그 수준이 설정된 경우 가장 낮은 수준이 사용된다.
{: .prompt-warning }

<br/>

## 4. Logback Configuration Logging

**다른 색상과 로깅 패턴, 콘솔 및 파일 출력 방식, 거대한 로그 파일 생성을 피하기 위한 적절한 롤링 정책**등에 대한 Logback 구성 방법을 살펴보자.

**resources 경로**에 파일 이름이 다음 중 하나인 경우, Spring Boot는 자동으로 기본 구성을 통해 해당 파일을 로드한다.

- *logback-spring.xml*
- *logback.xml*
- *logback-spring.groovy*
- *logback.groovy*

<br/>

Spring은 `-spring` 이 붙은 파일을 권장하므로 `logback-spring.xml` 파일을 작성해보자.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 로그 파일이 저장될 디렉토리 경로를 설정 -->
    <property name="LOGS" value="./logs" />

    <!-- 콘솔에 로그를 출력하는 Appender 설정 -->
    <appender name="Console"
        class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %white(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %yellow(%C{1}): %msg%n%throwable
            </Pattern>
        </layout>
    </appender>

    <!-- 파일에 롤링 로그를 기록하는 Appender 설정 -->
    <appender name="RollingFile"
        class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 로그 파일의 기본 경로 및 이름 설정 -->
        <file>${LOGS}/spring-boot-logger.log</file>
        <encoder
            class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!-- 로그 메시지의 포맷을 정의 -->
            <Pattern>%d %p %C{1} [%t] %m%n</Pattern>
        </encoder>

        <!-- 롤링 정책 설정: 일 단위 롤오버 및 파일 크기가 10MB에 도달하면 롤오버 -->
        <rollingPolicy
            class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 롤오버된 파일의 이름 패턴 설정 -->
            <fileNamePattern>${LOGS}/archived/spring-boot-logger-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- 단일 로그 파일의 최대 크기 설정 -->
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>
    
    <!-- 루트 로거 설정: 모든 로그를 INFO 레벨로 기록 -->
    <root level="info">
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </root>

    <!-- "com.devooks.backend" 패키지의 로그를 TRACE 레벨로 기록 -->
    <logger name="com.devooks.backend" level="trace" additivity="false">  
	    <appender-ref ref="RollingFile" />  
	    <appender-ref ref="Console" />  
	</logger>

</configuration>
```
{: file='src/main/resources/logback-spring.xml'}

![logback-example](/assets/img/logback-example2.png)
_실행 결과_

TRACE와 DEBUG 메시지가 기록되는 것을 확인할 수 있고, 전반적인 콘솔 패턴이 이전과 달라졌다.
또한 현재 경로 아래에 생성된 **/logs 폴더의 파일에 로그를 기록**하고 **롤링 정책을 통해 보관**된다.

<br/>

---

## Reference

<https://www.baeldung.com/spring-boot-logging>