## Game > GameAnvil > TypeScript 개발 가이드 > 설정

### GameAnvilConfig

커넥터 환경 설정을 할 수 있는 클래스입니다.

#### defaultRequestTimeoutMillis

타임아웃 기본 대기시간을 설정할 수 있습니다.

```typescript
GameAnvilConfig.defaultRequestTimeoutMillis = 3000;
```

#### packetTimeoutMillis

패킷이 지정된 시간안에 업데이트 되지 않으면, 연결해제 되었다고 판단합니다.
pingInterval 보다 높게 설정 해야 합니다.

```typescript
GameAnvilConfig.packetTimeoutMillis = 5000;
```

#### pingIntervalMillis

서버와의 연결을 확인하기 위해 Ping 메시지를 보내는 주기를 설정합니다.
사용하지 않을 경우 0으로 설정합니다.

```typescript
GameAnvilConfig.pingIntervalMillis = 3000;
```

#### useIPv6

접속시 IPv6 주소로 변환 여부를 설정합니다.

```typescript
GameAnvilConfig.useIPv6 = false;
```

#### useSocketNoDelay

소켓의 Nodelay 사용 여부를 설정합니다.

```typescript
GameAnvilConfig.useSocketNoDelay = true;
```