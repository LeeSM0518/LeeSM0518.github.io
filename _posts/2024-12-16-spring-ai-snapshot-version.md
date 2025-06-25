---
title: Spring AI (1.0.0-SNAPSHOT)
date: 2024-12-16 00:00:00 +0900
categories: spring
tags:
  - spring
  - ai
  - docs
  - kotlin
description: Spring AI를 활용하기 위한 Docs 정리
---

![spring-ai1](/assets/img/spring-ai1.svg)

## 1. 개요
---

`Spring AI` 는 불필요한 복잡성 없이 인공지능 기능을 통합한 애플리케이션 개발을 간소화하는 것을 목표로 한다. 즉, 애플리케이션의 데이터나 API를 AI 모델에 연결하는 기능을 제공한다.

![spring-ai2](/assets/img/spring-ai2.svg)

Spring AI는 AI 애플리케이션을 개발하기 위한 기반이 되는 추상화를 제공한다. 이로써, 최소한의 코드 변경으로 쉽게 구성 요소를 교체할 수 있다.

<br/>

## 2. 시작하기
---

> Spring AI는 Spring Boot 3.2.x 와 3.3.x 에서 지원된다.
{: .prompt-info }

<br/>

### 2.1. Snapshot 저장소와 의존 추가
---

> Spring AI는 아직 Release 버전이 존재하지 않으므로 Snapshot(개발 단계) 버전을 사용해야 한다.
{: .prompt-warning }

Snapshot 버전을 사용하려면 빌드 파일에 Spring Snapshot Repository에 대한 정보를 추가해야 한다.

<br/>

다음과 같이 Gradle 빌드 파일에 repository를 추가한다.

```kotlin
repositories {  
    mavenCentral()  
    maven("https://repo.spring.io/snapshot")  
}
```
{: file="build.gradle.kts"}

<br/>

다음과 같이 Gradle 빌드 파일에 의존을 추가한다.

```kotlin
dependencies {  
    implementation("org.springframework.ai:spring-ai-openai-spring-boot-starter:1.0.0-SNAPSHOT")
}
```
{: file="build.gradle.kts"}

<br/>

다음과 같이 사용할 AI 서비스의 API Key를 등록한다.

```yaml
spring:  
  ai:  
    openai:  
      api-key: {API키}
```
{: file="application.yaml"}

<br/>

## 3. Chat Client API
---

ChatClient는 AI 모델과 통신하기 위한 API를 제공한다.

API에는 AI 모델에 입력으로 전달되는 프롬프트의 구성 요소를 구축하는 방법이 있다. `Prompt` 에는 AI 모델의 출력 및 동작을 안내하는 텍스트가 포함되어 있다.

AI 모델은 메시지의 두 유형의 메시지를 처리한다.
1. `user` 메시지 : 사용자로부터 입력된 메시지
2. `system` 메시지 : 대화를 안내하기 위해 시스템이 생성한 메시지

<br/>

### 3.1. ChatClient 생성
---

`ChatClient` 는 `ChatClient.Builder` 객체를 활용해 생성할 수 있다. Spring Boot 자동 구성으로 생성된 `ChatClient.Builder` 인스턴스를 얻거나 프로그래밍 방식으로 생성할 수 있다.

<br/>

#### 3.1.1. 자동 구성된 ChatClient 사용

다음과 같이 `ChatClient.Builder` 를 주입하여 사용하면 된다.

```kotlin
@Component  
class ChatAiAdapter(builder: ChatClient.Builder) {

	private val chatClient: ChatClient = builder.build()
  
    suspend fun generation(userInput: String): String? =  
        chatClient  
            .prompt()  
            .user(userInput)  
            .call()  
            .content()  
}
```
- `call()` : AI 모델로 요청 전송
- `content()` : AI 모델의 응답을 반환

<br/>

#### 3.1.2. 직접 ChatClient 생성

`ChatClient.Builder` 의 자동 구성을 비활성화 하려면 예시와 같이 설정값을 변경하면 된다. (ex. `spring.ai.chat.client.enabled=false` ) 이러한 설정은 여러 대화 모델을 같이 사용할 때 유용하다.
이후, 다음과 같이 `ChatClient.Builder` 인스턴스를 통해 `ChatModel` 을 생성한다.
```kotlin
// 주로 자동 주입하여 사용
val myChatModel: ChatModel = ...

val builder: ChatClient.Builder = ChatClient.builder(this.myChatModel)

val chatClient: ChatClient = ChatClient.create(this.myChatModel)
```

<br/>

### 3.2. ChatClient Fluent API
---

다음과 같이 프롬프트를 만들 수 있다.

- `prompt()`
- `prompt(Prompt prompt)`
- `prompt(String content)`

<br/>

### 3.3. ChatClient 응답
---

`ChatClient` 여러 응답 포맷을 제공한다.

<br/>

#### 3.3.1. ChatResponse

본 응답에는 생성된 방법에 대한 메타데이터가 포함된다. 메타데이터에는 응답을 만드는 데 사용된 토큰의 수가 포함된다.

다음과 같이 `call()` 메서드 뒤에 `chatResponse()` 를 호출하여 객체를 반환할 수 있다.

```kotlin
chatClient  
    .prompt()  
    .user(userInput)  
    .call()  
    .chatResponse()
```

<br/>

#### 3.3.2. Entity

다음과 같이 `entity()` 메서드를 통해 엔티티로 매핑하여 응답할 수 있다.

```kotlin
data class Result(  
    val result: String,  
)  
  
suspend fun generation(userInput: String): Result? =  
    chatClient  
        .prompt()  
        .user(userInput)  
        .call()  
        .entity(Result::class.java)
```

<br/>

#### 3.3.3. Streaming

다음과 같이 `stream()` 메서드를 통해 비동기로 응답 받을 수 있다.

```kotlin
suspend fun generation(userInput: String): Flux<String> =  
    chatClient  
        .prompt()  
        .user(userInput)  
        .stream()  
        .content()
```

<br/>

다음과 같이 엔티티로 변환할 수도 있다.

```kotlin
val converter = BeanOutputConverter(object : ParameterizedTypeReference<List<Result>>() {})  
  
suspend fun generation(userInput: String): List<Result>? =  
    chatClient  
        .prompt()  
        .user(userInput)  
        .stream()  
        .content()  
        .collectList()  
        .awaitSingle()  
        ?.joinToString("")  
        ?.let { converter.convert(it) }
```

<br/>

### 3.4. `call()` 응답
---

`ChatClinet` 의 `call()` 메서드 호출 후 여러 타입으로 응답 받을 수 있다.

- `String content()` : 문자열 내용
- `ChatResponse chatResponse()` : 메타데이터가 포함된 `ChatResponse` 객체
- `entity()` 
	- `entity(ParameterizedTypeReference<T> type)` : 엔티티 타입의 `Collection` 을 반환
	- `entity(Class<T> type)` : 특정 엔티티 타입으로 반환
	- `entity(StructuredOutputConverter<T> converter)` : `String` 을 엔티티 타입으로 변환한 객체

`call()` 대신에 `stream()` 메서드를 사용할 수도 있다.

<br/>

### 3.5. `stream()` 응답
---

`ChatClient` 의 `stream()` 메서드 호출 후 여러 타입으로 응답 받을 수 있다.

- `Flux<String> content()` : 문자열의 `Flux`
- `Flux<ChatResponse> chatResponse()` : `ChatResponse` 객체의 `Flux`

<br/>

### 3.6. 기본값 사용
---

`@Configuration` 클래스에서 기본 시스템 텍스트로 `ChatClient` 를 생성하면 생성 코드를 간소화 할 수 있다. 기본값을 설정하면 `ChatClient` 를 호출할 때 사용자 텍스트만 지정하면 되므로 각 요청에 대한 시스템 텍스트를 설정할 필요가 없다.

<br/>

#### 3.6.1. 기본 시스템 텍스트
---

다음과 같이 기본 시스템 텍스트를 설정할 수 있다.

```kotlin
@Configuration  
class ChatClientConfiguration(  
    val builder: ChatClient.Builder  
) {  
  
    @Bean  
    fun chatClient(): ChatClient =  
        builder  
            .defaultSystem("You are a friendly chat bot that answers question in the voice of a Pirate")  
            .build()  
}
```

<br/>

그리고 다음과 같이 사용할 수 있다.

```kotlin
@RestController  
class ChatAiAdapter(  
    private val chatClient: ChatClient,  
) {  
  
    @GetMapping("/ai/simple")  
    suspend fun completion(  
        @RequestParam(value = "message", defaultValue = "Tell me a joke")  
        message: String,  
    ): Map<String, String?> =  
        mapOf("completion" to chatClient.prompt().user(message).call().content())  
  
}
```

<br/>

API를 호출한 결과는 다음과 같다.

```
GET http://localhost:8080/ai/simple?message=hello

HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 129

{
  "completion": "Ahoy, matey! How be ye this fine day on the seven seas? What be the treasure ye seek from this ole pirate? Arrr!"
}
```

<br/>

#### 3.6.2. 파라미터 기반 기본 시스템 텍스트
---

다음과 같이 시스템 텍스트에 placeholder를 사용하여 파라미터를 전달할 수도 있다.

```kotlin
@Configuration  
class ChatClientConfiguration(  
    val builder: ChatClient.Builder  
) {  
  
    @Bean  
    fun chatClient(): ChatClient =  
        builder  
            .defaultSystem("You are a friendly chat bot that answers question in the voice of {voice}")  
            .build()  
}
```

```kotlin
@RestController  
class ChatAiAdapter(  
    private val chatClient: ChatClient,  
) {  
  
    @GetMapping("/ai/simple")  
    suspend fun completion(  
        @RequestParam(value = "message", defaultValue = "Tell me a joke")  
        message: String,  
        @RequestParam(value = "voice")  
        voice: String,  
    ): Map<String, String?> =  
        mapOf("completion" to  
                chatClient  
                    .prompt()  
                    .system { it.param("voice", voice) }  
                    .user(message)  
                    .call()  
                    .content()  
        )  
  
}
```

```
GET http://localhost:8080/ai/simple?message=hello&voice='Robert De Niro'

HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 53

{
  "completion": "Hey, how you doin'? What's goin' on?"
}
```

<br/>

#### 3.6.3. 다른 기본값들
---

`ChatClient.Builder` 에서 기본 프롬프트 구성을 지정할 수 있다.

- `defaultOptions(ChatOptions chatOptions)` : `ChatOptions` 나 모델 특화된 `OpenAiChatOptions` 등을 지정할 수 있다.
- `defaultFunctions(String name, String description, Function<I, O> function)` : `name` 은 사용자 텍스트에서 함수를 참조하는 데 사용된다. `description` 은 함수의 목적을 설명하고 AI 모델이 정확한 응답을 할 수 있도록 한다. `function` 는 필요할 때 모델이 실행하는 함수 인스턴스이다.
- `defaultFunctions(String... functionsNames)`
- `defaultUser` : 사용자 텍스트를 정의
- `defaultAdvisors` : `prompt` 를 만드는 데 사용된 데이터를 수정하는 것을 허용한다. 

<br/>

### 3.7. Advisors
---

Advisors API는 Spring 애플리케이션에서 AI 기반 상호작용을 인터셉트, 수정, 향상시키는 방법이다.

사용자 텍스트로 AI 모델을 호출할 때 일반적인 패턴은 프롬프트에 문맥별 데이터를 추가하거나 증가시키는 것이다.

문맥 데이터는 여러 유형이 될 수 있다. 일반적인 유형은 다음과 같다.
- **개인 데이터** : AI 모델이 훈련받지 않은 데이터이다.
- **대화 내역** : 이전 대화 내용이 응답을 생성할 때 고려하도록 하려면 요청과 함께 이전 대화 내용을 보내야 한다.

<br/>

#### 3.7.1. ChatClient에 Advisor 설정
---

ChatClient fluen API는 advisors를 설정하기 위한 `AdvisorSpec` 인터페이스를 제공한다. 해당 인터페이스는 advisor 를 추가하는 메서드들을 지원한다.

```java
interface AdvisorSpec {
    AdvisorSpec param(String k, Object v);
    AdvisorSpec params(Map<String, Object> p);
    AdvisorSpec advisors(Advisor... advisors);
    AdvisorSpec advisors(List<Advisor> advisors);
}
```

<br/>

> 어드바이저 체인에 추가되는 순서는 어드바이저의 실행 순서를 결정하기 때문에 매우 중요하다.
{: .prompt-danger }

```kotlin
builder  
    .build()  
    .prompt()  
    .advisors(  
        MessageChatMemoryAdvisor(chatMemory),  
        QuestionAnswerAdvisor(vectorStore, SearchRequest.defaults())  
    )  
    .user(userText)  
    .call()  
    .content()
```
- 이 구성에서는 `MessageChatMemoryAdvisor` 가 먼저 실행되어 대화 내역이 프롬프트에 추가된다.
- 그 후, `QuestionAnswerAdvisor` 는 사용자의 질문과 추가된 대화 기록을 기반으로 검색을 수행하여 더욱 관련성 있는 결과를 제공할 가능성이 있다.

<br/>

#### 3.7.2. 검색 증강 생성 (RAG, Retrieval Augmented Generation)
---

벡터 데이터베이스는 AI 모델이 인식하지 못하는 데이터를 저장한다. 사용자 질문이 AI 모델에 전송되면 `QuestionAnswerAdvisor` 가 벡터 데이터베이스에서 사용자 질문과 관련된 문서를 쿼리한다.

벡터 데이터베이스의 응답은 사용자 텍스트에 추가되어 AI 모델이 응답을 생성할 수 있는 맥락을 제공한다.

이미 `VectorStore` 에 데이터를 로드했다고 가정하면 `ChatClient` 에 `QuestionAnswerAdvisor` 인스턴스를 제공하여 검색 증강 생성(RAG)을 수행할 수 있다.

```kotlin
builder  
    .build()  
    .prompt()  
    .advisors(  
        MessageChatMemoryAdvisor(chatMemory),  
        QuestionAnswerAdvisor(vectorStore, SearchRequest.defaults())  
    )  
    .user(userText)  
    .call()  
    .chatResponse()
```
- `SearchRequest.defaults()` 는 벡터 데이터베이스의 모든 문서에 대한 유사성 검색을 수행한다.
- 검색되는 문서 유형을 제한하기 위해 `SearchRequest` 는 모든 `VectorStore` 에서 이식 가능한 SQL과 유사한 필터 표현식을 사용한다.

<br/>

**동적 필터 표현식**

`FILTER_EXPRESSION` advisor 컨텍스트 매개변수를 사용하여 런타임에 `SearchRequest` 필터 표현식을 수정한다.

```kotlin
val chatClient: ChatClient = 
  ChatClient
    .builder(chatModel)
    .defaultAdvisors(new QuestionAnswerAdvisor(vectorStore, SearchRequest.defaults()))
    .build()

// Update filter expression at runtime
val content: String = 
  chatClient
    .prompt()
    .user("Please answer my question XYZ")
    .advisors(a -> a.param(QuestionAnswerAdvisor.FILTER_EXPRESSION, "type == 'Spring'"))
    .call()
    .content()
```
- 검색 결과를 동적으로 필터링할 수 있다.

<br/>

#### 3.7.3. Chat Memory
---

`ChatMemory` 인터페이스는 대화 내역을 위한 저장소이다. 대화에 메시지를 추가하고, 대화에서 메시지를 검색하고, 대화 내역을 지우는 방법을 제공한다.

현재 채팅 대화 내역을 메모리에 저장하고, 각각 `time-to-live` 로 저장하는 `InMemoryChatMemory` 와 `CassandraChatMemory` 라는 두 가지 구현체가 있다.

다음과 같이 `time-to-live` 를 사용하여 `CassandraChatMemory` 를 생성할 수 있다.

```kotlin
CassandraChatMemory
  .create(
    CassandraChatMemoryConfig
      .builder()
      .withTimeToLive(Duration.ofDays(1)
  )
  .build()
```

<br/>

다음 어드바이저 구현체들은 대화 내역을 프롬프트에 어드바이스하기 위해 `ChatMemory` 인터페이스를 사용한다. 
- `MessageChatMemoryAdvisor`, `PromptChatMemoryAdvisor`, `VectoreStoreChatMemoryAdvisor`
  - 자세한 내용은 Spring AI Docs 참고

<br/>

#### 3.7.4. 로깅
---

`SimpleLoggerAdvisor` 는 `ChatClient` 의 `request` 및 `response` 데이터를 기록하는 어드바이저이다. 이는 AI 상호작용을 디버깅하고 모니터링하는 데 유용하다.

로깅을 활성화하려면 ChatClient를 생성할 때 `SimpleLoggerAdvisor` 를 advisor 체인에 추가하면 된다.

```kotlin
@Configuration  
class ChatClientConfiguration(  
    val builder: ChatClient.Builder  
) {  
  
    @Bean  
    fun chatClient(): ChatClient =  
        builder  
            .defaultAdvisors(SimpleLoggerAdvisor())  
            .build()  
}
```

로그를 보려면 advisor 패키지의 로깅 수준을 `DEBUG` 로 설정하면 된다.

```yaml
logging:  
  level:  
    org.springframework.ai.chat.client.advisor: DEBUG
```
{: file="application.yaml"}

<br/>

## 4. OpenAI
---

### 4.1. 필수 조건
---

OpenAI에 가입한 후에 API Key를 생성해야 한다. 해당 키를 `spring.ai.openai.api-key` 에 설정해야 한다.

<br/>

### 4.2. 자동 설정
---

다음 의존을 사용하면 OpenAI Chat Client가 자동으로 설정된다.

```kotlin
dependencies {  
    implementation("org.springframework.ai:spring-ai-openai-spring-boot-starter:1.0.0-SNAPSHOT")  
}
```
{: file="build.gradle.kts"}

<br/>

#### 4.2.1. 대화 설정

**재시도 프로퍼티**

`spring.ai.retry` 는 OpenAI 채팅 모델의 재시도 메커니즘을 구성할 수 있는 속성이다.

```yaml
spring:
  ai:
    retry:
      # 재시도 최대 횟수를 설정합니다.
      max-attempts: 10
      backoff:
        # 재시도 하기 전 대기 시간을 설정합니다 (밀리초 단위).
        initial-interval: 2000
        # 재시도 대기시간 증가 비율을 설정합니다.
        multiplier: 5
        # 재시도 최대 시간을 설정합니다 (밀리초 단위).
        max-interval: 180000
      # 클라이언트 오류에 대해 재시도를 시도할지 여부를 설정합니다.
      on-client-errors: false
      # 재시도를 트리거하지 않아야 하는 HTTP 상태 코드 목록을 설정합니다.
      exclude-on-http-codes: []
      # 재시도를 트리거해야 하는 HTTP 상태 코드 목록을 설정합니다.
      on-http-codes: []
```

<br/>

**연결 프로퍼티**

`spring.ai.openai` 는 OpenAI의 연결 속성이다.

```yaml
spring:
  ai:
    openai:
      # 연결 URL
      base-url: https://api.openai.com/
      # API Key
      api-key: your-openai-api-key
      # 조직 ID
      organization-id: your-organization-id
      # 프로젝트 ID
      project-id: your-project-id
```

<br/>

**설정 프로퍼티**

`spring.ai.openai.chat` 은 OpenAI의 채팅 모델을 설정할 수 있는 속성이다.

```yaml
spring:
  ai:
    openai:
      chat:
        # OpenAI Chat 모델 사용을 활성화합니다.
        enabled: true
        # 채팅 전용 API 요청을 위한 기본 URL을 선택적으로 재정의합니다.
        base-url: https://api.openai.com/
        # 채팅 완료를 위한 기본 URL에 추가할 경로를 지정합니다.
        completions-path: /v1/chat/completions
        # 채팅 전용 API 요청을 위한 API 키를 선택적으로 재정의합니다.
        api-key: your-chat-api-key
        # API 요청 시 사용할 조직 ID를 선택적으로 지정할 수 있습니다.
        organization-id: your-organization-id
        # API 요청 시 사용할 프로젝트 ID를 선택적으로 지정할 수 있습니다.
        project-id: your-project-id
        options:
          # 사용할 OpenAI 채팅 모델의 이름을 선택합니다 (예: gpt-4o).
          model: gpt-4o
          # 생성된 완료의 창의성을 제어하는 샘플링 온도를 설정합니다 (기본값: 0.8).
          temperature: 0.8
          # 기존 빈도에 따라 새로운 토큰에 페널티를 적용하여 반복을 줄입니다 (기본값: 0.0f).
          frequencyPenalty: 0.0f
          # 완료에 특정 토큰이 나타날 확률을 조정합니다.
          logitBias: 
          # (Deprecated) 채팅 완료에서 생성할 최대 토큰 수를 설정합니다.
          maxTokens: 
          # 완료를 위해 생성할 수 있는 토큰 수의 상한을 설정합니다.
          maxCompletionTokens: 
          # 각 입력 메시지에 대해 생성할 완료 선택지의 수를 지정합니다 (기본값: 1).
          n: 1
          # 완료에서 생성할 출력 유형을 지정합니다 (예: text, audio).
          output-modalities: text,audio
          # 오디오 출력이 요청될 때 필요한 오디오 파라미터를 설정합니다.
          output-audio:
          # 텍스트에 나타난 토큰의 존재 여부에 따라 새로운 주제를 장려하는 페널티를 설정합니다.
          presencePenalty: 
          responseFormat:
            # 응답 형식 유형을 정의합니다 (예: JSON_OBJECT, JSON_SCHEMA).
            type: JSON_OBJECT
            # JSON_SCHEMA 유형에 대한 응답 형식 스키마 이름을 지정합니다 (기본값: custom_schema).
            name: custom_schema
            # JSON_SCHEMA 유형을 사용할 때의 응답 형식 JSON 스키마를 지정합니다.
            schema: 
            # JSON_SCHEMA 유형의 응답 형식 스키마 준수 엄격성을 설정합니다.
            strict: 
          # 동일한 시드와 파라미터로 반복 요청 시 동일한 결과를 반환하도록 결정론적 샘플링을 활성화합니다.
          seed: 
          # API가 추가 토큰 생성을 중지할 최대 4개의 시퀀스를 지정합니다.
          stop: 
          # 토큰 생성을 위한 누클리어 샘플링 확률 질량을 설정합니다.
          topP: 
          # 모델이 완료 중 호출할 수 있는 도구(함수) 목록을 지정합니다.
          tools: 
          # 완료 중 호출할 함수(있을 경우)를 제어합니다.
          toolChoice: auto
          # 최종 사용자를 나타내는 고유 식별자를 설정하여 남용을 모니터링하고 감지합니다.
          user: user-123
          # 단일 프롬프트 요청에서 함수 호출을 활성화할 함수 이름 목록을 지정합니다.
          functions: 
          # 스트리밍 요청에 대한 토큰 사용 통계를 추가하는 청크를 설정합니다.
          stream-usage: false
          # 도구 사용 중 병렬 함수 호출을 활성화합니다.
          parallel-tool-calls: true
          # 채팅 완료 요청에 추가할 선택적 HTTP 헤더를 지정합니다.
          http-headers:
            # 채팅 API 요청을 위한 Authorization 헤더입니다.
            Authorization: "Bearer your-chat-api-key"
          # 함수 호출을 내부적으로 처리하지 않고 클라이언트로 프록시할지 여부를 설정합니다.
          proxy-tool-calls: false
```

<br/>

### 4.3. 런타임 설정
---

`OpenAiChatOptions` 클래스는 사용할 Model, Temperature, Frequency Penalty 등의 모델 구성을 제공한다.

시작 시 기본 옵션은 `OpenAiChatModel(api, options)` 생성자나 `spring.ai.openai.chat.options.*` 속성을 사용하여 구성할 수 있다.

런타임에 `Prompt` 호출에 새로운 요청별 옵션을 추가하여 기본 옵션을 재정의할 수 있다.

```kotlin
val response: ChatResponse = chatModel.call(
    Prompt(
        "Generate the names of 5 famous pirates.",
        OpenAiChatOptions.builder()
            .withModel("gpt-4-o")
            .withTemperature(0.4)
        .build()
    ))
```

<br/>

### 4.4. Function 호출
---

`OpenAiChatModel` 에 사용자 정의 함수를 등록하면 OpenAI 모델이 하나 이상의 등록된 함수를 호출하기 위한 인수를 포함하는 JSON 객체를 출력하도록 할 수 있다.

<br/>

### 4.5. Multimodal
---

OpenAI는 텍스트, 비전, 오디오 입력을 지원한다.

<br/>

#### 4.5.1. Vision
---

비전 멀티모달 지원을 제공하는 OpenAI 모델에는 `gpt-4` , `gpt-4o` , `gpt-4o-mini` 가 있다.

OpenAI의 User Message API는 메시지에 base64로 인코딩된 이미지 또는 이미지 URL 목록을 전달할 수 있다. Spring AI의 Message 인터페이스는 Media 타입을 도입하여 멀티모달 AI 모델을 용이하게 한다. 이 타입은 메시지의 첨부 파일에 대한 데이터와 세부 정보를 포함하며 Spring의 `org.springframework.util.MimeType` 과 원시 미디어에 대한 `org.springframework.core.io.Resource` 를 활용한다.

다음은 `gpt-4o` 모델을 사용하여 사용자 텍스트와 이미지를 융합하는 방법을 보여준다.

```kotlin
val imageResource = ClassPathResource("/multimodal.test.png")

val userMessage = UserMessage("Explain what do you see on this picture?",
        Media(MimeTypeUtils.IMAGE_PNG, this.imageResource))

val response: ChatResponse = chatModel.call(new Prompt(this.userMessage,
        OpenAiChatOptions.builder().withModel(OpenAiApi.ChatModel.GPT_4_O.getValue()).build()))
```

<br/>

또는 `gpt-4o` 모델을 사용하여 이미지 URL을 전달할 수도 있다.

```kotlin
val userMessage = UserMessage("Explain what do you see on this picture?",
        Media(MimeTypeUtils.IMAGE_PNG,
                "https://docs.spring.io/spring-ai/reference/_images/multimodal.test.png"))

val response: ChatResponse = chatModel.call(new Prompt(this.userMessage,
        OpenAiChatOptions.builder().withModel(OpenAiApi.ChatModel.GPT_4_O.getValue()).build()))
```

<br/>

#### 4.5.2. Audio
---

다음은 `gpt-4o-audio-preview` 모델을 사용하여 사용자 텍스트와 오디오 파일을 융합하는 방법을 보여준다.

```kotlin
val audioResource = ClassPathResource("speech1.mp3")

val userMessage = UserMessage("What is this recording about?",
        List.of(Media(MimeTypeUtils.parseMimeType("audio/mp3"), audioResource)))

val response: ChatResponse = chatModel.call(Prompt(List.of(userMessage),
        OpenAiChatOptions.builder().withModel(OpenAiApi.ChatModel.GPT_4_O_AUDIO_PREVIEW).build()))
```
- 여러 개의 오디오 파일을 전달할 수도 있다.

<br/>

#### 4.6.3. Output Audio
---

다음은 `gpt-4o-audio-prview` 모델을 사용하여 사용자 텍스트로 요청ㅎ며 오디오 바이트 배열로 응답 받는 예시이다.

```kotlin
val userMessage = UserMessage("Tell me joke about Spring Framework")

val response: ChatResponse = chatModel.call(Prompt(List.of(userMessage),
        OpenAiChatOptions.builder()
            .withModel(OpenAiApi.ChatModel.GPT_4_O_AUDIO_PREVIEW)
            .withOutputModalities(List.of("text", "audio"))
            .withOutputAudio(AudioParameters(Voice.ALLOY, AudioResponseFormat.WAV))
            .build()))

val text: String = response.getResult().getOutput().getContent() // audio transcript

val waveAudio: ByteArray = response.getResult().getOutput().getMedia().get(0).getDataAsByteArray() // audio data
```
- 오디오 출력을 생성하려면 `OpenAiChatOptions` 에서 `audio` 모달리티를 지정해야 한다.
- `AudioParameters` 클래스는 오디오 출력에 대한 음성 및 오디오 형식을 제공한다.

<br/>

### 4.6. Structured Outputs
---

OpenAI는 `JSON Schema` 를 준수하여 응답을 생성할 수 있도록 맞춤형 구조화된 출력 API를 제공한다. 이러한 API는 기존의 Spring AI 모델에 독립적인 구조화된 출력 변환기 외에도 향상된 제어와 정밀성을 제공한다.

<br/>

#### 4.6.1. 설정
---

Spring AI를 사용하면 `OpenAiChatOptions` 빌더를 사용하여 프로그래밍 방식 또는 애플리케이션 속성을 통해 응답 형식을 구성할 수 있다.

<br/>

**Chat Options Builder 사용**

아래와 같이 `OpenAiChatOptions` 빌더를 사용하여 프로그래밍 방식으로 응답 형식을 설정할 수 있다.

```kotlin
val jsonSchema: String = """  
        {            "type": "object",            "properties": {                "steps": {                    "type": "array",                    "items": {                        "type": "object",                        "properties": {                            "explanation": { "type": "string" },                            "output": { "type": "string" }                        },                        "required": ["explanation", "output"],                        "additionalProperties": false                    }                },                "final_answer": { "type": "string" }            },            "required": ["steps", "final_answer"],            "additionalProperties": false        }                """.trimIndent()  
val prompt: Prompt = Prompt(  
    "how can I solve 8x + 7 = -23",  
    OpenAiChatOptions.builder()  
        .withModel(OpenAiApi.ChatModel.GPT_4_O_MINI)  
        .withResponseFormat(ResponseFormat(ResponseFormat.Type.JSON_SCHEMA, this.jsonSchema))  
        .build()  
)  
val response: ChatResponse = this.openAiChatModel.call(this.prompt)
```

<br/>

**BeanOutputConverter 유틸리티와 통합**

`BeanOutputConverter` 유틸리티를 활용하여 도메인 객체에서 JSON 스키마를 자동으로 생성하고 구조화된 응답을 도메인별 인스턴스로 변환할 수 있다.

```kotlin
data class MathReasoning(  
    @get:JsonProperty(required = true, value = "steps")  
    val steps: Steps,  
    @get:JsonProperty(required = true, value = "final_answer")  
    val finalAnswer: String  
) {  
  
    data class Steps(  
        @get:JsonProperty(required = true, value = "items")  
        val items: Array<Items>  
    ) {  
  
        data class Items(  
            @get:JsonProperty(required = true, value = "explanation")  
            val explanation: String,  
            @get:JsonProperty(required = true, value = "output")  
            val output: String  
        )  
    }  
}  
  
val outputConverter = BeanOutputConverter(MathReasoning::class.java)  
  
val jsonSchema = outputConverter.jsonSchema
  
val prompt = Prompt(  
    "how can I solve 8x + 7 = -23",  
    OpenAiChatOptions.builder()  
        .withModel(ChatModel.GPT_4_O_MINI)  
        .withResponseFormat(ResponseFormat(ResponseFormat.Type.JSON_SCHEMA, jsonSchema))  
        .build()  
)  
  
val response = openAiChatModel.call(prompt)  
val content = response.getResult().getOutput().getContent()  
  
val mathReasoning = outputConverter.convert(content)
```

> `@get:JsonProperty(required = true, ...)` 어노테이션을 사용해야 한다. 이는 필드를 `required` 로 정확하게  표시하는 스키마를 생성하는 데 중요하다. 구조화된 응답이 올바르게 작동하도록 이를 의무화한다.
{: .prompt-info }

<br/>

**애플리케이션 속성을 통한 구성**

OpenAI 자동 구성을 사용하는 경우 다음과 같이 응답 형식을 구성할 수 있다.

```yaml
spring:
  ai:
    openai:
      # OpenAI API에 접근하기 위한 API 키를 설정합니다.
      api-key: YOUR_API_KEY
      chat:
        options:
          # 사용할 OpenAI 채팅 모델의 이름을 선택합니다 (예: gpt-4o-mini).
          model: gpt-4o-mini
          responseFormat:
            # 응답 형식 유형을 정의합니다 (예: JSON_SCHEMA).
            type: JSON_SCHEMA
            # JSON_SCHEMA 유형에 대한 응답 형식 스키마 이름을 지정합니다.
            name: MySchemaName
            # JSON_SCHEMA 유형을 사용할 때의 응답 형식 JSON 스키마를 지정합니다.
            schema: >
              {
                "type": "object",
                "properties": {
                  "steps": {
                    "type": "array",
                    "items": {
                      "type": "object",
                      "properties": {
                        "explanation": {
                          "type": "string"
                        },
                        "output": {
                          "type": "string"
                        }
                      },
                      "required": ["explanation", "output"],
                      "additionalProperties": false
                    }
                  },
                  "final_answer": {
                    "type": "string"
                  }
                },
                "required": ["steps", "final_answer"],
                "additionalProperties": false
              }
            # JSON_SCHEMA 유형의 응답 형식 스키마 준수 엄격성을 설정합니다.
            strict: true
```

<br/>

### 5. 컨트롤러 예시
---

다음과 같이 `OpenAiChatModel` 을 주입 받아서 컨트롤러를 구현할 수 있다.

```kotlin
@RestController  
class ChatController(  
    private val chatModel: OpenAiChatModel  
) {  
  
    @GetMapping("/ai/generate")  
    fun generate(  
        @RequestParam(value = "message", defaultValue = "Tell me a joke")  
        message: String  
    ): Map<String, String> {  
        return mapOf("generation" to chatModel.call(message))  
    }  
  
    @GetMapping("/ai/generateStream")  
    fun generateStream(  
        @RequestParam(value = "message", defaultValue = "Tell me a joke")  
        message: String  
    ): Flux<ChatResponse> {  
        val prompt = Prompt(UserMessage(message))  
        return chatModel.stream(prompt)  
    }  
}
```

<br/>

### 6. 수동 구성
---

`OpenAiChatModel` 은 `ChatModel` 과 `StreamingChatModel` 을 구현하고 저수준 `OpenAiApiClient` 클라이언트를 사용하여 OpenAI 서비스에 연결한다.

다음은 `OpenAiChatModel` 을 통해 텍스트를 생성하는 예시이다.

```kotlin
val openAiApi = OpenAiApi(System.getenv("OPENAI_API_KEY"))  
val openAiChatOptions = OpenAiChatOptions.builder()  
    .withModel("gpt-3.5-turbo")  
    .withTemperature(0.4)  
    .withMaxTokens(200)  
    .build()  
val chatModel = OpenAiChatModel(this.openAiApi, this.openAiChatOptions)  
  
val response: ChatResponse = this.chatModel.call(  
    Prompt("Generate the names of 5 famous pirates.")  
)  
  
// Or with streaming responses  
val response: Flux<ChatResponse> = this.chatModel.stream(  
    Prompt("Generate the names of 5 famous pirates.")  
)
```

- `OpenAiChatOptions` 는 채팅 요청에 대한 구성 정보를 제공한다.
- `OpenAiChatOptions.Builder` 는 fluent 옵션 빌더이다.

<br/>

### 7. 저수준 OpenAiApi Client
---

`OpenAiApi` 는 OpenAI Chat API를 위한 가벼운 클라이언트를 제공한다.

다음 클래스 다이어그램은 `OpenAiApi` 채팅 인터페이스와 구성 요소를 보여준다.

![openai-chat-api](/assets/img/openai-chat-api.jpg)

<br/>

다음은 API를 프로그래밍 방식으로 사용하는 방법을 보여주는 예시이다.

```kotlin
var openAiApi: OpenAiApi = OpenAiApi(System.getenv("OPENAI_API_KEY"))  
var chatCompletionMessage: ChatCompletionMessage = ChatCompletionMessage("Hello world", Role.USER)  
  
// Sync request  
var response: ResponseEntity<ChatCompletion> = this.openAiApi.chatCompletionEntity(  
    ChatCompletionRequest(List.of<E>(this.chatCompletionMessage), "gpt-3.5-turbo", 0.8, false)  
)  
  
// Streaming request  
var streamResponse: Flux<ChatCompletionChunk> = this.openAiApi.chatCompletionStream(  
    ChatCompletionRequest(List.of<E>(this.chatCompletionMessage), "gpt-3.5-turbo", 0.8, true)  
)
```

<br/>

## 5. OpenAI 함수 호출
---

`OpenAiChatModel` 에 사용자 정의 함수를 등록하면 OpenAI 모델이 하나 이상의 등록된 함수를 호출하기 위한 인수를 포함하는 JSON 객체를 출력하도록 구성할 수 있다. 이를 통해 LLM 기능을 외부 도구 및 API와 연결할 수 있다. 

OpenAI API는 함수를 직접 호출하지 않는다. 대신 모델은 함수를 호출하고 결과를 모델에 반환하여 대화를 완료하는 데 사용할 수 있는 JSON을 생성한다.

Spring AI는 사용자 정의 함수를 등록하고 호출하는 방법을 제공한다. 일반적으로 사용자 정의 함수는 `name`, `description`, `signature` 를 제공해야 하며, 이를 통해 모델에 함수가 기대하는 인수를 알릴 수 있다. `description` 은 모델이 함수를 언제 호출해야 하는지 이해하는데 도움이 된다.

개발자로서, AI 모델에서 보낸 함수 호출 인수를 받고, 결과를 모델에 다시 응답하는 함수를 구현해야 한다.

Spring AI는 `java.util.Function` 을 반환하는 `@Bean` 을 정의하고, `ChatModel` 을 호출할 때 옵션으로 빈 이름을 제공하여 작업을 수행할 수 있도록 한다.

Spring은 POJO(함수)를 AI 모델과 상호작용할 수 있는 적절한 어댑터 코드로 래핑하여 지루한 보일러플레이트 코드를 작성할 필요가 없도록 해준다.

`FunctionCallback` 인터페이스와 함께 제공되는 빌더 유틸리티 클래스로 콜백 함수의 구현과 등록을 간소화한다.

<br/>

### 5.1. 동작 방식
---

예를 들어, AI 모델이 가지고 있지 않은 정보(ex. 주어진 위치의 현재 온도)로 응답하기를 원한다고 가정해 보자.

AI 모델에 우리의 기능에 대한 메타데이터를 제공할 수 있고, AI 모델은 프롬프트를 처리할 때 그 정보를 검색하는 데 활용할 수 있다.

프롬프트 처리 중에 AI 모델이 주어진 위치의 온도에 대한 추가 정보가 필요하다고 판단하는 경우에, 서버 측에서 상호작용이 발생한다. AI 모델은 클라이언트 측 함수를 호출한다. AI 모델은 JSON으로 메서드 호출에 필요한 정보를 전달하며, 해당 함수를 실행하고 응답을 반환하는 것은 클라이언트의 책임이다.

`@Bean` 으로 함수 정의를 제공한 다음 프롬프트 옵션에서 함수의 빈 이름을 전달할 수 있다. 또한, 프롬프트에서 여러 개의 함수 빈 이름을 참조할 수도 있다.

<br/>

### 5.2. 빠른 시작
---

우리가 만든 함수를 호출하여 질문에 답하는 챗봇을 만들어보자. 챗봇의 응답을 지원하기 위해 위치를 가져와 해당 위치의 현재 날씨를 반환하는 함수를 등록한다.

모델이 "보스턴의 날씨는 어때요?" 와 같은 질문에 답해야 할 때 AI 모델은 클라이언트를 호출하여 함수에 전달할 인수로 위치 값을 제공한다.

함수는 SaaS 기반 날씨 서비스 API를 호출하고 날씨 응답을 모델로 반환하여 대화를 완료한다. 이 예제에서는 다양한 위치의 온도를 하드코딩하는 `MockWeatherService` 라는 간단한 구현체를 사용할 것이다.

```kotlin
import java.util.function.Function  
  
data class Request(val location: String, val unit: Unit)  
  
data class Response(val temp: Double, val unit: Unit)  
  
enum class Unit { C, F }  
  
class MockWeatherService : Function<Request, Response> {  
    override fun apply(t: Request): Response {  
        return Response(30.0, Unit.C)  
    }  
}
```

<br/>

#### 5.2.1. 함수를 빈으로 등록
---

OpenAiChatModel Auto-Configuration을 통해 Spring 컨텍스트에 사용자 정의 함수를 빈으로 등록하는 방법이 여러가지가 있다.

<br/>

##### 일반 함수

애플리케이션 컨텍스트에 `@Bean` 을 정의하는 방법이다.

Spring AI `ChatModel` 은 AI 모델을 통해 `FunctionCallback` 의 인스턴스를 생성한다. `@Bean` 의 이름은 `ChatOption` 으로 전달된다.

```kotlin
@Configuration  
class Config {  
    @Bean  
    @Description("Get the weather in location") // function description  
    fun currentWeather(): Function<Request, Response> {  
        return MockWeatherService()  
    }  
}
```
- `@Description` 은 선택 사항이며 모델이 함수를 호출할 시기를 이해하는 데 도움이 되는 함수 설명을 제공한다. AI 모델이 호출할 함수를 결정하는데 도움이 되는 속성이다.

<br/>

함수 설명을 제공하기 위한 다른 방법은 `Request` 에 `@JsonClassDescription` 주석을 사용하는 것이다.

```kotlin
@JsonClassDescription("Get the weather in location")  
data class Request(val location: String, val unit: Unit)  
  
@Configuration  
class Config {  
    @Bean  
    fun currentWeather(): Function<Request, Response> {  
        return MockWeatherService()  
    }  
}
```
- 이처럼 요청 객체에 정보를 주석으로 추가하는 것이 가장 좋다.

<br/>

##### FunctionCallback Wrapper

함수를 등록하는 다른 방법은 다음과 같이 `FunctionCallback` 을 만드는 것이다.

```kotlin
@Configuration  
class Config {  
    @Bean  
    fun weatherFunctionInfo(): FunctionCallback {  
        return FunctionCallback.builder()  
            .function("CurrentWeather", MockWeatherService()) // (1) function name and instance  
            .description("Get the weather in location") // (2) function description  
            .inputType(MockWeatherService.Request::class.java) // (3) function input type  
            .build()  
    }  
}
```
- 써드파티 `MockWeatherService` 함수를 래핑하고 `OpenAiChatModel` 에 `CurrentWeather` 함수로 등록한다.
- 또한, 함수 호출에 대한 JSON 스키마를 생성하는데 사용되는 설명과 입력 유형도 제공한다.

<br/>

#### 5.2.2. Chat Options에서 함수 지정
---

모델에 `CurrentWeather` 함수를 알리고 호출하려면 프롬프트 요청에서 해당 함수를 활성화해야 한다.

```kotlin
val chatModel: OpenAiChatModel = ...  
  
val userMessage: UserMessage = UserMessage("What's the weather like in San Francisco, Tokyo, and Paris?")
  
val response: ChatResponse = this.chatModel.call(  
    Prompt(  
        this.userMessage,  
        OpenAiChatOptions  
            .builder()  
            .withFunction("CurrentWeather")  
            .build()  
    )  
) // Enable the function  
  
logger.info("Response: {}", response)
```

<br/>

위의 사용자 질문은 `CurrentWeather` 함수에 대한 3번의 호출(각 도시당 1번)을 수행하고 최종 응답은 다음과 같다.

```
Here is the current weather for the requested cities:
- San Francisco, CA: 30.0°C
- Tokyo, Japan: 10.0°C
- Paris, France: 15.0°C
```

<br/>

#### 5.2.3. Prompt Options을 사용하여 함수 등록/호출
---

자동 구성 외에도 `Prompt` 요청에 따라 동적으로 콜백 함수를 등록할 수 있다.

```kotlin
val chatModel: OpenAiChatModel = ...  
  
val userMessage: UserMessage = UserMessage("What's the weather like in San Francisco, Tokyo, and Paris?")  
  
var promptOptions = OpenAiChatOptions.builder()  
    .withFunctionCallbacks(List.of(  
        FunctionCallback.builder()  
        .function("CurrentWeather", MockWeatherService()) // (1) function name and instance  
        .description("Get the weather in location") // (2) function description  
        .inputType(MockWeatherService.Request.class) // (3) function input type  
            .build())) // function code  
        .build()  
  
val response: ChatResponse = this.chatModel.call(Prompt(this.userMessage, this.promptOptions))
```
- 이 방식을 사용하면 사용자 입력에 따라 호출할 다양한 함수를 동적으로 선택할 수 있다.

<br/>

#### 5.2.4. Tool Context 지원
---

Spring AI는 이제 도구 컨텍스트를 통해 함수 콜백에 추가적인 컨텍스트 정보를 전달하는 것을 지원한다. 이 기능을 사용하면 함수 실행 내에서 사용할 수 있는 추가 데이터를 제공하여 함수 호출의 유연성과 성능을 향상시킬 수 있다.

`java.util.BiFunction` 의 두 번째 인수로 전달되는 컨텍스트 정보이다. `ToolContext` 에는 변경 불가능한 `Map<String, Object>` 가 포함되어 있어 키-값 쌍에 액세스할 수 있다.

<br/>

##### Tool Context 사용 방법

채팅 옵션을 빌드할 때 Tool Context를 설정하고 콜백에 `BiFunction` 을 사용할 수 있다.

```kotlin
val weatherFunction: BiFunction<Request, ToolContext, Response> =  
    BiFunction<Request, ToolContext, Response> { request, toolContext ->  
        val sessionId = toolContext.context["sessionId"] as String  
        val userId = toolContext.context["userId"] as String  
  
        // Use sessionId and userId in your function logic  
        val temperature = 0.0  
        if (request.location().contains("Paris")) {  
            temperature = 15.0  
        } else if (request.location().contains("Tokyo")) {  
            temperature = 10.0  
        } else if (request.location().contains("San Francisco")) {  
            temperature = 30.0  
        }  
        Response(temperature, 15, 20, 2, 53, 45, Unit.C)  
    }  
  
val options: OpenAiChatOptions = OpenAiChatOptions.builder()  
    .withModel(OpenAiApi.ChatModel.GPT_4_O.getValue())  
    .withFunctionCallbacks(  
        List.of(  
            FunctionCallback.builder()  
                .function("getCurrentWeather", this.weatherFunction)  
                .description("Get the weather in location")  
                .inputType(Request::class.java)  
                .build()  
        )  
    )  
    .withToolContext(mapOf("sessionId" to "123", "userId" to "user456"))  
    .build()
```
- Tool Conext를 BiFunction으로 정의함. 이를 통해 함수 로직 내에서 컨텍스트에 직접 액세스 할 수 있다.

<br/>

채팅 모델을 호출할 때 다음 옵션을 사용할 수 있다.

```kotlin
val userMessage: UserMessage = UserMessage("What's the weather like in San Francisco, Tokyo, and Paris?")  
val response: ChatResponse = chatModel.call(Prompt(listOf(this.userMessage), options))
```
- 이러한 방식을 사용하면 세션별 또는 사용자별 정보를 함수에 전달하여 더욱 상황에 맞고 개인화된 응답을 제공할 수 있다.

### 5.3. 부록
---

#### 5.3.1. Spring AI 함수 호출 흐름
---

다음 다이어그램은 `OpenAiChatModel` 함수 호출의 흐름이다.

![openai-chatclient-function-call](/assets/img/openai-chatclient-function-call.jpg)

<br/>

#### 5.3.2. OpenAI API 함수 호출 흐름
---

다음 다이어그램은 OpenAI API 함수 호출의 흐름을 보여준다.

![openai-function-calling-flow](/assets/img/openai-function-calling-flow.jpg)

<br/>

## 6. OpenAI Text-to-Speech(TTS)
---

### 6.1. 음성 속성
---

#### 6.1.1. 설정 속성
---

```yaml
spring:
  ai:
    openai:
      # 연결 URL
      base-url: https://api.openai.com/
      # API Key
      api-key: YOUR_API_KEY
      # 조직 ID
      organization-id: your-organization-id
      # 프로젝트 ID
      project-id: your-project-id
      audio:
        speech:
          options:
            # 사용할 모델의 ID를 설정합니다. 현재 tts-1만 사용 가능합니다.
            model: tts-1
            # TTS 출력에 사용할 음성을 설정합니다. 사용 가능한 옵션: alloy, echo, fable, onyx, nova, shimmer.
            voice: alloy
            # 오디오 출력의 형식을 설정합니다. 지원되는 형식: mp3, opus, aac, flac, wav, pcm.
            response-format: mp3
            # 음성 합성의 속도를 설정합니다. 허용 범위는 0.25(가장 느림)에서 4.0(가장 빠름)까지입니다.
            speed: 1.0
```

<br/>

### 6.2. 런타임 옵션
---

`OpenAiAudioSpeechOptions` 클래스는 텍스트 음성 변환 요청을 할 때 사용할 수 있는 옵션을 제공한다. `spring.ai.openai.audio.speech` 에서 지정한 옵션이 사용되지만 런타임에 이를 재정의할 수 있다.

```kotlin
val speechOptions: OpenAiAudioSpeechOptions = OpenAiAudioSpeechOptions.builder()  
    .withModel("tts-1")  
    .withVoice(OpenAiAudioApi.SpeechRequest.Voice.ALLOY)  
    .withResponseFormat(OpenAiAudioApi.SpeechRequest.AudioResponseFormat.MP3)  
    .withSpeed(1.0f)  
    .build()  
val speechPrompt: SpeechPrompt = SpeechPrompt("Hello, this is a text-to-speech example.", this.speechOptions)  
val response: SpeechResponse = openAiAudioSpeechModel.call(this.speechPrompt)
```

<br/>

### 6.3. 수동 설정
---

다음과 같이 `OpenAiAudioSpeechModel` 생성할 수 있다.

```kotlin
val openAiAudioApi = OpenAiAudioApi(System.getenv("OPENAI_API_KEY"))  
val openAiAudioSpeechModel = OpenAiAudioSpeechModel(this.openAiAudioApi)  
  
val speechOptions = OpenAiAudioSpeechOptions.builder()  
    .withResponseFormat(OpenAiAudioApi.SpeechRequest.AudioResponseFormat.MP3)  
    .withSpeed(1.0f)  
    .withModel(OpenAiAudioApi.TtsModel.TTS_1.value)  
    .build()  
  
val speechPrompt = SpeechPrompt("Hello, this is a text-to-speech example.", this.speechOptions)  
val response: SpeechResponse = this.openAiAudioSpeechModel.call(this.speechPrompt)  
  
// Accessing metadata (rate limit info)  
val metadata: OpenAiAudioSpeechResponseMetadata = this.response.metadata  
val responseAsBytes: ByteArray = this.response.result.output
```

<br/>

### 6.4. 실시간 오디오 스트리밍
---

Speech API는 청크 전송 인코딩을 사용하여 실시간 오디오 스트리밍을 지원한다. 즉, 전체 파일이 생성되고 액세스 가능하게 되기 전에 오디오를 재생할 수 있다.

```kotlin
val openAiAudioApi = OpenAiAudioApi(System.getenv("OPENAI_API_KEY"))  
val openAiAudioSpeechModel = OpenAiAudioSpeechModel(this.openAiAudioApi)  
  
val speechOptions: OpenAiAudioSpeechOptions = OpenAiAudioSpeechOptions.builder()  
    .withVoice(OpenAiAudioApi.SpeechRequest.Voice.ALLOY)  
    .withSpeed(1.0f)  
    .withResponseFormat(OpenAiAudioApi.SpeechRequest.AudioResponseFormat.MP3)  
    .withModel(OpenAiAudioApi.TtsModel.TTS_1.value)  
    .build()  
  
val speechPrompt: SpeechPrompt =  
    SpeechPrompt("Today is a wonderful day to build something people love!", this.speechOptions)  
val responseStream: Flux<SpeechResponse> = this.openAiAudioSpeechModel.stream(this.speechPrompt)
```

<br/>

## 7. ETL Pipeline
---

Extract, Transform, Load (ETL) 프레임워크는 RAG(Retrieval Augmented Generation) 사례 내에서 데이터 처리의 중추 역할을 한다.

ETL 파이프라인은 원시 데이터 소스에서 구조화된 벡터 저장소로의 흐름을 조정하여 AI 모델이 검색할 수 있도록 데이터가 최적의 형식인지 확인한다.

RAG 사용 사례는 생성된 출력의 품질과 관련성을 향상시키기 위해 데이터에서 관련 정보를 검색하여 생성 모델의 기능을 증강하는 텍스트이다.

<br/>

### 7.1. API 개요
---

ETL 파이프라인은 `Document` 인스턴스를 생성, 변환 및 저장한다.

![spring-ai-document1-api](/assets/img/spring-ai-document1-api.jpg)

<br/>

`Document` 클래스에는 텍스트, 메타데이터가 포함되며, 선택적으로 이미지, 오디오, 비디오와 같은 추가 미디어 유형도 포함된다.

ETL 파이프라인에는 세 가지 주요 구성 요소가 있다.

- `Supplier<List<Document>>` 를 구현하는 `DocumentReader`
- `Function<List<Document>, List<Document>>` 를 구현하는 `DocumentTransformer`
- `Consumer<List<Document>>` 를 구현하는 `DocumentWriter`

 `DocumentReader` 가 PDF, 텍스트 파일 및 기타 문서 유형을 `Document` 로 생성한다.

<br/>

간단한 ETL 파이프라인을 구성하려면 각 타입의 인스턴스를 연결할 수 있다.

![etl-pipeline](/assets/img/etl-pipeline.jpg)

- `DocumentReader` 의 구현체인 `PagePdfDocumentReader` 
- `DocumentTransformer` 의 구현체인 `TokenTextSplitter`
- `DocumentWriter` 의 구현체인 `VectorStore`

<br/>

검색 증강 생성 패턴과 함께 사용하기 위해 벡터 데이터베이스에 기본적인 데이터 로딩을 수행하는 코드는 다음과 같다.

```kotlin
vectorStore.accept(tokenTextSplitter.apply(pdfReader.get()));
```

<br/>

또는, 다음과 같이 수행할 수도 있다.

```kotlin
vectorStore.write(tokenTextSplitter.split(pdfReader.read()));
```

<br/>

### 7.2. DocumentReaders
---

#### 7.2.1. PDF Page
---

`PagePdfDocumentReader` 는 Apache PdfBox 라이브러리를 사용하여 PDF 문서를 분석한다.

다음과 같이 의존을 추가한다.

```kotlin
dependencies {  
    implementation("org.springframework.ai:spring-ai-pdf-document-reader:1.0.0-SNAPSHOT")  
}
```

<br/>

##### 예시

```kotlin
@Component  
class MyPagePdfDocumentReader {  
    val docsFromPdf: List<Document>  
        get() {  
            val pdfReader: PagePdfDocumentReader = PagePdfDocumentReader(  
                "classpath:/sample1.pdf",  
                PdfDocumentReaderConfig.builder()  
                    .withPageTopMargin(0)  
                    .withPageExtractedTextFormatter(  
                        ExtractedTextFormatter.builder()  
                            .withNumberOfTopTextLinesToDelete(0)  
                            .build()  
                    )  
                    .withPagesPerDocument(1)  
                    .build()  
            )  
  
            return pdfReader.read()  
        }  
}
```

<br/>

#### 7.2.2. PDF Paragraph
---

`ParagraphPdfDocumentReader` 는 PDF 카탈로그(ex. TOC) 정보를 사용하여 입력 PDF를 텍스트 단락으로 분할하고 단락당 하나의 문서를 출력한다. 참고로 모든 PDF 문서에 PDF 카탈로그가 포함되어 있는 것은 아니다.

<br/>

##### 예시

```kotlin
@Component  
class MyPagePdfDocumentReader {  
    val docsFromPdfWithCatalog: List<Document>  
        get() {  
            val pdfReader: ParagraphPdfDocumentReader = ParagraphPdfDocumentReader(  
                "classpath:/sample1.pdf",  
                PdfDocumentReaderConfig.builder()  
                    .withPageTopMargin(0)  
                    .withPageExtractedTextFormatter(  
                        ExtractedTextFormatter.builder()  
                            .withNumberOfTopTextLinesToDelete(0)  
                            .build()  
                    )  
                    .withPagesPerDocument(1)  
                    .build()  
            )  
  
            return pdfReader.read()  
        }  
}
```

<br/>

### 7.3. Transformers
---

#### 7.3.1. TextSplitter
---

`TextSplitter` 는 AI 모델의 컨텍스트 윈도우에 맞게 문서를 나누는 데 도움이 되는 추상 클래스이다.

<br/>

#### 7.3.2. TokenTextSplitter
---

`TokenTextSplitter` 는 `CL_100K_BASE` 인코딩을 사용하여 토큰 수에 따라 텍스트를 청크로 분할하는 `TextSplitter` 의 구현체이다.

<br/>

##### 사용법

```kotlin
@Component  
class MyTokenTextSplitter {  
    fun splitDocuments(documents: List<Document>): List<Document> {  
        val splitter: TokenTextSplitter = TokenTextSplitter()  
        return splitter.apply(documents)  
    }  
  
    fun splitCustomized(documents: List<Document>): List<Document> {  
        val splitter: TokenTextSplitter = TokenTextSplitter(1000, 400, 10, 5000, true)  
        return splitter.apply(documents)  
    }  
}
```

<br/>

##### 생성자

- `TokenTextSplitter` 는 두 가지 생성자를 제공한다.
	1. `TokenTextSplitter()` : 기본 설정으로 splitter를 생성
	2. `TokenTextSplitter(int defaultChunkSize, int minChunkSizeChars, int minChunkLengthToEmbed, int maxNumChunks, boolean keepSeparator)`

<br/>

##### 파라미터

- `defaultChunkSize` : 토큰의 각 테스트 청크의 크기 (default: 800)
- `minChunkSizeChars` : 각 텍스트 청크의 최소 크기 (default: 350)
- `minChunkLengthToEmbed` : 포함될 청크의 최소 길이 (default: 5)
- `maxNumChunks` : 텍스트에서 생성할 청크의 최대 개수 (default: 10000)
- `keepSeparator` : 청크에서 구분 기호(줄 바꿈 등)를 유지할지 여부 (default: true)

<br/>

##### 동작

`TokenTextSplitter` 는 다음과 같이 텍스트 콘텐츠를 처리한다.
1. CL100K_BASE 인코딩을 사용하여 입력 테스트를 토큰으로 인코딩한다.
2. 인코딩된 텍스트를 `defaultChunkSize` 에 따라 청크로 분할한다.
3. 청크를 순회
	1. 이는 청크를 다시 텍스트로 디코딩한다.
	2. `minChunkSizeChars` 뒤에 적합한 중단점(마침표, 물음표, 느낌표 도는 줄 바꿈)을 찾는다.
	3. 중단점이 발견되면 해당 지점의 청크를 잘라낸다.
	4. `keepSeparator` 설정에 따라 청크를 잘라내고 선택적으로 줄 바꿈 문자를 제거한다.
	5. 결과 청크가 `minChunkLengthToEmbed` 보다 길면 출력에 추가된다.
4. 이 프로세스는 모든 토큰이 처리되거나 `maxNumChunks` 에 도달할 때까지 계속된다.
5. 남은 텍스트가 `minChunkLegthToEmbed` 보다 길면 최종 청크로 추가된다.

<br/>

##### 예시

```kotlin
val doc1 = Document(  
    "This is a long piece of text that needs to be split into smaller chunks for processing.",  
    mapOf("source" to "example.txt")  
)  
val doc2 = Document(  
    "Another document with content that will be split based on token count.",  
    mapOf("source" to "example2.txt")  
)  
  
val splitter = TokenTextSplitter()  
val splitDocuments = this.splitter.apply(listOf(this.doc1, this.doc2))  
  
splitDocuments.forEach { doc ->  
    println("Chunk: ${doc.formattedContent}")  
    println("Metadata: ${doc.metadata}")  
}
```

<br/>

#### 7.3.3. ContentFormatTransformer
---

모든 문서에서 동일한 컨텐츠 형식을 보장한다.

<br/>

#### 7.3.4. KeywordMetadataEnricher
---

`KeywordMetadataEnricher` 는 생성 AI 모델을 사용하여 문서 콘텐츠에서 키워드를 추출하고 이를 메타데이터로 추가하는 `DocumentTransformer` 이다.

<br/>

##### 사용법

```kotlin
@Component  
class MyKeywordEnricher(  
    private val chatModel: ChatModel  
) {  
  
    fun enrichDocuments(documents: List<Document>): List<Document> {  
        val enricher: KeywordMetadataEnricher = KeywordMetadataEnricher(this.chatModel, 5)  
        return enricher.apply(documents)  
    }  
}
```

<br/>

##### 생성자

두 개의 매개변수를 사용한다.

1. `ChatModel chatModel` : 키워드를 생성할 AI 모델
2. `int keywordCount` : 각 문서로부터 추출할 키워드 개수

<br/>

##### 동작

다음과 같이 문서를 처리한다.

1. 각 입력 문서에 대해 문서의 내용을 사용하여 프롬프트를 만든다.
2. 제공된 ChatModel에 프롬프트를 전송하여 키워드를 생성한다.
3. 생성된 키워드는 "excerpt_keywords" 키 아래에 문서 메타데이터에 추가된다.
4. 보강된 문서가 반환된다.

<br/>

##### 커스터마이징

키워드 추출 프롬프트는 클래스에서 `KEYWORD_TEMPLATE` 상수를 수정하여 사용자 정의할 수 있다. 기본 템플릿은 다음과 같다.

```
\{context_str}. Give %s unique keywords for this document. Format as comma separated. Keywords:
```
- `{content_str}` 은 문서 내용으로 대체되고, `%s` 는 지정된 키워드 수로 대체된다.

<br/>

##### 예시

```kotlin
val chatModel: OpenAiApi.ChatModel = // initialize your chat model  
val enricher = KeywordMetadataEnricher(chatModel, 5)  
  
val doc = Document("This is a document about artificial intelligence and its applications in modern technology.")  
  
val enrichedDocs: List<Document> = enricher.apply(listOf(this.doc))  
  
val enrichedDoc: Document = this.enrichedDocs[0]  
val keywords: String = this.enrichedDoc.metadata["excerpt_keywords"].toString()  
println("Extracted keywords: $keywords")
```

<br/>

#### 7.3.5. SummaryMetadataEnricher
---

`SummaryMetadataEnricher` 는 생성적 AI 모델을 사용하여 문서에 대한 요약을 만들고 이를 메타데이터로 추가하는 `DocumentTransformer` 이다. 현재 문서와 인접 문서(이전 및 다음)에 대한 요약을 생성할 수 있다.

##### 사용법

```kotlin
@Configuration  
class EnricherConfig {  
    @Bean  
    fun summaryMetadata(aiClient: OpenAiChatModel): SummaryMetadataEnricher {  
        return SummaryMetadataEnricher(  
            aiClient,  
            listOf(SummaryType.PREVIOUS, SummaryType.CURRENT, SummaryType.NEXT)  
        )  
    }  
}  
  
@Component  
class MySummaryEnricher(  
    private val enricher: SummaryMetadataEnricher  
) {  
  
    fun enrichDocuments(documents: List<Document>): List<Document> {  
        return enricher.apply(documents)  
    }  
}
```

<br/>

##### 생성자

두 생성자를 지원한다.

1. `SummaryMetadataEnricher(ChatModel chatModel, List<SummaryType> summaryTypes)`
2. `SummaryMetadataEnricher(ChatModel chatModel, List<SummaryType> summaryTypes, String summaryTemplate, MetadataMode metadataMode)`

<br/>

##### 파라미터

- `chatModel` : 요약 생성에 사용될 AI 모델
- `summaryTypes` : 생성할 요약을 나타내는 `SummaryType` 열거형 값의 목록 (`PREVIOUS` , `CURRENT` , `NEXT`)
- `summaryTemplate` : 요약 생성을 위한 사용자 정의 템플릿 (optional)
- `metadataMode` : 요약을 생성할 때 문서 메타데이터를 처리하는 방법을 지정 (optional)

<br/>

##### 동작

1. 각 입력 문서에 대해 문서의 내용과 지정된 요약 템플릿을 사용하여 프롬프트를 만든다.
2. 제공된 `ChatModel` 에 프롬프트를 전송하여 요약을 생성한다.
3. 지정된 `summaryTypes` 에 따라 각 문서에 다음 메타데이터가 추가된다.
	1. `section_summary` : 현재 문서의 요약
	2. `prev_section_summary` : 이전 문서의 요약 (요청시)
	3. `next_section_summary` : 다음 문서의 요약 (요청시)
4. 보강된 문서를 반환

<br/>

##### 커스터마이징

요약 생성 프롬프트는 사용자 정의 `summaryTemplate` 을 제공하여 사용자 정의할 수 있다. 기본 템플릿은 다음과 같다.

```
""" 
Here is the content of the section: 
{context_str} 

Summarize the key topics and entities of the section. 

Summary: 
"""
```

<br/>

##### 예시

```kotlin
val chatModel: ChatModel = // initialize your chat model  
val enricher = SummaryMetadataEnricher(  
    chatModel,  
    listOf(SummaryType.PREVIOUS, SummaryType.CURRENT, SummaryType.NEXT)  
)  
  
val doc1 = Document("Content of document 1")  
val doc2 = Document("Content of document 2")  
  
val enrichedDocs: List<Document> = enricher.apply(listOf(this.doc1, this.doc2))  
  
// Check the metadata of the enriched documents  
enrichedDocs.forEach { doc ->  
    println("Current summary: " + doc.metadata["section_summary"])  
    println("Previous summary: " + doc.metadata["prev_section_summary"])  
    println("Next summary: " + doc.metadata["next_section_summary"])  
}
```

<br/>

## Reference
---

[Spring: Spring AI](https://docs.spring.io/spring-ai/reference/index.html)