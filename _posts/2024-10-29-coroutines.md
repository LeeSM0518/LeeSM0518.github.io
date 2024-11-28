---
title: Coroutines
date: 2024-10-29 09:00:00 +0900
categories: kotlin
tags:
  - coroutines
  - kotlin
description: Kotlin Corutines 가이드
---

## 1. 코루틴
---

### 1.1. 코루틴과 일시 중단 함수
---

**일시 중단 함수**는 일반적인 함수를 더 일반화해 함수 본문의 원하는 지점에서 **함수에 필요한 모든 런타임 문맥을 저장하고 함수 실행을 중단**한 다음, **나중에 필요할 때 다시 실행**을 계속 진행할 수 있게 한 것이다.

코틀린에서는 이런 함수에 `suspend` 라는 변경자를 붙인다.

```kotlin
suspend fun foo() {
    println("${System.currentTimeMillis()} : Task started (${Thread.currentThread()})")
    delay(1000)
    println("${System.currentTimeMillis()} : Task finished (${Thread.currentThread()})")
}

suspend fun main() {
    foo()
    println("${System.currentTimeMillis()} : Main (${Thread.currentThread()})")
}
```

```
1720589906842 : Task started (Thread[main,5,main])
1720589907854 : Task finished (Thread[kotlinx.coroutines.DefaultExecutor,5,main])
1720589907855 : Main (Thread[kotlinx.coroutines.DefaultExecutor,5,main])
```

`delay()` 함수는 `Thread.sleep()` 과 비슷한 일을 한다. 하지만 현재 스레드를 블럭시키지 않고 자신을 호출한 함수를 일시 중단시키며 스레드를 다른 작업을 수행할 수 있게 풀어준다.

동시성 함수는 주로 공통적인 생명 주기와 문맥이 정해진 몇몇 작업이 정의된 구체적인 영역 안에서만 호출한다.

코루틴을 실행할 때 사용하는 여러 가지 함수를 **코루틴 빌더(coroutine builder)** 라고 부른다.

코루틴 빌더는 `CoroutineScope` 인스턴스의 확장 함수로 쓰인다. CoroutineScope에 대한 구현 중 가장 기본적인 것으로 `GlobalScope` 객체가 있다. GlobalScope 객체를 사용하면 독립적인 코루틴을 만들 수 있고, **이 코루틴은 자신만의 작업을 내포**할 수 있다.

자주 사용하는 `launch()` , `async()` , `runBlocking()` 이라는 코루틴 빌더를 살펴보자.

<br/>

### 1.2. 코루틴 빌더
---

`launch()` 함수는 **코루틴을 시작**하고, 코루틴을 실행 중인 작업의 상태를 추적하고 변경할 수 있는 `Job` 객체를 돌려준다. 이 함수는 `CoroutineScope.() -> Unit` 타입의 일시 중단 람다를 받는다.

```kotlin
import java.lang.System.currentTimeMillis
import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch

fun main() {
    val time = currentTimeMillis()

    GlobalScope.launch {
        delay(100)
        println("Task 1 finished in ${currentTimeMillis() - time} ms (${Thread.currentThread()})")
    }

    GlobalScope.launch {
        delay(100)
        println("Task 2 finished in ${currentTimeMillis() - time} ms (${Thread.currentThread()})")
    }

    GlobalScope.launch {
        delay(100)
        println("Task 3 finished in ${currentTimeMillis() - time} ms (${Thread.currentThread()})")
    }

    GlobalScope.launch {
        delay(100)
        println("Task 4 finished in ${currentTimeMillis() - time} ms (${Thread.currentThread()})")
    }

    GlobalScope.launch {
        delay(100)
        println("Task 5 finished in ${currentTimeMillis() - time} ms (${Thread.currentThread()})")
    }

    GlobalScope.launch {
        delay(100)
        println("Task 6 finished in ${currentTimeMillis() - time} ms (${Thread.currentThread()})")
    }

    Thread.sleep(200)
}
```

```
Task 5 finished in 131 ms (Thread[DefaultDispatcher-worker-3,5,main])
Task 6 finished in 131 ms (Thread[DefaultDispatcher-worker-1,5,main])
Task 3 finished in 131 ms (Thread[DefaultDispatcher-worker-6,5,main])
Task 4 finished in 131 ms (Thread[DefaultDispatcher-worker-5,5,main])
Task 1 finished in 131 ms (Thread[DefaultDispatcher-worker-4,5,main])
Task 2 finished in 131 ms (Thread[DefaultDispatcher-worker-2,5,main])
```

- **DefaultDispatcher-worker-n** : 스레드의 이름
	- **DefaultDispatcher** : 코루틴 디스패처 중 하나인 DefaultDispatcher에 의해 생성된 스레드임을 알 수 있음. 해당 디스패처는 주로 코어 수에 비례한 풀 크기를 가지며, CPU 집중적인 작업에 사용된다.
	- **worker-n** : DefaultDispatcher에서 생성된 스레드 중 하나이며, worker-n은 그 중 n 번째 스레드 임을 의미한다.
- **n** : 스레드 우선순위
	- 스레드 우선순위는 1(가장 낮음) 부터 10(가장 높음)까지 설정할 수 있으며, 5는 기본 우선순위 이다.
- **main** : 스레드 그룹
	- 이 스레드가 속한 스레그 그룹의 이름을 의미한다.

<br/>

모든 작업이 동시에 끝났다는 점에서 모든 작업이 실제로 **병렬적으로 실행**됐다는 것을 알 수 있다. 다만 **실행 순서가 보장되지 않으므로** 다른 작업이 먼저 표시될 수 있다. 코루틴 라이브러리는 필요할 때 **실행 순서를 강제할 수 있는 도구도 제공**한다.

코루틴을 처리하는 스레드는 **데몬 모드로 실행**되기 때문에 main 스레드가 코루틴보다 빨리 끝나버리면 자동으로 실행이 종료된다.

**코루틴은 스레드보다 훨씬 가볍다.** 특히 코루틴은 유지해야 하는 상태가 더 간단하며 일시 중단되고 재개될 때 완전한 문맥 전환을 사용하지 않아도 되므로 **엄청난 수의 코루틴을 충분히 동시에 실행할 수 있다.**

launch() 빌더는 동시성 작업이 결과를 만들어내지 않는 경우 적합하다. 그러므로 결과가 필요한 경우에는 `async()` 라는 다른 빌더 함수를 사용해야 한다. 이 함수는 `Deferred` 의 인스턴스를 돌려주고, 이 인스턴스는 Job의 하위 타입으로 `await()` 메서드를 통해 계산 결과에 접근할 수 있게 해준다. **await()는 계산이 완료되거나 계산 작업이 취소될 때까지 현재 코루틴을 일시 중단**시킨다.

```kotlin
suspend fun main() {
    println("Start : ${currentTimeMillis()} (${Thread.currentThread()})")
    val message = GlobalScope.async {
        println("Task 1 Start : ${currentTimeMillis()} (${Thread.currentThread()})")
        delay(1000)
        println("Task 1 End : ${currentTimeMillis()} (${Thread.currentThread()})")
        "abc"
    }

    val count = GlobalScope.async {
        println("Task 2 Start : ${currentTimeMillis()} (${Thread.currentThread()})")
        delay(2000)
        println("Task 2 End : ${currentTimeMillis()} (${Thread.currentThread()})")
        1 + 2
    }

    delay(4000)

    println("Middle : ${currentTimeMillis()} (${Thread.currentThread()})")
    val result = message.await().repeat(count.await())
    println("End : ${currentTimeMillis()}, $result (${Thread.currentThread()})")
}
```

```
Start : 1720592103548 (Thread[main,5,main])
Task 2 Start : 1720592103564 (Thread[DefaultDispatcher-worker-1,5,main])
Task 1 Start : 1720592103564 (Thread[DefaultDispatcher-worker-2,5,main])
Task 1 End : 1720592104574 (Thread[DefaultDispatcher-worker-2,5,main])
Task 2 End : 1720592105569 (Thread[DefaultDispatcher-worker-2,5,main])
Middle : 1720592107569 (Thread[kotlinx.coroutines.DefaultExecutor,5,main])
End : 1720592107583, abcabcabc (Thread[kotlinx.coroutines.DefaultExecutor,5,main])
```

launch()와 async() 빌더의 경우 (일시 중단 함수 내부에서) 스레드 호출을 블럭시키지 않지만, **백그라운드 스레드를 공유하는 풀을 통해 작업을 실행**한다.

<br/>

`runBlocking()` 빌더는 디폴트로 현재 스레드에서 실행되는 코루틴을 만들고 **코루틴이 완료될 때까지 현재 스레드의 실행을 블럭**시킨다.

```kotlin
suspend fun main() {
    GlobalScope.launch {
        delay(100)
        println("Background task : ${Thread.currentThread()}")
    }

    runBlocking {
        println("Primary task : ${Thread.currentThread()}")
        delay(200)
        GlobalScope.launch {
            delay(300)
            println("Primary Background task : ${Thread.currentThread()}")
        }.join()
    }
}
```

```
Primary task : Thread[main,5,main]
Background task : Thread[DefaultDispatcher-worker-1,5,main]
Primary Background task : Thread[DefaultDispatcher-worker-1,5,main]
```


runBlocking() 내부의 코루틴은 메인 스레드에서 실행된 반면, launch()로 시작한 코루틴은 공유 풀에서 백그라운드 스레드를 할당받았음을 알 수 있다.

이런 블러킹 동작 때문에 **runBlocking()을 다른 코루틴 안에서 사용하면 안된다.** runBlocking()은 블러킹 호출과 넌블러킹 호출 사이의 다리 역할을 하기 위해 고안된 코루틴 빌더이므로, **테스트나 메인 함수에서 최상위 빌더로 사용**하는 등의 경우에만 써야 한다.

<br/>

### 1.3. 코루틴 영역과 구조적 동시성
---

지금까지 살펴본 예제 코루틴은 전역 영역(global scope)에서 실행됐다. **전역 영역이란 코루틴의 생명 주기가 전체 애플리케이션의 생명 주기에 의해서만 제약되는 영역**이다.

동시성 작업 사이의 부모 자식 관계를 통해 실행 시간 제한이 가능하다. **어떤 코루틴을 다른 코루틴의 문맥에서 실행하면** 후자가 전자의 **부모**가 된다. 이 경우 **자식의 실행이 모두 끝나야 부모가 끝날 수 있도록 부모와 자식의 생명 주기가 연관된다.**

이런 기능을 **구조적 동시성(structured concurrency)** 이라고 부르며, 지역 변수 영역 안에서 블럭이나 서브 루틴을 사용하는 경우와 구조적 동시성을 비교할 수 있다. 예제를 살펴보자.

```kotlin
fun main() {
    runBlocking {
        println("Parent task started : ${Thread.currentThread()}")

        launch {
            println("Task A started : ${Thread.currentThread()}")
            delay(200)
            println("Task A finished : ${Thread.currentThread()}")
        }

        launch {
            println("Task B started : ${Thread.currentThread()}")
            delay(200)
            println("Task B finished : ${Thread.currentThread()}")
        }

        delay(100)
        println("Parent task finished : ${Thread.currentThread()}")
    }
    println("Shutting down... : ${Thread.currentThread()}")
}
```

```
Parent task started : Thread[main,5,main]
Task A started : Thread[main,5,main]
Task B started : Thread[main,5,main]
Parent task finished : Thread[main,5,main]
Task A finished : Thread[main,5,main]
Task B finished : Thread[main,5,main]
Shutting down... : Thread[main,5,main]
```

이 코드는 최상위 코루틴을 시작하고 현재 CoroutineScope 인스턴스 안에서 launch를 호출해 두 가지 자식 코루틴을 시작한다.

<br/>

`coroutineScope()` 호출로 코드 블럭을 감싸면 커스텀 영역을 도입할 수도 있다. runBlocking()과 마찬가지로 coroutineScope() 호출은 람다의 결과를 반환하고, 자식들이 완료되기 전까지 실행이 완료되지 않는다.

```kotlin
fun main() {
    runBlocking {
        println("Parent task started : ${Thread.currentThread()}")

        coroutineScope {
            launch {
                println("Task A started : ${Thread.currentThread()}")
                delay(200)
                println("Task A finished : ${Thread.currentThread()}")
            }

            launch {
                println("Task B started : ${Thread.currentThread()}")
                delay(200)
                println("Task B finished : ${Thread.currentThread()}")
            }
        }

        println("Parent task finished : ${Thread.currentThread()}")
    }
    println("Shutting down... : ${Thread.currentThread()}")
}
```

```
Parent task started : Thread[main,5,main]
Task A started : Thread[main,5,main]
Task B started : Thread[main,5,main]
Task A finished : Thread[main,5,main]
Task B finished : Thread[main,5,main]
Parent task finished : Thread[main,5,main]
Shutting down... : Thread[main,5,main]
```

- coroutineScope()은 일시 중단 함수라 현재 스레드를 블럭시키지 않는다.
- 두 자식 코루틴 실행이 끝날 때까지 앞의 coroutineScope() 호출이 일시 중단되므로 Parent task finished 메시지가 마지막에 표시된다.

<br/>

### 1.4. 코루틴 문맥

---

코루틴마다 `CoroutineContext` 인터페이스로 표현되는 문맥이 연관돼 있으며, 코루틴을 감싸는 변수 영역의 **coroutineContext 프로퍼티를 통해 이 문맥에 접근**할 수 있다. 문맥은 키-값 쌍으로 이뤄진 불변 컬렉션이며, 코루틴에서 사용할 수 있는 여러 가지 데이터가 들어 있다.

- 코루틴이 실행 중인 취소 가능한 작업을 표현하는 잡(job)
- 코루틴과 스레드의 연관을 제어하는 디스패처(dispatcher)

<br/>

일반적으로 문맥은 `CoroutineContext.Element` 를 구현하는 아무 데이터나 저장할 수 있다. 특정 원소에 접근하려면 `get()` 메서드나 인덱스 연산자에 키를 넘겨야 한다.

```kotlin
fun main() {
    runBlocking {
        launch {
            println("Job.Key : ${Job.Key}")
            println("coroutineContext : $coroutineContext")
            println("Task is active : ${coroutineContext[Job.Key]!!.isActive} (${Thread.currentThread()})")
        }
    }
}
```

```
Job.Key : kotlinx.coroutines.Job$Key@6325a3ee
coroutineContext : [StandaloneCoroutine{Active}@7b49cea0, BlockingEventLoop@887af79]
Task is active : true (Thread[main,5,main])
```

- 기본적으로 launch(), async() 등의 표준 코루틴 빌더에 의해 만들어지는 코루틴은 현재 문맥을 이어받는다.

<br/>

필요하면 빌더 함수에 context 파라미터를 지정해서 새 문맥을 넘길 수도 있다. 새 **문맥을 만들려면 두 문맥의 데이터를 합쳐주는 plus() 함수나 + 연산자를 사용**하거나, 주어진 키에 해당하는 원소를 문맥에서 제거해주는 minusKey() 함수를 사용하면 된다.

```kotlin
private fun CoroutineScope.showName() {
    println("Current coroutine : ${coroutineContext[CoroutineName]?.name}")
}

fun main() {
    runBlocking {
        showName()
        launch(coroutineContext + CoroutineName("TEST")) {
            showName()
        }
    }
}
```

```
Current coroutine : null
Current coroutine : TEST
```

코루틴을 실행하는 중간에 `withContext()`에 **새 문맥과 일시 중단 람다를 넘겨서 문맥을 전환**시킬 수도 있다.

<br/>

## 2. 코루틴 흐름 제어와 잡 생명 주기

---

잡은 동시성 작업의 생명 주기를 표현하는 객체다. 잡을 사용하면 작업 상태를 추적하고 필요할 때 작업을 취소할 수 있다.

활성화 상태(디폴트 상태)는 작업이 시작됐고 아직 완료나 취소로 끝나지 않았다는 뜻이다.

즉, **잡은 생성되자마자 활성화** 상태가 된다.

<br/>

launch()나 async()는 CoroutineStart 타입의 인자를 지정해서 잡의 초기 상태를 선택하는 기능을 제공하기도 한다.

- **_CoroutineStart.DEFAULT_** : 디폴트 동작, 즉시 시작
- **_CoroutineStart.LAZY_** : 신규 상태가 되고 시작을 기다림

```kotlin
fun main() {
    runBlocking {
        val job = launch(start = CoroutineStart.LAZY) {
            println("Job started")
        }
        println("delay start")
        delay(100)
        println("preparing to start")
        job.start()
        println("end")
    }
}
```

```
delay start
preparing to start
end
Job started
```

활성화 상태에서는 코루틴 장치가 잡을 반복적으로 일시 중단하고 재개시킨다.

<br/>

잡의 부모 자식 관계는 트리 형태의 의존 구조를 만든다.

children 프로퍼티를 통해 완료되지 않은 자식 잡들을 얻을 수 있다.

```kotlin
fun main() {
    runBlocking {
        val job = coroutineContext[Job.Key]!!

        launch { println("This is task A") }
        launch { println("This is task B") }

        println("${job.children.count()} children running")
        delay(100)
        println("${job.children.count()} children running")
    }
}
```

```
2 children running
This is task A
This is task B
0 children running
```

<br/>

잡의 join() 메서드를 사용하면 조인 대상 잡이 완료될 때까지 현재 코루틴을 일시 중단시킬 수 있다.

```kotlin
fun main() {
    runBlocking {
        val job = coroutineContext[Job.Key]!!

        val jobA = launch { println("This is task A") }
        val jobB = launch { println("This is task B") }

        jobA.join()
        jobB.join()

        println("${job.children.count()} children running")
    }
}
```

```
This is task A
This is task B
0 children running
```

<br/>

### 2.1. 취소
---

잡의 cancel() 메서드를 호출하면 잡을 취소할 수 있다.

취소를 할 경우에는 코루틴이 스스로 취소가 요청됐는지 검사해서 적절히 반응해줘야 한다.

```kotlin
fun main() {
    runBlocking {
        val squarePrinter = GlobalScope.launch(Dispatchers.Default) {
            var i = 1
            while (true) {
                println(i++)
            }
        }

        squarePrinter.cancel()
        println("after cancel")
    }
}
```

```
1
2
...
after cancel
...
```

- 계속해서 호출됨

<br/>

```kotlin
fun main() {
    runBlocking {
        val squarePrinter = GlobalScope.launch(Dispatchers.Default) {
            var i = 1
            while (isActive) {
                println(i++)
            }
        }

        squarePrinter.cancel()
        println("after cancel")
    }
}
```

```
1
...
after cancel
```

- cancel이 호출될 경우 잡이 활성화된 상태인지 검사하는 isActive가 false로 변경되어 코루틴이 중단된다.

<br/>

다른 방법은 상태를 검사하는 대신에 CancellationException 을 발생시키면서 취소에 반응할 수 있게 일시 중단 함수를 호출하는 것이다. delay()나 join 등의 모든 일시 중단 함수가 이 예외를 발생시켜준다.

```kotlin
fun main() {
    runBlocking {
        val squarePrinter = GlobalScope.launch(Dispatchers.Default) {
            var i = 1
            while (true) {
                yield()
                println(i++)
            }
        }

        squarePrinter.cancel()
        println("after cancel")
    }
}
```

```
1
...
after cancel
```

<br/>

부모 코루틴이 취소되면 자동으로 모든 자식의 실행을 취소한다.

```kotlin
fun main() {
    runBlocking {
        val parentJob = launch {
            println("P Start")

            launch {
                println("C1 Start")
                delay(500)
                println("C1 End")
            }

            launch {
                println("C2 Start")
                delay(500)
                println("C2 End")
            }

            delay(500)
            println("P End")
        }
        delay(100)
        parentJob.cancel()
    }
```

```
P Start
C1 Start
C2 Start
```

<br/>

### 2.2. 타임아웃
---

작업이 완료되기를 무작정 기다릴 수 없어서 타임아웃을 설정해야 할 때가 있다.

이럴 때 사용할 수 있는 `withTimeout()` 이라는 함수를 제공한다.

```kotlin
fun main() {
    runBlocking {
        val asyncData = async { delay(500) }
        try {
            val text = withTimeout(50) { asyncData.await() }
            println("Data loaded: $text")
        } catch (e: Exception) {
            println("Timeout exceeded")
        }
    }
}
```

```
Timeout exceeded
```

비슷한 함수로 `withTimeoutOrNull()` 이라는 것도 있다.

<br/>

### 2.3. 코루틴 디스패치하기
---

코루틴을 실행하려면 스레드와 연관시켜야 한다. 특정 코루틴을 실행할 때 사용할 스레드를 제어하는 작업을 담당하는 특별한 컴포넌트가 있다. 이 컴포넌트를 **코루틴 디스패처(dispatcher)** 라고 부른다.

디스패처는 코루틴 문맥의 일부다. 따라서 launch()나 runBlocking() 등의 코루틴 빌더 함수에서 이를 지정할 수 있다. 코루틴 빌더에 디스패처를 넘길 수도 있다.

```kotlin
fun main() {
    runBlocking {
        launch(Dispatchers.Default) {
            println(Thread.currentThread())
        }
    }
}
```

```
Thread[DefaultDispatcher-worker-1,5,main]
```

<br/>

코루틴 디스패처는 병렬 작업 사이에 스레드를 배분해주는 자바 실행기(executor)와 비슷하다.

실제로 asCoroutineDispatcher() 확장 함수를 사용하면 기존 실행기 구현을 그에 상응하는 코루틴 디스패처로 쉽게 바꿀 수 있다.

```kotlin
fun main() {
    val id = AtomicInteger(0)

    val executor = ScheduledThreadPoolExecutor(5) { runnable ->
        Thread(
            runnable,
            "WorkerThread-${id.incrementAndGet()}"
        ).also { it.isDaemon = true }
    }

    executor.asCoroutineDispatcher().use { dispatcher ->
        runBlocking {
            for (i in 1..3) {
                launch(dispatcher) {
                    println(Thread.currentThread())
                    delay(1000)
                }
            }
        }
    }
}
```

```
Thread[WorkerThread-1,5,main]
Thread[WorkerThread-2,5,main]
Thread[WorkerThread-3,5,main]
```

<br/>

기본적으로 몇 가지 디스패처 구현을 제공한다.
- **Dispatchers.Default** : 공유 스레드 풀로, 풀 크기는 디폴트로 사용 가능한 CPU 코어 수이거나 2다(둘 중 큰 값). 이 구현은 일반적으로 작업 성능이 주로 CPU 속도에 의해 결정되는 CPU 위주의 작업에 적합하다.
- **Dispatchers.IO** : 스레드 풀 기반이며 디폴트 구현과 비슷하지만, 파일을 읽고 쓰는 것처럼 잠재적으로 블러킹될 수 있는 I/O를 많이 사용하는 작업에 최적화돼 있다. 이 디스패처는 스레드 풀을 디폴트 구현과 함께 공유하지만, 필요에 따라 스레드를 추가하거나 종료시켜준다.
- **Dispatchers.Main** : UI 스레드에서만 배타적으로 작동하는 디스패처

<br/>

디스패처를 명시적으로 지정하지 않으면 코루틴을 시작한 영역으로부터 디스패처가 자동으로 상속된다.

```kotlin
fun main() {
    runBlocking {
        println("Root : ${Thread.currentThread()}, $coroutineContext")

        launch {
            println("Nested, inherited : ${Thread.currentThread()}, $coroutineContext")
        }

        launch(Dispatchers.Default) {
            println("Nested, explicit 1 : ${Thread.currentThread()}, $coroutineContext")
        }

        launch(Dispatchers.IO) {
            println("Nested, explicit 2 : ${Thread.currentThread()}, $coroutineContext")
        }
    }
}
```

```
Root : Thread[main,5,main], [BlockingCoroutine{Active}@5383967b, BlockingEventLoop@2ac273d3]
Nested, explicit 1 : Thread[DefaultDispatcher-worker-1,5,main], [StandaloneCoroutine{Active}@5d1ba695, Dispatchers.Default]
Nested, explicit 2 : Thread[DefaultDispatcher-worker-1,5,main], [StandaloneCoroutine{Active}@7836cc1d, Dispatchers.IO]
Nested, inherited : Thread[main,5,main], [StandaloneCoroutine{Active}@cd2dae5, BlockingEventLoop@2ac273d3]
```

<br/>

부모 코루틴이 없으면 암시적으로 Dispatchers.Default로 디스패처를 가정한다. 다만 runBlocking() 빌더는 현재 스레드를 사용한다. 코루틴이 생명 주기 내내 같은 디스패처를 사용할 필요도 없다. 디스패처가 코루틴 문맥의 일부이므로 withContext() 함수를 사용해 디스패처를 오버라이드 할 수 있다.

```kotlin
fun main() {
    newSingleThreadContext("Worker").use { worker ->
        runBlocking {
            println(Thread.currentThread())
            withContext(worker) {
                println(Thread.currentThread())
            }
            println(Thread.currentThread())
        }
    }
}
```

```
Thread[main,5,main]
Thread[Worker,5,main]
Thread[main,5,main]
```

- 이 기법은 중단 가능 루틴의 일부를 한 스레드에서만 실행하고 싶을 때 유용하다.

<br/>

### 2.4. 예외 처리
---

**예외를 부모 코루틴에 전달**하여 처리할 수 있다. 이 경우 예외는 다음과 같이 전파된다.

1. 부모 코루틴이 자식과 똑같은 오류로 취소된다. 이로 인해 부모의 나머지 자식도 모두 취소된다.
2. 자식들이 모두 취소되고 나면 부모는 예외를 코루틴 트리의 윗부분으로 전달한다.

전역 영역에 있는 코루틴에 도달할 때까지 이 과정이 반복된다. 그 후 예외가 CoroutineExceptionHanlder.Consider에 의해 처리된다.

```kotlin
fun main() {
    runBlocking {
        println("Root Start")

        launch {
            throw Exception("Error in task A")
            println("Task A completed")
        }

        launch {
            delay(1000)
            println("Task B completed")
        }

        delay(1000)
        println("Root Completed")
    }
}
```

```
Root Start
Exception in thread "main" java.lang.Exception: Error in task A
	at Test20Kt$main$1$1.invokeSuspend(Test20.kt:12)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:104)
	at kotlinx.coroutines.EventLoopImplBase.processNextEvent(EventLoop.common.kt:277)
	at kotlinx.coroutines.BlockingCoroutine.joinBlocking(Builders.kt:95)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking(Builders.kt:69)
	at kotlinx.coroutines.BuildersKt.runBlocking(Unknown Source)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking$default(Builders.kt:48)
	at kotlinx.coroutines.BuildersKt.runBlocking$default(Unknown Source)
	at Test20Kt.main(Test20.kt:8)
	at Test20Kt.main(Test20.kt)
```

- 첫 번째 코루틴이 예외를 던져서 최상위 작업이 취소되고, 최상위의 자식인 두 작업도 취소된다.
- 커스텀 핸들러가 지정되지 않았기 때문에 Thread.uncaughtExceptionHandler에 등록된 디폴트 동작을 실행한다.

<br/>

CoroutineExceptionHandler 는 현재 코루틴 문맥(CoroutineContext)과 던져진 예외를 인자로 전달받는다.

```kotlin
fun hanldeException(context: CoroutineContext, exception: Throwable)
```

다음과 같이 핸들러를 만들 수 있다.

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
	println("Caught $exception")
	}
```

핸들러가 예외를 처리하도록 지정하려면 코루틴 문맥에 인스턴스를 넣어야 한다.

```kotlin
suspend fun main() {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("Caught $exception")
    }

    GlobalScope.launch(handler) {
        println("Root start")
        launch {
            throw Exception("Error in task A")
            println("Task A completed")
        }

        launch {
            delay(1000)
            println("Task B completed")
        }
        delay(1000)
        println("Root completed")
    }.join()
}
```

```
Root start
Caught java.lang.Exception: Error in task A
```

CoroutineExceptionHandler는 전역 영역에서 실행된 코루틴에 대해서만 정의할 수 있고, CoroutineExceptionHandler가 정의된 코루틴의 자식에 대해서만 적용된다. 그러므로 runBlocking()을 그대로 사용하면 예외 핸들러가 동작하지 않는다.

<br/>

예외를 처리하는 다른 방법은 던져진 예외를 저장했다가 예외가 발생한 계산에 대한 await() 호출을 받았을 때 다시 던지는 것이다.

```kotlin
suspend fun main() {
    runBlocking {
        val deferredA = async {
            throw Exception("Error in task A")
            println("Task A completed")
        }

        val deferredB = async {
            println("Task B completed")
        }

        deferredA.await()
        deferredB.await()
        println("Root completed")
    }
}
```

```
Exception in thread "main" java.lang.Exception: Error in task A
	at Test23Kt$main$2$deferredA$1.invokeSuspend(Test23.kt:7)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:104)
	at kotlinx.coroutines.EventLoopImplBase.processNextEvent(EventLoop.common.kt:277)
	at kotlinx.coroutines.BlockingCoroutine.joinBlocking(Builders.kt:95)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking(Builders.kt:69)
	at kotlinx.coroutines.BuildersKt.runBlocking(Unknown Source)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking$default(Builders.kt:48)
	at kotlinx.coroutines.BuildersKt.runBlocking$default(Unknown Source)
	at Test23Kt.main(Test23.kt:5)
	at Test23Kt$main$3.invoke(Test23.kt)
	at Test23Kt$main$3.invoke(Test23.kt)
	at kotlin.coroutines.intrinsics.IntrinsicsKt__IntrinsicsJvmKt$createCoroutineUnintercepted$$inlined$createCoroutineFromSuspendFunction$IntrinsicsKt__IntrinsicsJvmKt$1.invokeSuspend(IntrinsicsJvm.kt:270)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlin.coroutines.ContinuationKt.startCoroutine(Continuation.kt:115)
	at kotlin.coroutines.jvm.internal.RunSuspendKt.runSuspend(RunSuspend.kt:19)
	at Test23Kt.main(Test23.kt)
```

- catch가 동작하긴 하지만 프로그램은 여전히 예외와 함께 중단된다.
- 자식이 실패한 경우에는 부모를 취소시키기 위해 자동으로 예외를 다시 던지기 때문이다.

<br/>

위와 같은 동작을 변경하려면 **슈퍼바이저(supervisor)** 잡을 사용해야 한다.

부모 코루틴을 슈퍼바이저로 변환하려면 coroutineScope() 대신 `supervisorScope()` 함수를 사용해 새로운 영역을 정의해야 한다.

```kotlin
fun main() {
    runBlocking {
        supervisorScope {
            val deferredA = async {
                throw Exception("Error in task A")
                println("Task A completed")
            }

            val deferredB = async {
                println("Task B completed")
            }

            try {
                deferredA.await()
                deferredB.await()
            } catch (e: Exception) {
                println("Caught $e")
            }
        }
    }
    println("completed")
}
```

```
Task B completed
Caught java.lang.Exception: Error in task A
completed
```

- 슈퍼바이저의 동작은 일반적인 취소에도 적용된다.
- 슈퍼바이저의 자식 중 하나에 cancel()을 호출해도 해당 코루틴의 형제자매나 슈퍼바이저 자신에는 아무 영향이 없다.

<br/>

## 3. 동시성 통신
---

코루틴과 액터 사이에서 동기화나 락을 사용하지 않고도 **변경 가능한 상태를 안전하게 공유할 수 있는 데이터 스트림**을 제공하는 메커니즘인 **채널(channel)**에 대해 알아보자.

<br/>

### 3.1. 채널
---

채널은 임의의 데이터 스트림을 코루틴 사이에 공유할 수 있는 편리한 방법이다.

채널이 제공하는 연산들

- `send()` : 데이터 전송

- `receive()` : 데이터 수신

제네릭 Channel() 함수를 사용해 채널을 만들 수 있다. 이 함수는 채널의 용량을 지정하는 정숫값을 받는다.

버퍼가 꽉 차면 최소 하나 이상의 채널 원소가 상대방에 의해 수신될 때까지 send() 호출이 일시 중단된다.

이와 비슷하게, 버퍼가 비어있으면 최소 하나 이상의 원소를 채널로 송신할 때까지 receive() 호출이 일시 중단된다.

```kotlin
fun main() {
    runBlocking {
        val streamSize = 5
        val channel = Channel<Int>(3) // 채널 용량 = 3

        launch {
            for (n in 1..streamSize) {
                delay(Random.nextLong(100))
                val square = n * n
                println("Sending: $square")
                channel.send(square)
            }
        }

        launch {
            for (i in 1..streamSize) {
                delay(Random.nextLong(100))
                val n = channel.receive()
                println("Receiving: $n")
            }
        }
    }
}
```

```
Sending: 1
Receiving: 1
Sending: 4
Receiving: 4
Sending: 9
Receiving: 9
Sending: 16
Sending: 25
Receiving: 16
Receiving: 25
```

- 채널은 모든 값이 송신된 순서 그대로 수신되도록 보장한다.

<br/>

Channel() 함수는 채널의 동작을 바꿀 수 있는 여러 특별한 값을 받을 수 있다.

- **Channel.UNLIMITED (= Int.MAX_VALUE)** : 채널의 용량은 제한이 없고, 내부 버퍼는 필요에 따라 증가한다.
- **Channel.RENDEZVOUS (= 0)** : 아무 내부 버퍼가 없는 랑데부 채널이 된다.
- **Channel.CONFLATED (= -1)** : 송신된 값이 합쳐지는 채널. 최대 하나만 버퍼에 저장하고 다른 send() 요청이 오면 기존의 값을 덮어 쓴다.

<br/>

Channel API는 생산자 쪽에서 close() 메서드를 사용해 더 이상 데이터를 보내지 않는다는 신호를 보낼 수 있게 해준다. 소비자 쪽에서는 이터레이션 횟수를 고정하는 대신 채널에서 들어오는 데이터에 대해 이터레이션을 할 수 있다.

```kotlin
fun main() {
    runBlocking {
        val streamSize = 5
        val channel = Channel<Int>(Channel.CONFLATED)

        launch {
            for (n in 1..streamSize) {
                delay(100)
                val square = n * n
                println("Sending: $square")
                channel.send(square)
            }
            channel.close()
            println("Channel closed")
        }

        launch {
            for (n in channel) {
                println("Receiving: $n")
                delay(200)
            }
        }
    }
}
```

```
Sending: 1
Receiving: 1
Sending: 4
Receiving: 4
Sending: 9
Sending: 16
Receiving: 16
Sending: 25
Channel closed
Receiving: 25
```

<br/>

여러 생산자 코루틴이 한 채널에 써넣은 데이터를 한 소비자 코루틴이 읽는 팬 인(fan in)도 있다. 여러 생산자와 여러 소비자가 여러 채널을 공유할 수도 있다. 어떤 채널에 대해 receive()를 맨 처음 호출한 코루틴이 다음 원소를 읽게 된다.

```kotlin
fun main() {
    runBlocking {
        val streamSize = 5
        val channel = Channel<Int>(Channel.CONFLATED)

        launch {
            for (n in 1..streamSize) {
                delay(100)
                val square = n * n
                println("Sending: $square")
                channel.send(square)
            }
            channel.close()
            println("Channel closed")
        }

        launch {
            for (n in channel) {
                println("Receiving 1 : $n")
                delay(200)
            }
        }

        launch {
            for (n in channel) {
                println("Receiving 2 : $n")
                delay(200)
            }
        }
    }
}
```

```
Sending: 1
Receiving 1 : 1
Sending: 4
Receiving 2 : 4
Sending: 9
Receiving 1 : 9
Sending: 16
Receiving 2 : 16
Sending: 25
Channel closed
Receiving 1 : 25
```

<br/>

### 3.2. 생산자
---

sequence() 함수와 비슷하게 동시성 데이터 스트림을 생성할 수 있는 produce()라는 특별한 코루틴 빌더가 있다.

```kotlin
fun main() {
    runBlocking {
        CoroutineScope(Dispatchers.IO).launch {
            val channel = produce {
                for (n in 1..5) {
                    val square = n * n
                    println("Sending: $square (${currentThread()})")
                    kotlinx.coroutines.delay(1000)
                    send(square)
                }
            }

            launch {
                channel.consumeEach { println("Receving: $it (${currentThread()})") }
            }
        }.join()
    }
}
```

```
Sending: 1 (Thread[DefaultDispatcher-worker-3,5,main])
Sending: 4 (Thread[DefaultDispatcher-worker-3,5,main])
Receving: 1 (Thread[DefaultDispatcher-worker-2,5,main])
Sending: 9 (Thread[DefaultDispatcher-worker-2,5,main])
Receving: 4 (Thread[DefaultDispatcher-worker-3,5,main])
Sending: 16 (Thread[DefaultDispatcher-worker-1,5,main])
Receving: 9 (Thread[DefaultDispatcher-worker-1,5,main])
Sending: 25 (Thread[DefaultDispatcher-worker-5,5,main])
Receving: 16 (Thread[DefaultDispatcher-worker-2,5,main])
Receving: 25 (Thread[DefaultDispatcher-worker-2,5,main])
```

- 코루틴이 종료되면 produce() 빌더가 채널을 자동으로 닫아준다.

<br/>

### 3.3. 티커
---

**티커(ticker)** 채널은 Unit 값을 계속 발생시키되 한 원소와 다음 원소의 발생 시점이 주어진 지연 시간만큼 떨어져 있는 스트림을 만든다. ticker() 함수를 사용할 때 다음을 지정할 수 있다.

- **delayMillis** : 티커 원소의 발생 시간 간격을 밀리초 단위로 지정한다.
- **initialDelayMillis** : 티커 생성 시점과 원소가 최초로 발생하는 시점 사이의 시간 간격 (default: delayMillis)
- **context** : 티커를 실행할 코루틴 문맥 (default: 빈 문맥)
- **mode** : 티커의 행동을 결정
	- **TickerMode.FIXED_PERIOD** : 지연 시간에 최대한 맞추기 위해 실제 지연 시간을 조정
	- **TickerMode.FIXED_DEALY** : 지연 시간만큼 시간을 지연시킨 후 다음 원소를 송신

```kotlin
fun main() {
    runBlocking {
        val ticker = ticker(100)
        println(withTimeoutOrNull(50) { ticker.receive() })
        println(withTimeoutOrNull(60) { ticker.receive() })
        delay(250)
        println(withTimeoutOrNull(1) { ticker.receive() })
        println(withTimeoutOrNull(60) { ticker.receive() })
        println(withTimeoutOrNull(60) { ticker.receive() })
    }
}
```

<br/>

### 3.4. 액터
---

가변 상태를 스레드 안전하게 공유하는 액터(actor)가 있다. 액터는 내부 상태와 다른 액터에게 메시지를 보내서 동시성 통신을 진행할 수 있는 수단을 제공하는 객체다. 액터는 자신에게 들어오는 메시지를 listen하고, 자신의 상태를 바꾸면서 메시지에 응답할 수 있으며, 다른 메시지를 보낼 수 있고, 새로운 액터를 시작할 수 있다. 액터 모델은 락 기반의 동기화와 관련한 여러 가지 문제로부터 자유로울 수 있다.

actor() 함수를 통해 액터를 만들 수 있다. 액터는 ActorScope를 만들며, 이 영역은 기본 코루틴 영역에 자신에게 들어오는 메시지에 접근할 수 있는 수신자 채널이 추가된 것이다.

다음 예제를 통해 은행 계좌 잔고를 유지하고 어떤 금액을 저축하거나 인출할 수 있는 액터를 살펴보자.

```kotlin
sealed class AccountMessage

class GetBalance(
    val amount: CompletableDeferred<Long>,
) : AccountMessage()

class Deposit(val amount: Long) : AccountMessage()

class Withdraw(
    val amount: Long,
    val isPermitted: CompletableDeferred<Boolean>,
) : AccountMessage()
```

- CompletableDeferred라는 타입의 프로퍼티를 사용해서 GetBalance 메시지를 보낸 코루틴에게 현재 잔고를 돌려준다.

<br/>

메시지가 들어오는 채널을 계속 풀링하면서 수신한 메시지의 종류에 따라 적절한 동작을 수행한다.

```kotlin
fun CoroutineScope.accountManager(
    initialBalance: Long,
) = actor<AccountMessage> {
    var balance = initialBalance

    for (message in channel) {
        when (message) {
            is GetBalance -> message.amount.complete(balance)

            is Deposit -> {
                balance += message.amount
                println("Deposited ${message.amount}")
            }

            is Withdraw -> {
                val canWithdraw = balance >= message.amount
                if (canWithdraw) {
                    balance -= message.amount
                    println("Withdrawn ${message.amount}")
                }
                message.isPermitted.complete(canWithdraw)
            }
        }
    }
}
```

- actor() 빌더는 produce()에 대응한다고 할 수 있다.
	- 액터는 데이터를 받기 위해 채널을 사용하고 생산자는 소비자에게 데이터를 보내기 위한 채널을 생성
	- 액터는 기본적으로 랑데부 채널을 사용 (용량을 변경하면 채널의 성격을 바꿀 수 있음)

CompletableDeferred에서 complete() 메서드를 사용하여 액터 클라이언트에게 요청 결과를 돌려준다.

<br/>

액터와 통신하는 코루틴을 만들어보자.

```kotlin
private suspend fun SendChannel<AccountMessage>.deposit(
    name: String,
    amount: Long,
) {
    send(Deposit(amount))
    println("$name: deposit $amount")
}

private suspend fun SendChannel<AccountMessage>.tryWithdraw(
    name: String,
    amount: Long,
) {
    val status = CompletableDeferred<Boolean>().let {
        send(Withdraw(amount, it))
        if (it.await()) "OK" else "DENIED"
    }
    println("$name: withdraw $amount ($status)")
}

private suspend fun SendChannel<AccountMessage>.printBalance(
    name: String,
) {
    val balance = CompletableDeferred<Long>().let {
        send(GetBalance(it))
        it.await()
    }
    println("$name: balance is $balance")
}

fun main() {
    runBlocking {
        val manager = accountManager(100)
        withContext(Dispatchers.Default) {
            launch {
                manager.deposit("Client #1", 50)
                manager.printBalance("Client #1")
            }

            launch {
                manager.tryWithdraw("Client #2", 100)
                manager.printBalance("Client #2")
            }
        }

        manager.tryWithdraw("Client #0", 1000)
        manager.printBalance("Client #0")
        manager.close()
    }
}
```

```
Client #1: deposit 50
Deposited 50
Withdrawn 100
Client #1: balance is 150
Client #2: withdraw 100 (OK)
Client #2: balance is 50
Client #0: withdraw 1000 (DENIED)
Client #0: balance is 50
```

- 액터가 사용하는 채널에 대해 send() 메서드를 호출하여 액터에게 메시지를 보냄
- 공개적으로 접근 가능한 가변 상태가 없기 때문에 락이나 임계 영역 같은 동기화 요소를 사용하지 않아도 된다.

<br/>

## 4. 자바 동시성 사용하기
---

코틀린 표준 라이브러리에서 제공하는 스레드 생성이나 동기화 등과 관련한 함수들에 대해 알아보자.

<br/>

### 4.1. 스레드 시작하기
---

범용 스레드를 시작하려면, 스레드에서 실행하려는 실행 가능(Runnable) 객체에 대응하는 람다와 스레드 프로퍼티들을 지정해서 **thread()** 함수를 사용하면 된다.

- **start** : 스레드를 생성하자마자 시작할지 여부 (default: true)
- **isDaemon** : 스레드를 데몬 모드로 시작할지 여부 (default: false)
- **contextClassLoader** : 스레드 코드가 클래스와 자원을 적재할 때 사용할 클래스 로더 (default: null)
- **name** : 스레드 이름 (default: null)
- **priority** : Thread.MIN_PRIORITY(=1) 부터 Thread.MAX_PRIORITY(=10) 사이의 값으로 구성된 우선순위 (default: -1, 자동으로 정해짐)
- **block** : () → Unit 타입의 함숫값. 스레드가 실행할 코드

```kotlin
import kotlin.concurrent.thread

fun main() {
    println("Starting a thread...")

    thread(name = "Worker", isDaemon = true) {
        for (i in 1..5) {
            println("${Thread.currentThread().name}: $i")
            Thread.sleep(150)
        }
    }

    Thread.sleep(500)
    println("Shutting down...")
}
```

```
Starting a thread...
Worker: 1
Worker: 2
Worker: 3
Worker: 4
Shutting down...
```

<br/>

지정한 시간 간격으로 동작을 수행하는 자바 타이머 관련 함수가 있다. `timer()` 함수는 고정된 시간 간격으로 실행하는 타이머를 설정한다. timer() 호출로 설정할 때 다음 옵션을 지정할 수 있다.
- **name** : 타이머 스레드의 이름 (default: null)
- **daemon** : 타이머 스레드를 데몬 스레드로 할지 여부 (default: false)
- **startAt** : 최초로 타이머 이벤트가 발생하는 시간
- **period** : 이벤트 사이의 시간 간격
- **action** : 타이머 이벤트가 발생할 때마다 실행될 `TimeTask.() → Unit` 타입의 람다

```kotlin
import kotlin.concurrent.timer

fun main() {
    println("Starting a thread...")
    var counter = 0

    timer(period = 150, name = "Worker", daemon = true) {
        println("${Thread.currentThread().name}: ${++counter}")
    }

    Thread.sleep(500)
    println("Shutting down...")
}
```

```
Starting a thread...
Worker: 1
Worker: 2
Worker: 3
Worker: 4
Shutting down...
```

또한, 두 타이머 이벤트 사이의 시간 간격을 최대한 일정하게 맞춰주는 fixedRateTimer() 함수들도 있다.

<br/>

### 4.2. 동기화와 락
---

동기화는 코드가 한 스레드에서만 실행되도록 보장하기 위한 요소이다. 진입하려고 시도하는 다른 스레드들은 모두 대기해야 한다. 자바에서는 코드에 동기화를 도입하는 두 가지 방법이 있다.

첫째, 락으로 사용하는 어떤 객체를 지정하는 특별한 동기화 블록을 사용해 동기화해야 하는 코드를 감쌀 수 있다.

```kotlin
fun main() {
    var counter = 0
    val lock = Any()

    for (i in 1..5) {
        thread(isDaemon = false) {
            synchronized(lock) {
                counter += i
                println(counter)
            }
        }
    }
}
```

```
1
6
10
13
15
```

자바에서 동기화에 사용하는 다른 방법은 메서드에 synchronized 변경자를 붙이는 것이다. 코틀린에서는 `@Synchronized` 애너테이션을 통해 동기화할 수 있다.

```kotlin
class Counter {
    private var value = 0

    @Synchronized
    fun addAndPrint(value: Int) {
        this.value = value
        println(value)
    }
}

fun main() {
    val counter = Counter()
    for (i in 1..5) {
        thread(isDaemon = false) { counter.addAndPrint(i) }
    }
}
```

```
1
2
3
4
5
```

<br/>

Lock 객체(java.util.concuirrent.locks)를 사용해 주어진 람다를 실행하게 해주는 `withLock()` 함수도 있다. withLock()을 사용하면 함수가 알아서 락을 풀어주므로, 예외가 발생할 때 락을 푸는 것을 신경 쓰지 않아도 된다.

```kotlin
class Counter2 {
    private var value = 0
    private val lock = ReentrantLock()

    fun addAndPrint(value: Int) {
        lock.withLock {
            this.value += value
            println(value)
        }
    }
}

fun main() {
    val counter = Counter2()
    for (i in 1..5) {
        thread(isDaemon = false) { counter.addAndPrint(i) }
    }
}
```

```
1
3
4
5
2
```

<br/>

## Reference
---

[Yes24: 코틀린 완벽 가이드](https://m.yes24.com/Goods/Detail/107698728)