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

- `List<Apple>`에서 녹색사과 고르기 예제
  - 조건을 파라미터화 하거나 일일이 구현해 주는것이 코드 복잡성을 증가시킴
  - 동작을 클래스 혹은 익명클래스로 넘겨도 너무 반복되는 코드가 많음(Verbosity)
  - 결국 람다나, 메소드 참조로 해결하게 됨.
- 실제로 `Testable`, `Runnable`, `Callable`등을 쓸 때 람다를 넘기면 유용함.

# Chapter3. Lambda Expression

- 기본적으로 다른 언어에서 사용되는 람다와 거의 같음
  - 다만 외부변수는 `final` 혹은 실질적 `final` 상태이어야 함
  - 즉 외부 변수로 참조하고 나서 값이 바뀌면 안됨.
    ```
    int p = 1;
    Runnable r = () -> System.out.println(p);
    p = 2;
    r.run();
    ```
- Functional Interface에 대입할 수 있음
  - 추상 메서드가 하나인 인터페이스
  - 단 디폴트 메서드는 함께 사용할 수 있음(추상이 아니므로)
  - `@FunctionalInterface`를 붙여주면 컴파일러가 유효성을 검사함
  - 람다의 타입을 추상메소드(Functional Descriptor)와 맞추어야함
  - 추상 메소드에 타입이 명기되어 있다면, 람다의 파라미터 타입을 생략해도 무방
    ```
    Consumer<Apple> consumer = (a) => {}
    ```
- 대표적인 Functional Interface
  - `Predicate<T>` = `(T) -> boolean`
  - `Consumer<T>` = `(T) -> void`
  - `Function<T, R>` = `(T) -> R`
  - `Supplier<T>` = `() -> T`
  - `UnaryOperator<T>` = `(T) -> T`
  - `BinaryOperator<T>` = `(T, T) -> T`
  - `BiPredicate<T, U>` = `(T, U) -> boolean`
  - `BiConsumer<T, U>` = `(T, U) -> void`
  - `BiFunction<T, U, R>` = `(T, U) -> R`
- Method Reference
  - 람다의 축약형 `Apple::getWeight` = `(Apple a) -> a.getWeight`
  - 목적: 가독성 향상
  - 생성자도 참조 가능
  - Functional Interface의 comparing + Method Reference
    ```java
    inventory.sort(Comparator.comparing(Apple::getWeight));
    ```
  - 3가지 방식의 Method Reference
    - `(args) -> ClassName.staticMethod(args)` = `ClassName::staticMethod`
    - `(arg, rest) -> arg0.instanceMethod(rest)` = `ClassName::instanceMethod`
    - `(args) -> expr.instanceMethod(args)` = `expr::instanceMethod`
  - `Default Method`로 선언된 여러 메소드를 이용 가능
    - 역순: `list.sort(comparing(Apple::getWeight).reversed())`
    - 조건추가: `list.sort(comparing(Apple::getWeight).thenComparing(Apple::getCountry))`
    - 논리연산: `redApple.negate().and(apple -> apple.getWeight() > 150)`
    - 합성: g(f(x)) = `f.andThen(g)` / f(g(x)) = `f.compose(g)`
  
