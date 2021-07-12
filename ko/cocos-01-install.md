## Game > GameAnvil > CocosCreator 개발 가이드 > GameAnvil 커넥터 설치

## GameAnvil 커넥터 설치

먼저 새 CocosCreator 프로젝트를 생성합니다. **Cocos Creator Dashboard > New Project > Empty Project**를 선택하고 새 프로젝트를 생성합니다. 

![new-project](https://static.toastoven.net/prod_gameanvil/images/client-2-new-project.png)

![new-project-empty](https://static.toastoven.net/prod_gameanvil/images/client-2-new-project-empty.png)

이제 생성된 프로젝트에 GameAnvil 커넥터를 추가합니다. GameAnvil 커넥터는 [여기](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript.zip)에서 다운로드 받을 수 있습니다. 다운로드한 파일의 압축을 풀어 assets 아래에 폴더를 만들어 넣습니다. 이때 다음과 같은 알림이 나타나면 **No**를 클릭합니다.

![set-as-a-plugin](https://static.toastoven.net/prod_gameanvil/images/client-2-set-as-a-plugin.png)

![gameanvil-project](https://static.toastoven.net/prod_gameanvil/images/client-2-gameanvil-project.png)

GameAnvil 커넥터는 protobufjs를 사용합니다. 그래서 GameAnvil 커넥터를 프로젝트에 포함하면 위와 같은 오류가 나오는 것을 볼수 있습니다. 

protobufjs는 Node.js의 install 기능을 이용해 프로젝트에 포함시킵니다. VSCode 터미널에서 다음 코드를 입력하여 package.json 파일을 생성합니다.

```
npm init
```

위 코드를 입력하면 package.json 파일 생성 프로세스가 시작되고 패키지 이름을 입력받습니다. 이때 그냥 Enter를 누르면 기본값으로 프로젝트 이름을 사용합니다. 이후로 입력받는 값들도 Enter를 누르면 기본값을 사용합니다. 자세한 내용은 [여기](https://docs.npmjs.com/cli/v6/commands/npm-init)를 참고하세요.  

![npm-init](https://static.toastoven.net/prod_gameanvil/images/client-2-npm-init.png)

기본값으로 생성한  package.json 파일을 열어보면 다음과 같습니다. 

```json
{
  "name": "newproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

package.json 파일을 생성했으면 이제 터미널에서 다음 코드를 입력하여 protobufjs를 설치합니다.

```
npm i protobufjs --save
```

![protobufjs](https://static.toastoven.net/prod_gameanvil/images/client-2-protobufjs.png)

설치가 완료되면 프로젝트 폴더 아래에 다음과 같은 폴더가 생성됩니다.

- node_modules
- .bin
- @protobufjs
- @types
- long
- protobufjs

package.json 파일의 dependencies에  gameanvil_connector가 추가된 것도 확인할 수 있습니다.

```json
{
  "name": "newproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "protobufjs": "^6.10.2"
  }
}
```

Internet Explorer 등 구버전 브라우저를 지원하려면 core-js, regenerator-runtime를 추가로 설치합니다.

```
npm i core-js regenerator-runtime
```

