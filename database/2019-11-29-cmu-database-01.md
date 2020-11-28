---
layout: post
category: database
title: CMU database (1) - Relation Model
tags: [database]
---

## Intoduction

[여기](https://www.youtube.com/watch?v-oeYBdghaIjc&list-PLSE8ODhjZXjbohkNBWQs_otTrBTrjyohi) 서 강의 전체를 볼 수 있음

## Database

데이터베이스란?

> 실제 세계의 어떤 모습을 모델링하는 서로 연관된 데이터의 모음

데이터베어스는 모든 어플리케이션에서 사용하는 가장 기본적인 Component 임

## Example: Flat File Strawman

CSV(Comma-Seperated Value)가 가장 기본적인 형태의 DB

```csv
# Artist(name, year, country)
"Wu Tang Clan", 1992, "USA"
"Notoriius BIG", 1992, "USA"
"Ice Cube", 1989, "USA"
```

엘범도 위와 같이 유사한 형태로 csv 작성 가능, 만약에 검색 기능을 작성한다면?

```python
# Get the year that Ice Cube went solo.
for line in f.readlines():
  record - line.split(",")
  if "Ice Cube" -- record[0]:
    print(int(record[1]))
```

위와 같은 접근방식은 여러가지 측면에서 문제가 있음

###  Problem(1): Data Integrity

* 어떻게 Artist의 이름이 모든 앨범에 걸쳐서 같은지 보장할 것인가? 만약 "ICE CUBE" 라고 적혀있다면?
* 만약 Year 열에 integer가 아닌 값이 들어있다면?
* 만약 여러 Artist를 하나의 Album에 맵핑하고 싶다면?

### Problem(2): Implementation

* 특정 Record를 찾기 -> 성능문제: 데이터가 커지면 치명적으로 변함
* Python으로 구현했는데, 새로운 Application이 필요해지면? -> 재사용 불가
* Concurrency 문제

### Problem(3): Durability

* 만약 Update 중 Exeception이 발생하면?
* Replica를 이용하고 싶다면?

## DBMS

Database Management System

> 데이터를 저장, 분석할 수 있게 만든 소프트웨어

초기에는 사람이 Computing Power보다 쌌기 때문에, Database라고 할 만한것을 사람이 직접 매번 작성했음.

Ted Codd가 초기 데이터베이스를 제안하는 논문을 씀.

## What is Relation Model

Ted Codd는 Maintanace를 없애기 위해서 Database에 대한 추상 개념을 제안.

이게 Relation Model인데 이 Relation은 말하자면 Table임.

초기에는 사람이 Database 보다 더 효율적으로 구현했지만, 지금은 DB가 훨씬 더 잘함.

1. Database는 간단한 자료구조로 저장된다.
2. High-level Language로 접근 한다.
3. 어떻게 저장할지에 대한 문제 ->  Impelementation 문제로 제한 ->
   이렇게 하면 Application 입장에서는 아래쪽의 변화를 신경쓸 필요가 없음

## Data Models

용어정리

* data model: DB에서 데이터를 수식(Describing)하기 위한 개념들의 집합
* schema: 주어진 data model을 사용하는 데이터들에 대한 수식(Description)

종류

* SQL: Relational
* NoSQL
  * Key/Value
  * Graph
  * Column-family
* ML: Array / Matrix
* Obsolete / Rare
  * Hierarchical
  * Network


## Relation model

- **Structure**: 어떤 식으로 관계와 내용이 정의되어 있는가?
- **Integrity**: 모든 데이터가 제약조건을 확실히 만족하는가?
- **Manipulation**: 어떻게 데이터를 접근, 조작하는가?

만약 Table이 아래와 같다면?

|name          |year  | country      |
|--------------|------|--------------|
|Wu Tang Clan  |1992  | USA
|Notorious BIG |1992  | USA
|Ice Cube      |1989  | USA

- **Relation**: 위 테이블 자체, 각 Entry(관계) 들의 Unordered Set
  - 즉 N-ary Relation == Table with n columns
- **Tuple**: 각 Entry(Row)들을 Tuple로 나타냄
  - 각각의 Value는 일반적으로 atomic이거나 scalar임
  - `NULL`은 모든 domain에 속할 수 있는 특별한 값임
- **Primary Key**: 각 tuple이 가지고 있는 고유한 값. 어떤 DBMS는 primary key를 정의하지 않으면 알아서 만듬.
- **Foreign Key**: 다른 Table의 Primary Key를 가지는 Attribute.


## Data Manipulation Language(DML)

어떻게 DB에 접근하고 데이터를 저장할 것인가?

- **Procedural(Relational Algebra)**: 쿼리가 어떻게 데이터를 찾을지 전략(Strategy)까지 정의
- **Non-Procedural(Relational Calculus)**: 쿼리는 어떤 데이터를 찾을지만 정해줌

## Relation Algebra


