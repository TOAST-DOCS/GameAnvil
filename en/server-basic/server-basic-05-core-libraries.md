## Game > GameAnvil > Server Concept Description > Core Libraries



## Key Libraries

The four key libraries used by GameAnvil are listed below. Quasar, ZeroMQ, and Netty are used inside the engine, so GameAnvil users don't need to use them themselves. Protocol Buffers are used to serialize/deserialize messages. If you understand these four libraries well, whether you use them directly or not, it will help you use the engine a lot.

| Library       | Usage                            |
| ---------------- | ------------------------------- |
| Quasar           | Supports Fiber-based Continuation |
| ZeroMQ           | Server's IPC                      |
| Netty            | Communication between server and client            |
| Protocol Buffers | Parallelization of messages between server and client   |

In particular, if you need to use the library directly during game development, we recommend that you use the same version as GameAnvil, rather than using another version separately. For example, if you need to use Netty, rather than adding dependencies for any version, please use the Netty that GameAnvil is using.
