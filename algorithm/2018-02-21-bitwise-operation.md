---
layout: post
category: algorithm
title: 유용한 비트 연산들
modified: 2018-02-21
tags: [bitwise operation, hack, algorithm]
---

## 가장 낮은 자리의 1 찾기(lowest set bit mask)

64bit 정수를 bit mask로 사용한다면, set의 개수를 세야하는 경우가 종종 나온다. 다음 연산을 활용하면 set의 위치를 나타내는 mask를 찾을 수 있다.

```
x ^ (x - 1)
```

원래 수 ```x```에 1을 빼면 lowest set을 중심으로 상위 bit은 변화가 없고, lowest bit을 포함한 하위 bit은 flip 된다. 그러므로 xor 연산을 하면 flip된 bit들만 mask되게 된다.

```
x           = 1010000 (2)
(x-1)       = 1001111 (2)

x ^ (x-1)   = 0011111 (2)
```

mask가 아닌 해당 bit만 올라오게 하려면 아래와 같은 연산을 사용하며 된다.

```
x & (~x + 1)
```

위 연산은 ```(~x + 1)``` 이 핵심이다. not 연산시 모든 bit flip 된 상태에서 1을 더하면, lowest set 만 bit flip이 안된 상태가 된다. 아래 예를 보자.

```
x           = 1010000 (2)
~x          = 0101111 (2)
(~x+1)      = 0110000 (2)

x & (~x+1)  = 0010000 (2)
```

## 가장 낮은 자리의 1 을 0으로 만들기

아래 연산을 이용하면, 가장 낮은 자리의 1을 0으로 만들 수 있다.

```
x & (x - 1)
```

```
x           = 1010000 (2)
(x-1)       = 1001111 (2)

x & (x-1)   = 1000000 (2)
```

## 1의 개수 찾기 (# of sets)

위에서 언급한 lowest set bit을 0으로 만드는 연산을 이용하면 set의 개수를 좀더 효율적으로 찾을 수 있다.

```c++
int cnt = 0;
whlie(x != 0) {
  x &= (x-1);
  cnt++;
}
```

위의 방법보다 더 빠른 방법이 있다. 다소 이해하거나 외우기는 어렵지만, 상수시간만에 set의 개수를 셀 수 있다. 다만 32bit에서만 가능하다.

```c++
// https://graphics.stanford.edu/~seander/bithacks.html#CountBitsSetParallel
v = v - ((v >> 1) & 0x55555555);                    // reuse input as temporary
v = (v & 0x33333333) + ((v >> 2) & 0x33333333);     // temp
c = ((v + (v >> 4) & 0xF0F0F0F) * 0x1010101) >> 24; // count
```

## 2의 제곱수인지 판별하기

앞서 언급한 lowest set bit만 남게 하는 연산을 이용하면 2의 제곱수인지 판별할 수 있다.

```
x == (x & (~x + 1))
```

lowest set bit만 남은 결과와 원래 수가 같다면 당연히 2의 제곱수일 것이다.