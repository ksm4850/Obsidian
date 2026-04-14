---
tags: [코테, 자료구조, 문자열]
---

# Trie (접두사 트리)

## 개요

문자열 집합을 트리 형태로 저장. 루트에서 리프까지의 경로가 하나의 문자열. **검색/삽입 O(L)** (L = 문자열 길이), 공간은 문자 종류 × 총 길이.

자동완성, 접두사 매칭, 사전 문제, XOR 최대값(비트 트라이)에서 활용.

## 기본 구현

```python
class Trie:
    def __init__(self):
        self.children = {}
        self.end = False

    def insert(self, word):
        node = self
        for ch in word:
            if ch not in node.children:
                node.children[ch] = Trie()
            node = node.children[ch]
        node.end = True

    def search(self, word):
        node = self._walk(word)
        return node is not None and node.end

    def starts_with(self, prefix):
        return self._walk(prefix) is not None

    def _walk(self, s):
        node = self
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node
```

## 배열 버전 (속도 최적화)

알파벳 소문자만 나올 때 dict 대신 배열로 하면 훨씬 빠름.

```python
class Trie:
    def __init__(self):
        self.child = [None] * 26
        self.end = False

    def insert(self, word):
        node = self
        for c in word:
            i = ord(c) - ord('a')
            if node.child[i] is None:
                node.child[i] = Trie()
            node = node.child[i]
        node.end = True
```

## 응용 1: 최장 공통 접두사

모든 문자열을 삽입한 뒤 **분기가 생기거나 end를 만날 때까지** 내려가면 LCP.

## 응용 2: XOR 최대값 (비트 트라이)

두 수 XOR의 최댓값을 구할 때 30비트 트라이를 만들고, 각 수에 대해 **반대 비트**로 탐욕적으로 내려감. O(N × 30).

```python
def max_xor(nums):
    root = [None, None]
    for x in nums:
        node = root
        for i in range(30, -1, -1):
            b = (x >> i) & 1
            if node[b] is None:
                node[b] = [None, None]
            node = node[b]
    best = 0
    for x in nums:
        node = root
        cur = 0
        for i in range(30, -1, -1):
            b = (x >> i) & 1
            if node[1-b] is not None:
                cur |= (1 << i)
                node = node[1-b]
            else:
                node = node[b]
        best = max(best, cur)
    return best
```

## 응용 3: 단어 사전 DP

[[DP 패턴|DP]]와 결합해 "문자열을 사전 단어로 분해 가능?" 문제. Trie로 매칭을 O(L)로 줄이면 전체 O(N²) 이하 가능.

## 함정

- dict 기반은 상수가 크다 → 10⁶ 이상 노드 다루면 TLE 위험. 배열 또는 `defaultdict(dict)` 선호.
- 메모리 폭발 주의: 문자 종류가 많으면 **트리 + 해시** 혼합 고려.

## 관련

[[KMP]]는 단일 패턴 매칭, Trie는 **다중 패턴** 집합 매칭. 둘을 합친 게 Aho-Corasick.

## 실행 예시

```python
class Trie:
    def __init__(self):
        self.children = {}
        self.end = False
    def insert(self, word):
        node = self
        for ch in word:
            if ch not in node.children:
                node.children[ch] = Trie()
            node = node.children[ch]
        node.end = True
    def search(self, word):
        node = self._walk(word)
        return node is not None and node.end
    def starts_with(self, prefix):
        return self._walk(prefix) is not None
    def _walk(self, s):
        node = self
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node

# --- 기본 ---
t = Trie()
for w in ["apple", "app", "apricot", "banana"]:
    t.insert(w)

print("search 'app':", t.search("app"))           # True
print("search 'ap':", t.search("ap"))             # False
print("starts_with 'ap':", t.starts_with("ap"))   # True
print("starts_with 'bat':", t.starts_with("bat")) # False

# --- XOR 최댓값 (비트 트라이) ---
def max_xor(nums):
    root = [None, None]
    for x in nums:
        node = root
        for i in range(30, -1, -1):
            b = (x >> i) & 1
            if node[b] is None:
                node[b] = [None, None]
            node = node[b]
    best = 0
    for x in nums:
        node = root
        cur = 0
        for i in range(30, -1, -1):
            b = (x >> i) & 1
            if node[1-b] is not None:
                cur |= (1 << i)
                node = node[1-b]
            else:
                node = node[b]
        best = max(best, cur)
    return best

print("max XOR:", max_xor([3, 10, 5, 25, 2, 8]))   # 28 (5 ^ 25)
```

