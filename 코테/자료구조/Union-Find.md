---
tags: [코테, 자료구조, 그래프]
---

# Union-Find (Disjoint Set Union)

## 개요

원소들을 **서로 소 집합**으로 관리. 두 원소가 같은 집합인지(`find`), 두 집합을 합치는(`union`) 연산을 거의 O(1)에 처리. **경로 압축** + **union by rank**를 함께 쓰면 아커만 역함수 O(α(N)) ≈ 상수.

[[MST]]의 Kruskal, 사이클 감지, 오프라인 쿼리 처리의 핵심.

## 핵심 구현

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.size = [1] * n  # 집합 크기 필요시

    def find(self, x):
        # 경로 압축: 재귀보다 반복이 스택 안전
        root = x
        while self.parent[root] != root:
            root = self.parent[root]
        while self.parent[x] != root:
            self.parent[x], x = root, self.parent[x]
        return root

    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False
        # union by rank
        if self.rank[ra] < self.rank[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        if self.rank[ra] == self.rank[rb]:
            self.rank[ra] += 1
        return True
```

## 응용 1: 사이클 감지

무방향 그래프에서 에지 추가 시 양 끝이 이미 같은 집합이면 사이클.

```python
def has_cycle(n, edges):
    dsu = DSU(n)
    for u, v in edges:
        if not dsu.union(u, v):
            return True
    return False
```

## 응용 2: 컴포넌트 수

```python
def count_components(n, edges):
    dsu = DSU(n)
    for u, v in edges:
        dsu.union(u, v)
    return len({dsu.find(i) for i in range(n)})
```

## 응용 3: 오프라인 연결성

쿼리를 **역순**으로 처리하면 간선 삭제가 간선 추가가 됨. "삭제 후 연결성" 문제에서 유용.

## 가중 Union-Find (응용)

각 원소가 루트까지의 "거리"를 갖는 변형. "A는 B보다 3 큼"처럼 관계 추적에 쓰임.

```python
class WeightedDSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.diff = [0] * n  # 부모와의 차이

    def find(self, x):
        if self.parent[x] == x:
            return x
        root = self.find(self.parent[x])
        self.diff[x] += self.diff[self.parent[x]]
        self.parent[x] = root
        return root

    def union(self, a, b, w):  # b - a = w
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return self.diff[b] - self.diff[a] == w
        self.parent[rb] = ra
        self.diff[rb] = self.diff[a] + w - self.diff[b]
        return True
```

## 함정

- 경로 압축만 있어도 실전은 충분 빠르지만, 이론적 최적은 rank까지 필요.
- 재귀 `find`는 N=10⁶에서 스택오버플로. 반복문 권장 또는 `sys.setrecursionlimit(10**7)`.

## 관련

[[MST]]의 Kruskal에서 "간선을 추가하면 사이클인가?" 판정에 사용. [[BFS DFS 응용]]과 비교: 정적 그래프의 컴포넌트는 DFS로도 되지만, **간선이 동적으로 추가**될 때는 DSU가 압도적.

## 실행 예시

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.size = [1] * n
    def find(self, x):
        root = x
        while self.parent[root] != root:
            root = self.parent[root]
        while self.parent[x] != root:
            self.parent[x], x = root, self.parent[x]
        return root
    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb: return False
        if self.rank[ra] < self.rank[rb]: ra, rb = rb, ra
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        if self.rank[ra] == self.rank[rb]: self.rank[ra] += 1
        return True

# --- 기본 동작 ---
dsu = DSU(6)
dsu.union(0, 1)
dsu.union(1, 2)
dsu.union(3, 4)
print("0과 2 연결?", dsu.find(0) == dsu.find(2))   # True
print("0과 3 연결?", dsu.find(0) == dsu.find(3))   # False
print("0이 속한 집합 크기:", dsu.size[dsu.find(0)])  # 3

# --- 사이클 감지 ---
def has_cycle(n, edges):
    dsu = DSU(n)
    for u, v in edges:
        if not dsu.union(u, v):
            return True
    return False

print("사이클(있음):", has_cycle(4, [(0,1),(1,2),(2,0)]))   # True
print("사이클(없음):", has_cycle(4, [(0,1),(1,2),(2,3)]))   # False

# --- 컴포넌트 수 ---
def count_components(n, edges):
    dsu = DSU(n)
    for u, v in edges:
        dsu.union(u, v)
    return len({dsu.find(i) for i in range(n)})

print("컴포넌트:", count_components(6, [(0,1),(1,2),(3,4)]))   # 3
```

