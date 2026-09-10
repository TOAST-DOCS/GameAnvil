<!-- pre-align:aligned sig=8bebccc60758 -->

<a id="game-gameanvil-typescript-development-guide-settings"></a>
## Game > GameAnvil > TypeScript Development Guide > Settings { #game-gameanvil-typescript-development-guide-settings }

<a id="gameanvilconfig"></a>
## GameAnvilConfig { #gameanvilconfig }

A class that can set the connector environment.

<a id="defaultrequesttimeoutmillis"></a>
### defaultRequestTimeoutMillis { #defaultrequesttimeoutmillis }

You can set the timeout default wait time.

```typescript
GameAnvilConfig.defaultRequestTimeoutMillis = 3000;
```

<a id="packettimeoutmillis"></a>
### packetTimeoutMillis { #packettimeoutmillis }

If the packet is not updated within the specified time period, it is judged that it has been disassociated. It must be set higher than pingInterval.

```typescript
GameAnvilConfig.packetTimeoutMillis = 5000;
```

<a id="pingintervalmillis"></a>
### pingIntervalMillis { #pingintervalmillis }

Set the interval of sending Ping messages to confirm the connection to the server. If not enabled, set it to 0.

```typescript
GameAnvilConfig.pingIntervalMillis = 3000;
```

<a id="useipv6"></a>
### useIPv6 { #useipv6 }

Set whether to convert to an IPv6 address when connecting.

```typescript
GameAnvilConfig.useIPv6 = false;
```

<a id="usesocketnodelay"></a>
### useSocketNoDelay { #usesocketnodelay }

Set whether to use Nodelay on the socket.

```typescript
GameAnvilConfig.useSocketNoDelay = true;
```