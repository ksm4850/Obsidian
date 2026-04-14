---
tags: [코테, 문자열, 회문]
---

# Manacher's Algorithm

## 개요

문자열의 **모든 회문 반지름**을 O(N)에 계산. 최장 회문 부분 문자열, 회문 개수 세기 등.

## 핵심 아이디어

각 중심 c에 대해 `P[c]` = c를 중심으로 하는 회문의 반지름. 이전에 구한 회문 정보를 **거울 대칭**으로 재활용해 선형.

중심이 **문자 사이**인 짝수 길이 회문도 다루려면, 원문에 구분자 삽입:

```
"abba" → "#a#b#b#a#"
```

이 변환 후엔 모든 회문을 홀수 길이로 다룰 수 있음.

## 구현

```python
def manacher(s):
    """
    반환: t[i]를 중심으로 한 회문 반지름 배열 (변환된 문자열 기준)
    """
    t = '#' + '#'.join(s) + '#'
    n = len(t)
    p = [0] * n
    c = r = 0    # 현재 회문 중심과 우측 경계

    for i in range(n):
        mirror = 2 * c - i
        if i < r:
            p[i] = min(r - i, p[mirror])
        while i + p[i] + 1 < n and i - p[i] - 1 >= 0 \
              and t[i + p[i] + 1] == t[i - p[i] - 1]:
            p[i] += 1
        if i + p[i] > r:
            c, r = i, i + p[i]

    return p
```

## 최장 회문 복원

변환 후 배열 `p`에서 최댓값의 위치·반지름을 잡고 역변환.

```python
def longest_palindrome(s):
    p = manacher(s)
    max_i = max(range(len(p)), key=lambda i: p[i])
    start = (max_i - p[max_i]) // 2
    return s[start:start + p[max_i]]
```

`p[i]` 자체가 **원 문자열에서의 회문 길이**. 변환 덕분에 짝수·홀수가 통합됨.

## 응용 1: 회문 개수 세기

각 위치 i에 대해 `(p[i] + 1) // 2`개의 회문이 그 중심에 있음. 모두 합하면 모든 회문의 개수(중복된 위치 포함).

```python
def count_palindromes(s):
    return sum((x + 1) // 2 for x in manacher(s))
```

## 응용 2: 특정 위치에서 시작/끝나는 최장 회문

각 시작 인덱스에서 "여기부터 시작하는 회문 중 최장"을 p 배열로부터 유도. 부가적 전처리가 필요.

## 함정

- 구분자는 원 문자열에 없는 문자여야 함. `#`이 입력에 등장할 수 있다면 `\x00` 등.
- `(max_i - p[max_i]) // 2` — 변환 좌표에서 원 좌표로 돌릴 때 인덱스 계산 주의.

## 관련

회문 전용 O(N) 알고리즘. 일반 패턴 매칭은 [[KMP]], [[Z Algorithm]]. 해시 기반 회문 판정은 [[Rabin-Karp]]의 **양방향** 해시로 O(1) 판정 가능하지만 상수가 크고 충돌 위험.

## 실행 예시

```python
def manacher(s):
    t = '#' + '#'.join(s) + '#'
    n = len(t)
    p = [0] * n
    c = r = 0
    for i in range(n):
        mirror = 2 * c - i
        if i < r:
            p[i] = min(r - i, p[mirror])
        while i + p[i] + 1 < n and i - p[i] - 1 >= 0 \
              and t[i + p[i] + 1] == t[i - p[i] - 1]:
            p[i] += 1
        if i + p[i] > r:
            c, r = i, i + p[i]
    return p

def longest_palindrome(s):
    p = manacher(s)
    max_i = max(range(len(p)), key=lambda i: p[i])
    start = (max_i - p[max_i]) // 2
    return s[start:start + p[max_i]]

print("최장 회문 'babad':", longest_palindrome("babad"))   # "bab" or "aba"
print("최장 회문 'cbbd':", longest_palindrome("cbbd"))     # "bb"
print("최장 회문 'abacdfgdcaba':", longest_palindrome("abacdfgdcaba"))  # "aba"

def count_palindromes(s):
    return sum((x + 1) // 2 for x in manacher(s))

print("회문 개수 'aaa':", count_palindromes("aaa"))   # 6 (a,a,a,aa,aa,aaa)
print("회문 개수 'abc':", count_palindromes("abc"))   # 3 (각 글자)
```

