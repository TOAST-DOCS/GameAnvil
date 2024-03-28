## Game > GameAnvil > Console User Guide > Safe Pause

## What is Safe Pause?

Safe Pause is a feature that allows you to pause any game node without stopping the service. SafePause's departure node transfers all users and room information in real time to the destination node that is performing the same service. Therefore, SafePause is not available unless more than two game node exist in the same service.

In the SafePause process, the sender and receiver change to **SAFE PAUSE** and **READY(LOCK)** states, respectively, as shown in the figure below. A node in the **READY(LOCK)** state cannot change its state arbitrarily until the corresponding SafePause transaction is completed. In other words, it means that no operation can be performed on the console.

![Figure](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/state.png)

When SafePause completes, the **SafePause**state changes to **Pause** state. Naturally, there are no users or rooms on that node, so you can shut it down safely. If you need a patch or check, you can proceed at this time. **Ready(Lock)** state also turn to **Ready** state upon completion of SafePause and can be operated on the console.

## How to use Safe Pause

Select the Operations menu to view the Safe Pause main screen. It shows a list of nodes associated with Safe Pause currently in progress. Click on **Select Node** to open a pop-up to select the node to perform SafePause as shown below.

![Figure](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/node-selection.png)

You can select one or more **departure nodes** and **destination nodes** in SafePause by clicking on more than one check boxes, respectively. At this time, it is advantageous to select a node with a small count of users and rooms being processed as **Destination Node**. When selection is done, press **Confirm** to **register** SafePause. Please take note that you are still in the registration stage, not in the performance stage yet.

The nodes associated with Safe Pause that were previously registered are exposed to the Safe Pause main screen as follows. If there are any nodes that are incorrectly registered, you can delete them by clicking **-** on the right side of the list.

![Figure](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/start.png)

When you are all ready, you can start SafePause by clicking **Run Check**. Once SafePause starts, you can check the state of progress in real time from the **Server** menu as follows.

![Figure](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/safe-pausing.png)

Through SafePause process, the user and room of the **departure node** are sent to the **destination node** and all the **departure nodes** automatically enter the PAUSE state after all processes are completed. The PAUSE nodes can now be safely shut down.   

![Figure](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/safe-pause-done.png)
