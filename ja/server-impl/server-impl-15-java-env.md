## Game > GameAnvil > サーバー開発ガイド > Java開発環境の設定



## IntelliJ開発環境のチェックポイント

IntelliJでJavaバージョンを8から11に、または11から8に切り替えた場合や、初回設定プロセスで一部の設定が抜けていた場合、サーバービルドおよび実行が意図したとおりに動作しないことがあります。この文書では、このような試行錯誤を減らし、簡単かつ便利に開発環境を確認できるようにガイドラインを提供します。


### JDKインストール

GameAnvilはAdoptOpenJDKを使用します。Javaバージョンは8と11をサポートしています。ユーザーは希望するJDKを直接インストールして開発環境を構成できます。特別な理由がなければ、[AdoptOpenJDK](https://adoptopenjdk.net/)を使用することを推奨します。



### JDK for Importer

1. **File**>**Settings...**をクリックします。



  ![jdk11-settings.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-setting.png)



2. **Build、Execution、Deployment**>**Build Tools**>**Maven**>**Importing**を選択した後**JDK for importer**をJDK 11に設定します。

  ![jdk11-importer.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-importer.png)

  

### Maven Runner

1. **Build, Execution, Deployment** > **Build Tools** > **Maven** > **Runner**を選択した後、JREを11に設定します。

  ![jdk11-maven-runner.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-maven-runner.png)

  


### Project SDK、Language levelの設定

1. **File**>**Project Structure...**をクリックします。

  ![jdk11-project-structure.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-project-structure.png)


2. **Project Settings**→**Project**を選択した後**Project SDK**と**Project language level**を8または11に同じように設定します。

  ![jdk11-project-sdk.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-project-sdk.png)



### 使用するモジュールのLanguage level設定

1. **Project Settings**→**Modules**を選択した後、ユーザーの開発プロジェクトを指定して**Language level**を以前のProject SDKのそれと同じように設定します。

  ![jdk11-lang-level.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-lang-level.png)



### Application configuration JRE 11に設定

- 最後にプロジェクトの構成を編集します。次のようにユーザーが作成しておいた構成を編集または、新たに作成できます。

 ユーザーのApplicationを選択してJREバージョンを確認した後、事前に設定したJDKバージョンと同じ値に設定します。

  ![jdk11-jre.png](https://static.toastoven.net/prod_gameanvil/images/jdk11-jre.png)
