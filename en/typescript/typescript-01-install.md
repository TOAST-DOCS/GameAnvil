## Game > GameAnvil > TypeScript 개발 가이드 > 설치

### GameAnvil Connector

커넥터는 GameAnvil 서버에 맞춘 클라이언트를 제작하기 위해 개발된 라이브러리입니다. GameAnvil Connector를 이용하면 GameAnvil에서 기본적으로 제공하는 패킷 전송, 유저 및 룸 기능 등을 간편하게 구현할 수 있습니다.


### gameanvil-connector.js 다운로드

커넥터는 아래에서 다운로드 받을 수 있습니다.

[gameanvil-connector-typescript.zip]()

npm을 통한 다운로드나 git을 통한 다운로드는 지원 예정입니다.

### gameanvil-connector.js 설치 예시

이 섹션에서는 GameAnvil Connector의 TypeScript 버전의 설치와 이용 방법에 대해 다룹니다. 

이 가이드에서는 node.js를 사용하기 때문에 미리 설치가 완료된 환경이 필요하며, 독자가 기본적인 node.js 지식이 있다고 가정하고 설명합니다. 만약 node.js 설치나 이용에 대해서 잘 모른다면 node.js 공식 자료를 참고하시길 바랍니다. node.js 외에 따로 사용하고 싶은 게임 제작용 라이브러리가 있다면 그것을 사용하실 수 있습니다. 예를 들어 Cocos나 Phaser 등의 html5 기반 게임 라이브러리를 사용하실 수 있습니다.

명령 프롬프트 또는 터미널을 실행합니다. 그리고 아래 명령어를 입력해 커넥터를 설치할 폴더를 생성합니다.

```cli
$ mkdir my-game
$ cd my-game
$ npm init
```

다운로드 받은 커넥터 파일을 압축을 풀고, 아까 생성한 my-game폴더 아래로 이동합니다. 그리고 npm을 통해 설치합니다.

```cli
$ npm install ./gameanvil-connector
```

이것으로 node.js 프로젝트에 gameanvil-connector 설치가 끝났습니다. 프로젝트 내부에서 아래와 같이 "gameanvil-connector"로 라이브러리를 사용하실 수 있습니다.

```typescript
// index.ts
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();

connector.host = "127.0.0.1";
connector.port = 18300;
await connector.connect();
```

위 코드는 커넥터를 사용하는 제일 간단한 예시 코드입니다. 간단히 동작을 설명하자면 아래와 같습니다.

1. 먼저 GameAnvilConnector 객체를 생성합니다. GameAnvilConnector는 서버와의 통신을 관리하고 서버에게 요청을 보내거나 서버의 응답을 받을 수 있는 객체입니다.
2. 그 다음 GameAnvilConnector 객체에 연결 정보(host, port)를 입력합니다. 이것은 개발 서버에 따라 내용이 달라질 수 있으므로, 반드시 개발된 서버에 맞게 내용을 수정해주십시오.
3. 마지막으로 connect() 함수를 호출하여 서버에 연결 요청을 보냅니다.

### gameanvil-connector.js 실행 예시

예제 코드를 실행하기 위해 Webpack을 설치하고 실행해보겠습니다.
index.html 파일을 생성합니다.

```html
<!DOCTYPE html>
<head>
    <title>My Game</title>
</head>
<body>

</body>
```

필요한 라이브러리들을 모두 설치합니다.

```cli
$ npm install webpack webpack-cli webpack-dev-server html-webpack-plugin babel-loader @babel/core  @babel/preset-env @babel/preset-typescript
```

webpack 설정파일을 생성하고 아래와 같이 작성합니다.

```javascript
// webpack.config.cjs
const webpack = require( 'webpack');
const path = require( 'path');
const HtmlWebpackPlugin = require( 'html-webpack-plugin');


module.exports = {
    mode: "development",
    devtool: 'source-map',
    entry: {
        entry: ['./index.ts']
    },
    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'bundle.js',
    },
    resolve: {
        extensions: ['.ts', '.js'],
    },
    devServer: {
        watchFiles: path.join(__dirname, 'dist'),
        compress: false,
        port: 8081
    },
    module: {
        rules: [
            {
                test: /\.(d\.ts|ts|js)$/,
                exclude: '/node_modules/',
                loader: 'babel-loader',
                options: {
                    presets: [
                        '@babel/preset-env', '@babel/preset-typescript'
                    ]
                },
                resolve: {
                    fullySpecified: false
                }
            },
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './index.html',
        })
    ]
};
```

package.json 파일에 아래 스크립트를 추가합니다.

```json
"scripts": {
  "start": "webpack serve --progress --open --config ./webpack.config.cjs"
},
```

작동을 좀 더 자세히 확인하기 위해, 예제 코드를 수정합니다.

```typescript
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();
connector.host = "127.0.0.1";
connector.port = 18300;

connector.connect()
  .then(()=>{
    // connect에 성공한 경우
    document.body.append("Connect Success");
  })
  .catch(()=>{
    // connect에 실패한 경우
    document.body.append("Connect Fail");
  })
```

웹팩을 실행시키고 동작을 확인합니다.

```
$ npm run start
```

인터넷 브라우저 창이 뜨면, Connect Success 또는 Connect Fail 문구가 출력됩니다.