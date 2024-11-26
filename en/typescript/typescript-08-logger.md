## Game > GameAnvil > TypeScript 개발 가이드 > 로거

### GameAnvillLogger

커넥터 내부의 동작에 대한 로그를 받아보고 싶을 때 사용합니다.

#### 리스너 설정

아래와 같이 설정하면 커넥터 내부 동작 로그가 콘솔에 표시됩니다.

```typescript
GameAnvilLogger.logListener = console.log;
```

필요에 따라 별도의 출력 함수를 정의할 수도 있습니다.

```typescript
GameAnvilLogger.logListener = (message) => {
    // print message here
}
```