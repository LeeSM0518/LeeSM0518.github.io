---
title: Stack
date: 2024-10-20 09:00:00 +0900
categories: data-structure
tags:
  - data-structure
description: 스택 자료구조에 대한 개념과 문제 풀이
---

## 1. 개념
---

먼저 들어간 것이 마지막에 나오는 선입후출(FILO, First In Last Out) 방식

<br/>

## 2. (문제) 괄호 회전하기
---

### 2.1. 설명
---

다음 규칙을 지키는 문자열을 올바른 괄호 문자열이라고 정의한다.

- `()` , `[]` , `{}` 는 모두 올바른 괄호 문자열이다.
- 만약 A가 올바른 괄호 문자열이라면, `(A)`, `[A]`, `{A}` 도 올바른 괄호 문자열이다.
- 만약 A, B가 올바른 괄호 문자열이라면, AB도 올바른 괄호 문자열이다.
    - A = `{}` , B = `([])`
    - `{}([])` 도 올바른 괄호 문자열
- 대괄호, 중괄호 그리고 소괄호로 이루어진 문자열 s가 매개변수로 주어진다.
- s를 왼쪽으로 x (0 <= x < s의 길이) 칸 만큼 회전시켰을 때 s가 올바른 괄호 문자열이 되게 하는 x의 개수를 반환하는 solution() 함수를 완성해라.

<br/>

### 2.2. 제약 조건
---

- s의 길이는 1 이상 1,000 이하

<br/>

### 2.3. 입출력의 예
---

|s|result|
|---|---|
|`[](){}`|3|
|`}]()[{`|2|

<br/>

### 2.4. 코드
---

```kotlin
import java.util.Stack

class Solution {
    fun isValid(s: String): Boolean {
        val stack = Stack<Char>()

        for (char in s) {
            // 2.1. 현재 문자를 확인
            when (char) {
                '(', '{', '[' -> stack.push(char)
                ')', '}', ']' -> {
                    // 스택에서 하나를 peek 함
                    if (stack.isEmpty()) return false
                    val top = stack.peek()
                    // 2.2. 현재 괄호와 peek한 괄호가 올바른 괄호가 동일한지 확인
                    if ((char == ')' && top == '(') ||
                        (char == '}' && top == '{') ||
                        (char == ']' && top == '[')) {
                        // 2.2.1. 동일할 경우 pop 한 후 횟수 증가
                        stack.pop()
                    } else {
                        // 2.2.2. 동일하지 않을 경우 false 반환
                        return false
                    }
                }
            }
        }
        return stack.isEmpty()
    }
    
    fun solution(s: String): Int {
        var count = 0
        val length = s.length

        for (i in 0 until length) {
            // 1. 문자열을 왼쪽으로 이동시킴
            val rotated = s.substring(i) + s.substring(0, i)

            // 2. 문자열을 순회하여 올바른 괄호인지 확인
            if (isValid(rotated)) {
                count++
            }
        }
        return count
    }
}
```

<br/>

## 3. (문제) 짝지어 제거하기
---

### 3.1. 설명
---

알파벳 소문자로 구성된 문자열에서 같은 문자열이 2개 붙어 있는 짝을 찾습니다. 짝을 찾은 다음에는 그 둘을 제거한 뒤 앞뒤로 문자열을 이어 붙입니다. 이 과정을 반복해서 문자열을 모두 제거한다면 짝지어 제거하기가 종료됩니다. 문자열 S가 주어졌을 때 짝지어 제거하기를 성공적으로 수행할 수 있는지 반환하는 함수를 완성하세요. 성공적으로 수행할 수 있으면 1을, 아니면 0을 반환해주면 됩니다.

<br/>

### 3.2. 제약 조건
---

- 문자열의 길이 : 1,000,000 이하의 자연수
- 문자열은 모두 소문자로 이루어져 있습니다.

<br/>

### 3.3. 입출력의 예
---

|s|result|
|---|---|
|baabaa|1|
|cdcd|0|

<br/>

### 3.4. 코드
---

```kotlin
fun main() {
    fun solution(s: String): Int {
        // 연결된 문자를 넣을 스택 정의
        // 이전 문자열을 저장할 변수 정의 후 빈문자열 저장
        // 변환 문자열을 저장할 변수 정의 후 원본 문자열 저장
        // 변환 문자열이 빈 문자열이 아니고 이전 문자열과 변환 문자열이 동일하지 않을 때 까지 무한 반복
        //  이전 문자열에 변환 문자열 저장
        //  변환 문자열 순회
        //      스택의 첫 번째 문자가 존재하는지 확인
        //          존재하지 않을 경우
        //              스택에 현재 문자를 추가
        //              다음 문자로 이동
        //          존재할 경우
        //              스택의 첫 번째 문자와 현재 문자가 동일한지 확인
        //                  동일할 경우
        //                      스택에서 문자 제거
        //                      다음 문자로 이동
        //                  동일하지 않을 경우
        //                      스택에 현재 문자 추가
        //                      다음 문자로 이동
        //  변환 문자열에 스택에 쌓인 문자들로 문자열 만든 후 저장
        //  스택 초기화
        // 변환 문자열이 빈 문자열일 경우 1을 빈 문자열이 아닐 경우 0을 반환
        var stack = Stack<Char>()
        var before = ""
        var after = s
        while (after.isNotEmpty() && before.equals(after).not()) {
            before = after
            for (current in after) {
                if (stack.isEmpty()) {
                    stack.push(current)
                } else {
                    if (stack.peek() == current) {
                        stack.pop()
                    } else {
                        stack.push(current)
                    }
                }
            }
            after = stack.joinToString("")
            stack.clear()
        }
        return if (after.isEmpty()) 1 else 0
    }

    println(solution("cdcd"))
}
```

<br/>

## Reference
---

[Programmers: 괄호 회전하기](https://school.programmers.co.kr/learn/courses/30/lessons/76502?language=kotlin)

[Programmers: 짝지어 제거하기](https://school.programmers.co.kr/learn/courses/30/lessons/12973)