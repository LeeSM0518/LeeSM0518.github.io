---
title: Kotlin Flow
date: 2024-12-13 09:00:00 +0900
categories: kotlin
tags:
  - kotlin
description: Flow 개념과 사용 방법에 대해 정리
---

## 1. Flow Guide
---

Flow와 suspend 함수를 통해 비동기적으로 계산된 여러 값을 반환할 수 있다.

<br/>

### 1.1. Representing multiple values
---

#### 1.1.1. Sequences
---

CPU를 많이 소모하는 차단 코드(각 계산에 100ms 소요)로 숫자를 계산하는 경우 시퀀스를 사용하여 숫자를 표현할 수 있다.

```kotlin
fun main() {  
    fun simple(): Sequence<Int> = sequence {  
        for (i in 1..3) {  
            Thread.sleep(100)  
            yield(i)  
        }  
    }  
  
    val duration = measureTimeMillis {  
        simple().forEach { value -> println(value) }  
    }    println("duration : $duration ms")  
}
```

```
1
2
3
duration : 315 ms
```

- 각 숫자를 출력하기 전에 100ms를 기다려야 한다.

<br/>

#### 1.1.2. Suspending functions
---

위에서 살펴본 계산 코드는 메인 스레드를 차단한다. 함수를 suspend로 변경하여 차단 없이 작업을 수행하고 결과를 목록으로 반환할 수 있도록 한다.

```kotlin
fun main() = runBlocking {  
    suspend fun simple(): List<Int> {  
        delay(100)  
        return listOf(1, 2, 3)  
    }  
  
    val duration = measureTimeMillis {  
        simple().forEach { value -> println(value) }  
    }    println("duration : $duration ms")  
}
```

```
1
2
3
duration : 111 ms
```

<br/>

#### 1.1.3. Flows
---

`List<Int>` 타입은 한 번에 모든 값을 반환한다. 비동기적으로 계산되는 값의 스트림을 나타내려면 다음과 같이 `Flow<Int>` 타입을 사용하면 된다.

```kotlin
fun main() = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        for (i in 1..3) {  
            delay(100)  
            emit(i)  
        }  
    }  
  
    launch {  
        for (k in 1..3) {  
            println("I'm not blocked $k")  
            delay(100)  
        }  
    }  
    val duration = measureTimeMillis {  
        simple().collect { value -> println(value) }  
    }    println("duration : $duration ms")  
}
```

```
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
duration : 325 ms
```
- `flow`  : Flow 타입의 빌더 함수
- `flow { ... }` 빌더 블록 내부의 코드는 일시 중단될 수 있다.
- 값은 `emit` 함수를 통해 flow에서 방출된다.
- `collect` 함수를 사용하여 flow에서 값을 수집한다.

<br/>

### 1.2. Flows are cold
---

Flow는 Sequence와 유사하게 cold stream이다. flow 빌더 내부의 코드는 collect 될 때까지 실행되지 않는다.

```kotlin
fun main() = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        println("Flow started")  
        for (i in 1..3) {  
            delay(100)  
            emit(i)  
        }  
    }  
  
    val duration = measureTimeMillis {  
        println("Calling simple function...")  
        val flow = simple()  
        println("Calling collect...")  
        flow.collect { value -> println(value) }  
        println("Calling collect again...")  
        flow.collect { value -> println(value) }  
    }    println("duration : $duration ms")  
}
```

```
Calling simple function...
Calling collect...
Flow started
1
2
3
Calling collect again...
Flow started
1
2
3
duration : 635 ms
```

<br/>

### 1.3. Flow cancellation basics
---

Flow는 코루틴의 취소 방식에 준수한다. 일시 중단 기능(ex. delay)에서 flow가 일시 중단되면 수집이 취소될 수 있다. 다음 예제를 통해 `withTimeoutOrNull` 블록에서 시간 초과로 인해 flow가 취소되고 실행이 중지되는 것을 볼 수 있다.

```kotlin
fun main() = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        for (i in 1..3) {  
            delay(100)  
            println("Emitting $i")  
            emit(i)  
        }  
    }  
  
    val duration = measureTimeMillis {  
        withTimeoutOrNull(250) {  
            simple().collect { value -> println(value) }  
        }    }    println("duration : $duration ms")  
}
```

```
Emitting 1
1
Emitting 2
2
duration : 254 ms
```

<br/>

### 1.4. Flow builders
---

앞에서 살펴본 `flow { ... }` 빌더는 기본적인 방법 중에 하나이다. 다음과 같이 여러 빌더들이 존재한다.

- `flowOf` : 값들의 고정된 집합을 방출
- `.asFlow()` : 다양한 컬렉션과 시퀀스를 Flow로 변환

```kotlin
(1..3).asFlow().collect { value -> println(value) }
```

<br/>

### 1.5. Intermediate flow operatiors
---

Flow는 컬렉션과 시퀀스를 변환하는 것과 같은 방식으로 연산자를 사용하여 변환할 수 있다. 중간 연산자는 upstream flow에 적용되고 downstream flow를 반환한다. 이러한 연산자는 Flow와 마찬가지로 cold 하게 동작하며, 새로운 변화된 flow를 빠르게 반환한다.

`map` 과 `filter` 와 같은 기본적인 연산이 제공되며, 연산자 내부의 코드 블록이 중단 함수를 호출할 수 있다.

```kotlin
fun main() = runBlocking {  
    suspend fun simple(value: Int): String {  
        delay(1000)  
        return "result $value"  
    }  
  
    val duration = measureTimeMillis {  
        (1..3)  
            .asFlow()  
            .map { value -> simple(value) }  
            .collect(::println)  
    }  
    println("duration : $duration ms")  
}
```

```
result 1
result 2
result 3
duration : 3036 ms
```

<br/>

#### 1.5.1. Transform operator
---

flow 변환 연산자 중 일반적인 연산자는 `transform` 이다. 이는 복잡한 변환을 수행하는데 사용할 수 있으며, 임의의 값을 임의의 횟수만큼 출력할 수 있다. 비동기 요청을 수행하기 전에 문자열을 내보내고 그 뒤에 응답을 보낼 수 있다.

```kotlin
fun main() = runBlocking {  
    suspend fun simple(value: Int): String {  
        delay(1000)  
        return "result $value"  
    }  
  
    val duration = measureTimeMillis {  
        (1..3)  
            .asFlow()  
            .transform { request ->  
                emit("Making request $request")  
                emit(simple(request))  
            }  
            .collect(::println)  
    }  
    println("duration : $duration ms")  
}
```

```
Making request 1
result 1
Making request 2
result 2
Making request 3
result 3
duration : 3035 ms
```

<br/>

#### 1.5.2. Size-limiting operators
---

`take` 와 같은 크기를 제안하는 중간 연산자는 해당 제한에 도달하면 flow를 취소한다. 코루틴에서의 취소는 항상 예외를 발생시켜 수행되므로 모든 리소스 관리 함수(`try { ... } finally { ... }`)는 취소가 발생한 경우에도 정상적으로 작동한다.

```kotlin
fun main() = runBlocking {  
    fun numbers(): Flow<Int> = flow {  
        try {  
            emit(1)  
            emit(2)  
            println("This line will not execute")  
            emit(3)  
        } finally {  
            println("Finally in numbers")  
        }  
    }  
  
    val duration = measureTimeMillis {  
        numbers()  
            .take(2)  
            .collect { value -> println(value) }  
    }    println("duration : $duration ms")  
}
```

```
1
2
Finally in numbers
duration : 13 ms
```

<br/>

### 1.6. Terminal flow operators

터미널 연산자는 flow 수집을 시작하는 기능을 일시 중단한다.
`collect` 연산자는 가장 기본적인 연산자이지만, 다음과 같이 이를 더 쉽게 만들어 줄 수 있는 다른 터미널 연산자도 있다. 

- `toList` , `toSet` : 다양한 컬렉션으로의 변환
- `first`, `single` : 첫 번째 값을 가져오고 흐름이 단일 값을 방출
- `reduce` , `fold` : flow의 값을 합친다

```kotlin
fun main() = runBlocking {  
    val duration = measureTimeMillis {  
        (1..5)  
            .asFlow()  
            .map { it * it }  
            .reduce { a, b -> a + b }  
            .also(::println)  
    }  
    println("duration : $duration ms")  
}
```

```
55
duration : 12 ms
```

<br/>

### 1.7. Flows are sequential
---

여러 flow에서 작동하는 특수 연산자를 사용하지 않은 한, flow의 각 개별 컬렉션은 순차적으로 수행된다. 컬렉션은 터미널 연산자를 호출하는 코루틴에서 직접 작동한다. 방출된 각 값은 업스트림에서 다운스트림까지 모두 중간 연산자에 의해 처리된 후 터미널 연산자에게 전달된다.

```kotlin
fun main() = runBlocking {  
    val duration = measureTimeMillis {  
        (1..5)  
            .asFlow()  
            .filter {  
                println("Filter $it")  
                it % 2 == 0  
            }  
            .map {  
                println("Map $it")  
                "string $it"  
            }.collect {  
                println("Collect $it")  
            }  
    }    println("duration : $duration ms")  
}
```

```
Filter 1
Filter 2
Map 2
Collect string 2
Filter 3
Filter 4
Map 4
Collect string 4
Filter 5
duration : 14 ms
```

<br/>

### 1.8. Flow Context
---

Flow 수집은 코루틴의 컨텍스트에서 호출할 수 있다. Flow의 구현 세부 사항과 관계없이 지정된 컨텍스트에서 실행된다. 이런 Flow의 속성을 컨텍스트 보존이라고 한다.

```kotlin
withContext(context) {
	simple().collect { value -> 
		println(value) // 특정 컨텍스트에서 실행
	}
}
```

<br/>

따라서, `flow { ... }` 빌더의 코드는 해당 Flow의 수집기에서 제공하는 컨텍스트에서 실행된다. 

```kotlin
fun simple(): Flow<Int> = flow {  
    println("${Thread.currentThread()} : Started simple flow")  
    for (i in 1..3) {  
        emit(i)  
    }  
}  
  
fun main() = runBlocking {  
    simple().collect { value -> println("${Thread.currentThread()} : $value") }  
}
```

```
Thread[#1,main,5,main] : Started simple flow
Thread[#1,main,5,main] : 1
Thread[#1,main,5,main] : 2
Thread[#1,main,5,main] : 3
```
- `simple().collect` 가 메인 스레드에서 호출되므로 `simple` 의 Flow도 메인 스레드에서 호출된다.

<br/>

#### 1.8.1. A common pitfall when using withContext

그러나 장시간 실행되는 CPU 사용 코드는 `Dispatchers.Default` 컨텍스트에서 실행되어야 할 수도 있고, UI 업데이트 코드는 `Dispatchers.Main` 컨텍스트에서 실행되어야 할 수도 있다. 일반적으로 `withContext` 는 코루틴을 사용하여 코드에서 컨텍스트를 변경하는 데 사용되지만 `flow { ... }` 빌더의 코드는 컨텍스트 보존 속성으로 인해 다른 컨텍스트에서 방출할 수 없다.

```kotlin
fun main() = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        // WARNING : 컨텍스트를 변경  
        withContext(Dispatchers.Default) {  
            println("${Thread.currentThread()} : Started simple flow")  
            for (i in 1..3) {  
                Thread.sleep(100) // CPU를 점유하는 작업  
                emit(i)  
            }  
        }  
    }    simple().collect { value -> println("${Thread.currentThread()} : $value") }  
}
```

```
Thread[#21,DefaultDispatcher-worker-1,5,main] : Started simple flow
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Flow was collected in [BlockingCoroutine{Active}@43068a32, BlockingEventLoop@44836ea3],
		but emission happened in [DispatchedCoroutine{Active}@6d919ba7, Dispatchers.Default].
		Please refer to 'flow' documentation or use 'flowOn' instead
	at ...
```

<br/>

#### 1.8.2. flowOn operator
---

앞서 살펴본 예외를 보면 컨텍스트를 변경할 경우 `flowOn` 을 사용할 것을 권고하고 있다. 

```kotlin
fun main() = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        println("${Thread.currentThread()} : Started simple flow")  
        for (i in 1..3) {  
            Thread.sleep(100) // CPU를 점유하는 작업  
            println("${Thread.currentThread()} : Progress simple flow")  
            emit(i)  
        }  
    }.flowOn(Dispatchers.Default)  
    simple().collect { value -> println("${Thread.currentThread()} : $value") }  
}
```

```
Thread[#21,DefaultDispatcher-worker-1,5,main] : Started simple flow
Thread[#21,DefaultDispatcher-worker-1,5,main] : Progress simple flow
Thread[#1,main,5,main] : 1
Thread[#21,DefaultDispatcher-worker-1,5,main] : Progress simple flow
Thread[#1,main,5,main] : 2
Thread[#21,DefaultDispatcher-worker-1,5,main] : Progress simple flow
Thread[#1,main,5,main] : 3
```
- `flow { ... }` 가 백그라운드 스레드에서 처리되는 반면, `collect` 는 메인 스레드에서 처리되는 것을 볼 수 있다.

<br/>

### 1.9. Buffering
---

Flow의 여러 부분을 서로 다른 코루틴에서 실행하는 것은 collect 하는데 걸리는 시간을 줄일 수 있다. 먼저 개선하기 전의 예시를 살펴보자.

```kotlin
fun main() = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        println("${Thread.currentThread()} : Started simple flow")  
        for (i in 1..3) {  
            delay(100)  
            emit(i)  
        }  
    }  
  
    val time = measureTimeMillis {  
        simple().collect { value ->  
            delay(300)  
            println("${Thread.currentThread()} : collect $value")  
        }  
    }    println("collected in $time ms")  
}
```

```
Thread[#1,main,5,main] : Started simple flow
Thread[#1,main,5,main] : collect 1
Thread[#1,main,5,main] : collect 2
Thread[#1,main,5,main] : collect 3
collected in 1240 ms
```

<br/>

collect 하는 것과 emit 하는 것을 동시에 실행하기 위해 `buffer` 연산자를 사용할 수 있다. 이를 활용해 개선한 예시를 살펴보자.

```kotlin
fun main() = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        println("${Thread.currentThread()} : Started simple flow")  
        for (i in 1..3) {  
            delay(100)  
            emit(i)  
        }  
    }  
  
    val time = measureTimeMillis {  
        simple()  
            .buffer()  
            .collect { value ->  
                delay(300)  
                println("${Thread.currentThread()} : collect $value")  
            }  
    }    println("collected in $time ms")  
}
```

```
Thread[#1,main,5,main] : Started simple flow
Thread[#1,main,5,main] : collect 1
Thread[#1,main,5,main] : collect 2
Thread[#1,main,5,main] : collect 3
collected in 1047 ms
```
- 첫 번째 숫자에 대한 100ms만 기다리고 숫자를 처리하는 데 300ms만 사용하면 된다.
- 이런 식으로 실행하는 데 약 1,000ms가 걸린다.

<br/>

## Flow API Docs
---

Flow란 값을 순차적으로 내보내고 정상적으로 완료되거나 예외가 발생하는 비동기 데이터 스트림이다. Flow의 인터페이스는 다음과 같다.

```kotlin
interface Flow<out T>
```

<br/>

`map` , `filter` , `take` , `zip` 등의 flow에 있는 중간 연산자는 upstream flow 또는 flow에 적용되는 함수이며, 추가 연산자를 적용할 수 있는 downstream flow를 반환한다. 
**중간 작업은 flow에서 어떤 코드도 실행하지 않으며 함수 자체를 중단하지 않는다.** 이는 향후 실행을 위한 **작업 체인을 구성하고 빠르게 반환한다.** 이를 **cold flow** 속성이라고 한다.

flow의 터미널 연산자는 `collect` , `single` , `reduce` , `toList` 등과 같은 일시 중단 함수이거나 지정된 scope에서 flow 수집을 `launchIn` 연산자이다. 이는 upstream flow에 적용되며 모든 작업의 실행을 트리거한다. flow의 실행은 flow 수집이라고도 하며 항상 실제 차단이 아닌 일시 중단 방식으로 수행된다.

<br/>

터미널 연산자는 upstream의 모든 flow 작업의 성공 또는 실패에 따라 정상적으로 또는 예외적으로 완료한다. 터미널 연산자의 기본 예시는 다음과 같다.

```kotlin
try {  
    flow.collect { value ->  
        println("Received $value")  
    }  
} catch (e: Exception) {  
    println("The flow has thrown an exception: $e")  
}
```

<br/>

기본적으로 flow는 순차적이며 모든 flow 작업은 동일한 코루틴에서 순차적으로 실행된다. 단, `buffer` 및 `flatMapMerge` 와 같이 flow 실행에 동시성을 도입하기 위해 특별히 설계된 몇 가지 작업은 예외이다. 일반적으로 flow는 cold stream 이지만 hot stream인 `SharedFlow` 도 있다. 이 외에도, `stateIn` 및 `shareIn` 연산자를 통해 모든 flow를 hot stream으로 전환할 수 있으며, `producerIn` 연산자를 통해 flow를 hot channel로 변환할 수도 있다.

<br/>

## Reference
---

[Kotlin: Flow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/)