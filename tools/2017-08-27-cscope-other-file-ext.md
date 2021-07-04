---
layout: post
title: 다른 확장자 파일 cscope에 추가하기
modified: 2017-08-21
tags: [cscope, development]
---

## 다른 확장자 파일 cscope에 추가하기 

아래와 같이 실행한다. script로 만들어 두어도 편하다.

```
rm cscope.files
find . -name "*.c" >> cscope.files
find . -name "*.cpp" >> cscope.files
find . -name "*.cc" >> cscope.files
find . -name "*.h" >> cscope.files
find . -name "*.java" >> cscope.files
cscope -b
cscope -d
```

한번 추가한 이후에는 ``` cscope -d ``` 로 실행하면 된다. 이후 업데이트가 필요한 경우에 다시 실행해주자.
