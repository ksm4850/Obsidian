---
tags: [코테, DP, 수열]
---

# LIS (최장 증가 부분 수열)

## 개요

수열에서 **인덱스가 증가하면서 값도 증가**하는 부분 수열 중 최장. 나이브 DP는 O(N²), [[이분 탐색]]을 쓰면 **O(N log N)**.

## O(N²) DP

`dp[i]` = i로 끝나는 LIS 길이.

```python
def lis_n2(arr):
    n = len(arr)
    dp = [1] * n
    for i in range(n):
        for j in range(i):
            if arr[j] < arr[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
```

## O(N log N) — tails 배열

`tails[k]` = "길이 k+1인 증가 수열의 마지막 값 중 최소". 각 원소를 `bisect_left`로 삽입 위치 찾기.

```python
from bisect import bisect_left

def lis(arr):
    tails = []
    for x in arr:
        i = bisect_left(tails, x)
        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x
    return len(tails)
```

**주의**: `tails` 자체는 실제 LIS가 아니라 **각 길이에서의 최솟값**을 추적하는 배열이다. 길이만 정확하다.

## 비감소 수열(LNDS) — `bisect_right`

```python
# 같은 값 허용하려면
i = bisect_right(tails, x)
```

## LIS 복원

각 원소의 **삽입 위치**를 기록, 위치 역추적.

```python
from bisect import bisect_left

def lis_with_path(arr):
    tails = []
    pos = []                # 각 원소의 tails 삽입 위치
    parent = [-1] * len(arr)
    indices = []            # tails[k]가 실제 arr의 어느 인덱스인지

    for i, x in enumerate(arr):
        k = bisect_left(tails, x)
        if k == len(tails):
            tails.append(x)
            indices.append(i)
        else:
            tails[k] = x
            indices[k] = i
        if k > 0:
            parent[i] = indices[k-1]
        pos.append(k)

    # 역추적
    length = len(tails)
    idx = indices[-1]
    path = []
    while idx != -1:
        path.append(arr[idx])
        idx = parent[idx]
    return path[::-1]
```

## 응용 1: 두 수열 공통 LIS

순열이면 한쪽 위치로 다른 쪽을 변환한 뒤 LIS. 일반 수열은 [[LCS]].

## 응용 2: 2D LIS (Patience / 박스 중첩)

`(a, b)` 쌍에서 a 오름차순, b 내림차순(동점 처리) 정렬 후 b에 대해 LIS. "동점 시 내림차순"이 **같은 a에서 여러 개를 한 사슬에 포함시키지 않게** 해줌.

```python
def box_stacking(boxes):
    # boxes: [(w, h), ...]
    boxes.sort(key=lambda x: (x[0], -x[1]))
    return lis([h for _, h in boxes])
```

## 응용 3: LDS, LNIS

반대 방향은 배열 뒤집기 또는 부호 변환. 부호 변환이 가장 깔끔.

## 함정

- `tails`의 k번째 값 ≠ LIS의 k번째 원소. **길이만** 맞는다.
- 엄밀히 증가(`<`) vs 비감소(`<=`)를 문제에서 확인. `bisect_left` vs `bisect_right` 구분 실수 자주 나옴.

## 관련

O(N log N) 풀이의 핵심이 [[이분 탐색]]. 두 수열 비교는 [[LCS]]. 박스 중첩 등은 [[DP 패턴]]과 결합.

## 실행 예시

```python
from bisect import bisect_left

def lis(arr):
    tails = []
    for x in arr:
        i = bisect_left(tails, x)
        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x
    return len(tails)

print("LIS 길이:", lis([10, 9, 2, 5, 3, 7, 101, 18]))   # 4 ([2,3,7,18])
print("LIS 길이:", lis([0, 1, 0, 3, 2, 3]))              # 4

# --- LIS 복원 ---
def lis_with_path(arr):
    tails = []
    parent = [-1] * len(arr)
    indices = []
    for i, x in enumerate(arr):
        k = bisect_left(tails, x)
        if k == len(tails):
            tails.append(x); indices.append(i)
        else:
            tails[k] = x; indices[k] = i
        if k > 0:
            parent[i] = indices[k-1]
    idx = indices[-1]
    path = []
    while idx != -1:
        path.append(arr[idx])
        idx = parent[idx]
    return path[::-1]

print("LIS 수열:", lis_with_path([10, 9, 2, 5, 3, 7, 101, 18]))
# [2, 3, 7, 18] (또는 [2, 5, 7, 18] 등 유효 해)

# --- 박스 중첩 ---
def box_stacking(boxes):
    boxes.sort(key=lambda x: (x[0], -x[1]))
    return lis([h for _, h in boxes])

print("max 중첩:", box_stacking([(5, 4), (6, 4), (6, 7), (2, 3), (1, 1)]))  # 3
```

