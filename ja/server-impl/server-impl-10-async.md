## Game > GameAnvil > サーバー開発ガイド > 非同期サポート

## 非同期サポート

GameAnvilは、次のような目的のために非同期処理をサポートします。

![async-goal.png](https://static.toastoven.net/prod_gameanvil/images/async-goal.png)

ファイバー上での非同期処理のためにGameAnvilはAsyncクラスを提供しています。次のようなimport文でAsyncクラスを利用すると、一般的なブロッキング/ノンブロッキング呼び出しをすべてファイバー化できます。

```java
import com.nhn.gameanvil.async.Async;
```


> [参考]
> 
> すべての非同期サポートAPIについての説明は[GameAnvil API Reference](https://gameplatform.nhncloud.com/docs/api/gameanvil/1.4/com/nhn/gameanvil/async/Async.html)においてJavaDocで作成された文書を確認できます。



呼び出し用のAPIは大きくcallとrunに分けられ、それぞれ戻り値がある場合とそうではない場合に使用します。その他のスレッドベースのfutureをファイバーベースで使用できるように切り替えます。それぞれの用途に応じた使用方法は、次の部分でさらに詳しく説明します。

### ブロッキング呼び出し処理

一般的なブロッキング呼び出しは、スレッドブロッキングを意味します。つまり、現在のコードが実行中のファイバーだけでなく、このファイバーをスケジューリングするスレッドまでブロッキングするという意味です。すなわち、ノードが停止するという意味であるため、絶対にブロッキング呼び出しを直接的に使用してはいけません。GameAnvilは、このようなスレッドブロッキング呼び出しをファイバーブロッキングで切り替えるAsync APIを提供しています。このAPIは外部executorを使用して該当ブロッキング呼び出しを処理した後、完了後のコードフローを再びファイバー化します。戻り値の有無によって、runBlocking()とcallBlocking()のいずれかを使用してください。また、基本概念で説明したように、これらのファイバーブロッキングAPIはSuspendableであるため、API呼び出しメソッドは必ずSuspendExecution例外シグネチャを明示する必要があります。

```java
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() throws SuspendExecution {

    Async.runBlocking(executor, runnable); // スレッドブロッキング呼び出しをファイバーブロッキング呼び出しに切り替え

}
import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() throws SuspendExecution {

    return Async.callBlocking(executor, callable);  // スレッドブロッキング呼び出しをファイバーブロッキング呼び出しに切り替え

}
```

### Future処理

Futureに対する待機はスレッドブロッキングを誘発します。例えば、次のようなコードは呼び出しスレッドをブロッキングします。

```java
Future<SomeObject> future = someAsyncJob();

SomeObject ret = future.get(); // スレッドブロッキングを誘発
```

GameAnvilは、このようなfutureに対する待機をスレッドブロッキングからファイバーブロッキングに切り替えるAPIを提供しています。ただし、このAPIはJavaのCompletableFutureとGuavaのListenableFutureのみサポートします。しかし、ほとんどのライブラリは、この2つのfutureをベースに非同期をサポートするため、問題なく適用できるはずです。次のコードは、このようなAsync APIを利用してfutureに対する待機をファイバーブロッキングで処理する例です。

```java
// lettuce future, jdk CompletableFutureなど
CompletionStage<SomeObject> future = someAsyncJob();

SomeObject ret = Async.awaitFuture(future); // 該当ファイバーのみをブロッキング
```

### ブロッキング処理の委任

ここまでブロッキング呼び出しに対する処理について見てきました。Async APIのrunBlocking()やcallBlocking()は、ブロッキング処理を完了した後、再び該当ファイバーの実行フローを継続する場合に使用します。一方で外部スレッドにブロッキング呼び出しを委任した後、その結果を気にする必要がなければ、実行フローを継続できます。この場合は、次のAPIを使用します。このAPIはブロッキング呼び出しの結果を待たないので、Suspendableしないという点に注意してください。

```java
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() { // NOT suspendable
    Async.exec(executor, runnable); // 外部スレッドにブロッキング呼び出しを委任したため、このファイバーはブロッキングされない。
}
```

## 非同期Redisサポート

GameAnvilユーザーは、どのRedisクライアントを使用するかを選択できますが、これによりRedisに関連する問題の種類と複雑度がユーザーが選択したRedisクライアントの種類や使用方式に比例して増加しました。したがって、GameAnvilはGameAnvilがサポートするRedisクライアントの種類と基本的な使用方法をAPI化し、Redisクライアントの使用方法を含む基本ガイドラインを提供しています。提供されるAPIではなく、他の種類のRedisクライアントを選択して使用することも可能ですが、特別な理由がなければ、使用を控えることを推奨します。

> [参考]
> 
> 以降の内容では、GameAnvilが提供するLettuceクラスと製品名である"Lettuce"を区別するために、電子の場合は可能な限り"Lettuceクラス"と表記し、一部の内容上必要に応じて"Lettuce"と表記することもあります。これと区別するために、製品名はすべて大文字のLETTUCEで表記します。この文章で説明するLETTUCEは、GameAnvilでの使用方法にフォーカスを当てているため、それ以上の説明が必要な場合は[LETTUCE公式ページ](https://github.com/lettuce-io/lettuce-core)を参照してください。


GameAnvilはRedisクライアントとしてLETTUCEの使用を推奨します。GameAnvilが提供しているRedisラッピングAPIもLETTUCEを使用します。また、LETTUCEは非同期Redisクライアントであり、大部分の非同期APIはCompletableFutureに基づいています。つまり、GameAnvilのAsync APIを利用してファイバーベースの非同期化に切り替えることができることを意味します。

GameAnvilが提供するRedisラッピングAPIは、大きく3つクラスであるLettuce、RedisCluster、RedisSingleに分けられます。Lettuceは最も一般的な形式の使用方法を提供し、内部的にLETTUCEオブジェクトを管理しないstaticクラスです。したがって、LETTUCEを最も一般的な形式で使用したい場合は、このLettuceクラスが最適です。RedisClusterとRedisSingleは、それぞれRedisクラスタとスタンドアローンに対応するためのクラスで、内部的にLETTUCEオブジェクトを管理します。

### Lettuce

Lettuceクラスはファイバー単位の処理のために最も核心的なstatic APIを提供します。内部的にRedisに関するいかなる状態も保管しないため、別途のオブジェクトを作成する必要がなく、すぐに使用できます。Lettuceライブラリにある程度慣れている場合は、Lettuceクラスを直接使用するのが最適です。

```java
import com.nhn.gameanvil.async.redis.Lettuce;
```

次の3つの注意事項の他に、基本的なLettuceの使い方をそのまま維持できます。

- 第一に、必ずconnectは、GameAnvilのLettuce.connect()もしくは、Lettuce.connectAsync()を使用する。コネクションは基本的にスレッドをブロッキングするため、これに対するファイバー化処理が含まれます。
- 第二に、shutdownもコネクションと同じ理由でLettuce.shutdown()を使用する必要があります。
- 第三に、RedisFutureに対する待機は必ずLettuce.awaitFuture()を使用してファイバーブロッキング化する必要があります。

これらのLettuceクラスを使用してRedisに接続するコードは次のとおりです。

```java
RedisURI clusterURI = RedisURI.Builder.redis(IP_ADDRESS, 7500).build();
clusterClient = RedisClusterClient.create(Arrays.asList(clusterURI));
clusterConnection = Lettuce.connect(RpsConfig.DB_THREAD_POOL, clusterClient);

if (clusterConnection.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### RedisCluster

```java
import com.nhn.gameanvil.async.redis.RedisCluster;
```

Redis Clusterに対するAPIをラッピングします。基本的に前述したLettuceと使い方はほとんど変わりません。しかし、このクラスはLettuce関連オブジェクト(e.g.RedisClusterClient、StatefulRedisClusterConnectionなど)を自ら管理します。このようなLettuceオブジェクトを直接管理するのではなく、RedisClusterで管理する時の使用が考えられます。

注意事項はLettuceの場合と全く同じです。以下は、RedisClusterを利用してRedisに接続するコードです。

```java
redisClient = RedisSingle.create("redis://IP_ADDRESS:6379");
redisAsyncCommands = Lettuce.connect(RpsConfig.DB_THREAD_POOL, redisClient).async();

if (redisClient.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### RedisSingle

```java
import com.nhn.gameanvil.async.redis.RedisSingle;
```

RedisClusterと比較すると、対象のRedisがスタンドアローンという点を除けば違いはありません。

注意事項はLettuceの場合と全く同じです。以下は、RedisSingleを利用してRedisに接続するコードです。

```java
redisCluster = new RedisCluster<>(IP_ADDRESS, 7500);
redisCluster.connect(RpsConfig.DB_THREAD_POOL, StringCodec.UTF8);
if (redisCluster.isConnected()) {
    logger.warn("============= Connected to Redis using Lettuce =============");
}
```

### RedisFutureをファイバーで使用する

Lettuce、RedisClusterそして、RedisSingleはすべてLettuceライブラリがサポートするRedisFutureをファイバー上で待機できるAPIを提供しています。内部実装はすべてエンジンが提供するAsync.awaitFuture()を同じように使用するため、混用しても構いません。次の4つのコードはすべて同じコードです。GameAnvilのファイバー上でRedisFutureに対するget()は、必ずこの4つのいずれかの方法を使用する必要があります。

- Async.awaitFuture()

```java
try {  
    Async.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {  
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- Lettuce.awaitFuture()

```java
try {
    Lettuce.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- RedisCluster.awaitFuture()

```java
try {
    redisCluster.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- RedisSingle.awaitFuture()

```java
try {
    redisSingle.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- **誤った使用方法**:直接Futureに対する待機をした場合、該当Node(Thread)がブロッキングされるため、絶対に次のようなコードを使用してはいけません。

```java
try {
    RedisFuture future = clusterAsyncCommands.mget("testKey", getUserId()));
    future.get(); // スレッドブロッキングを誘発
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

### set/get

最も基本となるsetとgetは、RedisCluserとRedisSingleが基本提供します。

- RedisClusterを利用したset/getの例

```java
String setResult = redisCluster.set(key, value);
String getResult = redisCluster.get(key);
```

- RedisSingleを利用したset/getの例

```java
String setResult = redisSingle.set(key, value);
String getResult = redisSingle.get(key);
```

- 直接LETTUCEのRedisAsyncCommandsオブジェクトを使用した例

```java
RedisFuture<String> setFuture = redisAsyncCommands.set(key, value);RedisFuture<String> getFuture = redisAsyncCommands.get(key);String setResult = Async.awaitFuture(setFuture);String getResult = Async.awaitFuture(getFuture);
```

### 本格的なLETTUCE非同期処理

Redisが提供するさまざまなコマンドは、LETTUCEのCommandsオブジェクトで使用可能です。基本的にLETTUCEはSync方式のCommandsオブジェクトとAsync方式のCommandsオブジェクトを提供していますが、GameAnvilはその中でもAync方式の使用を推奨します。基本的にAsyncCommandsは、Redis Clusterの場合とStandAloneの場合について、それぞれ次のとおりです。

- RedisAdvancedClusterAsyncCommands
- RedisAsyncCommands

以下の例は、このようなAsyncCommandsオブジェクトを利用してmgetを行う例です。LETTUCEの非同期処理は基本的にRedisFutureを使用し、このRedisFutureはCompletableFutureです。CompletableFutureの詳細については[Java公式リファレンス](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)で確認できます。また、以下の例は、LETTUCEに対する非同期処理のごく一部の方式のみを示しているため、そのまま使用するのではなく、開発中のコードに合わせて作成してください。完璧な非同期コードを制御するためには、必ず[LETTUCE](https://github.com/lettuce-io/lettuce-core)と[CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)に関する内容を熟知してください。

> [参考]
>
> thenApply()とthenAccept()は、任意の外部スレッドから呼び出されるため、Nodeで管理する内部リソースにアクセスしたり、リソースに対するLockを使用したりしてはいけません。
> 
> - 例1> key1とkey2に対する値を非同期で取得
>
> ```java
> Lettuce.awaitFuture(asyncCommands.mget("key1", "key2"));
> ```
> 
> - 例2> それ以降のコードの流れと関係がない場合は、future chainで外部スレッドに処理を委任(すなわち、mgetでの値取得が完了するまで待機する必要がない場合)
>
> ```java
> RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");
>
> future.thenApplyAsync(r -> {
>     Map<String, String> map = new HashMap<>();
>     for (KeyValue<String, String> kv : r)
>         map.put(kv.getKey(), kv.getValue());
>     return map;
> }).thenAccept(r -> {
>     for (Entry<String, String> entry : r.entrySet())
>         logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
> });
> ```
>
> - 例3> それ以降のコードの流れと関係がある場合は該当futureを待機した後、処理
>
> ```java
> RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");
>
> CompletionStage<Map<String, String>> cs = future.thenApplyAsync(r -> {
>     Map<String, String> map = new HashMap<>();
>     for (KeyValue<String, String> kv : r)
>         map.put(kv.getKey(), kv.getValue());
>     return map;
> });
>
> // do something here
>
> try {
>     // ファイバー上で該当futureを待機するためには、Lettuce.awaitFuture()を使用する必要があります。
>     Map<String, String> map = Lettuce.awaitFuture(cs);
>
>     for (Entry<String, String> entry : map.entrySet())
>         logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
> } catch (TimeoutException e) {
>     logger.error("GameUser::onLogin()", e);
> }
> ```

## 非同期HttpReqeust & HttpResponseの使用方法

Http処理に関する部分もRedisと同様にGameAnvilが基本的なAPIとガイドラインを提供します。もちろん、他の種類のHttpの使用も選択できますが、特別な理由がなければ、使用を控えることを推奨します。GameAnvilは非同期ベースのHttpを使用するため、内部的に[AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)を使用します。次に説明するAPIとその使用範囲を超える場合は、GameAnvilが提供するAPIより直接[AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)を使用することを推奨します。LETTUCEと同様にAsyncHttpClientも内部的にCompletableFutureを使用するため、futureに対する待機をAsync.awaitFuture()を利用してファイバー化するだけで、残りの一般スレッド上での使用方法と全く同じです。

```java
Async.awaitFuture(future.get()); // ファイバー上で該当futureを待機します。
```

GameAnvilが提供するHttp APIは、リクエストとレスポンス用のHttpRequest、HttpResponseクラスそして、結果に対する一般的な処理用のHttpResultTemplateクラスからなります。このクラスを利用すると、簡単かつ直感的にHttpリクエストとレスポンスを処理でき、その結果を希望する形にすることもできます。また、すべてのコードは非同期であるため、特別な処理が必要ありません。以下は、これを使用したサンプルコードです。

### HttpReqeust & HttpResponseの使用
HttpRequestライブラリは、GameAnvilで長い間使用されてきましたが、関連ライブラリがアップデートされず、使用時にいくつかの問題が発生しました。これらの問題を解決するために、内部のHttpライブラリを変更したHttpRequest2クラスをサポートしています。もしHttpRequestを使用中に問題が発生した場合は、HttpRequest2構成への変更を推奨します。今後、リリース時にHttpReuqest2に問題が発生しなかった場合は、既存のHttpRequestは削除される可能性があります。

- 例1> 最も基本的な使用方法は、内部的にファイバー単位のfuture処理をしてくれるので最も直感的です。特別な理由がなければ、これらの基本的な使用方法だけで問題ありません。

```java
HttpRequest request = new HttpRequest(URL);
HttpResponse response = request.GET();
```

- 例2> futureベースの非同期方式HTTPリクエストとレスポンス待機中に他の作業を行いたい場合は、次のようなfutureを直接利用できます。

```java
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

// Do something here

HttpResponse response = new HttpResponse(Async.awaitFuture(future, 10000, TimeUnit.MILLISECONDS));
```

- 例3> HTTPリクエストheader構成の例のように、AsyncHttpClientはさまざまなAPIを提供します。AsyncHttpClientについての詳しい使用方法は[公式ページ](https://github.com/AsyncHttpClient/async-http-client)を参照してください。

```java
HttpRequest request = new HttpRequest(url);
request.getBuilder()
    .addHeader("if-none-check-node", "false")
    .setRequestTimeout(timeout);
    .addQueryParam("serviceId", serviceId);

HttpResponse httpResponse = request.GET();
```

- 例4> それ以降のコードの流れと関係がない場合は、future chainで外部スレッドに処理を委任(Lettuceの場合と同じ方式)

```java
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

future.thenApplyAsync(r -> {
    try {
        HttpResponse response = new HttpResponse(r);
        String body = response.getContents(String.class);
        HttpResultTemplate<JsonObject> result = GameAnvilUtil.Gson().fromJson(body, new RestResponseParamType(JsonObject.class));
        if (!result.getHeader().getIsSuccessful()) {
            logger.warn("GET failed : resultCode {}, resultMessage {}",
                result.getHeader().getResultCode(),
                result.getHeader().getResultMessage());
        }
        return result.getContents();
    } catch (IOException e) {
        logger.error("Exception occurred: ", e);
        return null;
    }
}).thenAccept(r -> {
    if (r != null) {
        JsonElement element = r.get(ELEMENT_NAME);
        if (element != null) {
            logger.info("The response code: {}", element.getAsString());
        }
    }
});
```

- 例5> それ以降のコードの流れと関係がある場合は該当futureを待機した後、処理

```java
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

CompletionStage<JsonObject> cs = future.thenApplyAsync(r -> {
    try {
        HttpResponse response = new HttpResponse(r);
        String body = response.getContents(String.class);
        HttpResultTemplate<JsonObject> result = GameAnvilUtil.Gson().fromJson(body, new RestResponseParamType(JsonObject.class));
        if (!result.getHeader().getIsSuccessful()) {
            logger.warn("GET failed : resultCode {}, resultMessage {}",
                result.getHeader().getResultCode(),
                result.getHeader().getResultMessage());
        }
        return result.getContents();
    } catch (IOException e) {
        logger.error("GameUser::onLogin()", e);
        return null;
    }
});

// do something here

try {
    // ファイバー上で該当futureを待機するためには、Async.awaitFuture()を使用する必要がある。
    JsonObject jsonObject = Async.awaitFuture(cs);
    if (jsonObject != null) {
        JsonElement element = jsonObject.get(ELEMENT_NAME);
        if (element != null) {
            logger.info("The response code: {}", element.getAsString());
        }
    }
} catch (TimeoutException e) {
    logger.error("Exception occurred: ", e);
}
```

### HttpReqeust2の使用

上記のHttpRequestライブラリは、GameAnvilで長い間使用されてきましたが、関連ライブラリがアップデートされず、使用時にいくつかの問題が発生しました。これらの問題を解決するために、内部のHttpライブラリを変更したHttpRequest2クラスをサポートしています。もしHttpRequestを使用中に問題が発生した場合は、HttpRequest2構成への変更を推奨します。今後、リリース時にHttpReuqest2に問題が発生しなかった場合は、既存のHttpRequestは削除される可能性があります。

- 例1> その後のコードの流れと関係がある場合は該当futureを待機した後、処理

```java
HttpRequest2 request = new HttpRequest2(Method.GET, GET_LIST_URL);

try {
    HttpResponse2 httpResponse = request.execute();
    String body = httpResponse.getContents(String.class);
    System.out.println(body);
} catch (Exception e) {
      logger.error("Exception occurred: ", e);
}
```

## RDBMSの非同期処理

RDBMSに対するクエリは一般的にブロッキングです。このようなブロッキングクエリをGameAnvil上で処理する方法は、前述した他のAsyncの使い方とほとんど変わりません。ある種類のRDBMSを使用していたSQLクエリに対するコードは、同じ方法で実装できます。また、エンジンユーザーはDBへアクセスするために、自由にSQL MapperやORMを選択できます。

一方で、これらのクエリを基本的にノンブロッキング方式の非同期処理をしてくれる[MySQL X DevAPI](https://dev.mysql.com/doc/x-devapi-userguide/en/)や[jasync-sql](https://github.com/jasync-sql/jasync-sql)のような非同期DBドライバーがあります。GameAnvilはこの両方をサポートしていますが、MySQL X DevAPIにいくつかの欠陥が発見されたため、ベータバージョンの独立したライブラリの形でのみ提供します。つまり、GameAnvilはjasync-sqlをベースに非同期クエリを完璧にサポートします。特別な理由がなければ、ユーザーもjasync-sqlを使用することを推奨します。

### ブロッキングクエリ

ブロッキングクエリは呼び出しスレッドをブロッキングするため、必ず処理する必要があります。これらのブロッキングクエリに対する非同期処理は、クエリの結果が必要な場合とそうではない場合に分けられます。この2つは、クエリ結果の有無にのみ違いがあり、クエリ全体の実行が完了するまで該当ファイバーが待機する点は同じです。つまり、非同期でリクエストしたクエリが完了した後、次のコードに進むため、エンジンユーザーは一般的なブロッキングコードを作成する場合と同じように実装できます。

> [参考]
> 
> DBにクエリを実装するプロセスで最も重要であるのに、よく見逃してしまう部分は、DBに対するCP(ConnectionPool)サイズとこれを非同期で処理するTP(ThreadPool)の数の設定とこれらの関係に対する理解です。通常、これら2つの数値は処理するクエリの量を考慮して、同じ値に設定するか、TPをCPより少し多めに設定します。また、GameAnvilを利用した大規模性能テストの結果、サーバープロセス1つあたり6000～8000人の処理基準TPとCP250個の設定が最も良い結果を示しました。しかし、あくまでもクエリの複雑度と頻度など、複合的な要素を考慮しながら、できるだけ多くのテストを経て、最適な値を探すことが最善です。

第一に、クエリの結果を取得したい場合は、次の例のようにAsyncクラスのcallBlocking APIを使用します。callBlockingは、ファイバー上で任意のブロッキング呼び出しを実行した後、結果を返します。

```java
try {
    return Async.callBlocking("MyThreadPool", new Callable<List<T>>() {
        @Override
        public List<T> call() throws Exception {
            return myQueryCode();
        }
    });
} catch (TimeoutException e) {
    logger.error("TimeoutException occured: ", e);
}

logger.info("Query has finished.");
```

この時、非同期処理用のスレッドプールは、Bootstrap段階で事前に作成しておくことができます。

```
gameAnvilServer.createExecutorService("MyThreadPool", 250);
```

もしくは、エンジンユーザーが必要に応じて直接作成した外部スレッドプールを使用することもできます。

```
gameAnvilServer.createExecutorService(myExecutorService, 250);
```

第二に、クエリの結果が必要でない場合は、次の例のようにAsyncクラスのrunBlocking APIを使用します。runBlockingはファイバー上で任意のブロッキング呼び出しを実行します。

```java
try {
    Async.runBlocking("MyThreadPool", new Runnable() {
        @Override
        public void run() {
            try {
                myQueryCode();
            } catch (Exception e) {
                logger.error("Exception occured during query code: ", e);
            }
        }
    });
} catch (TimeoutException e) {
    logger.error("TimeoutException occured: ", e);
}

logger.info("Query has finished.");
```

この場合も同様に任意のスレッドプールをrunBlocking APIに引数で送信できます。

### ノンブロッキング非同期クエリ

前述したように、GameAnvilはjasync-sqlを基本の非同期DBドライバーとして使用します。その使用方法は非常に直感的かつ簡単なため、既存のブロッキングクエリよりコード生産性が高く、その性能もはるかに優れています。まず、GameAnvilが提供するJasync-sqlを使用するためには、次のようなimport文を追加します。

```java
import com.nhn.gameanvil.async.db.JAsyncSql
```

JasyncSqlクラスは非同期クエリ用の機能がGameAnvilファイバー上で柔軟に動作するようにサポートします。一般的に特別な理由がなければ、ノードごとに1つのJasyncSqlオブジェクトを作成して、使用する方法が最適です。そして、非同期クエリを使用する時は、ブロッキング方式とは異なり、ユーザーが別途のスレッドプールやコネクションプールを作成する必要がありません。

次は、JasyncSqlオブジェクトを作成するコードです。引数のうち64個の最大アクティブコネクション数は、使用用途とクエリ頻度に合わせて最適化できます。

```java
JAsyncSql jasyncSql = new JAsyncSql(new com.github.jasync.sql.db.Configuration(
                                    "gameanvil",
                                    "127.0.0.1",
                                    13306,
                                    "%gameanvil1",
                                    "GameDB_1"), 64));  // 64個の最大アクティブコネクション
```

JasyncSqlオブジェクトによって非同期クエリをリクエストした後、CompletableFutureを返してもらうことができます。一般的なfutureベースの非同期コードです。

```java
CompletableFuture<QueryResult> future = jasyncSql.executeAsync("SELECT * FROM UserInfo");

... // do something others

Async.awaitFuture(future); // 該当ファイバー上で非同期でfutureを待機
```

また、クエリ結果をすぐに取得するために該当ファイバーに対する待機を含む同期APIも提供します。

```java
QueryResult result = jasyncSql.execute("SELECT * FROM UserInfo");
```

このコードは、前述したfutureベースの非同期コードを1つにまとめたものと同じです。これらすべてのコードは、スレッド単位で非同期化するのではなく、ファイバー単位で動作します。

### ノンブロッキング非同期クエリvsブロッキングクエリ

この2つの方式は、使用方法とコード生産性だけでなく、性能も明確に異なります。同じ環境で2つのクエリ方式の性能を測定した結果は、次の図のとおりです。

![](https://static.toastoven.net/prod_gameanvil/images/mysql-async-performance.png)

jasync-sqlベースの非同期クエリが最も高性能です。これは、MapperやORMを使用したブロッキングクエリに比べて約2倍の性能向上を示しています。そのような側面から、GameAnvilはユーザーに特別な理由がなければ、これらの非同期クエリの使用を控えることを推奨します。
