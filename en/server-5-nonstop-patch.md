## Game > GameAnvil > Server Development Guide > How to Apply Uninterrupted Patch

This document explains how to apply uninterrupted patches. In addition, it covers potential issues that may arise while applying uninterrupted patches.

### Note

- Apply uninterrupted maintenance in alpha before using it live.
- Read and understand this guide document before using.



## 1. Situations in which NonStopPatch should not be used

- If the GameAnvil version is different in the server file to be patched and the deployed server file
- If a packet is edited or added
- If the Data Class passed back and forth among Transfer is changed



## 2. Tasks that need to be done to use NonStopPatch

### 2-1. If the Match feature is being used,

- a logic to check the version information of existing server and new server must be included.

### 2-2. TransferData

- If the transferData of existing server and new server is different, the transferData deleted or added by the onTranferIn function of the new GameUser/GameRoom must be processed.
- If data is passed to proto object, the class must be registered to kryo..



## 3. Things to know when using NonStopPatch

- Provide NonStopPatch Report information
  - Rest API
    - http://127.0.0.1:25150/managementEndPoint/non-stop-patch/report-page
    - Provide the progress and result of NonStopPatch
  - File
    - log/nonStopPatch/report-2020-08-10 205510.log
    - Save Report to a file in Json format when ending NonStopPatch
- The forced NonStopPatch end feature
  - If NonStopPatch is in progress, the status of other Nodes cannot be changed.
  - It may not end because a problem occurred while processing NonStopPatch.
  - Identifies the problem through NonStopPatch Report and log and determines whether to forcibly end NonStopPatch..
  -
    - If the NonStopPatch forcible end button is clicked, NonStopPatch is forcibly ended.
  - Processes paused nodes.

## 4. Starting NonStopPatch

- NonStopPatch is processed as Admin.

### 4-1. Selecting the Node That Process NonStopPatch

- Select starting point
- ![NonStopPatch-2.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-2.png)
- Select destination
- ![NonStopPatch-3.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-3.png)

### 4-2. Opening NonStopPatch Report Page

- http://10.162.3.241:25150/non-stop-patch/report-page?refreshTime=1000

### 4-3. Click the NonStopPatch Button

- Process NonStopPatch by clicking the NonStopPatch button
- ![NonStopPatch-4.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-4.png)
- ![NonStopPatch-5.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-5.png)
- The starting point Node changes to the NonStopPatch Src status
  - When NonStopPatch ends, the status becomes Pause
- Destination Node changes to the NonStopPatch Dst status
  - When NonStopPatch ends, the status becomes Ready

### 4-4. Checking NonStopPatch Result

- Check NonStopPatch Report
- ![NonStopPatch-6.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-6.png)
- Check the current status of GameNode
- ![NonStopPatch-7.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-7.png)

### 4-5. Ending Server

- End the paused servers
- ![NonStopPatch-8.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-8.png)
- Check ended server
- ![NonStopPatch-9.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-9.png)

### 4-6. Server Patch

- Delete deployed file and deploy the server file to be patched
  - Does not need to be deleted if the server file is not manually specified
- ![NonStopPatch-10.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-10.png)

### 4-7. Restarting Server

- Restart the server
- ![NonStopPatch-11.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-11.png)

### 4-8. The Opposite Node Progresses NonStopPatch

- Process with the opposite Node like above

### 4-9. Ending Server

- Process with the opposite Node like above

### 4-10. Server Patch

- Process with the opposite Node like above

### 4-11. Restarting Server

- Process with the opposite Node like above

### 4-12. Checking NonStopPatch

- Check if there is any issue after server patch

## 5. The Function Call Order During NonStopPatch

### 5-1. BaseGameRoom

- The Callback function that is called when creating
  - Fill the content of the Callback function appropriately.
  - Create GameRoom
    - onInit -> onCreate
  - Create GameRoom during NonStopPatch
    - onInit -> onTransferIn
- The Callback function that is called after NonStopPatch ended
  - The Callback function is called from GameRoom when the State of GameNode after NonStopPatch ended.
  - Starting point Node
    - onPause
    - As the status of the starting point Node becomes pause after NonStopPatch ended,
  - Destination Node
    - X

### 5-2. BaseUserRoom

- The Callback function that is called when creating
  - The content of the Callback function must be filled according to the situation.
  - Create GameUser
    - onLogin
  - NCreate GameUser when NonStopPatch
    - onTransferIn
- The Callback function that is called after NonStopPatch ended
  - GameUser calls the Callback function when the State of GameNode changes after NonStopPatch ended.
  - Starting point Node
    - onPause
    - As the status of the starting point Node becomes pause after NonStopPatch ended,
  - Destination Node
    - X

### 5-3. BaseGameNode

- The Callback function that is called when NonStopPatch
- When it is starting point Node

```
// Called when NonStopPatch is started
@Override
public boolean onNonStopPatchSrcStart() throws SuspendExecution {
    return true;
}
// Called when NonStopPatch ended
@Override
public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}
// The NonStopPatch end check function
@Override
public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}
```

- If it is destination Node

```
// Called when NonStopPatch is started
@Override
public boolean onNonStopPatchDstStart() throws SuspendExecution {
    return true;
}
// Called when NonStopPatch ended
@Override
public boolean onNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
// The NonStopPatch end check function
@Override
public boolean canNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
```
