## Game > GameAnvil > 서버 개발 가이드 > Java 개발 환경 설정



## IntelliJ 개발 환경 체크 포인트

IntelliJ에서 Java 21 버전을 최초 설정하는 과정에서 일부 설정이 누락되면 서버 빌드 및 실행이 의도한 대로 동작하지 않을 수 있습니다. 이 문서는 이러한 시행착오를 줄이고 쉽고 편하게 개발 환경을 확인할 수 있도록 가이드라인을 제공합니다.


### JDK 설치

GameAnvil은 AdoptOpenJDK를 사용합니다. Java 버전은 21을 지원합니다. 사용자는 원하는 JDK를 직접 설치하여 개발 환경을 구성할 수 있습니다. 특별한 이유가 없다면 [AdoptOpenJDK](https://adoptopenjdk.net/)를 사용할 것을 권장합니다.



### JDK for Importer

1. **File** > **Settings...** 를 클릭합니다.

  ![jdk21-settings.png](https://static.toastoven.net/prod_gameanvil/images/2024/jdk21-settings.png)



2. **Build, Execution, Deployment** > **Build Tools** > **Gradle** 을 선택한 뒤 **Gradle JVM**를 JDK 21로 설정합니다.

  ![jdk21-importer.png](https://static.toastoven.net/prod_gameanvil/images/2024/jdk21-gradle-jvm.png)



### Project SDK, Language level 설정

1. **File** > **Project Structure...** 를 클릭합니다.

  ![jdk21-project-structure.png](https://static.toastoven.net/prod_gameanvil/images/2024/jdk21-project-structure.png)


2. **Project Settings** -> **Project**를 선택한 뒤 **Project SDK** 와 **Project language level**을 21로 동일하게 설정합니다.

  ![jdk21-project-sdk.png](https://static.toastoven.net/prod_gameanvil/images/2024/jdk21-lang-level.png)



### 사용할 모듈의 Language level 설정

1. **Project Settings** -> **Modules**를 선택한 뒤 사용자의 개발 프로젝트를 지정하여 **Language level**을 이전의 Project SDK의 그것과 동일하게 설정합니다.

  ![jdk21-lang-level.png](https://static.toastoven.net/prod_gameanvil/images/2024/jdk21-lang-module-default.png)



