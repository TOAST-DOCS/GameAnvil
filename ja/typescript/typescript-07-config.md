## Game > GameAnvil > TypeScript 開発ガイド > 設定

## GameAnvilConfig

コネクタ環境設定ができるクラスです。

### defaultRequestTimeoutMillis

タイムアウト基本待機時間を設定できます。

```typescript
GameAnvilConfig.defaultRequestTimeoutMillis = 3000;
```

### packetTimeoutMillis

パケットが指定された時間内に更新されない場合、接続解除されたと判断します。
pingIntervalより高く設定する必要があります。

```typescript
GameAnvilConfig.packetTimeoutMillis = 5000;
```

### pingIntervalMillis

サーバーとの接続を確認するためにPingメッセージを送る周期を設定します。
使用しない場合は0に設定します。

```typescript
GameAnvilConfig.pingIntervalMillis = 3000;
```

### useIPv6

接続時にIPv6アドレスへ変換するかどうかを設定します。

```typescript
GameAnvilConfig.useIPv6 = false;
```

### useSocketNoDelay

ソケットのNodelay使用有無を設定します。

```typescript
GameAnvilConfig.useSocketNoDelay = true;
```
