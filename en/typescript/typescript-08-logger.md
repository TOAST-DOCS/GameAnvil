## Game > GameAnvil > TypeScript Development Guide > Logger

## GameAnvillLogger

Use to receive logs of the activity inside the connector.

### Listener Settings

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