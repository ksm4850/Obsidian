---
tags: [코테, DP, 수열, 문자열]
---

# LCS (최장 공통 부분 수열)

## 개요

두 수열의 **공통되는 부분 수열** 중 최장(연속일 필요 없음). O(N×M) DP. 문자열 유사도·diff·편집거리의 기초.

## 기본 점화식

`dp[i][j]` = `A[:i]`와 `B[:j]`의 LCS 길이.

```
dp[i][j] = dp[i-1][j-1] + 1           if A[i-1] == B[j-1]
         = max(dp[i-1][j], dp[i][j-1]) otherwise
```

## Python 구현

```python
def lcs(a, b):
    n, m = len(a), len(b)
    dp = [[0] * (m + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[n][m]
```

## 역추적으로 LCS 문자열 복원

```python
def lcs_string(a, b):
    n, m = len(a), len(b)
    dp = [[0]*(m+1) for _ in range(n+1)]
    for i in range(1, n+1):
        for j in range(1, m+1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    # 역추적
    i, j = n, m
    res = []
    while i > 0 and j > 0:
        if a[i-1] == b[j-1]:
            res.append(a[i-1])
            i -= 1; j -= 1
        elif dp[i-1][j] >= dp[i][j-1]:
            i -= 1
        else:
            j -= 1
    return ''.join(reversed(res))
```

## 공간 최적화 O(min(N, M))

`dp[i]`가 `dp[i-1]`만 참조 → 두 행만 유지.

```python
def lcs_length(a, b):
    if len(a) < len(b): a, b = b, a
    prev = [0] * (len(b) + 1)
    cur = [0] * (len(b) + 1)
    for i in range(1, len(a) + 1):
        for j in range(1, len(b) + 1):
            if a[i-1] == b[j-1]:
                cur[j] = prev[j-1] + 1
            else:
                cur[j] = max(prev[j], cur[j-1])
        prev, cur = cur, prev
    return prev[len(b)]
```

## 편집 거리 (Levenshtein)

LCS의 친척. 삽입·삭제·치환 각 1비용.

```python
def edit_distance(a, b):
    n, m = len(a), len(b)
    dp = [[0]*(m+1) for _ in range(n+1)]
    for i in range(n+1): dp[i][0] = i
    for j in range(m+1): dp[0][j] = j
    for i in range(1, n+1):
        for j in range(1, m+1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    return dp[n][m]
```

## 응용: 공통 부분 문자열 (substring)

연속이 필요하면 점화식이 달라짐. 불일치 시 0으로 초기화.

```python
def longest_common_substring(a, b):
    n, m = len(a), len(b)
    dp = [[0]*(m+1) for _ in range(n+1)]
    best = 0
    for i in range(1, n+1):
        for j in range(1, m+1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
                best = max(best, dp[i][j])
    return best
```

## 함정

- **LCS는 unique하지 않음** — 복원이 여러 개 나올 수 있으므로, 문제에서 정답이 여러 개여도 하나만 제출.
- 문자열 길이 10⁴ × 10⁴ = 10⁸ DP → Python에서 한계. PyPy나 더 나은 알고리즘 필요.

## 관련

[[LIS]]와 달리 두 수열을 다룸. 연속 부분 문자열 매칭은 [[KMP]], [[Rabin-Karp]]. 편집 거리도 동일 DP 패턴.

## 실행 예시

```python
def lcs(a, b):
    n, m = len(a), len(b)
    dp = [[0] * (m + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[n][m]

print("LCS 길이:", lcs("ABCBDAB", "BDCAB"))   # 4

# --- LCS 문자열 복원 ---
def lcs_string(a, b):
    n, m = len(a), len(b)
    dp = [[0]*(m+1) for _ in range(n+1)]
    for i in range(1, n+1):
        for j in range(1, m+1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    i, j = n, m
    res = []
    while i > 0 and j > 0:
        if a[i-1] == b[j-1]:
            res.append(a[i-1]); i -= 1; j -= 1
        elif dp[i-1][j] >= dp[i][j-1]:
            i -= 1
        else:
            j -= 1
    return ''.join(reversed(res))

print("LCS:", lcs_string("ABCBDAB", "BDCAB"))   # "BCAB" 또는 "BDAB"

# --- 편집 거리 ---
def edit_distance(a, b):
    n, m = len(a), len(b)
    dp = [[0]*(m+1) for _ in range(n+1)]
    for i in range(n+1): dp[i][0] = i
    for j in range(m+1): dp[0][j] = j
    for i in range(1, n+1):
        for j in range(1, m+1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    return dp[n][m]

print("edit distance:", edit_distance("kitten", "sitting"))   # 3

# --- 연속 공통 부분 문자열 ---
def longest_common_substring(a, b):
    n, m = len(a), len(b)
    dp = [[0]*(m+1) for _ in range(n+1)]
    best = 0
    for i in range(1, n+1):
        for j in range(1, m+1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
                best = max(best, dp[i][j])
    return best

print("공통 substring:", longest_common_substring("ABABC", "BABCA"))   # 4 ("BABC")
```

