## Game > GameAnvil > CocosCreator Development Guide > Connector

## Connector

To use GameAnvil connector, you have to create Connector first. It is for basic setting and agent management, and you can register a callback to view logs related to internal behavior.

### Create

You can create Connector as follows:

``` typescript
import { Connector } from 'GameAnvil/gameanvil';
let connector = Connector.Create();
```

### Set up

There are settings in which the Connector is used for operation. These settings are set to the default values when the Connector is created, but you can change the values directly, if necessary. 

``` typescript
import { Connector } from 'GameAnvil/gameanvil';
let connector = Connector.Create();
connector.config.PacketTimeout = 10;
```

The types of settings are as follows:

| Name                 | Description                                                         | Default value |
| -------------------- | ------------------------------------------------------------ | ------ |
| defaultReqestTimeout | TimeOut Default Wait Time Settings                                   | 3(sec) |
| packetTimeout        | If the packet is not updated within the specified time, it is determined that the connection has been disconnected. It should be set higher than the pingInterval. | 5(sec) |
| pingInterval         | Set ping cycle to verify connection to the server; not available if 0 | 3(sec) |

### Update

You have to call Update() function periodically to handle messages sent and received from the server. 
You are free to set the call cycle for Update() function, but if you do not, you will not be notified if you receive a message from the server.

```typescript
setInterval(() => { connector.Update(); }, 10);
```

### ConnectHandler

Create and use the following single-turn class for convenience. 

```typescript
/* 
To use the connector in an older version of browser, such as Internet Explorer
    You have to use plugin like core-js, regenerator-runtime, etc. 
*/ 
import 'core-js'; 
import 'regenerator-runtime'; 
 
import { Connector, ProtocolManager } from 'GameAnvil/gameanvil'; 

export default class GameAnvilManager { 
    private static manager: GameAnvilManager; 
    private connector: Connector; 
    private constructor() { 
        // Register protocol. 
        ProtocolManager.RegisterProtocol(0, GameMessages); 
 
        // Create connector. 
        this.connector = Connector.Create(); 
 
        // Message Loop. Call every 10ms. 
        let updater = setInterval(() => { this.connector.Update(); }, 10); 
    } 
 
    static GetInstance() { 
        if (this.manager == null) { 
            this.manager = new GameAnvilManager(); 
        } 
        return this.manager; 
    } 
 
    GetConnectionAgent() { 
        return this.connector.GetConnectionAgent(); 
    } 
 
    GetUserAgent(serviceName: string, subId: string) { 
        return this.connector.GetUserAgent(serviceName, subId); 
    } 
 
    CreateUserAgent(serviceName: string, subId: string) { 
        return this.connector.CreateUserAgent(serviceName, subId); 
    } 
}
```

This code is an example and can be altered as necessary. Now, the GameAnvil connector is ready to be used.