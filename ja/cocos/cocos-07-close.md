## Game > GameAnvil > CocosCreator開発ガイド > GameAnvilコネクタ終了

## GameAnvilコネクタ終了

ゲームプレイを終了する前に、Connector.CloseSoket() 関数を呼び出して接続を終了することをお勧めします。 終了しないと、サーバーがクライアントの終了を認識しない可能性があり、その場合、不要な動作を継続する可能性があります。Connectorを管理するコンポーネントをゲーム終了時にのみ破壊されるようにし、OnDestroy()でConnector.CloseSoket()を呼び出すようにするのも良い方法です。
