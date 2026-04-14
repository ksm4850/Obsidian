---
tags: [코테, DP, 배낭]
---

# Knapsack (배낭 문제)

## 개요

N개 아이템(무게 w, 가치 v) 중 총 무게 W 이하로 골라 **가치 최대화**. 변형:
- **0/1**: 각 아이템 한 번만
- **무한(Unbounded)**: 여러 번 가능
- **수량 제한(Bounded)**: 최대 k_i번

기본은 O(N·W). W가 10⁵ 넘어가면 다른 접근 필요.

## 0/1 Knapsack

```python
def knapsack01(items, W):
    """ items: [(w, v), ...] """
    dp = [0] * (W + 1)
    for w, v in items:
        for j in range(W, w - 1, -1):     # ← 역순이 핵심
            dp[j] = max(dp[j], dp[j - w] + v)
    return dp[W]
```

**역순 이유**: 한 아이템을 여러 번 쓰는 걸 방지. 왼쪽(작은 j)이 먼저 갱신되면 방금 담은 아이템이 또 쓰일 수 있음.

## 무한(Unbounded) Knapsack

```python
def knapsack_unbounded(items, W):
    dp = [0] * (W + 1)
    for w, v in items:
        for j in range(w, W + 1):         # ← 정방향
            dp[j] = max(dp[j], dp[j - w] + v)
    return dp[W]
```

정방향이므로 같은 아이템이 여러 번 쓰임 → 의도한 동작.

## 수량 제한 (Bounded)

각 아이템 k_i개면 k_i번 풀어도 되지만, **이진 분해**로 가속:

`k = 1 + 2 + 4 + ... + 2^m + r`로 쪼개 각 묶음을 0/1로 처리. O(N·W·log K).

```python
def knapsack_bounded(items, W):
    """ items: [(w, v, k), ...] """
    expanded = []
    for w, v, k in items:
        c = 1
        while c <= k:
            expanded.append((w*c, v*c))
            k -= c
            c *= 2
        if k > 0:
            expanded.append((w*k, v*k))
    return knapsack01(expanded, W)
```

## 부분합 (Subset Sum)

"N개 중 합이 S가 되는 부분집합 존재?" — 0/1 Knapsack의 bool 버전.

```python
def subset_sum(nums, S):
    dp = [False] * (S + 1)
    dp[0] = True
    for x in nums:
        for j in range(S, x - 1, -1):
            if dp[j - x]:
                dp[j] = True
    return dp[S]
```

비트셋 최적화 (Python int 무한 비트):

```python
def subset_sum_bitset(nums, S):
    dp = 1
    for x in nums:
        dp |= dp << x
    return (dp >> S) & 1 == 1
```

Python int 시프트는 빠름 → **10⁴ × 10⁴ 규모까지** 현실적. 단순 DP보다 훨씬 빠름.

## 개수 세기 (동전 교환)

"합이 S가 되는 방법의 수"는 무한 Knapsack 형태.

```python
def coin_change_count(coins, S):
    dp = [0] * (S + 1)
    dp[0] = 1
    for c in coins:
        for j in range(c, S + 1):
            dp[j] += dp[j - c]
    return dp[S]
```

동전 루프가 바깥일 때 **조합**(순서 무관), 안쪽일 때 **순열**(순서 유의) — 이 구분이 자주 출제.

## 최소 동전 개수

```python
def coin_change_min(coins, S):
    INF = float('inf')
    dp = [INF] * (S + 1)
    dp[0] = 0
    for j in range(1, S + 1):
        for c in coins:
            if c <= j:
                dp[j] = min(dp[j], dp[j - c] + 1)
    return dp[S] if dp[S] != INF else -1
```

## 함정

- 0/1과 무한의 차이 = 루프 방향. 한 글자 차이로 틀림.
- W가 10⁶ 이상이면 **Meet in the Middle** (N ≤ 40일 때 2^(N/2) 분할) 고려.
- 개수 세기 문제에서 "조합 vs 순열" 혼동 주의.

## 관련

[[DP 패턴]]의 대표 사례. [[Bitmask DP]]는 전혀 다른 상태 공간. 부분합을 무게 압축한 유형은 [[좌표 압축]]과 무관하게 직접 DP.

## 실행 예시

```python
# --- 0/1 Knapsack ---
def knapsack01(items, W):
    dp = [0] * (W + 1)
    for w, v in items:
        for j in range(W, w - 1, -1):
            dp[j] = max(dp[j], dp[j - w] + v)
    return dp[W]

items = [(2, 3), (3, 4), (4, 5), (5, 6)]
print("0/1 knapsack:", knapsack01(items, 8))   # 10

# --- 무한 Knapsack ---
def knapsack_unbounded(items, W):
    dp = [0] * (W + 1)
    for w, v in items:
        for j in range(w, W + 1):
            dp[j] = max(dp[j], dp[j - w] + v)
    return dp[W]

print("무한 knapsack:", knapsack_unbounded([(2, 3), (3, 4)], 8))   # 12 (2x4)

# --- 부분합 존재 ---
def subset_sum(nums, S):
    dp = [False] * (S + 1)
    dp[0] = True
    for x in nums:
        for j in range(S, x - 1, -1):
            if dp[j - x]:
                dp[j] = True
    return dp[S]

print("subset sum 11:", subset_sum([3, 34, 4, 12, 5, 2], 9))   # True
print("subset sum 30:", subset_sum([3, 34, 4, 12, 5, 2], 30))  # False

# --- 비트셋 최적화 ---
def subset_sum_bitset(nums, S):
    dp = 1
    for x in nums:
        dp |= dp << x
    return (dp >> S) & 1 == 1

print("bitset 9:", subset_sum_bitset([3, 34, 4, 12, 5, 2], 9))   # True

# --- 동전 방법 수 (조합) ---
def coin_change_count(coins, S):
    dp = [0] * (S + 1)
    dp[0] = 1
    for c in coins:
        for j in range(c, S + 1):
            dp[j] += dp[j - c]
    return dp[S]

print("동전 조합 수 (4원):", coin_change_count([1, 2, 3], 4))   # 4

# --- 최소 동전 개수 ---
def coin_change_min(coins, S):
    INF = float('inf')
    dp = [INF] * (S + 1)
    dp[0] = 0
    for j in range(1, S + 1):
        for c in coins:
            if c <= j:
                dp[j] = min(dp[j], dp[j - c] + 1)
    return dp[S] if dp[S] != INF else -1

print("최소 동전 수 (11원):", coin_change_min([1, 2, 5], 11))   # 3 (5+5+1)
```

