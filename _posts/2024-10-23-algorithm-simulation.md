---
title: Simulation
date: 2024-10-23 09:00:00 +0900
categories: algorithm
tags:
  - algorithm
description: 시뮬레이션 개념과 문제 풀이에 대해 살펴본다.
---

## 1. 개념
---

시뮬레이션 문제 풀이에 염두할 두 가지는 다음과 같다.

1. 하나의 문제를 최대한 여러 개로 분리한다.
2. 예외 처리가 필요하다면 독립 함수로 구현한다.

<br/>

### 1.1. 행렬 연산
---

각 행렬에서 같은 위치에 있는 값끼리 더하거나 빼는 연산이다.

<br/>

### 1.2. 전치 행렬
---

`arr[i][j]` 를 `arr[j][i]` 로 바꾸는 연산이다.

<br/>

### 1.3. 좌표 연산
---

좌푯값에 해당하는 배열의 위치를 활용한다. (3,4) 위치를 arr[4][3] = 1 과 같이 저장한다.

<br/>

### 1.4. 좌표 이동을 오프셋 값으로 쉽게 표현하기
---

좌표 이동을 다음과 같이 오프셋 값을 이용해 for문으로 이동이 가능하다.

dy[] = (-1, -1, -1, 0, 0, 1, 1, 1, 0), dx[] = (-1, 0, 1, -1, 1, -1, 0, 1, 0)

<br/>

### 1.5. 대칭, 회전 연산
---

- 대칭 : `A[i, j] = A[i, (N-1)-j]`
- 회전 : `A[i, j] = A[(N-1)-j, i]`

<br/>

## 2. (문제) 이진 변환 반복하기
---

### 2.1. 설명
---

0과 1로 이루어진 어떤 문자열 x에 대한 이진 변환을 다음과 같이 정의합니다.

1. x의 모든 0을 제거한다.
2. x의 길이를 c라고 하면 x를 'c를 2진법으로 표현한 문자열'로 바꾼다.

예를 들어 x = "0111010" 이면 이진 변환 과정은 "0111010" -> "1111" -> "100" 입니다. 0과 1로 이루어진 문자열 s가 주어지고 s가 "1"이 될 때까지 계속해서 이진 변환을 할 때 이진 변환의 횟수와 변환 과정에서 제거된 모든 0의 개수를 배열에 담아 반환하는 solution() 함수를 완성해주세요.

<br/>

### 2.2. 제약 조건
---

- s의 길이는 1 이상 150,000 이하입니다.
- s에는 '1'이 하나 이상 포함되어 있습니다.

<br/>

### 2.3. 입출력의 예
---

|s|result|
|---|---|
|`"110010101001"`|`[3,8]`|
|`"01110"`|`[3,3]`|
|`"1111111"`|`[4,1]`|

<br/>

### 2.4. 코드
---

```kotlin
fun solution(s: String): IntArray {
    var string = s
    var numberOfConversions = 0
    var numberOfRemoval = 0

    // 1. string이 1이 될 때까지 무한 반복
    while (string != "1") {
        //  1.1. 0의 개수를 센 후 numberOfRemoval에 더한다
        numberOfRemoval += string.count { it == '0' }
        //  1.2. string에서 0을 제거한다
        string = string.replace("0", "")
        //  1.3. numberOfConversions을 1 올린다
        numberOfConversions++
        //  1.4. string의 크기를 2진수로 변환한 후 string에 저장한다
        string = string.length.toString(2)
    }

    return intArrayOf(numberOfConversions, numberOfRemoval)
}

val message = solution("110010101001").toList()
println(message)
```

<br/>

## Reference
---

[Programmers: 이진 변환 반복하기](https://school.programmers.co.kr/learn/courses/30/lessons/70129)