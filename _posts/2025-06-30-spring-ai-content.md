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

**그 밖의 기본값들**

| **메서드**                                                     | **용도**                             |
| ----------------------------------------------------------- | ---------------------------------- |
| defaultOptions(ChatOptions)                                 | 공통/모델별 옵션 지정(예: OpenAiChatOptions) |
| defaultFunction(String name, String desc, Function<I,O> fn) | 프롬프트에서 호출할 Java 함수 기본 등록           |
| defaultFunctions(String…​ names)                            | 컨테이너에 정의된 다수의 함수 빈 이름 등록           |
| defaultUser(...)                                            | 기본 user 텍스트(또는 리소스/람다) 지정          |
| defaultAdvisors(Advisor…​)                                  | RAG 등 프롬프트 보강용 Advisor 기본 등록       |
| defaultAdvisors(Consumer\<AdvisorSpec>)                     | Advisor 여러 개를 람다로 한 번에 설정          |

<br/>

런타임에서는 default 접두어가 없는 동일 메서드로 언제든 덮어쓸 수 있다.

```kotlin
chatClient
	.prompt()
	.functions("weatherFunction")
	.advisors(SimpleLoggerAdvisor())
	// ...
```

<br/>

#### 3.1.7. Advisors
---

Advisors API는 Spring에서 AI 기반 상호작용을 인터셉트하고 수정 및 향상시키는 유연하고 강력한 방법을 제공한다.

AI 모델에 사용자 텍스트를 보낼 때, 프롬프트에 컨텍스트 데이터를 덧붙이는 패턴이 일반적이다.

컨텍스트 데이터의 대표적인 두 가지 유형은 다음과 같다.
1. **자체 데이터** : 모델이 학습하지 못한 사내 데이터 등. 프롬프트에 포함하면 모델이 본래 지식보다 이 데이터를 우선시한다.
2. **대화 히스토리** : LLM API는 상태가 없으므로, "내 이름은 ~야" 같은 정보를 매 요청마다 함께 보내야 이후 질문에 반영된다.

<br/>

**ChatClient에서 Advisor 설정**

`ChatClient` 의 fluent API에는 Advisor 설정용 `AdvisorSpec` 인터페이스가 있다.

```java
interface AdvisorSpec {
    AdvisorSpec param(String k, Object v);      // 파라미터 하나
    AdvisorSpec params(Map<String, Object> p);  // 파라미터 여러 개
    AdvisorSpec advisors(Advisor... advisors);  // Advisor 추가(가변 인수)
    AdvisorSpec advisors(List<Advisor> advisors);
}
```

> **중요** : 체인에 추가한 순서가 곧 실행 순서이다. 앞선 Advisor가 프롬프트/컨텍스트를 변경하면, 이후 Advisor는 그 결과를 입력으로 받는다.
{: .prompt-danger }

<br/>

```kotlin
chatClient  
  .prompt()  
  .advisors(  
    MessageChatMemoryAdvisor.builder(chatMemory).build(),   // 1. 대화 기록 추가
    QuestionAnswerAdvisor.builder(vectorStore).build()      // 2. RAG 검색
  ).user(userInput)  
  .call()  
  .content()
```
- `MessageChatMemoryAdvisor` 가 먼저 실행되어 대화 히스토리를 삽입
- `QuestionAnswerAdvisor` 가 그 히스토리를 포함한 질문을 기반으로 검색을 수행

<br/>

**로깅 Advisor**

`SimpleLoggerAdvisor` 는 요청 및 응답 데이터를 로깅하는 Advisor이다. 디버깅이나 모니터링에 유용하다.

```kotlin
ChatClient
	.create(chatModel)
	.prompt()
	.advisor(SimpleLoggerAdvisor())
	.user("Tell me a joke?")
	.call()
	.chatResponse()
```

<br/>

로그 활성화 방법은 다음과 같다.

```properties
logging.level.org.springframework.ai.chat.client.advisor=DEBUG
```

<br/>

#### 3.1.8. Chat Memory
---

`ChatMemory` 인터페이스는 대화 기록 저장소를 나타내며, 다음과 같은 메서드를 제공한다.

1. 메시지 추가
2. 메시지 조회
3. 대화 히스토리 초기화

<br/>

현재 기본으로 제공되는 구현은 다음과 같다.

- `MessageWindowChatMemory`
	- 최대 N개(default: 20)의 메시지를 보관하는 window 방식
	- 한도를 넘으면 오래된 메시지가 제거되지만, system 메시지는 보존된다.
	- 새로운 system 메시지가 들어오면 이전 system 메시지들은 모두 제거되어 가장 최신 컨텍스트만 남도록 한다.
	- 이렇게 하면 메모리 사용량을 제한하면서도 대화 흐름에 필요한 최신 정보는 유지할 수 있다.

<br/>

#### 3.1.9. 구현 참고 노트
---

- **ChatClient에서 명령형(Imperative)과 리액티브(Reactive) 프로그래밍 모델을 함께 쓰는 점**이 이 API의 독특한 특징입니다. 대부분의 애플리케이션은 두 모델 중 하나만 쓰지만, 여기서는 둘을 동시에 사용합니다.
    
- **모델(Model) 구현체의 HTTP 클라이언트 설정을 커스터마이즈할 때는** RestClient와 WebClient **둘 다** 설정해야 합니다.
    
- **Spring Boot 3.4 버그**로 인해 spring.http.client.factory=jdk 속성을 반드시 지정해야 합니다. 지정하지 않으면 기본값이 reactor로 설정되어 ImageModel 같은 일부 AI 워크플로가 깨집니다.
    
- **스트리밍 기능**은 **Reactive 스택**에서만 지원됩니다. 따라서 명령형(서블릿) 애플리케이션이라도 스트리밍을 쓰려면 spring-boot-starter-webflux 의존성을 추가해야 합니다.
    
- **비-스트리밍 호출**은 **Servlet 스택**에서만 지원됩니다. 리액티브 애플리케이션도 이러한 호출을 위해 spring-boot-starter-web을 포함해야 하며, 일부 호출이 블로킹된다는 점을 감안해야 합니다.
    
- **툴 호출(tool calling)은 명령형** 방식으로 동작하므로 블로킹 워크플로를 유발합니다. 이 때문에 Micrometer 관측(span)이 부분적으로 끊기거나 누락될 수 있습니다(예: ChatClient span과 tool-calling span이 연결되지 않아 첫 번째 span이 완료되지 않고 남는 현상).
    
- **내장 Advisor들은** 표준(블로킹) 호출에서는 블로킹 방식으로, 스트리밍 호출에서는 논블로킹 방식으로 동작합니다. 스트리밍 시 Advisor에 사용되는 **Reactor Scheduler**는 각 Advisor 클래스의 Builder에서 설정할 수 있습니다.

<br/>

## 4. Advisors API
---




## Reference
---

[Spring AI: Reference Doc.](https://docs.spring.io/spring-ai/reference/index.html)