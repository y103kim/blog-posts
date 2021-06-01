---
layout: post
title: 'Coursera Deeplearning 정리'
tags: [ML, Deeplearning]
---

# Introduction

5개 Section을 배움

1. NN / DL의 기초
2. DNN 개선: Hyperparameter 튜닝, 정규화, 최적화
3. ML 프로젝트의 구조
4. CNN
5. 자연어처리

신경망이란 무엇인가?

- ReLU = Rectified Linear Unit
  - $y = d(x-k) (x >= k)$ or $y = 0 (x < k)$ 형태의 함수
- 이런 Mapping 들이 하나의 Neuron 에 대응됨. Unit 이라고 함.
- 여러 Layer로 둘 수 있음.

# Supervised Learning

Training Set을 가지고 훈련시키기

종류를 구분해보기

- Standard NN
  - Homefeature --> Price
  - Ad, user info --> click on ad?
- CNN
  - Image, Object (1...1000) Photo tagging
- RNN(Recurrent Neural Network) Sequential Data 에 유용
  - Audio --> Speech Recognition
  - 번역 (좀더 복잡한 버전의 RNN)

데이터 종류에 따른 구분

- Table - Structured Data
- Audio, Image, Text - Unstructured Data

왜 Deep Learning이 잘 되는가?

- Data 커짐, Computation이 빨라짐, 알고리즘이 좋아짐
- Data가 작은 경우는 직관, 엔지니어의 실력이 더 크게 반영됨


# Logistic Regerssion

선형회귀는 Binary Classification(이진분류)를 위한 알고리즘 (예를들면 Image to cat vs non-cat)

입력

- cat or non-cat image
- image = 64 * 64 * 3 (RGB) = 12288 (Input Vector)

Notation

- 입력, 출력 pair = $(x,y)$
- m개의 training set = $\\{(x^{(1)}, y^{(1)}), (x^{(2)}, y^{(2)})\\}$
- Training set의 수 = $m_{test}$
- Input Matrix = $X = [x^{(1)}, ..., x^{(m)}]$
- Output Matrix = $Y = [y^{(1)}, ..., y^{(m)}]$

Logistic Regression

- Sigmoid Function = $$\sigma (z) = \frac{1}{1+e^{-z}}$$
- Sigmoid Function은 -Inf에서 0, Inf에서 1이 되는 함수, 좌우 대칭함수
- $\hat{y} = P(y=1\|x)$
- $w \in \mathbb{R}^{n_{x}}$, $b \in \mathbb{R}$
- $\hat{y} = \sigma ( w^{T}x + b )$
- 목표는 $w$를 잘 만들어서 x에 따른 y가 정확하게 나오도록 하는것.

Logistic Regression Cost Function

- Given $\\{(x^{(1)}, y^{(1)}), ..., (x^{(m)}, y^{(m)})\\}$, want $\hat{y}^{(m)} == y^{(i)}$
- Loss (error) function: $(L(\hat{y}, y) = -(y \log \hat{y} + (1-y) \log{(1-\hat{y})})$
- Loss Function은 $\hat{y}$가 $y$에서 벗어날 수록 커진다.











