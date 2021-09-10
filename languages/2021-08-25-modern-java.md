---
layout: post
category: languages 
title: Modern Java in Action 정리 
tags: [Java]
---

# Chapter 1. Introduction

- Java 8 = 함수형 프로그래밍으로의 변화
- 아래는 주요 5가지로 정리.

### Behavior parameterization

- 기존 자바의 함수는 first-class object가 아님 = 값으로 다룰 수 없음
  - 동작을 인자로 넣고 싶을때..
    - Java8 이전에는 Enum으로 다 구현하거나
    - Anonymous Class를 사용해야했음
- 자바8부터 함수를 대입할 수 있게 변경
- 기존처럼 추상 클래스를 이용할 필요 없음
- Lambda, Method 참조 등으로 구현해냄
- `Predicate<T>` 타입 등의 Functional Interface를 통해서 함수를 대입받음

### Stream API

- 파이프라이닝, Concurrency를 손쉽게 추구
- Concurrency의 선행조건
  - Behavior parameterization이 선행조건
  - No shared mutable data (No side-effect = pure function)
- 특정 동작들은 `ParallelStream`을 쓰면 병렬 처리가능
  - Filtering(조건에 만족하는것만 분리)
  - Groupping(두 그룹으로 분리)
  - Extracting(일부 필드 뽑기)

### 나머지

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

# Chapter 2. Behavior Parameterization

- `List<Apple>`에서 녹색사과 고르기 예제
  - 조건을 파라미터화 하거나 일일이 구현해 주는것이 코드 복잡성을 증가시킴
  - 동작을 클래스 혹은 익명클래스로 넘겨도 너무 반복되는 코드가 많음(Verbosity)
  - 결국 람다나, 메소드 참조로 해결하게 됨.
- 실제로 `Testable`, `Runnable`, `Callable`등을 쓸 때 람다를 넘기면 유용함.

# Chapter 3. Lambda Expression

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
  
# Chapter 4. Introducing Stream

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

# Chapter 5. Working with streams

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

# Chapter 6. Collecting data with stream

- `Collector`: `collect` 함수의 인자 타입으로 주어지는 Functional Interface
- `Collectors`: 팩토리 클래스, Collector를 반환하는 정적함수를 제공
- `downstream`: 연쇄 호출
  - `collectingAndThen`: Collector를 실행한 이후 Function을 연쇄
  - `groupingBy`: classifier로 분류 한 이후, 또 다른 Collector를 연쇄
  - `mapping`: mapper로 매핑 후, 또 다른 Collector를 연쇄
  - `partitoningBy`: predicator를 기준으로 쪼갠 여러개를, Collector로 연쇄

### Why factory design used in Collectors

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

### Collectors: Sum or Reduce

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

### Collectors: Group

- 기본 사용: `groupingBy(T->K)`
  - 다음과 동작이 같음 `groupingBy(T->K, toList())`
- 확장: `groupingBy(T->K, downtream)`
- `groupingBy`를 여러번 중첩시키면 다중 분할이 가능함
- `groupingBy`에 `count()`를 중첩시키면 하위 그룹의 개수 셀 수 있음.
- `partitioningBy`는 `groupingBy`의 특수한 케이스
  - true, false 두가지로 묶어줌

### 추가 조사: reduce vs collect

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

### 추가 조사: forEach, forEachOrdered, peek

- `forEach`, `forEachOrdered`, `peek`은 stateful한 lambda를 입력으로 받을 수 있음.
- 다른 모든 Stream API는 stateless를 요구함

# Chapter 7. Parallel streams

- `parallel()`, `sequential()`로 스트림의 병렬성을 조절할 수 있음
- Boolean flag여서, 마지막으로 호출된 것이 최종 효력을 발휘함.
- 병렬이 항상 빠른것은 아니다.
  - 병렬화 가능한 연산인지
  - Shared State는 없는지
  - 병렬화/쓰레드간 데이터 전송 비용에 비해 이득이 있는지
- 중점적으로 볼 사항들.
  - 순차/병렬 시간을 무조건 측정해 보아야 한다.
  - Boxing에 드는 비용을 고려해야한다.
  - 순서에 의존적인 `limit`과 `findFirst`는 거의 병렬이 느리다.
  - `unordered()`를 부른 상태에서는 `limit`등이 훨씬 효율적이다.
  - N의 수가 커야 병렬이 효율적이다.
  - `SIZED` 속성의 스트림은 스트림을 쪼갤 수 있어서 효율적일 수 있다.

### The fork/join framework

- `RecursiveTask<R>`을 상속받아서 `coumpute`함수를 구현
  - 일반적인 재귀 함수처럼, 종료조건, 분할후 재귀호출로 짜면 됨
    - 먼저 종료조건을 먼저 체크해서, 만족하는 경우 순차적으로 처리 후 리턴
    - 반으로 나눠서 두 `RecursiveTask`를 생성한 뒤에
    - 왼쪽은 `fork()`해주고
    - 오른쪽은 바로 `compute`해서 계산
    - 이후 왼쪽 Task가 완료될 때 까지 `join()`으로 기다림
    - 양쪽 결과를 합쳐서 리턴하면 끝.
- 최종 실행시 `ForkJoinPool().invoke(task)`로 실행하면 됨.
  - 일반적으로 `ForkJoinPool`은 한번만 static으로 생성함.
- Work Stealing
  - Thread 하나가 작업을 일찍 끝냈는데, 큐에 담긴 일이 없을 경우
  - 다른 Thread의 큐에서 작업을 빼와서 실행한다.
  - 때문에 약간 작은 Task 여러개로 나누는것이 균형잡힌 실행에 도움이 됨

### Spliterator

- 4개의 함수를 정의하면 구현할 수 있는 인터페이스
  - `tryAdvance`: 순차처리 이후, 처리할 것이 남아있으면 `true`리턴
  - `trySplit`: 현재 객체를 쪼갤 수 있으면, 생성해 리턴, 아니면 `null`리턴
  - `estimateSize`: 현재 객체가 포함하는 요소의 사이즈
  - `characteristics`: 다양한 특성 정의, Oring으로 쓸 수 있는 정수 플래그
- Collection이 Spliterator 인것을 생각하면 
  - 그 자체로 Container적인 속성을 띄어야 함
  - 즉 데이터를 포함하고 있어야 하며, 위 4개 함수를 구현해야함

# Chapter 8. Collection API enhancements

- Before Java9
  - `Array.asList`: immutable list
  - No `asSet` function
- Collection factories in Java9
  - `List.of`: immutable list
  - `Set.of`: immutable set 
  - `Map.of`: immutable map 
- Functions for List and set
  - Collection 자체를 바꾸는 함수들로, immutable에 쓸 경우 예외 발생
  - `removeIf`: Predicate를 만족하는 요소 삭제
  - `replaceAll`: 각 요소들을 함수에 넣어서 변경함
- Functions for Map
  - `forEach`: k,v를 입력으로 받는 람다 실행, 리턴은 없음
  - `entrySet`: k,v pair를 set으로 묶어서 리턴
  - `getOrDefault`: python의 get과 동일
  - `computeIfAbsent`: key 없으면 `K->V`로 set해줌
  - `computeIfPresent`: key 있으면 `(K,V)->V`로 값 바꿈
  - `compute`: 람다((K,V)->V)로 값 바꿈, 없으면 value로는 `null`들어옴
  - `remove`: key와 value 모두 매칭될 때 삭제하는 기능 있음
  - `replaceAll`: 모든 값을 바꿈
  - `putAll`: 다른 map을 다 밀어넣음(덮어씀)
  - `merge`: key와 value를 set할때, 겹치면 `(V,V)->V`를 불러줌

### 추가조사: Java의 Collection들

- 기본 사항들
  - `Concurrent`: 읽기에는 Lock을 걸지 않고 쓰기 시 해당 자원(버킷)에만 Lock을 건다.
  - `CopyOnWrite`: Fork시에 대상 쓰레드로 카피하지 않고, 실제 쓰기가 일어날때만 카피한다. 읽기에 유리함.
- List
  - `ArrayList`, `CopyOnWriteArrayList`: c++ vector처럼 가변 길이 배열
  - `LinkedList`: 연결리스트
- Set/Map
  - `HashSet/Map`,`ConcurrentHashMap`: Hash Table 기반
  - `LinkedHashSet/Map`: 순서 보존하는 Hash Table 기반
  - `EnumSet/Map`: Bit Vector 기반
  - `TreeSet/Map`: RB-tree 기반
  - `CopyOnWriteArraySet`: contains이 `O(n)`, 쓰레드에서 iterative하게 읽을때 유용
  - `ConcurrentSkipListSet/Map`: 2의 배수 크기마다 skip-node가 생기는 스킵리스트
  - `IdentityHashMap`: hash지정 규칙 재정의 가능
  - `WeakHashMap`: K,V가 임의로 GC 될 수 있음
- Queue
  - `PriorityQueue`: binary heap을 이용, 삭제가 `O(n)`이므로 주의
  - `ArrayDequeue`: 가변길이 배열을 이용한 덱인듯, 삭제가 `O(n)`이므로 주의 
  - `ConcurrentLinkedQueue`: Exactly-once 읽기가 불가능
  - `ArrayBlockingQueue`: Exactly-once 읽기가 가능, 그러나 읽기시에도 락검
  - `PriorirityBlockingQueue`: 마찬가지로, 쓰기 읽기 모두 락 걸고 동작
  - `LinkedBlockingQueue`: 마찬가지로, 쓰기 읽기 모두 락 걸고 동작
  - `SynchronousQueue`: 아예 버퍼가 없어서, 쓰는쪽이 읽어갈때 까지 polling
  - `DelayQeue`: 시간을 지연시킬 수 있는 큐, 시간을 키 값으로 한 Heap을 베이스로 함


# Chapter 9. Refactoring, testing, and debugging

- Refactoring: 3 ways
  - anonymous classes -> lambdas
  - lambdas -> method references
  - imperative-style -> streams
- improving code flexibility
  - Functional interface 사용
  - Conditional deferred execution
    - If문을 함수로 분리하고, 람다를 보내기


### refactoring OOP design patterns

- strategy pattern
  - 간략한 정리
    - 행위를 추상/구체화하는 패턴
    - concrete strategy class를 정의해서 사용
  - java8에서는 lambda로 행동을 정의해서 바로 넣어주면 됨
- template method pattern
  - 간략한 정리
    - abstract class가 대부분의 구현을 가지고 있고,
    - concrete class는 일부 추상 메소드만 구현
  - Java8에서는 상속받지 않고, 그냥 람다를 넘겨주는 식으로 구현할 수 있다.
- observer pattern
  - 간략한 정리
    - 이벤트가 일어나는 것을 감시하는 패턴
    - Observer 인터페이스는 notfiy를 구현할 것을 강제함
    - ConcreteObserver는 각각 notfiy를 구현해 이벤트 발생시 동작을 정의
    - Subject클래스에 여러 Observer 인스턴스를 등록
    - 이벤트 발생시 모든 Observer를 순회하면 notify를 호출
  - Java8에서는 Observer 자체를 lambda로 바꿀 수 있다.
- Chain of responsibility pattern
  - 간략한 정리
    - 어떤 문제에 대해 처리한 후, 나머지를 등록된 다음 클래스로 넘김
    - 약간 함수형 프로그래밍 비슷해짐
  - `UnaryOperator`등의 Interface에 정의된 `andThen`에 람다들을 넘겨서 연쇄
- Factory 패턴
  - 간략한 정리
    - 함수 호출과 인자 구성으로 서로 다른 객체를 생성함
  - Java8에서는 map에 람다를 값으로 집어넣어 구성할 수도 있다.

### Testing and debugging

- 람다 테스트 방법
  - 변수로 따로 저장해서 테스트 하거나
  - 람다를 사용하는 메서드를 검증하거나
  - 복잡한 람다를 몇몇 메서드로 나누거나
- 디버깅
  - 람다는 익명이라, 함수이름이 안나와서 디버깅에 어려움이 있음
  - 메소드 래퍼런스를 쓰면 이름이 나오므로 권장됨
  - 스트림의 디버깅시에는 peek을 써주면 유용함

# Chapter 10. Domain-specific languages using lambdas

- DSL: 특정 도메인을 위해 만들어진 언어(Ant, Maven, HTML)
  - 읽기 쉽고, 이해하기 쉽게: 여러번 읽혀질 것을 염두해 두고 디자인됨 
- 장점
  - Conciseness, Redability, Maintainability
  - 높은 수준의 추상화, 관심의 분리
- 단점
  - Difficulty of DSL Design, Development cost
  - 부가적인 레이어 생김, 학습 부담, 프로그래밍 언어적 제한
- 종류
  - Internal DSL: Java로 직접 DSL 작성
  - Polyglot DSL: JVM에서 도는 다른 언어들(Scala, Kotlin)로 DSL 작성
  - External DSL: 완전히 다른 새로운 언어로 DSL을 제공
- DSL 패턴
  - Method Chaining
    - 인자를 늘리는 대신, 전치사나, 동작 하나하나를 연쇄된 함수로 제공
    - 쓰는 입장에서는, 읽기 편하고, 분명한 코드를 작성할 수 있음.
    - builder를 제공하는 쪽에서는 복잡하고 많은 코드를 작성해주어야함
  - Nested Function 
    - 연쇄 체인이 리턴해 주는 대신, 사용자가 직접 중첩해서 사용하게 함
    - 괄호가 좀 더 많아지고, 파라미터를 추가해주기 어려움
    - postional argument를 사용자가 다 읽혀야함, IDE 도움을 받기 어려움
    - 구조가 더 잘 드러나고, 논리적으로 도메인 객체를 이해하기 쉬워짐
  - Lambda Sequencing
    - chain을 제공하되, 람다 함수의 입력인자로 제공함
    - 파라미터 추가도 용이하고, 사용자가 좀더 유연하게 행동을 바꿀 수 있음
  - 위 3가지를 적절히 조합해서 쓸 수도 있음

# Chapter 11. Using Optional as a better alternative to null

- `null`처리는 코드를 지저분하게 만들고, 실수할 여지가 크다.
  ```java
  // Dangerous for null exceptions
  person.getCar().getName();
  // Safe by Optional
  person.flatMap(Person::getCar).map(Car::getName).orElse("Unknown")
  ```
- 여러가지 해법들
  - `?.`처럼 safe navigation operation 이용
  - `Maybe`등 Wrapping 타입을 도입
- `Optional`
  - 값이 있을 경우: `Optional<Car>` 오브젝트를 Optional이 wrapping한다.
  - 값이 없을 경우: `Optional.empty()` 결과인 싱글톤 객체를 리턴한다.
- 장점들
  - 값이 비어있을 수 있다는 문맥적인 표현을 코드로 할 수 있다.
  - Optional이 안쓰인다면, 무조건 값이 있다는 인상을 주게 사용할 수 있다.
- 단점들
  - Wrapping 비용이 늘어나고
  - Serializable 하지 않다.
  - Primitive Optional은 여러 제약이 많아서 사용을 추천하지 않는다.
- 생성
  - `Optional.empty()`
  - `Optional.of(car)`
  - `Optional.ofNullable(car)`
- 값 읽기
  - `optInstance.map(...)`: `Optional` 타입으로 리턴함
  - `optInstance.flatMap(...)`: Optional이 중첩될 경우
  - `orElse`: 값이 없을 경우를 정해줌으로서, optional을 벗길 수 있음
  - 스트림에 `flatMap(Optional::stream)`을 쓰면, 빈 요소 제외 후 값 활용 가능
  - `get`:값이 없을 때 예외가 발생하므로, 주의해야함
  - `orElseGet`: 값 대신 람다(Suplier)를 넘길 수 있음
  - `orElseThrow`: 값 없을 때 명시적 예외 발생, 예외 생성 람다 넘길수 있음
  - `ifPresend`: 값이 존재할 때만 람다(Consumer) 수행
  - `ifPresendOrElse`: 값에 존재 유무에 따라, 람다 수행
  - `filter`: 값이 없거나, 조건을 만족하지 못하면, `Optional.empty()`

# Chapter 12. New Date and Time API

- Java8 이전의 Date API가 매우 좋지 못했음
- Java8 이후 합리적으로 라이브러리가 개선되었음
- 타임존 정보 없는 날짜/시각 객체 생성
  - `LocalDate.of`, `LocalTime.of`, `LocalDateTime.of`
  - `LocalDate.parse`, `LocalTime.parse`, `LocalDateTime.parse`
- 타임존 추가한 시각 객체 생성
  - 타임존 객체: `ZoneId.of("Europe/Rome")`
  - `ZonedDateTime.of`, `ZonedDateTime.parse`
  - 타임존 부여: `localDateTime.atZone(ZoneId)`
- 각 시각 객체들은 `now`도 지원함 
- `format`함수에 포맷터를 집어넣어서 출력 가능
- UTC기준 ns단위 타임스탬프는 `Instant` 객체를 사용
  - 생성: `Instanct.ofEpochSecond`, `Instanct.ofEpochMili`
  - 타임존 부여: `instant.atZone(ZoneId)`
- 시간 차이는 `Duration`, 날짜 차이는 `Period`
  - `Duration.between`, `Duration.ofDays`
- Date/Time 객체를 잠시 바꾸려면 `with`사용
  - `date.with(new NextWorkingDay())`

# Chapter 13. Default methods

- Default Method
  - 기존 class 구현을 변경하지 않고도, Interface에 함수를 추가하고 싶다.
  - 인터페이스에 함수 구현을 추가해버리자.
    - class에서 오버라이딩 하지 않아도 무방하고
    - 설사 겹치더라도, 클래스의 구현이 우선시됨
- Interface에 대한 조언
  - Minimal
  - Orthogonal functionality
- 다중상속으로 인한 충돌이 생기는 경우, 반드시 해결해야 컴파일 가능

# Chapter 14. Java Module System

- Why Module? - 2가지 디자인 원지
  - Seperation of concerns - 관련성을 바탕으로 코드를 캡슐화함
  - Information hiding - 외부 변화로 인한 영향을 받지 않도록 정보를 은닉

### 왜 Java module이 도입되었나.

- 모듈화를 위한 기능적 지원이 부족함
  - package 간의 가시성 제어 불가능
    - `public`: 모두가 볼 수 있다.
    - `private`: 클래스 내에서만 볼 수 있다.
    - `그러면 A 패키지의 내용을 B
  - classpath의 한계(1): 버전 구분 불가
    - 같은 이름의 클래스가 classpath내에 존재 = randomly selected
  - classpath의 한계(2):명시적 의존성 부여 불가
    - 어떤 클래스가 빠져있는지, 충돌이 나는지 실행하기 전까지 알 수 없다.
    - Maven이나 Gradle로 해결 가능하나, Module이 더 나은 해결책이 됨
- JDK가 monolithic하다
  - JDK의 크기가 커질수록 문제가 됨.

### Java modules: the big picture

- 3개 키워드 사용: `module`, `exports`, `requires`
- `module-info.java` 가 `module-info.class`로 컴파일됨
  - 해당 파일은 모듈의 루트 위치에 있어야 함.
  - `module` 모듈 이름과, 뒤 따라오는 block애 `exports`, `requires` 기입
  - `exports`: 다른 모듈에 공개할 패키지 기입
  - `requires`: 필요한 다른 모듈의 이름 기입
  - `requires static`: 다른 모듈이 필요하지만, 정적으로만 쓸 경우
  - `requires transitive`: 이 모듈을 사용하는 다른 모듈 또한, 대상을 사용 가능
  - `exports packageA to packageB`: 특정 모듈에만패키지 공개
  - `open`: 모듈 전체를 개방(개발시에 유용)
  - `opens`: 특정 패키지를 전체에 개방
  - `opens package to modules`: 특정 패키지를 특정 모듈에 개방
  - `provides interface with class`: *보충 필요*

# Chapter 15 Concepts behind CompletableFuture and reactive programming

- 두가지 아이디어 CompletableFuture, Flow APIs
   - publish-subscribe protocol로 reactive 프로그래밍 구현
- Java의 기존 병행처리 기법: ExecutorService
   - 정적 개수의 ThreadPool을 만듬, 재사용 가능
   - FIFO Queue로 들어온 Task를 처리
   - 다만, Sleep, IO, Network 대기상태에서 Blocking됨
- 대안1: `Future`
  - js의 Promise와 유사함
  - `Executor.submit` 에서 리턴 받은 결과
  - 결과를 얻으려면, `future.get()`으로 동기화 해야함.
  - 에러처리, 체이닝 등 기능이 부족함
  - 체이닝이 불가하기에 기다리는 쓰레드는 무조건 미리 만들어서 대기해야함
    - `a+b`상황이라면 `a`, `b`, `+` 쓰레드 3개가 떠야하고
    - `+` 쓰레드는 `a`, `b`가 모두 끝나길 기다려야하므로 낭비된다.
- 대안2: `CompletableFuture`
  - `js`의 Promise와 거의 유사하다.
  - 에러처리, 체이닝 등의 기능이 포함되어 있다.
- 대안3: `Reactive`
  - 여러번 호출되는 비동기 함수들 바탕으로 데이터 처리
  - pub-sub 모델로 메시지가 전달됨
  - 성공, 예외, 완료시의 callback을 넘김
  - 너무 많은 msg가 발행되는 경우(High-Pressure)
    - 이런 경우를 대비하여 `Subscription`을 통해 피드백을 줌(Backpressure)

# Chapter 16 CompletableFuture: composable asynchronous programming

- 생성(Factories)
  - `supplyAsync`: `suplier`기반으로 생성, Executor 지정 가능
  - `runAsync`: `Runnable`기반으로 생성, Executor 지정 가능
  - `allOf`, `anyOf`: 여러 Future들을 조합해서 생성
- Chaining
  - `thenApply`: `T->S`를 수행
  - `thenCompose`: CFuture를 생성해 이어지는 다른 async 작업 수행
  - `thenCombine`: 두개의 CFuture를 이용하는 콜백 수행
  - `thenRun`: `()->void`를 수행
  - `thenAccept`: `T->void`를 수행, 입력값을 소모하고 비움
- Stream 처리
  - List of future stream을 생성하고
  - 그 stream에 `forEach`로 join이나 get을 불러주면 된다.
  - 연쇄 함수들을 활용해서, 변환, 조합등이 가능하다.
  - `orTimeout`, `completeOnTimeout`을 통해서 타임아웃 지정 가능