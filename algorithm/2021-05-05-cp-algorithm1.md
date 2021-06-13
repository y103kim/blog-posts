---
layout: post
title: "CP 알고리즘 정리 (golang)"
tags: [algorithm, go]
---

올해는 [CP 알고리즘](https://cp-algorithms.com)을 꼭 다 습득할것!

# Binary Exponentiation

### Integer Power

일단은 간단한 거듭제곱 연산문제, 이진법 표기대로 더해준다고 생각하면 된다.

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/pow.go#L3-L15>

- [1629. 곱셈](https://www.acmicpc.net/problem/1629)
- [13172. Σ](https://www.acmicpc.net/problem/13272)
- [15717. 떡파이어](https://www.acmicpc.net/problem/15717)
- [15824. 너 봄에는 캡사이신이 맛있단다](https://www.acmicpc.net/problem/15824)
  - 개수를 세는 과정에서 pow가 사용된다. 개수 세는 과정에 대한 분류나 정리가 어려움,
  - 최대 최소가 동일한 조합의 수를 빠르게 센다고 생각하자.

### Matrix Power

행렬의 거듭제곱도 이진 제곱으로 손쉽게 풀린다.

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/pow.go#L17-L54>

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

### M이 소수가 아닐때, 곱셈의 역원

<https://github.com/y103kim/cp-algorithm/blob/main/algebrea/euclidean.go#L3-L14>

- [14565. 역원 구하기](https://www.acmicpc.net/problem/14565)
  - 확장 유클리드 알고리즘의 가장 기본적인 활용처
  - $ax = 1 (mod \space m)$ 이면 $ax + my = 1$ 임을 이용해서 확장 유클리드로 변환한다.


## String

### ETC

- [가장 큰 수](https://programmers.co.kr/learn/courses/30/lessons/42746)
  - 이어붙여서 더 큰지를 비교하려면, `a + b < b + a`를 비교하면 된다.