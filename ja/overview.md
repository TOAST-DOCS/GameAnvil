<!-- pre-align:aligned sig=6bcec4ef6ca3 -->

<a id="game-gameanvil-overview"></a>
## Game > GameAnvil > 概要 { #game-gameanvil-overview }

GameAnvilはJavaベースの高性能リアルタイムゲームサーバーエンジンです。ゲームサーバーの開発時間を短縮し、性能と安定性を高めることを目指しています。GameAnvilを利用して豊富なJavaエコシステムの恩恵を
受けながら、簡単かつ迅速にリアルタイムゲームサーバーを開発してみてください。初心者開発者でも簡単に習得してすぐにリアルタイムゲームサーバーを開発でき、UnityあるいはTypeScriptなどで開発中のクライアントを即座に連携できます。
またGameAnvilシステムは、ゲームサーバーの開発だけでなく機能/性能テストはもちろん、クラウド上での運用とモニタリングまで担うことのできる全般的なツールをあわせて提供します。

<a id="official-reference-game"></a>
## 公式リファレンスゲーム { #official-reference-game }

GameAnvilを利用して開発しサービス中の代表的なゲームは次のとおりです。

![gameanvil-references.png](https://static.toastoven.net/prod_gameanvil/images/gameanvil-references.png)

<a id="character"></a>
## 特徴 { #character }

GameAnvilの究極の目標は、経験の浅い開発者でも簡単にリアルタイムコンテンツを開発しサービスできるよう支援することです。このような側面からコードの生産性と使用の利便性を高め、可能な限りユーザーが感じる技術的
参入障壁を下げようとしています。またクラウド商品の長所を最大化し、より簡単にサービスを運用できるよう支援します。

<a id="character-increase-code-productivity"></a>
#### コード生産性の増大

* 簡潔かつ容易に順次コードを作成するだけで、エンジンが自動的に仮想スレッド(Virtual Thread)ベースで高性能な非同期処理を行います。

<a id="character-reliable-performance"></a>
#### 安定した性能

* 高性能ライブラリをベースに、最適な非同期処理と安定した性能を提供します。

<a id="character-easy-to-use"></a>
#### 便利な使用法

* クラウド上でサーバー管理とモニタリングはもちろん、テストまでサポートします。

<a id="character-flexible-server-configuration"></a>
#### 柔軟なサーバー構成

* 小規模ゲームから大規模ゲームまで、サービス規模と特性に合わせて最適な構成が可能です。

<a id="recommended-games"></a>
## 推奨対象ゲーム { #recommended-games }

現在GameAnvilは、以下のような種類のゲームにほぼ完璧に対応します。

* カジュアルゲーム
* ターン制ゲーム
* ボードゲーム
* リアルタイムコンテンツあるいはDBストレージなどが必要なインディーゲーム
* その他、リアルタイムコンテンツを簡単に作成してサービスしたい様々なゲーム

<a id="integrate-an-easy-to-use-unit-development-environment"></a>
## 容易なUnity開発環境の統合 { #integrate-an-easy-to-use-unit-development-environment }

GameAnvilサーバーと簡単に連携可能な**GameAnvilコネクタ**Unityパッケージを提供します。当該パッケージを通じて既存のUnity開発環境をすぐにGameAnvilサーバーと連携できます。

<a id="integrate-an-easy-to-use-unit-development-environment-fast-connection-and-authentication"></a>
#### 高速な接続及び認証

* サーバーへの接続及び認証を専用APIを通じて簡単かつ迅速に処理できます。

<a id="integrate-an-easy-to-use-unit-development-environment-rich-multiplayer-api"></a>
#### 豊富なマルチプレイAPI

* ルーム生成、ルーム入室、マッチメイキングなど、マルチプレイゲームを実装するにあたり必要な全ての機能がAPIで提供されます。

<a id="integrate-an-easy-to-use-unit-development-environment-support-synchronization-component"></a>
#### 同期コンポーネントサポート

* コンポーネントを登録するだけでも、別途サーバー実装なしでユーザー間の同期が可能です。
* 座標同期、強制同期、アニメーター同期の他に、ユーザーが定義した値の同期をサポートします。

<a id="recommended-developers"></a>
## 推奨対象開発者 { #recommended-developers }

**経験の浅い開発者**でも簡単に開発し安定してサービスできるよう、利便性と柔軟性を備えています。

| 簡単な開発                                                                     | 安定性                                                                                 | 利便性                                                                 | 柔軟性                                                                 |
|--------------------------------------------------------------------------|----------------------------------------------------------------------------------------|--------------------------------------------------------------------|------------------------------------------------------------------| 
| Java<br/>Virtual Thread<br/>Continuation<br/>Single Thread<br/>Lock Free | No SPOF<br/>Monitoring<br/>Load Balance<br/>Non-Stop Patchable<br/>Connection Recovery | PaaS<br/>Test Support<br/>Connectors<br/>Documentation<br/>Samples | Node<br/>Runtime Scalable<br/>TCP / IP<br/>WebSocket<br/>HTTP(S) | 

<a id="recommended-developers-developers-who-know-how-to-handle-java"></a>
#### 1. **Java**を扱える開発者

* Javaを扱える開発者にとっては最上の選択です。エンジンが提供する処理フローの上でコンテンツだけに集中できます。
* DBやRedis、そしてHTTPなど、ゲームサーバー開発に必要な全てのAPIをGameAnvilが提供します。

<a id="recommended-developers-developer-who-has-developed-a-game-server-in-other-languages-such-as-c-c-and-more"></a>
#### 2. **C++、C#**など他の言語でゲームサーバーを開発した経験がある開発者

* 他の言語でゲームサーバーを開発した経験がある開発者は、Javaの基本的な文法さえ覚えれば、すぐにコンテンツ開発を始めることができます。
* Javaと他の言語との間にある違い以外には、問題になることはありません。Javaを学びながらコンテンツ開発を進めてみてください。GameAnvil開発陣もC++からJavaへ大きな困難なく移行しました。

<a id="recommended-developers-a-junior-or-beginner-developer-who-has-never-developed-a-game-server"></a>
#### 3. **ゲームサーバーを開発した経験がないジュニアまたは新人開発者**

* ゲームサーバーを開発した経験がなくても、提供されるガイドドキュメントとリファレンスを基に、簡単かつ快適にリアルタイムコンテンツを開発できます。
* 実際のゲームサーバーで処理すべき大部分の機能はエンジンが担当するため、コンテンツだけに集中できます。

<a id="unsupported-games"></a>
## まだサポートしていないゲーム { #unsupported-games }

GameAnvilはまだ進化の段階にあります。そのため、まだ以下のようなスタイルのゲームはサポートしていません。しかし、早いうちにサポートできるよう、全ての開発陣が今も最善を尽くしています。

* MMO(RPG)ゲーム
* FPSまたはAOSなどのP2Pベースのゲーム

<a id="configure-reference-server"></a>
## 参考サーバー構成 { #configure-reference-server }

| ゲームスタイル | 最大同時接続者数 | リアルタイムコンテンツ有無 | マッチメイキング使用 | ** 最小VM台数 | 大まかな構成                                  |
|--------|-------------|------------|----------|-------------|-------------------------------------------|
| カジュアル    | 10000       | O          | O        | 5           | Gateway x 2, Game x 2, その他 x 1             |
| カジュアル    | 10000       | X          | X        | 3           | Gateway x 1, Game x 1, その他 x 1             |
| インディー      | 3000        | O          | X        | 1           | All in One x 1                               |
| ターン制ゲーム  | 200000      | O          | O        | 49          | Gateway x 20, Game x 25, Loc x 3, その他 x 1 |

** あくまで大まかな数値です。実際のコンテンツのボリュームやゲームスタイルによって異なる場合があります。

** VMのスペックによっても台数が異なる場合があります。

** DBやRedisなどのストレージは台数から除外されます。

<a id="additional-videos"></a>
## 参考動画 { #additional-videos }

* NDC 2021 [Javaでリアルタイムゲームサーバーエンジンの開発は可能？うん、可能](https://youtu.be/kQyu5pAChcA)
* NHN Cloud On 2022 ウェビナー [On.5 オンラインゲーム開発も簡単かつ迅速に](https://www.youtube.com/watch?v=Uv2a6fAU1xM)

<a id="information-on-personal-information-processing"></a>
## 個人情報処理に関する案内 { #information-on-personal-information-processing }

GameAnvilサービスを利用する過程で、お客様は利用者の個人情報を収集/利用する場合があり、この場合、お客様は個人情報保護法など関連法令を遵守する義務があります。
また、この過程で、お客様とNHN Cloud間で個人情報処理に関する業務委託関係が発生する場合があります。委託者の地位にあるお客様は、受託社であるNHN Cloudと別途書面による委託契約を締結することができ、
お客様が運営する個人情報処理方針に、以下の内容を参考にして告知できます。

* 受託業者：NHN Cloud(株)
* 委託業務の内容：GameAnvilサービスの提供
