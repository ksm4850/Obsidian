---
tags: [코테, 그래프, 최단경로]
---

# Floyd-Warshall

## 개요

**모든 쌍 최단경로** O(V³). V ≤ 500 정도까지 실용. 음수 간선 허용(음수 사이클 없을 때). 구현 3중 for문으로 압도적으로 짧음.

## 핵심 점화식

`dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`. **k를 맨 바깥 루프**로 두는 것이 핵심 — "정점 0..k까지 경유 허용" 의미.

## Python 구현

```python
def floyd_warshall(n, edges):
    INF = float('inf')
    dist = [[INF] * n for _ in range(n)]
    for i in range(n):
        dist[i][i] = 0
    for u, v, w in edges:
        dist[u][v] = min(dist[u][v], w)

    for k in range(n):
        for i in range(n):
            if dist[i][k] == INF:
                continue
            dik = dist[i][k]
            for j in range(n):
                if dik + dist[k][j] < dist[i][j]:
                    dist[i][j] = dik + dist[k][j]
    return dist
```

중간에 `dist[i][k] == INF` 체크로 3중 for 내부를 자주 스킵 → Python에선 체감 속도 크게 달라짐.

## 음수 사이클 탐지

`dist[i][i] < 0`인 i가 존재하면 음수 사이클 존재.

## 응용 1: 도달 가능성 (Transitive Closure)

거리 대신 bool 행렬로 OR 연산.

```python
reach = [[False]*n for _ in range(n)]
for i in range(n): reach[i][i] = True
for u, v in edges: reach[u][v] = True
for k in range(n):
    for i in range(n):
        if reach[i][k]:
            for j in range(n):
                if reach[k][j]:
                    reach[i][j] = True
```

## 응용 2: 경로 복원

`next_hop[i][j]`를 갱신 때마다 기록.

```python
nxt = [[j if dist[i][j] != INF else -1 for j in range(n)] for i in range(n)]
# 갱신부에서:
if dik + dist[k][j] < dist[i][j]:
    dist[i][j] = dik + dist[k][j]
    nxt[i][j] = nxt[i][k]

def path(i, j):
    if nxt[i][j] == -1: return []
    res = [i]
    while i != j:
        i = nxt[i][j]
        res.append(i)
    return res
```

## 응용 3: 최소 사이클 길이

각 `i`에 대해 `min(dist[i][j] + w(j, i))`. 간선을 건너고 다시 돌아오는 최소 비용.

## 함정

- **k가 최외곽** 아니면 답이 틀림. 내부 두 for의 순서는 상관없지만 k는 반드시 바깥.
- V=1000이면 10⁹ 연산 → Python에서 TLE. PyPy 또는 numpy로 벡터화 고려.

## 관련

단일 출발점만 필요하면 [[Dijkstra]]. 음수 간선이면 [[Bellman-Ford]]. 희소 그래프면 [[Dijkstra]] V번 돌리기가 더 빠름.

## 실행 예시

```python
def floyd_warshall(n, edges):
    INF = float('inf')
    dist = [[INF] * n for _ in range(n)]
    for i in range(n):
        dist[i][i] = 0
    for u, v, w in edges:
        dist[u][v] = min(dist[u][v], w)
    for k in range(n):
        for i in range(n):
            if dist[i][k] == INF: continue
            dik = dist[i][k]
            for j in range(n):
                if dik + dist[k][j] < dist[i][j]:
                    dist[i][j] = dik + dist[k][j]
    return dist

edges = [
    (0, 1, 3), (0, 2, 8), (0, 4, -4),
    (1, 3, 1), (1, 4, 7),
    (2, 1, 4),
    (3, 0, 2), (3, 2, -5),
    (4, 3, 6),
]
dist = floyd_warshall(5, edges)
for row in dist:
    print(row)
# 0:  [0, 1, -3, 2, -4]
# 1:  [3, 0, -4, 1, -1]
# 2:  [7, 4, 0, 5, 3]
# 3:  [2, -1, -5, 0, -2]
# 4:  [8, 5, 1, 6, 0]
```

