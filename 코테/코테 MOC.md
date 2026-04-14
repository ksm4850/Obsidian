---
tags: [코테, MOC]
---

# 코테 MOC

코딩테스트 중급~고급 자료구조 및 알고리즘 인덱스. Python 기준.

## 자료구조

선형/비선형을 넘어서 코테에서 자주 등장하는 **변별력 있는** 구조들.

- [[Heap]] — 우선순위 큐, K번째 원소, 스트림 중앙값
- [[Union-Find]] — DSU, 경로 압축. [[MST]]·연결성 판정의 기반
- [[Trie]] — 문자열 prefix 검색, 자동완성
- [[Segment Tree]] — 구간 합/최대/최소, lazy propagation
- [[Fenwick Tree]] — BIT. [[Segment Tree]]의 경량 대안
- [[Monotonic Stack]] — 다음 큰 수, 히스토그램 최대 사각형

## 그래프

- [[BFS DFS 응용]] — 격자, 멀티소스, 컴포넌트
- [[Dijkstra]] — 양의 가중치 최단경로 ([[Heap]] 활용)
- [[Bellman-Ford]] — 음수 가중치, 음수 사이클 탐지
- [[Floyd-Warshall]] — 모든 쌍 최단경로, O(V³)
- [[위상 정렬]] — DAG, 작업 순서
- [[MST]] — Kruskal ([[Union-Find]]), Prim ([[Heap]])
- [[SCC]] — Tarjan, Kosaraju

## DP

- [[DP 패턴]] — 메모이제이션 vs 타뷸레이션, 점화식 설계
- [[LIS]] — O(N log N) ([[이분 탐색]] 활용)
- [[LCS]] — 2D DP, 역추적
- [[Knapsack]] — 0/1, 무한, 부분합
- [[Bitmask DP]] — TSP, 외판원
- [[트리 DP]] — 서브트리 DP, 리루팅

## 문자열

- [[KMP]] — 실패 함수 기반 매칭
- [[Z Algorithm]] — Z 배열, [[KMP]] 대안
- [[Manacher]] — 회문 O(N)
- [[Rabin-Karp]] — 롤링 해시

## 탐색 기법

- [[이분 탐색]] — `bisect`, Parametric Search
- [[Two Pointers]] — 정렬 + 투포인터
- [[슬라이딩 윈도우]] — deque, 가변 윈도우

## 수학

- [[에라토스테네스]] — 소수, 선형 체
- [[확장 유클리드]] — GCD, 베주 항등식
- [[모듈러 역원]] — 페르마 소정리, [[확장 유클리드]]
- [[조합론]] — nCr mod p, 카탈란 수
- [[행렬 거듭제곱]] — 피보나치 O(log N)

## 기타 기법

- [[백트래킹]] — 가지치기 패턴 ([[Bitmask DP]]와 비교)
- [[좌표 압축]] — 큰 좌표 → 인덱스 매핑 ([[Segment Tree]] 전처리)
- [[분할 정복]] — 머지소트, 역원 카운트

## 빈출 조합

- 격자 BFS + 비트마스크 상태 → [[BFS DFS 응용]] + [[Bitmask DP]]
- 구간 쿼리 + 좌표 압축 → [[Segment Tree]] + [[좌표 압축]]
- 매개변수 탐색 → [[이분 탐색]] (결정 문제로 변환)
- 트리에서 두 정점 경로 → [[트리 DP]] 또는 LCA
