---
tags: [코테, 자료구조, 스택]
---

# Monotonic Stack / Queue

## 개요

스택(또는 덱) 안의 원소가 항상 **단조 증가 또는 감소** 상태를 유지. 각 원소가 push/pop 각 1회 → 전체 O(N). "다음 큰 수", "히스토그램 최대 직사각형", [[슬라이딩 윈도우]] 최댓값에서 핵심.

## 패턴 1: 다음 큰 수 (Next Greater Element)

```python
def next_greater(arr):
    n = len(arr)
    res = [-1] * n
    stack = []  # 인덱스 보관, 값이 감소하는 스택
    for i, x in enumerate(arr):
        while stack and arr[stack[-1]] < x:
            res[stack.pop()] = x
        stack.append(i)
    return res
```

핵심: **x보다 작은 원소가 스택 top에 있으면, x가 그 원소의 다음 큰 수이므로 pop하며 답 기록.**

## 패턴 2: 히스토그램 최대 직사각형

각 막대에 대해 "왼쪽·오른쪽으로 자기보다 낮은 첫 막대"를 찾으면, 그 사이가 해당 막대를 높이로 하는 최대 직사각형의 너비.

```python
def largest_rectangle(heights):
    stack = []
    max_area = 0
    heights = heights + [0]  # sentinel
    for i, h in enumerate(heights):
        while stack and heights[stack[-1]] > h:
            top = stack.pop()
            left = stack[-1] if stack else -1
            width = i - left - 1
            max_area = max(max_area, heights[top] * width)
        stack.append(i)
    return max_area
```

## 패턴 3: 슬라이딩 윈도우 최댓값 (Monotonic Deque)

덱에 인덱스를 유지하되 **값이 감소**하도록. front가 현재 윈도우의 최댓값.

```python
from collections import deque

def sliding_max(arr, k):
    dq = deque()
    res = []
    for i, x in enumerate(arr):
        while dq and arr[dq[-1]] < x:
            dq.pop()                    # 작은 값 제거
        dq.append(i)
        if dq[0] <= i - k:
            dq.popleft()                # 윈도우 밖 인덱스 제거
        if i >= k - 1:
            res.append(arr[dq[0]])
    return res
```

## 패턴 4: 수열에서 "범위 최솟값 곱 합" 유형

각 원소가 **최솟값이 되는 구간의 개수** × (원소 값)을 합산. 백준 6549, LeetCode 907 유형.

```python
def sum_subarray_mins(arr, mod=10**9+7):
    n = len(arr)
    left = [0] * n   # 각 원소가 최솟값인 가장 먼 왼쪽 경계
    right = [0] * n
    stack = []
    for i in range(n):
        while stack and arr[stack[-1]] >= arr[i]:
            stack.pop()
        left[i] = stack[-1] if stack else -1
        stack.append(i)
    stack.clear()
    for i in range(n - 1, -1, -1):
        while stack and arr[stack[-1]] > arr[i]:
            stack.pop()
        right[i] = stack[-1] if stack else n
        stack.append(i)
    return sum(arr[i] * (i - left[i]) * (right[i] - i) for i in range(n)) % mod
```

중복값 처리: **한쪽은 `>=`, 다른 쪽은 `>`** 로 tie-breaking 해야 중복 카운트 방지.

## 함정

- 스택에 **값이 아닌 인덱스**를 넣는 게 거의 항상 유연함.
- 단조성 조건(`<` vs `<=`)을 문제마다 다시 따질 것. 중복 처리 차이가 결정적.

## 관련

[[슬라이딩 윈도우]]에서 deque 버전이 자주 등장. [[Segment Tree]]로도 범위 최솟값이 되지만, monotonic은 더 가볍고 빠름.

## 실행 예시

```python
from collections import deque

# --- 다음 큰 수 ---
def next_greater(arr):
    n = len(arr)
    res = [-1] * n
    stack = []
    for i, x in enumerate(arr):
        while stack and arr[stack[-1]] < x:
            res[stack.pop()] = x
        stack.append(i)
    return res

print("next greater:", next_greater([2, 1, 2, 4, 3]))
# [4, 2, 4, -1, -1]

# --- 히스토그램 최대 직사각형 ---
def largest_rectangle(heights):
    stack = []
    max_area = 0
    heights = heights + [0]
    for i, h in enumerate(heights):
        while stack and heights[stack[-1]] > h:
            top = stack.pop()
            left = stack[-1] if stack else -1
            width = i - left - 1
            max_area = max(max_area, heights[top] * width)
        stack.append(i)
    return max_area

print("largest rect:", largest_rectangle([2, 1, 5, 6, 2, 3]))  # 10

# --- 슬라이딩 윈도우 최댓값 ---
def sliding_max(arr, k):
    dq = deque()
    res = []
    for i, x in enumerate(arr):
        while dq and arr[dq[-1]] < x:
            dq.pop()
        dq.append(i)
        if dq[0] <= i - k:
            dq.popleft()
        if i >= k - 1:
            res.append(arr[dq[0]])
    return res

print("sliding max:", sliding_max([1, 3, -1, -3, 5, 3, 6, 7], 3))
# [3, 3, 5, 5, 6, 7]
```

