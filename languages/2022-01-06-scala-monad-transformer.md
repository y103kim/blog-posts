---
layout: post
category: language
title: Scala Monad Transformer
tags: [scala, monad, transformer]
---

Source: <https://youtu.be/jd5e71nFEZM>

Functor, Monad, Transformer를 아주 다양한 예제로 설명해주는 좋은 영상이다.

# Functor

### 문제상황

- `Future`와 `List`과 겹쳐져 있음
- 두번의 map을 쓰는게 마음에 들지 않음

```scala
val x: Future[List[Int]] = ???
x.map(list => list.map(f))
```

### 해결법: Functor

- `Functor`는 `map`을 가지고 있다.
- `map`은 Wrapper 객체를 대상으로 한다.
- `map`은 Wrapper의 내용만 다른 타입/값으로 변경한다.

```scala
trait Functor[F[_]]:
  def map[A, B](fa: F[A])(f: A => B): F[B]

val futureListF = Functor[Future].compose(Functor[List])
futureListF.map(x)(f)
```

# Monad

### 문제 상황

- 두번의 Future가 겹치는 상황에서 해결 불가능
- 겹치는 연산을 하나로 합성해줄 필요가 있음
  - `A => Future[B]` + `B => Future[C]`를 합쳐서
  - `A => Future[C]`로 만들고싶음

```scala
def getUser(name: String): Future[User]
def getAddress(user: User): Future[Address]

val getCity: Future[Future[String]] =  // want Future[String]
  getUser("Gabriele").map(
    gab => getAddress(gab).map(
      address => address.city
    )
  )
```

### 해결법: Monad

- `Monad`는 `id`, `map`, `flatmap` 3가지를 가지고 있다.
- 즉 위 상황을 깔끔하게 해결해준다. `Future`는 모나드이다.
- `for`를 쓰면 monad를 더 깔끔하게 풀 수 있다.

```scala
val city: Future[String] = 
  getUser("Gabriele").flatMap(
    gab => getAddress(gab).map(
      address => address.city
    )
  )

val city: Fturue[String] = for
  gab     <- getUser("Gabriele")
  address <- getAddress(gab)
yield address.city
```

# 중쳡된 Monad

### 문제상황

- 실제 개발에서는 Option이 포함되기 마련
- 두 다른 Monad인 `Future`와 `Option`이 겹침

```scala
def getUser(name: String): Future[Option[User]]
def getAddress(user: User): Future[Option[Address]]

val city: Fturue[Option[String]] = for
  gab     <- getUser("Gabriele")
  address <- getAddress(gab) // Error!
yield address.city

// verbose & ugly solution
val city: Future[Option[String]] = for
  maybeUser <- getUser("Gabriele")
  maybeCity <- mayBeUser match { // ugly!
    case Some(user) => getAddress(user).map(_.map(_.city))
    case None => Future.successful(None)
  }
yield maybeCity
```

- 서로 다른 두 모나드는 중첩할 수 없다. 왜?
  - flatMap을 위한 로직을 알 수 없기 때문
  - `Future[Option[String]]`을 flatMap한다고 하자.
  - `F[O[F[O[String`을 `F[F[O[O[String`으로 바꿀 수 있어야 한다.
  - 그걸 위해서는 `Option`의 로직을 알야하 한다.

### 해결방법: Transformer!

- flatMap 로직을 내장하고 있는 Transformer를 따로 구현하자.
- `OptionT[Future, String]`이라고 하면
  - 실질적으로는 `Future[Option[String]]`인 것을 뜻한다.
  - 단 flatMap이 된다는것이 다르다.

```scala
val f: OptionT[Future, String] = 
  for
    gab     <- OptionT(getUser("Gabriele"))
    address <- OptionT(getAddress(gab))
  yield address.city
```