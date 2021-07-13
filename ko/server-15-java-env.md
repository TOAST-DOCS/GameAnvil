## Game > GameAnvil > 서버 개발 가이드 > Java 개발 환경 설정



## 1. IntelliJ 개발 환경 체크 포인트

IntelliJ에서 Java 버전을 8에서 11로 (혹은 11에서 8로) 전환하거나 최초 세팅하는 과정에서 일부 설정이 누락되면 서버 빌드 및 실행이 의도한대로 동작하지 않을 수 있습니다. 이 문서는 이러한 시행착오를 줄이고 가능한 쉽고 편하게 개발 환경을 확인할 수 있도록 가이드 라인을 제공합니다.



### 1.1. JDK 설치

GameAnvil은 AdoptOpenJDK를 사용합니다. 지원하는 Java 버전은 8과 11 두 가지입니다. 사용자는 원하는 JDK를 직접 설치하여 개발 환경을 구성할 수 있습니다. 특별한 이유가 없다면 [AdoptOpenJDK](https://adoptopenjdk.net/) 사용을 제안합니다.



### 1.2. JDK for Importer

- File -> Settings 선택



![jdk11-settings.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-setting.png)



- Build, Execution, Deployment -> Build Tools -> Maven -> Importing 선택 후 "JDK for importer" 에서 JDK 11 설정

  ![jdk11-importer.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-importer.png)

  

### 1.3. Maven Runner

- Build, Execution, Deployment -> Build Tools -> Maven -> Runner 선택 후 JRE 11 설정

  ![jdk11-maven-runner.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-maven-runner.png)

  


### 1.4. Project SDK, Language level 설정

- File -> Project Structure 선택
  ![jdk11-project-structure.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-project-structure.png)


- Project Settings -> Project 선택 후 "Project SDK" 와 "Project language level"을 동일하게 설정 (8 혹은 11)
  ![jdk11-project-sdk.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-project-sdk.png)



### 1.5. 사용할 모듈의 Language level 설정

- Project Settings -> Modules 선택 후 사용자의 개발 프로젝트를 지정하여 Language level을 이전의 Project SDK의 그것과 동일하게 설정
  ![jdk11-lang-level.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-lang-level.png)



### 1.6. Application configuration JRE 11 로 설정

- 마지막으로 프로젝트의 구성을 편집합니다. 다음과 같이 사용자가 생성해둔 구성을 편집하거나 새롭게 작성할 수 있습니다.

  사용자의 Application을 선택한 후 JRE 버전을 확인한 후 앞서 설정한 JDK 버전과 같은 값으로 설정합니다.
  ![jdk11-jre.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-jre.png)

