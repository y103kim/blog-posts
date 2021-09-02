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
  
# Chapter4. Introducing Stream

- Iterator + Parallelism in **Declarative way**
  - 선언적: 동작을 기술하고, 구현은 감춘다. `string.replace(/ /g, '-')`
  - Threading과 Query를 분리한다. (DB 적인 접근) - 병렬화 쉬움
  - Composable = 이어붙이기 쉽다.
- 정의 : "a sequence of elements from a source that supports data-processing operations"
  - 소스(Collections, arrays, I/O) -> 연속된 요소 -> 데이터 처리 동작(내부 루프) -> 파이프라이닝
- Stream vs Collections
  - Collection은 모든 요소를 메모리에 올림 / Stream은 필요한 요소만 메모리에 올림
    - python의 generator처럼 필요할때 데이터를 계산해서 내어줌
    - 다르게 표현하면: Lazily Constructed Collection
  - 한번만 탐색 가능 iterator과 유사, 정확한 표현으로는 일회용
  - Stream은 내부반복임
- Stream 연산들
  - 중간연산: 또다른 스트림을 리턴, 실제로 연산이 일어나지 않음(Lazy).
  - 최종연산: 스트림이 아닌 다른 요소를 모아서 리턴, 연산이 일어남.

# Chapter5. Working with streams

- 중간 연산
  - 필터링: `filter(Predicate)`, `distinct()`
  - 슬라이싱: `takeWhile(Predicate)`, `dropWhile(Predicate)`, `limit(int)`, `skip(int)`
  - 매핑: `map(A -> B)`, `flatmap(X -> stream<Y>)`
- 최종 연산
  - 매칭: `allMatch(Predicate)`, `anyMatch`, `NoneMatch`
  - 검색(Optional 리턴): `findAny`, `findFirst`
  - 리듀스: `T reduce(T, (T,T) -> T)`, `Optional<T> reduce((T,T) -> T)`
  - 최대, 최소: `Optional<T> max((T,T) -> T)`
  - 다음 장에서 다룰 최종연산들: `collect`, `reduce`, `count`, `forEach`
- 상태에 따른 분류
  - Stateful: `distinct`, `skip`, `limit`, `sorted`, `reduce`
  - Stateless: 나머지
- 숫자형 스트림
  - 변환: `mapToInt`, `mapToDouble`, `mapToLong`
  - 스트림: `IntStream`, `DoubleStream`, `LongStream`
  - Optional로 반환될 경우: `OptionalInt`, `OptionalDouble`, `OptionalLong`
  - 다시 일반 Stream으로 변환: `boxed()`
  - 생성: `IntStream.rangeClosed(1, 100)`
- 스트림 생성하기
  - 값으로 부터 자동 생성 `Stream.of`, `Arrays.stream`
  - Iterate
    - `Stream.iterate(T, T -> T)`: 무한 실행(limit으로 제한 가능)
    - `Stream.iterate(T, T -> boolean, T -> T)`: 조건까지 실행
    - Iterate는 Immutable
  - Generate
    - `Stream.generate(() -> T)`
    - generate는 mutable도 가능.

# Chapter6. Collecting data with stream

- `Collector`: `collect` 함수의 인자 타입으로 주어지는 Functional Interface
- `Collectors`: 팩토리 클래스, Collector를 반환하는 정적함수를 제공
- `downstream`: 연쇄 호출
  - `collectingAndThen`: Collector를 실행한 이후 Function을 연쇄
  - `groupingBy`: classifier로 분류 한 이후, 또 다른 Collector를 연쇄
  - `mapping`: mapper로 매핑 후, 또 다른 Collector를 연쇄
  - `partitoningBy`: predicator를 기준으로 쪼갠 여러개를, Collector로 연쇄

## Why factory design used in Collectors

- 모듈화를 위해서!
- 먼저 아래에서 공통을 쏘이는 타입들.
  - `T`: Entity, 입력 스트림의 타입
  - `A`: Accumulated, 중간 누적 타입
  - `R`: 리턴 타입
- 아래 메소드들 정의하면 `Collector`를 만드는 형태를 모듈화 할 수 있음
  - `supplier`: 새 컨테이너 생성 `() -> A` 
  - `accumulator`: 컨테이너에 요소 추가 `(A, T) -> A`
  - `finisher`: 마지막 변환에 사용 `A -> R`
- 추가로 정의해 병렬화에 이득을 볼 수 있는 메소드(분할 정복 이용함)
  - `combiner`: 컨테이너 합치기 `(A, A) -> A`
  - `characteristics`: 셋중에 하나를 리턴
    - `UNORDERED`: 누적, 방문의 순서가, Reduction 결과에 영향을 주지 않음
    - `CONCURRENT`: 병행 처리를 해도 무방함.
    - `IDENTIFY_FINISH`: `A == R`이므로 바로 리턴해도 무방.
    - 단 정렬된 스트림은, 병행화시 `UNORDERED`, `CONCURRENT` 둘다 필요.

## Collectors: Sum or Reduce

- count: `counting()`
- Min,Max: `maxBy(Comparator)`
- Sum: `summingInt(T -> int)`
- Average: `averagingInt(T -> int)`
- Statistics: `summarizing(T -> int)`
- Joining Strings: `joining()`, `joining(", ")`
- General reducer
  - `Optional<T>` 리턴: `reducing((T,T)->T)`
  - 초기값 포함, `T` 리턴: `reducing(T, (T,T)->T)`
  - Mapper까지 포함, `T` 리턴: `reducing(U, T->U, (U,U)->U))`

## Collectors: Group

- 기본 사용: `groupingBy(T->K)`
  - 다음과 동작이 같음 `groupingBy(T->K, toList())`
- 확장: `groupingBy(T->K, downtream)`
- `groupingBy`를 여러번 중첩시키면 다중 분할이 가능함
- `groupingBy`에 `count()`를 중첩시키면 하위 그룹의 개수 셀 수 있음.
- `partitioningBy`는 `groupingBy`의 특수한 케이스
  - true, false 두가지로 묶어줌

# 추가 조사: reduce vs collect

- Source: [Part1](https://youtu.be/oWlWEKNM5Aw), [Part2](https://youtu.be/H7VbRz9aj7c)
- 일반적인 프로그래밍 패러다임
  - functional = immutable types
  - imperative = mutable types
- Java has **mutable reduction**, **stateful operations** 
  - becuase Java is OOP
  - not classical function programming reduction
- 몇가지 용어
  - associatvie: 결합 법칙을 준수한다.
  - stateless: 람다는 클로저로서 받아온 객체를 수정하지 말아야 한다.
  - non-interfering: input으로 들어온 source를 수정하지 않는다.
- `reduce()` vs `collect()`
  - 두 함수은 거의 비슷함
  - `accumlator`과 `combiner` 모두: associatve, non-interfering, stateless
  - 둘다 입력 스트림을 고치면 안됨! (non-interfering)
  - 두 함수는 호출 순서가 보존됨.
  - reduce
    - immutable reduction: 중간 결과도 고치면 안됨
    - 1st arg: `identity` = 연산의 정의역 **값**, 모든 연산이 공유함, 즉 연산 중에 절대 변하면 안됨
    - 2nd arg: `accumulator` = `Bifunction`, 새로운 값을 만들어서 리턴해야함
    - 3rd arg: `combiner` = `BinaryOperator`, 새로운 값을 만들어서 리턴해야함
  - collect
    - mutable reduction: 중간 결과를 고칠 수 있음
    - 1st arg: `supplier` = `Supplier<R>`, 빈 컨테이너를 생성하는 함수
    - 2nd arg: `accumulator` = `BiConsumer`, 리턴이 없음, 중간 결과(첫번째 인자)를 고쳐야 함
    - 3rd arg: `combiner` = `BiConsumer`, 리턴이 없음, 중간 결과(첫번째 인자)를 고쳐야 함
- string concat은 매번 copy하므로 매우 느림
  - reduce를 통해 immutable type인 `String`을 concat하면 매우 느림
  - collect를 통해 mutable type인 `StringBuilder`로 append하면 빠름

# 추가 조사: forEach, forEachOrdered, peek

- `forEach`, `forEachOrdered`, `peek`은 stateful한 lambda를 입력으로 받을 수 있음.
- 다른 모든 Stream API는 stateless를 요구함

# Chapter7. Parallel streams

