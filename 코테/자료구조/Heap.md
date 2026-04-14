---
tags: [코테, 자료구조, 우선순위큐]
---

# Heap (우선순위 큐)

## 개요

완전 이진트리 형태의 자료구조로, 부모-자식 간 우선순위 관계를 유지. 삽입/삭제 모두 **O(log N)**, 최솟값(최댓값) 조회 O(1). Python은 `heapq`로 **min-heap만** 제공 — max-heap은 부호를 뒤집어 사용.

[[Dijkstra]], [[MST]]의 Prim, K번째 원소, 스트림 중앙값 등에서 핵심.

## 배열 = 트리 (0-based 인덱스)

Python `heapq`는 **일반 리스트를 그대로 힙으로 사용**. 별도 트리 노드 없이 배열 인덱스로 부모-자식 관계를 표현한다.

```
인덱스:   0    1    2    3    4    5    6
배열:   [ 1,   3,   2,   5,   4,   6,   7 ]

트리로 보면:
            1(0)
          /      \
       3(1)      2(2)
      /    \    /    \
    5(3)  4(4) 6(5) 7(6)
```

### 인덱스 계산식 (0-based)

| 관계 | 공식 | 예시 (i=1) |
|------|------|-----------|
| **부모** | `(i - 1) // 2` | (1-1)//2 = **0** |
| **왼쪽 자식** | `2 * i + 1` | 2*1+1 = **3** |
| **오른쪽 자식** | `2 * i + 2` | 2*1+2 = **4** |

```python
def parent(i):     return (i - 1) // 2
def left(i):       return 2 * i + 1
def right(i):      return 2 * i + 2
```

> 1-based 인덱스(교과서 스타일)면 `parent = i//2`, `left = 2*i`, `right = 2*i+1`로 더 깔끔하지만, Python `heapq`는 **0-based**이므로 위 공식을 써야 함.

## 핵심 연산

```python
import heapq

h = []
heapq.heappush(h, 3)        # O(log N)
heapq.heappush(h, 1)
heapq.heappush(h, 2)
heapq.heappop(h)            # 1, O(log N)
h[0]                        # 최솟값 peek, O(1)

# 리스트를 힙으로: O(N)
arr = [5, 3, 8, 1, 2]
heapq.heapify(arr)

# push + pop 동시에 (더 빠름)
heapq.heappushpop(h, 5)
heapq.heapreplace(h, 5)     # pop 먼저, push 나중
```

## Max-Heap

부호 뒤집기 또는 튜플 첫 원소 음수화.

```python
h = []
heapq.heappush(h, -5)       # 5를 넣고 싶을 때
top = -heapq.heappop(h)     # 5

# 객체와 함께 우선순위 관리
heapq.heappush(h, (-priority, obj))
```

## 응용 1: K번째 큰 원소

크기 K짜리 min-heap을 유지. 새 원소가 top보다 크면 교체. **O(N log K)**.

```python
def kth_largest(nums, k):
    h = nums[:k]
    heapq.heapify(h)
    for x in nums[k:]:
        if x > h[0]:
            heapq.heapreplace(h, x)
    return h[0]
```

## 응용 2: 스트림 중앙값

max-heap(작은 절반) + min-heap(큰 절반) 균형 유지.

```python
class MedianFinder:
    def __init__(self):
        self.lo = []  # max-heap (음수)
        self.hi = []  # min-heap

    def add(self, x):
        heapq.heappush(self.lo, -heapq.heappushpop(self.hi, x))
        if len(self.lo) > len(self.hi):
            heapq.heappush(self.hi, -heapq.heappop(self.lo))

    def median(self):
        if len(self.hi) > len(self.lo):
            return self.hi[0]
        return (self.hi[0] - self.lo[0]) / 2
```

## 응용 3: K개 정렬된 리스트 병합

```python
def merge_k_sorted(lists):
    h = []
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(h, (lst[0], i, 0))
    result = []
    while h:
        val, i, j = heapq.heappop(h)
        result.append(val)
        if j + 1 < len(lists[i]):
            heapq.heappush(h, (lists[i][j+1], i, j+1))
    return result
```

## 함정

- `heapq`는 동일 우선순위에서 **두 번째 원소까지 비교**. 객체를 넣을 땐 `(priority, idx, obj)` 튜플로 idx를 tie-breaker로 둘 것.
- max-heap 흉내내려고 음수화하면 부동소수점 정밀도 손실 가능.

## 관련

[[Dijkstra]]는 매번 가장 가까운 정점을 꺼내야 하므로 힙이 필수. [[Segment Tree]]와 달리 힙은 **임의 위치 갱신**이 비효율(O(N)).

## 실행 예시

```python
import heapq

# --- 기본 heap 연산 ---
h = []
for x in [5, 3, 8, 1, 2]:
    heapq.heappush(h, x)
print("min:", h[0])                      # min: 1
print("pop:", heapq.heappop(h))          # pop: 1
print("heap after pop:", h)              # [2, 3, 8, 5]

# --- K번째 큰 원소 ---
def kth_largest(nums, k):
    h = nums[:k]
    heapq.heapify(h)
    for x in nums[k:]:
        if x > h[0]:
            heapq.heapreplace(h, x)
    return h[0]

print("3rd largest:", kth_largest([3, 2, 1, 5, 6, 4], 3))   # 4

# --- 스트림 중앙값 ---
class MedianFinder:
    def __init__(self):
        self.lo = []
        self.hi = []
    def add(self, x):
        heapq.heappush(self.lo, -heapq.heappushpop(self.hi, x))
        if len(self.lo) > len(self.hi):
            heapq.heappush(self.hi, -heapq.heappop(self.lo))
    def median(self):
        if len(self.hi) > len(self.lo):
            return self.hi[0]
        return (self.hi[0] - self.lo[0]) / 2

mf = MedianFinder()
for x in [1, 2, 3, 4, 5]:
    mf.add(x)
    print(f"add {x} -> median {mf.median()}")
# add 1 -> 1, add 2 -> 1.5, add 3 -> 2, add 4 -> 2.5, add 5 -> 3

# --- K개 정렬 리스트 병합 ---
def merge_k_sorted(lists):
    h = []
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(h, (lst[0], i, 0))
    result = []
    while h:
        val, i, j = heapq.heappop(h)
        result.append(val)
        if j + 1 < len(lists[i]):
            heapq.heappush(h, (lists[i][j+1], i, j+1))
    return result

print("merged:", merge_k_sorted([[1, 4, 7], [2, 5], [3, 6, 9]]))
# merged: [1, 2, 3, 4, 5, 6, 7, 9]
```

