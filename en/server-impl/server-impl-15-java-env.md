## Game > GameAnvil > Server Development Guide > Java Development Environment Setup



## IntelliJ Development Environment Checkpoint

When switching Java versions from 8 to 11 (or 11 to 8) in IntelliJ, or during initial setup, if you miss some settings, your server build and run might not work as intended. This document provides guidelines to reduce this trial and error and make checking out your development environment as easy and comfortable as possible.


### Install JDK

GameAnvil uses Adoptium Temurin. There are two Java versions supported: 8 and 11. Users can configure their development environment by installing the JDK of their choice. We suggest using [TemurinJDK](https://adoptium.net/temurin/releases) if for no other reason.



### JDK for Importer

- Click **File** -> **Settings...**

  ![jdk21-settings.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-settings.png)



- Select **Build, Execution, Deployment** > **Build Tools** > **Gradle** and set JDK 11 under **Gradle JVM**.

  ![jdk21-importer.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-gradle-jvm.png)


  
### Set Project SDK, Language level

1. Click **File** > **Project Structure...**.

  ![jdk21-project-structure.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-project-structure.png)


2. Select **Project Settings** -> **Project** and set **Project SDK** and **Project language level** to the same (21).

  ![jdk21-project-sdk.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-lang-level.png)



### Set the Language level of the module to Use

1. Select **Project Settings** -> **Modules**, then specify your development project and set the **Language level** to the same as that of the previous Project SDK.

  ![jdk21-lang-level.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-lang-module-default.png)


