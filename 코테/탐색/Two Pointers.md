---
tags: [코테, 탐색, 투포인터]
---

# Two Pointers

## 개요

두 인덱스가 **단조로 움직이는** 기법. N중 루프를 O(N)으로 축약. 정렬 + 투포인터 패턴이 가장 흔함.

## 패턴 1: 정렬 + 합이 S인 쌍 찾기

```python
def two_sum_sorted(arr, S):
    l, r = 0, len(arr) - 1
    pairs = []
    while l < r:
        s = arr[l] + arr[r]
        if s == S:
            pairs.append((arr[l], arr[r]))
            l += 1; r -= 1
        elif s < S:
            l += 1
        else:
            r -= 1
    return pairs
```

양쪽 포인터가 단조 → O(N).

## 패턴 2: 부분합이 S 이상인 최소 길이

연속 부분배열, 가변 윈도우. [[슬라이딩 윈도우]]의 한 케이스로 볼 수도 있음.

```python
def min_len_subarray(arr, S):
    n = len(arr)
    l = 0
    cur = 0
    best = float('inf')
    for r in range(n):
        cur += arr[r]
        while cur >= S:
            best = min(best, r - l + 1)
            cur -= arr[l]
            l += 1
    return best if best != float('inf') else 0
```

## 패턴 3: 3Sum

정렬 후 바깥 루프 + 내부 투포인터.

```python
def three_sum(arr):
    arr = sorted(arr)
    n = len(arr)
    res = []
    for i in range(n - 2):
        if i > 0 and arr[i] == arr[i-1]: continue
        l, r = i + 1, n - 1
        while l < r:
            s = arr[i] + arr[l] + arr[r]
            if s == 0:
                res.append((arr[i], arr[l], arr[r]))
                l += 1; r -= 1
                while l < r and arr[l] == arr[l-1]: l += 1
                while l < r and arr[r] == arr[r+1]: r -= 1
            elif s < 0:
                l += 1
            else:
                r -= 1
    return res
```

전체 O(N²).

## 패턴 4: 합쳐진 두 정렬 배열

머지소트의 머지 단계.

```python
def merge(a, b):
    i = j = 0
    res = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            res.append(a[i]); i += 1
        else:
            res.append(b[j]); j += 1
    res.extend(a[i:]); res.extend(b[j:])
    return res
```

## 패턴 5: 조건 만족 부분배열 개수

"합이 S 이하인 연속 부분배열 개수". r을 고정하면 가능한 l의 범위가 단조.

```python
def count_subarrays_sum_le(arr, S):
    l = 0; cur = 0; count = 0
    for r, x in enumerate(arr):
        cur += x
        while cur > S and l <= r:
            cur -= arr[l]; l += 1
        count += r - l + 1
    return count
```

**음수가 섞이면 단조성 깨짐** → 투포인터 대신 다른 방법 (prefix + [[Fenwick Tree]] 등).

## 투포인터 vs [[슬라이딩 윈도우]]

거의 같은 개념이나 관례적으로:
- **투포인터**: 포인터 두 개가 **서로 다른 속도/방향**으로 움직임 (3Sum처럼 양 끝에서)
- **슬라이딩 윈도우**: l, r이 **같은 방향으로 단조 증가**

경계는 모호. 둘 다 amortized O(N).

## 함정

- 투포인터는 **단조성**이 전제. 단조가 깨지면 성립 안 함 — 음수 원소, 복잡한 조건 등.
- 정렬이 필요한 패턴인지 먼저 확인. 정렬 O(N log N)이 병목일 수 있음.

## 관련

[[슬라이딩 윈도우]]와 경계가 겹침. 부분합 쿼리 문제에서는 [[Fenwick Tree]]도 대안.

## 실행 예시

```python
# --- 정렬 + 두 수의 합 ---
def two_sum_sorted(arr, S):
    l, r = 0, len(arr) - 1
    pairs = []
    while l < r:
        s = arr[l] + arr[r]
        if s == S:
            pairs.append((arr[l], arr[r]))
            l += 1; r -= 1
        elif s < S:
            l += 1
        else:
            r -= 1
    return pairs

print("pairs sum=7:", two_sum_sorted([1, 2, 3, 4, 5, 6], 7))   # [(1,6),(2,5),(3,4)]

# --- 최소 길이 부분배열 합 >= S ---
def min_len_subarray(arr, S):
    l = 0; cur = 0; best = float('inf')
    for r in range(len(arr)):
        cur += arr[r]
        while cur >= S:
            best = min(best, r - l + 1)
            cur -= arr[l]; l += 1
    return best if best != float('inf') else 0

print("min len (S=7):", min_len_subarray([2, 3, 1, 2, 4, 3], 7))   # 2 ([4,3])

# --- 3Sum ---
def three_sum(arr):
    arr = sorted(arr)
    n = len(arr)
    res = []
    for i in range(n - 2):
        if i > 0 and arr[i] == arr[i-1]: continue
        l, r = i + 1, n - 1
        while l < r:
            s = arr[i] + arr[l] + arr[r]
            if s == 0:
                res.append((arr[i], arr[l], arr[r]))
                l += 1; r -= 1
                while l < r and arr[l] == arr[l-1]: l += 1
                while l < r and arr[r] == arr[r+1]: r -= 1
            elif s < 0:
                l += 1
            else:
                r -= 1
    return res

print("3Sum:", three_sum([-1, 0, 1, 2, -1, -4]))
# [(-1, -1, 2), (-1, 0, 1)]

# --- 합 <= S인 부분배열 개수 (음수 없을 때) ---
def count_subarrays_sum_le(arr, S):
    l = 0; cur = 0; count = 0
    for r, x in enumerate(arr):
        cur += x
        while cur > S and l <= r:
            cur -= arr[l]; l += 1
        count += r - l + 1
    return count

print("합 <= 6인 부분배열 수:", count_subarrays_sum_le([1, 2, 3, 4], 6))   # 7
```

