---
layout: post
category: react
title: React Lazylog Example
tags: [Typescript, React, react-lazylog, flask, streaming]
---

[react-lazylog](https://github.com/mozilla-frontend-infra/react-lazylog)는 Text 형태의 Streaming Response를 알아서 잘 뿌려주는 라이브러리다. 간단한 코드를 통해서 테스트 해봤다.

## Backend

일단 Streaming을 할 만한 대용량의 자료가 필요했는데, `adb logcat`이 딱이었다. Flask를 이용해서 아래와 같이 구현했다. 일단 아래와 같이 환경을 설정하자.

```
python -m venv env
source env/bin/activate
pip install flask flask-cors
```

Subprocess로 Pipe를 열고, Generator를 만들어서 Flask로 넘기면 된다. 핵심 함수는 `stream_with_context` 이다. 테스트이니 CORS는 그냥 전체로 열자.

```py
import subprocess as sp
from flask import Flask, stream_with_context, Response
from flask_cors import CORS

app = Flask(__name__)
cors = CORS(app, resources={r"/": {"origins": "*"}})

@app.route("/")
def adbSteram():
    p = sp.Popen(["adb", "logcat"], stdout=sp.PIPE)
    def gen():
        while True:
            yield p.stdout.readline()
    return Response(stream_with_context(gen()))

app.run(debug=True);
```

## Front-end

CRA로 앱을 하나 만들고 react-lazylog를 설치하자. 타입 스크립트로 써야하니 @types 패키지도 설치한다.

```
npx create-react-app react-practice --typescript
cd react-practice
npm install --save @types/react-lazylog react-lazylog 
```

App.tsx 파일을 아래와 같이 변경해주자.

```ts
import React from 'react';
import { LazyLog, ScrollFollow } from 'react-lazylog';

const App: React.FC = () => {
    const url = 'http://localhost:5000/';
    const style = { height: 500, width: 902 };
    return (
        <div style={style}>
            <ScrollFollow
                startFollowing
                render={({ onScroll, follow, startFollowing, stopFollowing }) => (
                    <LazyLog url={url} stream follow={follow} />
                )}
            />
        </div>
    );
};

export default App;
```

짜증나는 점은 `@types/react-lazylog` 패키지에 빠져있는 Attribute가 너무 많다는 것이다. 공식 사이트의 [예제](https://mozilla-frontend-infra.github.io/react-lazylog/#scrollfollow)도 안돈다. 이럴 경우 `node_modules/@types/react-lazylog/build/LazyLog.d.ts` 파일에, 대충 `any`타입으로 추가해주면 작동한다.

```typescript
export interface LazyLogProps {
    url: string;
    fetchOptions?: RequestInit;
    stream?: boolean;
    height?: string | number;
    width?: string | number;
    follow?: boolean;
    scrollToLine?: number;
    highlight?: number | number[];
    selectableLines?: boolean;
    formatPart?: (text: string) => ReactNode;
    onLoad?: () => any;
    onError?: (error: any) => any;
    onHighlight?: (range: Range) => any;
    rowHeight?: number;
    overscanRowCount?: number;
    containerStyle?: CSSProperties;
    style?: CSSProperties;
    // 요 아래 부분을 추가했다.
    enableSearch?: boolean;
    onScroll?: any;
}
```