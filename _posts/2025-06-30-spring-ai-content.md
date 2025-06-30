---
title: Spring AI
date: 2025-06-30 09:00:00 +0900
categories: spring
tags:
  - spring
  - ai
  - rag
  - docs
description: Spring AI 설정 및 사용 가이드
---

## 1. Ollama 환경 설정
---

brew를 사용해 `ollama` 를 설치한다.

```sh
brew install ollama
```

<br/>

ollama를 실행한다.

```sh
ollama start
```

<br/>

ollama로 deepseek-r1 모델을 다운받고 실행한다.

```sh
ollama run deepseek-r1
```

<br/>

## 2. 프로젝트 설정
---

Spring AI 의존을 추가한다.

```kotlin
extra["springAiVersion"] = "1.0.0"

dependencyManagement {  
    imports {  
        mavenBom("org.springframework.ai:spring-ai-bom:${property("springAiVersion")}")  
    }  
}

dependencies {   
    implementation("org.springframework.ai:spring-ai-starter-model-ollama")  
}
```

<br/>

## 3. Chat Client API
---

`ChatClient` 는 AI 모델과 연동할 수 있는 fluent API를 제공하며, 동기식 및 스트리밍식 프로그램 모델을 모두 지원한다.

fluent API는 AI 모델에 입력으로 전달할 프롬프트를 단계적으로 조립할 수 있는 메서드를 제공한다.

`Prompt` 는 여러 개의 메시지로 구성되며, AI 모델은 크게 두 종료의 메시지를 처리한다.

1. `user` : 사용자로부터 직접 입력되는 메시지
2. `system` : 시스템이 대화를 안내하기 위해 삽입하는 메시지

메시지 안에는 런타임 시점에 값이 치환되는 플레이스홀더(ex. {name})를 넣어, 사용자 입력에 맞춰 AI 응답을 맞춤화할 수 있다.

또한 어떤 AI 모델을 쓸지, 창의성(temperature) 같은 프롬프트 옵션도 지정할 수 있다.

<br/>

### 3.1. ChatClient 생성
---

#### 3.1.1. ChatClient.Builder 로 직접 빌드
---

`ChatClient` 인스턴스는 `ChatClient.Builder` 를 통해 만든다.

<br/>

#### 3.1.2. Spring Boot 자동 설정 사용
---

Spring AI의 Spring Boot Auto-configuration을 사용하면, `ChatClient.Builder` 빈이 자동으로 등록된다. 이를 주입받아 간단히 사용할 수 있다.

```kotlin
@RestController  
class ChatController(  
  chatClientBuilder: ChatClient.Builder,  
) {  
  private val chatClient = chatClientBuilder.build()  
  
  @GetMapping("/chat")  
  suspend fun generation(userInput: String): Flow<String> =  
    chatClient  
      .prompt()  
      .user(userInput)  
      .stream()  
      .content()  
      .asFlow()  
}
```

<br/>


## Reference
---

[Spring AI: Reference Doc.](https://docs.spring.io/spring-ai/reference/index.html)