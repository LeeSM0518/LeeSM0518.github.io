---
title: Tree
date: 2024-10-17 09:00:00 +0900
categories: data-structure
tags:
  - data-structure
description: 트리 자료구조에 대한 개념과 문제 풀이
---

## 1. 개념
---

트리는 데이터를 저장하고 탐색하기에 유용한 구조를 갖고 있다.

노드는 트리를 구성하는 요소이다. 노드 중 가장 위에 있는 노드를 루트 노드라고 한다.

노드와 노드 사이에는 간선이라고 하는 이어주는 선이 있다. 간선으로 연결된 노드들은 부모-자식 관계가 있다고 표현한다. 자식이 없는 노드는 리프 노드라고 한다.

차수란 특정 노드에서 아래로 향하는 간선의 개수이다.

<br/>

### 1.1. 이진 트리
---

이진 트리는 노드 하나가 최대 2개의 자식 노드를 갖는다.

이진 트리 순회 방법

- 전위 순회 : 부모 노드 > 왼쪽 자식 노드 > 오른쪽 자식 노드
- 중위 순회 : 왼쪽 자식 노드 > 부모 노드 > 오른쪽 자식 노드
- 후위 순회 : 왼쪽 자식 노드 > 오른쪽 자식 노드 > 부모 노드

<br/>

## 2. (문제) 예상 대진표
---

### 2.1. 설명
---

게임 대회가 개최되었습니다. 이 대회는 N명이 참가하고 토너먼트 형식으로 진행합니다. N명의 참가자에게 1부터 N번의 번호를 차례로 배정하고, 1번 <> 2번, 3번 <> 4번, ... , N - 1번 <> N번 번호를 부여한 참가자끼리 게임을 진행합니다. 각 게임에서 이긴 사람은 다음 라운드에 진출합니다.

이때 다음 라운드에 진출할 참가자의 번호는 다시 1번부터 N/2번과 같이 차례로 배정합니다. 만약 1번 <> 2번끼리 겨루는 게임에서 2번이 승리하면 다음 라운드에서 2번은 1번으로 번호를 부여받습니다. 3번 <> 4번끼리 겨루는 게임에서 3번이 승리하면 다음 라운드에서 3번은 2번을 부여받습니다. 게임은 최종 한 명이 남을 때까지 진행됩니다.

이때 첫 라운드에서 A번을 가진 참가자는 경쟁자로 생각하는 B번 참가자와 몇 번째 라운드에서 만나는지 궁금해졌습니다. 게임 참가자 수 N, 참가자 번호 A, 경쟁자 번호 B가 함수 solution()의 인수로 주어질 때 첫 라운드에서 A 번을 가진 참가자는 경쟁자로 생각하는 B번 참가자와 몇 번째 라운드에서 만나는지를 반환하는 solution() 함수를 완성하세요. 단, A번 참가자와 B번 참가자는 서로 만나기 전까지 항상 이긴다고 가정합니다.

<br/>

### 2.2. 제약 조건
---

- N : 2^1 이상 2^20 이하인 자연수 (2의 지수로 주어지므로 부전승은 없음)
- A, B : N 이하인 자연수 (단, A != B)

<br/>

### 2.3. 입출력의 예
---

|N|A|B|answer|
|---|---|---|---|
|8|4|7|3|

<br/>

### 2.4. 코드
---

```kotlin
fun solution(n: Int, a: Int, b: Int): Int {
    var result = 0
    var x = a
    var y = b

    // 다음 라운드에서 받을 번호는 이전 라운드를 반으로 나눴다고 생각하면 된다.
    // 두 참가자의 번호가 같을 때까지 계산을 반복한다.
    while (x != y) {
        x = (x + 1) / 2
        y = (y + 1) / 2
        result++
    }

    return result
}
```

<br/>

## 3. (문제) 다단계 칫솔 판매
---

### 3.1. 설명
---

민호는 다단계 조직을 이용해 칫솔을 판매합니다. 판매원이 칫솔을 판매하면 그 이익이 피라미드 조직을 타고 조금씩 분배되는 형태의 판매망입니다. 어느 정도 판매가 이루어진 후 조직을 운영하던 민호는 조직 내 누가 얼마만큼의 이득을 가져갔는지 궁금해졌습니다.

모든 판매원은 칫솔 판매로 생기는 이익에서 10%를 계산해 자신을 조직에 참여시킨 추천인에게 배분하고 나머지는 자신이 가집니다. 그리고 모든 판매원은 자신이 칫솔 판매에서 발생한 이익뿐만 아니라 자신이 조직에 추천하여 가입시킨 판매원의 이익금의 10%를 자신이 갖습니다. 자신에게 발생하는 이익 또한 마찬가지의 규칙으로 자신의 추천인에게 분배됩니다. 단, 10%를 계산할 때는 원 단위에서 자르고, 10%를 계산한 금액이 1원 미만이면 이득을 분배하지 않고 자신이 모두 가집니다.

각 판매원의 이름을 담은 배열 enroll, 각 판매원을 다단계 조직에 참여시킨 다른 판매원의 이름을 담은 배열 referral, 판매량 집계 데이터의 판매원 이름을 나열한 배열 seller, 판매량 집계 데이터의 판매 수량을 나열한 배열 amount가 주어질 때 각 판매원이 얻은 이익금을 나열한 배열을 반환하는 solution() 함수를 작성하세요. 판매원에게 배분된 이익금의 총합을 정수형으로 계산해 입력으로 주어진 enroll에 이름이 포함된 순서에 따라 나열하면 됩니다.

<br/>

### 3.2. 제약 조건
---

- enroll의 길이는 1 이상 10,000 이하입니다.
    - enroll에 민호의 이름은 없습니다. 따라서 enroll의 길이는 민호를 제외한 조직 구성원의 총 수 입니다.
- referral의 길이는 enroll의 길이와 같습니다.
    - referral 내에서 i 번째에 있는 이름은 배열 enroll 내에서 i 번째에 있는 판매원을 조직에 참여시킨 사람의 이름입니다.
    - 어느 누구의 추천도 없이 조직에 참여한 사람에 대해서는 referral 배열 내에 추천인의 이름을 기입하지 않고 - 를 기입합니다.
    - enroll에 등장하는 이름은 조직에 참여한 순서에 따릅니다.
    - 즉, 어느 판매원의 이름이 enroll의 i번째에 등장한다면, 이 판매원을 조직에 참여시킨 사람의 이름, 즉 referral의 i 번째 노드는 이미 배열 enroll의 j번째 (j < i)에 등장했음이 보장됩니다.
- seller의 길이는 1 이상 100,000 이하입니다.
    - seller 내의 i번째에 있는 이름은 i번째 판매 집계 데이터가 어느 판매원에 의한 것인지를 나타냅니다.
    - seller에는 같은 이름이 중복해서 들어 있을 수 있습니다.
- amount의 길이는 seller의 길이와 같습니다.
    - amount 내의 i번째에 있는 수는 i번째 판매 집계 데이터의 판매량을 나타냅니다.
    - 판매량의 범위, 즉, amount의 노드들의 범위는 1 이상 100 이하인 자연수 입니다.
- 칫솔 1개를 판매한 이익금은 100원으로 정해져 있습니다.
- 모든 조직 구성원들의 이름은 10 글자 이내의 영문 알파벳 소문자들로만 이루어져 있습니다.

<br/>

### 3.3. 입출력의 예
---

|enroll|referral|seller|amount|result|
|---|---|---|---|---|
|`["john", "mary", "edward", "sam", "emily", "jaimie", "tod", "young"]`|`["-", "-", "mary", "edward", "mary", "mary", "jaimie", "edward"]`|`["young", "john", "tod", "emily", "mary"]`|[12, 4, 2, 5, 10]|[360, 958, 108, 0, 450, 18, 180, 1080]|
|`["john", "mary", "edward", "sam", "emily", "jaimie", "tod", "young"]`|`["-", "-", "mary", "edward", "mary", "mary", "jaimie", "edward"]`|`["sam", "emily", "jaimie", "edward"]`|[2, 3, 5, 4]|[0, 110, 378, 180, 270, 450, 0, 0]|

<br/>

### 3.4. 코드
---

```kotlin
fun solution(
        enroll: Array<String>,
        referral: Array<String>,
        seller: Array<String>,
        amount: IntArray,
    ): IntArray {
        //  enroll과 referral를 저장할 treeHashMap 선언
        val treeHashMap = hashMapOf<String, String>()
        //  seller와 amount를 저장하기 위해 sellerList 선언
        val sellerList = ArrayList<Pair<String, Int>>()
        //  판매자와 이익금을 저장할 resultLinkedHashMap 선언
        val resultLinkedHashMap = LinkedHashMap<String, Int>()
        //  enroll 크기만큼 순회
        for (i in enroll.indices) {
            //      treeHashmap[enroll[i]]에 referral[i] 저장
            treeHashMap[enroll[i]] = referral[i]
            resultLinkedHashMap[enroll[i]] = 0
        }
        //  seller 크기만큼 순회
        for (i in seller.indices) {
            val realAmount = amount[i] * 100
            sellerList.add(seller[i] to realAmount)
        }

        fun treeTraversal(key: String, value: Int) {
            val profit = ceil((value * 0.9)).toInt()
            val divideAmount = value - profit
            if (divideAmount < 1) {
                resultLinkedHashMap[key] = resultLinkedHashMap[key]!! + value
                return
            } else {
                resultLinkedHashMap[key] = resultLinkedHashMap[key]!! + profit
            }

            //          treeHashMap[key].value 가 "-" 인가?
            if (treeHashMap[key] != "-") {
                //              아닐 경우 treeHashMap[key].value를 key로 하고 value * 0.1를 value로 재귀 함수 호출
                treeTraversal(treeHashMap[key]!!, divideAmount)
            } else {
                //              맞을 경우 함수 종료
                return
            }
        }
        //  sellerList 순회
        sellerList.forEach { (key, value) ->
            //      sellerList[i]의 key와 value로 재귀 함수 호출
            treeTraversal(key, value)
        }
        return resultLinkedHashMap.values.toIntArray()
    }
```

<br/>

## Reference
---

[Programmers: 예상 대진표](https://school.programmers.co.kr/learn/courses/30/lessons/12985)

[Programmers: 다단계 칫솔 판매](https://school.programmers.co.kr/learn/courses/30/lessons/77486)