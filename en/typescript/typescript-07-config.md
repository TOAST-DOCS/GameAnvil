## Game > GameAnvil > TypeScript Development Guide > Settings

## GameAnvilConfig

A class that can set the connector environment.

### defaultRequestTimeoutMillis

You can set the timeout default wait time.

```typescript
GameAnvilConfig.defaultRequestTimeoutMillis = 3000;
```

### packetTimeoutMillis

If the packet is not updated within the specified time period, it is judged that it has been disassociated. It must be set higher than pingInterval.

```typescript
GameAnvilConfig.packetTimeoutMillis = 5000;
```

### pingIntervalMillis

Set the interval of sending Ping messages to confirm the connection to the server. If not enabled, set it to 0.

```typescript
GameAnvilConfig.pingIntervalMillis = 3000;
```

### useIPv6

Set whether to convert to an IPv6 address when connecting.

```typescript
GameAnvilConfig.useIPv6 = false;
```

### useSocketNoDelay

Set whether to use Nodelay on the socket.

```typescript
GameAnvilConfig.useSocketNoDelay = true;
```