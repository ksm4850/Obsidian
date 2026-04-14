---
tags: [코테, 그래프, 탐색]
---

# BFS / DFS 응용

## 개요

- **BFS**: 큐 기반, 간선 가중치가 모두 1일 때 **최단 거리**를 줌. 레벨 단위 탐색.
- **DFS**: 스택(또는 재귀) 기반, 경로 탐색·위상 정렬·SCC·[[트리 DP]]의 뼈대.

중급 이상에선 "단순 탐색"을 넘어 **격자 + 상태**, **멀티소스**, **0-1 BFS** 같은 변형이 관건.

## 격자 BFS — 최단 거리

```python
from collections import deque

def grid_bfs(grid, sr, sc):
    R, C = len(grid), len(grid[0])
    dist = [[-1] * C for _ in range(R)]
    dist[sr][sc] = 0
    dq = deque([(sr, sc)])
    while dq:
        r, c = dq.popleft()
        for dr, dc in ((-1, 0), (1, 0), (0, -1), (0, 1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C and grid[nr][nc] != '#' and dist[nr][nc] == -1:
                dist[nr][nc] = dist[r][c] + 1
                dq.append((nr, nc))
    return dist
```

## 멀티소스 BFS

여러 시작점에서 동시에 퍼져나갈 때 — 큐에 **모든 소스를 한 번에 넣고** 시작. "모든 불에서 가장 가까운 거리" 같은 문제.

```python
dq = deque()
for r, c in sources:
    dist[r][c] = 0
    dq.append((r, c))
# 이후는 동일
```

## 0-1 BFS

간선 가중치가 0 또는 1만 있을 때 [[Dijkstra]] 대신 **덱**으로 O(V+E).

```python
def zero_one_bfs(graph, src, n):
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    dq = deque([src])
    while dq:
        u = dq.popleft()
        for v, w in graph[u]:   # w ∈ {0, 1}
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                if w == 0:
                    dq.appendleft(v)
                else:
                    dq.append(v)
    return dist
```

## BFS + 상태 (비트마스크 / 여분 차원)

"열쇠 K개를 모으며 최단 경로" 같은 문제는 `(r, c, mask)`를 상태로.

```python
def bfs_with_keys(grid, sr, sc, num_keys):
    # visited[r][c][mask]
    ...
    state = (sr, sc, 0)
    dq = deque([(state, 0)])
    seen = {state}
    while dq:
        (r, c, mask), d = dq.popleft()
        if mask == (1 << num_keys) - 1:
            return d
        for dr, dc in ((-1,0),(1,0),(0,-1),(0,1)):
            nr, nc = r+dr, c+dc
            if not (0 <= nr < R and 0 <= nc < C): continue
            cell = grid[nr][nc]
            if cell == '#': continue
            nmask = mask
            if 'a' <= cell <= 'f':
                nmask |= 1 << (ord(cell) - ord('a'))
            if 'A' <= cell <= 'F' and not (mask >> (ord(cell)-ord('A'))) & 1:
                continue   # 열쇠 없음
            ns = (nr, nc, nmask)
            if ns not in seen:
                seen.add(ns)
                dq.append((ns, d + 1))
    return -1
```

## DFS 반복 (스택오버플로 회피)

Python 재귀 깊이 기본 1000. 깊은 그래프는 반복 버전으로.

```python
def dfs_iter(graph, start):
    visited = {start}
    stack = [start]
    order = []
    while stack:
        u = stack.pop()
        order.append(u)
        for v in graph[u]:
            if v not in visited:
                visited.add(v)
                stack.append(v)
    return order
```

또는 맨 위에 `sys.setrecursionlimit(10**6)`.

## DFS로 사이클 감지 (방향 그래프)

```python
WHITE, GRAY, BLACK = 0, 1, 2

def has_cycle_directed(n, graph):
    color = [WHITE] * n
    def dfs(u):
        color[u] = GRAY
        for v in graph[u]:
            if color[v] == GRAY:
                return True
            if color[v] == WHITE and dfs(v):
                return True
        color[u] = BLACK
        return False
    return any(dfs(i) for i in range(n) if color[i] == WHITE)
```

무방향이면 부모 노드 제외만으로 판정. 동적으로 간선이 추가되면 [[Union-Find]] 활용.

## 함정

- 격자 BFS에서 **시작 시 dist=0 세팅**을 빼먹고 큐에만 넣으면 중복 방문.
- 재귀 DFS로 N=10⁵ 깊이면 터짐 → 반복 또는 `sys.setrecursionlimit`.

## 관련

가중치가 있으면 [[Dijkstra]]. 음수 가중치면 [[Bellman-Ford]]. DAG 순서 문제는 [[위상 정렬]]. 강결합은 [[SCC]].

## 실행 예시

```python
from collections import deque

# --- 격자 BFS ---
def grid_bfs(grid, sr, sc):
    R, C = len(grid), len(grid[0])
    dist = [[-1] * C for _ in range(R)]
    dist[sr][sc] = 0
    dq = deque([(sr, sc)])
    while dq:
        r, c = dq.popleft()
        for dr, dc in ((-1, 0), (1, 0), (0, -1), (0, 1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C and grid[nr][nc] != '#' and dist[nr][nc] == -1:
                dist[nr][nc] = dist[r][c] + 1
                dq.append((nr, nc))
    return dist

grid = [
    list("....."),
    list(".###."),
    list("....."),
]
dist = grid_bfs(grid, 0, 0)
for row in dist:
    print(row)
# [0, 1, 2, 3, 4]
# [1, -1, -1, -1, 5]
# [2, 3, 4, 5, 6]

# --- 0-1 BFS ---
def zero_one_bfs(graph, src, n):
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    dq = deque([src])
    while dq:
        u = dq.popleft()
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                if w == 0: dq.appendleft(v)
                else: dq.append(v)
    return dist

graph = {0: [(1, 1), (2, 0)], 1: [(3, 1)], 2: [(1, 0), (3, 1)], 3: []}
print("0-1 BFS dist:", zero_one_bfs(graph, 0, 4))   # [0, 0, 0, 1]

# --- 방향 그래프 사이클 감지 ---
def has_cycle_directed(n, graph):
    WHITE, GRAY, BLACK = 0, 1, 2
    color = [WHITE] * n
    def dfs(u):
        color[u] = GRAY
        for v in graph[u]:
            if color[v] == GRAY: return True
            if color[v] == WHITE and dfs(v): return True
        color[u] = BLACK
        return False
    return any(dfs(i) for i in range(n) if color[i] == WHITE)

print("cycle (있음):", has_cycle_directed(3, {0:[1], 1:[2], 2:[0]}))  # True
print("cycle (없음):", has_cycle_directed(3, {0:[1], 1:[2], 2:[]}))   # False
```

