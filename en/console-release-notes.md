## Game > GameAnvil > Console > Release Note

### 1.0.3 (2021.05.25)

#### New

* Created a separate page for management as services could only be created but not edited or deleted

#### Change

* Changed the instance settings so that they are now stored and managed as an instance rather than as a node
* Changed the size of the VM Option input message up to 10,240 bytes

#### Fix

* Fixed an issue in which the user guide of each node would not move to the proper guide page when creating instance settings

### 1.0.2 (2021.04.27)

#### New

* Applied the English translation

#### Change

* Changed VM Option input message to allow up to 512 bytes
* Applied Toast UI Chart Vue version 4.2.1

#### Fix

* Fixed an issue where the machine would be forcibly set to Java 11 if Retry button was clicked after the occurrence of a machine setting error
* Fixed an issue where all filtered machines would be selected when a machine was selected from Register Instance

### 1.0.1 (2021.03.23)

#### Change

* Changed the system so that it shows the machine registration page if the instance registration screen is accessed while there is no registered machine.
* Changed the system so that the user would be redirected to the machine list after a machine file is uploaded.
* Changed the system so that the monitoring dashboard concurrent user influx graph data would display the hour and minute of the data
* Changed the text of the Check for Duplicates button to Check for Port Duplicates in the instance registration/edit screen

#### Fix

* Changed the system so that a popup window would appear if Resume/Pause is clicked without selecting node monitoring

### 1.0.0 (2021.02.23)

#### New

##### Release GameAnvil Console

* Can check the changes in concurrent users and monitor user distribution.
* Can manage the GameAnvil engine server.
* Can check the events occurred from instances and nodes.
