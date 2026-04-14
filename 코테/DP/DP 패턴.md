---
tags: [코테, DP]
---

# DP 패턴

## 개요

부분 문제의 답을 저장해 **중복 계산을 제거**. 요구 조건:
1. **최적 부분 구조** — 부분 문제의 최적해로 전체 최적해 구성
2. **중복되는 부분 문제** — 같은 상태가 여러 번 등장

두 방식:
- **Top-down (메모이제이션)**: 재귀 + 캐시. 설계가 자연스러움.
- **Bottom-up (타뷸레이션)**: 반복문. 상수가 작고 스택 걱정 없음.

## 점화식 설계 체크리스트

1. **상태를 무엇으로 두지?** — `dp[i]`가 "i번 원소까지 썼을 때 최댓값"인가, "정확히 i번을 쓴 최댓값"인가. 정의를 끝까지 일관.
2. **상태 전이는?** — 이전 상태에서 어떤 선택으로 현재에 오는가.
3. **초기값 / 기저 사례**
4. **답은 어느 상태?** — 전체 답이 `dp[n]`인지 `max(dp)`인지.

## Top-down 템플릿

```python
import sys
from functools import lru_cache
sys.setrecursionlimit(10**6)

@lru_cache(maxsize=None)
def solve(i, state):
    if base_case(i):
        return base_value
    best = worst_case
    for choice in choices:
        best = better(best, solve(next_i(i, choice), next_state(state, choice)) + cost(choice))
    return best
```

`lru_cache`는 편하지만 상태가 **해시 가능**해야 함. 튜플로 감쌀 것.

## Bottom-up 템플릿

```python
dp = [[INF] * M for _ in range(N)]
# 기저
for j in range(M): dp[0][j] = base(j)
# 전이
for i in range(1, N):
    for j in range(M):
        dp[i][j] = min(dp[i-1][j], dp[i-1][j-1] + cost)
return dp[N-1][?]
```

## 공간 최적화

`dp[i]`가 `dp[i-1]`만 참조하면 두 행만 유지. 또는 **역방향 순회**로 한 행만.

```python
# 0/1 Knapsack 공간 최적화
dp = [0] * (W + 1)
for w, v in items:
    for j in range(W, w - 1, -1):    # 역순 필수
        dp[j] = max(dp[j], dp[j-w] + v)
```

역순인 이유: 같은 아이템을 여러 번 쓰는 걸 막기 위해 **방금 갱신된 `dp[j-w]`를 다시 참조하지 않도록**.

## 자주 등장하는 상태 설계

| 문제 유형 | 상태 |
|----------|------|
| 순서가 중요한 수열 | `dp[i]` = i까지의 최적 |
| 2D 격자 | `dp[i][j]` |
| 부분집합 | `dp[mask]` — [[Bitmask DP]] |
| 구간 | `dp[l][r]` — 구간 DP |
| 트리 | `dp[v][state]` — [[트리 DP]] |
| 문자열 매칭 | `dp[i][j]` — [[LCS]], 편집거리 |

## 구간 DP 예시 (행렬 체인 곱셈)

```python
def matrix_chain(dims):
    n = len(dims) - 1
    dp = [[0]*n for _ in range(n)]
    for length in range(2, n+1):
        for i in range(n - length + 1):
            j = i + length - 1
            dp[i][j] = min(
                dp[i][k] + dp[k+1][j] + dims[i]*dims[k+1]*dims[j+1]
                for k in range(i, j)
            )
    return dp[0][n-1]
```

길이를 **바깥 루프**로 두는 게 포인트.

## 함정

- **재귀 깊이**: 10⁵ 이상이면 `sys.setrecursionlimit`. 그래도 느리면 bottom-up.
- **상태 누락**: 같은 함수 인자여도 의미가 다르면 캐시 오염.
- **부동소수점**: 확률 DP에서 비교 시 `abs(a-b) < 1e-9`.

## 관련

수열 DP의 대표: [[LIS]], [[LCS]]. 부분합: [[Knapsack]]. 상태 압축: [[Bitmask DP]]. 트리 구조: [[트리 DP]].

## 실행 예시

```python
from functools import lru_cache

# --- Top-down: 계단 오르기 (1 또는 2칸) ---
@lru_cache(maxsize=None)
def climb(n):
    if n <= 1: return 1
    return climb(n - 1) + climb(n - 2)

print("climb(10):", climb(10))   # 89

# --- Bottom-up: 0/1 Knapsack 공간 최적화 ---
def knapsack(items, W):
    dp = [0] * (W + 1)
    for w, v in items:
        for j in range(W, w - 1, -1):
            dp[j] = max(dp[j], dp[j-w] + v)
    return dp[W]

items = [(2, 3), (3, 4), (4, 5), (5, 6)]
print("knapsack (W=8):", knapsack(items, 8))   # 10

# --- 구간 DP: 행렬 체인 곱셈 ---
def matrix_chain(dims):
    n = len(dims) - 1
    dp = [[0]*n for _ in range(n)]
    for length in range(2, n+1):
        for i in range(n - length + 1):
            j = i + length - 1
            dp[i][j] = min(
                dp[i][k] + dp[k+1][j] + dims[i]*dims[k+1]*dims[j+1]
                for k in range(i, j)
            )
    return dp[0][n-1]

# 30x35 * 35x15 * 15x5 * 5x10 * 10x20 * 20x25
print("matrix chain:", matrix_chain([30, 35, 15, 5, 10, 20, 25]))   # 15125
```

