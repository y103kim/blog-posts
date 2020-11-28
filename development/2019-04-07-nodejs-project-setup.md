---
layout: post
category: development
title: node.js Project 개발환경 setup
modified: 2019-04-07
tags: [Nodejs, javascript, vscode]
---

## 기본 프로그램 설치

1. node 설치 (zip파일로 설치하는 것을 선호)
2. 환경변수 설정
3. (Canvas) GYP 설치: npm install --global --production windows-build-tools
4. (Canvas) GTK 설치: [github wiki 참고](https://github.com/Automattic/node-canvas/wiki/Installation:-Windows#2-installing-gtk-2)
5. yarn 설치: ```npm install -g yarn```

## 프로젝트 셋업

이번 post에서는 [vis 프로젝트](https://github.com/almende/vis.git)를 대상으로 진행
1. 프로젝트 다운: ```git clone https://github.com/almende/vis.git```
2. 의존성 패키지 설치: ```yarn```
3. 디버깅용 패키지 설치: ```yarn add -g browserify babelify uglify-js watchify```

## VSCode

Extension 설치
1. Debugger for Chrome
2. .vscode/launch.json 파일 설정: file 부분만 원하는 html로 바꾸면 됨

```json
{
  "version":"0.2.0",
  "configurations": [
    {
        "name": "Launch index.html",
        "type": "chrome",
        "request": "launch",
        "file": "${workspaceFolder}/test.html"
    },
  ]
}
```

## 테스트 파일 셋업

- custom.js

```javascript
var DataSet = require("./lib/DataSet");
var Timeline = require("./lib/timeline/Timeline");

var groups = new DataSet([
  {id: 0, content: 'First', value: 1},
  {id: 1, content: 'Third', value: 3},
  {id: 2, content: 'Second', value: 2}
]);

// create a dataset with items
// note that months are zero-based in the JavaScript Date object, so month 3 is April
var items = new DataSet([
  {id: 0, group: 0, content: 'item 0', start: new Date(2014, 3, 17), end: new Date(2014, 3, 21)},
  {id: 1, group: 0, content: 'item 1', start: new Date(2014, 3, 19), end: new Date(2014, 3, 20)},
  {id: 2, group: 1, content: 'item 2', start: new Date(2014, 3, 16), end: new Date(2014, 3, 24)},
  {id: 3, group: 1, content: 'item 3', start: new Date(2014, 3, 23), end: new Date(2014, 3, 24)},
  {id: 4, group: 1, content: 'item 4', start: new Date(2014, 3, 22), end: new Date(2014, 3, 26)},
  {id: 5, group: 2, content: 'item 5', start: new Date(2014, 3, 24), end: new Date(2014, 3, 27)}
]);

// create visualization
var container = document.getElementById('visualization');
var options = {
  // option groupOrder can be a property name or a sort function
  // the sort function must compare two groups and return a value
  //     > 0 when a > b
  //     < 0 when a < b
  //       0 when a == b
  groupOrder: function (a, b) {
    return a.value - b.value;
  },
  editable: true
};

var timeline = new Timeline(container);
timeline.setOptions(options);
timeline.setGroups(groups);
timeline.setItems(items);
```

- test.html

```html
<!DOCTYPE HTML>
<html>
<head>
  <title>Timeline | Groups ordering</title>

  <style>
    body, html {
      font-family: arial, sans-serif;
      font-size: 11pt;
    }

    #visualization {
      box-sizing: border-box;
      width: 100%;
      height: 300px;
    }
  </style>


  <link href="dist/vis-timeline-graph2d.min.css" rel="stylesheet" type="text/css" />
  
</head>
<body>
<p>
  This example demonstrates custom ordering of groups.
</p>
<div id="visualization"></div>
<script src="output.js"></script>
</body>
</html>
```

## 빌드 및  Debugger 실행

1. ```watchify custom.js -t [ babelify --presets [es2015] ] -o output.js --debug```
2. Debugger F5를 눌러서 실행
