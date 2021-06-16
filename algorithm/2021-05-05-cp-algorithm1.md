---
layout: post
title: "CP 알고리즘 정리 (golang)"
tags: [algorithm, go]
---

올해는 [CP 알고리즘](https://cp-algorithms.com)을 꼭 다 습득할것!

# Binary Exponentiation

### Integer Power

일단은 간단한 거듭제곱 연산문제, 이진법 표기대로 더해준다고 생각하면 된다.

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/pow.go#L7-L17>

- [1629. 곱셈](https://www.acmicpc.net/problem/1629)
- [13172. Σ](https://www.acmicpc.net/problem/13272)
- [15717. 떡파이어](https://www.acmicpc.net/problem/15717)
- [15824. 너 봄에는 캡사이신이 맛있단다](https://www.acmicpc.net/problem/15824)
  - 개수를 세는 과정에서 pow가 사용된다. 개수 세는 과정에 대한 분류나 정리가 어려움,
  - 최대 최소가 동일한 조합의 수를 빠르게 센다고 생각하자.

### Matrix Power

행렬의 거듭제곱도 이진 제곱으로 손쉽게 풀린다.

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/pow.go#L23-L59>

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

### M이 소수가 아닐때, EEA로의 유도

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/euclidean.go#L3-L14>

- [14565. 역원 구하기](https://www.acmicpc.net/problem/14565)
  - 확장 유클리드 알고리즘의 가장 기본적인 활용처
  - $ax = 1 (mod \space m)$ 이면 $ax + my = 1$ 임을 이용해서 확장 유클리드로 변환한다.
- [3955. 캔디 분배](https://www.acmicpc.net/problem/3955)
  - $K*X+1 \equiv 0 \space (mod C)$
  - $K*X + C*Y = 1$
  - $X$가 반드시 음수가 되어야 하므로, $X \ge 0$일 경우는 $X$에 $C$를 빼고 $Y$에 $K$를 더한다.

### M이 소수일때, 페르마의 소정리를 통한 응용

소수로 모듈러 연산을 할때, 곱셈에 대한 역원(나눗셈)이 `Pow(a, m-2)`이 됨을 이용하면 된다.

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/pow.go#L19-L21>

- [20412. 추첨상 사수 대작전! (Hard)](https://www.acmicpc.net/problem/20412)

# String

### ETC

- [가장 큰 수](https://programmers.co.kr/learn/courses/30/lessons/42746)
  - 이어붙여서 더 큰지를 비교하려면, `a + b < b + a`를 비교하면 된다.

# Stack

### 요철 형태의 계산 찾기

- 히스토그램 처럼 스택에 감소 혹은 증가 하는 요소만 남기면 되는 형태
- "이 요소는 이후 계산에서 절대 쓰이지 않는다"를 논증하면 해법을 찾기가 쉬워진다.
- [1725. 히스토그램](https://www.acmicpc.net/problem/1725)
  - 증가하는 요소만 저장해야 한다.
  - 사이즈를 나중에 알기 위해서 위치도 함께 저장하자.
- [3015. 오아시스 재결합](https://www.acmicpc.net/problem/3015)
  - 이 경우는 히스토그램과 반대로 감소하는 수열을 저장해야 한다.
  - 키가 같은 사람에 대한 처리를 위해서 스택에 연속된 사람수를 저장해야 한다.
- [Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)
  - 히스토그램 풀이를 매 행마다 만복해 주면 된다.

