---
layout: post
category: development
title: ELF hack with LIEF
modified: 2019-07-11
tags: [LIEF, python, ELF]
---

## 목표

데이터가 존재하지 않는 elf 파일에 데이터를 심자!

## 입력파일

gPtr에 데이터가 존재하지 않음

```c++
//main.cpp
char* gPtr = (char*)0x601130;
int main() {
  return gPtr[0];
}
```

## 확인

빌드는 간단하게!

```
clang main.cpp
```

gdb로 찍어보면 아래와 같음

```
❯ gdb a.out
(gdb) p gPtr
$1 = 0x601130 <error: Cannot access memory at address 0x601130>
```

## LIEF로 elf에 section 추가하기

몇가지 포인트
1. Section 추가는 기본적으로 Executable만 됨
2. 가상주소를 명시해야함
3. segments는 자동으로 생김
4. flags를 할당하지 않으면 GDB가 로딩 안함, 데이터랑 똑같은 flags를 주면 될듯

```python
import lief
bin = lief.ELF.parse("a.out")
sec = lief.ELF.Section()
sec.name =".data2"
sec.content = b"Hello!\x00"
sec.virtual_address = 0x601130
sec.flags = 3
bin.add(sec, True)
bin.write("a.out2")
```

## 결과 확인!

gPtr 변수가 gdb로 보인다.

```
❯ gdb a.out2
(gdb) p gPtr
$1 = 0x601130 "Hello!"
```

## 추가로 겹쳐서 로딩해보기

.data section이 0x601020부터 시작하는데 0x601030에 애매하게 겹치기로 로딩해보자.

```
 [23]     0x00601020->0x00601038 at 0x00001020: .data ALLOC LOAD DATA HAS_CONTENTS
```

앞의 값들은 .data section 값을 하드코딩으로 가져옴

```
import lief
bin = lief.ELF.parse("a.out")
sec = lief.ELF.Section()
sec.name =".data2"
sec.content = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 48, 17, 96, 0, 0, 0, 0, 0] + ([0]*(0x100-24)) + list(b"Hello!\x00")
sec.virtual_address = 0x601030
sec.flags = 3
bin.add(sec, True)
bin.write("a.out3")
```

실행결과는.. 잘된다!

```
❯ gdb a.out3
(gdb) p gPtr
$1 = 0x601130 "Hello!"
```