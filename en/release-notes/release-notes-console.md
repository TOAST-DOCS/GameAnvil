<!-- pre-align:aligned sig=9a3c9a5812cb -->

<a id="game-gameanvil-release-notes-console"></a>
## Game > GameAnvil > Release Notes > Console { #game-gameanvil-release-notes-console }

<a id="2025-06-30"></a>
## 2025. 06. 30. { #2025-06-30 }
<a id="change"></a>
### Change { #change }
* Support GameAnvil 2.1 or later

<a id="fix"></a>
### Fix { #fix }
* Improved stability

<a id="2024-12-10"></a>
## 2024. 12. 10. { #2024-12-10 }
<a id="2024-12-10-change"></a>
### Change { #2024-12-10-change }
* GameAnvil 1.x support discontinued
* GameAnvil 2.0 support
* Updated SafePause screen
  * Added feature to retain selected nodes even after leaving the page.
  * Updated SafePause target node list screen.
  * Updated SafePause progress confirmation screen with added detailed node information.
  * Updated SafePause history screen with added detailed node information.
  * Match node SafePause support.
    * Multiple SafePauses can be performed simultaneously.
* Dashboard > Server Status
  * Added the graph for the number of rooms
* Multi-node support in GameAnvil 2.0
  * Changed the `channeldIDs` field in Config to a double array <br/>
    An array containing channels represents a single thread. If the array contains two or more channels, two or more game nodes can run on a single thread <br/>
```
{
  "game": [
    {
      "serviceId": 1,
      "serviceName": "RPSGame",
      "nodeCnt": 3,
      "channelIDs": [
        ["ch1"], 
        ["ch2"],
        ["ch3", "ch4"]
      ]
    }
  ]
}
```

<a id="2024-12-10-fix"></a>
### Fix { #2024-12-10-fix }
* Improved stability

<a id="april-9-2024"></a>
## April 9, 2024 { #april-9-2024 }
<a id="april-9-2024-change"></a>
### Change { #april-9-2024-change }
* Improved SafePause

<a id="january-9-2024"></a>
## January 9, 2024 { #january-9-2024 }
<a id="january-9-2024-fix"></a>
### Fix { #january-9-2024-fix }
* Improved stability

<a id="january-9-2024-change"></a>
### Change { #january-9-2024-change }
* Server screen reorganized
  * Separated general server from autoscale group menu
  * Added detailed search and control of created servers and autoscale groups
  * Added server and autoscale group details page
  * Added the feature to copy servers and autoscale groups
  * Added the feature to bulk modify general servers
  * Added the feature to bulk modify autoscale group deployment files
  * Added the feature to recover from failed autoscale group instances
  * Added autoscale group scale event history view page
  * Added the GameAnvil Config feature
  * Modified the deployment files menu
    * Deleted default deployment files
    * Modified the search feature
  * Modified the Safe Pause screen 
  * Reorganized the node menu
    * Added the feature to view and refine information
  * Modified server, autoscale group creation screen
    * GameAnvil Config can be created by writing it in JSON format in advance


<a id="september-26-2023"></a>
## September 26, 2023 { #september-26-2023 }
<a id="new"></a>
### New { #new }
* Added a software license agreement screen

<a id="september-26-2023-change"></a>
### Change { #september-26-2023-change }
* Added a progress display when activating a GameAnvil project


<a id="august-29-2023"></a>
## August 29, 2023 { #august-29-2023 }
<a id="august-29-2023-change"></a>
### Change { #august-29-2023-change }
* Improved usability to allow additional servers to be created and controlled during server creation and control

<a id="august-29-2023-fix"></a>
### Fix { #august-29-2023-fix }
* Improved stability


<a id="july-25-2023"></a>
## July 25, 2023 { #july-25-2023 }
<a id="july-25-2023-fix"></a>
### Fix { #july-25-2023-fix }
* Fixed an issue where deployment files in Auto-Scale groups are not modified
* Modified to go to the NHN Cloud login page when login expires
* Improved stability


<a id="july-11-2023"></a>
## July 11, 2023 { #july-11-2023 }
<a id="july-11-2023-fix"></a>
### Fix { #july-11-2023-fix }
* Modified to allow bulk changes to files deployed on multiple servers
* Improved stability


<a id="june-27-2023"></a>
## June 27, 2023 { #june-27-2023 }
<a id="changes"></a>
### Changes { #changes }
* Improved the monitoring dashboard screen
    * Added a graph of how many CPU cores are currently in use
    * Added per-server, per-node health monitoring
    * Added CPU, Memory, Disk, and Network usage graphs for a server
    * Added the auto scale group health monitoring
    * Added user distribution history and graphs

<a id="june-27-2023-fix"></a>
### Fix { #june-27-2023-fix }
* Improved stability


<a id="may-30-2023"></a>
## May 30, 2023 { #may-30-2023 }
<a id="may-30-2023-new"></a>
### New { #may-30-2023-new }
* Integration with CloudTrail to track user actions

<a id="may-30-2023-change"></a>
### Change { #may-30-2023-change }
* Modified to expose nodes in the Auto-Scale group to the node monitoring page
* Modified to ensure additional servers that incompatible with those already running do not start by checking the major version of the deployment file

<a id="may-30-2023-fix"></a>
### Fix { #may-30-2023-fix }
* Fixed an issue where the gateway in an Auto-Scale group is not connected
* Improved stability


<a id="december-27-2022"></a>
### December 27, 2022 { #december-27-2022 }

Starting with GameAnvil 1.3.0, it is integrated with the all-new console. The new console is not just a simple update to the previous version. Almost everything has been reimplemented to integrate GameAnvil servers with NHN Cloud infrastructure and provide a new UX that makes running your game service easier and more comfortable. For this reason, the console is skipping 1.1 and 1.2 and going straight to 1.3 to keep up with GameAnvil.

<a id="december-27-2022-new"></a>
#### New

* You can create and manage your own ````infrastructure````.
    * Console management of compute instances (VMs) and related infrastructure, including load balancers (L4s)
* Support for more stable, advanced ````Safe Pause````.
    * Included many hotfixes performed from live service technical support
* Support for ````Auto-Scale```` to flexibly respond to server load during service.
    * Set the number of active users and rooms, as well as the resource usage of the infrastructure as scaling conditions
* Provides a variety of ````monitoring```` capabilities.
    * Provides real-time aggregate metrics for server and node status, as well as users, rooms, sessions, etc.
* Quickly and easily ````deploy```` server binaries.
    * Full support for deployment history
* You only need to configure and manage the ````Gateway, Game, Match, and Support```` nodes.
    * Manage location nodes and management nodes on the platform
* ```Easy, GUI-based server and node configuration```.
* The overall ```screen design and UX has been advanced```.
  * Provides an at-a-glance UI for server health
  * Drag & Drop UI for easy and comfortable node configuration


<a id="december-27-2022-fix"></a>
#### Fix

* None

<a id="december-27-2022-change"></a>
#### Change

* None

---

<a id="july-13-2021"></a>
### July 13, 2021 { #july-13-2021 }

<a id="july-13-2021-change"></a>
#### Change

* Updated the guide documentation for each input value in the instance settings
* Modified to terminate instances normally via the management node instead of forcing a kill when stopped
* Changed default port when setting up an instance (new instances only)

<a id="july-13-2021-fix"></a>
#### Fix

* Fixed an intermittent error when the order of authentication filters was not explicit.

---

<a id="june-15-2021"></a>
### June 15, 2021 { #june-15-2021 }

<a id="june-15-2021-new"></a>
#### New

* Applied Japanese translation

<a id="june-15-2021-change"></a>
#### Change

* Modified how often monitoring runs and how many errors are tolerated to reduce false positives in monitoring

---

<a id="may-25-2021"></a>
### May 25, 2021 { #may-25-2021 }

<a id="may-25-2021-new"></a>
#### New

* Created a separate page for management as services could only be created but not edited or deleted

<a id="may-25-2021-change"></a>
#### Change

* Changed the instance settings so that they are now stored and managed as an instance rather than as a node
* Changed VM Option input message to allow up to 10,240 bytes

<a id="may-25-2021-fix"></a>
#### Fix

* Fixed an issue in which the user guide of each node would not move to the proper guide page when creating instance settings

---

<a id="april-27-2021"></a>
### April 27, 2021 { #april-27-2021 }

<a id="april-27-2021-new"></a>
#### New

* Applied English translation

<a id="april-27-2021-change"></a>
#### Change

* Changed VM Option input message to allow up to 512 bytes
* Applied Toast UI Chart Vue version 4.2.1

<a id="april-27-2021-fix"></a>
#### Fix

* Fixed an issue where the machine would be forcibly set to Java 11 if Retry button was clicked after the occurrence of a machine setting error
* Fixed an issue where all filtered machines would be selected when a machine was selected from Register Instance

---

<a id="march-23-2021"></a>
### March 23, 2021 { #march-23-2021 }

<a id="march-23-2021-change"></a>
#### Change

* Changed the system so that it shows the machine registration page if the instance registration screen is accessed while there is no registered machine.
* Changed the system so that the user would be redirected to the machine list after a machine file is uploaded.
* Changed the system so that the monitoring dashboard concurrent user influx graph data would display the hour and minute of the data
* Changed the text of the Check for Duplicates button to Check for Port Duplicates in the instance registration/edit screen

<a id="march-23-2021-fix"></a>
#### Fix

* Changed the system so that a popup window would appear if Resume/Pause is clicked without selecting node monitoring

---

<a id="february-23-2021"></a>
### February 23, 2021 { #february-23-2021 }

<a id="february-23-2021-new"></a>
#### New

##### Released GameAnvil Console

* Check the changes in concurrent users and monitor user distribution
* Manage the GameAnvil engine server.
* Check the events occurred from instances and nodes.
