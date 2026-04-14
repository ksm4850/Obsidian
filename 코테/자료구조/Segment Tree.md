---
tags: [코테, 자료구조, 구간쿼리]
---

# Segment Tree

## 개요

배열의 **구간 쿼리**(합/최대/최소/GCD 등)와 **점 업데이트**를 각각 O(log N)에 처리. 트리 높이 ⌈log₂ N⌉ + 1, 메모리 4N.

구간 업데이트까지 필요하면 **Lazy Propagation** 추가.

## 반복 버전 (가장 실전적)

1-indexed, 크기 2N. Bottom-up이라 빠르고 코드 짧음.

```python
class SegTree:
    def __init__(self, arr):
        self.n = len(arr)
        self.tree = [0] * (2 * self.n)
        for i, x in enumerate(arr):
            self.tree[self.n + i] = x
        for i in range(self.n - 1, 0, -1):
            self.tree[i] = self.tree[2*i] + self.tree[2*i+1]

    def update(self, i, val):
        i += self.n
        self.tree[i] = val
        while i > 1:
            i //= 2
            self.tree[i] = self.tree[2*i] + self.tree[2*i+1]

    def query(self, l, r):  # [l, r)
        res = 0
        l += self.n
        r += self.n
        while l < r:
            if l & 1:
                res += self.tree[l]
                l += 1
            if r & 1:
                r -= 1
                res += self.tree[r]
            l //= 2
            r //= 2
        return res
```

## 최댓값/최솟값 세그

합을 `max`로 바꾸면 됨. 초기값은 `-inf` 또는 `inf`.

```python
# __init__에서
self.tree = [-float('inf')] * (2 * self.n)
# 갱신 / 쿼리에서 += 대신 = max(...)
```

## Lazy Propagation — 구간 업데이트

"구간 [l, r]에 전부 +x" 같은 연산도 O(log N)에 가능.

```python
class LazySegTree:
    def __init__(self, n):
        self.n = n
        self.tree = [0] * (4 * n)
        self.lazy = [0] * (4 * n)

    def _push(self, node, s, e):
        if self.lazy[node]:
            self.tree[node] += (e - s + 1) * self.lazy[node]
            if s != e:
                self.lazy[2*node] += self.lazy[node]
                self.lazy[2*node+1] += self.lazy[node]
            self.lazy[node] = 0

    def update(self, l, r, val, node=1, s=0, e=None):
        if e is None: e = self.n - 1
        self._push(node, s, e)
        if r < s or e < l: return
        if l <= s and e <= r:
            self.lazy[node] += val
            self._push(node, s, e)
            return
        m = (s + e) // 2
        self.update(l, r, val, 2*node, s, m)
        self.update(l, r, val, 2*node+1, m+1, e)
        self.tree[node] = self.tree[2*node] + self.tree[2*node+1]

    def query(self, l, r, node=1, s=0, e=None):
        if e is None: e = self.n - 1
        self._push(node, s, e)
        if r < s or e < l: return 0
        if l <= s and e <= r: return self.tree[node]
        m = (s + e) // 2
        return self.query(l, r, 2*node, s, m) + self.query(l, r, 2*node+1, m+1, e)
```

## 응용

- **역원 카운트**: 우측부터 세그에 넣으면서 "나보다 작은 수" 쿼리 — O(N log N)
- **좌표가 너무 크면** [[좌표 압축]]과 결합
- **k번째 원소 찾기**: 빈도 세그 → 이진탐색으로 O(log N)

## 세그 vs [[Fenwick Tree]]

| 구분 | Segment Tree | Fenwick Tree |
|------|-------------|--------------|
| 구현 | 복잡 | 짧음 |
| 메모리 | 4N | N |
| 연산 | 임의 결합법칙 가능 (max, gcd, ...) | 합/XOR 계열만 |
| 구간 업데이트 | lazy로 가능 | 이중 BIT 필요 |

**합만 필요하면 Fenwick, 그 외엔 Segment Tree**.

## 함정

- 재귀 버전은 Python에서 N=10⁶ 수준이면 TLE. 반복 버전 또는 PyPy.
- lazy 전파 누락 버그가 많음 — 모든 재귀 진입 직전에 `_push` 호출.

## 관련

[[Fenwick Tree]]는 더 가벼운 대안. [[좌표 압축]]과 조합해 큰 좌표계 쿼리 처리.

## 실행 예시

```python
class SegTree:
    def __init__(self, arr):
        self.n = len(arr)
        self.tree = [0] * (2 * self.n)
        for i, x in enumerate(arr):
            self.tree[self.n + i] = x
        for i in range(self.n - 1, 0, -1):
            self.tree[i] = self.tree[2*i] + self.tree[2*i+1]
    def update(self, i, val):
        i += self.n
        self.tree[i] = val
        while i > 1:
            i //= 2
            self.tree[i] = self.tree[2*i] + self.tree[2*i+1]
    def query(self, l, r):   # [l, r)
        res = 0
        l += self.n; r += self.n
        while l < r:
            if l & 1: res += self.tree[l]; l += 1
            if r & 1: r -= 1; res += self.tree[r]
            l //= 2; r //= 2
        return res

arr = [1, 3, 5, 7, 9, 11]
st = SegTree(arr)
print("sum [1, 4):", st.query(1, 4))       # 3+5+7 = 15
print("sum [0, 6):", st.query(0, 6))       # 36
st.update(2, 10)                            # arr[2]: 5 -> 10
print("after update, [1, 4):", st.query(1, 4))  # 3+10+7 = 20

# --- Lazy Propagation: 구간 업데이트 + 구간 합 ---
class LazySegTree:
    def __init__(self, n):
        self.n = n
        self.tree = [0] * (4 * n)
        self.lazy = [0] * (4 * n)
    def _push(self, node, s, e):
        if self.lazy[node]:
            self.tree[node] += (e - s + 1) * self.lazy[node]
            if s != e:
                self.lazy[2*node] += self.lazy[node]
                self.lazy[2*node+1] += self.lazy[node]
            self.lazy[node] = 0
    def update(self, l, r, val, node=1, s=0, e=None):
        if e is None: e = self.n - 1
        self._push(node, s, e)
        if r < s or e < l: return
        if l <= s and e <= r:
            self.lazy[node] += val
            self._push(node, s, e)
            return
        m = (s + e) // 2
        self.update(l, r, val, 2*node, s, m)
        self.update(l, r, val, 2*node+1, m+1, e)
        self.tree[node] = self.tree[2*node] + self.tree[2*node+1]
    def query(self, l, r, node=1, s=0, e=None):
        if e is None: e = self.n - 1
        self._push(node, s, e)
        if r < s or e < l: return 0
        if l <= s and e <= r: return self.tree[node]
        m = (s + e) // 2
        return self.query(l, r, 2*node, s, m) + self.query(l, r, 2*node+1, m+1, e)

lz = LazySegTree(6)
lz.update(1, 3, 5)                      # [1..3]에 +5
lz.update(2, 5, 3)                      # [2..5]에 +3
print("lazy [0, 5]:", lz.query(0, 5))   # 5+8+8+3+3 = 27
print("lazy [2, 3]:", lz.query(2, 3))   # 8+8 = 16
```

