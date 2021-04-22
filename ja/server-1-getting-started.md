## Game > GameAnvil > サーバー開発ガイド > 始める

GameAnvilを利用してゲームサーバーを開発する方法を詳しく説明します。

## システム要求事項

| 開発言語 | 開発IDE      | サポートOS               | プロジェクト管理 | 使用可能なネットワークプロトコル | セキュリティ |
| ---------- | ------------- | --------------------- | ------------- | ----------------------------- | ---- |
| Java 8, 11 | IntelliJ IDEA | Linux, Windows, MacOS | Maven         | TCP/IP、WebSocket、HTTP/HTTPS | SSL  |



## 1分毎にチャットサーバーを作成する

GameAnvilは基本的な骨格の作業を迅速に完了するために、独自のテンプレートを提供します。テンプレートを使用すると、数回クリックするだけで基本的なゲームサーバーが完成します。こうして作成されたサーバーは、簡単なチャット機能が含まれています。詳しくは、下記のサーバーテンプレートの使用方法を参照してください。

- [テンプレートのダウンロード](http://static.toastoven.net/prod_gameanvil/files/GameAnvil-Template.zip)

- 実行する

- IntelliJを開き、**File > Settings**を選択します。

   ![GameAnvilTemplate-1](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-1.png)

- **Import Settings**ダイアログボックスで、先にダウンロードした**GameAnvil-Template.zip**ファイルを選択してインポートします。
  ![GameAnvilTemplate-2](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-2.png)

- 取得が完了した後、IntelliJを再起動します。

- 再起動すると、次のようなダイアログボックスが表示されます。**Create New Project**を選択します。    ![GameAnvilTemplate-3](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-3.png)

- これで登録したテンプレートを選択できます。 **Next**ボタンを押すと適用されます。    ![GameAnvilTemplate-4](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-4.png)

- 適用中に次のように**Enable Auto-Import**を確認するウィンドウが表示されたら**Enable**を選択します。    ![GameAnvilTemplate-5](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-5.png)

- 全て完了するとサーバーテンプレートプロジェクトのソースコードが表示されます。**Run**を実行すると、次のようにチャットサーバーが起動します。    ![GameAnvilTemplate-6](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-6.png)



## サーバー起動

テンプレートを利用した基本的なサーバーの構成が完了したら実行できます。サーバーが無事に起動したら、次のように各ノードの**onReady**ログが出力されます。次に準備したサーバーにクライアントを利用して接続します。

```
[2020-03-13 17:36:16,925] [INFO ] [MANAGEMENT@1005@ThinkServer@1] [c.n.t.m.ManagementNode] onReady
[2020-03-13 17:36:20,008] [INFO ] [LOCATION@30122@ThinkServer@2] [c.n.t.l.LocationNode] onReady
[2020-03-13 17:36:18,316] [INFO ] [SPACE@1@ThinkServer@0] [c.n.t.s.i.SpaceNode] onReady
[2020-03-13 17:36:19,008] [INFO ] [SESSION@1001@ThinkServer@2] [c.n.t.s.i.SessionNode] onReady
```



## リファレンス

GameAnvilはテンプレートだけでなく、リファレンスプロジェクトを提供します。GameAnvilユーザーが参考にできるようにテンプレートで構成した基本骨格にさまざまな機能を実装しておきました。これらのサーバーとクライアントのリファレンスプロジェクトは別のメニューで確認できます。またGameAnvil APIとUMLダイアグラムをJavaDoc文書で提供しているので参考にしてください。

[GameAnvil JavaDoc API Reference Site](http://10.162.4.61:9090/gameanvil)(JavaDocサイトをNHN Cloud内に含める場合、該当ドメインに変更予定)
