---
title: Spring SSE(Server-Sent Events)
date: 2024-11-01 08:59:00 +0900
categories: spring
tags:
  - spring
  - notification
description: Spring SSE에 대해 알아보고 적용하는 방법에 대한 가이드
---

## 1. 개요
---

**SSE(Server-Sent-Events)** 는 웹 애플리케이션이 단방향 이벤트 스트림을 처리하고 서버가 데이터를 내보낼 때마다 업데이트를 받을 수 있도록 하는 HTTP 표준이다.

<br/>

## 2. SSE 구현
---

### 2.1. 컨트롤러 구현
---

SSE 스트리밍 엔드포인트를 생성하려면 MIME type을 _text/event-stream_ 으로 지정해야 하지만, 응답을 ServerSentEvent 타입으로 매핑할 경우 MIME type을 지정하지 않아도 된다.

```kotlin
@RestController
class NotificationRouter {

    @GetMapping("/notification")
    suspend fun streamNotification(): Flow<ServerSentEvent<String>> =
        Flux.interval(Duration.ofSeconds(1))
            .map { sequence ->
                ServerSentEvent
                    .builder<String>()
                    .id(sequence.toString())
                    .event("periodic-event")
                    .data("\\"SSE\\" : \\"${LocalDate.now()}\\"")
                    .build()
            }
            .asFlow()
}
```
- 재연결 시간을 지정하는 _comment_ 속성과 _retry_ 값을 추가할 수도 있다.
- data는 JSON 형식이 아닐 경우 에러가 발생하므로 주의해야 한다.
    - JSON으로 구성하지 않을 경우 다음과 같은 에러 발생함

<br/>

### 2.2. 테스트 구현
---

WebTestClient를 사용하여 요청을 보내보고 응답이 정상적으로 오는지 확인

```kotlin
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
internal class NotificationRouterTest @Autowired constructor(
    private val webTestClient: WebTestClient,
) {

    @Test
    fun be_able_to_receive_notification(): Unit = runBlocking {
        val result = webTestClient
            .get()
            .uri("/notification")
            .accept(MediaType.TEXT_EVENT_STREAM)
            .exchange()
            .expectStatus().isOk
            .returnResult(ServerSentEvent::class.java)

        val eventStream = result.responseBody.blockFirst()!!
        assertThat(eventStream.event()).isEqualTo("periodic-event")
    }
}
```

<br/>

## 3. SSE 이해

---

SSE는 단방향 스트리밍 이벤트를 활용하기 위해 대부분의 브라우저에서 채택한 사양이다. 이벤트는 UTF-8로 인코딩된 텍스트 데이터 스트림이다. 이벤트의 형식은 줄바꿈으로 구분된 일련의 키-값 요소(id, retry, data, event, name, comment)로 구성된다.

스트리밍 기능을 사용할 때 고려할 점은 SSE 스트리밍과 WebSocket 사용의 차이점이다. WebSocket은 서버와 클라이언트 간의 양방향 통신을 제공하는 반면 SSE는 단방향 통신을 사용한다. 또한 WebSocket은 HTTP 프로토콜이 아니며 SSE와 달리 오류 처리 표준을 제공하지 않는다. 이러한 내용들을 참고하여 서비스에 적합한 통신 방식을 선택하여 사용해야 한다.

<br/>

## 4. 예제 코드
---

[GitHub: notification-service](https://github.com/LeeSM0518/notification-service/tree/%235/feat/apply-tutorial-about-spring-sse)

<br/>

## Reference

---

[Baeldung: Server-Sent Events in Spring](https://www.baeldung.com/spring-server-sent-events)