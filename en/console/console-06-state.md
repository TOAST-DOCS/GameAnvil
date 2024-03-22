## Game > GameAnvil > Console User Guide > State


## State
Multiple game servers can be configured for a single game service. And each game server can be configured with multiple nodes. These servers and nodes have different state values. The rest of this document describes the state of the server and nodes.



## Server State

Server states are a combination of the state of processes (S/W) and instances (H/W). The dashboard, which you can see on the Server Management page, lists these server’s state and displays the number of servers in each state. It also displays each state in a distinct color.

![Figure](https://static.toastoven.net/prod_gameanvil/images/console/state/dashboard.png)

<br>
The following describes each server state.

| Server State      | Description                                                                                        |
|------------|-------------------------------------------------------------------------------------------|
| ERROR      | The server represents all anomaly conditions. In this case, only reboot command is available in the server. another command is enabled when rebooted and entered into the STANDBY state. |
| RUNNING    | The server is normally running. In other words, the game server process has started well without any issues.                                       |
| STANDBY    | The server instance has been booted successfully and can communicate with the GameAnvil service. In this case, you can run the game server process using the start command.    |
| SAFE PAUSE | Some nodes on the server are performing SafePause.                                |
| TRANSIT    | Switching between two states. You cannot command that server until the transition to the target state is completed.                                |


## Node State

Node state shows the state of multiple nodes that make up a game server. Also, nodes on the same server can be in different state. 

| Node State | Description |
| ----------- | --------------------------- |
| INIT | Initializing a node |
| PREPARE | Preparation work is in progress after node initialization |
| READY | Node up and ready for service |
| READY(LOCK) | Receiving user/room from a node that is in SAFE PAUSE (beta availability) |
| PAUSE | Paused state |
| RESUME | Resume from PAUSE |
| SAFE PAUSE | SAFE PAUSE is in progress. When SAFE PAUSE is completed, it will be in PAUSE state. |
| INVALID | Temporary failure to check node information when node load is high (automatically restored) |
| DISABLE | Service is not available due to long-lasting INVALID state or failure (not automatically restored) |



## States related to Safe Pause

Some of the server state and node states are related to Safe Pause. If you proceed with Safe Pause for any node, the server and node will be transited to that state. A further explanation is as follows.


#### State of nodes undergoing Safe Pause

* SAFE PAUSE: If you safely pause (Safe Pause) on any node, it will securely transfer the processing information to other nodes. This series of processes is represented in the SAFE PAUSE state. Nodes in this state enter the PAUSE state after transferring all information.
* READY (LOCK): It is the state of the node to be transferred information such as user/room that was being processed by the node to be Safe Pause. 
A node in this state cannot perform an external command until the transfer is complete. In other words, it will be in a beta READY state until SafePause is complete.

#### State of the server running SafePause

* Some of the multiple nodes that make up the server may be in the SAFE PAUSE or READY (LOCK) state. This is implicitly expressed as SAFE PAUSE state.

For more information on Safe Pause, see [Safe Pause](console-09-safe-pause.md).
