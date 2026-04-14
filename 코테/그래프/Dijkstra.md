---
tags: [코테, 그래프, 최단경로]
---

# Dijkstra

## 개요

**음이 아닌** 가중치 그래프에서 단일 출발점 최단 경로. [[Heap]] 기반으로 **O((V + E) log V)**. 음수 가중치면 [[Bellman-Ford]] 사용.

## 핵심 아이디어

"현재까지 확정된 거리가 가장 짧은 정점"을 매번 꺼내 이웃 거리 갱신. 힙에서 꺼낸 거리가 **이미 저장된 dist보다 크면 스킵**(lazy deletion).

## Python 구현

```python
import heapq

def dijkstra(graph, src, n):
    """
    graph: adj list, graph[u] = [(v, w), ...]
    """
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    pq = [(0, src)]
    while pq:
        d, u = heapq.heappop(pq)
        if d > dist[u]:
            continue                 # stale entry
        for v, w in graph[u]:
            nd = d + w
            if nd < dist[v]:
                dist[v] = nd
                heapq.heappush(pq, (nd, v))
    return dist
```

## 경로 복원

`prev[v] = u`를 함께 기록하고 역추적.

```python
prev = [-1] * n
# 갱신부에서:
if nd < dist[v]:
    dist[v] = nd
    prev[v] = u
    heapq.heappush(pq, (nd, v))

def trace(prev, t):
    path = []
    while t != -1:
        path.append(t)
        t = prev[t]
    return path[::-1]
```

## K번째 최단 경로

힙에서 같은 정점을 **K번까지** 꺼내도록 허용. `cnt[v] < K`일 때만 처리.

```python
def kth_shortest(graph, src, n, K):
    cnt = [0] * n
    dist = [[] for _ in range(n)]    # 각 정점별 상위 K 거리
    pq = [(0, src)]
    while pq:
        d, u = heapq.heappop(pq)
        if cnt[u] >= K: continue
        cnt[u] += 1
        dist[u].append(d)
        for v, w in graph[u]:
            if cnt[v] < K:
                heapq.heappush(pq, (d + w, v))
    return [dist[v][K-1] if len(dist[v]) >= K else -1 for v in range(n)]
```

## 제약 상태 Dijkstra

"연료 K 이하로 주유하며 이동" 같은 문제는 상태를 `(정점, 잔여자원)`으로 확장.

```python
# dist[u][fuel] 차원으로 관리
dist = [[INF] * (F + 1) for _ in range(n)]
pq = [(0, src, F)]
...
```

## 함정

- `heappop`한 `d`가 `dist[u]`보다 크면 버리는 체크는 **필수**. 없으면 같은 정점을 여러 번 처리해 로그 요인이 제곱이 될 수 있음.
- 음수 가중치 한 개라도 있으면 **정답 보장 안 됨** → [[Bellman-Ford]].
- 밀집 그래프(E ≈ V²)에서는 O(V²) 인접행렬 버전이 오히려 빠름.

## 관련

우선순위 큐는 [[Heap]]. 0-1 가중치 특수 케이스는 [[BFS DFS 응용]]의 0-1 BFS. 모든 쌍 최단경로는 [[Floyd-Warshall]].

## 실행 예시

```python
import heapq

def dijkstra(graph, src, n):
    INF = float('inf')
    dist = [INF] * n
    prev = [-1] * n
    dist[src] = 0
    pq = [(0, src)]
    while pq:
        d, u = heapq.heappop(pq)
        if d > dist[u]: continue
        for v, w in graph[u]:
            nd = d + w
            if nd < dist[v]:
                dist[v] = nd
                prev[v] = u
                heapq.heappush(pq, (nd, v))
    return dist, prev

def trace(prev, t):
    path = []
    while t != -1:
        path.append(t)
        t = prev[t]
    return path[::-1]

# 그래프 예:
#   0 --1-- 1 --2-- 3
#   |       |       |
#   4       7       3
#   |       |       |
#   2 --2-- 3       4  (no, 3 appears twice in the diagram; using 0-3 direct)
graph = {
    0: [(1, 1), (2, 4)],
    1: [(2, 2), (3, 5)],
    2: [(3, 1)],
    3: [],
}
dist, prev = dijkstra(graph, 0, 4)
print("dist from 0:", dist)        # [0, 1, 3, 4]
print("path 0->3:", trace(prev, 3))  # [0, 1, 2, 3]
```

