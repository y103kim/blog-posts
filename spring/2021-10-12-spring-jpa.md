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

