---
layout: post
category: react
title: React-Electron App Tutorial (2)
modified: 2018-09-12
tags: [nodejs, web programming, electron, react]
---

# React-Electron 앱 Debug Mode 실행

윈도우용 데스크톱 앱을 작성하기 위한 기록이기 때문에, Windows에 설치하는 것을 기준으로 작성하였다. 이번 포스팅에서는 React-Electron 앱을 실행해 볼 것이다.

----

## React App 살펴보기

create-react-app을 통해서 app을 생성하면 아래와 같은 폴더 구조가 생성된다.

```
react-app
├─node_modules
│  └─...
├─public
│  ├─favicon.ico
│  ├─index.html
│  └─manifest.json
├─src
│  ├─App.css
│  ├─App.js
│  ├─App.test.js
│  ├─index.js
│  ├─index.css
│  ├─logo.svg
│  └─registerServiceWorker.js
├─package.json
├─README.md
└─yarn.lock
```

각각은 아래와 같은 용도이다.
- node_modules: 앞선 포스팅에서 설치한 nodejs 모듈들이 담겨있다.
- public: 실제 Client에 제공되는 파일이 담겨지는 폴더이다.
- src: react를 통해서 실행되는 파일이 담겨지는 폴더이다.
- package.json: 현재 앱에 대한 전반적인 정보가 담긴다.
- yarn.lock: yarn이 현재 설치된 패키지에 대한 정보를 기록한 파일이다.

## React와 Electron 각각 실행해보기

[이전 포스팅](http://doocong.com/react/react-electron-setup-1/)에서 설치를 마쳤지만, Electron 위에서 React가 돌게 만드려면 몇 단계의 추가적인 과정이 필요하다.

### React App 실행하기

먼저 React를 실행해보자. React는 Web Server 형태로 작동한다. [이전 포스팅](http://doocong.com/react/react-electron-setup-1/)에서 이미 create-react-app을 실행했기 때문에 문제 없이 실행될 것이다.

```
yarn start
```

잠시 후 기본 브라우저와 함께 http://localhost:3000/ 로 접속된 것이 보일 것이다. react의 로고가 빙글빙글 보인다면 성공이다.

### Electron 기본 앱 실행하기

이제는 Electron을 실행해보자. Electron을 실행하기 위해서는 실행 파일이 필요하다. [Electron tutorial page](https://electronjs.org/docs/tutorial/first-app#electron-development-in-a-nutshell)에서 가져온 소스를 이용하자. 이 파일을 ```src/electron-starter.js``` 라는 이름으로 생성한다.

```javascript
const {app, BrowserWindow} = require('electron')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let win

function createWindow () {
  // Create the browser window.
  win = new BrowserWindow({width: 800, height: 600})

  // and load the index.html of the app.
  win.loadFile('index.html')

  // Open the DevTools.
  win.webContents.openDevTools()

  // Emitted when the window is closed.
  win.on('closed', () => {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    win = null
  })
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.on('ready', createWindow)

// Quit when all windows are closed.
app.on('window-all-closed', () => {
  // On macOS it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // On macOS it's common to re-create a window in the app when the
  // dock icon is clicked and there are no other windows open.
  if (win === null) {
    createWindow()
  }
})

// In this file you can include the rest of your app's specific main process
// code. You can also put them in separate files and require them here.
```

추가로 ```index.html```파일을 root에 생성하자. 이 파일은 테스트 용으로 추후에는 삭제해도 무방하다.

```html
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using node <script>document.write(process.versions.node)</script>,
    Chrome <script>document.write(process.versions.chrome)</script>,
    and Electron <script>document.write(process.versions.electron)</script>.
  </body>
</html>
```

이제는 package.json 을 고쳐야 한다. package.json에서 electron의 entry로 사용되는 ```main```항목과, 현재 폴더에 설치된 node_module을 실행하기 위한 명령인 ```script```항목을 다음과 같이 변경하자.

```json
  "main": "src/electron-starter.js",
  "scripts": {
    "react-start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject",
    "electron": "electron ."
  },
```

자 이제 electron을 실행해보자. 아래 명령을 실행했을 때 Helloworld가 보이면 성공이다.

```
yarn run electron
```


## React-Electorn 동시에 실행하기

지금까지는 각각 실행했다면 이번에는 동시에 실행해보자. 기본적으로 두 가지 앱은 하나로 합쳐질 수 없다. 그러므로 두개의 프로세스를 동시에 실행하여 상호작용 하도록 만들어야 한다.

### Electron이 react를 참조하도록 수정하기

아까 index.html 파일을 생성했던 것을 기억할 것이다. 이 파일을 삭제하고, 대신에 react를 읽어올 수 있도록 변경해야 한다. react 서버가 실행되면 항상 electron을 참조하도록 ```loadFile``` 부분을 ```loadURL```로 변경하자. ```src/electron-starter.js``` 파일을 변경하면 된다. 

```js
const startUrl = process.env.ELECTRON_START_URL;
win.loadURL(startUrl);
```

### Electron과 React가 동시에 실행되도록 하기

지난번 포스팅에서 설치한 Foreman은 여러 프로세스를 동시에 실행 할 수 있도록 하는 프로세스다. ```Procfile```을 root에 만들고 ```nf start```명령으로 foreman을 실행하면 여러 프로세스가 동시에 실행된다. ```Procfile``` 파일을 아래와 같이 작성하자.

```
react: npm run react-start
electron: node src/electron-wait-react
```

이제 ```src/electron-wait-react.js``` 파일을 작성해야 한다. 이 파일은 react와 electron이 동시에 실행된 이후 electron이 react의 완전한 실행을 기다리도록 하는 파일이다. 2번째 줄에 포트 번호에 100을 빼주는 것은, Foreman이 원래 프로세스의 Port에 100만큼 offset을 주는 특성 탓이다.

```js
const net = require('net');
const port = process.env.PORT ? (process.env.PORT - 100) : 3000;

process.env.ELECTRON_START_URL = `http://localhost:${port}`;

const client = new net.Socket();

let startedElectron = false;
const tryConnection = () => client.connect({port: port}, () => {
        client.end();
        if(!startedElectron) {
            console.log('starting electron');
            startedElectron = true;
            const exec = require('child_process').exec;
            exec('yarn run electron');
        }
    }
);

tryConnection();

client.on('error', (error) => {
    setTimeout(tryConnection, 1000);
});
```

자 이제 Foreman 실행을 위해 package.json의 script를 수정하자.

```json
  "scripts": {
    "react-start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject",
    "electron": "electron .",
    "start" : "nf start"
  },
```

마지막으로 ```yarn start```를 입력해 주면 react 로고가 회전하는 화면을 Electron 위에서 볼 수 있다.

## 결론 

이번 포스팅에서는 react와 Electron을 함께 실행시켜 보았다. 하지만 아직 끝이 아니다. 배포를 위해서는 react-electron이 동시에 동작하도록 빌드할 수 있어야 한다. 다음 포스팅에서는 electorn-builder를 이용해 react-electron 앱을 빌드하는 방법을 다룬다.

## Reference

- <https://medium.freecodecamp.org/building-an-electron-application-with-create-react-app-97945861647c>