## Game > GameAnvil > TypeScript 開発ガイド > ロガー

## GameAnvillLogger

コネクタ内部の動作に関するログを受け取りたい時に使用します。

### リスナー設定

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
