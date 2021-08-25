---
layout: post
category: development
title: Modern Java in Action 정리 
tags: [Java]
---

# Chaper1. Introduction

- Java 8 = 함수형 프로그래밍으로의 변화
- 아래는 주요 5가지로 정리.

## Behavior parameterization

- 기존 자바의 함수는 first-class object가 아님 = 값으로 다룰 수 없음
  - 동작을 인자로 넣고 싶을때..
    - Java8 이전에는 Enum으로 다 구현하거나
    - Anonymous Class를 사용해야했음
- 자바8부터 함수를 대입할 수 있게 변경
- 기존처럼 추상 클래스를 이용할 필요 없음
- Lambda, Method 참조 등으로 구현해냄
- `Predicate<T>` 타입 등의 Functional Interface를 통해서 함수를 대입받음

## Stream API

- 파이프라이닝, Concurrency를 손쉽게 추구
- Concurrency의 선행조건
  - Behavior parameterization이 선행조건
  - No shared mutable data (No side-effect = pure function)
- 특정 동작들은 `ParallelStream`을 쓰면 병렬 처리가능
  - Filtering(조건에 만족하는것만 분리)
  - Groupping(두 그룹으로 분리)
  - Extracting(일부 필드 뽑기)

## 나머지

- Default Method
  - 기존 class 구현을 변경하지 않고도, Interface에 함수를 추가하고 싶다.
  - 인터페이스에 함수 구현을 추가해버리자.
    - class에서 오버라이딩 하지 않아도 무방하고
    - 설사 겹치더라도, 클래스의 구현이 우선시됨
- Module
  - 패키지 단위의 가시성 지원이 되지 않아 생긴 개념
  - 특정 Package에만 private하게 지정할 수 있는 기능이 없었음
- FP적인 요소들
  - Optional, Reactive, Future 등이 도입됨.

# Chapter2. Behavior Parameterization

