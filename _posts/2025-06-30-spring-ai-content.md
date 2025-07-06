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

Spring AI의 Advisors API는 Spring 애플리케이션에서 AI 기반 상호작용을 인터셉트 하고, 수정 및 향상시킬 수 있는 유연하고 강력한 방식이다. Advisors API를 활용하면 개발자는 더 정교하고 재사용 가능하며 유지 보수가 쉬운 AI 컴포넌트를 만들 수 있다.

<br/>

주요 이점은 다음과 같다.

- 반복적으로 등장하는 생성형 AI 패턴을 캡슐화
- LLM과 주고 받는 데이터를 변환
- 다양한 모델 및 사용 사례 간 이식성 확보

<br/>

**Advisor 구성 예시**

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
- Advisor는 `defaultAdvisors()` 로 빌드 타임에 등록해 두는 것을 권장한다.
- Advisors는 Observability 스택에도 참여하므로, 실행 메트릭 및 트레이스를 확인할 수 있다.

<br/>

### 4.1. Core Components
---

![spring-ai6](/assets/img/spring-ai6.jpg)

- 비스트리밍 : `CallAroundAdvisor` , `CallAroundAdvisorChain`
- 스트리밍 : `StreamAroundAdvisor` , `StreamAroundAdvisorChain`
- 미가공 Prompt 요청 : `AdvisedRequest`
- Chat Completion 응답 : `AdvisedResponse`
- 어드바이저 체인 전반에서 상태를 공유할 `adviseContext: Map<String, Obj>`
- 체인 내 실행 순서 결정 : `getOrder` (값이 낮을수록 먼저 실행)
- 고유 어드바이저 이름 : `getName`

<br/>

**어드바이저 체인 동작 방식**

![spring-ai7](/assets/img/spring-ai7.jpg)

Spring AI가 사용자의 Prompt로부터 `AdvisedRequest` 를 생성하고, 빈 `AdvisorContext` 객체를 준비한다.

1. 체인의 각 어드바이저가 요청을 순차적으로 처리 및 수정한다.
   (또는 다음 엔티티 호출을 생략해 요청을 차단할 수도 있으며, 이 경우 어드바이저가 직접 응답을 작성한다)
2. 프레임워크가 자동으로 추가한 마지막 어드바이저가 LLM에 실제 요청을 전송한다.
3. LLM 응답이 어드바이저 체인을 거꾸로 타고 올라가며 `AdvisedResponse` 로 변환되고, 공유된 `AdvisorContext` 가 함께 전달된다.
4. 각 어드바이저가 응답을 검사 및 수정할 수 있다.
5. 최종 `AdvisedResponse` 에서 ChatCompletion 이 추출되어 클라이언트에게 반환된다.

<br/>

### 4.2. Examples
---

#### 4.2.1. Logging Advisor
---

체인에서 다음 어드바이저를 호출하기 전에 `AdvisedRequest` 를, 호출이 끝난 후에 `AdvisedResponse` 를 로그에 남긴다.

```java
public class SimpleLoggerAdvisor implements CallAroundAdvisor, StreamAroundAdvisor {

    private static final Logger logger = LoggerFactory.getLogger(SimpleLoggerAdvisor.class);

    @Override
    public String getName() { 
        return this.getClass().getSimpleName();
    }

    @Override
    public int getOrder() { 
        return 0;   // 값이 낮을수록 먼저 실행
    }

    @Override
    public AdvisedResponse aroundCall(AdvisedRequest advisedRequest, CallAroundAdvisorChain chain) {

        logger.debug("BEFORE: {}", advisedRequest);

        AdvisedResponse advisedResponse = chain.nextAroundCall(advisedRequest);

        logger.debug("AFTER: {}", advisedResponse);

        return advisedResponse;
    }

    @Override
    public Flux<AdvisedResponse> aroundStream(AdvisedRequest advisedRequest, StreamAroundAdvisorChain chain) {

        logger.debug("BEFORE: {}", advisedRequest);

        Flux<AdvisedResponse> advisedResponses = chain.nextAroundStream(advisedRequest);

        return new MessageAggregator().aggregateAdvisedResponse(
                advisedResponses,
                advisedResponse -> logger.debug("AFTER: {}", advisedResponse));
    }
}
```

<br/>

#### 4.2.2. Re-Reading(Re2) Advisor
---

Re-Reading(Re2) 기법은 입력 프롬프트를 보강해 추론 능력을 향상시킨다.

```java
public class ReReadingAdvisor implements CallAroundAdvisor, StreamAroundAdvisor {

    // 프롬프트 보강 로직
    private AdvisedRequest before(AdvisedRequest advisedRequest) {

        Map<String, Object> params = new HashMap<>(advisedRequest.userParams());
        params.put("re2_input_query", advisedRequest.userText());

        return AdvisedRequest.from(advisedRequest)
            .userText("""
                {re2_input_query}
                Read the question again: {re2_input_query}
                """)
            .userParams(params)
            .build();
    }

    @Override
    public AdvisedResponse aroundCall(AdvisedRequest advisedRequest, CallAroundAdvisorChain chain) {
        return chain.nextAroundCall(before(advisedRequest));
    }

    @Override
    public Flux<AdvisedResponse> aroundStream(AdvisedRequest advisedRequest, StreamAroundAdvisorChain chain) {
        return chain.nextAroundStream(before(advisedRequest));
    }

    @Override
    public int getOrder() { 
        return 0;
    }

    @Override
    public String getName() { 
        return this.getClass().getSimpleName();
    }
}
```

<br/>

#### 4.2.3. Spring AI Built-in Advisors
---

| **분류**                 | **어드바이저**                    | **역할**                                                    |
| ---------------------- | ---------------------------- | --------------------------------------------------------- |
| **Chat Memory**        | MessageChatMemoryAdvisor     | 대화 기록을 **메시지 컬렉션** 형태로 프롬프트에 추가(모든 모델이 지원하지는 않음)          |
|                        | PromptChatMemoryAdvisor      | 대화 기록을 **system 텍스트**에 삽입                                 |
|                        | VectorStoreChatMemoryAdvisor | VectorStore에서 기록을 검색해 **system 텍스트**에 삽입                  |
| **Question Answering** | QuestionAnswerAdvisor        | VectorStore 기반 **RAG**(Retrieval-Augmented Generation) 구현 |
| **Content Safety**     | SafeGuardAdvisor             | 유해·부적절 콘텐츠 생성을 방지하는 간단한 필터                                |

<br/>

### 4.3. 구현 참고 노트
---

- **어드바이저를 특정 작업에 집중**시켜 모듈성을 높이세요.
- 필요할 때는 **adviseContext로 어드바이저 간 상태를 공유**하세요.
- 최대한 유연하게 사용하려면 **스트리밍·논-스트리밍 버전을 모두 구현**하세요.
- 데이터 흐름이 올바르게 유지되도록 **체인 내 어드바이저 실행 순서를 신중히 설계**하세요.

<br/>

## 5. Prompts
---

프롬프트는 AI 모델이 특정 출력을 생성하도록 유도하는 입력이다.

<br/>

### 5.1. API Overview: `Prompt`
---

대부분의 경우 `ChatModel` 의 `call()` 메서드는 `Prompt` 인스턴스를 받아 `ChatResponse` 를 반환하도록 사용한다.

`Prompt` 클래스는 여러 개의 `Message` 와 `ChatOptions` 요청 옵션을 담는 컨테이너 역할을 한다. 각 `Message` 는 프롬프트 내에서 고유한 `Role(역할)` 을 가진다. 사용자 질문, AI가 생성한 응답, 배경 정보 등 다양한 요소가 이 Roles 안에 들어갈 수 있다.

<br/>

다음은 축약 버전의 `Prompt` 클래스이다.

```java
public class Prompt implements ModelRequest<List<Message>> {

    private final List<Message> messages;

    private ChatOptions chatOptions;
}
```

<br/>

### 5.2. API Overview: `Message`
---

`Message` 인터페이스는 `Prompt` 의 내용, 여러 메타데이터 속성, `MessageType` 을 하나로 묶어 둔 객체이다.

다음은 인터페이스 정의이다.

```java
public interface Content {

    String getContent();                 // 실제 문자열 콘텐츠
    Map<String, Object> getMetadata();   // 임의의 메타데이터
}

public interface Message extends Content {

    MessageType getMessageType();        // 역할(ROLE)에 해당
}
```

- 멀티모달 메시지 타입은 `MediaContent` 인터페이스도 구현하여 이미지, 오디오 등 `Media` 객체 목록을 제공한다.

<br/>

`Message` 인터페이스의 구현체마다 AI 모델이 처리할 수 있는 다양한 메시지 카테고리를 표현한다.

![spring-ai8](/assets/img/spring-ai8.jpg)

<br/>

**Roles (역할)**

| **역할**            | **설명**                                                                                                                                               |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **System**        | AI의 동작 방식·응답 스타일을 안내하는 지시문.대화를 시작하기 전에 AI에게 규칙·매개변수를 설정해 줍니다.                                                                                        |
| **User**          | 사용자의 입력(질문, 명령, 진술 등).AI의 응답이 이 역할을 기반으로 생성됩니다.                                                                                                      |
| **Assistant**     | AI가 User 입력에 대해 생성한 응답.대화 흐름을 유지하는 핵심 역할로, 이전 Assistant 메시지를 추적해 일관성 있고 맥락에 맞는 대화를 보장합니다.또한 함수/툴 호출 요청 정보를 포함할 수도 있어 계산·데이터 조회 등 특별 기능을 수행할 때 사용됩니다. |
| **Tool/Function** | Assistant 메시지에 의해 호출된 **툴/함수**가 반환하는 추가 정보를 담는 역할입니다.                                                                                                |

<br/>

### 5.3. API Overview: `PromptTemplate`
---

Spring AI에서 프롬프트 템플릿을 위해 핵심적으로 사용하는 클래스가 `PromptTemplate` 이다. 이 클래스는 구조화된 프롬프트를 손쉽게 작성해 AI 모델로 전송할 수 있도록 돕는다.

```java
public class PromptTemplate implements PromptTemplateActions, PromptTemplateMessageActions {

    // 기타 메서드는 생략
}
```

<br/>

`PromptTemplate` 은 `TemplateRenderer` API를 이용해 템플릿을 렌더링한다.

```java
public interface TemplateRenderer extends BiFunction<String, Map<String, Object>, String> {

	@Override
	String apply(String template, Map<String, Object> variables);
}
```

<br/>

`<>` 구분자를 사용하는 예시

```java
PromptTemplate promptTemplate = PromptTemplate.builder()
    .renderer(
        StTemplateRenderer.builder()
            .startDelimiterToken('<')
            .endDelimiterToken('>')
            .build())
    .template("""
            Tell me the names of 5 movies whose soundtrack was composed by <composer>.
            """)
    .build();

String prompt = promptTemplate.render(Map.of("composer", "John Williams"));
```

<br/>

### 5.4. Example Usage
---

다음은 `PromptTemplates` 예제의 간단한 코드이다.

```java
PromptTemplate promptTemplate =
        new PromptTemplate("Tell me a {adjective} joke about {topic}");

Prompt prompt =
        promptTemplate.create(Map.of("adjective", adjective, "topic", topic));

return chatModel.call(prompt).getResult();
```

<br/>

다음은 메시제에 대한 여러 Role을 활용한 예시이다.

```java
String userText = """
    Tell me about three famous pirates from the Golden Age of Piracy and why they did.
    Write at least a sentence for each pirate.
    """;

Message userMessage = new UserMessage(userText);

String systemText = """
  You are a helpful AI assistant that helps people find information.
  Your name is {name}
  You should reply to the user's request with your name and also in the style of a {voice}.
  """;

SystemPromptTemplate systemPromptTemplate = new SystemPromptTemplate(systemText);
Message systemMessage =
        systemPromptTemplate.createMessage(Map.of("name", name, "voice", voice));

Prompt prompt = new Prompt(List.of(userMessage, systemMessage));

List<Generation> response = chatModel.call(prompt).getResults();
```

<br/>

문자열 대신 리소스를 사용하는 예시이다.

```java
@Value("classpath:/prompts/system-message.st")
private Resource systemResource;

SystemPromptTemplate systemPromptTemplate =
        new SystemPromptTemplate(systemResource);
```

<br/>

### 5.5. Prompt Engineering
---

프롬프트의 품질과 구조에 따라 AI 출력의 효과가 크게 달라진다.

<br/>

#### 5.5.1. 효과적인 프롬프트 구성 요소

1. **Instructions** : AI에게 주는 명확하고 직접적인 지시.
2. **External Context** : 답변에 필요한 배경 정보나 추가 지침
3. **User Input** : 사용자의 실제 질문, 요청
4. **Output Indicator** : 원하는 응답 형식(ex. JSON)

또한 예시 질문과 답변을 함께 제공하면 AI가 의도와 형식을 더 잘 파악해 정밀한 답을 내놓는다.

<br/>

#### 5.5.2. 참고할 기술 및 기법
---

Text Summarization 
: 긴 문서를 핵심만 간략히 요약

```
~~내용~~
한 문장으로 요약: 
```
{: file='Prompt'}

<br/>

Question Answering
: 주어진 텍스트에서 정확한 답 추출

```
아래 문맥(Context)을 바탕으로 질문에 답하십시오.  
답변은 짧고 간결하게 작성하세요.  
정답이 확실하지 않으면 "정답을 확신할 수 없음"이라고 답하세요.

문맥: 테플리주맙(Teplizumab)의 기원은 뉴저지의 제약사인 Ortho Pharmaceutical로 거슬러 올라갑니다. 이곳에서 과학자들은 OKT3라 불리는 항체의 초기 버전을 만들어 냈습니다. OKT3는 처음에는 쥐에서 유래한 분자로, T 세포 표면에 결합해 세포의 살상 능력을 제한할 수 있었습니다. 1986년에는 신장 이식 후 장기 거부반응을 예방하기 위해 승인되며, 인체 사용이 허가된 첫 치료용 항체가 되었습니다.

질문: OKT3는 원래 무엇에서 유래했습니까?  
답변:
```
{: file='Prompt'}

<br/>

Text Classification
: 텍스트를 카테고리로 분류

```
다음을 중립, 긍정, 부정 중 하나로 분류하세요.

문장: I think the food was okay.
감정:
```
{: file='Prompt'}

<br/>

Conversation
: 자연스러운 대화 주고받기

```
다음은 AI 연구 보조와 나누는 대화입니다.  
AI의 말투는 기술적이고 과학적입니다.  

Human: Hello, who are you?  
AI: Greeting! I am an AI research assistant. How can I help you today?  
Human: Can you tell me about the creation of blackholes?  
AI:
```
{: file='Prompt'}

<br/>

Code Generation
: 설명을 바탕으로 코드 스니펫 생성

```
departments 테이블, 컬럼 = [DepartmentId, DepartmentName]
students 테이블, 컬럼 = [DepartmentId, StudentId, StudentName]
컴퓨터 공학과(Computer Science Department)에 소속된 모든 학생을 조회하는 MySQL 쿼리를 작성하세요.
```
{: file='Prompt'}

<br/>

Zero-shot, Few-shot Learning
: 예시가 거의 없거나 전혀 없는 상황에서도 추론

```
다음 텍스트를 중립(neutral), 부정적(negative), 긍정적(positive)으로 분류하세요.
텍스트: I think the vacation is okay.
감정:
```
{: file='Prompt'}

```
"Whatpu"는 탄자니아가 원산지인 작고 털이 많은 동물입니다. "whatpu"라는 단어를 사용한 문장의 예시는 다음과 같습니다:
We were traveling in Africa and we saw these very cute whatpus.

"Farduddle"이란 단어는 매우 빠르게 위아래로 뛰어오르는 것을 의미합니다. "farduddle"이 들어간 문장의 예시는 다음과 같습니다:
```
{: file='Prompt'}

<br/>

Chain-of-Thought
: 단계별 사고 과정을 드러내며 응답

```
시장에 가서 사과 10개를 샀습니다. 이웃에게 2개를 주고 수리공에게도 2개를 주었습니다. 그 후 사과를 5개 더 샀고, 1개를 먹었습니다. 남은 사과는 총 몇 개일까요?
Let's think step by step.
```
{: file='Prompt'}

<br/>

ReAct (Reason + Act)
: "생각 -> 행동" 패턴으로 분석 후 실행

```
"콜로라도 조산운동의 동쪽 부분이 확장된 지역의 해발 고도 범위는 얼마인가?"
- Thought 1: “콜로라도 조산운동(Colorado orogeny)을 검색해서 동쪽 부분이 확장된 지역을 찾고, 그 지역의 해발 고도 범위를 찾아야겠다.”
    
- Action 1: 검색[Colorado orogeny]
    
- Observation 1: “콜로라도 조산운동은 콜로라도 및 주변 지역의 조산운동(산 형성 과정)이다.”
    
- Thought 2: “동쪽 부분에 대한 언급이 없다. 동쪽 부분(eastern sector)을 검색해 보겠다.”
    
- Action 2: 검색[eastern sector]
    
- Observation 2: “동쪽 부분은 High Plains 지역으로 뻗어 있으며, Central Plains orogeny라 불린다.”
    
- Thought 3: “동쪽 부분이 High Plains까지 확장되었으므로, High Plains를 검색하여 해발 고도 범위를 찾자.”
    
- Action 3: 검색[High Plains]
    
- Observation 3: “High Plains는 두 개의 별개 지역 중 하나를 나타낸다.”
    
- Thought 4: “정확히 미국의 High Plains를 검색해야겠다.”
    
- Action 4: 검색[High Plains (United States)]
    
- Observation 4: “High Plains 지역의 해발 고도는 동쪽에서 서쪽으로 약 1,800피트에서 7,000피트까지 증가한다.”
    
- Thought 5: “해발 고도는 1,800피트에서 7,000피트이므로, 정답은 1,800에서 7,000피트다.”
    
- Action 5: Finish[1,800 to 7,000 ft]
```
{: file='Prompt'}

<br/>

Framework for Prompt Creation and Optimization
: 마이크로소프트가 제시한 체계적인 프롬프트 설계 및 최적화 가이드

<br/>

## 6. Structured Output
---

Spring AI의 `Structured Output Converters` 는 LLM의 출력을 구조화된 포맷으로 손쉽게 변환할 수 있도록 돕는다. 아래의 다이어그램과 같이, 이 접근법은 LLM의 텍스트 완성(completion) API 엔드포인트를 중심으로 작동한다.

![spring-ai9](/assets/img/spring-ai9.jpg)

- LLM 호출 이전 : 변환기(converter)는 포맷 지침(format instructions)을 프롬프트(prompt)에 추가하여 모델이 원하는 출력 구조를 생성하도록 명확한 안내를 제공한다.
- LLM 호출 이후 : 변환기는 모델이 생성한 텍스트 출력을 받아 구조화된 타입의 인스턴스로 변환한다. 이 변환 과정에서는 텍스트 결과를 파싱하고 JSOM, XML, 도메인 특정 데이터 구조 등으로 대응하는 구조화된 표현으로 매핑하는 작업이 포함된다.

<br/>

인공지늠 모델이 항상 요청된 구조를 정확히 따라 출력하리라는 보장은 없다. 따라서 실제 서비스에서는 모델의 출력이 원하는 구조를 따르고 있는지를 검증(validation)하는 추가적인 절차를 구현하는 것이 바람직하다.

또한 `StructuredOutputConverter` 는 LLM Tool Calling(도구 호출) 방식에서는 사용되지 않는다. 왜냐하면 Tool Calling 방식 자체가 이미 구조화된 출력을 기본적으로 제공하기 때문이다.

<br/>

### 6.1. Structured Output API
---

`StructuredOutputConverter` 인터페이스를 사용하면 AI 모델의 텍스트 기반 출력을 클래스나 값 배열과 같은 구조화된 형태로 손쉽게 얻을 수 있다.

```java
public interface StructuredOutputConverter<T> extends Converter<String, T>, FormatProvider {
}
```

<br/>

![spring-ai10](/assets/img/spring-ai10.jpg)

```
[사용자 입력] --(FormatProvider가 형식 지침 추가)--> [프롬프트 생성] --(LLM 호출)--> [LLM 출력 텍스트] --(Converter가 텍스트를 구조화된 타입으로 변환)--> [구조화된 출력]
```

- 구조화된 출력 API를 사용할 때의 데이터 흐름을 보여준다.

<br/>

이러한 형식 지침은 일반적으로 프롬프트 템플릿(PromptTemplate)을 통해 사용자 입력의 끝에 추가된다.

```java
StructuredOutputConverter outputConverter = ...;
String userInputTemplate = """
    ... 사용자 입력 텍스트 ...
    {format}
    """; // "format" 플레이스홀더가 포함된 사용자 입력.

Prompt prompt = new Prompt(
    new PromptTemplate(
        this.userInputTemplate,
        Map.of(..., "format", outputConverter.getFormat()) 
        // "format" 플레이스홀더를 converter의 포맷으로 치환
    ).createMessage()
);
```

<br/>

#### 6.1.1. 사용 가능한 변환기(Converters)
---

![spring-ai11](/assets/img/spring-ai11.jpg)

<br/>

### 6.2. 변환기(Converters) 사용 방법
---

#### 6.2.1. BeanOutputConverter
---

목표 타입 정의

```java
record ActorsFilms(String actor, List<String> movies) {}
```

<br/>

사용 예시
```java
// 고수준 API 사용
ActorsFilms actorsFilms = ChatClient.create(chatModel).prompt()
    .user(u -> u.text("Generate the filmography of 5 movies for {actor}.")
                .param("actor", "Tom Hanks"))
    .call()
    .entity(ActorsFilms.class);

// 저수준 API 사용
BeanOutputConverter<ActorsFilms> beanOutputConverter =
    new BeanOutputConverter<>(ActorsFilms.class);

String format = beanOutputConverter.getFormat();

String actor = "Tom Hanks";

String template = """
    Generate the filmography of 5 movies for {actor}.
    {format}
    """;

Prompt prompt = new PromptTemplate(template, Map.of("actor", actor, "format", format)).create();

Generation generation = chatModel.call(prompt).getResult();

ActorsFilms actorsFilms = beanOutputConverter.convert(generation.getOutput().getText());
```

<br/>

#### 6.2.2. 생성된 스키마(JSON)에서의 프로퍼티 순서 지정하기
---

JSON 출력에서 속성 순서를 명확하게 지정한다.

```java
@JsonPropertyOrder({"actor", "movies"})
record ActorsFilms(String actor, List<String> movies) {}
```

<br/>

**Generic Bean 타입 지정하기**

보다 복잡한 데이터 구조로 출력받고 싶다면, `ParameterizedTypeReference` 를 사용할 수 있다.

```java
// 고수준 방식
List<ActorsFilms> actorsFilms = ChatClient.create(chatModel).prompt()
    .user("Generate the filmography of 5 movies for Tom Hanks and Bill Murray.")
    .call()
    .entity(new ParameterizedTypeReference<List<ActorsFilms>>() {});

// 저수준 방식
BeanOutputConverter<List<ActorsFilms>> outputConverter = new BeanOutputConverter<>(
    new ParameterizedTypeReference<List<ActorsFilms>>() { });

String format = outputConverter.getFormat();
String template = """
    Generate the filmography of 5 movies for Tom Hanks and Bill Murray.
    {format}
    """;

Prompt prompt = new PromptTemplate(template, Map.of("format", format)).create();

Generation generation = chatModel.call(prompt).getResult();

List<ActorsFilms> actorsFilms = outputConverter.convert(generation.getOutput().getText());
```

<br/>

#### 6.2.3. MapOutputConverter

모델 출력을 Java Map 형태로 변환

```java
Map<String, Object> result = ChatClient.create(chatModel).prompt()
    .user(u -> u.text("Provide me a List of {subject}")
                .param("subject", "an array of numbers from 1 to 9 under the key name 'numbers'"))
    .call()
    .entity(new ParameterizedTypeReference<Map<String, Object>>() {});
```

<br/>

#### 6.2.4. ListOutputConverter
---

모델 출력을 Java List 형태로 변환

```java
List<String> flavors = ChatClient.create(chatModel).prompt()
    .user(u -> u.text("List five {subject}")
                .param("subject", "ice cream flavors"))
    .call()
    .entity(new ListOutputConverter(new DefaultConversionService()));
```

<br/>

### 6.3. 내장 JSON 모드 (Built-in JSON mode)
---

일부 AI 모델은 별도의 내장 옵션을 통해 구조화된 출력을 생성할 수 있도록 지원한다.

<br/>

**OpenAI Structured Outputs**
- `JSON_OBJECT` : 모델이 생성하는 메시지가 유효한 JSON 포맷을 따르도록 보장한다.
- `JSON_SCHEMA` : 사용자가 제공한 JSON Schema에 엄격히 부합하는 응답을 생성하도록 보장한다.

```
spring.ai.openai.chat.options.responseFormat={"type":"json_object"}
```

<br/>

**Ollama**
- 출력의 형식을 JSON으로 명시할 수 있으며, 현재 유일한 지원값은 json이다.

```
spring.ai.ollama.chat.options.format=json
```

<br/>

## 7. Multimodality API
---

멀티모달리티(multimodality)란 모델이 텍스트뿐만 아니라 이미지, 오디오, 영상 등 여러 형태의 데이터를 함께 이해하고 처리할 수 있는 능력을 의미한다.

Spring AI의 Message API는 이러한 멀티모달 언어모델을 쉽게 지원하기 위해 추상화를 제공한다.

![spring-ai11](/assets/img/spring-ai11.jpg)

<br/>

## 8. Chat Model API
---

Chat Model API는 AI 기반의 채팅 완성 기능을 애플리케이션에 쉽게 통합할 수 있도록 제공한다.

이 API는 질문이나 대화 일부(prompt)를 모델에 전달하여 모델이 학습된 데이터를 기반으로 자연어를 이해하고, 그 결과를 응답의 형태로 반환하는 방식으로 작동한다.

Spring AI의 Chat Model API는 다양한 AI 모델과 상호작용하기 위한 간결하고 휴대성이 높은 인터페이스를 제공하며, 모델 전환 시 최소한의 코드 변경만으로 가능하도록 설계되었다.

또한 Prompt(입력 캡슐화), ChatResponse(출력 처리)와 같은 보조 클래스들을 통해 Chat Model API는 AI 모델과의 통신을 통합한다.

<br/>

### 8.1. API 개요
---

#### 8.1.1. ChatModel 인터페이스
---

```java
public interface ChatModel extends Model<Prompt, ChatResponse> {

	default String call(String message) {...}

	@Override
	ChatResponse call(Prompt prompt);
}
```

- 문자열 매개변수를 사용하는 `call()` 메서드는 초기 사용을 간소화하여, Prompt 및 ChatResponse 클래스의 복잡함을 피할 수 있도록 한다.
- Prompt 인스턴스를 전달하고 ChatResponse를 반환하는 `call()` 메서드가 더 일반적이다.

<br/>

#### 8.1.2. StreamingChatModel 인터페이스
---

```java
public interface StreamingChatModel extends StreamingModel<Prompt, ChatResponse> {

    default Flux<String> stream(String message) {...}

    @Override
	Flux<ChatResponse> stream(Prompt prompt);
}
```

- `stream()` 메서드는 ChatModel과 유사하게 String 또는 Prompt를 매개변수로 받아 응답을 리액티브 Flux API를 통해 스트리밍한다.

<br/>

#### 8.1.3. Prompt 인터페이스
---

```java
public class Prompt implements ModelRequest<List<Message>> {

    private final List<Message> messages;

    private ChatOptions modelOptions;

	@Override
	public ChatOptions getOptions() {...}

	@Override
	public List<Message> getInstructions() {...}

    // constructors and utility methods omitted
}
```

- `Prompt` 는 `Message` 객체 목록과 선택적인 모델 요청 옵션을 캡슐화하는 `ModelRequest` 이다.

<br/>

#### 8.1.4. Message 인터페이스
---

```java
public interface Content {

	String getText();

	Map<String, Object> getMetadata();
}

public interface Message extends Content {

	MessageType getMessageType();
}

public interface MediaContent extends Content {

	Collection<Media> getMedia();

}
```

- `Message` 는 프롬프트의 텍스트 콘텐츠, 메타데이터 속성 집합, `MessageType` 을 캡슐화한다.

<br/>

**Spring AI Message API**

![spring-ai13](/assets/img/spring-ai13.jpg)

Chat Completion Endpoint는 대화의 역할에 따라 메시지 유형을 구분한다.

예를 들어 OpenAI는 `system`, `user`, `function` , `assistant` 등의 역할별 메시지 유형을 인식한다.

<br/>

**Chat Options**

AI 모델에 전달될 수 있는 옵션을 나타낸다. `ChatOptions` 는 `ModelOptions` 의 하위 인터페이스로, 다음과 같이 정의된다.

```java
public interface ChatOptions extends ModelOptions {

	String getModel();
	Float getFrequencyPenalty();
	Integer getMaxTokens();
	Float getPresencePenalty();
	List<String> getStopSequences();
	Float getTemperature();
	Integer getTopK();
	Float getTopP();
	ChatOptions copy();

}
```

<br/>

다음은 Spring AI가 ChatModel 실행과 구성을 처리하는 흐름도이다.

![spring-ai14](/assets/img/spring-ai14.jpg)

1. Start-up Configuration : ChatModel/StreamingChatModel이 초기 설정된 옵션이다.
2. Runtime Configuration : Prompt 객체에 런타임 옵션이 포함될 수 있으며, 이는 초기 설정을 덮어쓴다.
3. Option Merging Process : 런타임 옵션이 있을 경우 우선 적용되어 병합된다.
4. Input Processing : 입력을 모델 전용 포맷으로 변환한다.
5. Output Processing : 모델 응답을 ChatResponse 형식으로 변환한다.

<br/>

#### 8.1.5. ChatResponse 인터페이스
---

```java
public class ChatResponse implements ModelResponse<Generation> {

    private final ChatResponseMetadata chatResponseMetadata;
	private final List<Generation> generations;

	@Override
	public ChatResponseMetadata getMetadata() {...}

    @Override
	public List<Generation> getResults() {...}

    // other methods omitted
}
```

- `ChatResponse` 는 AI 모델의 출력 결과를 담고 있으며, 하나의 `Prompt` 요청에 대해 여러 개의 `Generation` 결과를 가질 수 있다.

<br/>

#### 8.1.6. Generation 인터페이스
---

```java
public class Generation implements ModelResult<AssistantMessage> {

	private final AssistantMessage assistantMessage;
	private ChatGenerationMetadata chatGenerationMetadata;

	@Override
	public AssistantMessage getOutput() {...}

	@Override
	public ChatGenerationMetadata getMetadata() {...}

    // other methods omitted
}
```

- `Generation` 클래스는 `ModelResult` 를 상속하며, AI의 출력 메시지 및 관련 메타데이터를 표현한다.

<br/>

## 9.  Chat Model 비교
---

- Multimodality : 다양한 입력 처리 여부
- Tool/Function Calling : 외부 도구 사용이나 함수 호출 가능 여부
- Streaming : 스트리밍 응답 제공 여부
- Retry : 자동 재시도 지원 여부
- Observability : 모니터링 및 디버깅 기능 제공 여부
- Built-in JSON : JSON 형식의 출력 지원 여부
- Local deployment : 로컬 실행 가능 여부
- OpenAI API Compability : 해당 모델이 Open AI API와 호환되는지 여부

| Provider                                                                                                | Multimodality                                     | Tools/Functions                                                    | Streaming                                                          | Retry                                                              | Observability                                                      | Built-in JSON                                                      | Local                                                              | OpenAI API Compatible                                              |  
| ------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------ |  
| [Anthropic Claude](https://docs.spring.io/spring-ai/reference/api/chat/anthropic-chat.html)             | text, pdf, image                                  | O | O | O | O | | | |  
| [Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)              | text, image                                       | O | O | O | O | O | | O |  
| [DeepSeek (OpenAI-proxy)](https://docs.spring.io/spring-ai/reference/api/chat/deepseek-chat.html)       | text                                              | | O | O | O | O | O | O |  
| [Google VertexAI Gemini](https://docs.spring.io/spring-ai/reference/api/chat/vertexai-gemini-chat.html) | text, pdf, image, audio, video                    | O | O | O | O | O | | O |  
| [Groq (OpenAI-proxy)](https://docs.spring.io/spring-ai/reference/api/chat/groq-chat.html)               | text, image                                       | O | O | O | O | | | O |  
| [HuggingFace](https://docs.spring.io/spring-ai/reference/api/chat/huggingface.html)                     | text                                              | | | | | | | |  
| [Mistral AI](https://docs.spring.io/spring-ai/reference/api/chat/mistralai-chat.html)                   | text, image                                       | O | O | O | O | O | | O |  
| [MiniMax](https://docs.spring.io/spring-ai/reference/api/chat/minimax-chat.html)                        | text                                              | O | O | O | O | | |                                                                    |  
| [Moonshot AI](https://docs.spring.io/spring-ai/reference/api/chat/moonshot-chat.html)                   | text                                              | | O | O | O | | |                                                                    |  
| [NVIDIA (OpenAI-proxy)](https://docs.spring.io/spring-ai/reference/api/chat/nvidia-chat.html)           | text, image                                       | O | O | O | O | | | O |  
| [OCI GenAI/Cohere](https://docs.spring.io/spring-ai/reference/api/chat/oci-genai/cohere-chat.html)      | text                                              | | | | O | | | |  
| [Ollama](https://docs.spring.io/spring-ai/reference/api/chat/ollama-chat.html)                          | text, image                                       | O | O | O | O | O | O | O |  
| [OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/openai-chat.html)                          | In: text, image, audio Out: text, audio           | O | O | O | O | O | | O |  
| [Perplexity (OpenAI-proxy)](https://docs.spring.io/spring-ai/reference/api/chat/perplexity-chat.html)   | text                                              | | O | O | O | | | O |  
| [QianFan](https://docs.spring.io/spring-ai/reference/api/chat/qianfan-chat.html)                        | text                                              | | O | O | O | | | |  
| [ZhiPu AI](https://docs.spring.io/spring-ai/reference/api/chat/zhipuai-chat.html)                       | text                                              | O | O | O | O | | | |  
| [Amazon Bedrock Converse](https://docs.spring.io/spring-ai/reference/api/chat/bedrock-converse.html)    | text, image, video, docs (pdf, html, md, docx …) | O | O | O | O | | | |

<br/>

## 10. Ollama Chat
---

Spring AI는 Ollama 채팅 통합을 위한 Spring Boot 자동 구성을 제공한다. 아래와 같이 의존성을 추가하면 된다.

```xml
<dependency>
   <groupId>org.springframework.ai</groupId>
   <artifactId>spring-ai-starter-model-ollama</artifactId>
</dependency>
```

<br/>

### 10.1. 기본 설정
---

|**속성(Property)**|**설명(Description)**|**기본값(Default)**|
|---|---|---|
|spring.ai.ollama.base-url|Ollama API 서버가 실행 중인 기본 URL|localhost:11434|

<br/>

다음은 Ollama 통합 초기화 및 모델 자동 다운로드(Auto-pulling)에 관련된 속성들입니다:

|**속성(Property)**|**설명(Description)**|**기본값(Default)**|
|---|---|---|
|spring.ai.ollama.init.pull-model-strategy|애플리케이션 시작 시 모델을 다운로드할지 여부 및 방식 설정|never|
|spring.ai.ollama.init.timeout|모델 다운로드 시 대기할 최대 시간|5m|
|spring.ai.ollama.init.max-retries|모델 다운로드 작업의 최대 재시도 횟수|0|
|spring.ai.ollama.init.chat.include|초기화 작업에 챗용 모델 포함 여부|true|
|spring.ai.ollama.init.chat.additional-models|기본 속성 외에 추가로 초기화할 모델 목록|[]|

<br/>

### 10.2. 채팅 설정
---

- **활성화**: spring.ai.model.chat=ollama (기본값은 활성화 상태)
- **비활성화**: spring.ai.model.chat=none (또는 ollama와 일치하지 않는 어떤 값)

<br/>

#### 10.2.1. 주요 구성 속성

spring.ai.ollama.chat.options는 Ollama 챗 모델을 구성하기 위한 속성 접두어입니다. 이 속성은 모델 이름, 응답 형식, 메모리 유지 시간 등의 고급 요청 파라미터와 Ollama 모델 세부 옵션을 포함합니다.

|**속성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.ollama.chat.options.model|사용할 모델 이름|mistral|
|spring.ai.ollama.chat.options.format|응답 형식 (json만 허용됨)|-|
|spring.ai.ollama.chat.options.keep_alive|모델을 메모리에 유지할 시간|5m|

<br/>

#### 10.2.2. 고급 요청 및 성능 최적화 옵션

|**속성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.ollama.chat.options.numa|NUMA 사용 여부|false|
|spring.ai.ollama.chat.options.num-ctx|컨텍스트 윈도우 크기|2048|
|spring.ai.ollama.chat.options.num-batch|최대 배치 크기|512|
|spring.ai.ollama.chat.options.num-gpu|GPU에 보낼 레이어 수 (-1은 자동)|-1|
|spring.ai.ollama.chat.options.main-gpu|다중 GPU 사용 시 작은 텐서를 처리할 GPU 인덱스|0|
|spring.ai.ollama.chat.options.low-vram|VRAM 사용 최소화 옵션|false|
|spring.ai.ollama.chat.options.f16-kv|16비트 키-값 캐시 사용|true|
|spring.ai.ollama.chat.options.logits-all|모든 토큰에 대한 로짓 반환 여부|-|
|spring.ai.ollama.chat.options.vocab-only|단어 사전만 로드, 가중치는 로드하지 않음|-|
|spring.ai.ollama.chat.options.use-mmap|메모리 매핑 사용 여부|null|
|spring.ai.ollama.chat.options.use-mlock|모델을 메모리에 고정 (swap 방지)|false|
|spring.ai.ollama.chat.options.num-thread|사용할 스레드 수 (0 = 자동 감지)|0|
|spring.ai.ollama.chat.options.num-keep|이전 토큰 수 유지|4|
|spring.ai.ollama.chat.options.seed|텍스트 생성을 위한 랜덤 시드 설정|-1|
|spring.ai.ollama.chat.options.num-predict|최대 생성 토큰 수 (-1=무제한, -2=컨텍스트 채우기)|-1|

<br/>

#### 10.2.3. 생성 다양성 및 제어 관련 옵션

|**속성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.ollama.chat.options.top-k|높은 값 = 다양성 증가, 낮은 값 = 보수적|40|
|spring.ai.ollama.chat.options.top-p|top-k와 함께 다양성 조절|0.9|
|spring.ai.ollama.chat.options.min-p|가장 가능성 높은 토큰 기준 필터 임계값|0.0|
|spring.ai.ollama.chat.options.tfs-z|덜 가능성 있는 토큰 영향 감소|1.0|
|spring.ai.ollama.chat.options.typical-p|전형적 샘플링 확률|1.0|
|spring.ai.ollama.chat.options.repeat-last-n|반복 방지를 위해 참조할 이전 토큰 수|64|
|spring.ai.ollama.chat.options.temperature|창의성 조절 (높을수록 창의적)|0.8|
|spring.ai.ollama.chat.options.repeat-penalty|반복에 대한 패널티 강도|1.1|
|spring.ai.ollama.chat.options.presence-penalty|새로운 단어 등장 유도|0.0|
|spring.ai.ollama.chat.options.frequency-penalty|자주 등장하는 단어 억제|0.0|

<br/>

#### 10.2.4. Mirostat 관련 옵션

|**속성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.ollama.chat.options.mirostat|Mirostat 샘플링 활성화 (0=비활성화, 1=Mirostat 1.0, 2=Mirostat 2.0)|0|
|spring.ai.ollama.chat.options.mirostat-tau|집중도 vs 다양성 조절|5.0|
|spring.ai.ollama.chat.options.mirostat-eta|학습률 (응답 조정 속도)|0.1|

<br/>

#### 10.2.5. 기타 옵션

|**속성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.ollama.chat.options.penalize-newline|줄바꿈 문자 패널티 적용 여부|true|
|spring.ai.ollama.chat.options.stop|텍스트 생성을 멈출 패턴 지정|-|
|spring.ai.ollama.chat.options.functions|사용할 함수 이름 목록 (Function Calling)|-|
|spring.ai.ollama.chat.options.proxy-tool-calls|함수 호출을 클라이언트로 프록시할지 여부|false|

<br/>

### 10.3. 런타임 옵션
---

기본 설정을 덮어쓰고 요청별로 다른 설정을 적용하려면, `Prompt` 호출 시 요청 전용 옵션을 추가하면 된다.

```java
ChatResponse response = chatModel.call(
    new Prompt(
        "Generate the names of 5 famous pirates.",
        OllamaOptions.builder()
            .model(OllamaModel.LLAMA3_1)  // LLAMA 3.1 모델 사용
            .temperature(0.4)             // 낮은 창의성 설정
            .build()
    ));
```

<br/>

### 10.4. 함수 호출
---

Spring AI의 `OllamaChatModel` 은 사용자 정의 Java 함수를 등록하고, Ollama 모델이 해당 함수 중 하나 또는 여러 개를 지능적으로 선택해 호출할 수 있도록 지원한다.

모델은 적절한 타이밍에 호출할 함수와 인자를 담은 JSON 객체를 출력하며, 이를 통해 LLM의 능력을 외부 도구 및 API와 연결할 수 있다.

<br/>

### 10.5. 구조화된 출력
---

Ollama는 JSON Schema에 정확히 부합하는 응답을 생성하도록 강제할 수 있는 고유 Structured Output API를 제공한다.

Spring AI는 기존의 범용 `StructuredOutputConverter` 외에도, Ollama 전용 API를 통해 더 정밀하고 통제 가능한 구조화된 출력 기능을 제공한다.

<br/>

#### 10.5.1. 구성 방법
---

Spring AI에서는 `OllamaOptions` 빌더를 사용하여 응답 포맷을 프로그래밍 방식으로 지정할 수 있다.

<br/>

**예시1. 직접 JSON Schema 지정**

```java
String jsonSchema = """
{
  "type": "object",
  "properties": {
    "steps": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "explanation": { "type": "string" },
          "output": { "type": "string" }
        },
        "required": ["explanation", "output"],
        "additionalProperties": false
      }
    },
    "final_answer": { "type": "string" }
  },
  "required": ["steps", "final_answer"],
  "additionalProperties": false
}
""";

Prompt prompt = new Prompt("how can I solve 8x + 7 = -23",
    OllamaOptions.builder()
        .model(OllamaModel.LLAMA3_2.getName())
        .format(new ObjectMapper().readValue(jsonSchema, Map.class))
        .build());

ChatResponse response = this.ollamaChatModel.call(prompt);
```

<br/>

**예시2. 도메인 객체를 통한 자동 변환**

`BeanOutputConverter` 를 사용하면 도메인 객체로부터 자동으로 JSON Schema를 생성하고, 나중에 모델 응답을 해당 객체로 변환할 수 있다.

```java
record MathReasoning(
    @JsonProperty(required = true, value = "steps") Steps steps,
    @JsonProperty(required = true, value = "final_answer") String finalAnswer) {

    record Steps(
        @JsonProperty(required = true, value = "items") Items[] items) {

        record Items(
            @JsonProperty(required = true, value = "explanation") String explanation,
            @JsonProperty(required = true, value = "output") String output) {
        }
    }
}

var outputConverter = new BeanOutputConverter<>(MathReasoning.class);

Prompt prompt = new Prompt("how can I solve 8x + 7 = -23",
    OllamaOptions.builder()
        .model(OllamaModel.LLAMA3_2.getName())
        .format(outputConverter.getJsonSchemaMap())
        .build());

ChatResponse response = this.ollamaChatModel.call(prompt);
String content = response.getResult().getOutput().getText();

MathReasoning result = outputConverter.convert(content);
```

<br/>

#### 10.5.2. OpenAI API 호환성
---

Ollama는 Open API와 호환되므로, Spring AI의 OpenAI 클라이언트를 사용해 Ollama에 연결하고 도구(Function Calling) 기능도 그대로 활용할 수 있다.

<br/>

**구성 방법**

OpenAI 클라이언트를 사용하여 Ollama 인스턴스에 연결하려면 아래와 같이 설정하면 된다.

```
spring.ai.openai.chat.base-url=http://localhost:11434
spring.ai.openai.chat.options.model=mistral
```

<br/>

#### 10.5.3. 사용 예시
---

`spring-ai-starter-model-ollama` 의존을 추가한다.

`src/main/resources` 디렉터리에 `application.yaml` 파일을 생성하고, Ollama 채팅 모델을 활성화하고 구성한다.

```yaml
spring:
  ai:
    ollama:
      base-url: http://localhost:11434
      chat:
        options:
          model: mistral
          temperature: 0.7
```

<br/>

`@RestController` 예제는 다음과 같다.

```java
@RestController
public class ChatController {

    private final OllamaChatModel chatModel;

    @Autowired
    public ChatController(OllamaChatModel chatModel) {
        this.chatModel = chatModel;
    }

    @GetMapping("/ai/generate")
    public Map<String, String> generate(@RequestParam(value = "message", defaultValue = "Tell me a joke") String message) {
        return Map.of("generation", this.chatModel.call(message));
    }

    @GetMapping("/ai/generateStream")
    public Flux<ChatResponse> generateStream(@RequestParam(value = "message", defaultValue = "Tell me a joke") String message) {
        Prompt prompt = new Prompt(new UserMessage(message));
        return this.chatModel.stream(prompt);
    }

}
```

<br/>

#### 10.5.4. 수동 설정
---

Spring Boot의 자동 구성 기능을 사용하지 않고 수동으로 OllamaChatModel을 설정할 수도 있다.

이를 사용하려면 다음과 같이 의존을 추가한다.

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama</artifactId>
</dependency>
```

<br/>

다음과 같이 `OllamaChatModel` 인스턴스를 수동으로 생성하고, 텍스트 생성을 요청할 수 있다.

```java
var ollamaApi = OllamaApi.builder().build();

var chatModel = OllamaChatModel.builder()
    .ollamaApi(ollamaApi)
    .defaultOptions(
        OllamaOptions.builder()
            .model(OllamaModel.MISTRAL)
            .temperature(0.9)
            .build())
    .build();

ChatResponse response = chatModel.call(
    new Prompt("Generate the names of 5 famous pirates."));

// 또는 스트리밍 응답으로
Flux<ChatResponse> response = chatModel.stream(
    new Prompt("Generate the names of 5 famous pirates."));
```

<br/>

## 11. OpenAI Chat
---

### 11.1. 사전 준비 사항
---

1. OpenAI에서 계정을 생성한다.
2. API 키 페이지에서 API 토큰을 발급받는다.

`spring.ai.openai.api-key` 라는 구성 속성을 통해 API 키를 설정할 수 있다.

```
spring.ai.openai.api-key=<your-openai-api-key>
```

<br/>

### 11.2. 자동 구성
---

다음 의존을 추가하여 자동으로 구성할 수 있다.

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-openai</artifactId>
</dependency>
```

<br/>

### 11.3. 채팅 속성
---

#### 11.3.1. Retry 설정
---

|**속성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.retry.max-attempts|최대 재시도 횟수|10|
|spring.ai.retry.backoff.initial-interval|최초 재시도 대기 시간|2초|
|spring.ai.retry.backoff.multiplier|재시도 대기 시간 증가 비율|5|
|spring.ai.retry.backoff.max-interval|최대 재시도 대기 시간|3분|
|spring.ai.retry.on-client-errors|false일 경우, 4xx 응답에 대해 재시도하지 않고 예외 발생|false|
|spring.ai.retry.exclude-on-http-codes|재시도를 하지 않을 HTTP 상태 코드 목록|없음|
|spring.ai.retry.on-http-codes|재시도를 유발할 HTTP 상태 코드 목록|없음|

<br/>

#### 11.3.2. Connection 설정
---

|**속성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.openai.base-url|연결할 OpenAI API URL|https://api.openai.com|
|spring.ai.openai.api-key|OpenAI API 키|(필수 입력)|
|spring.ai.openai.organization-id|요청에 사용할 조직 ID (선택사항)|없음|
|spring.ai.openai.project-id|요청에 사용할 프로젝트 ID (선택사항)|없음|

<br/>

#### 11.3.3. Chat Model 설정
---

**활성화 설정**

|**성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.model.chat|사용할 채팅 모델 지정 (openai 또는 none)|openai|

<br/>

**모델 및 옵션 설정**

|**속성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.openai.chat.options.model|사용할 모델 이름 (예: gpt-4o, gpt-3.5-turbo)|gpt-4o-mini|
|spring.ai.openai.chat.options.temperature|창의성 조절 (0.0~2.0)|0.8|
|spring.ai.openai.chat.options.frequencyPenalty|반복 억제 정도 (-2.0~2.0)|0.0|
|spring.ai.openai.chat.options.presencePenalty|새 주제 유도 정도 (-2.0~2.0)|없음|
|spring.ai.openai.chat.options.maxCompletionTokens|최대 생성 토큰 수 (입력+출력 포함)|없음|
|spring.ai.openai.chat.options.n|생성할 응답 개수|1|
|spring.ai.openai.chat.options.store|응답 저장 여부|false|
|spring.ai.openai.chat.options.metadata|사용자 정의 메타데이터|빈 맵|
|spring.ai.openai.chat.options.topP|토큰 상위 누적 확률 선택|없음|
|spring.ai.openai.chat.options.stop|텍스트 생성을 중지할 시퀀스 (최대 4개)|없음|
|spring.ai.openai.chat.options.seed|반복 요청 시 동일 결과를 위한 시드 값 (베타)|없음|

<br/>

**JSON 응답 형식 설정**

|**속성**|**설명**|
|---|---|
|spring.ai.openai.chat.options.responseFormat.type|응답 형식 (예: JSON_OBJECT, JSON_SCHEMA)|
|spring.ai.openai.chat.options.responseFormat.schema|JSON 스키마 (type이 JSON_SCHEMA일 경우 필수)|
|spring.ai.openai.chat.options.responseFormat.name|응답 스키마 이름|
|spring.ai.openai.chat.options.responseFormat.strict|스키마 준수 강도 설정|

<br/>

**툴 사용 관련**

|**속성**|**설명**|
|---|---|
|spring.ai.openai.chat.options.tools|사용할 툴(함수) 목록|
|spring.ai.openai.chat.options.toolChoice|툴 사용 방식 (예: auto, none, 특정 함수)|
|spring.ai.openai.chat.options.functions|단일 프롬프트에 사용할 함수 이름 목록|
|spring.ai.openai.chat.options.parallel-tool-calls|병렬 함수 호출 허용 여부 (true)|
|spring.ai.openai.chat.options.proxy-tool-calls|Spring이 아닌 클라이언트 측에서 함수 실행할지 여부 (false)|

<br/>

**기타 설정**

|**속성**|**설명**|**기본값**|
|---|---|---|
|spring.ai.openai.chat.options.user|고유 사용자 ID (악용 방지용)|없음|
|spring.ai.openai.chat.options.stream-usage|스트리밍 중 사용량 정보 포함 여부|false|
|spring.ai.openai.chat.options.http-headers|추가 HTTP 헤더 설정 (예: Authorization)|없음|

<br/>

### 11.4. 구조화된 출력
---

OpenAI는 JSON Schema를 기반으로 모델이 응답을 구조화된 형식으로 생성하도록 보장하는 Structured Outputs API를 제공한다.

<br/>

#### 11.4.1. JSON Schema 기반 응답 구성 예시
---

```java
String jsonSchema = """
        {
            "type": "object",
            "properties": {
                "steps": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "explanation": { "type": "string" },
                            "output": { "type": "string" }
                        },
                        "required": ["explanation", "output"],
                        "additionalProperties": false
                    }
                },
                "final_answer": { "type": "string" }
            },
            "required": ["steps", "final_answer"],
            "additionalProperties": false
        }
        """;

Prompt prompt = new Prompt("how can I solve 8x + 7 = -23",
        OpenAiChatOptions.builder()
            .model(ChatModel.GPT_4_O_MINI)
            .responseFormat(new ResponseFormat(ResponseFormat.Type.JSON_SCHEMA, this.jsonSchema))
            .build());

ChatResponse response = this.openAiChatModel.call(this.prompt);
```

<br/>

#### 11.4.2. 도메인 객체 기반 자동 변환
---

```kotlin
data class MathReasoning(
	val steps: Steps,
	@get:JsonProperty(value = "final_answer") val finalAnswer: String) {

	data class Steps(val items: Array<Items>) {

		data class Items(
			val explanation: String,
			val output: String)
	}
}

val outputConverter = BeanOutputConverter(MathReasoning::class.java)

val jsonSchema = outputConverter.jsonSchema;

val prompt = Prompt("how can I solve 8x + 7 = -23",
	OpenAiChatOptions.builder()
		.model(ChatModel.GPT_4_O_MINI)
		.responseFormat(ResponseFormat(ResponseFormat.Type.JSON_SCHEMA, jsonSchema))
		.build())

val response = openAiChatModel.call(prompt)
val content = response.getResult().getOutput().getContent()

val mathReasoning = outputConverter.convert(content)
```
- Kotlin의 경우 널 여부와 기본값에 따라 필수 여부가 추론되므로 `@get:JsonProperty(required = true)` 는 불필요하다.

<br/>

#### 11.4.3. 설정 파일 기반 구성
---

구성 파일을 통해 구조화된 출력 설정이 가능하다.

```
spring.ai.openai.api-key=YOUR_API_KEY
spring.ai.openai.chat.options.model=gpt-4o-mini

spring.ai.openai.chat.options.response-format.type=JSON_SCHEMA
spring.ai.openai.chat.options.response-format.name=MySchemaName
spring.ai.openai.chat.options.response-format.schema={"type":"object","properties":{"steps":{"type":"array","items":{"type":"object","properties":{"explanation":{"type":"string"},"output":{"type":"string"}},"required":["explanation","output"],"additionalProperties":false}},"final_answer":{"type":"string"}},"required":["steps","final_answer"],"additionalProperties":false}
spring.ai.openai.chat.options.response-format.strict=true
```

<br/>

### 11.5. 사용 예시
---

먼저 의존을 추가한다

```xml
dependencies {
    implementation 'org.springframework.ai:spring-ai-starter-model-openai'
}
```

<br/>

`application.properties` 구성한다

```
spring.ai.openai.api-key=YOUR_API_KEY
spring.ai.openai.chat.options.model=gpt-4o
spring.ai.openai.chat.options.temperature=0.7
```

<br/>

컨트롤러를 구성한다

```java
@RestController
public class ChatController {

    private final OpenAiChatModel chatModel;

    @Autowired
    public ChatController(OpenAiChatModel chatModel) {
        this.chatModel = chatModel;
    }

    @GetMapping("/ai/generate")
    public Map<String, String> generate(@RequestParam(value = "message", defaultValue = "Tell me a joke") String message) {
        return Map.of("generation", this.chatModel.call(message));
    }

    @GetMapping("/ai/generateStream")
    public Flux<ChatResponse> generateStream(@RequestParam(value = "message", defaultValue = "Tell me a joke") String message) {
        Prompt prompt = new Prompt(new UserMessage(message));
        return this.chatModel.stream(prompt);
    }
g}
```

<br/>

### 11.6. 수동 설정
---

수동으로 연결하고 싶을 경우 다음과 같이 진행한다.

먼저 의존을 추가한다

```groovy
dependencies {
    implementation 'org.springframework.ai:spring-ai-openai'
}
```

<br/>

**OpenAiChatModel 직접 생성**

```java
var openAiApi = OpenAiApi.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .build();

var openAiChatOptions = OpenAiChatOptions.builder()
    .model("gpt-3.5-turbo")
    .temperature(0.4)
    .maxTokens(200)
    .build();

var chatModel = new OpenAiChatModel(openAiApi, openAiChatOptions);

// 동기 호출
ChatResponse response = chatModel.call(
    new Prompt("Generate the names of 5 famous pirates."));

// 스트리밍 호출
Flux<ChatResponse> responseStream = chatModel.stream(
    new Prompt("Generate the names of 5 famous pirates."));
```

<br/>

## Reference
---

[Spring AI: Reference Doc.](https://docs.spring.io/spring-ai/reference/index.html)