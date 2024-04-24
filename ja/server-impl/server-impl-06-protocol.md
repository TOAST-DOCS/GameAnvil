## Game > GameAnvil > サーバー開発ガイド > プロトコルの定義



## プロトコルの定義とコンパイル

GameAnvilは[Google Protocol Buffers](https://developers.google.com/protocol-buffers)を使用してプロトコルを定義し、ビルドします。次の例では、これらのプロトコルを定義する方法とビルドする方法を説明します。まず、SampleGame.protoファイルをテキストエディタで作成した後、必要なプロトコルを定義します。プロトコルバッファの詳細については[公式Protocol Buffersガイド](https://developers.google.com/protocol-buffers/docs/proto3)を参照してください。

```protobuf
package [パッケージ名];

// 参照するべき他のprotoファイルがあればimport
import [protoファイル名]

enum SampleUserTypeEnum {
    USER_TYPE_NONE = 0;
    USER_TYPE_ONE = 1;
}

message SampleUserData {
    int32 data = 1;
}

enum SampleRoomTypeEnum {
    ROOM_TYPE_NONE = 0;
    ROOM_TYPE_ONE = 1;
}

message SampleRoomData {
    int32 data = 1;
}

message SampleUserMessage {
    SampleUserTypeEnum sampleUserTypeEnum = 1;
    int32 id = 2;
    int64 money = 3;
    string nickname = 4;
    double score = 5;
    repeated SampleUserData sampleUserData = 6; // list
}

message SampleRoomMessage {
    SampleRoomTypeEnum sampleRoomTypeEnum = 1;
    int32 id = 2;
    int64 potMoney = 3;
    string roomName = 4;
    repeated SampleRoomData sampleRoomData = 5; // list
}
```

また、次のようなコマンドラインでプロトコルスクリプトをコンパイルできます。この時、シェルスクリプトやバッチファイルを事前に作成して、使用すると便利です。GameAnvilテンプレートを利用してプロジェクトを構成した場合は、プロジェクト内のprotoフォルダでプロトコルスクリプトとバッチファイルが参考になります。コンパイルが正常に完了すると、プロトコルに関連するすべてのJavaクラスが自動的に作成されます。

```bash
protoc  ./MyGame.proto --java_out=../java
```

Unityクライアント用のC#プロトコルが必要な場合は、次のような言語を追加して作成することもできます。この場合、C#結果ファイルはその作成パスによって手動でUnityプロジェクトにコピーしなければならないこともあります。

```bash
protoc ./MyGame.proto --java_out=../java --csharp_out=./
```


## GeneratedMessageV3とパケット

GameAnvilサーバーでは、どんな転送が可能なメソッドでもプロトバッファオブジェクトをそのまま使用できるように、ほとんどの`com.google.protobuf.GeneratedMessageV3`クラスをサポートしています。通常、プロトバッファオブジェクトをそのまま使用しても問題ありませんが、複数のクライアントに転送するなど、特定の状況では、`com.nhn.gameanvil.packet.Packet`クラスを使用することで性能を向上させることができます。

送信されたGeneratedMessageV3クラスは、内部的にパケットに変換されシリアル化した後に転送するため、次の`broadcastMessage`を呼び出す状況では、パケットに変更するプロセスで同じプロトバッファを何度もシリアル化する問題が発生する可能性があります。

```java
// 次のような状況では、何度もシリアル化することで性能の問題が発生する可能性があります。
void broadcastMessage(List<GameUser> users, GeneratedMessageV3 message) {
    for (GameUser user : users) {
        user.send(message);
    }
}
```

このような状況を防ぐため、次のように修正します。パケットオブジェクトでは、内部的にシリアル化された情報を含んでいるため、何度転送しても正常に動作します。

```java
// 性能問題の修正
void broadcastMessage(List<GameUser> users, GeneratedMessageV3 message) {
    Packet p = Packet.makePacket(message); 
    for (GameUser user : users) {
        user.send(p);
    }
}
```

次のような状況では、パケットを使用しなくても問題はありませんが、コードの可読性が落ちる可能性があるため、プロトバッファオブジェクトをそのまま渡すことを推奨します。

```java
// 次のような状況では、GeneratedMessageV3を使用すると利便性が向上します。 
void sendMessage(GameUser user, GeneratedMessageV3 message) {
    user.send(message);
}
```
