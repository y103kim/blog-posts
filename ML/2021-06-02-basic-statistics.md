---
layout: post
category: ML
title: "기초 통계학 정리"
tags: [Statistics, ML]
---

# 확률에 대한 정의

- 빈도주의 확률
  - 무한한 시행으로 확률을 계산, 임의성에 기반함
  - 그러나 빈도가 매우 작은 사건의 확률을 정의하기 어려움
  - 동전을 무한히 던져서 절반이 앞면이므로, 앞면이 나올 확률은 0.5이다.
- 베이지안 확률
  - 주관적인 신뢰도를 기반으로 확률을 정함
  - 동전은 앞뒤가 동일한 넓이이므로, “앞면이 나왔다”는 주장의 신뢰도가 0.5이다
  - 가설에 대한 확률도 정할 수 있다. (베이지안 추정)
  - 빈도가 낮은 사건의 확률을 정의할 수 있다. (주관적 신뢰도이므로)

# 표본공간, 확률변수, 확률함수, 확률분포

- 표본공간: 실험으로 부터 나온 시행들의 경우의 수
- 확률변수: 표본공간의 Element를 실수로 변환
- 확률함수: 확률 변수를 0과 1사이의 실수로 변환
- 확률분포: 확률 함수로부터 나오는 확률들의 전반적인 패턴

# 통계량

- 평균, 편차, 분산, 표준편차와 같은 것들
  - 표준편차는 평균과 같은 단위로 다룰 수 있음
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
  - 근데 그냥 테이블 그리면 풀기는 쉬움, 전체 개수를 임의 값으로 두어야함
  - $P(B_x \vert A) = \dfrac {P(B_x,A)} {P(A)}=\dfrac {P(A \vert B_x)P(B_x)} {\sum_{i=1}^n P(A \vert B_i)P(B_i)}$

# 독립, 조건부 독립

- 독립: 확률의 곱이 동시에 일어날 확률과 같음
  - $P(A,B) = P(A)P(B)$
  - $P(A \vert B) = P(A)$
- 조건부 독립: C가 일어났을때의 독립
  - $P(A,B \vert C) = P(A \vert C)P(B \vert C)$
  - $P(A \vert B,C) = P(A \vert C)$

# 이산확률분포

- 베르누이 분포
  - p와 1-p의 확률을 가진 시행을 1번 진행
  - $f(x;p) = p^x(1-p)^{(1-x)}, \space x = 0,1$
  - $E[X] = p, \space V[X] = pq$
- 이항 분포
  - 여러번의 베르누이 시행을 반복, 각 시행은 독립
  - $f(x;N,p) = \dbinom{N}{x} p^x(1-p)^{N-x}, \space x = 0,1 \dots n$
  - $E[X] = np, \space V[X] = npq$
  - n이 크면 정규분포로 근사 가능
- 푸아송 분포
  - 단위 시간동안 특정 사건이 i번 발생할 확률
  - $\lambda$는 단위 시간동안 특정 사건이 평균 발생했던 횟수
  - $f(k;\lambda) = \dfrac{\lambda^k}{k!}  e^{-\lambda}, \space x = 0,1 \dots$
  - $E[X] = \lambda, \space V[X] =  \lambda$
  - 이항분포가 $n \rightarrow \infty, p \rightarrow 0$ 이면 푸아송 분포로 근사된다.
- 기하 분포
  - 첫 번재 성공이 일어날 때 까지의 시행횟수가 $x$일 확률
  - $f(x;p) = (1-p)^{x-1}p, \space x = 1,2 \dots$
  - $E[X] = 1/p, \space V[X] = (1-p) / p^2$
  - 무기억성
    - 이전에 얼마를 시행했던간에, 처음 시행하는거랑 똑같다.
    - $P(X=x+k \vert X>k) = P(X=x), \space x=1,2,\cdots$
- 음이항분포
  - r번째 성공이 일어날 때 까지 시행횟수가 $x$일 확률
  - $f(x;r,p) = \dbinom{x-1}{r-1}p^r (1-p)^{x-r},\space x=r, r+1, \cdots$
  - $E[X] = r/p, \space V[X] = r(1-p) / p^2$
  - 기하분포의 평균 분산에다가 r만 곱해놓은 꼴
- 초기하분포
  - $N$개의 공에서 $M$이 하얀색, $N-M$이 빨간색이라면, $K$개를 비복원추출했을때 $x$개가 흰색일 확률
  - $f(x;N,M,K) =\dfrac{\binom{M}{x} \binom{N-M}{K-x}}{\binom{N}{K}}$
  - $x=max\lbrace 0, M-N+K\rbrace, \cdots , min\lbrace K,M\rbrace$
  - $E[X] = K \dfrac{M}{N}, \space V[X] = K \dfrac{M}{N} \dfrac{N-M}{N} \dfrac{N-K}{N-1}$
  - 평균과 분산은 이항분포를 떠올리면 좋음, $K = n$, $\frac{M}{N} = p$, $\frac{N-M}{N} = 1-p$에 해당.

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
  - $E[X] = 1/\lambda, \space V[X] = 1/\lambda^2$
- 감마분포
  - $\alpha$번의 사건이 발생하기 까지의 시간
  - $\alpha$는 형상모수(shape parameter), $\lambda$는 척도모수(rate parameter)임
  - $f(x;\alpha,\lambda) = \dfrac{\lambda^\alpha}{\Gamma(\alpha)}x^{a-1}e^{-\lambda x}, \space x > 0$
  - $\Gamma(\alpha)=\displaystyle\int_{0}^{\infty}t^{\alpha-1}e^{-t}dt$
  - 감마함수는 $n!$을 모든 실수 범위로 확장한 것.
  - 지수분포의 일반화 = $\alpha$개의 지수분포의 합은 감마분포임
    - $X = X_1 + X_2 + \cdots + X_\alpha$
    - $X_i$가 모두 같은 $\lambda$를 가지는 지수분포이고, 독립이면
    - X는 형상모수 $\alpha$를 가지는 감마분포.
  - $E[X] = \alpha/\lambda, \space V[X] = \alpha/\lambda^2$
- 베타분포
  - 비율과 관련된 분포(불량률, 불순률, 고장률)
  - $f(x;a,b) = \dfrac{\Gamma(a+b)}{\Gamma(a)\Gamma(b)} x^{a-1}(1-x)^{b-1}$
  - 베타함수
    - $B(a,b) =  \dfrac{\Gamma(a+b)}{\Gamma(a)\Gamma(b)}$
    - 교환 법칙: $B(a,b) = B(b,a)$
    - $B(1,b) = 1/b$
    - $B(a+1,b) = \dfrac{a}{a+b} B(a,b)$
  - $E[X] = \dfrac{a}{a+b}, \space V[X] = \dfrac{ab}{(a+b)^2(a+b+1)}$
  - $a == b$인 경우 대칭임

# 결합확률변수

- 확률변수: 표본공간에서 하나의 실수로 매핑
- 결합확률변수: 표본공간에서 여러개의 실수로 매핑
- 결합확률함수
  - 여러개의 확률변수를 하나의 확률(0과 1사이의 실수)로 매핑
  - $f_{XY}(x,y) = P[X = x, Y = y]$
  - 이산이면 sum으로, 연속이면 적분으로 확률을 구함
- 주변확률함수(Marginal Probability Function)
  - 결합확률함수에서, 특정 변수로 합/적분하고 난 이후의 함수
- 주변확률분포(Marginal Probability Distribution)
  - 결합확률분포에서 하나의 변수를 무한대로 보내버림
  - $F_X(a) = \displaystyle\lim_{b \rightarrow \infty} F_{XY}(a, b)$
- 두 확률변수의 합 $X + Y$
  - 합의 확률변수는 각 변수의 합성곱(Convolution)
  - $f_{X+Y}(a) = \displaystyle\int_{-\infty}^{\infty} f_X(a-y)f_Y(a) dy$
  - 지수분포의 합은 감마분포, 개수가 형상모수가 됨
  - 감마분포의 합은 감마분포, 두 형상모수의 합이 형상모수가 됨
  - 졍규분포의 합은 정규분포, 평균은 두 평균의 합, 분산은 두 분산의 합
  - 졍규분포의 차은 정규분포, 평균은 두 평균의 차, 분산은 두 분산의 **합**
  - 푸아송 분포의 합은 푸아송분포, $\lambda$는 두 $\lambda$의 합임
  - 이항분포의 합은 이항분포

# 공분산과 상관관계

- 공분산은 두 확률변수의 선형관계를 나타냄
- 독립과 공분산
  - 독립이면 공분산이 0, (독립이면 선형관계가 없음)
  - 공분산이 0이여도 독립인것은 아님.
- $Cov(X,Y) = E[XY] - E[X]E[Y]$
  - 위 식으로 분산 공식도 유도 가능 $V[X] = E[X^2] - E[X]^2$
- 공분산의 성질
  - $Cov(X,Y) = Cov(Y,X)$
  - $Cov(X,X) = V[X]$
  - $Cov(aX,Y) = aCov(X,Y)$
  - $Cov(X+Y,Z) = Cov(X,Z) + COV(Y,Z)$
  - $Cov(\sum_i{X_i},\sum_j{Y_j}) = \sum_i \sum_j Cov(X_i, Y_j)$
  - $V[X_1 + X_2] = V[X_1] + 2Cov(X_1, X_2) + V[X_2]$
  - 두 변수가 독립이면, $V[X_1 + X_2] = V[X_1] + V[X_2]$
- 상관계수, 선형관계를 나타내면서 $[-1, 1]$의 범위를 지님
  - $\rho(X,Y) = \dfrac{Cov(X,Y)}{\sqrt{V[X]V[Y]}}$
  - 1에 가까우면 양의 상관관계
  - -1에 가까우면 음의 상관관계
- 기대값, 분산, 왜도, 첨도
  - $E[X]$: 평균, 기댓값
  - $E[X^2]$: 분산, 퍼진 정도
  - $E[X^3]$: 왜도
    - 양수면 좌로 기운 분포
    - 음수면 우로 기운 분포
  - $E[X^4]$: 첨도
    - 0이면 정규분포
    - 양수면 더 뾰족하고 꼬리가 정규분포보다 두꺼움
    - 음수면 더 뭉툭하고 꼬리가 정규분포보다 얇거나 없음