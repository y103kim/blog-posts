---
layout: post
category: development
title: Spring JPA 정리
tags: [Spring, JPA]
---

# Basics

- java 지영의 공식 ORM
- `EntityManager`를 통해서 entity 단위의 연산을 함
- persistence context 안에 캐시를 가지고 중간 저장을 함
- transaction 기반으로 DB 수정
- kotlin과 함께 쓸때는 `jpa` plugin 추가 필요
  - `@Entity`에 기본생성자가 필요한데, 자동 추가해줌

# cache & lazy write

- 트랜젝션 단위의 1차 캐시
  - 쓰기(remove, update, persist) / 읽기(find)
  - 쓰기는 트랜젝션 commit 호출시까지 지연 한 후 flush
  - 쓰기는 1차 캐시에 저장됨
  - 읽기는 즉시 실행
  - 읽기한 내용도 1차 캐시에 저장됨
  - 읽기시 1차 캐시에 저장된 내용이 우선시됨
- 동일성 보장
  - 같은 Entity는 동일한 객체임
  - 반복 가능한 읽기를 지원
- Flush
  - 1차캐시에 저장된 내용은 DB에 반영
  - 변경사항(update) 체크도 일어남
  - `em.flush` / `em.commit` / JPQL 쿼리시 실행됨
  - `hibernate.jdbc.batch` 기능을 이용하면 여러개의 쿼리를 하나로 합칠 수 있음
  - 트랜젝션 하에서만 유효한 매커니즘: 트랜잭션이 끝나기 전에 다 써주기만 하면 됨
  - 타입을 지정해서 쿼리시에는 생략 가능하나, 그러지 말자

# Entity mapping

### DDL

- 스키마 자동생성 (DDL)에 사용
- 로컬에서만 사용할 것
- 동작 모드: `hibernate.hbm2ddl.auto` 프로퍼티의 값 지정
  - `create`: 삭제 후 생성
  - `create-drop`: 생성 및 작업 이후 삭제
  - `update`: 새 컬럼 추가시 생성, 삭제는 안됨
  - `validate`: 정상 매핑 여부 확인
  - `none`: 사용 안함
- DDL 생성 기능
  - 혹은 `unique`, `size` 등 DB에 제약 사항을 주는 옵션들
  - 개발용으로만 쓰자!

### Mapping annotations

- `@Table`: 테이블 매핑
  - `name`: 이름을 따로 매핑
- `@Column`: 컬럼 매핑
  - `name`: 이름을 따로 매핑
  - `nullable`, `unique`, `length`: DB상 컬럼에 제약조건 부여
  - `insertable`, `updatable`: commit시에 쿼리 안날림
  - `columnDefinition`: 직접 컬럼 생성 정보 줌
- `@Temporal`: 날짜 매핑
- `@Enumerated`: enum 타입 매핑
  - `EnumType.ORDINAL`: 숫자로 기록, 변경시 대응 불가. 쓰지 말자.
  - `EnumType.STRING`: 문자로 기록, 권장
- `@Lob`: clob, blob 타입 매핑

### Primary key

- `@Id`: 직접 할당
- `@GeneratedValue`: 자동 할당
  - `IDENTITY`: 데이터베이스에 일임
  - `SEQUENCE`: 데이터베이스 시퀀스 오브젝트에 일임
  - `TABLE`: 키 생성용 테이블 따로 만듦
  - `AUTO`: DB에 따라서 알아서 사용
  - `name`: 이름을 따로 매핑
- `IDENTITY`시에 쓰기 지연 예외 생김
  - PK를 바로 얻어와야 하므로, Insert가 즉시 발생
- `SEQUENCE`, `TABLE`은 `allocationSize`을 활용해 최적화
  - PK를 미리 size만큼 올려놓고 잘라서 쓴다
  - 동시성 문제도 없고, PK를 위한 쿼리수도 줄어듦

# Foreign key mapping

- 테이블: FK는 Join시 양방향으로 활용 가능
- 객체: 참조는 단방향
- 양방향 연관관계를 위해서 각 객체는 참조를 가짐

### 단방향 연관관계

- FK를 들고있는 쪽: `@ManyToOne`, `@JoinColumn` 사용
- FK 없는쪽에는 구현 안함
- `EAGAR`: join한 테이블을 얻어옴
- `LAZY`: 각각 테이블을 lazy하게 얻어옴

### 양방향 연관관계

- FK를 들고있는 쪽: `@ManyToOne`, `@JoinColumn` 사용
- FK 없는 쪽: `@OneToMany(mappedBy="team")`
- `mappedBy`에는 FK의 필드 이름을 줘야함
- `mappedBy`로 매핑하면 쓰기가 불가능
- Kotlin에서는 아래와 같이 해야함

```kotlin
@OneToMany(mappedBy = "member")
var orders: MutableList<Order> = ArrayList(),
```

### 양방향 연관관계 주의사항

- 추가 시
  - 양쪽에 값을 다 넣어줘야함
  - 1차 캐시에 값이 잘못들어있을 경우를 대비
  - setter에 다른쪽 초기화를 넣어줘도 유용함
- 삭제 시
  - 삭제시에는 너무 복잡해 질 수 있음
  - `flush`, `clear`한 후 DB에서 업데이트
- `toString`, `data class`, `lombok`, `json 생성` 시 주의
  - toString 호출 시, 양방향 참조 때문에 무한루프를 돈다
  - 컨트롤러에서는 `Entity`를 직접 반환 하지 말 것
    - API 스펙과의 의존성을 제거
    - DTO로 변환해서 반환할 것

# 상속관계 매핑

- `@Inheritance(strategy=...)`
  - `JOINED`: 최대 정규화된 각각 테이블로 나눔
  - `SINGLE_TABLD`: 단일 테이블
  - `TABLE_PER_CLASS`: 구현 클래스마다 테이블 전략
- 장단점
  - `JOINED`: 최대 정규화, 공간상 이점 / Insert 2번, 잦은 join
  - `SINGLE_TABLD`: 단순한 로직, 성능상 이점 / null 허용해야함
  - `TABLE_PER_CLASS`: 쓰지 말자. 조회시 union사용됨
- `@DiscriminatorColumn(name="DTYPE")`
  - 상속시 각 타입의 종류를 스트링으로 저장
  - `@DiscriminatorValue("XXX")`: 저장되는 스트링 이름 바꾸기

### MappedSuperclass

- `@MappedSuperclass`
- 공통되는 필드를 묶어주는 역할
- DB와는 무관하게, 코드상의 상속관계를 만듬
- 엔티티가 아님
- 추상클래스로 만들어서 실수를 방지하길 권장

# Proxy

- 두가지 생성 방식
  - `em.getReference()`: Proxy entity 리턴
  - 지연 로딩 사용시 프록시를 대신 삽입 
- Delegation 역할을 수행함
- 기존 클래스를 상속받았으므로, 동형임
- 한번 필드에 접근하면 그때 내부 엔티티 레퍼런스를 초기화함

### Proxy / Entity 동일성

- 같은 대상에 대한 Entity와 Proxy는 항상 동일함
- `em.getReference`와 `em.find`의 동일성이 보장됨
- 그러므로 상황에 따라서 `em.find`에서 프록시가 튀어나올 수 도 있음
- 타입 비교가 예상과 다를 수 있음. `==` 대신 `instanceof`를 쓸것

### 지연 로딩

- `LAZY`로 설정하면 지연 로딩 가능
- 무조건 지연 로딩을 사용
- 즉시 로딩 하면 JPQL 쿼리시 **N+1문제**를 발생시킴

### 영속성 전이, 고아 객체

- django에서 cascade 설정하는거랑 같다.
- 부모 엔티티 저장할때, 자식도 함께 저장되고
- 부모 엔티를 삭제하면, 고아가 된 자식도 함께 삭제된다.
- 트리구조(대상의 하나의 부모만 가질때) 사용하는게 적합.

# 임베디드 타입

- `@Embedded`, `@Embeddable`
- DB상의 Column들을 묶어서 객체로 만들 수 있음
- DB에는 영향을 주지 않음
- `@AttributeOverrides`: 타입 이름 재정의 가능
- 임베디드 타입은 참조를 전달할 수도 있으므로, 불변으로 사용하는게 좋다.
  - setter를 모두 없애자
- 비교를 위한 `equals`구현 필요
- 값 타입 컬랙션은 쓰지 말자.

# JPQL

- JPQ에서 SQL 쿼리하는 방법
  - Native SQL: 직접 SQL 작성
  - JPQL: 엔티티 대상의 SQL 쿼리
  - Query DSL: JPQL 빌더
- 특징
  - Entity, Attribute는 대소문자 구분함
  - JPQL 키워드 (Select 등)은 대소문자 구분 없음
  - 테이블 이름 대신, 엔티티 이름을 써야함
  - 별칭은 필수, `as`는 생략 가능
  - `count`, `sum`, `avg`, `max`, `min` 가능
  - `group by`, `having`, `order by` 가능

### 반환, 파라미터

- `TypedQuery`, `Query`로 반환받음
- `getResultList`, `getSingleResult`
- 단일 반환은 없거나 많으면 예외발생하므로 주의
- 파라미터 바인딩 가능 `setParameter`

### 프로젝션: 조회할 대상을 전사하기

- 쿼리, 타입 리플렉션
- `em.createQuery("SELECT m FROM Member m", Member.class)`
- `SELECT xxx FROM`: xxx에 무엇이 오느냐에 따라 리턴이 달라짐
- `SELECT m FROM Member m`: entity projection
- `SELECT m.team FROM Member m`: entity projection with join (사용하지 말것)
- `SELECT m.address FROM Member m`: embedded projection
- `SELECT m.username FROM Member m`: string projection

### 여러 값 조회

- `SELECT m.username, m.age FROM Member m`
  - `Object[]`로 리턴받아서 조회가능
- `SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m`
  - DTO의 new를 호출하는 식으로 리턴받기 가능
  - 순서와 타입이 일치하는 생성자 필요
- 페이징: `setFirstResult`, `setMaxResults`

### Join

- inner join: `SELECT m FROM Member m INNER JOIN m.team t`
- outer join: `SELECT m FROM Member m OUTER JOIN m.team t`
- theta join: `select count(m) from Member m, Team t where m.username = t.name`
- `on`: 조인 대상 필터링, 연관관계 없는 외부 조인 가능
  - SQL은 join시 연관관계를 따로 on으로 넣어야함
  - JPQL은 연관된 필드로 join하면 on은 자동으로 넣어줌
  - 추가로 필요한 필터링을 on으로 넣으면 됨

### Subquery

- 괄호로 묶어서 쿼리를 중첩 가능
- select, where, having 절에서 사용 가능
- from 절에서 사용 불가능
  - join으로 따로 풀어서 쓸 것
- 서브 쿼리의 최적화를 위해서, 엔티티를 다 따로 만들어서 쓸 것
- `ALL`, `ANY`, `SOME`, `IN` 등의 함수를 지원함

```sql
-- subquery with seperate entity
select m from Member m
  where m.age > (select avg(m2.age) from Member m2)
```

### 기타 문법들

- `case when .. then .. else`: 구문 사용 가능
- `COALESCE`: null을 특정 값으로 대체
- `NULLIF`: 특정값을 null로 대체
- 기본 함수들
  - 문자열: `CONCAT`, `SUBSTRING`, `TRIM`, `LOWER`, `UPPER`, `LENGTH`, `LOCATE`
  - 숫자: `ABS`, `SQRT`, `MOD`
  - 기타: `SIZE`, `INDEX`
- 사용자 정의 함수도 미리 정의하면 사용 가능

### 경로 표현식

- 필드에 접근한는 것 이외에도
- 연관된 다른 엔티티에 접근하거나
- oneToMany로 매핑된 컬랙션 값도 접근 가능
- 그러나 필드를 접근하는거 이외에는 묵시적 join이 생김
  - 쓰지말자

### Fetch join

- 지연로딩으로 기본으로 해둘 때, 즉시 로딩을 할 수 있는 방법
- 엔티티의 연관관계들을 찾아서 Persistence를 채워준다.
- 일대다, 다대일 다 가능
- Fetch join된 대상에는 별칭을 주어서 필터링 하지 말것
  - 데이터 정합성을 해치는 결과를 초래

### Collection fetch Join

- 1:N 연관에 대해서 생기는 문제들
- 중복이 생길 수 있음
  - DISTINCT를 넣을 경우 쿼리에 DISTINCT를 넣어 join된 테이블에 중복 제거
  - 더해서 어플리케이션 단에서도 전사 대상의 중복을 추가로 제거해줌
- paging api 사용 불가
  - 1:N -> N:1 변경
  - `@BatchSize`를 사용할 수 있음