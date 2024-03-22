## Game > GameAnvil > Cocos Creator Development Guide > Install GameAnvil Connector

## Install GameAnvil Connector

First, create a new CocosCreator project. Select **Cocos Creator Dashboard** New Project > Empty Project and create a new project. 

![new-project](https://static.toastoven.net/prod_gameanvil/images/client-2-new-project.png)

![new-project-empty](https://static.toastoven.net/prod_gameanvil/images/client-2-new-project-empty.png)

Now you add GameAnvil connectors to your created projects. GameAnvil connectors can be downloaded from [here](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript.zip). Unzip the downloaded file and create a folder under assets. 
At this time click on **No** when the following notification appears.

![set-as-a-plugin](https://static.toastoven.net/prod_gameanvil/images/client-2-set-as-a-plugin.png)

![gameanvil-project](https://static.toastoven.net/prod_gameanvil/images/client-2-gameanvil-project.png)

The GameAnvil connector uses protobufjs. Therefore, if the GameAnvil connector is included in a project, the following error will occur. 

protobufjs needs to be included in a project using the install feature of Node.js. Enter the following code in the VSCode terminal to create the package.json file.

```
npm init
```

When you enter the above code, the package.json file creation process starts and receives the package name.
At this time, simply press Enter to use the project name as the default value. The values you receive afterwards also use the default value when you press Enter. See [here](https://docs.npmjs.com/cli/v6/commands/npm-init) for more information.  

![npm-init](https://static.toastoven.net/prod_gameanvil/images/client-2-npm-init.png)

The default package.json file contains the following: 

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

When the package.json file is created, enter the following code in the terminal to install protobufjs.

```
npm i protobufjs --save
```

![protobufjs](https://static.toastoven.net/prod_gameanvil/images/client-2-protobufjs.png)

When the installation is complete, the following folder will be created in the project folder.

- node_modules
- .bin
- @protobufjs
- @types
- long
- protobufjs

gameanvil_connector will be added in the dependencies of the package.json file as well.

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

core-js and regenerator-runtime need to be installed to support older versions of browsers such as Internet Explorer.

```
npm i core-js regenerator-runtime
```

