---
title: Graph
date: 2024-10-15 09:00:00 +0900
categories: data-structure
tags:
  - data-structure
description: 그래프 자료구조에 대한 개념과 문제 풀이
---

## 1. 개념
---

그래프는 노드(vertex)와 간선(edge)을 이용한 비선형 데이터 구조이다.

데이터를 노드로, 노드 간의 관계나 흐름을 간선으로 표현한다.

간선은 방향이 있을 수도 있고 없을 수도 있다.

만약 관계나 흐름에서 정도를 표현할 필요가 있다면 가중치로 표현한다.

<br/>

### 1.1. 그래프 구현
---

그래프의 구현 방식에는 인접 행렬(adjacency matrix)과 인접 리스트(adjacency list)가 있다.

<br/>

#### 1.1.1. 인접 행렬
---

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252FF5nLV2xV9dMsmpDX2lib%252Fimage.png%3Falt%3Dmedia%26token%3D76ab2b22-ef9e-4705-b5ca-5420efbf86c4&width=768&dpr=4&quality=100&sign=73c41eb2&sv=1)

- 구조
    - 배열 구조 : 2차원 배열   
    - 배열의 인덱스 : 노드
    - 배열의 값 : 노드의 가중치
    - 행 인덱스 : 출발
    - 열 인덱스 : 도착
- 장점
    - 간선의 정보를 확인할 때의 시간 복잡도가 O(1) 이다.   
- 단점
    - 노드 수에 비해 간선 수가 매우 적을 경우 N x N 크기의 공간이 낭비된다.   
    - 노드들의 값의 차이가 매우 클 경우 해당 값으로 행렬의 크기를 잡아야 하므로 낭비된다.

<br/>

#### 1.1.2. 인접 리스트
---

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252FuZtCKr7Z0ec9dVzKhMmA%252Fimage.png%3Falt%3Dmedia%26token%3D6ff3f7e6-55ff-4593-a571-1c2ebd8040d9&width=768&dpr=4&quality=100&sign=62739864&sv=1)

- 노드 : [정점(v), 가중치(w)]
- 장점
    - 정점에 연결된 간선의 정보를 빠르게 알 수 있다.   
    - 공간 복잡도가 O(E[간선수] + V[정점수]) 이다.
- 단점
    - A에서 B로 연결된 간선을 리스트에 연결된 개수만큼 소요된다.   

<br/>

### 1.2. 그래프 탐색
---

그래프 탐색에는 깊이 우선 탐색(depth-first search, DFS)과 너비 우선 탐색(breadth first search, BFS)이 있다.

<br/>

#### 1.2.1. 깊이 우선 탐색
---

깊이 우선 탐색은 시작 노드부터 탐색을 시작하여 간선을 따라 최대 깊이 노드까지 이동하며 차례대로 방문한다.

일단 시작 노드를 정하고, 스택에 시작 노드를 push 한다. 스택에 있는 노드는 아직 방문하지 않았지만 문할 예정인 노드이다. 시작 노드를 push 했으면 다음 과정을 반복한다.

1. 스택이 비었는지 확인한다. 스택이 비었으면 탐색을 종료한다.
2. 스택에서 노드를 pop 한다.
3. pop 한 노드의 방문 여부를 확인하고 방문 처리한다.
4. 방문한 노드와 인접한 모든 노드를 확인하고, 방문하지 않았다면 노드를 스택에 push 한다.

이 과정을 코드로 구현할 때 다음 세 가지 사항을 고려해야 한다.

1. 탐색할 노드가 없을 때까지 간선을 타고 내려갈 수 있어야 한다.
2. 가장 최근에 방문한 노드를 알아야 한다.
3. 이미 방문한 노드인지 확인할 수 있어야 한다.

깊이 우선 탐색의 핵심은 '가장 깊은 노드까지 방문한 후에 더 이상 방문할 노드가 없으면 최근 방문한 노드로 돌아온 다음, 해당 노드에서 방문할 노드가 있는지 확인한다' 이다.

<br/>

깊이 우선 탐색 구현 코드

```kotlin
// 깊이 우선 탐색 (DFS) 구현 예제

// 그래프를 인접 리스트로 표현하기 위한 클래스
class Graph(private val numVertices: Int) {
    // 각 정점에 연결된 인접 정점들의 리스트를 저장하는 배열
    private val adjacencyList: Array<MutableList<Int>> = Array(numVertices) { mutableListOf() }

    // 간선 추가 함수
    fun addEdge(source: Int, destination: Int) {
        adjacencyList[source].add(destination)
        // 무방향 그래프의 경우 다음 줄의 주석을 해제하여 양방향 간선 추가
        // adjacencyList[destination].add(source)
    }

    // DFS 탐색 시작 함수
    fun dfs(startVertex: Int) {
        val visited = BooleanArray(numVertices) // 방문한 정점을 표시하는 배열
        dfsRecursive(startVertex, visited) // 재귀적으로 DFS 수행
    }

    // DFS를 재귀적으로 구현한 함수
    private fun dfsRecursive(vertex: Int, visited: BooleanArray) {
        visited[vertex] = true // 현재 정점을 방문으로 표시
        println("방문한 노드: $vertex") // 방문한 정점을 출력

        // 현재 정점에 인접한 모든 정점을 순회
        for (adjacentVertex in adjacencyList[vertex]) {
            if (!visited[adjacentVertex]) {
                dfsRecursive(adjacentVertex, visited) // 미방문 정점에 대해 재귀 호출
            }
        }
    }
}

fun main() {
    val graph = Graph(6) // 정점의 개수가 6인 그래프 생성

    // 간선 추가
    graph.addEdge(0, 1)
    graph.addEdge(0, 2)
    graph.addEdge(1, 3)
    graph.addEdge(1, 4)
    graph.addEdge(2, 5)

    // DFS 탐색 시작
    println("깊이 우선 탐색 결과:")
    graph.dfs(0) // 노드 0에서 DFS 수행
}
```

<br/>

#### 1.2.2. 너비 우선 탐색
---

너비 우선 탐색은 시작 노드와 거리가 가장 가까운 노드를 우선하여 방문하는 방식의 알고리즘이다.

일단 시작 노드를 정하고, 큐에 시작 노드를 add 한다. 시작 노드를 큐에 add 하면서 방문 처리한다. 큐에 있는 노드는 이미 방문 처리했고, 그 노드와 인접한 노드는 아직 탐색하지 않은 상태라고 생각하면 된다. 이후 다음 과정을 반복한다.

1. 큐가 비었는지 확인한다. 큐가 비었다면 탐색을 종료한다.
2. 큐에서 노드를 poll 한다.
3. poll 한 노드와 인접한 모든 노드를 확인하고, 방문하지 않은 노드를 큐에 add 하며 방문 처리한다.

이 과정을 코드로 구현할 때 다음 두 가지 사항을 고려해야 한다.

1. 현재 방문한 노드와 직접 연결된 모든 노드를 방문할 수 있어야 한다.
2. 이미 방문한 노드인지 확인할 수 있어야 한다.

<br/>

너비 우선 탐색 구현 코드

```kotlin
// 너비 우선 탐색 (BFS) 구현 예제

import java.util.LinkedList
import java.util.Queue

// 그래프를 인접 리스트로 표현하기 위한 클래스
class Graph(private val numVertices: Int) {
    // 각 정점에 연결된 인접 정점들의 리스트를 저장하는 배열
    private val adjacencyList: Array<MutableList<Int>> = Array(numVertices) { mutableListOf() }

    // 간선 추가 함수
    fun addEdge(source: Int, destination: Int) {
        adjacencyList[source].add(destination)
        // 무방향 그래프의 경우 다음 줄의 주석을 해제하여 양방향 간선 추가
        // adjacencyList[destination].add(source)
    }

    // BFS 탐색 시작 함수
    fun bfs(startVertex: Int) {
        val visited = BooleanArray(numVertices) // 방문한 정점을 표시하는 배열
        val queue: Queue<Int> = LinkedList() // 탐색에 사용할 큐

        visited[startVertex] = true // 시작 정점을 방문으로 표시
        queue.add(startVertex) // 시작 정점을 큐에 추가

        while (queue.isNotEmpty()) {
            val vertex = queue.poll() // 큐에서 정점을 하나 꺼냄
            println("방문한 노드: $vertex") // 방문한 정점을 출력

            // 현재 정점에 인접한 모든 정점을 순회
            for (adjacentVertex in adjacencyList[vertex]) {
                if (!visited[adjacentVertex]) {
                    visited[adjacentVertex] = true // 인접 정점을 방문으로 표시
                    queue.add(adjacentVertex) // 인접 정점을 큐에 추가
                }
            }
        }
    }
}

fun main() {
    val graph = Graph(6) // 정점의 개수가 6인 그래프 생성

    // 간선 추가
    graph.addEdge(0, 1)
    graph.addEdge(0, 2)
    graph.addEdge(1, 3)
    graph.addEdge(1, 4)
    graph.addEdge(2, 5)

    // BFS 탐색 시작
    println("너비 우선 탐색 결과:")
    graph.bfs(0) // 노드 0에서 BFS 수행
}
```

<br/>

#### 1.2.3. 깊이 우선 탐색과 너비 우선 탐색 비교
---

- 깊이 우선 탐색
    - 더 이상 탐색할 수 없으면 백트래킹하여 최근 방문 노드부터 다시 탐색을 진행한다는 특성
    - 모든 가능한 해를 찾는 백트래킹 알고리즘을 구현할 때 사용
    - 그래프의 사이클을 감지해야 하는 경우 활용
    - 탐색을 해야할 때, 최단 경로를 찾는 문제가 아니면 깊이 우선 탐색을 우선 고려
- 너비 우선 탐색
    - 찾은 노드가 시작 노드로부터 최단 경로임을 보장한다.
    - 답이 많은 경우 가장 가까운 답을 찾을 때 유용하다.
    - 미로 찾기 문제에서 최단 경로를 찾거나, 네트워크 분석 문제를 풀 때 활용

<br/>

### 1.3. 그래프 최단 경로 구하기
---

최단 경로는 그래프의 종류에 따라 다르게 해석될 수 있다.
- 가중치가 없는 경우 : 간선 개수가 가장 적은 경로
- 가중치가 있는 경우 : 간선의 가중치의 총합이 최소

최단 경로를 구하는 대표적인 알고리즘인 다익스트라 알고리즘, 벨만-포드 알고리즘이 있다.

<br/>

#### 1.3.1. 다익스트라 알고리즘
---

가중치가 있는 그래프의 최단 경로를 구하는 문제는 대부분 다익스트라 알고리즘을 사용한다.

다익스트라 알고리즘은 다음과 같은 과정으로 동작한다.

1. 시작 노드를 설정하고 시작 노드로부터 특정 노드까지의 최소 비용을 저장할 공간과 직전 노드를 저장할 공간을 마련한다.
    1. 최소 비용을 저장할 공간은 모두 매우 큰 값(INF, 무한대)으로 초기화한다. 직전 노드를 저장할 공간도 INF로 초기화한다.
    2. 시작 노드의 최소 비용은 0, 직전 노드는 자신으로 한다.
2. 해당 노드를 통해 방문할 수 있는 노드 중, 즉 아직 방문하지 않은 노드 중 현재까지 구한 최소 비용이 가장 적은 노드를 선택한다.
    1. 해당 노드를 거쳐서 각 노드까지 가는 최소 비용과 현재까지 구한 최소 비용을 비교하여 작은 값을 각 노드의 최소 비용으로 갱신한다.
    2. 이때 직전 노드도 같이 갱신한다.
3. 노드 개수에서 1을 뺀 만큼 반복한다.

<br/>

다익스트라 구현 코드

```kotlin
import java.util.PriorityQueue

// 간선을 나타내는 데이터 클래스
data class Edge(val to: Int, val weight: Int)

// 우선순위 큐에서 사용할 노드를 나타내는 데이터 클래스
data class Node(val index: Int, val distance: Int) : Comparable<Node> {
    // 거리(distance)를 기준으로 정렬하기 위해 Comparable 인터페이스 구현
    override fun compareTo(other: Node): Int = this.distance - other.distance
}

// 다익스트라 알고리즘 함수
fun dijkstra(graph: List<List<Edge>>, start: Int): IntArray {
    val n = graph.size
    // 시작 노드로부터 각 노드까지의 최단 거리를 저장하는 배열
    val distances = IntArray(n) { Int.MAX_VALUE }
    distances[start] = 0 // 시작 노드까지의 거리는 0으로 설정

    // 우선순위 큐를 사용하여 방문할 노드를 관리
    val priorityQueue = PriorityQueue<Node>()
    priorityQueue.add(Node(start, 0)) // 시작 노드를 큐에 추가

    while (priorityQueue.isNotEmpty()) {
        val currentNode = priorityQueue.poll() // 가장 작은 거리를 가진 노드를 꺼냄
        val currentIndex = currentNode.index

        // 이미 처리된 노드라면 스킵
        if (distances[currentIndex] < currentNode.distance) continue

        // 현재 노드와 인접한 모든 노드를 확인
        for (edge in graph[currentIndex]) {
            val nextIndex = edge.to
            val newDist = distances[currentIndex] + edge.weight // 새로운 거리 계산

            // 새로운 거리가 기존 거리보다 작으면 업데이트
            if (newDist < distances[nextIndex]) {
                distances[nextIndex] = newDist
                priorityQueue.add(Node(nextIndex, newDist)) // 다음 노드를 큐에 추가
            }
        }
    }

    return distances // 시작 노드로부터 각 노드까지의 최단 거리 배열 반환
}

fun main() {
    // 그래프 초기화
    val n = 5 // 노드의 개수
    val graph = List(n) { mutableListOf<Edge>() } // 각 노드에 대한 인접 리스트 생성

    // 간선 추가 (예시)
    graph[0].add(Edge(1, 2)) // 노드 0에서 노드 1로 가중치 2인 간선
    graph[0].add(Edge(2, 5)) // 노드 0에서 노드 2로 가중치 5인 간선
    graph[1].add(Edge(2, 1)) // 노드 1에서 노드 2로 가중치 1인 간선
    graph[1].add(Edge(3, 2)) // 노드 1에서 노드 3으로 가중치 2인 간선
    graph[2].add(Edge(3, 3)) // 노드 2에서 노드 3으로 가중치 3인 간선
    graph[2].add(Edge(4, 1)) // 노드 2에서 노드 4로 가중치 1인 간선
    graph[3].add(Edge(4, 1)) // 노드 3에서 노드 4로 가중치 1인 간선

    val startNode = 0 // 시작 노드 설정
    val distances = dijkstra(graph, startNode) // 다익스트라 알고리즘 실행

    // 결과 출력
    for (i in distances.indices) {
        println("노드 $startNode 에서 노드 $i 까지의 최단 거리: ${distances[i]}")
    }
}
```

<br/>

#### 1.3.2. 벨만-포드 알고리즘
---

벨만-포드 알고리즘은 매 단계마다 모든 간선의 가중치를 다시 확인하여 최소 비용을 갱신하므로 음의 가중치를 가지는 그래프에서도 최단 경로를 구할 수 있다.

밸만-포드 알고리즘은 다음과 같이 동작한다.

1. 시작 노드를 설정한 다음 시작 노드의 최소 비용은 0, 나머지 노드는 INF로 초기화한다. 이후 최소 비용을 갱신할 때 직전 노드도 갱신한다.
2. 노드 개수 - 1 만큼 다음 연산을 반복한다.
    1. 시작 노드에서 갈 수 있는 각 노드에 대하여 전체 노드 각각을 거쳐갈 때 현재까지 구한 최소 비용보다 더 적은 최소 비용이 있는지 확인하여 갱신한다. 최소 비용을 갱신할 때, V의 직전 노드 값도 같이 갱신한다.
3. 과정 (2)를 마지막으로 한 번 더 수행하여 갱신되는 최소 비용이 있는지 확인한다. 만약 있다면 음의 순환이 있음을 의미한다.

<br/>

벨만-포드 구현 코드

```kotlin
// 간선을 나타내는 데이터 클래스
data class Edge(val from: Int, val to: Int, val weight: Int)

// 벨만-포드 알고리즘 함수
fun bellmanFord(edges: List<Edge>, vertexCount: Int, start: Int): Pair<IntArray, Boolean> {
    // 시작 노드로부터 각 노드까지의 최단 거리를 저장하는 배열
    val distances = IntArray(vertexCount) { Int.MAX_VALUE }
    distances[start] = 0 // 시작 노드까지의 거리는 0으로 설정

    // 최단 경로가 업데이트되었는지 여부를 나타내는 변수
    var updated: Boolean

    // 그래프에 있는 모든 간선에 대해 V-1번 반복
    for (i in 0 until vertexCount - 1) {
        updated = false
        for (edge in edges) {
            // 현재 간선을 통해 더 짧은 경로를 발견한 경우 업데이트
            if (distances[edge.from] != Int.MAX_VALUE && distances[edge.from] + edge.weight < distances[edge.to]) {
                distances[edge.to] = distances[edge.from] + edge.weight
                updated = true
            }
        }
        // 더 이상 업데이트가 발생하지 않으면 조기 종료
        if (!updated) break
    }

    // 음수 사이클 존재 여부 검사
    for (edge in edges) {
        if (distances[edge.from] != Int.MAX_VALUE && distances[edge.from] + edge.weight < distances[edge.to]) {
            // 음수 사이클이 존재하면 true 반환
            return Pair(distances, true)
        }
    }

    // 음수 사이클이 없으면 false 반환
    return Pair(distances, false)
}

fun main() {
    // 노드의 개수와 간선의 개수 설정
    val vertexCount = 5
    val edgeList = mutableListOf<Edge>()

    // 간선 추가 (예시)
    edgeList.add(Edge(0, 1, 6))   // 노드 0에서 노드 1로 가중치 6인 간선
    edgeList.add(Edge(0, 2, 7))   // 노드 0에서 노드 2로 가중치 7인 간선
    edgeList.add(Edge(1, 2, 8))   // 노드 1에서 노드 2로 가중치 8인 간선
    edgeList.add(Edge(1, 3, 5))   // 노드 1에서 노드 3으로 가중치 5인 간선
    edgeList.add(Edge(1, 4, -4))  // 노드 1에서 노드 4로 가중치 -4인 간선
    edgeList.add(Edge(2, 3, -3))  // 노드 2에서 노드 3으로 가중치 -3인 간선
    edgeList.add(Edge(2, 4, 9))   // 노드 2에서 노드 4로 가중치 9인 간선
    edgeList.add(Edge(3, 1, -2))  // 노드 3에서 노드 1로 가중치 -2인 간선
    edgeList.add(Edge(4, 0, 2))   // 노드 4에서 노드 0으로 가중치 2인 간선
    edgeList.add(Edge(4, 3, 7))   // 노드 4에서 노드 3으로 가중치 7인 간선

    val startNode = 0 // 시작 노드 설정

    // 벨만-포드 알고리즘 실행
    val (distances, hasNegativeCycle) = bellmanFord(edgeList, vertexCount, startNode)

    if (hasNegativeCycle) {
        println("음수 사이클이 존재합니다.")
    } else {
        // 결과 출력
        for (i in distances.indices) {
            val distanceStr = if (distances[i] == Int.MAX_VALUE) "도달 불가" else distances[i].toString()
            println("노드 $startNode 에서 노드 $i 까지의 최단 거리: $distanceStr")
        }
    }
}
```

<br/>

## 2. (문제) 게임 맵 최단 거리
---

### 2.1. 설명
---

ROR 게임은 두 팀으로 나누어서 진행하며 상대 팀 진영을 먼저 파괴하면 이기는 게임입니다. 그러므로 각 팀은 상대팀 진영에 최대한 빨리 도착해야 게임을 유리하게 이끌 수 있습니다. 지금부터 당신은 어느 한팀의 팀원이 되어 게임을 진행하려고 합니다. 다음은 5x5 크기의 맵에 당신의 캐릭터가 (1,1) 위치에 있고 상대 팀의 진영은 (5,5) 위치에 있는 예를 보여줍니다.

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252FsfwhPwOC8tBQHo7kj6QS%252Fimage.png%3Falt%3Dmedia%26token%3D9c052fb0-fbf6-4ee1-b617-2fd3c76b9e24&width=768&dpr=4&quality=100&sign=c07ef3f&sv=1)

위 그림에서 검은색은 벽이고, 흰색은 길입니다. 캐릭터가 움직일 때는 동, 서, 남, 북 방향으로 한 칸씩 이동하며 맵을 벗어날 수 없습니다. 다음은 현재 상황에서 캐릭터가 상태 팀 진영으로 가는 2가지 방법을 보여줍니다. 그림에서 보듯 첫 번째 방법이 상대 팀 진영에 도착하는 방법입니다.

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252FGfWzhaPEVWgvJm3soHqb%252Fimage.png%3Falt%3Dmedia%26token%3Dd7239812-939d-4552-84ba-440e6c88cfe3&width=768&dpr=4&quality=100&sign=d65d5fdb&sv=1)

만약 이렇게 벽이 세워져 있다면 캐릭터는 상대 팀 진영에 도착할 수 없습니다.

![](https://jimmyblog.gitbook.io/~gitbook/image?url=https%3A%2F%2F3077334021-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FxFAbVboeQj3rgqWe5uLh%252Fuploads%252FUlh0JJqMJqKnx26nMjE6%252Fimage.png%3Falt%3Dmedia%26token%3D7c67da5c-2d8d-4731-9c1f-031dee5af824&width=768&dpr=4&quality=100&sign=c9e22738&sv=1)

게임 맵이 maps로 주어질 때 우리 팀 캐릭터가 상대 팀 진영에 도착하기 위해 지나가야 하는 길의 최소 개수를 반환하는 solution() 함수를 완성하세요. 만약 상대 팀 진영에 도착할 수 없다면 -1을 반환하세요.

<br/>

### 2.2. 제약 조건
---

- maps는 n x m 크기의 게임 맵의 상태가 들어 있는 2차원 배열입니다.
    - n과 m은 각각 1 이상 100 이하의 자연수입니다.
    - n과 m은 서로 같거나 다를 수 있습니다.    
    - n과 m이 모두 1이면 입력으로 주어지지 않습니다.
- maps는 0과 1로 이루어져 있습니다.
    - 0은 벽이 있는 자리입니다.
    - 1은 벽이 없는 자리입니다.
- 처음에 캐릭터는 (1, 1) 위치에 있고, 상대 팀 진영은 (n, m) 위치에 있습니다.

<br/>

### 2.3. 입출력의 예
---

|maps|answer|
|---|---|
|[[1,0,1,1,1],[1,0,1,0,1],[1,0,1,1,1],[1,1,1,0,1],[0,0,0,0,1]]|11|
|[[1,0,1,1,1],[1,0,1,0,1],[1,0,1,1,1],[1,1,1,0,0],[0,0,0,0,1]]|-1|

<br/>

### 2.4. 코드
---

```kotlin
fun main() {
    class Node(val row: Int, val col: Int)

    fun solution(maps: List<List<Int>>): Int {
        // 1. 이동할 수 있는 방향을 나타내는 리스트 생성
        val direction = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

        // 2. 맵의 크기를 저장
        val n = maps.size
        val m = maps[0].size

        // 3. 최단 거리를 저장할 배열 생성
        val distance = MutableList(n) { MutableList(m) { 0 } }

        // 4. bfs 탐색을 위한 큐 생성
        val queue = ArrayDeque<Node>()

        // 5. 시작 정점에 대해서 큐에 추가, 최단 거리 저장
        queue.addLast(Node(0, 0))
        distance[0][0] = 1

        // 6. queue가 빌 때까지 반복
        while (queue.isNotEmpty()) {
            val now = queue.removeFirst()

            // 7. 현재 위치에서 이동할 수 있는 모든 방향
            for (i in 0..3) {
                val nextRow = now.row + direction[i].first
                val nextCol = now.col + direction[i].second

                // 8. 맵 밖으로 나가는 경우 예외 처리
                if (nextRow < 0 || nextCol < 0 || nextRow >= n || nextCol >= m) continue

                // 9. 벽으로 가는 경우 예외 처리
                if (maps[nextRow][nextCol] == 0) continue

                // 10. 이동한 위치가 처음 방문하는 경우, queue에 추가하고 거리 갱신
                if (distance[nextRow][nextCol] == 0) {
                    queue.addLast(Node(nextRow, nextCol))
                    distance[nextRow][nextCol] = distance[now.row][now.col] + 1
                }
                distance.forEach { it.forEach { print("$it ") }; println() }
                println()
            }
        }

        return if (distance[n - 1][m - 1] == 0) -1 else distance[n - 1][m - 1]
    }

    val maps = listOf(
        listOf(1, 0, 1, 1, 1),
        listOf(1, 0, 1, 0, 1),
        listOf(1, 0, 1, 1, 1),
        listOf(1, 1, 1, 0, 1),
        listOf(0, 0, 0, 0, 1)
    )
    println(solution(maps))
}
```

<br/>

## 3. (문제) 네트워크
---

### 3.1. 설명
---

네트워크란 컴퓨터 상호 간에 정보를 교환하도록 연결된 어떤 형태를 의미합니다. 예를 들어 컴퓨터 A, B가 직접 연결되어 있고 컴퓨터 B, C가 직접 연결되어 있을 때 컴퓨터 A, C는 간접 연결되어 있어 정보를 교환할 수 있습니다. 그러면 컴퓨터 A, B, C는 모두 같은 네트워크 상에 있다고 할 수 있습니다. 컴퓨터 개수가 n, 연결 정보가 담긴 2차원 배열 computers가 주어질 때 네트워크 개수를 반환하는 solution() 함수를 작성하세요.

<br/>

### 3.2. 제약 조건
---

- 컴퓨터의 개수 n은 1 이상 200 이하인 자연수입니다.
- 각 컴퓨터는 0부터 n - 1인 정수로 표현합니다.
- i번 컴퓨터와 j번 컴퓨터가 연결되어 있으면 computers[i][j]를 1로 표현합니다.
- computers[i][i]는 항상 1입니다.

<br/>

### 3.3. 입출력의 예
---

|n|computers|return|
|---|---|---|
|3|[[1, 1, 0], [1, 1, 0], [0, 0, 1]]|2|
|3|[[1, 1, 0], [1, 1, 1], [0, 1, 1]]|1|

<br/>

### 3.4. 코드
---

```kotlin
fun solution(n: Int, computers: Array<IntArray>): Int {
    // 네트워크 수를 컴퓨터의 개수로 초기화
    var count = computers.size
    // computers 배열을 복제하여 그래프 생성
    val graph = computers.clone()
    // 컴퓨터 방문 여부를 저장할 배열 초기화
    val visited = Array(count) { false }
    // 자기 자신과의 연결을 제거 (대각선 요소를 0으로 설정)
    for (i in graph.indices) {
        graph[i][i] = 0
    }

    // 깊이 우선 탐색 함수 정의
    fun dfs(i: Int, j: Int) {
        // i와 j 사이의 연결을 제거하여 중복 방문 방지
        graph[i][j] = 0
        graph[j][i] = 0
        // 컴퓨터 i를 방문으로 표시
        visited[i] = true
        // 컴퓨터 j가 방문되지 않았다면
        if (!visited[j]) {
            // 컴퓨터 j를 방문으로 표시하고 네트워크 수 감소
            visited[j] = true
            count--
        }
        // 컴퓨터 j에 연결된 모든 컴퓨터를 탐색
        for (k in graph[j].indices) {
            // 연결이 없으면 다음으로 이동
            if (graph[j][k] == 0) continue
            // 연결이 있으면 재귀적으로 dfs 호출
            else dfs(j, k)
        }
    }

    // 그래프를 순회하며 연결된 컴퓨터를 찾음
    for (i in graph.indices) {
        for (j in graph[i].indices) {
            // i와 j가 연결되어 있으면 dfs 호출
            if (graph[i][j] == 1) {
                dfs(i, j)
            }
        }
    }
    return count
}
```

<br/>

## Reference
---

[Programmers: 게임 맵 최단거리](https://school.programmers.co.kr/learn/courses/30/lessons/1844)

[Programmers: 네트워크](https://school.programmers.co.kr/learn/courses/30/lessons/43162)