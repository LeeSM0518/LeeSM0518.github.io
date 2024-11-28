---
title: Backtracking
date: 2024-10-25 09:00:00 +0900
categories: algorithm
tags:
  - algorithm
description: 백트래킹 알고리즘에 대한 개념과 문제 풀이에 대해 살펴본다.
---

## 1. 개념
---

**가능성이 없는 곳에서는 되돌아가고, 가능성이 있는 곳을 탐색**하는 알고리즘을 백트래킹 알고리즘이라고 한다.

해가 될 가능성이 없는 탐색 대상을 배제할 수 있으므로 완전 탐색하는 방법보다 효율적이다.

<br/>

## 2. 유망 함수
---

백트래킹 알고리즘의 핵심은 '해가 될 가능성을 판단하는 것'이다. 가능성은 유망 함수라는 것을 정의하여 판단한다.

백트래킹 알고리즘은 다음과 같은 과정으로 진행한다.

1. 유효한 해의 집합을 정의한다.
2. 위 단계에서 정의한 집합을 그래프로 표현한다.
3. 유망 함수를 정의한다.
4. 백트래킹 알고리즘을 활용해서 해를 찾는다.

<br/>

## 3. (문제) 피로도
---

### 3.1. 설명
---

든든앤파이터라는 게임에는 피로도 시스템이 있습니다. 피로도는 정수로 표시하며 일정 피로도를 사용해서 던전을 탐험할 수 있습니다. 각 던전마다 탐험을 시작하기 위해 필요한 최소 필요 피로도와 던전 탐험을 마쳤을 때 소모되는 소모 피로도가 있습니다. 예를 들어 최소 필요 피로도가 80, 소모 피로도가 20인 던전을 탐험하기 위해서는 현재 남은 피로도는 80 이상이어야 하고, 던전을 탐험한 후에는 피로도 20이 소모된다. 이 게임에는 하루에 한 번만 탐험할 수 있는 던전이 여러 개 있습니다. 한 유저가 오늘 던전을 최대한 많이 탐험하려 합니다. 유저의 현재 피로도 k와 각 던전별 최소 필요 피로도, 소모 피로도가 담긴 2차원 배열 dungeons가 매개변수로 주어질 때 유저가 탐험할 수 있는 최대 던전 수를 반환하는 solution() 함수를 완성하세요.

<br/>

### 3.2. 제약 조건
---

- k는 1 이상 5,000 이하인 자연수입니다.
- dungeons의 세로 길이, 즉 던전 개수는 1 이상 8 이하입니다.
    - dungeons의 가로 길이는 2입니다.
    - dungeons의 각 행은 각 던전의 [최소 필요 피로도, 소모 피로도]입니다.
    - 최소 필요 피로도는 항상 소모 피로도보다 크거나 같습니다.
    - 최소 필요 피로도와 소모 피로도는 1 이상 1,000 이하인 자연수입니다.
    - 서로 다른 던전의 [최소 필요 피로도, 소모 피로도]가 같을 수 있습니다.

<br/>

### 3.3. 입출력의 예
---

|k|dungeons|result|
|---|---|---|
|80|[[80,20],[50,40],[30,10]]|3|

<br/>

### 3.4. 코드
---

```kotlin
fun main() {
    fun solution(k: Int, daungeons: Array<IntArray>): Int {
        var result = 0

        fun visitDaungeon(fatigue: Int, daungeonInfo: IntArray, daungeonIndex: Int, count: Int, visit: IntArray) {
            val visited = visit.clone()
            //   1.1.1 현재 피로도로 현재 방문한 던전을 돌 수 있는지 확인한 후 돌지 못 할 경우 함수를 종료한다.
            if (fatigue < daungeonInfo[0]) {
                return
            }
            //   1.1.2 현재 던전을 방문 여부에 넣는다.
            visited[daungeonIndex] = 1
            //   1.1.3 결과에 1을 더한다.
            val visitedCount = count + 1
            //   1.1.4 현재 피로도에서 던전의 소요 피로도를 뺀다.
            val spentFatigue = fatigue - daungeonInfo[1]
            //   1.1.5 던전들에서 아직 방문하지 않은 던전을 찾아서 돈다.
            for (i in daungeons.indices) {
                if (visited[i] == 1) continue
                visitDaungeon(spentFatigue, daungeons[i], i, visitedCount, visited)
            }
            //   1.1.6 방문하지 않은 던전이 없을 경우 현재 방문 횟수가 결과 횟수보다 크면 결과 횟수를 변경한다.
            if (result < visitedCount) {
                result = visitedCount
            }
            //   1.1.7 함수를 종료한다.
            return
        }

        // 1. 각 던전을 순회한다.
        for (i in daungeons.indices) {
            //  1.1. 재귀 함수 호출
            visitDaungeon(k, daungeons[i], i, 0, IntArray(daungeons.size))
        }

        return result
    }

    val k = 80
    val daungeons = arrayOf(intArrayOf(80, 20), intArrayOf(50, 40), intArrayOf(30, 10))
    println(solution(k, daungeons))
}
```

<br/>

## Reference
---

[Programmers: 피로도](https://school.programmers.co.kr/learn/courses/30/lessons/87946?language=kotlin)