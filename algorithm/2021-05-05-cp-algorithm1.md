---
layout: post
title: "CP 알고리즘 정리 (golang)"
tags: [algorithm, go]
---

올해는 [CP 알고리즘](https://cp-algorithms.com)을 꼭 다 습득할것!

# Binary Exponentiation

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/pow.go>

### Integer Power

일단은 간단한 거듭제곱 연산문제, 이진법 표기대로 더해준다고 생각하면 된다.

- [1629. 곱셈](https://www.acmicpc.net/problem/1629)
- [13172. Σ](https://www.acmicpc.net/problem/13272)
- [15717. 떡파이어](https://www.acmicpc.net/problem/15717)
- [15824. 너 봄에는 캡사이신이 맛있단다](https://www.acmicpc.net/problem/15824)
  - 개수를 세는 과정에서 pow가 사용된다. 개수 세는 과정에 대한 분류나 정리가 어려움,
  - 최대 최소가 동일한 조합의 수를 빠르게 센다고 생각하자.

### Matrix Power

행렬의 거듭제곱도 이진 제곱으로 손쉽게 풀린다.

- [11444. 피보나치 수 6](https://www.acmicpc.net/problem/11444)
  - 피보나치 수의 경우는 `[[1,1],[1,0]]` 행렬을 제곱하면 된다.
- [11442. 홀수번째 피보나치 수의 합](https://www.acmicpc.net/problem/11442)
  - 피보나치 수의 홀수항의 합은 피보나치 수열이다.
- [11443. 짝수번째 피보나치 수의 합](https://www.acmicpc.net/problem/11443)
  - 피보나치 수의 짝수항의 합은 피보나치 수열 - 1이다.
- [2086. 피보나치 수의 합](https://www.acmicpc.net/problem/2086)
  - 피보나치 수의 합은 피보나치 수열 -1이다.

### Power of Adjacency Matrix

인접 행렬의 거듭제곱은 A에서 B까지 갈 수 있는 경우의 수이다.

- [12850. 본대 산책2](https://www.acmicpc.net/problem/12850)

# Extended Euclidean Algorithm

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/euclidean.go>

### M이 소수가 아닐때, EEA로의 유도

- [14565. 역원 구하기](https://www.acmicpc.net/problem/14565)
  - 확장 유클리드 알고리즘의 가장 기본적인 활용처
  - $ax = 1 (mod \space m)$ 이면 $ax + my = 1$ 임을 이용해서 확장 유클리드로 변환한다.
- [3955. 캔디 분배](https://www.acmicpc.net/problem/3955)
  - $K*X+1 \equiv 0 \space (mod C)$
  - $K*X + C*Y = 1$
  - $X$가 반드시 음수가 되어야 하므로, $X \ge 0$일 경우는 $X$에 $C$를 빼고 $Y$에 $K$를 더한다.

### M이 소수일때, 페르마의 소정리를 통한 응용

소수로 모듈러 연산을 할때, 곱셈에 대한 역원(나눗셈)이 `Pow(a, m-2)`이 됨을 이용하면 된다.

- [20412. 추첨상 사수 대작전! (Hard)](https://www.acmicpc.net/problem/20412)

# Prime numbers

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/prime.go>

### Sieve of Eratosthenes

별 설명이 필요없는 에라토스테네스의 체, [포스팅](https://doocong.com/algorithm/sieve-of-eratosthenes/)한 적 있음

- [1747. 소수&팰린드롬](https://www.acmicpc.net/problem/1747)
  - 소수인지 판단하고, N보다 큰 경우 펠린드롬인지 확인하면 끝

### Miller-Rabin

- [5615. 아파트 임대](https://www.acmicpc.net/problem/5615)
  - $(2x + 1)(2y + 1) = (2k + 1)$이므로 k가 소수이면 x, y가 존재하지 않음
  - k가 매우 커질 수 있으므로 에라토스테네스의 체로는 안풀리고, 밀러 라빈을 써야함


# Array

### Inversion Count

- 버블소트를 할때 필요한 Swap의 수
- <https://github.com/y103kim/cp-algorithm/blob/main/array/inversion_count.go>
- 머지소트를 할때, Merge 단계에서 역전되는 수를 세거나
- Fenwick Tree나 Segement Tree로 구할 수 있다.

- [1517. 버블 소트](https://www.acmicpc.net/problem/1517)
  - 전형적인 Inversion Count 문제. 
- [New Year Chaos](https://www.hackerrank.com/challenges/new-year-chaos)
  - 2번의 이동제한이 있어서 머지소트로는 못푸는 문제
  - 그냥 뒤에서부터 2번 제한으로 Linear 검사하면, 선형시간에 풀 수 있음

# Query

### Range Query

- Fenwick Tree로 푸는 형태
  - [2042. 구간 합 구하기](https://www.acmicpc.net/problem/2042)
    - 가장 간단한 형태의 구간쿼리
  - [17353. 하늘에서 떨어지는 1, 2, ..., R-L+1개의 별](https://www.acmicpc.net/problem/17353)
    - X에 관한 식으로 합을 나타낼 수 있음을 이용하는 문제
    - 만약 2개의 update가 각각 $L_1, L_2$에서 있었다면
    - $2X - (L_1-1) - (L_2-1)$이 X에서의 합이다.
    - 이를 이용하여 두개의 펜윅 트리에 각각 1차, 상수항을 저장하면 풀린다.
  - [3653. 영화 수집](https://www.acmicpc.net/problem/3653)
    - 개수를 세는 유형
    - 위치가 변경될 것을 대비해서 아래로 공간을 확보해두어야한다.
- Segment Tree로 푸는 형태
  - [2357. 최솟값과 최대값](https://www.acmicpc.net/problem/2357)
    - 값이 전부 주어지고, 이후 최대, 최소 쿼리 계산
  - [1395. 스위치](https://www.acmicpc.net/problem/1395)
    - 스위치 반전 Update, 켜진 스위치의 합 Query
    - Range Update이기 때문에 Lazy Propgation 필요
- Array로 끝나는 형태
  - [Array Manipulation](https://www.hackerrank.com/challenges/crush)
    - Query가 단 한번이라면, Fenwick이나 Segment tree를 만들면 더 느리다.
    - 그냥 쭉 정리해놓고, N으로 Linear하게 탐색하면 된다.

# String

### ETC

- [가장 큰 수](https://programmers.co.kr/learn/courses/30/lessons/42746)
  - 이어붙여서 더 큰지를 비교하려면, `a + b < b + a`를 비교하면 된다.

# Stack

### 요철 형태의 계산 찾기

- 히스토그램 처럼 스택에 감소 혹은 증가 하는 요소만 남기면 되는 형태
- *"이 요소는 이후 계산에서 절대 쓰이지 않는다"*를 논증하면 해법을 찾기가 쉬워진다.
- [1725. 히스토그램](https://www.acmicpc.net/problem/1725)
  - 증가하는 요소만 저장해야 한다.
  - 사이즈를 나중에 알기 위해서 위치도 함께 저장하자.
- [3015. 오아시스 재결합](https://www.acmicpc.net/problem/3015)
  - 이 경우는 히스토그램과 반대로 감소하는 수열을 저장해야 한다.
  - 키가 같은 사람에 대한 처리를 위해서 스택에 연속된 사람수를 저장해야 한다.
- [Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)
  - 히스토그램 풀이를 매 행마다 반복해 주면 된다.

# Graph

### Topological Sort


# Combinatorics

### Cyclic Permutation

- [Minimum Swaps 2](https://www.hackerrank.com/challenges/minimum-swaps-2)
  - 순열 사이클 찾는 문제, 계속 Swap 하면 됨.