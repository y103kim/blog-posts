---
layout: post
category: python
title: Windows 환경에 Python 및 pip 설치하기
modified: 2017-09-03
tags: [python, pip]
---

## python 설치하기

먼저 python 2.7을 [다운로드](https://www.python.org/ftp/python/2.7.13/python-2.7.13.msi) 하자. 몇번 next를 클릭하면 간단하게 설치가 끝난다.

이제 환경변수를 설정해 주어야 한다. 내 컴퓨터의 드롭다운 메뉴에서 속성을 선택하자. 아래와 같은 화면에서 고급 시스템 설정을 클릭하자.

[![](/images/006.python-pip/1.png)](/images/006.python-pip/1.png)

고급 시스템 설정에서 가장 하단에 환경변수를 선택하자.

[![](/images/006.python-pip/2.png)](/images/006.python-pip/2.png)

윈도우 10 기준으로 아래와 같은 환경변수 설정 창이 뜬다. (윈도우 7과 10은 설정 창 형태가 바뀌었으니 참고 바람)

Path를 찾아서 편집을 누르자. 혼자 PC를 사용한다면 사용자 변수로 하던, 시스템 변수로 하던 상관없다. 필자의 경우 사용자 변수에 추가했다.

[![](/images/006.python-pip/3.png)](/images/006.python-pip/3.png)

열린 환경변수 창에 아래와 같이 ```C:\python27``` 과 ```C:\python27\Scripts``` 를 추가하자. 

[![](/images/006.python-pip/4.png)](/images/006.python-pip/4.png)

마지막으로 powershell 이나, cmd 창에 python이 실행되는지 확인하면 된다. (필자는 cmder을 사용한다.)

[![](/images/006.python-pip/5.png)](/images/006.python-pip/5.png)


## easy_install, pip 설치하기

다음으로는 easy_install을 설치하자. 다음 [설치 파일](https://bootstrap.pypa.io/ez_setup.py)을 다운받자. powershell의 경우 아래 명령을 사용하자.

	wget https://bootstrap.pypa.io/ez_setup.py -Outfile ez_setup.py

다은이 받아졌으면 아래 명령으로 설치하자

	python ez_setup.py

일반적으로 ```C:\python27\Scripts```에 설치된다. 이미 환경변수에 등록해 놓았으므로, 바로 실행될 것이다. 아래와 같이 명령을 주어 pip를 설치하자.

	easy_install pip

powershell에 pip를 입력해 설치가 정상적으로 이루어졌는지 확인하면 된다.

[![](/images/006.python-pip/6.png)](/images/006.python-pip/6.png)
