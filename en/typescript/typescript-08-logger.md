<!-- pre-align:aligned sig=033ad3290f52 -->

<a id="game-gameanvil-typescript-development-guide-logger"></a>
## Game > GameAnvil > TypeScript Development Guide > Logger { #game-gameanvil-typescript-development-guide-logger }

<a id="gameanvilllogger"></a>
## GameAnvillLogger { #gameanvilllogger }

Use to receive logs of the activity inside the connector.

<a id="listener-settings"></a>
### Listener Settings { #listener-settings }

If set as shown below, the connector internal behavior logs appear in the console:

```typescript
GameAnvilLogger.logListener = console.log;
```

You can also define separate output functions according to your needs.

```typescript
GameAnvilLogger.logListener = (message) => {
    // print message here
}
```