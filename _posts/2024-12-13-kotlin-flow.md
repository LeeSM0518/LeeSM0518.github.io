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

![kotlin-flow1](/assets/img/kotlin-flow1.png)

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
            simple().c
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

#### 1.9.1. Conflation
---

Flow에서 각 값을 처리할 필요가 없고 가장 최근의 값만 처리하면 될 수도 있다. 이러한 경우에 `conflate` 연산자는 수집기가 처리하기에 너무 느릴 때 중간 값을 건너뛰는 데 사용할 수 있다.

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
            .conflate()  
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
Thread[#1,main,5,main] : collect 3
collected in 748 ms
```
- 첫 번째 숫자가 아직 처리되고 있는 동안 두 번째와 세 번째 숫자가 이미 생성되어 두 번째 숫자가 합쳐지고 가장 최근의 숫자(세 번째 숫자)만 수집자에게 전달된다.

<br/>

#### 1.9.2. Processing the latest value
---

`conflate` 는 emitter와 collector가 모두 느릴 때 처리 속도를 높이는 한 가지 방법이다. 이는 emitted value를 삭제함으로써 가능하다. 다른 방법은 느린 collector를 취소하고 새로운 값이 emitted 될 때 마다 다시 시작하는 것이다. `xxxLatest` 연산자들이 존재하며, 이 연산자들은 `xxx` 연산자와 동일한 로직이 수행되지만, 새 값이 입력되면 로직을 취소한다.

```kotlin
fun main() = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        println("Started simple flow")  
        for (i in 1..3) {  
            delay(100)  
            println("${Thread.currentThread()} : emit $i")  
            emit(i)  
        }  
    }  
  
    val time = measureTimeMillis {  
        simple()  
            .collectLatest { value ->  
                println("collect start $value")  
                delay(300)  
                println("collect end $value")  
            }  
    }    println("collected in $time ms")  
}
```

```
Started simple flow
Thread[#1,main,5,main] : emit 1
collect start 1
Thread[#1,main,5,main] : emit 2
collect start 2
Thread[#1,main,5,main] : emit 3
collect start 3
collect end 3
collected in 653 ms
```
- `collectLatest` 가 모든 값에 대해 실행되지만 마지막 값에 대해서만 완료되는 것을 볼 수 있다.

<br/>

### 1.10. Composing multiple flows
---

다중 Flow를 구성하는 방법은 다양하다.

<br/>

#### 1.10.1. Zip
---

`zip` 연산자를 통해 두 Flow의 값을 결합할 수 있다.

```kotlin
fun main() = runBlocking {  
    val nums = (1..3).asFlow()  
    val strs = flowOf("one", "two", "three")  
  
    nums.zip(strs) { a, b -> "$a -> $b" }  
        .collect { println(it) }  
}
```

```
1 -> one
2 -> two
3 -> three
```

<br/>

#### 1.10.2. Combine
---

Flow가 변수 또는 연산의 가장 최근 값을 나타내는 경우, 해당 Flow의 가장 최근 값에 따라 계산을 수행하고 upstream flow에서 값을 내보낼 때마다 해당 계산을 다시 계산해야 할 수도 있다. 이에 해당하는 연산자를 `combine` 이라 한다.

예를 들어, 이전 예제의 숫자가 300ms마다 업데이트되지만 문자열은 400ms마다 업데이트되는 경우 `zip` 연산자를 사용하여 압축하면 결과는 같지만 400ms마다 출력된다.

```kotlin
fun main() = runBlocking {  
    val nums = (1..3).asFlow().onEach { delay(300) }  
    val strs = flowOf("one", "two", "three").onEach { delay(400) }  
    val startTime = System.currentTimeMillis()  
  
    nums.zip(strs) { a, b -> "$a -> $b" }  
        .collect { value ->  
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")  
        }  
}
```

```
1 -> one at 429 ms from start
2 -> two at 830 ms from start
3 -> three at 1240 ms from start
```

<br/>

하지만 `zip` 대신 `combine` 연산자를 사용하는 경우, nums 또는 strs에서 각 emit 마다 출력되는 것을 확인 할 수 있다.
```kotlin
fun main() = runBlocking {  
    val nums = (1..3).asFlow().onEach { delay(300) }  
    val strs = flowOf("one", "two", "three").onEach { delay(400) }  
    val startTime = System.currentTimeMillis()  
  
    nums.combine(strs) { a, b -> "$a -> $b" }  
        .collect { value ->  
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")  
        }  
}
```

```
1 -> one at 430 ms from start
2 -> one at 638 ms from start
2 -> two at 841 ms from start
3 -> two at 944 ms from start
3 -> three at 1243 ms from start
```

<br/>

### 1.11. Flattening flows
---

Flow는 비동기적으로 수신된 일련의 값을 나타내므로 각 값이 다른 일련의 값에 대한 요청을 트리거하는 상황이 발생하기 쉽다. 예를 들어, 500ms 간격으로 두 개의 문자열 흐름을 반환하는 함수가 있다.

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {  
    emit("$i: First")  
    delay(500)  
    emit("$i: Second")  
}
```

다음과 같이 `requestFlow` 를 호출한다고 가정해 보자.

```kotlin
(1..3).asFlow().map { requestFlow(it) }
```
- 단일 flow로 평면화해야 하는 flow의 flow(`Flow<Flow<String>>`)가 생긴다.
- Flow의 비동기적 특성으로 인해 서로 다른 flattening 이 필요하다.

<br/>

#### 1.11.1 flatMapConcat
---

flow의 flow 연결은 `flatMapConcat` 및 `flattenConcat` 연산자를 통해 제공된다. 해당 연산자들은 다음 flow를 수집하기 전에 내부 flow가 완료되기를 기다린다.

```kotlin
fun main(): Unit = runBlocking {  
    fun requestFlow(i: Int): Flow<String> = flow {  
        emit("$i: First")  
        delay(500)  
        emit("$i: Second")  
    }  
  
    val startTime = System.currentTimeMillis()  
  
    (1..3)  
        .asFlow()  
        .onEach { delay(100) }  
        .flatMapConcat { requestFlow(it) }  
        .collect { value ->  
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")  
        }  
}
```

```
1: First at 120 ms from start
1: Second at 631 ms from start
2: First at 737 ms from start
2: Second at 1242 ms from start
3: First at 1349 ms from start
3: Second at 1854 ms from start
```

<br/>

#### 1.11.2. flatMapMerge
---

다른 flattening 작업은 모든 수신 flow를 동시에 수집하고 해당 값을 단일 flow로 병합하여 가능한 한 빨리 값을 내보낸다. 이는 `flatMapMerge` 및 `flattenMerge` 연산자에 의해 구현된다. 둘 다 동시에 수집되는 동시 flow 수를 제한하는 매개변수를 제공한다.

```kotlin
fun main(): Unit = runBlocking {  
    fun requestFlow(i: Int): Flow<String> = flow {  
        emit("$i: First")  
        delay(500)  
        emit("$i: Second")  
    }  
  
    val startTime = System.currentTimeMillis()  
  
    (1..3)  
        .asFlow()  
        .onEach { delay(100) }  
        .flatMapMerge { requestFlow(it) }  
        .collect { value ->  
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")  
        }  
}
```

```
1: First at 131 ms from start
2: First at 231 ms from start
3: First at 337 ms from start
1: Second at 634 ms from start
2: Second at 735 ms from start
3: Second at 840 ms from start
```

<br/>

#### 1.11.3. flatMapLatest
---

`collectLatest` 연산자와 유사한 방식으로, 이전 flow의 수집이 새 flow가 방출되자마자 취소되는 "Latest" flattening 방식이다.

```kotlin
fun main(): Unit = runBlocking {  
    fun requestFlow(i: Int): Flow<String> = flow {  
        emit("$i: First")  
        delay(500)  
        emit("$i: Second")  
    }  
  
    val startTime = System.currentTimeMillis()  
  
    (1..3)  
        .asFlow()  
        .onEach { delay(100) }  
        .flatMapLatest { requestFlow(it) }  
        .collect { value ->  
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")  
        }  
}
```

```
1: First at 133 ms from start
2: First at 240 ms from start
3: First at 346 ms from start
3: Second at 851 ms from start
```

<br/>

### 1.12. Flow exceptions
---

내부 연산이 예외를 throw 할 경우 flow 수집이 예외와 함께 완료될 수 있다. 이러한 예외를 처리하는 방법은 여러 가지가 있다.

<br/>

#### 1.12.1. Collector try and catch
---

`try/catch` 를 사용하여 예외를 처리할 수 있다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        for (i in 1..3) {  
            println("Emitting $i")  
            emit(i)  
        }  
    }  
  
    try {  
        simple().collect { value ->  
            println(value)  
            check(value <= 1) { "Collected $value" }  
        }    
	} catch (e: Throwable) {  
        println("Caught $e")  
    }  
}
```

```
Emitting 1
1
Emitting 2
2
Caught java.lang.IllegalStateException: Collected 2
```
- 예외를 잘 처리하는 것을 확인할 수 있다.

<br/>

#### 1.12.2. Everything is caught
---

이전 예제는 emitter 나 중간 또는 터미널 연산자에서 발생하는 모든 예외를 처리한다. 또한, 다음과 같이 map 연산에서 발생한 예외도 잘 처리되는 것을 볼 수 있다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<String> =  
        flow {  
            for (i in 1..3) {  
                println("Emitting $i")  
                emit(i)  
            }  
        }.map { value ->  
            check(value <= 1) { "Crashed on $value" }  
            "string $value"  
        }  
  
    try {  
        simple().collect { value -> println(value) }  
    } catch (e: Throwable) {  
        println("Caught $e")  
    }  
}
```

```
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2
```

<br/>

### 1.13. Exception transparency
---

emitter의 코드는 어떻게 예외 처리 동작을 캡슐화할 수 있을까?

`try/catch` 내부의 `flow { ... }` 빌더에서 값을 내보내는 것은 예외 투명성을 위반하는 것이다. 이렇게 하면 예외를 발생시키는 수집기가 항상 이전 예와 같이 `try/catch` 를 사용하여 예외를 잡을 수 있다.

emitter는 예외 투명성을 보존하고 예외 처리를 캡슐화하는 것을 허용하는 `catch` 연산자를 사용할 수 있다. `catch` 연산자를 통해 예외를 분석하고 예외에 따라 다양한 방식으로 대응할 수 있다.

- 예외를 다시 throw 할 수 있다.
- `catch` 에서 `emit` 을 사용하여 예외를 값 방출로 전환할 수 있다.
- 예외를 무시하거나, 기록하거나 다른 코드에 처리를 위임할 수 있다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<String> =  
        flow {  
            for (i in 1..3) {  
                println("Emitting $i")  
                emit(i)  
            }  
        }.map { value ->  
            check(value <= 1) { "Crashed on $value" }  
            "string $value"  
        }  
  
    simple()  
        .catch { e -> emit("Caught $e") }  
        .collect { value -> println(value) }  
}
```

```
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2
```
- `try/catch` 를 사용하지 않더라도 출력이 동일한 것을 볼 수 있다.

<br/>

#### 1.13.1. Transparent catch
---

`catch` 연산자는 upstream 예외만 `catch` 한다. 만약 `catch` 하위에 있는 `collect { ... }`  블록이 예외를 `throw` 하면 해당 블록은 종료된다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<Int> =  
        flow {  
            for (i in 1..3) {  
                println("Emitting $i")  
                emit(i)  
            }  
        }  
  
    simple()  
        .catch { e -> println("Caught $e") }  
        .collect { value ->  
            check(value <= 1) { "Crashed on $value" }  
            println(value)  
        }  
}
```

```
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Crashed on 2
	...
```

<br/>

#### 1.13.2. Catching declaratively
---

`catch` 연산자로 모든 예외를 처리하려면 `collect` 연산자의 본문을 `onEach` 로 옮기고 `catch` 연산자 앞에 두어야 한다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<Int> =  
        flow {  
            for (i in 1..3) {  
                println("Emitting $i")  
                emit(i)  
            }  
        }  
  
    simple()  
        .onEach { value ->  
            check(value <= 1) { "Crashed on $value" }  
            println(value)  
        }  
        .catch { e -> println("Caught $e") }  
        .collect {}  
}
```

```
Emitting 1
1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2
```
- 이처럼 모든 예외가 처리되는 것을 볼 수 있다.

<br/>

### 1.14. Flow completion
---

flow 수집이 완료되면 명령형 또는 선언형 방식으로 작업을 실행할 수 있다.

<br/>

#### 1.14.1. Imperative finally block
---

`finally` 를 사용하여 수집 완료 시 작업을 실행할 수 있다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<Int> =  
        flow {  
            for (i in 1..3) {  
                println("Emitting $i")  
                emit(i)  
            }  
        }  
  
    try {  
        simple()  
            .collect { value ->  
                check(value <= 1) { "Crashed on $value" }  
                println(value)  
            }  
    } catch (e: Exception) {  
        println("Done")  
    }  
}
```

```
Emitting 1
1
Emitting 2
Done
```

<br/>

#### 1.14.2. Declarative handling
---

선언적 방식의 경우 `onCompletion` 연산자를 통해 완료된 flow를 처리할 수 있다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<Int> =  
        flow {  
            for (i in 1..3) {  
                println("Emitting $i")  
                emit(i)  
            }  
        }  
  
    simple()  
        .onCompletion { println("Done") }  
        .collect { value -> println(value) }  
}
```

```
Emitting 1
1
Emitting 2
2
Emitting 3
3
Done
```

<br/>

`onCompletion` 사용시 람다의 nullable `Throwable` 매개변수를 통해 작업이 정상 완료되었는지 확인할 수 있다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<Int> =  
        flow {  
            emit(1)  
            throw RuntimeException()  
        }  
  
    simple()  
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") }  
        .catch { cause -> println("Caught exception : $cause") }  
        .collect { value -> println(value) }  
}
```

```
1
Flow completed exceptionally
Caught exception : java.lang.RuntimeException
```
- `onCompletion` 이 실행되어도 downstream으로 전달되는 것을 볼 수 있다.

<br/>

#### 1.14.3. Successful completion
---

`onCompletion` 은 성공적으로 완료되어야 `null` 예외를 수신한다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<Int> = (1..3).asFlow()  
  
    simple()  
        .onCompletion { cause -> println("Flow completed with $cause") }  
        .catch { cause -> println("Caught exception : $cause") }  
        .collect { value ->  
            check(value <= 1) { "Collected $value" }  
            println(value)  
        }  
}
```

```
1
Flow completed with java.lang.IllegalStateException: Collected 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
	...
```
- 다운스트림 예외로 인해 흐름이 중단되어 완료 원인이 `null` 이 아닌 것을 확인할 수 있다.

<br/>

### 1.15. Imperative vs declarative
---

사용자의 선호도와 코드 스타일에 따라 명령형과 선언형 방식을 선택해야 한다.

<br/>

### 1.16. Launching flow
---

비동기 이벤트 발행하는 예시를 살펴보자. `onEach` 뒤에 `collect` 연산자를 사용하면 그 다음 차례의 flow는 collect 될 때까지 기다려야 한다.

```kotlin
fun main(): Unit = runBlocking {  
    fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }  
  
    val start = System.currentTimeMillis()  
    events()  
        .onEach { event ->  
            println("Event: $event at ${System.currentTimeMillis() - start}ms")  
        }  
        .collect {}  
    println("Done")  
}
```

```
Event: 1 at 109ms
Event: 2 at 217ms
Event: 3 at 320ms
Done
```

<br/>

`launchIn` 연산자를 사용하면 다음 차례의 flow를 즉시 실행하게 된다.

```kotlin
fun main(): Unit = runBlocking {  
    fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }  
  
    val start = System.currentTimeMillis()  
    events()  
        .onEach { event ->  
            println("Event: $event at ${System.currentTimeMillis() - start}ms")  
        }  
        .launchIn(this)  
    println("Done")  
}
```

```
Done
Event: 1 at 118ms
Event: 2 at 231ms
Event: 3 at 332ms
```
- `launchIn` 은 코루틴이 시작되는 `CoroutineScope` 을 지정해야 한다.
- `launchIn` 은 `Job` 을 반환하므로 이를 통해 코루틴을 취소하는데 사용할 수 있다.

<br/>

#### 1.16.1. Flow cancellation checks
---

flow 빌더는 각 emitted value에 대해 취소를 위한 `ensureActive` 검사를 수행한다. 즉, `flow { ... }` 에서 emit 하는 busy loop는 취소가 가능하다는 의미이다.

```kotlin
fun main(): Unit = runBlocking {  
    fun simple(): Flow<Int> = flow {  
        for (i in 1..10000000) {  
            println("Emitting $i")  
            emit(i)  
        }  
    }  
  
    simple()  
        .collect { value ->  
            if (value == 3) cancel()  
            println(value)  
        }  
    println("Done")  
}
```

```
Emitting 1
1
Emitting 2
2
Emitting 3
3
Emitting 4
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job=BlockingCoroutine{Cancelled}@21bcffb5
```

<br/>

그러나 다른 flow 연산자는 성능상의 이유로 추가적인 취소 확인을 직접 수행하지 않는다. `asFlow` 를 사용하면 취소되지 않는다.

```kotlin
fun main(): Unit = runBlocking {  
    (1..5)  
        .asFlow()  
        .collect { value ->  
            if (value == 3) cancel()  
            println(value)  
        }  
    println("Done")  
}
```

```
1
2
3
4
5
Done
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job=BlockingCoroutine{Cancelled}@7a79be86
```
- `runBlocking` 이 완료되기 전에만 취소가 감지된다.

<br/>

취소를 감지하려면 `cancellable` 연산자를 사용해야 한다.

```kotlin
fun main(): Unit = runBlocking {  
    (1..5)  
        .asFlow()  
        .cancellable()  
        .collect { value ->  
            if (value == 3) cancel()  
            println(value)  
        }  
    println("Done")  
}
```

```
1
2
3
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job=BlockingCoroutine{Cancelled}@2d6e8792
```

<br/>

## 2. Flow API Docs
---

![kotlin-flow2](/assets/img/kotlin-flow2.png)

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

[Kotlin: Asynchronous Flow](https://kotlinlang.org/docs/flow.html#sequences)

[Kotlin: Flow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/)