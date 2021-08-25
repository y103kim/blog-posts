---
layout: post
category: etc
title: Ubuntu 16.04 설치하기
modified: 2017-08-17
tags: [ubuntu, webserver]
---

## Ubuntu 16.04 설치용 USB 만들기
 
먼저 iso 이미지 파일을 다운받자. 이 이미지를 USB에 설치할 것이다. [우분투 다운로드 사이트](https://www.ubuntu.com/download/desktop)에 접속해서 Download를 선택하자. LTS 버전인 16.04를 선택하는 것이 좋다. LTS는 Long-Term Support의 약자로, 다른 버전보다 오랜 기간 동안 보안업데이트를 해준다는 의미이다. 보통 짝수버전이 LTS 버전이다.

[![](/images/001.ubuntu-setup/1.png)](/images/001.ubuntu-setup/1.png)

이후 기부할 것을 요구하는데, 감사한 마음을 가지고 건너뛰어주자. "Not now, take me to the download" 를 클릭하면, 새로운 페이지로 이동하면서 다운로드가 시작된다.

[![](/images/001.ubuntu-setup/2.png)](/images/001.ubuntu-setup/2.png)

이제는 iso를 USB에 구워야 한다. 이를 위해 iso를 USB에 구워주는 툴인 UUI(Universal USB Installer)를 [여기에 접속](https://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/)해 다운받자.

[![](/images/001.ubuntu-setup/3.png)](/images/001.ubuntu-setup/3.png)

UUI는 설치 하지 않아도 되는 편리한 툴이다. 실행 시킨 후에 약관에 동의하면 설정화면이 뜨는데, 아래 처럼 입력하자. 최신 버전에는 NTFS와 FAT32를 선택할 수 있는 기능이 생겼는데, 별 차이가 없는것 같다. NTFS를 선택할 경우 ISO가 통째로 복사되어 USB로 들어간다.

[![](/images/001.ubuntu-setup/4.png)](/images/001.ubuntu-setup/4.png)

F드라이브를 선택했다면, F드라이브의 MBR이 삭제될것이고, 라벨이 바뀔 것이니 맞는 드라이브인지 확인하라는 경고가 뜬다. 혹시나 실수하지 않도록, 드라이브 명을 다시 확인해주고, 예를 누르면 설치가 시작된다.

[![](/images/001.ubuntu-setup/5.png)](/images/001.ubuntu-setup/5.png)

---

## Ubuntu 16.04 설치하기

자 이제는 설치할 PC에 USB를 장착하고, 부팅 순서 설정을 위해 바이오스 설정 화면으로 들어가자. 부팅 후 F7 키를 연타하면 된다. 필자가 설치한 PC는 NUC라서 인텔의 바이오스 기준이다. 다른 회사의 바이오스의 경우 구글에서 다른 블로그의 글들을 참조하시길.

부팅이 끝나면 설치 아이콘을 클릭해서 설치를 시작하자.

[![](/images/001.ubuntu-setup/inner/1.png)](/images/001.ubuntu-setup/inner/1.png)

영어로 선택하는 면이 여러가지 면에서 깔끔해서, 필자는 English를 언어로 선택했다. 취향에 따라 선택하시길. 물론 설치 이후에도 얼마든지 바꿀 수 있으니 걱정하지 않아도 된다.

[![](/images/001.ubuntu-setup/inner/2.png)](/images/001.ubuntu-setup/inner/2.png)

와이파이 설정은 일단 건너뛰었다. 필요한 패키지들은 나중에 설치하면 된다.

[![](/images/001.ubuntu-setup/inner/3.png)](/images/001.ubuntu-setup/inner/3.png)

같은 맥락에서 third-party 소프트웨어 설치도 스킵하자.

[![](/images/001.ubuntu-setup/inner/4.png)](/images/001.ubuntu-setup/inner/4.png)

필자는 이미 Ubuntu를 해당 PC에 깔아서 사용하고 있었기 때문에, 현재 사용중인 파티션을 unmount할 것을 묻는다. 전부 삭제하고 새로 설치할 것이므로 당연히 yes를 클릭. (NTFS 로 부팅했다면 sda에 있는 USB를 unmount 할 것인지 물을 것이다. 이 경우는 No를 선택하자.) 그림이 작다면 클릭.

[![](/images/001.ubuntu-setup/inner/5.png)](/images/001.ubuntu-setup/inner/5.png)

다음으로 문제의 파티션 설정이다. 본인이 SSD를 사용하고 있고 메모리가 충분하다면, SSD 수명을 늘리기 위해 Swap partition을 사용하지 않는 것이 좋다. 필자는 과거에 이를 모르고 swap partition을 잡아서 설치하고 있었다. 이번에는 Swap 없이 설치해보자. 이를 위해 세부설정이 필수이니 Something else를 선택하여 세부 설정에 들어가자.

[![](/images/001.ubuntu-setup/inner/6.png)](/images/001.ubuntu-setup/inner/6.png)

파티션들이 쭉 나열되어 나온다. 기존 파티션을 클릭하고 -를 차례로 눌러주면, 아래와 같이 모든 파티션이 Free Space로 편입된다.

[![](/images/001.ubuntu-setup/inner/7.png)](/images/001.ubuntu-setup/inner/7.png)

다음으로 New Partition Table을 선택하면 현재 디스크의 모든 파티션이 지워진다고 경고가 나온다. 필자는 미련없이 삭제했다.

[![](/images/001.ubuntu-setup/inner/7-1.png)](/images/001.ubuntu-setup/inner/7-1.png)

이제 파티션을 나눠주자. /boot를 나누어주는 것도 좋지만 /(root)의 암호화와 같은 특수한 목적 이외에는 필요한 경우가 거의 없다. 앞서 말한 것 과 같이 ssd를 사용하고, 램 용량이 충분하다면, /swap도 필요 없다. 더군다나 용량이 부족한 ssd이기 때문에 파티션 조정으로 스트레스 받는것도 싫어서, 모두한 한방에 /에 할당했다. 그림이 작다면 클릭.

[![](/images/001.ubuntu-setup/inner/8.png)](/images/001.ubuntu-setup/inner/8.png)

솔직히 말하면 /home을 분리할당하는것은 큰 디스크에서 시스템을 새로 포맷해야 할 때 유용하다. 하지만 겨우 256G의 SSD라면, home을 나누지 않기를 권한다. 가장 큰 이유는 귀찮음이다. 포맷이 필요할 때 그냥 외장하드 드라이브나, 다른 시스템에 백업해 놓는 것을 권한다. 기가비트 네트워크를 통한다면 20~30분이면 백업할 수 있다.

이제 Install Now를 선택하자. 아마도 Swap이 없다고 경고를 띄울 것인데, 그냥 Continue를 클릭하면 된다.

[![](/images/001.ubuntu-setup/inner/9.png)](/images/001.ubuntu-setup/inner/9.png)

다음은 디스크 파티션이 지워질꺼라는 경고이다. Continue를 클릭하자. 그림이 작다면 클릭.

[![](/images/001.ubuntu-setup/inner/10.png)](/images/001.ubuntu-setup/inner/10.png)

이후에는 별거 없다. 일사천리로 세팅을 진행해보자. 서울을 찾아서 선택하자. 그림이 작다면 클릭.

[![](/images/001.ubuntu-setup/inner/11.png)](/images/001.ubuntu-setup/inner/11.png)

다음은 영어 키보드를 선택하자. 기본으로 선택되어 있으므로 그냥 Continue를 클릭하면 된다.

[![](/images/001.ubuntu-setup/inner/12.png)](/images/001.ubuntu-setup/inner/12.png)

마지막으로 유저 이름 설정이다. 입맛대로 채워주자.

[![](/images/001.ubuntu-setup/inner/13.png)](/images/001.ubuntu-setup/inner/13.png)

Continue를 누르면 설치가 진행된다.

[![](/images/001.ubuntu-setup/inner/14.png)](/images/001.ubuntu-setup/inner/14.png)

물론 설치는 어마무시하게 오래걸린다. ssd에 진행했음에도 불구하고 굉장히 오래 걸리는데 이유가 궁금하다. 답답해서 NTFS랑 FTP 둘다 해봤는데 Saving Installed Packages에서 어마어마한 시간을 잡아먹는다. 인내심을 기다리는게 좋을듯 하다.
긴긴 설치를 마치면 드디어 Ubuntu Desktop 화면을 볼 수 있다.  다음 포스팅에서는 기본적인 패키지 설치 등을 다루어 보려고 한다.
