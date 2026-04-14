---
tags: [코테, DP, 비트마스크]
---

# Bitmask DP

## 개요

부분집합을 **비트로 표현**하는 DP. N이 작을 때(보통 N ≤ 20) "어떤 원소들을 이미 처리했는가"를 상태로 압축 — 2^N × N 공간.

TSP(외판원), 집합 분할, 할당 문제의 표준 해법.

## 비트 연산 치트시트

```python
mask | (1 << i)         # i번째 비트 켜기
mask & ~(1 << i)        # i번째 비트 끄기
mask & (1 << i)         # i번째 비트 확인 (0 또는 2^i)
(mask >> i) & 1         # i번째 비트 (0 또는 1)
bin(mask).count('1')    # 켜진 비트 수
mask.bit_count()        # Python 3.10+, 위와 동일
```

부분집합 순회 (subset enumeration):

```python
sub = mask
while sub > 0:
    # sub는 mask의 부분집합
    sub = (sub - 1) & mask
# 전체 복잡도는 3^N (각 원소가 in-sub / in-mask-not-sub / out 3가지)
```

## TSP (외판원)

N개 도시 모두 방문 + 출발지 복귀 최소 비용. `dp[mask][u]` = mask의 도시들을 방문했고 마지막이 u일 때 최소 비용.

```python
def tsp(dist):
    """ dist: N x N 비용 행렬, 출발지 0 """
    n = len(dist)
    INF = float('inf')
    dp = [[INF] * n for _ in range(1 << n)]
    dp[1][0] = 0    # 0번 도시 방문, 현재 위치 0

    for mask in range(1 << n):
        for u in range(n):
            if dp[mask][u] == INF: continue
            if not (mask >> u) & 1: continue
            for v in range(n):
                if (mask >> v) & 1: continue
                nm = mask | (1 << v)
                if dp[mask][u] + dist[u][v] < dp[nm][v]:
                    dp[nm][v] = dp[mask][u] + dist[u][v]

    full = (1 << n) - 1
    return min(dp[full][u] + dist[u][0] for u in range(1, n))
```

## 할당 문제 (Assignment)

N명에게 N개 작업 배정, 비용 최소. `dp[mask]` = mask 작업들이 i명에게 할당된 최소 비용 (i = popcount(mask)).

```python
def assignment(cost):
    n = len(cost)
    INF = float('inf')
    dp = [INF] * (1 << n)
    dp[0] = 0
    for mask in range(1 << n):
        if dp[mask] == INF: continue
        i = bin(mask).count('1')
        if i == n: continue
        for j in range(n):
            if not (mask >> j) & 1:
                nm = mask | (1 << j)
                if dp[mask] + cost[i][j] < dp[nm]:
                    dp[nm] = dp[mask] + cost[i][j]
    return dp[(1 << n) - 1]
```

## 집합 분할 DP (sum-over-subsets)

"각 mask에 대해 모든 부분집합 합"을 O(N · 2^N)에 계산. 외판원보다 고급 테크닉.

```python
def sos(f, n):
    """ f[S] 값을, 각 S에 대해 f[subset of S] 합으로 변환 """
    dp = f[:]
    for i in range(n):
        for mask in range(1 << n):
            if (mask >> i) & 1:
                dp[mask] += dp[mask ^ (1 << i)]
    return dp
```

## Traveling Salesman Variants

- **시작점 자유**: `dp[mask][u]`의 답을 모든 u에 대해.
- **경로 복원**: `parent[mask][u]`를 기록.

## 함정

- N = 20이면 `2^20 = 10⁶` 상태, N × 2^N = 2×10⁷ 전이. Python에선 tight. PyPy 권장 경계.
- 비트마스크가 해시 키일 때 정수는 O(1)이지만, tuple로 감싸면 느려짐 — 그냥 int 사용.
- `1 << n` 대신 `2**n`은 어차피 같지만 시프트가 관습.

## 관련

[[백트래킹]]과 유사한 "모든 선택 탐색"이지만, DP는 **중복 상태를 캐싱**해 지수를 다항으로 (2^N으로) 줄임. [[DP 패턴]]의 상태 설계 장에서 "집합" 상태 = 비트마스크.

## 실행 예시

```python
# --- TSP ---
def tsp(dist):
    n = len(dist)
    INF = float('inf')
    dp = [[INF] * n for _ in range(1 << n)]
    dp[1][0] = 0
    for mask in range(1 << n):
        for u in range(n):
            if dp[mask][u] == INF: continue
            if not (mask >> u) & 1: continue
            for v in range(n):
                if (mask >> v) & 1: continue
                nm = mask | (1 << v)
                if dp[mask][u] + dist[u][v] < dp[nm][v]:
                    dp[nm][v] = dp[mask][u] + dist[u][v]
    full = (1 << n) - 1
    return min(dp[full][u] + dist[u][0] for u in range(1, n))

dist = [
    [0, 10, 15, 20],
    [10, 0, 35, 25],
    [15, 35, 0, 30],
    [20, 25, 30, 0],
]
print("TSP 최소 비용:", tsp(dist))   # 80 (0->1->3->2->0)

# --- 할당 문제 ---
def assignment(cost):
    n = len(cost)
    INF = float('inf')
    dp = [INF] * (1 << n)
    dp[0] = 0
    for mask in range(1 << n):
        if dp[mask] == INF: continue
        i = bin(mask).count('1')
        if i == n: continue
        for j in range(n):
            if not (mask >> j) & 1:
                nm = mask | (1 << j)
                if dp[mask] + cost[i][j] < dp[nm]:
                    dp[nm] = dp[mask] + cost[i][j]
    return dp[(1 << n) - 1]

cost = [[9, 2, 7, 8], [6, 4, 3, 7], [5, 8, 1, 8], [7, 6, 9, 4]]
print("할당 최소:", assignment(cost))   # 13 (2+3+4+... optimal)

# --- 부분집합 순회 데모 ---
mask = 0b1011      # 원소 {0, 1, 3}
sub = mask
subsets = []
while sub > 0:
    subsets.append(bin(sub))
    sub = (sub - 1) & mask
subsets.append(bin(0))
print("부분집합 순회:", subsets)
# ['0b1011', '0b1010', '0b1001', '0b1000', '0b11', '0b10', '0b1', '0b0']
```

