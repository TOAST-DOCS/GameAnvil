<!-- pre-align:aligned sig=033ad3290f52 -->

<a id="game-gameanvil-typescript-development-guide-logger"></a>
## Game > GameAnvil > TypeScript 開発ガイド > ロガー { #game-gameanvil-typescript-development-guide-logger }

<a id="gameanvilllogger"></a>
## GameAnvillLogger { #gameanvilllogger }

コネクタ内部の動作に関するログを受け取りたい時に使用します。

<a id="listener-settings"></a>
### リスナー設定 { #listener-settings }

以下のように設定すると、コネクタ内部の動作ログがコンソールに表示されます。

```typescript
GameAnvilLogger.logListener = console.log;
```

必要に応じて別途の出力関数を定義することもできます。

```typescript
GameAnvilLogger.logListener = (message) => {
    // print message here
}
```
