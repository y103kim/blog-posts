---
layout: post
category: webserver
title: dropbox와 jekyll 연동하기
modified: 2017-11-02
tags: [webserver, jekyll, dropbox]
---

## dropbox와 jekyll의 연동

내가 생각했던 jekyll의 최대 단점은, blog 파일 수정시 반드시 ssh를 이용하거나, samba를 연결해야 한다는 점이었다. 

하지만 이번에 dropbox를 연결한 이후에 그런 생각이 싹 사라졌다. 완전 혁신적이고 편리한 dropbox와 jekyll의 연동을 소개하고자 한다.

## dropbox 설치하기

ubuntu webserver에 dropbox를 설치하는 과정은 아래와 같다. 먼저 dropbox를 다운받자. [공식 홈페이지](https://www.dropbox.com/ko/install-linux)의 설명 중 명령줄을 이용한 dropbox 데몬을 다운받을 것이다.

```
cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -
```

다음에는 dropbox를 한번 실행하여서 계정에 로그인 해주어야 한다. 아래 명령을 실행하면 safari가 뜨면서 로그인 페이지가 보인다. 물론 XWindow 연결은 필수이다.

```
~/.dropbox-dist/dropboxd
```

## dropbox 서비스로 설정하기

위와 같이 dropbox를 실행하면, 홈 폴더 아래에 Dropbox 폴더가 생기면서 파일들이 모두 동기화 된다. 다만 ```ctrl-c```를 눌러서 종료하면, 동기화까지 종료된다. 지속적으로 동기화 시키려면, dropbox를 서비스로 등록해 주어야 한다. 파일 하나를 작성하자

```
sudo vim /etc/init.d/dropbox
```

인터넷에 많은 소스들이 있지만 그중 아래 소스가 잘 작동한다. 딱 한곳만 고치면 되는데, ```DROPBOX_USERS```에 리눅스 계정 명을 입력해 주면 된다. 코드를 붙여 넣은 뒤 저장하자.

```sh
#!/bin/sh
#
# Copied from http://forums.dropbox.com/topic.php?id=38529#post-414344
# Modified by jay@gooby.org (@jaygooby on Twitter)

### BEGIN INIT INFO
# Provides: dropbox
# Required-Start: $local_fs $remote_fs $network $syslog $named
# Required-Stop: $local_fs $remote_fs $network $syslog $named
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# X-Interactive: false
# Short-Description: dropbox service
### END INIT INFO

# dropbox service
DROPBOX_USERS="doocong"

start() {
  echo "Starting dropbox..."
  for dbuser in $DROPBOX_USERS; do
    HOMEDIR=$(getent passwd $dbuser | cut -d: -f6)

    DAEMON=$(find $HOMEDIR/.dropbox-dist/dropbox-lnx.* -name dropboxd)

    if [ -x $DAEMON ]; then
      HOME="$HOMEDIR" start-stop-daemon -b -o -c $dbuser -S -u $dbuser -x $DAEMON -p "$HOMEDIR/.dropbox/dropbox.pid"
    fi
  done
}

stop() {
  echo "Stopping dropbox..."
    for dbuser in $DROPBOX_USERS; do
      HOMEDIR=$(getent passwd $dbuser | cut -d: -f6)

      DAEMON=$(find $HOMEDIR/.dropbox-dist/dropbox-lnx.* -name dropbox)

      if [ -x $DAEMON ]; then
        start-stop-daemon -o -c $dbuser -K -u $dbuser -x $DAEMON
      fi
  done
}

status() {
  for dbuser in $DROPBOX_USERS; do
    dbpid=$(pgrep -u $dbuser dropbox-lnx)
    if [ -z $dbpid ] ; then
      echo "dropboxd for USER $dbuser: not running."
    else
      echo "dropboxd for USER $dbuser: running (pid $dbpid)"
    fi
 done
}

case "$1" in

start)
start
;;

stop)
stop
;;

restart|reload|force-reload)

stop
start
;;

status)
status
;;

*)
echo "Usage: /etc/init.d/dropbox {start|stop|reload|force-reload|restart|status}"
exit 1

esac

exit 0
```

이제는 서비스를 실행시켜 주어야 한다. 아래 명령을 실행하면 실행 끝이다.

```
sudo chmod +x /etc/init.d/dropbox
sudo update-rc.d dropbox defaults
```

## jekyll 폴더 연동하기

위의 서비스 등록까지 했으면 ```~/Dropbox``` 폴더가 실시간으로 동기화된다. 여기에 symbolic link만을 하나 걸어주면 된다.

```
cd ~/Dropbox
ln -s $JEKYLL_ROOT/_posts
```

자 이제 dropbox에 _posts 폴더가 생기면서 dropbox에 항상 동기화 된다. 필자의 경우 jekyll이 실시간으로 업데이트 되도록 서버에 아래 명령을 항상 실행시켜 놓는다.

```
bundle exec jekyll build --watch
```

이제 그럼 다양한 플랫폼에서 Textedit을 열어 dropbox의 _posts 폴더를 변경하자. 약간의 딜레이는 발생하겠지만, 언제 어디서든 실시간으로 blog에 포스팅이 가능하다.