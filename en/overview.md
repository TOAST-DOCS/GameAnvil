## Game > GameAnvil > Overview

GameAnvil is a high-performing real-time game server engine based on Java. The goal is to reduce the game server's development time and increase performance and reliability.
Build your real-time game servers easily and quickly while taking advantage of the rich Java ecosystem with GameAnvil. Even beginner developers can easily learn to develop a real-time game server to immediately integrate developing clients with Unity or Type Script, etc.
In addition, the GameAnvil system not only provides development of game servers, but also feature/performance tests and even operations and monitoring in the cloud.

## Official Reference Game

The following are the main games being developed and serviced using GameAnvil:

![gameanvil-references.png](https://static.toastoven.net/prod_gameanvil/images/gameanvil-references.png)

## Character

GameAnvil's final goal is to make it super easy for even inexperienced developers to develop and service real-time content. In that sense, we want to increase code productivity and ease of use, and lower technical entry barriers for possible users.
We also want to maximize the benefits of our cloud products to make it easier to operate our services.

#### Increase Code Productivity

* When you write simple and easy sequential codes, your engine takes it to perform high-performing asynchronous processing based on Virtual Thread.

#### Reliable Performance

* Provide optimal asynchronous processing and reliable performance based on a high-performance library.

#### Easy to Use

* We also support server management and monitoring on the cloud and even tests.

#### Flexible Server Configuration

* From small-scale games to large-scale games, the optimal configuration is possible according to the scale and characteristics of the service.

## Recommended Games

Currently, GameAnvil is almost perfect for the following types of games:

* Casual game
* Turn-based game
* Board game
* Indie games that require real-time content or DB repository, etc.
* Other games that want to easily write and serve real-time content

## Integrate an Easy to Use Unit Development Environment

Provide a package of **GameAnvil connectors** that can be easily integrated with the GameAnvil server. This package lets you connect an existing unit development environment directly to the GameAnvil server.

#### Fast Connection and Authentication

* Connect and authenticate to the server easily and quickly via a dedicated API.

#### Rich Multiplayer API

* With API, you get all the features you need to implement multiplayer games, such as room creation, room input, and matchmaking.

#### Support Synchronization Component

* Only registering components allows synchronization between users without a separate server implementation.
* Support user-defined value synchronization in addition to coordinate synchronization, force synchronization, and animator synchronization.

## Recommended Developers

Comfort and flexibility allowing easy development for even **less experienced developers** to provide reliable service.

| Easy Development | Reliability | Comfort | Flexibility |
|-------------------------------------------------------------------------- |-------------------------------------------------------------------------------------------------------- |-------------------------------------------------------------------- |------------------------------------------------------------------ |
| Java<br/>Virtual Thread<br/>Continuation<br/>Single Thread<br/>Lock Free | No SPOF<br/>Monitoring<br/>Load Balance<br/>Non-Stop Patchable<br/>Connection Recovery | PaaS<br/>Test Support<br/>Connectors<br/>Documentation<br/>Samples | Node<br/>Runtime Scalable<br/>TCP / IP<br/>WebSocket<br/>HTTP(S) | 

#### 1. Developers who know how to handle **Java**

* This is the best choice for developers familiar with Java. You can only focus on the content over the processing flow provided by the engine.
* GameAnvil offers all the APIs required to develop a game server, such as DB, Redis, and HTTP.

#### 2. Developer who has developed a game server in other languages, such as **C++, C#** and more.

* Developers that have developed a game server in a different language can immediately start developing content only when they learn the basic syntax of Java.
* Besides the difference that comes from Java and other languages, there is no problem. Learn Java and continue with content development. GameAnvil has been transitioning from C++ to Java without much difficulty.

#### 3. **A junior or beginner developer who has never developed a game server**

* Even if you have never developed a game server, you can easily and conveniently develop real-time content based on the guidelines and references provided.
* Most functions that need to be handled on a real-world game server are led by the engine, so you can focus on content only.

## Unsupported Games

GameAnvil is still in the evolutionary phase. However, the following games are not supported yet. But now all developers are doing their best to get support within a quick time.

* MMO (RPG) games
* P2P-based games such as FPS or AOS

## Configure Reference Server

| Game Style | Max Concurrent Players | Live Content | Matchmaking | \** Minimum Number of VMs | Approximate Configuration |
|--------|-------------|-------------|-------------|--------------|-------------------------------|
| Casual | 10000 | Yes | Yes | 5 | 2 Gateways, 2 Game, 1 Other |
| Casual | 10000 | X | X | 3 | 1 Gateway, 1 Game, 1 Other |
| Indie | 3000 | Yes | X | 1 | All in One |
| Turn-Based | 200000 | Yes | Yes | 49 | 20 Gateways, 25 Game, 3 Loc, 1 Other |

** This is only an approximate figure. It may vary depending on the actual content volume or game style.

** The number may also vary depending on the specifications of the VM.

** Storages such as DB or Redis are excluded from the number.

## Additional Videos

* NDC 2021 [Is it possible to develop a real-time game server engine using Java? Yes, it is possible.](https://youtu.be/kQyu5pAChcA)
* NHN Cloud On 2022 Webinar [On.5 Easy and Fast Online Game Development](https://www.youtube.com/watch?v=Uv2a6fAU1xM)

## Information on Personal Information Processing

In the process of using GameAnvil services, customers may collect and use customers’ personal information, in which case the customer is bound by applicable law, such as the privacy law.
In addition, a business trust relationship regarding the processing of personal information between the customers and NHN Cloud may occur during this process.
Customers who are at the point of contact can enter a separate written consignment agreement with NHN Cloud, a consignment company, which is operated by customers, with reference to the privacy policy provided below:

* Subcontractor: NHN Cloud Co., Ltd.