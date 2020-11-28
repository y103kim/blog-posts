---
layout: post
category: react
title: React-Electron App Tutorial (1)
modified: 2018-09-06
tags: [nodejs, web programming, electron, react]
---

# React-Electron 앱 설치하기

윈도우용 데스크톱 앱을 작성하기 위한 기록이기 때문에, Windows에 설치하는 것을 기준으로 작성하였다.

----

## System Installation

프로젝트 생성을 위해 아래의 순서로 설치를 진행해야 한다. 아래 List는 시스템에 설치하는 것들이다.

react와 electron을 설치하지 않는 것이 이상할 수 있다. 이는 패키지 관리를 프로젝트 별로 로컬 폴더에 설치 하는 nodejs의 특징 때문이다. 나중에 프로젝트 생성 후 설치하게 된다.

1. nodejs
2. yarn
3. create-react-app

### Install nodejs

먼저 nodejs를 설치한다. 참고로 Installer를 통해 설치할수도 있고 zip으로 다운받을 수도 있다. 필자의 경우는 zip으로 압축된 파일을 다운받았다.

1. [nodejs.org](http://nodejs.org)에서 설치 파일을 다운받자.
2. 적당한 경로에 압축을 풀자. 필자의 경우는 ```C:\node```에 압축을 풀었다.
3. 환경변수를 설정해주자. path에 ```C:\node```를 추가하면 된다.

이제 powershell이나 cmd를 열어서 두가지를 확인하자.

```
$ node --version
v8.11.4
$ npm --version
v5.6.0
```

### Install yarn

yarn은 npm과 유사한 nodejs의 패키지 매니저이다. 웹 상에서 검색해보면 두 패키지 매니저의 차이점은 yarn이 크게 아래와 같은 기능을 제공해준다는 점이다.

- offline cache나 lisence관련 기능을 제공
- 조금 더 빠르다?

yarn은 npm을 이용해서 설치한다. 시스템 전체에서 사용할 수 있도록 설치할 것이므로 먼저 설치될 위치를 확인하자.

```
$ npm root -g
c:\node\node_modules
```

```-g``` 는 global을 의미한다. 일반적으로 npm root를 하면 현재 작업중인 프로젝트의 npm root이 출력되는데, ```-g```를 붙이면 패키지들이 설치되는 시스템 디렉토리가 찍힌다. 필자의 경우 아까 ```c:\node```에 설치했으므로 그 경로가 찍히게 된다.

이제 yarn을 설치하자. 아까와 동일하게 ```-g``` 옵션을 주어서 시스템 경로에 설치한다.

```
npm install -g yarn
```

### Install create-react-app

create-react-app은 기본 react 앱과 기본적인 디렉토리 구조를 잡아주는 패키지이다. 실행하면 react와 관련 패키지까지 모두 설치해준다. 일단 설치방법은 아래와 같다.

```
yarn global add create-react-app
```

이후 환경 변수 추가가 필요하다. path에 ```yarn global bin```의 결과를 추가하자. 필자의 경우 아래와 같다.

----

## Local Installation

이제 시스템에 설치할 패키지들은 준비가 끝났다. 개별 프로젝트에 설치를 진행해보자.

### Execute create-react-app

위에서 설명한대로, react 앱을 만들기 위해서는 react가 필요가 없다. 왜나하면 프로젝트 디렉토리 내에 react가 설치되기 때문이다. 아래 명령으로 react를 설치하자.

```
create-react-app my-first-react-app
```

아마도 한참을 설치할 것이다. 폴더를 만들고, 그 안에 필요한 패키지까지 알아서 채워넣어준다. react도 자동으로 다운받는다.

참고로 회사나 고립된 망에서 위 작업을 할 경우 npm이나 yarn 모두 꿈쩍도 하지 않을 가능성이 있다. 이럴때는 아래 명령을 실행해서 proxy와 ssl 체크 여부, 인증서를 설치하자.

```
yarn config set strict-ssl false
npm config set strict-ssl false
npm config set proxy [your-proxy-addr]
npm config set https-proxy [your-proxy-addr]
```

### Install electron

이제 electron을 설치하자. 아까 만든 my-first-react-app 폴더로 들어가서 yarn으로 electron을 설치하면 된다.

```
yarn add -D electron@latest
```

다만 조금 독특한 점은 ```-D``` 옵션을 사용한다는 점인데, ```-D``` 옵션은 Dev Dependency로 패키지를 추가하라는 옵션이다. 설치 후 package.json에 들어가 보아도 devDependencies에 electron 항목이 추가되어 있다.

```
  "devDependencies": {
    "electron": "^2.0.8"
  },
```

devDependecny은 npm의 publish 기능을 사용할 때 중요하다. 이 패키지는 개발시에만 필요한 것이니, npm을 통해 다른 사람에게 배포될 때는 설치 할 필요가 없다고 알려주는 기능이다.

그러나 electron을 설치한 개발자가, npm을 통해 배포할 패키지를 개발할 가능성은 크지 않다. 보통은 데스크톱 앱 개발이 주 목적일 것이기 때문이다. 그러므로 있으나 아니나 크게 중요하지는 않다. 
;
다만 electron-builder같은 놈을 실행할 때, electron이 devDependency인지 체크하므로 주의할 것.

### Install foreman

기본적으로 electorn과 react를 함께 사용하려면 여러개의 Thread가 필요하다. 클라이언트 역할을 하는 electron과 전용 웹서버 역할을 하는 react가 동시에 돌아야 하기 때문이다.

때문에 Process 관리 패키지인 foreman이 필요하다. 사용법은 다음에 다루기로 하고 일단은 설치하자.

```
yarn add -D foreman
```

----

## 결론

자 이제 모든 준비가 끝났다. 하지만 현재 상태로는 react는 실행되지만, electron을 통해서 react가 생성한 화면을 보는 것을 불가능하다.

다음 포스팅에서는 electron을 통해서 react 화면을 띄워보자.
