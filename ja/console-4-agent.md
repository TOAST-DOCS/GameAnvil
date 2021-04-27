## Game > GameAnvil > コンソール使用ガイド > Agent

ConsoleからGameAnvilサーバーを実行/停止/配布するためのAgentをインストールする必要があります。

### Agentのダウンロードおよび実行

* [Agentダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-agent-1.1.3.tar)

* ダウンロードしたtarファイルを解凍すると、次のファイルを確認できます。

  * ファイル構成

    | ファイル名                       | 説明                  | 備考 |
    | ----------------------------------- | ------------------------- | ---- |
    | gameanvil-agent-*.jar ( *はバージョン名) | gameanvil-agentファイル  |      |
    | gameanvil-agent.properties          | gameanvil-agent設定ファイル |      |
    | start.sh                            | 実行スクリプト         |      |
    | stop.sh                             | 実行中止スクリプト    |      |

    

* GameAnvil-Agent設定ファイルの変更

  * server.addressとserver.portを確認します。

    | 設定名  | 説明              | 例                      | 備考                                                        |
    | -------------- | --------------------- | ----------------------------- | ------------------------------------------------------------ |
    | server.address | Agentが使用するip   | server.address=10.160.194.108 | この設定値が空白の場合、マシンに割り当てられたすべてのIPで接続できるため、使用するIPを必ず指定します。 |
    | server.port    | Agentが使用するport | server.port=19080             | consoleで設定されたGameAnvil Agent Portと値が同じでなければいけません。(デフォルト値：19080) |

    

* start.shスクリプトを利用してGameAnvil-Agentを実行します。

  * 該当スクリプトの権限が設定されていない場合は、chmodコマンドを利用して適切な権限を設定してください。
