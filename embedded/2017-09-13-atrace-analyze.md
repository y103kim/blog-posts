---
layout: post
category: embedded
title: atrace 분석
modified: 2017-09-13
tags: [atrace, development, android, systrace]
---

## ftrace, atrace, systrace

처음 회사에서 trace 분석 툴을 접했을 때 다양한 용어들이 혼재되어 있어서 혼란스러웠다. 대표적인 것이 아래 3가지가 있는데 하나하나 정리해 보자.

**ftrace**
: 리눅스 커널 분석을 위한 디버깅 툴이다.

**atrace**
: Android 용으로 확장된 trace 툴이다. ftrace를 이용해 커널의 trace를 가져오고, 추가로 Android Platform 단에도 trace를 삽입할 수 있다. 이를 통해 Android 전체의 디버깅이 가능해진다. ftrace를 사용하려면 sysfs node에 다소 복잡한 셋팅을 해야 하는데 이를 간단하게 해준다.

**systrace**
: 텍스트 문서 형태의 atrace를 [Catapult](https://github.com/catapult-project/catapult)라는 툴을 활용해 html로 변환한다. 이를 통해 사용자는 trace 결과물을 그래픽으로 볼 수 있다.

## atrace의 동작 방식

atrace의 동작 방식은 사실 ftrace의 nop 기능을 그대로 쓰는 것이다. ftrace에서 function을 nop로 선택하면, ftrace의 여러 기능들 중, 단순히 들어오는 입력들을 기록하는 기능이 활성화 된다. 아래 **atrace 미리보기** 항목에서 보여주는 모습과 같다. nop일 때 trace가 찍히는 경우는 크게 두가지이다.

첫 번째는 sysfs node로 찍히는 경우이다. ```/sys/kernel/debug/tracing/trace_marker``` 라는 node가 있는데 이 ```trace_marker```라는 node에 문자열을 던져주면, trace가 기록되게 된다. 콘솔에서도 테스트 해 볼 수 있는데, 예를 들어 아래와 같이 hello world를 던지면 찍힌다.

```
##### source : https://lwn.net/Articles/366796/
[tracing]# echo hello world > trace_marker
[tracing]# cat trace
# tracer: nop
#
#           TASK-PID    CPU#    TIMESTAMP  FUNCTION
#              | |       |          |         |
            <...>-3718  [001]  5546.183420: 0: hello world
```

두 번째는 커널 내부에서 직접 C로 찍어주는 경우이다. 참고로 유저 영역에서는 ```trace_marker``` 로만 찍을 수 있다. 이 방법으로는 불가능하다.

```c
/* source : https://lwn.net/Articles/365835/ */
trace_printk("read foo %d out of bar %p\n", bar->foo, bar);
```

위와 같이 ```trace_printk``` 커널에서 직접 찍어주고, 아래와 같이 trace를 확인해 보면, trace가 잘 기록됨을 알 수 있다.

```
[tracing]# cat trace
# tracer: nop
#
#           TASK-PID    CPU#    TIMESTAMP  FUNCTION
#              | |       |          |         |
            <...>-10690 [003] 17279.332920: : read foo 10 out of bar ffff880013a5bef8
```


## atrace 미리보기

atrace로 trace를 뜨면 다음과 같은 결과물을 볼 수 있다. 사실은 ftrace의 결과물과 형식은 똑같다. 

```
# tracer: nop
#
# entries-in-buffer/entries-written: 21436/21436   #P:6
#
#                                      _-----=> irqs-off
#                                     / _----=> need-resched
#                                    | / _---=> hardirq/softirq
#                                    || / _--=> preempt-depth
#                                    ||| /     delay
#           TASK-PID    TGID   CPU#  ||||    TIMESTAMP  FUNCTION
#              | |        |      |   ||||       |         |
          logcat-17544 (17544) [000] dn.4 146411.312396: sched_wakeup: comm=kworker/0:2 pid=18706 prio=120 success=1 target_cpu=000
          logcat-17544 (17544) [000] d..3 146411.312409: sched_switch: prev_comm=logcat prev_pid=17544 prev_prio=120 prev_state=R+ ==> next_comm=kworker/0:2 next_pid=18706 next_prio=120
     kworker/0:2-18706 (18706) [000] d..4 146411.312436: sched_wakeup: comm=adbd pid=13978 prio=120 success=1 target_cpu=002
     kworker/0:2-18706 (18706) [000] d..3 146411.312449: sched_switch: prev_comm=kworker/0:2 prev_pid=18706 prev_prio=120 prev_state=S ==> next_comm=logcat next_pid=17544 next_prio=120
```

먼저 각 항목의 의미와 주의점을 살펴보자. 이번에 Trace를 파싱하는 프로그램을 짰었는데, 꽤나 함정들이 숨어있다.

**TASK-PID**
: PID라고 설명은 나와있지만, 사실은 개념상 TID가 찍힌다. 리눅스 커널은 사실상 Process와 Thread를 모두 같은 Task로 취급하기 때문이다.

: 다만 이 부분은 atrace가 찍어주는 것이라서 확실히 찍힌다는 보장이 없다. 빈칸으로 나오거나, 정보가 없는 경우 직접 찍는 경우에는 0으로 출력되어 주의를 요한다. 예를 들면 아래와 같이 출력된다.

```
<idle>-0     (-----) [003] d..3 294113.375631: sched_switch: prev_comm=swapper/3 prev_pid=0 prev_prio=120 prev_state=R ==> next_comm=tee_irq_bh next_pid=3117 next_prio=120
 <...>-1641  (-----) [001] d..4 294113.375639: sched_wakeup: comm=tee_scheduler pid=3118 prio=120 success=1 target_cpu=002
```

**TGID**
: 리눅스에서 TGID는 개념상 우리가 이해하는 PID라고 보면 된다. 위에서 예로 든 것처럼, 빈칸으로 나올 수 있으니 주의가 필요하다.

**CPU#**
: 스케줄된 CPU의 번호가 출력된다.

**irqs-off, need-resched, hardirq/softirq, preempt-depth**
: 솔직히 뭔지 모르겠다. 나중에 보고서를 써야할 때는 조금 더 찾아보자. Trace 파싱할때는 사용하지 않았다.

**TIMESTAMP**
: 시간이 us 단위로 찍힌다. 꽤나 정교한 시간을 얻을 수 있다.

<br>

## atrace의 각종 Function 들

**FUNCTION** 항목은 따로 떼어서 보자. 기능을 추가함에 따라 많은 것들이 포함될 수 있다. 이번 포스트에서는 대표적인 것 3가지 만 소개한다.

### sched_wakeup

sched 관련 항목들은 아래 리눅스 프로세스의 상태 변화 다이어그램을 참고하면 이해가 쉽다.

[![](/images/008.atrace/1.png)](/images/008.atrace/1.png)

sched_wakeup 항목은 task가 sleep 상태에서 실행 대기 상태(wait) 상태로 깨어난 것을 나타낸다. 이후 스케줄러의 형태마다 다르겠지만, 큐 등에서 대기했다가 스케줄링 된다. 

```
sched_wakeup: comm=adbd pid=13978 prio=120 success=1 target_cpu=002
```

```comm``` 항목은 프로세스의 이름으로 proc/pid/comm에 저장되어 있는 것이 기록된다. ```pid```와 ```prio```는 각각 tid와 우선순위 값이다.


### sched_switch
sched_switch 항목은 task가 대기 상태(wait)에서 스케줄링 된 순간을 나타낸다. 해당 CPU에서 이전에 실행되는 Task의 정보와, 바뀐 Task의 정보가 나란히 출력된다.

```
sched_switch: prev_comm=swapper/3 prev_pid=0 prev_prio=120 prev_state=R ==> next_comm=tee_irq_bh next_pid=3117
```


### tracing_mark_write
tracing_mark_write 항목은 Android platform에서 찍어주는 Trace이다. Trace 메시지를 | 로 구분해 놓았는데 가장 첫 알파벳이 Trace의 종류를 나타낸다.

```C``` : 값을 입력하는 경우이다. 메시지의 가장 마지막에 숫자가 나온다.

```B``` : 함수의 시작을 나타낼 때 쓰인다. 메시지에 함수 이름이 들어 있다. platform의 C코드에서 ATRACE_CALL() 매크로를 사용하면 입력된다.

```E``` : 함수의 끝을 나타낼 때 쓰인다. E만 덩그러니 오기 때문에 중첩을 고려해서 매칭되는 B를 찾아야 한다.

```S``` : async trace를 넣으면 생긴다. 메시지에 trace로 날린 텍스트가 들어있다.

```F``` : async trace를 넣으면 생긴다. 메시지에 trace로 날린 텍스트가 들어있어서 S와 매칭하면 된다.
