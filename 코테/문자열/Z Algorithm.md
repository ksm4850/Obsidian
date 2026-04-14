---
tags: [코테, 문자열, 매칭]
---

# Z Algorithm

## 개요

문자열 S에 대해 **Z 배열** 계산: `Z[i]` = "S와 S[i:]의 최장 공통 접두사 길이". 구축 **O(N)**. 패턴 매칭, 문자열 주기, [[KMP]]와 대체 가능.

## 직관

문자열 `aabcaabxaaz`의 Z값은:

```
i:  0 1 2 3 4 5 6 7 8 9 10
s:  a a b c a a b x a a  z
Z:  - 1 0 0 3 1 0 0 2 1  0
```

`Z[4]=3` → S[4:7]="aab"는 S[0:3]="aab"와 같다.

## 구현

"현재까지 발견한 가장 오른쪽으로 확장된 Z-box" [l, r]을 유지.

```python
def z_array(s):
    n = len(s)
    z = [0] * n
    z[0] = n
    l = r = 0
    for i in range(1, n):
        if i < r:
            z[i] = min(r - i, z[i - l])
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            z[i] += 1
        if i + z[i] > r:
            l, r = i, i + z[i]
    return z
```

## 패턴 매칭

`P + '#' + T`의 Z 배열을 계산. `Z[i] == len(P)`인 `i` 위치는 매칭.

```python
def z_search(text, pattern):
    if not pattern: return list(range(len(text) + 1))
    sep = '\x00'  # 양쪽에 없는 구분자
    combined = pattern + sep + text
    z = z_array(combined)
    m = len(pattern)
    matches = []
    for i in range(m + 1, len(combined)):
        if z[i] >= m:
            matches.append(i - m - 1)
    return matches
```

## 응용 1: 문자열의 주기

i가 주기 → `Z[i] == n - i`. 가장 작은 그런 i가 최소 주기.

```python
def smallest_period(s):
    n = len(s)
    z = z_array(s)
    for i in range(1, n):
        if i + z[i] == n and n % i == 0:
            return i
    return n
```

## 응용 2: 서로 다른 부분 문자열 개수

suffix array/LCP로 O(N log N)이 표준이나, Z 배열도 보조로 쓰일 수 있음.

## Z vs [[KMP]]

| 구분 | KMP | Z |
|------|-----|---|
| 주요 배열 | fail (접두사=접미사) | Z (S와 S[i:]의 LCP) |
| 직관 | 실패 시 점프 | 매 위치의 공통 접두사 길이 |
| 구현 난이도 | 중 | 중 (Z-box) |
| 용도 | 매칭, 주기 | 매칭, 주기, 더 다양한 쿼리 |

둘 다 선형. 개인 취향 및 문제 요구에 따라 선택.

## 함정

- `min(r - i, z[i - l])`에서 **경계 외 접근 방지** 필수. `i < r`일 때만 이 가지로 진입.
- Z[0]을 0으로 두는 구현도 많다. 전체 길이로 정의하는 게 자연스러움.

## 관련

[[KMP]]의 대체. 해시 방법은 [[Rabin-Karp]]. 회문은 [[Manacher]].

## 실행 예시

```python
def z_array(s):
    n = len(s)
    z = [0] * n
    z[0] = n
    l = r = 0
    for i in range(1, n):
        if i < r:
            z[i] = min(r - i, z[i - l])
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            z[i] += 1
        if i + z[i] > r:
            l, r = i, i + z[i]
    return z

print("Z 'aabcaabxaaz':", z_array("aabcaabxaaz"))
# [11, 1, 0, 0, 3, 1, 0, 0, 2, 1, 0]

def z_search(text, pattern):
    if not pattern: return list(range(len(text) + 1))
    combined = pattern + '\x00' + text
    z = z_array(combined)
    m = len(pattern)
    matches = []
    for i in range(m + 1, len(combined)):
        if z[i] >= m:
            matches.append(i - m - 1)
    return matches

print("Z 매칭:", z_search("ababcababab", "abab"))   # [0, 5, 7]

# --- 최소 주기 ---
def smallest_period(s):
    n = len(s)
    z = z_array(s)
    for i in range(1, n):
        if i + z[i] == n and n % i == 0:
            return i
    return n

print("주기('abcabcabc'):", smallest_period("abcabcabc"))   # 3
print("주기('abcabd'):", smallest_period("abcabd"))         # 6 (주기 없음)
```

