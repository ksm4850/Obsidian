---
tags: [코테, 그래프, MST]
---

# MST (최소 신장 트리)

## 개요

연결 무방향 가중 그래프에서 **모든 정점을 연결**하면서 간선 가중치 합이 최소인 트리.

- **Kruskal**: 간선 정렬 + [[Union-Find]]. O(E log E). 희소 그래프에 유리.
- **Prim**: [[Heap]] 기반 확장. O(E log V). 밀집 그래프에 유리.

## Kruskal

```python
def kruskal(n, edges):
    """
    edges: [(w, u, v), ...]
    """
    edges.sort()
    parent = list(range(n))
    rank = [0] * n

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]   # path compression
            x = parent[x]
        return x

    def union(a, b):
        ra, rb = find(a), find(b)
        if ra == rb: return False
        if rank[ra] < rank[rb]: ra, rb = rb, ra
        parent[rb] = ra
        if rank[ra] == rank[rb]: rank[ra] += 1
        return True

    total = 0
    used = 0
    for w, u, v in edges:
        if union(u, v):
            total += w
            used += 1
            if used == n - 1:
                break
    return total if used == n - 1 else None   # 연결 불가
```

## Prim

시작 정점에서 이웃 간선들을 힙에 넣고, 가장 가벼운 간선으로 확장.

```python
import heapq

def prim(n, graph, start=0):
    visited = [False] * n
    visited[start] = True
    pq = [(w, v) for v, w in graph[start]]
    heapq.heapify(pq)
    total = 0
    count = 1
    while pq and count < n:
        w, u = heapq.heappop(pq)
        if visited[u]: continue
        visited[u] = True
        total += w
        count += 1
        for v, nw in graph[u]:
            if not visited[v]:
                heapq.heappush(pq, (nw, v))
    return total if count == n else None
```

## 언제 어느 쪽?

| 상황 | 추천 |
|------|------|
| E ≈ V (희소) | Kruskal |
| E ≈ V² (밀집) | Prim (인접행렬 O(V²) 버전) |
| 오프라인 간선 목록 주어짐 | Kruskal |
| 동적으로 확장 | Prim |

## 응용 1: 두 번째 MST

Kruskal로 MST 구성 후, **MST에 속하지 않은 간선 각각을** 추가했을 때 생기는 사이클에서 **MST 내 최대 간선**을 제거하면 교환 비용. 최소 교환이 정답.

MST 경로 최댓값을 빠르게 구하려면 트리에 LCA + 구간 최대 전처리.

## 응용 2: 간선 가중치 제한된 MST

"각 간선은 최대 K번만 사용" 등 특수 조건은 보통 MST 핵심이 변형되므로, Kruskal 순회 중 조건 체크 삽입.

## 응용 3: 네트워크 디자인

"필수 연결 정점 집합만 이어라" → [[BFS DFS 응용]]로 불필요 가지 제거하며 MST 축소.

## 함정

- 그래프가 연결되지 않으면 MST 자체가 없음 → `used == n-1` 판정 필수.
- Kruskal에서 `find` 경로 압축 없으면 최악 O(N) — 꼭 포함.

## 관련

[[Union-Find]]의 대표 응용. Prim의 우선순위 큐는 [[Heap]]. DAG에서 최소 가중치 간선 집합은 위상 정렬 + DP로 다른 문제.

## 실행 예시

```python
import heapq

# --- Kruskal ---
def kruskal(n, edges):
    edges.sort()
    parent = list(range(n))
    rank = [0] * n
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x
    def union(a, b):
        ra, rb = find(a), find(b)
        if ra == rb: return False
        if rank[ra] < rank[rb]: ra, rb = rb, ra
        parent[rb] = ra
        if rank[ra] == rank[rb]: rank[ra] += 1
        return True
    total, used = 0, 0
    for w, u, v in edges:
        if union(u, v):
            total += w; used += 1
            if used == n - 1: break
    return total if used == n - 1 else None

# 그래프: 5개 정점, 7개 간선
edges = [(1,0,1),(2,0,2),(3,1,2),(4,1,3),(5,2,3),(6,2,4),(7,3,4)]
print("Kruskal MST:", kruskal(5, edges))   # 1+2+4+6 = 13 ? 실제론 1+2+4+6=13 확인 필요
# 실제 MST: (0-1,1),(0-2,2),(1-3,4),(2-4,6) = 13

# --- Prim ---
def prim(n, graph, start=0):
    visited = [False] * n
    visited[start] = True
    pq = [(w, v) for v, w in graph[start]]
    heapq.heapify(pq)
    total = 0
    count = 1
    while pq and count < n:
        w, u = heapq.heappop(pq)
        if visited[u]: continue
        visited[u] = True
        total += w
        count += 1
        for v, nw in graph[u]:
            if not visited[v]:
                heapq.heappush(pq, (nw, v))
    return total if count == n else None

# 동일 그래프 인접 리스트 형태
graph = {
    0: [(1, 1), (2, 2)],
    1: [(0, 1), (2, 3), (3, 4)],
    2: [(0, 2), (1, 3), (3, 5), (4, 6)],
    3: [(1, 4), (2, 5), (4, 7)],
    4: [(2, 6), (3, 7)],
}
print("Prim MST:", prim(5, graph))   # 13
```

