---
title: Hash
date: 2024-10-19 09:00:00 +0900
categories: data-structure
tags:
  - data-structure
description: 해시 자료구조에 대한 개념과 문제 풀이
---

## 1. 개념
---

해시 함수를 사용해서 변환한 값을 인덱스로 삼아 키와 값을 저장해서 빠른 데이터 탐색을 제공하는 자료구조이다.

<br/>

## 2. (문제) 완주하지 못한 선수
---

### 2.1. 설명
---

많은 선수 중 단 한 명의 선수를 제외하고 모든 선수가 마라톤을 완주하였습니다. 마라톤에 참여한 선수들의 이름이 담긴 배열 participant와 완주한 선수들의 이름이 담긴 배열 completion이 있을 때 완주하지 못한 선수의 이름을 반환하는 solution() 함수를 작성하세요.

<br/>

### 2.2. 제약 조건
---

- 마라톤 경기에 참여한 선수 수는 1명 이상 100,000명 이하입니다.
- completion 길이는 participant 길이보다 1 작습니다.
- 참가자 이름은 1개 이상 20개 이하의 알파벳 소문자로 이루어져 있습니다.
- 참가자 중에는 동명이인이 있을 수 있습니다.

<br/>

### 2.3. 입출력의 예
---

|participant|completion|return|
|---|---|---|
|["leo", "kiki", "eden"]|["eden", "kiki"]|"leo"|
|["marina", "josipa", "nikola", "vinko", "filipa" ]|["josipa", "filipa", "marina", "nikola" ]|"vinko"|
|["mislav", "stanko", "mislav", "ana"]|["stanko", "ana", "mislav"]|"mislav"|

<br/>

### 2.4. 코드
---

```kotlin
fun solution(participant: List<String>, completion: List<String>): String {
    var result = ""

    // 1. key는 String이고 value는 Int인 hashmap을 정의
    val hashmap = HashMap<String, Int>()

    // 2. participant 순회
    for (name in participant) {
        // 2.1. participant 값이 hashmap의 key로 존재하는지 확인
        if (hashmap.containsKey(name)) {
            // 2.1.true. 해당 key의 value를 1 올림
            hashmap[name] = hashmap[name]!! + 1
        } else {
            // 2.1.false. 해당 key를 추가하고 value를 1 올림
            hashmap[name] = 1
        }
    }

    // 3. completion 순회
    for (name in completion) {
        // 3.1. completion 값이 hashmap의 key로 존재하는지 확인
        if (hashmap.containsKey(name)) {
            // 3.1.true.1. 존재할 경우 해당 key의 value를 1 낮춤
            hashmap[name] = hashmap[name]!! - 1
            // 3.1.true.2. value가 0일 경우 해당 key를 hashmap에서 제거
            if (hashmap[name] == 0) {
                hashmap.remove(name)
            }
        }
    }

    // 4. hashmap의 첫 번째 key를 result로 저장
    result = hashmap.keys.first()

    return result
}
```

<br/>

## 3. (문제) 할인 행사
---

### 3.1. 설명
---

XYZ 마트는 일정 금액을 지불하면 10일 동안 회원 자격을 부여합니다. XYZ 마트에서는 회원을 대상으로 매일 1가지 제품을 할인하는 행사를 합니다. 할인 제품은 하루에 하나만 구매할 수 있습니다. 알뜰한 정현이는 자신이 원하는 제품과 수량이 할인하는 날짜와 10일 연속으로 일치할 때에 맞춰서 회원가입을 하려 합니다.

예를 들어 정현이가 원하는 제품이 바나나 3개, 사과 2개, 쌀 2개, 돼지고기 2개, 냄비 1개이고, XYZ 마트에서 14일간 회원을 대상으로 할인하는 제품이 날짜 순서대로 치킨, 사과, 사과, 바나나, 쌀, 사과, 돼지고기, 바나나, 돼지고기, 쌀, 냄비, 바나나, 사과, 바나나면 첫째 날부터 열흘 동안 냄비는 할인하지 않으므로 첫째 날에는 회원가입을 하지 않습니다. 셋째, 넷째, 다섯째 날부터 각각 열흘 동안은 원하는 제품과 수량이 일치하므로 셋 중 하루에 회원가입을 합니다.

정현이가 원하는 제품을 나타내는 문자열 배열 want와 정현이가 원하는 제품의 수량을 나타내는 정수 배열 number, XYZ 마트에서 할인하는 제품을 나타내는 문자열 배열 discount가 있을 때 회원가입 시 정현이가 원하는 제품을 모두 할인받을 수 있는 회원 등록 날짜의 총 일수를 반환하는 solution() 함수를 완성하세요. 가능한 날이 없으면 0을 return 합니다.

<br/>

### 3.2. 제약 조건
---

- 1 <= want 의 길이 = number 의 길이 <= 10
    - 1 <= number 의 원소 <= 10
    - number[i] 는 want[i]의 수량
    - number의 총합 10
- 10 <= discount의 길이 <= 100,000
- want와 discount의 원소들은 알파벳 소문자로 이루어진 문자열
    - 1 <= want의 원소의 길이, discount의 원소의 길이 <= 12

<br/>

### 3.3. 입출력의 예
---

|want|number|discount|result|
|---|---|---|---|
|["banana", "apple", "rice", "pork", "pot"]|[3, 2, 2, 2, 1]|["chicken", "apple", "apple", "banana", "rice", "apple", "pork", "banana", "pork", "rice", "pot", "banana", "apple", "banana"]|3|
|["apple"]|[10]|["banana", "banana", "banana", "banana", "banana", "banana", "banana", "banana", "banana", "banana"]|0|

<br/>

### 3.4. 코드
---

```kotlin
fun solution(
    want: Array<String>,
    number: IntArray,
    discount: Array<String>,
): Int {
    var answer = 0
    //  want를 key로 number를 value로 하는 map 생성
    val map = mutableMapOf<String, Int>()
    //  map에 want와 number를 입력
    for (i in want.indices) {
        map[want[i]] = number[i]
    }
    //  할인 목록을 저장할 queue 생성
    val queue = ArrayDeque<String>()
    //  discount 순회
    for (s in discount) {
        //      queue의 길이가 10 이상인가?
        if (queue.size >= 10) {
            //          맞을 경우
            //              queue의 첫 번째를 꺼내서 map에 동일한 key의 value를 1 증가
            val first = queue.removeFirst()
            map[first]?.also { map[first] = it + 1 }
        }
        //      map에서 discount와 동일한 key의 value를 1 감소
        map[s]?.also { map[s] = it - 1 }
        //      queue의 마지막으로 discount 입력
        queue.addLast(s)
        //      map의 모든 value가 모두 0인가?
        if (map.values.none { it != 0 }) {
            //          맞을 경우 answer를 1 증가
            answer++
        }
    }

    return answer
}
```

<br/>

## Reference
---

[Programmers: 완주하지 못한 선수](https://school.programmers.co.kr/learn/courses/30/lessons/42576)

[Programmers: 할인 행사](https://school.programmers.co.kr/learn/courses/30/lessons/131127)