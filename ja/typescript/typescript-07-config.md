<!-- pre-align:aligned sig=8bebccc60758 -->

<a id="game-gameanvil-typescript-development-guide-settings"></a>
## Game > GameAnvil > TypeScript 開発ガイド > 設定 { #game-gameanvil-typescript-development-guide-settings }

<a id="gameanvilconfig"></a>
## GameAnvilConfig { #gameanvilconfig }

コネクタ環境設定ができるクラスです。

<a id="defaultrequesttimeoutmillis"></a>
### defaultRequestTimeoutMillis { #defaultrequesttimeoutmillis }

タイムアウト基本待機時間を設定できます。

```typescript
GameAnvilConfig.defaultRequestTimeoutMillis = 3000;
```

<a id="packettimeoutmillis"></a>
### packetTimeoutMillis { #packettimeoutmillis }

パケットが指定された時間内に更新されない場合、接続解除されたと判断します。
pingIntervalより高く設定する必要があります。

```typescript
GameAnvilConfig.packetTimeoutMillis = 5000;
```

<a id="pingintervalmillis"></a>
### pingIntervalMillis { #pingintervalmillis }

サーバーとの接続を確認するためにPingメッセージを送る周期を設定します。
使用しない場合は0に設定します。

```typescript
GameAnvilConfig.pingIntervalMillis = 3000;
```

<a id="useipv6"></a>
### useIPv6 { #useipv6 }

接続時にIPv6アドレスへ変換するかどうかを設定します。

```typescript
GameAnvilConfig.useIPv6 = false;
```

<a id="usesocketnodelay"></a>
### useSocketNoDelay { #usesocketnodelay }

ソケットのNodelay使用有無を設定します。

```typescript
GameAnvilConfig.useSocketNoDelay = true;
```
