---
layout: post
category: development
title: jsoncpp 설치 및 사용법
modified: 2017-09-04
tags: [json, cpp, jsoncpp]
---

## jsoncpp 설치하기

[jsoncpp 공식 git](https://github.com/open-source-parsers/jsoncpp.git)에서 clone 한 후, amalgamate.py를 활용해 cpp와 h파일을 만들자.

```
  git clone https://github.com/open-source-parsers/jsoncpp.git
  cd jsoncpp
  python amalgamate.py
```

완료하면 다음과 같이 dist 폴더 안에 파일이 생긴다.

[![](/images/007.jsoncpp/1.png)](/images/007.jsoncpp/1.png)

 dist 폴더 안의 내용을 복사해서 사용하기만 하면 된다.

## json 파일 읽기

단 몇줄로 json파일을 읽을 수 있다.

```c++
#include <fstream>
#include <iostream>
#include "json/json.h"
using namespace std;

int main() {
  Json::Value root;
  std::ifstream config_doc("config_doc.json", std::ifstream::binary);
  config_doc >> root;
}
```

## json 객체 내용 확인하기

읽은 json 객체의 내용을 읽으려면 python에서처럼 [] 연산자로 접근하면 된다.

```c++
  config_doc >> root;
  cout << root["test"];
```

int나 cstring 또는 string 으로 변환하고 싶다면 asInt나 asCString 함수를 활용하자.

```c++
  int n = root["test_int"].asInt();
  const char * cstr = root["test_cstr"].asCString();
  string str = root["test_cstr"].asString();
```

전체 파일 내용을 화면에 뿌려주고 싶다면 한줄이면 된다.

```c++
  cout << root;
```

Json::Value 객체에는 begin()과 end() 함수가 정의되어 있기 때문에, range-based for 구문도 사용 가능하다.

```c++
  for (auto &v : root["test_array"])
    cout << v;
```


## json 객체 수정하기

객체를 수정하거나, 새로운 항목을 추가하는 것도 직관적으로 그냥 대입하면 된다.

```c++
root["encoding"] = "hello";
root["indent"]["length"] = 10;
```

수정할 때 역시 range-based for문 사용 가능하다. 다만 레퍼런스 지정을 잊지 말자.

```c++
  for (auto &v : root["test_array"])
    v = 0;
```

## json 파일로 저장하기

객체를 수정한 이후에 저장도 간단하다. 

```c++
  ofstream outFile("test.json",ios::out);
  outFile << root;
```

