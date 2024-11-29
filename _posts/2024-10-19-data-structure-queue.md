---
title: Queue
date: 2024-10-19 09:00:00 +0900
categories: data-structure
tags:
  - data-structure
description: 큐 자료구조에 대한 개념과 문제 풀이
---

## 1. 개념
---

먼저 들어간 것이 먼저 나오는 선입선출(FIFO, First In Fast Out) 방식

<br/>

## 2. (문제) 기능 개발
---

### 2.1. 설명
---

프로그래머스팀에서는 기능 개선 작업을 수행 중입니다. 각 기능은 진도가 100%일 때 서비스에 반영할 수 있습니다. 또 각 기능의 개발 속도는 모두 다르므로 뒤의 기능이 앞의 기능보다 먼저 개발될 수도 있습니다. 이때, 뒤의 기능은 앞의 기능이 배포될 때 함께 배포되어야 합니다.

배포 순서대로 작업 진도가 적힌 정수 배열 progresses와 각 작업의 개발 속도가 적힌 정수 배열 speeds가 주어질 때 각 배포마다 몇 개의 기능이 배포되는지를 반환하도록 solution() 함수를 완성하세요.

<br/>

### 2.2. 제약 조건
---

- 작업 개수(progresses, speeds의 배열 길이)는 100개 이하입니다.
- 작업 진도는 100 미만의 자연수입니다.
- 작업 속도는 100 이하의 자연수입니다.
- 배포는 하루에 한 번만 할 수 있으며, 하루의 끝에 이루어진다고 가정합니다.
    - 예를 들어 진도율이 95%인 작업의 개발 속도가 하루에 4%라면 배포는 2일 뒤에 이루어집니다.

<br/>

### 2.3. 입출력의 예
---

|progresses|speeds|return|
|---|---|---|
|[93, 30, 55]|[1, 30, 5]|[2, 1]|
|[95, 90, 99, 99, 80, 99]|[1, 1, 1, 1, 1]|[1, 3, 2]|

<br/>

### 2.4. 코드
---

```kotlin
fun solution(progresses: IntArray, speeds: IntArray): IntArray {
    val result = mutableListOf<Int>()

    // 1. progresses와 speeds를 각 인덱스의 값을 key와 value로 만든 객체를 Queue에 입력함
    val queue: Queue<Pair<Int, Int>> = LinkedList()
    for (i in progresses.indices) {
        queue.offer(Pair(progresses[i], speeds[i]))
    }

    // 2. Queue가 비어질 때까지 무한 반복함
    while (queue.isNotEmpty()) {
        // 2.1. Queue에서 첫 번째 원소의 key가 100이 넘을 때까지 계속 순회함
        while (queue.peek().first < 100) {
            // 2.1.1. 각 원소의 key에 value를 더함
            val size = queue.size
            repeat(size) {
                val (progress, speed) = queue.poll()
                queue.offer(Pair(progress + speed, speed))
            }
        }

        // 2.2. Queue에서 key가 100이 넘지 않은 원소까지 추출하면서 개수를 셈
        var count = 0
        while (queue.isNotEmpty() && queue.peek().first >= 100) {
            queue.poll()
            count++
        }

        // 2.3. 개수를 결과로 추가함
        result.add(count)
    }

    return result.toIntArray()
}
```

<br/>

## 3. (문제) 카드 뭉치
---

### 3.1. 설명
---

코니는 영어 단어가 적힌 카드 뭉치 2개를 선물로 받았습니다. 코니는 다음과 같은 규칙으로 카드에 적힌 단어들을 사용해 원하는 순서의 단어 배열을 만들 수 있는지 알고 싶습니다.

- 원하는 카드 뭉치에서 카드를 순서대로 한 장씩 사용합니다.
- 한 번 사용한 카드는 다시 사용할 수 없습니다.
- 카드를 사용하지 않고 다음 카드로 넘어갈 수 없습니다.
- 기존에 주어진 카드 뭉치의 단어 순서는 바꿀 수 없습니다.

<br/>

### 3.2. 제약 조건
---

- 1 <= cars1 의 길이, cards2 의 길이 <= 10
    - 1 <= cards1[i] 의 길이, cards2[i] 의 길이 <= 10
    - cards1과 cards2에는 서로 다른 단어만 있음
- 2 <= goal의 길이 <= cards1의 길이 + cards2의 길이
    - 1 <= gaol[i]의 길이 <= 10
    - goal의 원소는 cards1과 cards2의 원소들로만 이루어져 있음
- cards1, cards2, goal의 문자열들은 모두 알파벳 소문자로만 이루어져 있음

<br/>

### 3.3. 입출력의 예
---

|cards1|cards2|goal|result|
|---|---|---|---|
|["i", "drink", "water"]|["want", "to"]|["i", "want", "to", "drink", "water"]|"Yes"|
|["i", "water", "drink"]|["want", "to"]|["i", "want", "to", "drink", "water"]|"No"|

<br/>

### 3.4. 코드
---

```kotlin
fun solution(
    cards1: Array<String>,
    cards2: Array<String>,
    goal: Array<String>,
): String {
    //  첫 번째 카드 뭉치의 문자열 위치를 저장할 card1Index 변수 선언 후 0 저장
    var card1Index = 0
    //  두 번째 카드 뭉치의 문자열 위치를 저장할 card2Index 변수 선언 후 0 저장
    var card2Index = 0
    // 일치 횟수를 저장할 변수 선언 후 0 저장
    var count = 0
    //  goal 배열 순회
    for (string in goal) {
        //      goal 문자열과 첫 번째 카드 뭉치의 card1Index 위치 문자열과 일치하는지 확인
        //          일치할 경우
        //              card1Index 1 증가, count 1 증가
        if (card1Index < cards1.size && string == cards1[card1Index]) {
            card1Index++
            count++
        }
        //      goal 문자열과 두 번째 카드 뭉치의 card2Index 위치 문자열과 일치하는지 확인
        //          일치할 경우
        //              card2Index 1 증가, count 1 증가
        else if (card2Index < cards2.size && string == cards2[card2Index]) {
            card2Index++
            count++
        }
    }
    //  count와 goal 크기와 같을 경우 Yes 아닐 경우 No 반환
    return if (goal.size == count) "Yes" else "No"
}
```

<br/>

## Reference
---

[Programmers: 기능 개발](https://school.programmers.co.kr/learn/courses/30/lessons/42586?language=kotlin)

[Programmers: 카드 뭉치](https://school.programmers.co.kr/learn/courses/30/lessons/159994)