## Game > GameAnvil > Release Note > Console


## January 9, 2024
### Fix
* Improved stability

### Change
* Reorganized the server screen
  * Separated the general server from the autoscale group menu
  * Added the detailed search and control of created servers and autoscale groups
  * Added the server and autoscale group details page
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

## September 26, 2023
### New
* Added a software license agreement screen

### Change
* Added a progress display when activating a GameAnvil project


## August 29, 2023
### Change
* Improved usability to allow additional servers to be created and controlled during server creation and control

### Fix
* Improved stability


## July 25, 2023
### Fix
* Fixed an issue where deployment files in Auto-Scale groups are not modified
* Modified to go to the NHN Cloud login page when login expires
* Improved stability


## July 11, 2023
### Fix
* Modified to allow bulk changes to files deployed on multiple servers
* Improved stability


## June 27, 2023
### Changes
* Improved the monitoring dashboard screen
    * Added a graph of how many CPU cores are currently in use
    * Added per-server, per-node health monitoring
    * Added CPU, Memory, Disk, and Network usage graphs for a server
    * Added the auto scale group health monitoring
    * Added user distribution history and graphs

### Fix
* Improved stability


## May 30, 2023
### New
* Integration with CloudTrail to track user actions

### Change
* Modified to expose nodes in the Auto-Scale group to the node monitoring page
* Modified to ensure additional servers that incompatible with those already running do not start by checking the major version of the deployment file

### Fix
* Fixed an issue where the gateway in an Auto-Scale group is not connected
* Improved stability


### December 27, 2022

Starting with GameAnvil 1.3.0, it is integrated with the all-new console. The new console is not just a simple update to the previous version. Almost everything has been reimplemented to integrate GameAnvil servers with NHN Cloud infrastructure and provide a new UX that makes running your game service easier and more comfortable. For this reason, the console is skipping 1.1 and 1.2 and going straight to 1.3 to keep up with GameAnvil.

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


#### Fix

* None

#### Change

* None

---

### July 13, 2021

#### Change

* Updated the guide documentation for each input value in the instance settings
* Modified to terminate instances normally via the management node instead of forcing a kill when stopped
* Changed default port when setting up an instance (new instances only)

#### Fix

* Fixed an intermittent error when the order of authentication filters was not explicit.

---

### June 15, 2021

#### New

* Applied Japanese translation

#### Change

* Modified how often monitoring runs and how many errors are tolerated to reduce false positives in monitoring

---

### May 25, 2021

#### New

* Created a separate page for management as services could only be created but not edited or deleted

#### Change

* Changed the instance settings so that they are now stored and managed as an instance rather than as a node
* Changed VM Option input message to allow up to 10,240 bytes

#### Fix

* Fixed an issue in which the user guide of each node would not move to the proper guide page when creating instance settings

---

### April 27, 2021

#### New

* Applied English translation

#### Change

* Changed VM Option input message to allow up to 512 bytes
* Applied Toast UI Chart Vue version 4.2.1

#### Fix

* Fixed an issue where the machine would be forcibly set to Java 11 if Retry button was clicked after the occurrence of a machine setting error
* Fixed an issue where all filtered machines would be selected when a machine was selected from Register Instance

---

### March 23, 2021

#### Change

* Changed the system so that it shows the machine registration page if the instance registration screen is accessed while there is no registered machine.
* Changed the system so that the user would be redirected to the machine list after a machine file is uploaded.
* Changed the system so that the monitoring dashboard concurrent user influx graph data would display the hour and minute of the data
* Changed the text of the Check for Duplicates button to Check for Port Duplicates in the instance registration/edit screen

#### Fix

* Changed the system so that a popup window would appear if Resume/Pause is clicked without selecting node monitoring

---

### February 23, 2021

#### New

##### Released GameAnvil Console

* You can check the changes in concurrent users and monitor user distribution
* You can manage the GameAnvil engine server.
* You can check the events occurred from instances and nodes.
