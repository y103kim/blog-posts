---
layout: post
category: language
title: Parallel Computing with Scala
tags: [scala, bigdata]
---

Coursera의 Parallel Computing with Scala 정리
  - 링크 <https://coursera.org/learn/scala-parallel-programming/>

# Parallel Programming

- resource based locking `syncronized`
  - `x.syncronized`: atomic execution 보장
  - 두개 이상의 syncronized를 겹치면 데드락 발생 가능
- 병렬처리 `parallel`
  - `parallel(sumSegment(A), sumSegment(B))`
  - call-by-name으로 선언되어 있으므로, statement를 그대로 넣으면 됨
  - 재귀 호출시에 side-effect가 없으면 parallel을 붙이기가 쉽다.
- fork with `task`
  - `task`로 statement를 넘기면 쓰레드가 포크되고, 문장이 실행 시작
  - `task.join`으로 blocking하고 결과를 받아옴
  - `parallel`을 `task`로 구현함
  
# Parallel Algorithms

- `List`는 parallel mapping에 적합하지 않음
  - 절반 쪼개거나 합치는 비용이 비쌈
  - `array`, `tree`가 대안
- `reduceLeft`, `reduceRight`
  - 두 연산이 associative하다면 `reduceLeft(...) == reduceRight(...)`
  - associative한 연산들에 대해서 병렬화가 가능
- `foldLeft`, `foldRight`
  - fold는 초기값이 각각 왼쪽, 오른쪽끝으로 감
  - 그러므로 associative, commutative이면 `foldLeft(...) == foldRight(...)`
  - 마찬가지로 associative, commutative이면 병렬화 가능
- `scanLeft`, `scanRight`
  - 

### associativity

- 연산들: associative, commutative
  - 정수의 합과 곱
  - 집합의 union, intersection
  - 논리연산
  - 다항식의 합과 곱
  - 행렬이나 백터의 합
- 연산들: associative, not commutative
  - concat, append, matmul, `function.compose`
- 주의 해야 할 연산들
  - floating point연산은 associative하지 않다. under/overflow때문
- 어떤 연산을 comutative로 만들기
  - if문으로 순서를 강제하면 됨
  - associative로 만드는것은 어려움
- tuple의 associative
  - `f((a,b), (c,d)) = (f(a,c), f(b,d))`
  - 위 식을 만족하면 associative 성립
  - pairwise연산을 지원하는 여부라고 보면 쉬움
