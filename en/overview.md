## Game > GameAnvil > Overview

GameAnvil is a high-performing, real-time game server engine based on Java. It aims to shorten the development time of game servers and increase their performance and reliability. With GameAnvil, you can quickly and easily develop real-time game servers while benefiting from the rich Java ecosystem. Even novice developers can easily pick up and start developing real-time game servers and link them with clients developed in Unity or CocosCreator. The GameAnvil system also comes with a full suite of tools to help you not only develop game servers, but also to test their functionality and performance, as well as operate and monitor them in the cloud.

## Major References

Major games developed and serviced using GameAnvil are as follows.

![gameanvil-references.png](https://static.toastoven.net/prod_gameanvil/images/gameanvil-references.png)
## Characteristics
GameAnvil's ultimate goal is to help even less experienced developers easily develop and service real-time content. In this respect, we want to improve code productivity and ease of use, and ease technical entry barriers for possible users. We also aim to maximize the advantages of cloud products for easy service operation.

![gameanvil-references.png](https://static.toastoven.net/prod_gameanvil/images/overview-features.png)

## Recommended Games for Use

Currently GameAnvil is pretty much perfect for the following kinds of games.

* Casual game
* Turn-based strategy game
* Board game
* Indie games that require real-time content or DB storage
* Other games that you want to easily create and service real-time content


## Integration of Easy Unity Development Environments

Provides a **GameAnvil Connector** Unity package that can easily link with GameAnvil servers. This package allows you to link directly with GameAnvil servers in your existing Unity development environment.

#### Fast connectivity and authentication
* Connecting and authenticating to servers can be handled quickly and easily through a dedicated API.

#### Rich multiplayer API
* All the features necessary to implement multi-play games such as room creation, room entry, and matchmaking are available as an API.

#### Synchronization component support
* Just by registering components, synchronization between users is possible without a separate server implementation.
* Supports user-defined value synchronization in addition to coordinate synchronization, forced synchronization, and enumerator synchronization.

## Recommended Target Developers

![overview-target-developer.png](https://static.toastoven.net/prod_gameanvil/images/overview-target-developer.png)


#### 1. developers who know **Java**

* It's the best choice for developers who can handle Java. You can focus solely on your content on the processing flow that the engine provides.
* GameAnvil provides all APIs necessary for game server development, such as DB, Redis, and HTTP.

#### 2. Developers who have developed game servers in other languages, such as **C++, C#**

* Developers who have developed game servers in other languages can start developing content as soon as they learn the basic grammar of Java.
* There are no problems other than the differences between Java and other languages. Try to develop content while learning Java. GameAnvil developers also switched from C++ to Java without much difficulty.

#### 3. **Junior or new developer who has never developed a game server**

* Even if you've never developed a game server, you can easily and comfortably develop real-time content based on the guide documents and references provided.
* The engine takes care of most of the features that the real game server needs to handle, so you can only focus on the content.


## - License:

GameAnvil is protected by a unique [Software License](https://gameplatform.toast.com/kr/services/gameanvil/license). GameAnvil is only available if you agree to the corresponding software license.

## Games Not Yet Supported

GameAnvil is still in the evolving phase, so we don't support the following styles of games yet, but all our developers are doing our best to support them as soon as possible.

* MMO(RPG) game
* P2P-based games such as FPS or AOS

## Server Configuration

| Game style | Maximum number of concurrent users | Whether Real-time Content Present or not | Use Matchmaking | \** Minimum VM Count | Rough configuration                                |
| ----------- | ------------------- | ------------------ | --------------- | --------------- | ------------------------------------------- |
| Casual      | 10000               | O                  | O               | 5               | Gateway x 2, Game x 2, and other x 1            |
| Casual      | 10000               | X                  | X               | 3               | Gateway x 1, Game x 1, and other x 1            |
| Indi        | 3000                | O                  | X               | 1               | All in One x 1                              |
| Turn-based strategy game   | 200000              | O                  | O               | 49              | Gateway x 20, Game x 25, Loc x 3, and other x 1 |

** At any case, it's a rough estimate, depending on the volume of the actual content or the style of the game.

** Depending on the specifications of the VM, the number may also vary. 

** Storage such as DB or Redis is excluded from the count.

## Reference videos

* Can NDC 2021 [ Java develop real-time game server engine? Yes, it is possible ](https://youtu.be/kQyu5pAChcA)
* NHN Cloud On 2022 Webinar [On.5 Online game development is easy and fast ](https://cloudon.nhn.com/webinar_past?idx=6)

## Information on Processing of Personal Information

In the process of using GameAnvil service, customer can collect/use the user's personal information, and in this case, the customer is obligated to comply with relevant laws and regulations such as the Personal Information Protection Act.
In addition, in this process, there may be a consignment relationship between the customer and NHN Cloud regarding the processing of personal information. Customers in the position of a consignor may enter into a separate contract in writing with NHN Cloud, the consignee company, and may be notified by referring to the following information in the personal information processing policy operated by the customer.

* Consignee: NHN Cloud Corp.
* Consignment work: Provision of GameAnvil service