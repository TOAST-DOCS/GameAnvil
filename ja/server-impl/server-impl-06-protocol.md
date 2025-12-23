## Game > GameAnvil > サーバー開発ガイド > プロトコル定義



## プロトコル定義とコンパイル

GameAnvilは[Google Protocol Buffers](https://protobuf.dev/)を使用してプロトコルを定義し、ビルドします。以下の例は、これらのプロトコルを定義する方法とビルドする方法を説明します。まず、SampleGame.protoファイルをテキストエディタで作成した後、希望のプロトコルを定義します。プロトコルバッファの詳細な文法は、[公式Protocol Buffersガイド](https://protobuf.dev/programming-guides/proto3/)を参照できます。

```protobuf
package [パッケージ名];

// 参照すべき他のprotoファイルがあればimport
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

また、以下のようなコマンドラインでこのプロトコルスクリプトをコンパイルできます。このとき、シェルスクリプトやバッチファイルを生成しておいて使用すると便利です。GameAnvilテンプレートを利用してプロジェクトを構成した場合、プロジェクト内のprotoフォルダでプロトコルスクリプトとバッチファイルを参照できます。コンパイルが成功裏に完了すると、プロトコルに関連する全てのJavaクラスが自動的に生成されます。

```bash
protoc  ./MyGame.proto --java_out=../java
```

UnityクライアントのためのC#プロトコルが必要な場合、以下のように言語を追加して生成することもできます。この場合、C#結果ファイルはその生成パスに従って手動でUnityプロジェクトにコピーする必要があるかもしれません。

```bash
protoc ./MyGame.proto --java_out=../java --csharp_out=./
```


## GeneratedMessageとパケット

GameAnvilサーバーでは、いかなる送信可能なメソッドでもプロトバッファオブジェクトをそのまま使用できるように、大部分`com.google.protobuf.GeneratedMessage`クラスをサポートします。一般的な状況ではプロトバッファオブジェクトをそのまま使用しても問題ありませんが、多数のクライアントへ送信するなど特定の状況では`com.nhn.gameanvil.packet.Packet`クラスを使用して性能を向上させることができます。

多数のクライアントへメッセージを送信する際は次のように使用できます。
```java
/**
  * 入力されたリストのユーザークライアントへメッセージ送信
 *
 * @param userList メッセージを送信するユーザーリスト
 * @param message  送信するメッセージ
 * @param <P>      送信するメッセージタイプ
 */
default <P extends GeneratedMessage.Builder<P>> void sendToClients(@NotNull final Collection<IUserContext> userList, @NotNull final GeneratedMessage.Builder<P> message) {
}

/**
  * 入力されたリストのユーザークライアントへメッセージ送信
 *
 * @param userList メッセージを送信するユーザーリスト
 * @param message  送信するメッセージ
 */
default void sendToClients(@NotNull final Collection<IUserContext> userList, @NotNull final GeneratedMessageV3 message) {
}

/**
 * 入力されたリストのユーザークライアントへパケット送信
 *
 * @param userList メッセージを送信するユーザーリスト
 * @param packet   送信するパケット
 */
void sendToClients(@NotNull final Collection<IUserContext> userList, @NotNull final Packet packet)
```
