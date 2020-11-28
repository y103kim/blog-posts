---
layout: post
category: webserver
title: samba 설치 및 설정
modified: 2017-08-21
tags: [samba, webserver]
---

## samba 설치하기

이번 포스팅에서는 삼바 설치 및 셋팅을 해보자. 윈도우 상에서 리눅스 서버를 제어하기에는 삼바만한 것이 없다. 설치는 한줄로 간단하게 끝난다.

	sudo apt-get install samba

## samba 설정툴 설치하기

다음은 삼바 환경설정을 해야한다. 삼바 계정은 리눅스 계정과 별개로 관리되니 따로 설정해 주어야 한다. X11 forwarding이 설정되어 있다면 system-config-samba를 설치해 간단하게 GUI로 samba를 설정할 수 있다. 아래와 같이 설치하자.

	sudo apt-get install system-config-samba

설치가 완료되고 아래와 같은 명령으로 실행하면 된다. 그러나 필자의 경우 몇가지 에러가 있었고, 그에 상응햐는 조치들이 필요할 수 있다. 에러와 문제 해결은 아래에서 자세히 다루겠다.

	sudo system-config-samba

## Trouble Shoting

첫 번째 에러는 DISPLAY 설정 에러이다. X11 forwarding이 제대로 설정 안된 경우로 다음 포워딩을 참조해 설정해면 된다.

	doocong@doocong:~$ sudo system-config-samba
	No protocol specified
	/usr/lib/python2.7/dist-packages/gtk-2.0/gtk/__init__.py:57: GtkWarning: could not open display

두 번째 에러는 libuser.conf가 없다는 에러이다.

```
doocong@doocong:~$ sudo system-config-samba
Xlib:  extension "RANDR" missing on display "192.168.0.111:0.0".
Traceback (most recent call last):
  File "/usr/sbin/system-config-samba", line 45, in <module>
    mainWindow.MainWindow(debug_flag)
  File "/usr/share/system-config-samba/mainWindow.py", line 121, in __init__
    self.basic_preferences_win = basicPreferencesWin.BasicPreferencesWin(self, self.xml, self.samba_data, self.samba_backend, self.main_window)
  File "/usr/share/system-config-samba/basicPreferencesWin.py", line 97, in __init__
    self.admin = libuser.admin()
SystemError: could not open configuration file `/etc/libuser.conf': No such file or directory
```

이 경우 libuser.conf를 만들어 주면 된다.

```
sudo touch /etc/libuser.conf
```

## samba 설정하기

system-config-samba이 실행되면 먼저 user를 추가해야 한다. Preferences에서 Samba Users...를 선택하자.
[![](/images/005.samba/1.png)](/images/005.samba/1.png)

사용자를 추가하려면 Add user를 누르고 대응되는 unix user 계정명(linux 유저 이름)과 윈도우에서 로그인 할 때 쓸 계정명을 차례로 입력하자. 이미 기본적으로 생성되어 있는 많은 unix 계정중에 메인으로 사용하는 계정을 찾으면 된다.
[![](/images/005.samba/2.png)](/images/005.samba/2.png)

windows username에는 windows에서 로그인 할 때 쓸 계정명을 입력하고, 암호도 입력해준다.
[![](/images/005.samba/2-1.png)](/images/005.samba/2-1.png)

이제는 공유 폴더를 추가해보자. Add share를 클릭하면 공유 폴더를 추가할 수 있다.
[![](/images/005.samba/2-2.png)](/images/005.samba/2-2.png)

Browser를 눌러 공유를 원하는 폴더를 선택하자. 일반적으로 홈폴더 전체를 공유하는 편이다.
[![](/images/005.samba/3.png)](/images/005.samba/3.png)

공유할 계정을 선택하자. 필자의 경우 방금 만든 doocong 계정에 체크하였다.
[![](/images/005.samba/4.png)](/images/005.samba/4.png)

이제는 윈도우에서 확인해보자. 내 컴퓨터의 드롭다운 메뉴에서 "네트워크 드라이브 연결"을 선택한다.
[![](/images/005.samba/4-1.png)](/images/005.samba/4-1.png)

아래와 같이 경로를 입력하고 마침을 클릭하자
[![](/images/005.samba/5.png)](/images/005.samba/5.png)

로그인 창이 뜨는데 여기에 아까 정한 계정명과 암호를 입력하면 로그인이 완료된다.
[![](/images/005.samba/6.png)](/images/005.samba/6.png)

