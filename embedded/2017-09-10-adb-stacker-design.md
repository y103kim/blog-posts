---
layout: post
title: ADB accel design
modified: 2017-09-10
tags: [adb stacker, development, adb]
---

## ADB accel 설계 목적

Chip vendeor에서 Android Performace에 관한 일을 하면서 느끼는 것이, adb가 무지막지하게 느리다느 점이다.

Adb가 느린 것에는 여러가지 요인들이 있겠지만, USB를 통한 통신 딜레이가 이 느린 속도의 주범임은 확실하다. PC의 상태에 따라서는 1초까지도 Delay가 발생하고 한다. 참고로 USB 보안을 위해 각종 보안프로그램을 덕지덕지 발라놓은 우리 회사야말로 최악의 USB 통신 속도를 자랑한다.

때문에 smart하게 이 문제를 해결할 방법이 필요했다. 아이디어는 다수의 adb 명령을 모아서 한번에 보내는 처리하는 것이다. 간편하게 Python으로 구현해 보려 한다.

## ADB accel 설계

### 1. Command Buffer

Commnad Buffer는 사용자가 exec 명령을 통해 전해주는 명령을 쌓아두는 역할을 한다. 하나하나 각각 실행할 경우, 시간이 너무 오래걸리므로, Buffer에 모아두었다가 실행하자는 아이디어이다.

### 2. adb.put

put 명령어는 사용자가 필요로 하는 명령어를 하나씩 Buffer에 쌓아주는 역할을 한다. 다만, stdout이 필요할 경우 어디에다 저장할지, silent stdout 옵션이 필요한지는 선택적으로 지정하게 한다.

### 3. adb.exec

exec 명령은 사용자가 저장했떤 모든 명령을 한꺼번에 실행한다. 실행하면서 stdout을 저장해야 할 것은 실시간으로 array에 저쟁해 두고 리턴하면 된다.

### 고민이 되는 점

처음 설계 당시에는 push는 dependent를 고려해서 처음에 한꺼번에 할 수 있도록하고, pull은 마지막에 한꺼번에 수행한다. 로 구현하려 했지만 생각보다 직관적인 인터페이스 만들기기 쉽지 않다. 일단은 push나 pop을 만나게 되면 무조건 그 앞에 명령들은 수행하도록 구현하고 추후 개선해 봐야겠다.

일단은 재빨리 작성해 둔 코드, 시간이 나면 더 수정해 볼 예정

``` python
import os, subprocess;

cmd_buf = []
push_buf = []
pull_buf = []
my_env = os.environ.copy();

def put(cmd1, cmd2, save_out=False, silent=False):
    if cmd1 == "shell":
        cmd_buf.append([cmd2, save_out, silent]);

def parse(buf):
    cmd = ""
    for entry in buf:
        cmd += entry[0] + ";";
        cmd += 'echo vvvvvv;';
    cmd = 'adb shell "%s"' % cmd;
    return cmd;

def exec():
    parsed_cmd = parse(cmd_buf);
    p = subprocess.Popen(parsed_cmd, env=my_env, stdout=subprocess.PIPE)
    # Grab stdout line by line as it becomes available.  This will loop until 
    # p terminates.
    cnt = 0;
    ret = [];
    while p.poll() is None:
        if cnt >= len(cmd_buf):
            break;
        l = p.stdout.readline().decode('ascii').strip()
        if "vvvvvv" in l:
            cnt += 1;
            continue;
        if cmd_buf[cnt][1] == True:
            ret.append(l);
        if cmd_buf[cnt][2] == False:
            print(l);

    # When the subprocess terminates there might be unconsumed output 
    # that still needs to be processed.
    print(p.stdout.read().decode('ascii'))
    return ret;


put("shell", "ls | grep sdcard", True, True);
put("shell", "dumpsys gfxinfo", True, True);
print(exec());
```