---
tags: [코테, 그래프, 강결합]
---

# SCC (강결합 컴포넌트)

## 개요

**방향 그래프**에서 서로 도달 가능한 정점끼리의 최대 부분 그래프. 모든 SCC를 하나의 정점으로 축약하면 DAG가 됨(응집 그래프, condensation).

대표 알고리즘:
- **Tarjan**: DFS 한 번, O(V + E), 구현 약간 섬세
- **Kosaraju**: DFS 두 번(원본 + 역방향), O(V + E), 직관적

## Tarjan

각 정점의 **방문 순서**(`disc`)와 스택을 통해 도달 가능한 **최소 순서**(`low`)를 추적. `disc[u] == low[u]`이면 u가 SCC의 루트 → 스택에서 u까지 pop.

```python
import sys
sys.setrecursionlimit(10**6)

def tarjan_scc(n, graph):
    disc = [-1] * n
    low = [0] * n
    on_stack = [False] * n
    stack = []
    sccs = []
    timer = [0]

    def dfs(u):
        disc[u] = low[u] = timer[0]
        timer[0] += 1
        stack.append(u)
        on_stack[u] = True
        for v in graph[u]:
            if disc[v] == -1:
                dfs(v)
                low[u] = min(low[u], low[v])
            elif on_stack[v]:
                low[u] = min(low[u], disc[v])
        if low[u] == disc[u]:
            comp = []
            while True:
                x = stack.pop()
                on_stack[x] = False
                comp.append(x)
                if x == u: break
            sccs.append(comp)

    for i in range(n):
        if disc[i] == -1:
            dfs(i)
    return sccs
```

## Kosaraju

1. 원본 그래프 DFS → 종료 순서 스택
2. 간선 **뒤집은** 그래프에서 스택 top부터 DFS → 한 번에 방문되는 정점 집합이 하나의 SCC

```python
def kosaraju(n, graph):
    rev = [[] for _ in range(n)]
    for u in range(n):
        for v in graph[u]:
            rev[v].append(u)

    visited = [False] * n
    order = []
    def dfs1(u):
        stack = [(u, iter(graph[u]))]
        visited[u] = True
        while stack:
            v, it = stack[-1]
            found = False
            for w in it:
                if not visited[w]:
                    visited[w] = True
                    stack.append((w, iter(graph[w])))
                    found = True
                    break
            if not found:
                order.append(v)
                stack.pop()

    for i in range(n):
        if not visited[i]:
            dfs1(i)

    comp = [-1] * n
    def dfs2(u, c):
        stack = [u]
        comp[u] = c
        while stack:
            v = stack.pop()
            for w in rev[v]:
                if comp[w] == -1:
                    comp[w] = c
                    stack.append(w)

    c = 0
    for u in reversed(order):
        if comp[u] == -1:
            dfs2(u, c)
            c += 1
    sccs = [[] for _ in range(c)]
    for i, ci in enumerate(comp):
        sccs[ci].append(i)
    return sccs
```

## 응용 1: 2-SAT

부울식 `(a ∨ b) ∧ ...`를 **함의 그래프**로 변환. 각 절 `a ∨ b`는 `¬a → b`, `¬b → a` 두 간선. SCC 계산 후 **같은 SCC에 x와 ¬x가 있으면 UNSAT**. 그렇지 않으면 위상 역순으로 값 지정.

```python
# 변수 i에 대해 정점 2i(참), 2i+1(거짓)
# add_clause(a, b): a가 거짓이면 b는 참이어야 → (a의 부정) → b
```

## 응용 2: DAG 축약 후 DP

SCC 번호로 정점을 병합, 간선 재구성 → [[위상 정렬]] 하고 [[DP 패턴|DP]] 돌림. "한 번 방문 가능한 최장 경로" 등.

## 함정

- Tarjan의 `elif on_stack[v]`가 핵심. `disc[v] != -1`만 체크하면 오답.
- 재귀 깊이 10⁵ 이상이면 반복 스택 버전으로. 위 Kosaraju dfs1은 이미 반복.

## 관련

[[위상 정렬]]은 사이클 없는 DAG 전용. SCC 축약 후 위상 정렬로 확장. [[Union-Find]]는 무방향 연결성만.

## 실행 예시

```python
import sys
sys.setrecursionlimit(10**6)

def tarjan_scc(n, graph):
    disc = [-1] * n
    low = [0] * n
    on_stack = [False] * n
    stack = []
    sccs = []
    timer = [0]
    def dfs(u):
        disc[u] = low[u] = timer[0]
        timer[0] += 1
        stack.append(u)
        on_stack[u] = True
        for v in graph[u]:
            if disc[v] == -1:
                dfs(v)
                low[u] = min(low[u], low[v])
            elif on_stack[v]:
                low[u] = min(low[u], disc[v])
        if low[u] == disc[u]:
            comp = []
            while True:
                x = stack.pop()
                on_stack[x] = False
                comp.append(x)
                if x == u: break
            sccs.append(comp)
    for i in range(n):
        if disc[i] == -1:
            dfs(i)
    return sccs

# 그래프:  0→1→2→0 (SCC1),  1→3, 3→4→3 (SCC2),  4→5 (SCC3: {5})
graph = {
    0: [1],
    1: [2, 3],
    2: [0],
    3: [4],
    4: [3, 5],
    5: [],
}
sccs = tarjan_scc(6, graph)
print("SCCs:", sccs)
# 예: [[5], [3, 4], [0, 1, 2]] (순서는 DFS 탐색 순서에 따라)
print("개수:", len(sccs))   # 3
```

