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

#### 3.1.3. 여러 개의 모델 다루기
---

**왜 여러 모델이 필요할까?**

- 복잡한 추론에는 고성능 모델, 단순 작업엔 저렴한 모델
- 한 서비스가 다운될 때를 대비한 폴백(fallback)
- A/B 테스트
- 사용자에게 모델 선택권 제공
- 기능 틍화 모델 조합(코드, 창작 등)

<br/>

**설정 방법**

먼저 기본 `ChatClient.Builder` 자동 설정을 끈다.

```
spring.ai.chat.client.enabled=false
```

<br/>

그 후 필요한 모델마다 `ChatClient` 를 직접 만들어 빈으로 등록한다.

```kotlin
@Configuration  
class ChatClientConfig {  
  @Bean  
  fun openAiChatClient(chatModel: OpenAiChatModel): ChatClient = 
    ChatClient.create(chatModel)  
    
  @Bean  
  fun ollamaChatClient(chatModel: OllamaChatModel): ChatClient = 
    ChatClient.create(chatModel)  
}
```

<br/>

컨트롤러, 서비스 등에서는 `@Qualifier` 로 원하는 클라이언트를 주입받는다.

<br/>

#### 3.1.4. Fluent API
---

```kotlin
chatClient
	.prompt()               // 프롬프트 빌드 시작
	.system("시스템 메시지")   // 시스템 메시지
	.user("유저 메시지")      // 사용자 메시지
	.call()                // 동기 호출
	.content()             // 순수 텍스트 결과
```
- `prompt()` 오버로드
	- `prompt()` : 빈 프롬프트로 시작 후 fluent 방식으로 구성
	- `prompt(Prompt p)` : 미리 만든 `Prompt` 객체 사용
	- `prompt(String s)` : 편의 메서드. 문자열을 곧바로 user 메시지로 사용
- 응답 포맷 선택
	- `.call().content()` : String 반환, 순수 답변
	- `.call().chatResponse()` :  `ChatResponse` 반환, 토큰 수 등 메타데이터 포함
	- `.call().entity(Class<T>)` : `T` , 문자열을 자바 객체로 매핑
	- `.stream().content()` : `Flux<String>` , 토큰이 생성되는 대로 스트림

<br/>

#### 3.1.5. Prompt Templates
---

`ChatClient` 의 fluent API를 사용하면 사용자 메시지와 시스템 메시지를 템플릿 형태로 지정할 수 있으며, 런타임에 변수 값이 치환된다.

```kotlin
val answer: String =
	ChatClient
		.create(chatModel)
		.prompt()
		.user { u -> 
			u.text("Tell me the names of 5 movies whose soundtrack was composed by {composer}")
			.param("composer", "John Williams")
		}
		.call()
		.content()
```

<br/>

다른 템플릿 엔진을 쓰고 싶다면 `TemplateRenderer` 인터페이스를 구현한 커스텀 클래스를 `ChatClient` 에 넘기면 된다.

예를 들어 기본 구분자인 `{}` 대신 `<>` 를 쓰고 싶다면 다음처럼 설정할 수 있다.
```kotlin
val answer =
	chatClient  
	  .prompt()  
	  .user { userText ->  
	    userText  
	      .text("Tell me the names of 5 movies whose soundtrack was composed by <composer>")  
	      .param("composer", "John Williams")  
	  }.templateRenderer(  
	    StTemplateRenderer  
	      .builder()  
	      .startDelimiterToken('<')  
	      .endDelimiterToken('>')  
	      .build()  
	  )  
	  .call()  
	  .content()
```

<br/>

#### 3.1.6. 기본값 활용하기
---

`@Configuration` 클래스에서 기본(system) 메시지를 설정해 두면, 런타임 코드에서는 user 메시지만 넘겨주면 되므로 훨씬 간결해진다.

<br/>

**기본 System 텍스트**

다음과 같이 설정하면 이후 컨트롤러에서 system 메시지를 넣을 필요가 없다.

```kotlin
@Configuration  
class Config {  
  @Bean  
  fun chatClient(builder: ChatClient.Builder): ChatClient =  
    builder  
      .defaultSystem("You are a friendly chat bot that answers question in the voice of a Pirate")  
      .build()  
}
```

<br/>

**파라미터를 포함한 기본 System 텍스트**

설정 시점이 아니라 호출 시점에 값을 넣고 싶다면, system 메시지에 플레이스홀더를 넣는다.

```kotlin
@Configuration  
class Config {  
  @Bean  
  fun chatClient(builder: ChatClient.Builder): ChatClient =  
    builder  
      .defaultSystem("You are a friendly chat bot that answers question in the voice of a {voice}")  
      .build()  
}
```

```kotlin
@RestController  
class ChatController(  
  chatClientBuilder: ChatClient.Builder,  
) {  
  private val chatClient = chatClientBuilder.build()
  
  @GetMapping("/ai")  
  suspend fun completion(  
    message: String,  
    voice: String,  
  ) = mapOf(  
    "completion" to  
      chatClient  
        .prompt()  
        .system { sp -> sp.param("voice", voice) } 
        .user(message)
        .call()
        .content()
  )  
}
```


<br/>


## Reference
---

[Spring AI: Reference Doc.](https://docs.spring.io/spring-ai/reference/index.html)