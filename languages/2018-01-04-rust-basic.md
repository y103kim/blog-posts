---
layout: post
title: Rust 한방에 배우기(1) - 변수와 타입
modified: 2017-11-21
tags: [rust language, rust guide, quick start]
---

## Rust 를 배워보자

시작하기 앞서서, 이 글은 빠른 Rust 학습을 위한 요약본임을 알린다.
[The Rust Programming Language](https://doc.rust-lang.org/book/) 를 요약했다.
필자 역시 학습하면서 기록 차원에서 이 포스트를 남기는 것이므로, 틀린 부분이 있을 수 있다.

## 설치

리눅스 기준으로 설치는 한줄이면 끝난다.

```
curl https://sh.rustup.rs -sSf | sh
```

이후에 path 만 추가해주자. ```~/.bashrc```에 추가하면 된다.

```
export PATH="$HOME/.cargo/bin:$PATH"
```

## compile과 cargo tool

위의 설치 과정을 끝냈으면 ```rustc````로 컴파일이 가능하다. cargo 라는 tool을 활용하면 이미 많이 업로드 되어 있는 rust 프로젝트들을 다운받을 수도 있다.

아래 명령을 실행하면 Hello world 프로젝트를 다운받은 후 빌드할 수 있다.

```
cargo new hello_cargo --bin
cd hello_cargo

rustc src/main.rs  # rustc 로 직접 컴파일 하기
./main             # 컴파일한 executable 직접 실행하기

cargo build        # cargo 로 컴파일 하기
cargo run          # cargo 로 실행하기
```

## 변수

변수 선언은 아래와 같이 한다. 참고로 주석 방식은 ```//``` 만 지원

```rust
let x = 1;                  // let 으로 변수를 선언
                            // 타입은 명기 필요 없음

x = 2;                      // 변수는 기본이 immutable
                            // 즉 이 코드는 컴파일 에러

let x = 2;                  // 하지만 재정의 하는 것은 가능
                            // 이 경우 x가 shadow 되었다 라고 표현
```

Mutability와 const는 다음과 같이 사용한다.

```rust
let mut y = 2;              // mutable로 선언한 경우 이후 변경 가능
y = 3;                      // 컴파일 에러 발생 안함

const z : i32 = 100_000;    // 상수는 타입을 명시
                            // 숫자를 표현할 때는 _ 를 사용하는 것이 가능
                            // 상수는 mut  사용 불가
```

## 타입

정수형 타입은 아래와 같다. 타입을 명시하지 않을 경우 기본타입으로 지정된다.

```rust
let byte : i8 = 1;          // byte
let ubyte : u8 = 1;         // unsigned byte

let short : i16 = 0;        // short
let ushort : u16 = 0;       // unsigned short

let int : i32 = 2;          // int
let int = 2;                // 정수 상수의 기본 타입은 int
let uint : u32 = 2;         // unsigned int

let longlong : i64 = 3;     // long long
let ulonglong : u64 = 3;    // unsigned long long

let arch_dep : isize = 4;   // 32bit 운영체제에서는 int
                            // 64bit 운영체제에서는 long long
let uarch_dep : usize = 4;  // 32bit : unsigned int
                            // 64bit : unsigned long long
```

정수형 상수의 진법 표현은 아래와 같다.

```rust
let decimal = 100_000;      // 10진법
let hex = 0xff;             // 16진법
let octal = 0o77;           // 8진법
let binary = 0b1111_1101;   // 2진법
let byte_or_char = b'a';    // 바이트
```

실수형 타입은 아래와 같이 쓴다.

```rust
let x = 2.0;                // 실수형 기본 타입은 f64 (double)
let y: f32 = 3.0;           // 이 경우에는 f32
```

사칙연산은 다른 언어들과 동일하다.

```rust
let sum = 5 + 10;           // 덧셈
let difference = 9.5 - 4.3; // 뺄셈
let product = 4 * 30;       // 곱셈
let quotient = 56.7 / 32.2; // 나눗셈
let remainder = 43 % 5;     // 나머지
```

부울 타입이 있다.

```rust
let t = true;
let f: bool = false;
```

문자 타입은 유니코드가 기본이다. ```char```로 명시할 수 있다.

```rust
let c : char = 'z';
let z = 'ℤ';
let heart_eyed_cat = '😻';
```

Python에서 익숙한 Tuple 타입을 지원한다.

```rust
let tup = (500, 6.4, 1);                    // 소괄호와 쉼표로 사용
let tup: (i32, f64, u8) = (500, 6.4, 1);    // 타입 명시도 가능하다.

let (x, y, z) = tup;                        // Tuple로 개별 변수를 초기화 가능하다.
let five_hundred = tup.0;                   // 마침표와 숫자로 개별 접근이 가능하다.
```

모든 언어에서 흔히 보이는 Array 타입을 지원한다.

```rust
let a = [1, 2, 3, 4, 5];    // 대괄호와 쉼표로 사용
let first = a[0];           // 익숙한 방식으로 개별 접근 가능

let error = a[6];           // Bound 체크를 해준다, 즉 에러
```

