<!-- pre-align:aligned sig=dd9a8996ee81 -->

<a id="game-gameanvil-unity-advanced-development-guide-packet"></a>
## Game > GameAnvil > Unity 応用開発ガイド > パケット { #game-gameanvil-unity-advanced-development-guide-packet }

<a id="packets"></a>
## パケット { #packets }

GameAnvilは基本メッセージプロトコルとしてProtocolBufferメッセージを使用します。そしてこれらのメッセージはパケットに込められて処理されます。ほとんどの場合、GameAnvilConnectorを利用する時はProtocolBufferメッセージのみを使用しても問題ありませんが、状況によってはPacketを利用しなければならない場合もあります。 

<a id="create"></a>
### 生成 { #create }
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
<a id="compression"></a>
### 圧縮 { #compression }

サーバーへ送信できるパケットの最大サイズは64Kbytesに制限されています。パケットサイズが64Kbytesを超える場合、圧縮を通じてサイズ制限を回避できます。
ペイロードも内部的にはパケットとして処理されるため、同様に64Kbytesを超えることはできません。

```c#
Packet packet = Packet.MakePacket(new Protocol.SampleRequest(), PacketOption.Compress);
```

<a id="sent"></a>
### 送信 { #sent }
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

<a id="payload"></a>
### Payload { #payload }

GameAnvilが提供する基本APIを利用する際、追加のデータが必要になる場合があります。このために基本APIには、追加データを渡すことができるペイロードというパラメータが含まれています。このペイロードに追加で必要なデータを追加でき、このように追加したデータをサーバーへ送ったり、サーバーから受け取り利用できます。 

```c#
Payload payload = new Payload(new Protocol.SampleRequest());
payload.Add(new Protocol.SampleSend());

Packet packet = payload.GetPacket(Protocol.SampleReceive.Descriptor);
Protocol.EchoRecv echoRecv = payload.GetProtoBuffer<Protocol.SampleReceive>();
```
