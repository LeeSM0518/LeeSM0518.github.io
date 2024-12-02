---
title: DB 세션 누수 해결
date: 2024-12-02 00:00:00 +0900
categories: troubleshooting
tags:
  - troubleshooting
description: 누수 발생 지점을 찾아 해결하고, 세션 및 트랜잭션 관리를 추상화하여 서버 안정화와 코드 개선
---

## 개요
---

회사 서비스에서 DB 세션 누수가 발생하여 원인을 파악하고 문제를 해결한 작업에 대해 기록하고 회고한다.

<br/>

## 배경 및 목적
---

고객사에서 서비스의 여러 기능을 사용하면서 데이터가 많이 증가하지 않았음에도 불구하고 전반적인 기능이 **느려지는 현상이 발생**했다. 심지어 서버가 오랫동안 켜져 있으면 **재시작하는 문제가 발생**하여 긴급하게 해결을 진행했다.

<br/>

## 현상 재현
---

고객사와 동일한 문제가 사내에서도 발생하는지 확인해보기 위해 현상을 재현해봤다. 크롬으로 서비스에 접속해서 빠르게 여러 기능들을 번갈아가며 사용해보니, **점차 느려지고 슬로우 쿼리가 발생**하는 것을 볼 수 있었다. 데이터베이스의 **성능이 부족**한 것인지, **커넥션 풀이 부족**한 것인지, **세션 누수**가 발생하는 것인지를 확인하기 위해 원인 분석을 진행했다.

<br/>

## 원인 분석
---

다음 쿼리를 통해 현재 활동 중인 모든 세션을 살펴봤다. 하루가 지나도 종료되지 않은 세션이 있는 것을 확인하여 **세션 누수가 발생**하고 있음을 발견했다. 종료되지 않은 쿼리를 실행하는 코드를 분석한 결과, **세션을 닫지 않고 작업을 완료**하여 세션이 종료되지 않고 있음을 확인할 수 있었다.

```sql
SELECT * FROM pg_stat_activity;
```
{: file='연결 세션 확인 쿼리'}

<br/>

문제가 발생한 서버는 Spring을 사용하지 않고 Koin이라는 의존 주입 프레임워크를 사용하고 있었으며, JPA가 아닌 Hibernate를 활용하여 세션을 직접 열고 닫으며 데이터베이스에 접근하고 있었다. 이로 인해 개발자가 세션을 직접 관리하다 보니 실수로 세션을 닫지 않는 문제가 발생하게 된 것이다.

```kotlin
open class HibernateRepository<T : Any, ID : Serializable>(private val kClass: KClass<T>) : Repository<T, ID> {
    private var managingSession: Session? = null
    protected val session: Session
        get() = managingSession ?: HibernateManager.getSession().also { managingSession = it }
	  
    // ...
	  
   override fun close() {
        managingSession?.close()
    }
}
```
{: file='직접 구현해서 사용하던 HibernateRepository'}

<br/>

## 1차 문제 해결
---

서버에 세션 누수 문제가 발생하지 않도록 세션을 닫지 않던 코드를 `use` 를 사용하여 작업이 완료되면 **세션을 닫아주도록 변경**했다.

```kotlin
// use를 사용하지 않고 세션을 종료하지 않은 코드 예시 
accessHistoryRepo.findBy(accountId)

// use를 사용하여 세션을 종료한 코드 예시 
accessHistoryRepo.use { it.findBy(accountId) } 
```

<br/>

위와 같이 코드를 수정하여 문제는 해결했지만, 세션 관리를 개발자가 직접 하도록 유지할 경우 지금과 같은 **문제가 반드시 다시 발생할 것**이라고 예상됐다. 데이터베이스 접근 로직을 수정하는 것은 매우 큰 작업이었지만, 지금과 같은 치명적인 문제가 다시 발생하는 것을 막기 위해 긴 고민 끝에 **세션 관리를 추상화하는 작업을 진행**했다.

<br/>

## 2차 문제 해결
---

기존에는 일반 메서드로 구성된 `Repository` 인터페이스에 `@Deprecated` 를 선언하고, `suspend` 메서드로 구성된 `CoroutineRepository` 인터페이스를 정의했다.

```kotlin
@Deprecated(  
    message = "replace",  
    replaceWith = ReplaceWith(  
        "CoroutineRepository<T, ID>",  
        "com.sia.core.repository.domain.CoroutineRepository"  
    )  
)  
interface Repository<T : Any, ID : Any> : Closeable {  
    
    fun <S : T> save(entity: S): S  
    // ...
}
```
{: file='Repository Interface'}

```kotlin
interface CoroutineRepository<T : Any, ID : Any> {  
  
    suspend fun <S : T> save(entity: S): S   
    // ...
}
```
{: file='CoroutineRepository Interface'}

<br/>

그 다음으로 `CoroutineRepository` 인터페이스에 전역 함수로 `withTransactional` 함수를 구현했다. 이 함수는 개발자가 세션과 트랜잭션을 관리하지 않도록 하기 위해 세션을 사용하는 함수를 전달 받아서 **세션 및 트랜잭션 처리를 함수 내부에서 수행**하도록 구현했다. 또한, 기존 코드에는 모두 `use` 가 반영되어 있으므로, 기존 코드를 전부 변경하지 않고 활용할 수 있도록 `use` 호출 시 `use` 내부의 로직을 `withTransactional` 로 감싸서 수행하는 `use` 함수를 구현했다.

```kotlin
companion object {  
    val sessionThreadLocal = ThreadLocal<Session>()  
  
    suspend fun <T> withTransactional(block: suspend (Session) -> T): T =  
        withContext(Dispatchers.IO + sessionThreadLocal.asContextElement()) {  
            var session = sessionThreadLocal.get()  
  
            if (session == null || session.isOpen.not()) {  
                session = HibernateManager.getSession()  
                session.beginTransaction()  
                sessionThreadLocal.set(session)  
                val result =  
                    runCatching {  
                        block(session)  
                    }.onSuccess {  
                        session.transaction.commit()  
                    }.onFailure {  
                        session.transaction.rollback()  
                    }  
                session.close()  
                result.getOrThrow()  
            } else {  
                block(session)  
            }  
        }  
  
    suspend fun <T, R> T.use(block: suspend (T) -> R): R =  
        withTransactional {  
            block(this)  
        }  
}
```
{: file='CoroutineRepository Interface의 companion object'}

<br/>

`CoroutineRepository` 인터페이스를 `CoroutineHibernateRepository` 클래스로 다음과 같이 구현했다.

```kotlin
open class CoroutineHibernateRepository<T : Any, ID : Serializable>(
	private val kClass: KClass<T>
) : CoroutineRepository<T, ID> {  
  
    protected val session: Session  
        get() = sessionThreadLocal.get()  
  
    override suspend fun <S : T> save(entity: S): S =  
        withTransactional {  
            // ...
        }

	// ...

}
```
{: file='CoroutineHibernateRepository'}

<br/>

위와 같이 정의 및 구현한 인터페이스와 클래스를 기반으로 기존 로직들을 다음과 같이 변경했다. 개선 이전에는 트랜잭션 처리를 직접 해줬으나, 개선 이후에는 **트랜잭션 처리를 구현하지 않아도 되어 코드가 단순**해진 것을 확인할 수 있었다.

```kotlin
class NotificationHibernateRepository : NotificationRepository, 
  HibernateRepository<AdminNotification, UUID>(AdminNotification::class) {

  override fun update(id: UUID) {
    val sql = "UPDATE ..."
	session.transaction.begin()
	try {
	  session.createNativeQuery(sql).setParameter("id", id).executeUpdate()
	  session.transaction.commit()
	} catch (e: Exception) {
	  session.transaction.rollback()
	  throw e
	}
  }
  
  // ...

}
```
{: file='개선 이전 코드'}

```kotlin
class NotificationHibernateRepository : NotificationRepository, 
  CoroutineHibernateRepository<AdminNotification, UUID>(AdminNotification::class) {

  override suspend fun update(id: UUID) {
    withTransactional {
      val sql = "UPDATE ..."
      session.createNativeQuery(sql).setParameter("id", id).executeUpdate()
    }
  }

  // ...

}
```
{: file='개선 이후 코드'}

<br/>

위와 같이 `HibernateRepository` 를 사용하던 Repository들을 전부 `CoroutineRepository` 로 변경하고, 세션이나 트랜잭션에 접근하던 코드를 전부 제거하여 **약 400개의 파일을 수정함으로써 개선을 완료**했다. 개선 이후 팀원들이 `CoroutineRepository` 를 잘 활용할 수 있도록 **가이드 문서를 작성한 후에 공유**했으며, 매주 진행하는 백엔드 미팅에서 **사용 가이드 문서를 설명**해줬다.

<br/>

## 정리 및 회고
---

DB 세션 누수로 인해 서버가 다운되는 심각한 문제가 발생했으나, 활성 세션 분석과 코드 분석을 통해 누수 발생 지점을 찾아 해결했다. 뿐만 아니라 문제 재발 방지를 위해 세션 및 트랜잭션 관리를 추상화하여 **약 400개 파일의 코드를 수정함으로써 서버 안정화와 코드 개선을 이뤄냈다.** 이번 버그 수정 작업을 통해 **세션 관리와 추상화의 중요성**을 배웠다. 앞으로는 세션이나 트랜잭션과 같은 자원 관리나 개발자가 실수할 수 있는 부분은 반드시 추상화하여 개발하도록 하자.