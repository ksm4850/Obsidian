---
tags: [코테, 문자열, 해시]
---

# Rabin-Karp (롤링 해시)

## 개요

문자열을 **다항식 해시**로 변환 후 윈도우를 1칸씩 밀면서 O(1) 갱신. 매칭 평균 O(N + M), 최악 O(NM). 해시 충돌은 이중 해시로 극복.

핵심 장점: 다중 패턴, 문자열 비교를 정수 비교로 환원, 부분문자열 상등 판정 O(1).

## 롤링 해시 기본

```python
def rolling_hash(s, m, base=131, mod=10**9 + 7):
    """
    길이 m 윈도우의 해시들을 반환 (각 시작 위치별)
    """
    n = len(s)
    if n < m: return []
    hashes = []
    h = 0
    power = 1
    for i in range(m):
        h = (h * base + ord(s[i])) % mod
        if i < m - 1:
            power = (power * base) % mod
    hashes.append(h)
    for i in range(m, n):
        h = (h * base + ord(s[i]) - ord(s[i-m]) * power * base) % mod
        hashes.append(h)
    return hashes
```

엄밀히는: `new = (old - s[i-m] * base^(m-1)) * base + s[i]`. 위 코드는 이를 정리한 형태.

## 매칭

```python
def rk_search(text, pattern, base=131, mod=10**9 + 7):
    n, m = len(text), len(pattern)
    if m > n: return []
    ph = 0
    th = 0
    power = 1
    for i in range(m):
        ph = (ph * base + ord(pattern[i])) % mod
        th = (th * base + ord(text[i])) % mod
        if i < m - 1:
            power = (power * base) % mod
    matches = []
    for i in range(n - m + 1):
        if ph == th and text[i:i+m] == pattern:   # 충돌 검증
            matches.append(i)
        if i < n - m:
            th = (th * base + ord(text[i+m]) - ord(text[i]) * power * base) % mod
    return matches
```

## 이중 해시 (충돌 극복)

두 개의 (base, mod) 쌍으로 동시 해시. 둘 다 같을 확률은 거의 0. 대회에선 직접 비교 생략 가능.

```python
MOD1, MOD2 = 10**9 + 7, 10**9 + 9
BASE1, BASE2 = 131, 137
# 두 쌍을 병렬로 유지, 매칭은 (h1, h2) 튜플 비교
```

## 접두사 해시 + 구간 해시 O(1)

전체 문자열 해시를 한 번 만들어두면 임의 부분 문자열 해시를 O(1)에.

```python
class StringHash:
    def __init__(self, s, base=131, mod=10**9 + 7):
        n = len(s)
        self.base = base
        self.mod = mod
        self.h = [0] * (n + 1)       # prefix hash
        self.p = [1] * (n + 1)       # base^i
        for i in range(n):
            self.h[i+1] = (self.h[i] * base + ord(s[i])) % mod
            self.p[i+1] = (self.p[i] * base) % mod

    def get(self, l, r):             # s[l..r] (inclusive)
        return (self.h[r+1] - self.h[l] * self.p[r-l+1]) % self.mod
```

## 응용 1: 중복 부분 문자열

"길이 k인 중복 부분 문자열 존재?" — 모든 길이 k 부분의 해시를 set에 삽입. 이분탐색과 결합하면 최장 중복 부분을 O(N log N)에.

## 응용 2: 순환 동치 판별

`s + s` 안에 `t`가 substring으로 있는지. 롤링 해시로 O(N).

## 함정

- 해시 충돌: 단일 해시는 대회에서 저격(anti-hash test) 가능. 이중 해시 권장.
- mod 연산 음수: `% mod` 결과가 음수일 수 있음 → `(x % mod + mod) % mod`.
- base는 **모든 문자보다 큰** 값 선택 (아스키면 131, 137 등).

## 관련

[[KMP]], [[Z Algorithm]]는 확정적 O(N)이라 해시 충돌 걱정 없음. 대신 다중 패턴이나 임의 부분 매칭엔 해시가 유연. [[Manacher]]는 회문 전용.

## 실행 예시

```python
def rk_search(text, pattern, base=131, mod=10**9 + 7):
    n, m = len(text), len(pattern)
    if m > n: return []
    ph = 0; th = 0; power = 1
    for i in range(m):
        ph = (ph * base + ord(pattern[i])) % mod
        th = (th * base + ord(text[i])) % mod
        if i < m - 1:
            power = (power * base) % mod
    matches = []
    for i in range(n - m + 1):
        if ph == th and text[i:i+m] == pattern:
            matches.append(i)
        if i < n - m:
            th = (th * base + ord(text[i+m]) - ord(text[i]) * power * base) % mod
    return matches

print("RK 매칭:", rk_search("ababcababab", "abab"))   # [0, 5, 7]
print("RK 매칭:", rk_search("hello world", "world"))   # [6]

# --- 구간 해시로 부분 문자열 상등 판정 ---
class StringHash:
    def __init__(self, s, base=131, mod=10**9 + 7):
        n = len(s)
        self.base = base
        self.mod = mod
        self.h = [0] * (n + 1)
        self.p = [1] * (n + 1)
        for i in range(n):
            self.h[i+1] = (self.h[i] * base + ord(s[i])) % mod
            self.p[i+1] = (self.p[i] * base) % mod
    def get(self, l, r):
        return (self.h[r+1] - self.h[l] * self.p[r-l+1]) % self.mod

s = "ababab"
sh = StringHash(s)
print("s[0:2] == s[2:4]?", sh.get(0, 1) == sh.get(2, 3))   # True  ("ab" == "ab")
print("s[0:3] == s[1:4]?", sh.get(0, 2) == sh.get(1, 3))   # False ("aba" != "bab")
print("s[0:4] == s[2:6]?", sh.get(0, 3) == sh.get(2, 5))   # True  ("abab" == "abab")
```

