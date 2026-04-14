---
tags: [코테, 그래프, 최단경로]
---

# Bellman-Ford

## 개요

**음수 가중치 허용** 단일 출발점 최단 경로. V-1번 모든 간선을 **완화(relax)**. O(V·E). 음수 사이클이 닿으면 **V번째 반복에서 갱신이 일어남** → 음수 사이클 탐지에 이용.

[[Dijkstra]]가 훨씬 빠르지만 음수 가중치가 있으면 쓸 수 없음.

## 핵심 구현

```python
def bellman_ford(n, edges, src):
    """
    edges: [(u, v, w), ...]
    반환: (dist, has_neg_cycle)
    """
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    for i in range(n):
        updated = False
        for u, v, w in edges:
            if dist[u] != INF and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                updated = True
                if i == n - 1:
                    return dist, True    # 음수 사이클
        if not updated:
            break
    return dist, False
```

최적화: 한 번 반복에서 갱신이 없으면 즉시 종료 (`updated` 플래그).

## 음수 사이클이 **도달 가능한 정점만** 영향

단순히 음수 사이클 존재 여부가 아니라 "**src에서 도달 가능한 음수 사이클**"을 판정하려면:

```python
# V-1번 완화 후 한 번 더 돌며 갱신되면, 그 갱신된 정점은 -INF로 마킹.
# BFS로 마킹을 전파해 도달 가능 영역 전체를 -INF로.
```

## SPFA (큐 기반 최적화)

평균적으로 훨씬 빠른 구현. 최악은 여전히 O(V·E)지만 실전 성능 우수.

```python
from collections import deque

def spfa(graph, src, n):
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    in_queue = [False] * n
    cnt = [0] * n
    dq = deque([src])
    in_queue[src] = True
    while dq:
        u = dq.popleft()
        in_queue[u] = False
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                cnt[v] += 1
                if cnt[v] >= n:
                    return None   # 음수 사이클
                if not in_queue[v]:
                    dq.append(v)
                    in_queue[v] = True
    return dist
```

## 언제 쓰는가

- 음수 간선이 있는 그래프
- 음수 사이클 탐지
- "**정확히 K개 간선 사용**" 제약 최단경로: V 대신 K번만 돌리면 됨

## 함정

- `dist[u] == INF`인 정점의 간선을 완화하면 오버플로·오염 → **반드시** `dist[u] != INF` 체크.
- 시간복잡도 O(V·E)는 V=10³, E=10⁴ 정도가 한계. 더 크면 다른 알고리즘 검토.

## 관련

음수 없으면 [[Dijkstra]]가 빠름. 모든 쌍이 필요하면 [[Floyd-Warshall]](O(V³)). DAG에서 최단경로는 [[위상 정렬]]로 O(V+E).

## 실행 예시

```python
def bellman_ford(n, edges, src):
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    for i in range(n):
        updated = False
        for u, v, w in edges:
            if dist[u] != INF and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                updated = True
                if i == n - 1:
                    return dist, True
        if not updated:
            break
    return dist, False

# --- 음수 간선 있지만 사이클 없음 ---
edges = [(0, 1, 4), (0, 2, 5), (1, 2, -3), (2, 3, 4), (1, 3, 5)]
dist, neg = bellman_ford(4, edges, 0)
print("dist:", dist)                 # [0, 4, 1, 5]
print("negative cycle:", neg)        # False

# --- 음수 사이클 존재 ---
edges_neg = [(0, 1, 1), (1, 2, -1), (2, 0, -1)]
dist, neg = bellman_ford(3, edges_neg, 0)
print("negative cycle:", neg)        # True
```

