---
layout: post
category: bigdata
title: Spark with scala
tags: [spark, scala, bigdata]
---

Coursera의 Big Data Analysis with Scala and Spark 정리
  - 링크 <https://coursera.org/learn/scala-spark-big-data>

# Basics of Spark's RDDs

### same with scala's immutable sequence

- `map`, `flatMap`, `filter`, `reduce`, `fold`, `aggregate` 지원
- aggregate는 signature가 다름
  - aggergate는 병렬 처리를 가정하고, 순차처리 및 결합 연산을 둘다 넣어야함
  - 초기값이 여러 node로 옮겨져야 하므로 call-by-value를 하는듯
- scala의 collection함수를 쓰듯이 사용

```scala
// scala aggregate -> call-by-name
aggregate[B](z: => B)(seqop: (B,A) => B, combop: (B,B) => B): B
// rdd aggregate -> call-by-value
aggregate[B](z: B)(seqop: (B,A) => B, combop: (B,B) => B): B

// scala 함수처럼 사용
val result = wiki.filter(page => page.contains("EPFL")).count()

// counter 만들기, map-reduce처럼 코딩
val rdd = spark.textFile("hdfs://...")
val count = rdd.flatmap(line => line.split(" "))
               .map(word => (word, 1))
               .reduceByKey(_ + _)
```

### How to create an RDD?

- 다른 RDD로 부터 리턴 받기
- `SparkContext` 또는 `SparkSession`에서 생성
  - `parallelize`: scala collection을 RDD로 전환
  - `textFile`: text file로 부터 RDD 생성

### Transformation and Actions

- scala's HOF는 2가지 종류가 있음
  - tranformers: `map`, `filter`, `flatMap`, `groupBy` 등, 다른 콜랙션을 만들어줌
  - accessors: `reduce`, `fold`, `aggregate` 등, 한개의 값을 리턴함
- spark에도 대응되는 두가지가 있음
  - Transformations: lazy, 또다른 rdd를 결과로 냄
  - Actions: eager, 결과를 외부 스토리지에 저장함
  - 두가지로 연산을 나눔으로서 네트워크 통신 횟수를 효과적으로 줄임
  - 예를들어 `filter(...).task(10)`을 하면, 10개를 모으는 즉시 연산이 완료됨
- Transformations
  - `map[B](f: A => B): RDD[B]`
  - `flatMap[B](f: A => TraversableOnce[B]): RDD[B]`
  - `filter(pred: A => Boolean): RDD[A]`
  - `distinct(): RDD[B]`
- Actions
  - `collect(): Array[T]`
  - `count(): Long`
  - `take(num: Int): Array[T]`
  - `reduce(op: (A, A) => A): A`
  - `foreach(f: T => Unit): Unit`
  - `aggregate[B](z: B)(seqop: (B,A) => B, combop: (B,B) => B): B`
  - `aggregate`는 rdd의 reduction연산들 중에 유일하게 타입을 바꿀 수 있다.
- Transformations on Two RDDs
  - `union(other: RDD[T]): RDD[T]`
  - `intersection(other: RDD[T]): RDD[T]`
  - `subtract(other: RDD[T]): RDD[T]`
  - `cartesian(other: RDD[T]): RDD[T]`
- Useful RDD actions
  - `takeSample(withRepl: Boolean, num: Int): Array[T]`: 랜덤 추출(복원/비복원)
  - `takeOrdered(num: Int)(implicit ord: Ordering[T]): Array[T]`
  - `saveAsTextFile(path: String)`
  - `saveAsSequenceFile(path: String)`

### Evaluation in Spark: Unlike Scala Collections!

- RDD는 scala collection과는 다르다.
  - 연산을 미루기 때문에 생기는 문제
  - 모든 transformation을 action이 불릴때마다 반복한다.
  - 즉 베이스 RDD로부터 계산이 매번 반복되는것
- `persist` or `cache`
  - 지정한 연산을 한번 계산된 이후 메모리에 올린다.
  - 단 메모리에 올라가는 시점은 첫 action이 불려서 중간 상태가 계산된 이후이다.
- `cahce`는 기본 자바 객체를 메모리에 올린다.
- `persist`는 `cache`를 포함하는 5가지 옵션을 제공한다.
  - `MEMORY_ONLY`, `MEMORY_ONLY_SER`: 메모리만 저장, Serialized 여부 설정 가능
  - `MEMORY_AND_DISK`, `MEMORY_AND_DISK_SER`: 메모리 부족시 디스크에 저장
  - `DISK`: 디스크에만 저장

```scala
val data = sc.textFile(path).map(...) // bad
val data = sc.textFile(path).map(...).persist() // good
for _ <- 0 until iterationCount do
  data.map(...).reduce(...)
```

### Cluster Topology Matters!

- Spark는 크게 3가지 요소로 구성됨
  - Scala Driver, Scala Context 소유
  - Cluster Manager(Yarn, Mesos): Worker 관리
  - Worker Node: Executor로 실제 실행
- 그러므로 `rdd.foreach(println)`을 부르면
  - worker에서 print를 찍게 된다.
  - action으로 값을 얻어오기 전까지는 worker에서 실행됨을 기억해야함

### Pair RDDs

- `RDD`를 Pair에 대해 정의하면, 암시적으로 PairRDD로 바뀐다.
  - `RDD[(key,value)] -> PairRDD[(key, value)]`
  - 암시적으로 바뀌므로 그냥 `map`으로만 구성을 바꿔주어도 PairRDD가 된다.
- 여러 멤버 함수가 추가됨
  - reduction연산들이 action이 아니라, transformation임
  - 단일 값이 아니라, 키-값 리스트이므로 transformation으로 보는듯 
- PairRDD, Transformations
  - `groupByKey(): RDD[K, Iterable[V]] `
  - `reduceByKey`, `foldByKey`, `aggregateByKey`
  - `mapValues`: value에 대한 map
  - `keys`: key만 따로 RDD로 만듦
  - `join(other: RDD[(K,W)]): RDD[K, (V,W)]`
  - `leftOuterJoin`, `rightOuterJoin`, `fullOuterJoin`도 있음
- PairRDD, Actions
  - `countByKey`: 요것만 action임
- `groupByKey` 사용시 주의할 것

# Partitioning and Shuffling

### Shuffling

- 모든 노드로 전체 데이터를 전송하는 동작
- `groupByKey`, `reduceByKey`등을 실행 중에 shuffling이 일어난다.
- `groupByKey` 실행에 특히 비쌈
  - `reduceByKey`등은 먼저 각 노드의 키를 reduction함
  - 그러므로 전 노드에 shuffle해도 전송량이 작음
  - `groupByKey`는 그러한 사전 reduction이 불가능함

### Partitioning

- RDD를 특정 기준으로 나눔
- 여러 Node가 하나의 파티션을 분산해 가지는 경우는 없음
- 그러므로 Executor에 잘 나눠지도록 partition하면 성능이 증가
- Partitioner는 두가지 종류가 있음
  - hash partitioner: hash를 구한 뒤 hash 공간을 partition 수로 나눔
  - range partitioner: key의 min/max를 바탕으로 range를 partition수로 나눔
- partitioner는 두가지 원인으로 지정됨
  - 명시적 지정: `partitionBy`
  - 암시적 지정: 특정 transform을 거치면 맥락에 알맞게 자동 지정됨

### Optimizing with Partitioner

- 매우 큰 RDD를 파티션 하면 성능 증가
  - 상대적으로 작은 RDD와 join하는 경우 특별히 중요
  - join과정에서 대규모 shuffle이 발생하지 않음
- shuffle이 발생하는 조건
  - 자기 자신의 원소와 의존성이 발생
  - 다른 RDD의 원소와 의존성이 발생
- `toDebugString`을 활용하면 실행 계획을 볼 수 있음

### Wide vs Narrow Dependencies

- RDD는 transform을 거치게 되고, 이를 계승 그래프로 나타냄
- RDD는 4가지로 구성됨
  - partitons: 분할 정보
  - dependencies: Child, parent RDD와의 의존성 정보
  - function: 어떤 동작을 수행하는지
  - metadata: rdd구성에 필요한 메타 정보들
- narrow vs wide dep
  - narrow: partition이 child RDD 당 최대 1개의 partition과 의존성 관계를 가짐
  - wide: partition이 child RDD의 다수의 partition과 의존성 관계를 가짐
  - 당연히 narrow가 좋다.
- `dependency`로 직접 찍어볼 수 있다.
  - narrow: `OneToOneDependency`, `PruneDependency`, `RangeDependency`
  - wide: `ShuffleDependency`

# Structured data: SQL, Dataframes, and Datasets

- Dataframe은 RDD에 column이름을 추가한 형태
  - sql과 유사한 함수형태의 조작만 가능하게 변함
  - 단 쿼리 최적화가 강력하게 지워되므로 RDD보다 빠를 수 있음
- Dataset은 Dataframe에 column type까지 추가한 형태
  - `groupByKey`등 RDD에서 지원하던 함수 기반의 기능들을 활용 가능
  - 그러나 함수 기반 기능을 쓰면 최적화가 안됨