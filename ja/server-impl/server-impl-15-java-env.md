<!-- pre-align:aligned sig=b74bc45b3f53 -->

<a id="game-gameanvil-server-development-guide-java-development-environment-setup"></a>
## Game > GameAnvil > サーバー開発ガイド > Java開発環境設定 { #game-gameanvil-server-development-guide-java-development-environment-setup }



<a id="intellij-development-environment-checkpoint"></a>
## IntelliJ開発環境チェックポイント { #intellij-development-environment-checkpoint }

IntelliJでJava 21バージョンを初期設定する過程で一部の設定が漏れると、サーバービルド及び実行が意図した通りに動作しない場合があります。このドキュメントは、このような試行錯誤を減らし、簡単かつ便利に開発環境を確認できるようにガイドラインを提供します。


<a id="install-jdk"></a>
### JDKインストール { #install-jdk }

GameAnvilはAdoptium Temurinを使用します。Javaバージョンは21をサポートしています。ユーザーは希望するJDKを直接インストールして開発環境を構築できます。特別な理由がなければ、[TemurinJDK](https://adoptium.net/temurin/releases)の使用を推奨します。



<a id="jdk-for-importer"></a>
### JDK for Importer { #jdk-for-importer }

1. **File** > **Settings...** をクリックします。

  ![jdk21-settings.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-settings.png)



2. **Build, Execution, Deployment** > **Build Tools** > **Gradle** を選択した後、**Gradle JVM**をJDK 21に設定します。

  ![jdk21-importer.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-gradle-jvm.png)



<a id="set-project-sdk-language-level"></a>
### Project SDK, Language level設定 { #set-project-sdk-language-level }

1. **File** > **Project Structure...** をクリックします。

  ![jdk21-project-structure.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-project-structure.png)


2. **Project Settings** -> **Project**を選択した後、**Project SDK**と**Project language level**を21に同様に設定します。

  ![jdk21-project-sdk.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-lang-level.png)



<a id="set-the-language-level-of-the-module-to-use"></a>
### 使用するモジュールのLanguage level設定 { #set-the-language-level-of-the-module-to-use }

1. **Project Settings** -> **Modules**を選択した後、ユーザーの開発プロジェクトを指定して、**Language level**を先ほどのProject SDKと同じに設定します。

  ![jdk21-lang-level.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/15-java-env/jdk21-lang-module-default.png)
