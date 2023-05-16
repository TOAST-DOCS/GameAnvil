## Game > GameAnvil > Overview

GameAnvil is a Java-based, real-time game server engine, used by a number of game projects.

Use GameAnvil to leap the benefit of the rich Java ecosystem and easily and quickly develop a game server. Use the provided client connector, test tools, and web console to easily develop and prepare game services.



## Key Features

- Fiber based Continuation support and sequential code flow
- Single threading and Lock Free
- Automatically restores previous session when reconnecting
- Supports user matchmaking
- Supports room matchmaking
- No SPOF(Single Point Of Failure)
- Supports run-time server scaling
- Transfers users and rooms between servers
- Supports dedicated test systems
- Supports dedicated the web console
- Supports client connector (Unity, CocosCreator)
- Supports uninterrupted maintenance and patch

## Term

The explanation of the terms used by GameAnvil.

| Term               | Description                                                         |
| ------------------ | ------------------------------------------------------------ |
| Machine               | The machine on which an GameAnvil instance (process) is run                 |
| Instance           | The run unit of a GameAnvil process (JVM)                            |
| Node               | The basic units of the GameAnvil server configuration [detailed information](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-2-basic) |
| Master machine        | The machine on which Management node is run                              |
| Location management machine | The machine on which Location Node is run                                |
| Machine settings          | The task that is used to specify master machine and location management machine             |
| Setup template        | Stores the setting value per node and provides them to be used in multiple instances |

## Libraries using GameAnvil

There are four key libraries used by GameAnvil: Because Quasar, ZeroMQ, and Netty are used in the engine, GameAnvil users are not likely directly use them. Protocol Buffers is used when parallelize/serialize messages. Understanding the following four libraries, regardless of direct use, helps when using the engine.

| Library       | Purpose                            |
| ---------------- | ------------------------------- |
| Quasar           | Supports Fiber-based Continuation |
| ZeroMQ           | Server's IPC                      |
| Netty            | Communication between server and client            |
| Protocol Buffers | Parallelization of messages between server and client   |

## Information on Privacy Policy

In the process of using GameAnvil service, the customer may collect/use personal information of users, and in this case, the customer is obliged to comply with relevant laws such as the Personal Information Protection Act.
Also during this process, work consignment relation regarding the processing of personal information may arise between the customer and NHN Cloud. The customer who assumes the position of consignor may enter into a consignment contract with the consignee, NHN Cloud, separately in writing, and post a privacy policy notice by referencing the following.

* Consignee: NHN Cloud Corp.
* Consignment work: Provision of GameAnvil service