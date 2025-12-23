## Game > GameAnvil > Unity 応用開発ガイド > パケット

## パケット

GameAnvilは基本メッセージプロトコルとしてProtocolBufferメッセージを使用します。そしてこれらのメッセージはパケットに込められて処理されます。ほとんどの場合、GameAnvilConnectorを利用する時はProtocolBufferメッセージのみを使用しても問題ありませんが、状況によってはPacketを利用しなければならない場合もあります。 

### 生成
次のようにProtocolBufferメッセージを利用してパケットを生成できます。 
```c#
Packet packet = Packet.MakePacket(new Protocol.SampleRequest());
```

ProtocolBufferメッセージを利用しなくてもパケットを生成できます。
```c#
byte[] requestMsg = Encoding.UTF8.GetBytes(JsonString);
Packet packet = Packet.MakeCustomPacket(customMsgId, requestMsg);

ByteString bytes = packet.ToByteString();
string JsonString = bytes.ToStringUtf8();
```
### 圧縮

サーバーへ送信できるパケットの最大サイズは64Kbytesに制限されています。パケットサイズが64Kbytesを超える場合、圧縮を通じてサイズ制限を回避できます。
ペイロードも内部的にはパケットとして処理されるため、同様に64Kbytesを超えることはできません。

```c#
Packet packet = Packet.MakePacket(new Protocol.SampleRequest(), PacketOption.Compress);
```

### 送信
このように生成したパケットは、ProtocolBufferメッセージを送信するのと同じ方法でサーバーへ送信できます。 
```c#
public async void RequestPacket()
{
    try
    {
        Packet packet = Packet.MakePacket(new Protocol.SampleRequest());
        ErrorResult<ResultCode, Protocol.SampleResponse> result = await connector.Request<Protocol.SampleResponse>(packet);
        if (result.ErrorCode == ResultCode.SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    } catch (Exception e)
    {
        // 例外
    }
}
```

### Payload

GameAnvilが提供する基本APIを利用する際、追加のデータが必要になる場合があります。このために基本APIには、追加データを渡すことができるペイロードというパラメータが含まれています。このペイロードに追加で必要なデータを追加でき、このように追加したデータをサーバーへ送ったり、サーバーから受け取り利用できます。 

```c#
Payload payload = new Payload(new Protocol.SampleRequest());
payload.Add(new Protocol.SampleSend());

Packet packet = payload.GetPacket(Protocol.SampleReceive.Descriptor);
Protocol.EchoRecv echoRecv = payload.GetProtoBuffer<Protocol.SampleReceive>();
```
