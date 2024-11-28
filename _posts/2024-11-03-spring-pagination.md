---
title: Spring Paginration
date: 2024-11-03 09:00:00 +0900
categories: spring
tags:
  - spring
  - pagination
  - webflux
  - kotlin
description: Pagination in Spring Webflux and Spring Data Reactive
---

## 1. 개요

---

본 글에는 다음 세 가지에 대해 작성되어 있다.

- 데이터 조회에 있어서 Pagination의 중요성
- Pagination 사용에 있어서 Spring Data Reactive와 Spring Data의 차이점
- Pagination 구현 예제

<br/>

## 2. Pagination의 중요성

---

Pagination은 대용량 데이터를 응답할 때 필수적이다. Pagination은 **대용량 데이터를 작은 덩어리인 페이지 단위로 나누어 데이터를 효율적으로 조회하고 응답**할 수 있도록 한다.

<br/>

## 3. Spring Data와 Spring Data Reactive의 차이

---

**Spring Data**는 Java 애플리케이션의 **데이터 접근을 단순화하고 향상시키는 것을 목표**로 하는 Spring Framework의 프로젝트 중 하나이다. Spring Data는 보일러 플레이트 코드를 줄이고 모범 사례를 촉진하여 코드를 단순화하는 공통 추상화 및 기능을 제공한다.

[Spring Data Pagination example](https://www.baeldung.com/spring-data-jpa-pagination-sorting)에서 설명했듯이 _page, size, sort_ 로 구성된 [_PageRequest_](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html)를 사용하여 페이지를 구성하고 요청할 수 있다. Spring Data에는 pagination과 sorting 추상화를 사용하여 엔티티 검색 메서드를 제공하는 [_PagingAndSortingRepository_](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html) 인터페이스가 정의되어 있다. 해당 Repository의 메서드에서 [_Page_](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Page.html) 정보를 반환하기 위해 [_Pageable_](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Pageable.html) 과 [_Sort_](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.html) 객체를 사용할 수 있다. _Page_ 객체에는 내부적으로 쿼리를 실행할 때 사용되는 _totalElements_와 _totalPages_ 속성이 포함되어 있다. 이 정보들은 다음 페이지를 요청할 때 사용될 수 있다.

반면에, **Spring Data Reactive**는 **pagination을 완전하게 지원하지는 않는다.** 이러한 이유는 Spring Reactive는 비동기 논블로킹을 지원하므로 **특정 페이지 크기에 해당하는 모든 데이터가 반환될 때까지 블로킹이 되면 효율적이지 않기 때문**이다. 그러나 Spring Data Reacrive는 여전히 _Pageable_을 지원한다. _PageRequest_ 객체를 사용하여 특정 데이터의 청크와 데이터의 개수를 조회하도록 구성할 수 있다.

페이지와 레코드에 대한 메타데이터가 포함된 Spring Data를 사용할 때 Page 대신 Flux로 구성된 응답을 받을 수 있다.

<br/>

## 4. WebFlux와 Data Reactive를 활용한 Pagination 예제 코드

---

_Page_와 _Size_ 정보를 갖고 있는 _Pageable_ 객체를 통해 Repository로부터 페이징된 목록을 조회할 수 있다.

```kotlin
@Repository
interface NotificationRepository : CoroutineCrudRepository<NotificationEntity, UUID> {

    suspend fun findAllByReceiverIdOrderByNotifiedDateDesc(
        receiverId: UUID,
        pageable: Pageable,
    ): Flow<NotificationEntity>

    suspend fun countByReceiverId(
        receiverId: UUID,
    ): Long
}
```

- 페이징 정보와 별도로 전체 개수를 조회할 수 있도록 구성함


Repository로부터 페이징된 리스트를 조회하기 위해 다음과 같이 _PageRequest.of_를 활용하여 _Pageable_ 객체를 생성한다.

```kotlin
data class GetNotificationsRequest(
    val memberId: UUID,
    private val page: Int,
    private val size: Int,
) {
    val pageable: Pageable = PageRequest.of(page, size)
}
```

```kotlin
@Component
class NotificationQueryAdapter(
    private val notificationRepository: NotificationRepository,
) : LoadNotificationPort {

    override suspend fun loadCount(memberId: UUID): Long =
        notificationRepository.countByReceiverId(memberId)

    override suspend fun loadNotifications(request: GetNotificationsRequest): Flow<Notification> =
        notificationRepository
            .findAllByReceiverIdOrderByNotifiedDateDesc(request.memberId, request.pageable)
            .map { it.toDomain() }
}
```

그 후 조회된 결과와 전체 개수로 PageImpl 객체를 생성한다.

```kotlin
@Service
class NotificationQueryService(
    private val loadNotificationPort: LoadNotificationPort,
) : GetNotificationUseCase {

    override suspend fun getNotifications(request: GetNotificationsRequest): Page<NotificationResponse> {
        val notifications = loadNotificationPort.loadNotifications(request).map { it.toResponse() }
        val count = loadNotificationPort.loadCount(request.memberId)
        return PageImpl(notifications.toList(), request.pageable, count)
    }

}
```

Page 정보를 곧바로 응답하면 에러가 발생하므로, 다음과 같이 PageResponse 사용자 정의 객체로 변환하여 응답한다.

```kotlin
data class PageResponse<T>(
    val data: List<T>,
    val pageable: PageableResponse,
) {
    companion object {
        fun <T> Page<T>.toResponse() =
            PageResponse(
                data = this.content,
                pageable = this.toPageable(),
            )
    }

    data class PageableResponse(
        val totalPages: Int,
        val totalElements: Long,
    ) {
        companion object {
            fun Page<*>.toPageable() =
                PageableResponse(
                    totalPages = this.totalPages,
                    totalElements = this.totalElements
                )
        }
    }
}
```

```kotlin
@RestController
class NotificationRouter(
    private val getMemberUseCase: GetMemberUseCase,
    private val getNotificationUseCase: GetNotificationUseCase,
) {

    @GetMapping("/notifications")
    suspend fun getNotifications(
        @RequestHeader(AUTHORIZATION)
        authorization: String,
        @RequestParam(name = "page", defaultValue = "0")
        page: Int,
        @RequestParam(name = "size", defaultValue = "10")
        size: Int,
    ): PageResponse<NotificationResponse> {
        val memberId: UUID = getMemberUseCase.getMemberIdBy(authorization)
        val request = GetNotificationsRequest(memberId, page, size)
        val notifications: Page<NotificationResponse> = getNotificationUseCase.getNotifications(request)
        return notifications.toResponse()
    }

}
```

<br/>

## 5. 예제 저장소

---

[feat: 알림 페이징 조회 구현](https://github.com/LeeSM0518/notification-service/commit/a1bfabd573a3fb89b5c293eb4cc969400101317b)

<br/>

## Reference

---

<https://www.baeldung.com/spring-data-webflux-pagination>