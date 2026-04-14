---
tags: [코테, DP, 트리]
---

# 트리 DP

## 개요

트리 구조에서의 DP. 일반 그래프 DP와 달리 **사이클 없음** → 자식 서브트리의 답을 합쳐 부모로 올리면 됨. 보통 O(N).

고급 기법:
- **기본**: 루트 고정, DFS로 bottom-up
- **리루팅(Rerooting)**: 모든 정점을 루트로 했을 때의 답을 한 번에 계산 — O(N)

## 기본: 서브트리 DP

가장 흔한 예 — **서브트리의 정점 수**, **서브트리 내 경로 길이** 등.

```python
import sys
sys.setrecursionlimit(10**6)

def subtree_size(n, graph, root=0):
    size = [1] * n
    def dfs(u, parent):
        for v in graph[u]:
            if v != parent:
                dfs(v, u)
                size[u] += size[v]
    dfs(root, -1)
    return size
```

## 트리 독립집합 (이웃 두 개를 동시에 고르지 않기)

각 정점 u에 대해 `dp[u][0]` = u 미선택, `dp[u][1]` = u 선택 시 서브트리의 최대 독립집합 크기.

```python
def max_indep_set(n, graph, weight, root=0):
    dp = [[0, 0] for _ in range(n)]
    def dfs(u, parent):
        dp[u][1] = weight[u]
        for v in graph[u]:
            if v != parent:
                dfs(v, u)
                dp[u][0] += max(dp[v][0], dp[v][1])
                dp[u][1] += dp[v][0]
    dfs(root, -1)
    return max(dp[root])
```

## 트리 내 최장 경로 (Diameter)

각 정점을 거쳐가는 경로의 두 가장 긴 가지를 합친 것 = 그 정점 기준 최장. 전체 최대가 지름.

```python
def tree_diameter(n, graph):
    best = [0]
    def dfs(u, parent):
        top1 = top2 = 0
        for v in graph[u]:
            if v != parent:
                d = dfs(v, u) + 1
                if d > top1:
                    top2 = top1; top1 = d
                elif d > top2:
                    top2 = d
        best[0] = max(best[0], top1 + top2)
        return top1
    dfs(0, -1)
    return best[0]
```

## 리루팅 (Rerooting) — 모든 루트 후보 동시 처리

"각 정점을 루트로 했을 때 트리의 어떤 값"을 구하는 유형. 첫 DFS로 고정 루트 기준 계산, 두 번째 DFS에서 부모 쪽 기여를 전달.

예: **각 정점에서 모든 다른 정점까지의 거리 합**.

```python
def sum_distances(n, edges):
    graph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    size = [1] * n
    ans = [0] * n

    # 1차: 루트 0 기준 서브트리 크기와 루트에서의 거리 합
    def dfs1(u, parent):
        for v in graph[u]:
            if v != parent:
                dfs1(v, u)
                size[u] += size[v]
                ans[u] += ans[v] + size[v]
    dfs1(0, -1)

    # 2차: 부모 쪽으로 리루팅
    def dfs2(u, parent):
        for v in graph[u]:
            if v != parent:
                # v가 루트가 되면, v쪽으로 size[v]개가 1 가까워지고
                # 나머지 (n - size[v])개가 1 멀어짐
                ans[v] = ans[u] - size[v] + (n - size[v])
                dfs2(v, u)
    dfs2(0, -1)

    return ans
```

이 패턴(1차로 정보 축적, 2차로 전파)은 거의 모든 리루팅 문제의 틀.

## 트리 DP + Knapsack (그룹화)

서브트리 크기를 "배낭 용량"처럼 쓰는 유형. "정점 K개 골라 인접 관계 만족"류.

```python
# dp[u][k] = u의 서브트리에서 k개 선택 시 최적
def dfs(u, parent):
    dp[u] = [0] * (K + 1)
    dp[u][1] = weight[u]
    for v in graph[u]:
        if v != parent:
            dfs(v, u)
            # 결합
            new = [-INF] * (K + 1)
            for i in range(K + 1):
                for j in range(K + 1 - i):
                    if dp[u][i] + dp[v][j] > new[i + j]:
                        new[i + j] = dp[u][i] + dp[v][j]
            dp[u] = new
```

양측 서브트리 크기만큼만 돌리면 전체 O(N²) (증명: 임의의 두 정점 쌍이 LCA에서 정확히 한 번 매칭).

## 함정

- 재귀 깊이: N = 10⁵면 반드시 `sys.setrecursionlimit(10**6)`. 그래도 Python recursion은 느림 — 반복 DFS가 안전.
- 무방향 간선이므로 **parent 전달** 필수 (visited 배열 대체).

## 관련

[[BFS DFS 응용]]의 DFS 템플릿과 [[DP 패턴]]의 상태 설계 결합. 리루팅은 "사전+사후 DP"의 트리 버전.

## 실행 예시

```python
import sys
sys.setrecursionlimit(10**6)

# 트리: 0 -- 1 -- 2
#        \     \-- 3
#         4 -- 5
# edges: (0,1),(1,2),(1,3),(0,4),(4,5)
n = 6
edges = [(0,1),(1,2),(1,3),(0,4),(4,5)]
graph = [[] for _ in range(n)]
for u, v in edges:
    graph[u].append(v); graph[v].append(u)

# --- 서브트리 크기 ---
def subtree_size(n, graph, root=0):
    size = [1] * n
    def dfs(u, parent):
        for v in graph[u]:
            if v != parent:
                dfs(v, u)
                size[u] += size[v]
    dfs(root, -1)
    return size

print("subtree sizes:", subtree_size(n, graph))   # [6, 3, 1, 1, 2, 1]

# --- 트리 지름 ---
def tree_diameter(n, graph):
    best = [0]
    def dfs(u, parent):
        top1 = top2 = 0
        for v in graph[u]:
            if v != parent:
                d = dfs(v, u) + 1
                if d > top1: top2 = top1; top1 = d
                elif d > top2: top2 = d
        best[0] = max(best[0], top1 + top2)
        return top1
    dfs(0, -1)
    return best[0]

print("트리 지름:", tree_diameter(n, graph))   # 4 (2->1->0->4->5)

# --- 거리 합 리루팅 ---
def sum_distances(n, edges):
    graph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v); graph[v].append(u)
    size = [1] * n
    ans = [0] * n
    def dfs1(u, parent):
        for v in graph[u]:
            if v != parent:
                dfs1(v, u)
                size[u] += size[v]
                ans[u] += ans[v] + size[v]
    dfs1(0, -1)
    def dfs2(u, parent):
        for v in graph[u]:
            if v != parent:
                ans[v] = ans[u] - size[v] + (n - size[v])
                dfs2(v, u)
    dfs2(0, -1)
    return ans

print("각 정점에서의 거리 합:", sum_distances(n, edges))
# [8, 7, 11, 11, 8, 12] 류 (트리 구조에 따라)
```

