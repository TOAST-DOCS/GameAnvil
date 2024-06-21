## Game > GameAnvil > Server Development Guide > Java Development Environment Setup



## IntelliJ Development Environment Checkpoint

When switching Java versions from 8 to 11 (or 11 to 8) in IntelliJ, or during initial setup, if you miss some settings, your server build and run might not work as intended. This document provides guidelines to reduce this trial and error and make checking out your development environment as easy and comfortable as possible.


### Install JDK

GameAnvil uses AdoptOpenJDK. There are two Java versions supported: 8 and 11. Users can configure their development environment by installing the JDK of their choice. We suggest using [AdoptOpenJDK](https://adoptopenjdk.net/) if for no other reason.



### JDK for Importer

- Go to File -> Settings



![jdk11-settings.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-setting.png)



- Select Build, Execution, Deployment -> Build Tools -> Maven -> Importing and set JDK 11 under "JDK for importer"

  ![jdk11-importer.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-importer.png)

  

### Maven Runner

- Select Build, Execution, Deployment -> Build Tools -> Maven -> Select Runner and set JRE 11

  ![jdk11-maven-runner.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-maven-runner.png)

  


### Set Project SDK, Language level

- Go to File -> Project Structure
![jdk11-project-structure.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-project-structure.png)


- Go to Project Settings -> Project and set "Project SDK" and "Project language level" to the same (8 or 11)
![jdk11-project-sdk.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-project-sdk.png)



### Set the Language level of the module to Use

- Go to Project Settings -> Modules and specify your development project to set the language level to the same as that of the previous Project SDK.
![jdk11-lang-level.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-lang-level.png)



### Set the application configuration to JRE 11

- Finally, edit the project's configuration. You can edit or create new configurations that you have created, as follows.

  After selecting your application, check the JRE version and set it to the same value as the JDK version you set earlier.
![jdk11-jre.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-jre.png)

