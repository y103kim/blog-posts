---
layout: post
title: '기초 통계학 정리'
tags: [Statistics, ML]
---

# 표본공간, 확률번수, 확률함수, 확률분포

- 표본공간: 실험으로 부터 나온 시행들의 경우의 수
- 확률변수: 표본공간의 Element를 실수로 변환
- 확률함수: 확률 변수를 0과 1사이의 실수로 변환
- 확률분포: 확률 함수로부터 나오는 확률들의 전반적인 패턴


# 통계량

- 평균, 편차, 분산, 표준편차와 같은 것들
- 표본평균: n개를 뽑았을때 평균, 그 자체로 확률변수
  - 표본평균의 기댓값 = 모집단의 평균
- 표본분산: n개를 뽑았을때 분산, 편차 제곱합을 $n-1$로 나눠야함
  - 표본분산의 기댓값 = 모집단의 분산
  - $n-1$로 나눠야만 기댓값이 모집단의 분산이 된다.
  - 즉 불편추정량(unbiased, 기댓값이 모집단과 그것과 같음)이 된다.
  - $n-1$은 자유도(독립 변수의 개수)이다.
- 표본평균의 분산: 표본 평균을 여러개 만들었을때의 분산
  - $V[\bar{X}] = \sigma/n$


# 조건부 확률

- 조건부 확률 공식
  - A,B 동시에 일어날 확률 = $P(A,B) = P(A \vert B)P(B)$
  - B가 일어났을때 A의 확률 = $P(A \vert B) = P(A,B)/P(B)$
- 전확률 - 다른 모든 확률들의 합이 1이면, 조건부를 붙이면 그 확률이 됨
  - if $\sum_{i=1}^n P(B_i) = 1$
  - then $P(A) = \sum_{i=1}^n P(A \vert B_i)P(B_i)$
- 베이즈 정리 - 전확률 + 조건부 확률 공식
  - $B$들이 일어났을 때 $A$의 확률을 전부 알음
  - $A$가 일어났을 때 $B_x$의 확률을 알 수 있음
  - 근데 그냥 테이블 그리면 풀기는 쉬움, 전체 개수를 임의 값으로 두어함
  - $P(B_x \vert A) = \dfrac {P(B_x,A)} {P(A)}=\dfrac {P(A \vert B_x)P(B_x)} {\sum_{i=1}^n P(A \vert B_i)P(B_i)}$


# 독립, 조건부 독립

- 독립: 확률의 곱이 동시에 일어날 확률과 같음
  - $P(A,B) = P(A)P(B)$
  - $P(A \vert B) = P(A)$
- 조건부 독립: C가 일어났을때의 독립
  - $P(A,B \vert C) = P(A \vert C)P(B \vert C)$
  - $P(A \vert B,C) = P(A \vert C)$


# 이산확률분포

- 베르누이 분포: p와 1-p의 확률을 가진 시행을 1번 진행
  - $\text{Bern}(x;p) = p^x(1-p)^{(1-x)}, \space x = 0,1$
  - $E[X] = p, \space V[X] = pq$
- 이항 분포: 여러번의 베르누이 시행을 반복, 각 시행은 독립
  - $\text{Bin}(x;N,p) = \dbinom{N}{x} p^x(1-p)^{N-x}, \space x = 0,1 \dots n$
  - $E[X] = np, \space V[X] = npq$
  - n이 크면 정규분포로 근사 가능
- 푸아송 분포: 단위 시간동안 특정 사건이 i번 발생할 확률
  - $\text{Poi}(k;\lambda) = \dfrac{\lambda^k}{k!}  e^{-\lambda}, \space x = 0,1 \dots$
  - $E[X] = \lambda, \space V[X] =  \lambda$
  - 이항분포가 $n \rightarrow \infty, p \rightarrow 0$ 이면 푸아송 분포로 근사된다.
- 기하 분포: 첫 번재 성공이 일어날 때 까지의 시행횟수가 $x$일 확률
  - $\text{Geo}(x;p) = (1-p)^{x-1}p, \space x = 1,2 \dots$
  - $E[X] = 1/p, \space V[X] = (1-p) / p^2$
  - 무기억성: 이전에 얼마를 시행했던간에, 처음 시행하는거랑 똑같다.
    - $P(X=x+k \vert X>k) = P(X=x), \space x=1,2,\cdots$
- 음이항분포, 초기하분포도 있음


# 연속확률변수

- 정규분포
  - $N(x;\mu,\sigma^2) = \dfrac{1}{\sqrt{2\pi\sigma^2}} \exp \left(-\dfrac{(x-\mu)^2}{2\sigma^2}\right)$
  - 평균과 중앙값이 같다.
  - 분산이 커질수록 넓적해진다. 분산이 작을수록 뾰족해진다.
  - 평균으로부터의 거리를 $\sigma$ 단위로 나타낼 수 있다.
  - 이항분포의 n이 커지면 정규분포로 근사된다.
- 표준 정균분포
  - $Z(x) = \dfrac{1}{\sqrt{2\pi}} \exp \left(-\dfrac{x^2}{2}\right)$
  - 평균 0, 분산 1로 만든 형태
  - 표가 있어서 값을 쉽게 구할 수 있다.
- 지수분포
  - 푸아송 분포의 Event간의 발생 시간
    - 어떤 사건의 발생 횟수가 푸아송 분포를 따르면, 대기시간은 지수분포를 따름
  - $\lambda$는 단위시간 동안 발생한 사건의 평균 수
  - $f(x) = \lambda e^{-\lambda x}, \space x\ge 0$
  - 마찬가지로 무기억성이 있음.
    - 과거 $t$만큼 시간이 지났더라도, 처음 시작하는거랑 똑같다.
    - 즉 이제 흐를 $s$만을 고려하면 된다.
    - $P(X \ge s+t \vert X \ge t) = P(X \ge s), \space \forall s,t \ge 0$