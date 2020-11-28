---
layout: post
category: algorithm
title: Fast Input by mmap
modified: 2018-02-11
tags: [algorithm, mmap, IO, scanf]
---

## Fast IO?

일반적으로 알고리즘 문제를 해결 할 때, iostream의 cout/cin이나 stdio의 printf, scanf를 사용한다. 

몇몇 알고리즘 문제에서는 입력되는 정수의 수가 100만개가 넘어가게 되는데, 이 때 부터는 위에 거론한 입력 수단의 속도가, 문제 해결 시간에 영향을 미친다. 

Beakjun Online Judge에 각 입력/출력 수단의 속도가 정리되어 있다.

[https://www.acmicpc.net/blog/view/56](https://www.acmicpc.net/blog/view/56)

## mmap 으로 input 구현

scanf가 범용적으로 구현되어 있기 때문에, 특정 상황에 대해서 쓰기에는 체크코드가 너무 많아 속도가 느려지는 것 같다.

mmap으로 stdin에 해당하는 버퍼를 직접 가지고 오면 훨씬 빠르게 입력을 처리할 수 있다.

아래는 mmap으로 input을 받아오는 코드이다.

```c++
#include <sys/stat.h>
#include <sys/mman.h>
class fio {
  size_t rsize;
  unsigned char* rbuf;
  int ridx;

  public:
  fio() : ridx(0) {
    struct stat rstat;
    fstat(0, &rstat);
    rsize = rstat.st_size;
    rbuf = (unsigned char*)mmap(0,rsize,PROT_READ,MAP_FILE|MAP_PRIVATE,0,0);
  }

  inline bool isBlank() {
    return
      rbuf[ridx] == '\n' || rbuf[ridx] == '\t' || rbuf[ridx] == '\r' ||
      rbuf[ridx] == '\f' || rbuf[ridx] == '\v' || rbuf[ridx] == ' ';
  }
  inline void consumeBlank() { while (isBlank()) ridx++; }

  inline int readInt(){
    int res = 0, flag = 0;
    consumeBlank();
    if (rbuf[ridx] == '-'){
      flag = 1;
      ridx++;
    }
    while (!isBlank() && ridx < rsize)
      res = 10 * res + rbuf[ridx++] - '0';
    return flag ? -res : res;
  }
};
```



