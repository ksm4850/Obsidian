---
tags: [코테, 문자열, 매칭]
---

# KMP

## 개요

문자열 T(길이 N)에서 패턴 P(길이 M) 매칭을 **O(N + M)**에. 핵심은 **실패 함수**(failure function, LPS): `fail[i]` = P[0..i]의 "진짜 접두사 = 접미사"인 최대 길이.

## 실패 함수 구축

```python
def failure(p):
    m = len(p)
    fail = [0] * m
    k = 0
    for i in range(1, m):
        while k > 0 and p[k] != p[i]:
            k = fail[k - 1]
        if p[k] == p[i]:
            k += 1
        fail[i] = k
    return fail
```

`fail[i]` = P[0..i]의 접두사이자 접미사인 최대 길이(자기 자신 제외).

## 매칭

```python
def kmp_search(text, pattern):
    if not pattern:
        return list(range(len(text) + 1))
    fail = failure(pattern)
    matches = []
    k = 0
    for i, c in enumerate(text):
        while k > 0 and pattern[k] != c:
            k = fail[k - 1]
        if pattern[k] == c:
            k += 1
        if k == len(pattern):
            matches.append(i - len(pattern) + 1)
            k = fail[k - 1]
    return matches
```

## 왜 선형인가

`while k > 0`에서 k가 감소, 바깥 for에서 최대 1 증가 → amortized O(N+M).

## 응용 1: 문자열의 주기

`n - fail[n-1]`이 **가능한 주기**. 전체 길이가 이 값의 배수면 그 주기로 완전 반복.

```python
def is_repeated(s):
    n = len(s)
    f = failure(s)
    period = n - f[-1]
    return period < n and n % period == 0
```

## 응용 2: 두 문자열 연결 후 경계 매칭

"S의 접미사이자 T의 접두사인 최대 길이" → `T + '#' + S`의 실패 함수 끝값.

```python
def longest_overlap(s, t):
    combined = t + '#' + s         # # 은 양쪽에 없는 구분자
    f = failure(combined)
    return f[-1]
```

## 응용 3: 주기 수 세기

`fail[i]`를 따라 올라가면서 P[0..i]의 모든 접두사-접미사 길이를 열거.

```python
def all_borders(p):
    f = failure(p)
    k = len(p)
    res = []
    while k > 0:
        k = f[k - 1] if k == len(p) else f[k - 1]
        res.append(k)
        if k == 0: break
    return res
```

## 함정

- `fail[i - 1]` vs `fail[i]` 인덱싱 실수. 여러 구현체가 다르니 자기 스타일을 고정.
- 패턴이 빈 문자열일 때 엣지 케이스 처리.

## 관련

[[Z Algorithm]]도 같은 문제를 다른 배열로 품. 다중 패턴은 Aho-Corasick. 해시 기반은 [[Rabin-Karp]] — 충돌 위험 있으나 구현 쉬움.

## 실행 예시

```python
def failure(p):
    m = len(p)
    fail = [0] * m
    k = 0
    for i in range(1, m):
        while k > 0 and p[k] != p[i]:
            k = fail[k - 1]
        if p[k] == p[i]:
            k += 1
        fail[i] = k
    return fail

def kmp_search(text, pattern):
    if not pattern: return list(range(len(text) + 1))
    fail = failure(pattern)
    matches = []
    k = 0
    for i, c in enumerate(text):
        while k > 0 and pattern[k] != c:
            k = fail[k - 1]
        if pattern[k] == c:
            k += 1
        if k == len(pattern):
            matches.append(i - len(pattern) + 1)
            k = fail[k - 1]
    return matches

print("failure 'ababcabab':", failure("ababcabab"))
# [0, 0, 1, 2, 0, 1, 2, 3, 4]

print("KMP 매칭:", kmp_search("ababcabcabababd", "ababd"))
# [10]

print("KMP 다중 매칭:", kmp_search("aaaaa", "aa"))
# [0, 1, 2, 3]

# --- 문자열 주기 판정 ---
def is_repeated(s):
    n = len(s)
    f = failure(s)
    period = n - f[-1]
    return period < n and n % period == 0

print("abcabc 주기?", is_repeated("abcabc"))   # True
print("abcabd 주기?", is_repeated("abcabd"))   # False
print("ababab 주기?", is_repeated("ababab"))   # True

# --- 접두-접미 최대 오버랩 ---
def longest_overlap(s, t):
    combined = t + '#' + s
    f = failure(combined)
    return f[-1]

print("overlap:", longest_overlap("abcde", "deabc"))   # 2 ("de")
```

