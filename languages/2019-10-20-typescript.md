---
layout: post
title: Typescript Study for React
modified: 2018-10-20
tags: [Typescript, React, ]
---

## Basic Types

`boolean`, `number`, `string` 은 자바스크립트와 같음. 타입을 변수 선언 뒤에 콜론과 함께 붙인다.

```typescript
let isDone: boolean = false;
```

`Array`의 경우는 `[]`로 표기하거나 Generic을 사용해서 타입을 표기할 수 있다. 개인적으로는 Generic 방식으로 쓰는게 더 가독성이 좋은듯.

```typescript
let list: number[] = [1,2,3];
let list: Array<number> = [1,2,3];
```

`Tuple`은 대괄호를 이용해서 입력한다. 순서, 개수 모두 중요함.

```typescript
let tuple: [string, number] = ["hello", 5];
```

`enmu`은 C 스타일로 선언하고, `.`로 접근한다. `[]`를 사용하면 이름을 문자열로 받아온다.

```typescript
enum Color {Red = 1, Green = 2, Blue = 4}
// Red's value is 1 
console.log(`${Color[1]}'s value is ${Color.Red}`)
```

`any`는 모든 타입의 합집합이다. 다만 최대한 사용을 자제하자.

`void`는 공집합인 타입이다. 다만 `null`이나 `undefined`는 `void`에도 할당이 가능하다. 하지만 기본적으로 `--strictNullChecks` 이걸 켜고 사용하기 때문에, 그냥 할당이 잘 안된다고 보면 된다.

`never`의 경우는 예외 처리 로직이나 절대 빠지면 안되는 로직에 리턴 타입으로 둔다. `null`이나 `undefined`와는 다르게 `any`에도 할당될 수 없다.

```typescript
function error(message: string): never {
    throw new Error(message);
}
```

`object`는 위에 언급된 기본 타입인 `number`, `string`, `boolean`, `symbol`,`null`, `undefined`를 제외한 모든 타입이다.

`Type Assertion`은 컴파일러의 타입 체크를 회피하는 옵션이다. `as`나 `<>`를 활용해서 변수를 특정 타입으로 간주 할 수 있다. 어디 써먹어야 하는지는 잘 모르겠다.

```typescript
let strLength: number = (<string>someValue).length;
let strLength: number = (someValue as string).length;
```

---

## Variable Declaration

변수 선언부에서는 `var`은 그냥 안쓰는걸로 한다. `let`은 아래 특징을 지닌다.
* 블록 스코프, 재선언 불가능, Closure 등도 사용 가능

기존 JavaScript에서 발전되어서 let을 이용한 block-level Shadowing이 가능하지만 쓰지 말자. 헷갈림

아래 예에서 논리적으로는 계속 무한 루프를 돌아야 하는데, 딱 3번 돌고 끝남.
```typescript
function loop():void {
    for (let i = 0; i < 3; i++)
        for (let i = 0; i < 2; i++)
            console.log(i);
    // 0 1 0 1 0 1
}
```

---

## Deconstructing

배열이나 객체에 대한 Deconstructing은 잘못쓰면 코드가 굉장히 거지같아짐
1차 구조로만 비구조화 하면 됨.

```typescript
let [first, ...rest] = [1, 2, 3, 4];
let [first, second] = [1, 2];
let { a, b } = { a: "baz", b: 101 };
```

타입 사용시에는 가능하면 Interface로 뺄것

```typescript
interface IArray {
    first: number;
    rest: Array<number>;
}
let [first, ...rest]: IArray = [1, 2, 3, 4];
```

Property Renaming은 그냥 쓰지 말자. Default Value 도 그냥 쓰지 말자. 한줄 더쓰면 된다.

---

## Interface

기본적으로 오브젝트를 표현할 때 쓴다. 아래의 경우 label 프로퍼티 하나를 가진 객체의 타입이 된다.

```typescript
interface ILabeledValue {
    label: string;
}
```

Optional Property가 가능하다. `readonly`는 Property를 읽기전용으로 지정 가능하다.

```typescript
interface SquareConfig {
    color?: string;
    readonly width: number;
}
```

지정한 멤버 외에 다른 Property도 허용할 수 있다. number를 Index로 넣어줄 경우 Indexable 임을 내포한다.

```typescript
interface SquareConfig {
    color?: string;
    readonly width: number;
    [propName: string]: any; // 다른 Property
    [idx: number]: string; // Indexable
}
```

함수는 괄호를 활용한다.

```typescript
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

Interface를 Java처럼 `implements`를 활용해서 쓸 수 있다. 이 경우 멤버를 강제한다.
`constructor`의 경우는 정적 멤버이기 때문에 `implements`를 활용한 interface에 포함될 수 없다. 아래 예를 보자.

```typescript
interface ClockConstructor {
  new (hour: number, minute: number);
}

interface ClockInterface {
  tick();
}

const Clock: ClockConstructor = class Clock implements ClockInterface {
  constructor(h: number, m: number) {}
  tick() {
      console.log("beep beep");
  }
}
```

Interface끼리 extend를 써서 확장할 수 있다.

Callable 오브젝트로 쓰려면 `as`를 활용해야 한다. 다만 `as`를 쓰면 컴파일러 에러가 씹히므로 매우 주의해서 쓸 것.

```typescript
interface CounterCtor {
    (start: number): string;
}

interface Counter extends CounterCtor {
    interval: number;
    reset(): void;
}

let counter = (function (start: number) { }) as Counter;
counter.interval = 123;
counter.reset = function () { };
```

---

## Classes

public, private, protected를 쓸 수 있고, 그 의미는 다른 언어와 동일

readonly를 멤버에 붙일 수 있음.

readonly를 constructor의 인자에 붙여주면, 따로 초기화 로직을 안넣어도 됨.

```typescript
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) { }
}
let octo = new Octopus("John");
console.log(octo.name) // John
```

`get`과 `set`을 쓰면 getter와 setter를 쓸 있음. 다만 ES5 이상만 지원하므로 IE10 이하로는 못쓴다고 보면 됨.

```typescript
class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (newName && newName.length > fullNameMaxLength) {
            throw new Error("fullName has a max length of " + fullNameMaxLength);
        }
        
        this._fullName = newName;
    }
}
```

`static`도 가능하다. 클래스 이름으로 바로 접근 가능.

클래스를 인터페이스로써만 사용할 수 있음. `abstract` 클래스도 됨.

---

## Functions

타입을 화살표 함수 형태로 작성 가능, 한쪽에 미리 함수를 정의를 해놓으면 인자에 따로 기록하지 않아도 됨.

간결한 코드 작성에 도움을 주므로, C++에서 처럼 타입 정의를 분리해 두는것도 좋음.

```typescript
type IFunc = (x: number, y: number) => number;
let myAdd: IFunc = (x,y) => x+y;
```

조건부 인자는 `?`를 사용하고, 기본 인자는 그냥 대입하면 됨.

```typescript
function buildName(firstName = "Kim", lastName?: string) {
    // ...
}
```

가변길이 인자는 Spread 연산자 `...`를 써야함.

this에 대한 binding이 일반 Function과 Arrow Function이 다른것은 기존 JavaScript와 똑같음.

<https://www.typescriptlang.org/docs/handbook/functions.html#this-and-arrow-functions>

`this`는 파이썬처럼, 명시적으로 첫번째 인자로 사용될 수 있다. 이를 통해서 타입을 명시해 주는게 권장됨.

```typescript
function f(this: void) {
    // make sure `this` is unusable in this standalone function
}
```

함수 Overloading 가능.

```typescript
function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
```

---

## Generic

테스트

