## Game > GameAnvil > Server Development Guide > Getting Started

This section explains how to develop a game server using GameAnvil.

## System requirements

| Development language  | Development IDE      | Supported OS               | Manage project | Available network protocol | Security |
| ---------- | ------------- | --------------------- | ------------- | ----------------------------- | ---- |
| Java 8, 11 | IntelliJ IDEA | Linux, Windows, MacOS | Maven         | TCP/IP, WebSocket, HTTP/HTTPS | SSL  |



## Creating a chatting server within a minute

GameAnvil provides its own templates to quickly finish basic framework. The user can create a basic game server by clicking a couple of times when using templates. The server created in this way includes a simple chatting feature. For more information, refer to the server template guide below:

- [Download template](http://static.toastoven.net/prod_gameanvil/files/GameAnvil-Template.zip)

- Walkthrough

- Open IntelliJ and select **File > Settings**.

   ![GameAnvilTemplate-1](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-1.png)

- **Import Settings** Select the **GameAnvil-Template.zip** file downloaded from the dialog box and import it.
  ![GameAnvilTemplate-2](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-2.png)

- Once import is done, restart IntelliJ.

- When it is restarted, a dialog box appears as below. Select **Create New Project**.     ![GameAnvilTemplate-3](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-3.png)

- Now, the previously registered template can be selected. Click the **Next** button to apply.    ![GameAnvilTemplate-4](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-4.png)

- If the window prompting **Enable Auto-Import** appears as below while applying, select **Enable**.    ![GameAnvilTemplate-5](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-5.png)

- Once everything is finished, the server template project's source code appears. When **Run** is run, the chatting server is initiated as below:    ![GameAnvilTemplate-6](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-6.png)



## Running server

Once the basic server configuration is finished using a template, it can be run. If the server runs without problem, the **onReady** log is displayed per node. Now use the client to connect to the prepared server.

```
[2020-03-13 17:36:16,925] [INFO ] [MANAGEMENT@1005@ThinkServer@1] [c.n.t.m.ManagementNode] onReady
[2020-03-13 17:36:20,008] [INFO ] [LOCATION@30122@ThinkServer@2] [c.n.t.l.LocationNode] onReady
[2020-03-13 17:36:18,316] [INFO ] [SPACE@1@ThinkServer@0] [c.n.t.s.i.SpaceNode] onReady
[2020-03-13 17:36:19,008] [INFO ] [SESSION@1001@ThinkServer@2] [c.n.t.s.i.SessionNode] onReady
```



## Reference

GameAnvil provides not only templates but also reference projects. We implemented a variety of features to the basic framework for GameAnvil users to refer to. Reference clients for servers and references like this can be checked from a separate menu. In addition, refer to the GameAnvil API and UML diagrams provided in a JavaDoc document.

[GameAnvil JavaDoc API Reference Site](http://10.162.4.61:9090/gameanvil)(If JavaDoc site is included to NHN Cloud, it is scheduled to replace with that domain)
