---
tags: [코테, 자료구조, 구간쿼리]
---

# Fenwick Tree (BIT)

## 개요

"Binary Indexed Tree". **점 업데이트 + 구간 합**을 둘 다 O(log N)에. 구현이 짧고 상수가 작아 [[Segment Tree]]보다 실전에서 선호되는 경우가 많음.

한계: 결합법칙이 있는 "역연산 가능한" 연산(합, XOR)만. 최댓값/최솟값은 불가 → 이 경우 Segment Tree 사용.

## 핵심 트릭

`i & -i`가 가장 낮은 비트(LSB)를 뽑아줌. 이 값을 더하거나 빼면서 트리를 오르내림.

```python
class BIT:
    def __init__(self, n):
        self.n = n
        self.tree = [0] * (n + 1)  # 1-indexed

    def update(self, i, delta):
        i += 1  # 내부 1-indexed
        while i <= self.n:
            self.tree[i] += delta
            i += i & -i

    def prefix(self, i):  # [0..i] 합
        i += 1
        s = 0
        while i > 0:
            s += self.tree[i]
            i -= i & -i
        return s

    def range(self, l, r):  # [l..r] 합
        return self.prefix(r) - (self.prefix(l - 1) if l > 0 else 0)
```

## 응용 1: 역원 카운트

우측부터 BIT에 넣으면서, 현재 원소보다 **작은 수의 개수**(= prefix(x-1))를 누적.

```python
def inversions(arr):
    # 좌표 압축 생략 가능하면 그대로. 값이 크면 압축 필요.
    n = max(arr) + 1
    bit = BIT(n)
    inv = 0
    for x in reversed(arr):
        inv += bit.prefix(x - 1) if x > 0 else 0
        bit.update(x, 1)
    return inv
```

## 응용 2: 구간 업데이트 + 점 쿼리

"차분 배열"로 변환하면 BIT 하나로 가능.

```python
# [l, r]에 +v 업데이트 → diff[l] += v, diff[r+1] -= v
# 점 i의 값 = prefix(i)
bit.update(l, v)
bit.update(r + 1, -v)
val = bit.prefix(i)
```

## 응용 3: 구간 업데이트 + 구간 쿼리

BIT 두 개(`B1`, `B2`) 사용.

```python
class RangeBIT:
    def __init__(self, n):
        self.n = n
        self.b1 = BIT(n + 2)
        self.b2 = BIT(n + 2)

    def update(self, l, r, v):
        self.b1.update(l, v)
        self.b1.update(r + 1, -v)
        self.b2.update(l, v * (l - 1))
        self.b2.update(r + 1, -v * r)

    def prefix(self, i):
        return self.b1.prefix(i) * i - self.b2.prefix(i)

    def query(self, l, r):
        return self.prefix(r) - (self.prefix(l - 1) if l > 0 else 0)
```

## 응용 4: 2D BIT

`tree[N+1][N+1]`로 2차원 확장. 2D 직사각형 합 쿼리에 유용.

## 언제 [[Segment Tree]] 대신 BIT인가

- **합/XOR만 필요**
- 구현 시간 절약 필요
- 상수 중요 (10⁷ 쿼리 이상)

## 함정

- 1-indexed 기반 구현이 전통. 0-indexed로 쓰려면 인덱스 변환 주의.
- 최댓값/최솟값은 BIT로 **업데이트가 감소 방향이면 불가능**. (증가 방향만 지원하는 변형은 있음.)

## 관련

[[Segment Tree]]와 비교 표는 해당 문서 참조. 역원 카운트는 [[분할 정복]]의 머지소트로도 O(N log N)에 가능 — 상황에 따라 선택.

## 실행 예시

```python
class BIT:
    def __init__(self, n):
        self.n = n
        self.tree = [0] * (n + 1)
    def update(self, i, delta):
        i += 1
        while i <= self.n:
            self.tree[i] += delta
            i += i & -i
    def prefix(self, i):
        i += 1
        s = 0
        while i > 0:
            s += self.tree[i]
            i -= i & -i
        return s
    def range(self, l, r):
        return self.prefix(r) - (self.prefix(l - 1) if l > 0 else 0)

# --- 기본 ---
bit = BIT(6)
for i, v in enumerate([1, 3, 5, 7, 9, 11]):
    bit.update(i, v)

print("prefix 2 (0..2):", bit.prefix(2))    # 1+3+5 = 9
print("range [1..4]:", bit.range(1, 4))     # 3+5+7+9 = 24
bit.update(2, 10)                            # arr[2] += 10
print("range [1..4] after:", bit.range(1, 4))  # 3+15+7+9 = 34

# --- 역원 카운트 ---
def inversions(arr):
    n = max(arr) + 1
    bit = BIT(n)
    inv = 0
    for x in reversed(arr):
        inv += bit.prefix(x - 1) if x > 0 else 0
        bit.update(x, 1)
    return inv

print("inversions:", inversions([3, 1, 2, 4]))   # 2  (3>1, 3>2)
print("inversions:", inversions([5, 4, 3, 2, 1])) # 10 (완전 역순)
```

