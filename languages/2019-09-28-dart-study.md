---
layout: post
category: languages
title: Dart(1)
tags: [dart, flutter]
---

## 발단

업무 중에 멀티플랫폼 앱을 개발해야 할 일이 생겨서 여러가지 솔루션을 모색했다. 여기에는 Android, Windows를 지원해야 하는 요구사항이 있었다. Flutter, React-Native, React-Electron 정도가 물망에 올랐다. 일단 근본이 Native 개발자이고 아직까지는 C++이 제일 편한 천성인지라, Flutter를 이용해보자 결심했다.

----

## Dart를 배우자

Dart의 Document를 찬찬히 읽고 특정들을 파악해 보려고 한다. C++과 JavaScript, Python등을 어느정도 잘 알고 있는 상태에서 배워보는 것이기 때문에, 공통점 차이점 위주로 좀 정리해보자.

----

## Variables

Dart는 기본적으로 타입이 있는 언어이다. 다만 Type Inference를 강력하게 지원하기 때문에, C보다는 C++에 가깝게 느껴진다.

### Dart Primitive 타입에 대한 정리

* `String`, `int`(64bits), `double`, `List`, `Map`을 지원한다.
* `num`은 `int`와 `double`을 포함한다.
* `var`을 쓰면 타입을 자동으로 추론한다.
  ```dart
  var name = 'Voyager I';
  var year = 1977;
  var antennaDiameter = 3.7;
  var flybyObjects = ['Jupiter', 'Saturn', 'Uranus', 'Neptune'];
  var image = {
    'tags': ['saturn'],
    'url': '//path/to/saturn.jpg'
  };
  ```
* 초기화를 안하면 `null`이다.
* 여러 개의 타입을 List나 Map의 원소로 섞어 쓸 수 있다. 이 경우 타입을 Object로 간주한다. Dart는 모든 변수를 Object에서 상속받아 만든다.
  ```dart
  var flybyObjects = ['Jupiter', 123, 'Uranus', 'Neptune'];
  ```
* 참고로 [DartPad](https://dartpad.dartlang.org/)에서 타입을 체크해 보면 전부 JS가 붙은 자바스크립트 타입의 객체로 바뀌어 찍힌다.
  ```dart
  var image = {'tags': ['saturn'], 'url': '//path/to/saturn.jpg'};
  print([1,2].runtimeType);   //JSArray<int>
  print([1,"2"].runtimeType); //JSArray<Object>>
  print(image.runtimeType);   //JsLinkedHashMap<String, Object>
  ```

### String

* `String`은 큰따옴표(`"`)와 작은 따옴표(`'`)를 사용하면 된다.
* Python이나 Javascript처름 큰/작은 따옴표를 서로를 문자의 일부로 인식한다.
  ```dart
  print("hello! 'Jony'!") // hello! 'Jony'!
  ```
* 문자열을 쪼개서 쓰면 그냥 붙어있는 것으로 인식한다. 여러줄로 써서 가독성을 높일 수 있다.
  ```dart
  var foo = "123" "345";
  print(foo); // 123345
  ```
* Javascript에서 Grave accent(`)와 달러표시($)를 이용해 변수를 string 내로 인라인 하는 것이, 그냥 따옴표 String에서 가능하다. 식을 사용하려면 중괄호로 묶어야 한다.
  ```dart
  var name = "Doocong";
  print("My name is $name");
  print("The lenght of my name is ${name.length}");
  ```

### List

* `List`에는 Fixed-length List와 Growable List가 있다.
* 생성시 길이를 주냐 안주냐의 차이로 달라진다.
  ```dart
  var growable = <int>[];
  var fixed = List<int>(5);
  ```
* Fixed-length의 경우 add와 같이 insertion, deletion 동작을 하는 함수를 사용할 수 없다.
  ```
  var fixed = List<int>(5);
  fixed.add(1); // error!
  ```
* Growable이라고 하더라도 JavaScript의 List처럼 대괄호(`[]`) 로 사이즈를 키우는 짓은 하지 못한다.
  ```dart
  var growable = <int>[];
  growable[5000] = 3; //error!
  ```
* List 관련 에러는 신기하게도 컴파일 타임에는 체크를 못한다. 즉 위에서 `//error`가고 표시한 부분 에러 모두 런타임 에러로 발생한다. 이유는 잘 모르긋다.

### Map

* `Map`은 기본적으로 `LinkedHashMap`이다.
* 학교에서 배운 기본적인 Hash Map은 순서를 유지할 수 없는데, `LinkedHashMap`은 삽입 순서 유지가 보장된다.
  ```dart
  Map<String, Object> image = {'tags': ['saturn'], 'url': '//path/to/saturn.jpg'};
  ```
* 순서 유지가 안되는 `HashMap`이나 로그시간으로 동작하는 `SplayTreeMap`으로도 사용 가능하다.

### Others

* 그 외에는 sets, runes(유니코드), symbols가 있다.
* `Symbol`는 말 그대로 Symbol로 사용할 수 있는 타입이다. `#foo`와 같이 쓰면 실제로는 `Symbol("foo")`와 같다.
```dart
print(#foo);
```
* `reflect`라고 해서 런타임에 클래스의 명세 자체를 변경하는 것이 가능한데, `Symbol`이 이때 실제 멤버의 이름 역할을 한다.
* `final`과 `const`로 상수나 불변함을 표현해 줄 수 있다. `final`은 런타임 지정 가능하지만, `const`는 컴파일 타임에 결정되어야 한다.

----

## Functions

### 기본적인 내용

함수에 관한 문법은 C++과 거의 일맥상통한다. 기본적으로는 타입이 있기 때문에다. 하지만 `dynamic`이라는 수상하게 짝이없는 타입도 있다. 분명 Type 기반의 언어인데, 묘하게 타입을 벗어나 사용할 수가 있다. 특징을 정리하자.

* Curly Brace(`{`) 를 사용한 전형적인 함수 형태를 지닌다. Javascript나 C++의 형태와 같다.
  ```dart
  void main() {
    print('Hello, World!');
  }
  ```
* C++에서 리턴 타입에 `auto`를 사용할 수 있는 것 처럼, 리턴 타입을 생략 할 수 있다.
  ```dart
  hello() {
    return "Hello world!";
  }
  ```
* 인자의 타입도 생략 가능하다. 이 경우 알아서 dynamic 타입이 된다.
  ```dart
  hello(name) {
    return "Hello $name";
  }
  ```
* 저 `dynamic`타입 때문인지, 뭔지는 몰라도 Method Overloading을 지원 안한다.
* 당연히 연산자 오버로딩도 안됨
* 클래스 내에서 Object가 가지고 있던 연산자를 Overridng 하는것은 가능.

### Arrow Functions

Lambda 함수(Arrow Function)이 지원된다.

* C++의 기묘한 형태가 아니고 JavaScript-ish Arrow Function이 지원된다. 
  ``` dart
  hello() => "Hello world!";
  ```
* JavaScript에서는 화살표 함수에 이름을 붙이는 기능은 없는데, Dart는 화살표 함수에 이름도 붙여진다. 
  ``` dart
  String helloWithString() => "Hello world!";
  ```
* 반면 JavaScript에서는 인자가 1개일 경우는 괄호를 빼고 Arrow Function을 만드는 것이 가능했는데, Dart는 그런건 없다.
  ```dart
  var helloLambda = () => "Hello world!";
  var helloWithArg = name => "Hello $name"; // wrong
  var helloWithArg2 = (name) => "Hello $name";
  ```

### Anonymous function

* 화살표 함수와 매우 유사하지만 이름이 없는 함수 형태로도 쓸 수 있다.
* 주로 callback을 전달할 때 유용해 보인다.
  ```dart
  var list = ['apples', 'bananas', 'oranges'];
  list.forEach((item) {
    print('${list.indexOf(item)}: $item');
  });
  ```


### `dynamic` type in function

* 함수의 경우는 컴파일 시점에 타입이 결정되지 않는 경우 dynamic으로 간주한다.
* `helloWithArg2` 함수를 집중해서 본다면 Argument에 타입이 정해지지 않는 것을 볼 수 있다. Dart에서는 runtime type을 찍어볼 수 있는데  아래를 찍어보면 Dart의 Type이 추론이 어떻게 돌아가는지 대충 짐작할 수 있다.
  ``` dart
  helloWithArg2(name) => "Hello $name";
  helloWithArg3(dynamic name) => "Hello $name";
  helloWithArg4<R>(R name) => "Hello $name";

  print(helloWithArg2.runtimeType);             //(dynamic) => dynamic
  print(helloWithArg3.runtimeType);             //(dynamic) => dynamic
  print(helloWithArg4.runtimeType);             //<T1>(T1) => dynamic
  ```
* 함수가 `dynamic` 타입이더라도 대입하고 나면 Runtime Type이 딱 정해진다.
  ```dart
  print(helloWithArg2("doocong").runtimeType);  //String
  print(helloWithArg3("doocong").runtimeType);  //String
  print(helloWithArg4("doocong").runtimeType);  //String
  ```

### Lexical Scope

* Dart의 Scope은 변수가 선언된 Curly Bracket(`{}`)을 기준으로 안쪽이다.즉 변수가 Lexical(어휘적) scope 내에서 작동한다.
  ```dart
  bool topLevel = true;

  void main() {
    var insideMain = true;
    void myFunction() {
      var insideFunction = true;
      assert(topLevel);
      assert(insideMain);
      assert(insideFunction);
    }
  }
  ```
* Lexical의 의미는 `실제로 코드가 중첩되어 있다` 정도로 이해하면 된다. Dynamic Scope와 비교해 보면 이해가 슆다. 다이나믹은 backet은 다 무시하고, x가 그냥 전역이라고 생각하고 바꿔버린다. 
  ```dart
  // Dynamic scope 라고 가정한 Dart
  var x = 1; //x1
  foo() => print(x);
  bar() {
    var x = 2; //x2
    foo();
  }

  foo(); // Dynamic(1), Lexical(1)
  bar(); // Dynamic(2), Lexical(1)
  x = 3;
  foo(); // Dynamic(3), Lexical(3)
  bar(); // Dynamic(2), Lexical(3)
  ```
* 위 예제에서 Lexical Scope를 따른다며, 함수가 **선언될 때**가 중요해진다. 즉 foo가 선언될 때는 `x1` Lexical에서는 `x1`과 `x2`를 구분하는게 중요해진다.
* 임의의 Bracket을 통해서 Scope을 만드는게 가능하다. (Javascript는 안된다. 아래와 같이 하려면 function을 선언해야함.)
  ```dart
  {
    var a = 1;
    print(a);
  }
  print(a); //error!
  ```

### Lexical Closures

* 클로저 역시 된다. 콜백 내 변수의 scope이 끝나더라도, 다시 부르게 되면 값을 유지한다.
  ```dart
  makeAdd(int x) {
    return (y) => y + x;
  }
  var add3 = makeAdd(3);
  print(add3(5)); // 8
  ```

### Optional Arguments

* 중괄호로 묶어서 쓰면 Named Optional Arguments 를 사용할 수 있다.
  ```dart
  void enableFlags({bool bold, bool hidden}) {...}
  ```
* 호출 시에는 특이하게도 세미콜론을 쓴다. 인자를 입력하지 않으면 null로 처리된다.
  ```dart
  enableFlags(bold: true, hidden: false);
  ```
* `@required` Anotation을 통해서 필수 인자를 표기할 수 있다.
  ```dart
  void enableFlags({bool bold, @requried bool hidden}) {...}
  ```
* Default Arguments 설정도 가능하다.
  ```dart
  void enableFlags({bool bold = false, bool hidden = false}) {...}
  ```
* 위치 기반의 Optional Arguments 는 대괄호로 묶는다. 인자를 입력하지 않으면 null로 초기화된다.
  ```dart
  void hello([String name]) => print("Hello $name");
  hello("Doocong") // Hello Doocong
  hello() // Hello null
  ```
----

## Control flow statements

Control Flow에는 조건문과 반복문 두가지가 있다. 거의 대부분이 C나 C++과 동일하다.

* If 문은 C와 완전히 똑같다. Python3에서 되는 3중 비교는 안된다.
* 3항 연산자도 똑같다. 그런데 유사한 `??` 연산자가 생겼는데 매우 유용하다. 아래와 같은 경우에서는 `shouldInitialized`이 초기화 되었을 때는 `shouldInitialized`의 값이 result로 들어간다. 그러나 `shouldInitialized`이 `null` 이라면 `??`뒤에 들어오는 10이 result로 들어간다.
  ```dart
  var result = shouldInitialized ?? 10;
  ```
* Loop도 마찬가지다. 사용법이 거의 똑같다. 다만 `for...in` 문법은 C에는 존재하지 않는다. `Iterable` class에 대해서는 `for...in` loop 구성이 가능하다.
  ```dart
  var collection = [0, 1, 2];
  for (var x in collection) {
    print(x); // 0 1 2
  }
  ```
* `break`나 `continue`를 사용할 때는 `lable`을 이용해서 이중 루프를 건너뛰는게 가능하다.
  ```dart
  outer: for (int i = 0; i < 10; i++)
    for (int j = 0; j < 10; j++)
      if (j == 5) break outer;
  ```
* `Iterable` 일 경우 멤버 함수로 `forEach`를 가지고 있다. 여기에 인자로 콜백을 집어넣으면 순환하면서 실행이 가능하다.
* `assert`는 디버그 모드에서만 동작하는 키워드이다. 참이면 그냥 통과하지만 거짓이면 Exception을 발생시켜 준다.

----

## Operator

Operator는 다른 언어들과 거의 공통이기 때문에, 특이한 것만 정리해본다.

| Operator   | Description                                     |
| ---------- | ----------------------------------------------- |
| `/`        | 나누기 연산은 항상 실수 결과를 내놓는다.        |
| `~/`       | 이놈이 정수 나누기이다. 파이썬에선 `//`         |
| `++`, `--` | 파이썬에는 없는 전치 후치 연산이 있다.          |
| `as`       | 타입 캐스팅, 서브타입으로만 변환 가능하다.      |
| `is`       | 타입이 같은지 체크                              |
| `is!`      | 타입이 다른지 체크                              |
| `?:`       | 삼항연산자                                      |
| `??`       | null일 경우는 뒤에 나오는걸로 할당              |
| `..`       | Cascade annotation                              |
| `?.`       | 대상이 null일 경우는 null을 리턴하고 접근 안함. |


* 타입 캐스팅의 경우는, 논리상 Subset으로만 변환 가능하다.
  ```dart
  print(1 as double); //  1
  print(1.2 as int);  // error
  ```
* Cascade 표기법을 활용하면 초기화 코드가 좀 간단해진다. 또 한줄로 그 자체가 리턴이 되도록 짤 수 있다. UI 스러운 코드를 위해서 나온 표기법같다. 다만 해당 함수들이 setter(void 리턴) 이어야 한다.
  ```dart
  querySelector('#confirm') // Get an object.
    ..text = 'Confirm' // Use its members.
    ..classes.add('important')
    ..onClick.listen((e) => window.alert('Confirmed!'));
  
  // Same with above notation
  var button = querySelector('#confirm');
  button.text = 'Confirm';
  button.classes.add('important');
  button.onClick.listen((e) => window.alert('Confirmed!'));
  ```

----

## Exception

* 시작할 때 `try`, 잡을 때는 `on .. catch(e)`, 던질 때 `throw`, 마지막에는 `finally`를 써서 예외를 처리할 수 있다.
* 예외를 특정할 때는 `on .. catch(e)`를 쓰고, 아닐때는 그냥 `catch(e)`를 쓴다.
  ```dart
  try {
    // ···
  } on Exception catch (e) {
    print('Exception details:\n $e');
  } catch (e) {
    print('Exception details:\n $e');
  }
  ```
* catch의 두번째 인자는 Backtrace로 들어온다.
  ```
  } catch (e, s) {
    print('Exception details:\n $e');
    print('Stack trace:\n $s');
  }
  ```
* 파이썬에서 없어서 항상 아쉬웠던 exception propagation이 Dart에는 있다. `rethrow`를 사용하면 바로 예외를 다시 던질 수 있다.

