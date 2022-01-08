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