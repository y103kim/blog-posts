---
layout: post
category: react
title: React-Electron App Tutorial (3)
modified: 2018-09-12
tags: [nodejs, web programming, electron, react]
---

# React-Electron 앱 빌드 후 실행

윈도우용 데스크톱 앱을 작성하기 위한 기록이기 때문에, Windows에 설치하는 것을 기준으로 작성하였다. 이번 포스팅에서는 React-Electron 앱을 빌드해 볼 것이다.

## React App 폴더 정리하기

[React-Electron App Tutorial (1)](http://doocong.com/react/react-electron-setup-1/) 에서 create-react-app을 이용해서 앱을 생성했기 때문에, React에는 사용하지 않는 기본 파일들이 존재할 것이다. 아래와 같다. 지난번에 만든 `electron*.js`만 제외하고는 모두 삭제하자.

```
├─src
   ├─electron-starter.js
   ├─electron-wait-react.js
   ├─index.js
```

그리고 `index.js`에 Hello world 코드를 작성하자.

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
    <div>
        Hello World!
    </div>,
     document.getElementById('root')
);
```

## electron-builder 설치 및 세팅

아래 명령으로 설치하자.

```
yarn add -D electron-builder
```

### Load 할 HTML 설정하기

이제 몇가지 설정을 해주어야 한다. [React-Electron App Tutorial (2)](http://doocong.com/react/react-electron-setup-2/) 에서는 ```electron-wait-react.js```과 ```foreman```을 이용해서 두 개으 프로세스를 실행했었다.

그러나 electron-builder를 사용할 경우에는 이미 빌드가 완료된 react 파일을 사용할 것이기 때문에, 두개의 프로세스는 필요하지 않다. 단지 빌드된 react app의 폴더 명만 알려주면 된다. react를 빌드할 경우 결과물은 ```build```폴더에 생성되기 때문에 ```electron-starter.js```에서 ```loadURL``` 부분을 아래와 같이 변경하자.

```javascript
  const startUrl = process.env.ELECTRON_START_URL || url.format({
    pathname: path.join(__dirname, '/../build/index.html'),
    protocol: 'file:',
    slashes: true
  });
  win.loadURL(startUrl);
```

직접 실행할 경우 `electron-wait-react.js`에서 `ELECTRON_START_URL`을 설정해 주기 때문에, `startUrl`이  `ELECTRON_START_URL`으로 설정될 것이다. 하지만 build 할 경우 `electron-wait-react.js`을 거치지 않기 때문에, 현재 폴더인 `src` 기준으로 `../build/index.html`에 있는 파일을 실행하게 된다.

### package.json 설정하기

다음은 ```package.json```에 몇가지 설정을 추가로 해주어야 한다.

첫 번째는 homepage 설정이다. ```create-react-app```이 파일을 참조할 때 절대경로를 사용하기 때문에, 이를 상대경로로 바꾸기 위해서 아래 설정을 해주어야 한다.

```json
  "homepage": "./",
```

두 번째는 Script 설정이다. `dist` 명령과 `react-build`명령을 아래와 같이 설정하자. dist에서 `yarn run react-build`는 react app을 먼저 빌드하는 부분이고, `build -w`는 electron-builder를 통해 electron app을 빌드하는 부분이다.

```json
  "scripts": {
    "react-start": "react-scripts start",
    "react-build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject",
    "electron": "electron .",
    "start": "nf start",
    "dist": "yarn run react-build && build -w"
  },
```

세 번째는 `build` 부분을 설정해 주어야 한다. 빌드 타입을 설정할 수도 있다. `zip` 대신에 다른 옵션을 넣어주면 된다. 인스톨러를 사용하려면 `nsis`, 포터블 exe로 만드려면 `portable`을 넣어주자.

```json
  "build": {
    "extends": null,
    "directories": {
      "buildResources": "public"
    },
    "target" : "zip"
  }
```

## 빌드하고 결과 확인하기

아래 명령으로 빌드하자. 약 1분이 걸린다.

```
yarn run dist
```

`dist` 폴더 안에는 설정한 인스톨러나, PE파일이 들어있다. `dist\win-unpacked` 폴더 안에 가보면 exe 파일을 볼 수 있다. `dist\win-unpacked` 폴더의 모습이 눈에 익다면, 당신은 Visual Studio Code나 Atom 사용자일 가능성이 높다. 왜냐면 Visual Studio Code나 Atom도 Electron-React 앱이기 때문이다.


## Reference

- <https://medium.freecodecamp.org/building-an-electron-application-with-create-react-app-97945861647c>
- <https://www.electron.build/configuration/configuration>
- <https://gist.github.com/matthewjberger/6f42452cb1a2253667942d333ff53404>
