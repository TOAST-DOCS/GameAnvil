## Game > GameAnvil > 서버 개발 가이드 > 비동기 지원

## 1. 비동기 지원

GameAnvil은 다음과 같은 목적을 위해 비동기 지원을 합니다.

![async-goal.png](https://static.toastoven.net/prod_gameanvil/images/async-goal.png)

파이버 상에서의 비동기 처리를 위해 GameAnvil은 Async 클래스를 제공합니다. 아래와 같은 import문을 통해 Async 클래스를 이용하면 일반적인 블로킹/논블로킹 호출을 모두 파이버화 할 수 있습니다.

```java
import com.nhn.gameanvil.async.Async;
```



### Note

*모든 비동기 지원 API에 대한 설명은 [GameAnvil API Reference](https://gameplatform.toast.com/docs/api/com/nhn/gameanvil/async/Async.html)에서 JavaDoc으로 작성된 문서를 확인할 수 있습니다.*



호출용 API는 크게 call과 run으로 나뉘며 각각 반환값이 있는 경우와 그렇지 않은 경우에 사용합니다. 그 외 스레드 기반의 future를 파이버 기반으로 사용할 수 있도록 전환해줍니다. 각각의 용도에 따른 사용법은 문서의 다음 부분에서 더 자세하게 다루도록 합니다.

### **1.1.블로킹 호출 처리**

일반적인 블로킹 호출은 스레드 블로킹을 의미합니다. 즉, 현재 코드가 수행 중인 파이버 뿐만 아니라 이 파이버를 스케줄링하는 스레드까지 블로킹 시킨다는 뜻입니다. 이 말은 곧 노드가 멈춘다는 의미이므로 절대 블로킹 호출을 직접적으로 사용해서는 안 됩니다. GameAnvil은 이러한 스레드 블로킹 호출을 파이버 블로킹으로 전환해 주는 Async API를 제공합니다. 이 API는 외부 executor를 사용하여 해당 블로킹 호출을 처리한 후 완료 이후의 코드 흐름을 다시 파이버화 합니다. 반환값 유무에 따라 runBlocking()과 callBlocking() 중 하나를 사용하면 됩니다. 또한 기본 개념에서 설명했듯이 이러한 파이버 블로킹 API는 Suspendable하므로 API 호출 메서드는 반드시 SuspendExecution 예외 시그니처를 명시해야 합니다.

```java
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() throws SuspendExecution {

    Async.runBlocking(executor, runnable); // 스레드 블로킹 호출을 파이버 블로킹 호출로 전환

}
import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() throws SuspendExecution {

    return Async.callBlocking(executor, callable);  // 스레드 블로킹 호출을 파이버 블로킹 호출로 전환

}
```

### **1.2.Future 처리**

Future에 대한 대기는 스레드 블로킹을 유발합니다. 예를 들어 아래와 같은 코드는 호출 스레드를 블로킹합니다.

```java
Future<SomeObject> future = someAsyncJob();

SomeObject ret = future.get(); // 스레드 블로킹을 유발
```

GameAnvil은 이런 future에 대한 대기를 스레드 블로킹에서 파이버 블로킹으로 전환해주는 API를 제공합니다. 단, 이 API들은 Java의 CompletableFuture와 Guava의 ListenableFuture만 지원합니다. 다행히도 대부분의 라이브러리는 이 2가지의 future를 기반으로 비동기를 지원하기 때문에 큰 무리 없이 적용 가능할 것입니다. 아래의 코드는 이러한 Async API를 이용해서 future에 대한 대기를 파이버 블로킹으로 처리하는 예입니다.

```java
Future<SomeObject> future = someAsyncJob();

SomeObject ret = Async.awaitFuture(future); // 해당 파이버만 블로킹
```

### **1.3.블로킹 처리 위임**

앞서 블로킹 호출에 대한 처리를 살펴보았습니다. Async API의 runBlocking()이나 callBlocking()은 블로킹 처리를 완료한 이후에 다시 해당 파이버의 실행 흐름을 이어가는 경우에 사용합니다. 반면 외부 스레드로 블로킹 호출을 위임한 후 그 결과에 대해 신경 쓸 필요가 없다면 실행 흐름을 계속 이어갈 수 있을 것입니다. 이런 경우에는 아래의 API를 사용하면 됩니다. 이 API는 블로킹 호출의 결과를 대기하지 않으므로 Suspendable하지 않음에 주의하세요.

```java
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() { // NOT suspendable

    Async.exec(executor, runnable); // 외부 스레드로 블로킹 호출을 위임했으므로 이 파이버는 블로킹되지 않는다.

}
import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() {  // NOT suspendable

    return Async.exec(executor, callable);  // 외부 스레드로 블로킹 호출을 위임했으므로 이 파이버는 블로킹되지 않는다.

}
```

## 2. 비동기 Redis 지원

그동안 어떤 종류의 Redis 클라이언트를 사용할지는 전적으로 GameAnvil 사용자의 선택이었습니다. 하지만 이로 인해 Redis 관련 이슈의 종류와 복잡도가 사용자가 선택한 Redis 클라이언트 종류와 사용 방식의 차이에 비례해서 증가한다는 사실을 알게 되었습니다. 우리는 이를 방지하고자 GameAnvil의 기본적인 가이드라인에 Redis 클라이언트에 관한 사용법을 포함하기로 결정했습니다. 이 가이드라인은 GameAnvil에서 지원하는 Redis 클라이언트의 종류와 그 기본적인 사용법을 API화하여 통일시키는 것을 목표로 합니다. 물론 제공되는 API가 아닌 다른 종류의 Redis 클라이언트를 선택해서 별도로 사용하는 것도 가능하지만 틀별한 이유가 없다면 지양하길 권합니다.

### **[Note]**

*이후의 내용에서 GameAnvil에서 제공하는 Lettuce 클래스와 제품명인 "Lettuce"를 구분하기 위해 전자의 경우는 가능한 "Lettuce 클래스"라고 표기하고 일부 내용상 필요에 의해 그냥 "Lettuce"로 표기할 수도 있습니다. 이와 구분하기 위해 제품명은 전체 대문자 ***LETTUCE***로 표기합니다. 이 글에서 설명하는 LETTUCE는 GameAnvil에서의 사용 방법에 포커스를 두기 때문에 그 이상의 설명이 필요할 경우에는 [LETTUCE 공식 페이지](https://github.com/lettuce-io/lettuce-core)를 참고하세요.*

GameAnvil은 Redis 클라이언트로 LETTUCE의 사용을 권장합니다. GameAnvil에서 제공하는 Redis 래핑 API 또한 LETTUCE를 사용합니다. 참고로 LETTUCE는 비동기 Redis 클라이언트로서 대부분의 비동기 API는 CompletableFuture를 기반으로 합니다. 이는 곧 GameAnvil의 Async API를 이용해서 파이버 기반의 비동기화로 전환할 수 있음을 의미합니다.

GameAnvil에서 제공하는 Redis 래핑 API는 크게 3가지의 클래스인 Lettuce, RedisCluster 그리고 RedisSingle로 나뉩니다. Lettuce는 가장 일반적인 형태의 사용법을 제공하며 내부적으로 LETTUCE 객체를 관리하지 않는 static 클래스입니다. 그러므로 LETTUCE를 가장 일반적인 형태로 사용하고 싶을 경우에는 이 Lettuce 클래스가 가장 적합합니다. RedisCluster와 RedisSingle은 각각 Redis 클러스터와 스탠드얼론에 대응하기 위한 클래스로서 내부적으로 LETTUCE 객체들을 관리합니다.

### **2.1.Lettuce**

Lettuce 클래스는 파이버 단위의 처리를 위한 가장 핵심적인 static API들을 제공합니다. 내부적으로 Redis에 관한 그 어떤 상태도 보관하지 않으므로 별도의 객체를 만들 필요가 없이 바로 사용이 가능합니다. 만일 Lettuce 라이브러리에 대해 어느 정도 익숙하다면 Lettuce 클래스를 직접 사용하는 것이 가장 좋습니다.

```java
import com.nhn.gameanvil.async.redis.Lettuce;
```

다음의 3가지 주의 사항 외에는 기본적인 Lettuce 사용법을 그대로 유지할 수 있습니다.

- 첫째, 반드시 connect는 GameAnvil의 Lettuce.connect() 혹은 Lettuce.connectAsync()를 사용한다. 커넥션은 기본적으로 스레드를 블로킹하므로 이에 대한 파이버화 처리를 포함합니다.
- 둘째, shutdown 또한 커넥션과 동일한 이유로 Lettuce.shutdown()을 사용해야 합니다.
- 셋째, RedisFuture에 대한 대기는 반드시 Lettuce.awaitFuture()를 사용해서 파이버 블로킹화해야 합니다.

이런 Lettuce 클래스를 사용하여 Redis에 접속하는 코드는 아래와 같습니다.

```java
RedisURI clusterURI = RedisURI.Builder.redis(IP_ADDRESS, 7500).build();
clusterClient = RedisClusterClient.create(Arrays.asList(clusterURI));
clusterConnection = Lettuce.connect(RpsConfig.DB_THREAD_POOL, clusterClient);

if (clusterConnection.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### **2.2.RedisCluster**

```java
import com.nhn.gameanvil.async.redis.RedisCluster;
```

Redis Cluster에 대한 API를 래핑합니다. 기본적으로 앞서 설명한 Lettuce 와 사용법은 크게 다르지 않습니다. 하지만 이 클래스는 Lettuce 관련 객체들(e.g.RedisClusterClient, StatefulRedisClusterConnection 등)을 자체적으로 관리합니다. 이러한 Lettuce 객체들을 직접 관리하기 보다 RedisCluster를 통해 관리하고 싶다면 사용을 고려해보세요.

주의 사항은 Lettuce의 경우와 완전히 동일합니다. 아래는 RedisCluster를 이용해서 Redis에 접속하는 코드입니다.

```java
redisClient = RedisSingle.create("redis://IP_ADDRESS:6379");
redisAsyncCommands = Lettuce.connect(RpsConfig.DB_THREAD_POOL, redisClient).async();

if (redisClient.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### **2.3.RedisSingle**

```java
import com.nhn.gameanvil.async.redis.RedisSingle;
```

RedisCluster와 비교했을 때, 대상 Redis가 스탠드얼론이라는 차이점 밖에 없습니다.

주의 사항은 Lettuce의 경우와 완전히 동일합니다. 아래는 RedisSingle을 이용해서 Redis에 접속하는 코드입니다.

```java
redisCluster = new RedisCluster<>(IP_ADDRESS, 7500);redisCluster.connect(RpsConfig.DB_THREAD_POOL, StringCodec.UTF8);if (redisCluster.isConnected()) {    logger.warn("============= Connected to Redis using Lettuce =============");}
```

### **2.4.RedisFuture를 파이버에서 사용하기**

Lettuce, RedisCluster 그리고 RedisSingle은 모두 Lettuce 라이브러리가 지원하는 RedisFuture를 파이버 상에서 대기할 수 있는 API를 제공합니다. 내부 구현은 모두 엔진에서 제공하는 Async.awaitFuture()를 동일하게 사용하므로 혼용해도 무방합니다. 아래의 4가지 코드는 모두 동일한 코드입니다. GameAnvil의 파이버 상에서 RedisFuture에 대한 get()은 반드시 이 4가지 중 하나의 방법을 사용해야 합니다.

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

- **잘못된 사용법**: 직접 Future에 대한 대기를 할 경우 해당 Node(Thread)가 블로킹되므로 절대 아래와 같은 코드는 사용하면 안됩니다.

```java
try {
    RedisFuture future = clusterAsyncCommands.mget("testKey", getUserId()));
    future.get(); // 스레드 블로킹을 유발
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

### **2.5.set/get**

가장 기본이 되는 set과 get은 RedisCluser와 RedisSingle에서 기본 제공합니다.

- RedisCluster를 이용한 set/get 예제

```java
String setResult = redisCluster.set(key, value);
String getResult = redisCluster.get(key);
```

- RedisSingle을 이용한 set/get 예제

```java
String setResult = redisSingle.set(key, value);
String getResult = redisSingle.get(key);
```

- 직접 LETTUCE의 RedisAsyncCommands 객체를 사용한 예제

```java
RedisFuture<String> setFuture = redisAsyncCommands.set(key, value);RedisFuture<String> getFuture = redisAsyncCommands.get(key);String setResult = Async.awaitFuture(setFuture);String getResult = Async.awaitFuture(getFuture);
```

### **2.6.본격적인 LETTUCE 비동기 처리**

Redis가 제공하는 다양한 커맨드들은 LETTUCE의 Commands 객체를 통해 사용 가능합니다. 기본적으로 LETTUCE는 Sync방식의 Commands 객체과 Async방식의 Commands 객체를 제공하는데 GameAnvil은 그 중 Aync방식의 사용을 권장합니다. 기본적으로 AsyncCommands는 Redis Cluster인 경우와 StandAlone인 경우에 대해 각각 아래와 같습니다.

- RedisAdvancedClusterAsyncCommands
- RedisAsyncCommands

아래의 예제들은 이런 AsyncCommands 객체를 이용하여 mget을 수행하는 예제들입니다. LETTUCE의 비동기 처리는 기본적으로 RedisFuture를 사용하고 이 RedisFuture는 CompletableFuture입니다. CompletableFuture에 대한 자세한 내용은 [Java 공식 레퍼런스](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)에서 확인 가능합니다. 참고로 아래의 예제들은 LETTUCE에 대한 비동기 처리의 극히 일부 방식만을 보여주고 있으므로 그대로 사용하기 보다는 개발중인 코드에 알맞게 작성하세요. 완벽한 비동기 코드의 제어를 위해서는 반드시 [LETTUCE](https://github.com/lettuce-io/lettuce-core)와 [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)에 대한 내용을 숙지해야 합니다.

### **[Note]**

thenApply()와 thenAccept() 등은 임의의 외부 스레드에서 호출되므로 Node에서 관리하는 내부 리소스에 대한 접근을 하거나 리소스에 대한 Lock을 사용하면 안됩니다.

- 예제1> key1과 key2에 대한 값을 비동기로 획득

```java
Lettuce.awaitFuture(asyncCommands.mget("key1", "key2"));
```

- 예제2> 이 후의 코드 흐름과 상관없는 경우 future chain으로 외부 스레드에 처리를 위임 (즉, mget으로 값 획득을 완료할 때까지 대기할 필요가 없을 경우)

```java
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

future.thenApplyAsync(r -> {
    Map<String, String> map = new HashMap<>();
    for (KeyValue<String, String> kv : r)
        map.put(kv.getKey(), kv.getValue());
    return map;
}).thenAccept(r -> {
    for (Entry<String, String> entry : r.entrySet())
        logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
});
```

- 예제3> 이후의 코드 흐름과 상관있는 경우에 해당 future를 대기한 후 처리

```java
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

CompletionStage<Map<String, String>> cs = future.thenApplyAsync(r -> {
    Map<String, String> map = new HashMap<>();
    for (KeyValue<String, String> kv : r)
        map.put(kv.getKey(), kv.getValue());
    return map;
});

// do something here

try {
    // 파이버 상에서 해당 future를 대기하기 위해 Lettuce.awaitFuture()를 사용해야 함을 명심하세요
    Map<String, String> map = Lettuce.awaitFuture(cs);

    for (Entry<String, String> entry : map.entrySet())
        logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin()", e);
}
```

## 3. 비동기 HttpReqeust & HttpResponse 사용법

Http 처리에 관한 부분도 Redis와 마찬가지로 GameAnvil에서 기본적인 API와 가이드라인을 제공합니다. 물론 다른 종류의 Http 사용법 역시 선택이 가능하지만 특별한 이유가 없다면 지양하길 권합니다. GameAnvil은 비동기 기반의 Http 사용을 위해 내부적으로 [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)를 사용합니다. 다음에서 설명할 API와 그 사용 범위를 넘는 경우에는 저희가 제공하는 API 보다 직접 [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)를 사용하길 권합니다. LETTUCE와 마찬가지로 AsyncHttpClient도 내부적으로 CompletableFuture를 사용하므로 future에 대한 대기를 Async.awaitFuture()를 이용해서 파이버화 해주기만 하면 나머지는 일반 스레드 상에서의 사용법과 완전히 동일합니다.

```java
Async.awaitFuture(future.get()); // 파이버 상에서 해당 future를 대기합니다.
```

GameAnvil에서 제공하는 Http API는 요청과 응답을 위한 HttpRequest, HttpResponse 클래스 그리고 결과에 대한 일반적인 처리를 위한 HttpResultTemplate 클래스로 이루어집니다. 이 클래스들을 이용하면 간단하고 직관적으로 Http 요청과 응답을 처리할 수 있으며 그 결과를 원하는 형태로 취할 수도 있습니다. 또한 모든 코드는 비동기이므로 특별한 처리가 필요없습니다. 다음은 이를 사용한 예제 코드들입니다.

- 예제1> 가장 기본적인 사용법 내부적으로 파이버 단위의 future 처리를 알아서 해주므로 가장 직관적인 방식입니다. 특별한 이유가 없다면 이러한 기본적인 사용법만으로도 충분합니다.

```java
HttpRequest request = new HttpRequest(URL);
HttpResponse response = request.GET();
```

- 예제2> future 기반의 비동기 방식 HTTP 요청과 응답 대기 사이에 다른 작업을 하고 싶을 경우 아래와 같이 future를 직접 이용할 수 있습니다.

```java
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

// Do something here

HttpResponse response = new HttpResponse(Async.awaitFuture(future, 10000, TimeUnit.MILLISECONDS));
```

- 예제3> HTTP 요청 header 구성 아래의 예제와 같이 AsyncHttpClient는 다양한 API를 제공합니다. AsyncHttpClient에 대한 자세한 사용법은 [공식 페이지](https://github.com/AsyncHttpClient/async-http-client)를 참고하세요.

```java
HttpRequest request = new HttpRequest(url);
request.getBuilder()
    .addHeader("if-none-check-node", "false")
    .setRequestTimeout(timeout);
    .addQueryParam("serviceId", serviceId);

HttpResponse httpResponse = request.GET();
```

- 예제4> 이후의 코드 흐름과 상관없는 경우 future chain으로 외부 스레드에 처리를 위임 (Lettuce의 경우와 동일한 방식)

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

- 예제5> 이후의 코드 흐름과 상관있는 경우에 해당 future를 대기한 후 처리

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
    // 파이버 상에서 해당 future를 대기하기 위해 Async.awaitFuture()를 사용해야 함을 명심하자.
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

## 4. RDBMS 비동기 처리

RDBMS에 대한 쿼리는 일반적으로 블로킹입니다. 이런 블로킹 쿼리를 GameAnvil 상에서 처리하는 방법은 앞서 살펴보았던 다른 Async 사용법과 크게 다르지 않습니다. 어떤 종류의 RDBMS를 사용하던 SQL 쿼리에 대한 코드는 동일한 방법으로 구현할 수 있습니다. 또한 엔진 사용자는 DB 접근을 위해 자유롭게 SQL Mapper나 ORM 등을 선택할 수 있습니다.

반면에 이러한 쿼리를 기본적으로 넌블러킹 방식의 비동기 처리를 해주는 [MySQL X DevAPI](https://dev.mysql.com/doc/x-devapi-userguide/en/)나 [jasync-sql](https://github.com/jasync-sql/jasync-sql)과 같은 비동기 DB 드라이버가 있습니다. GameAnvil은 이 두가지 모두를 지원합니다. 하지만 MySQL X DevAPI는 몇 가지 결함이 발견되어 베타 버전의 독립된 라이브러리 형태로만 제공합니다. 즉, GameAnvil은 jasync-sql을 기반으로 비동기 쿼리를 완벽하게 지원합니다. 특별한 이유가 없다면 사용자도 jasync-sql을 사용하길 제안합니다.

### 4.1.블러킹 쿼리

블러킹 쿼리는 호출 스레드를 블러킹시키므로 반드시 비동기 처리를 해주어야 합니다. 이런 블러킹 쿼리에 대한 비동기 처리는 크게 2가지로 나눌 수 있습니다. 쿼리의 결과가 필요한 경우와 그렇지 않은 경우입니다. 이 두 경우는 쿼리의 결과 유무 차이만 있을 뿐 전체 쿼리 수행이 완료될 때까지 해당 파이버가 대기하는 것은 동일합니다. 즉, 비동기로 요청한 쿼리가 완료된 후 다음 코드로 진행되므로 엔진 사용자는 일반적인 블로킹 코드를 작성하듯이 구현할 수 있습니다.

### Note

*DB에 대한 쿼리를 구현하는 과정에서 가장 중요하지만 흔히 놓치는 부분은 DB에 대한 CP(ConnectionPool) 크기와 이를 비동기로 처리할 TP(ThreadPool)의 개수에 대한 설정과 이들 사이의 관계에 대한 이해입니다. 일반적으로 이들 두 수치는 처리할 쿼리의 양을 고려하여 동일한 값으로 설정하거나 TP를 CP보다 조금 더 넉넉하게 설정하면 됩니다. 참고로 GameAnvil를 이용한 대규모 성능 테스트 결과, 서버 프로세스 하나 당 6000~8000명 처리 기준 TP와 CP 250개 설정이 가장 좋은 결과를 보여주었습니다. 이는 어디까지나 쿼리 복잡도와 빈도 등 복합적인 요소를 고려하여 가능한 많은 테스트를 거쳐 최적의 값을 찾는 것이 최선입니다.*

첫째, 쿼리의 결과를 획득하고자 할 경우에는 다음의 예제와 같이 Async 클래스의 callBlocking API를 사용합니다. callBlocking은 파이버 상에서 임의의 블로킹 호출을 수행한 후 결과를 반환합니다.

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

이때, 비동기 처리를 위한 스레드풀은 Bootstrap 단계에서 미리 생성해둘 수 있습니다.

```
bootstrap.createExecutorService("MyThreadPool", 250);
```

혹은 엔진 사용자가 필요에 따라 직접 생성한 외부 스레드풀을 사용할 수도 있습니다.

```
bootstrap.createExecutorService(myExecutorService, 250);
```

둘째, 쿼리의 결과가 필요 없는 경우에는 다음의 예제와 같이 Async 클래스의 runBlocking API를 사용합니다. runBlocking은 파이버 상에서 임의의 블로킹 호출을 수행합니다.

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

이 경우도 마찬가지로 임의의 스레드풀을 runBlocking API에 매개변수로 전달할 수 있습니다.

### 4.2 논블러킹 비동기 쿼리

앞서 설명하였듯이 GameAnvil은 jasync-sql을 기본 비동기 DB 드라이버로 사용합니다. 그 사용법은 매우 직관적이고 쉽기 때문에 기존의 블러킹 쿼리보다 코드 생산성이 올라가며 그 성능 또한 훨씬 우수합니다.  우선 GameAnvil에서 제공하는 Jasync-sql를 사용하기 위해서는 다음과 같은 import문을 추가합니다.

```java
import com.nhn.gameanvil.async.db.JAsyncSql
```

JasyncSql 클래스는 비동기 쿼리를 위한 기능을 GameAnvil 파이버 상에서 유연하게 동작하도록 지원합니다. 일반적으로 특별한 이유가 없다면 노드 당 하나의 JasyncSql 객체를 만들어두고 사용하는 것이 가장 좋습니다. 그리고 비동기 쿼리를 사용할 때는 블러킹 방식과 달리 사용자가 별도의 스레드 풀이나 커넥션 풀을 생성할 필요가 없습니다.

다음은 JasyncSql 객체를 생성하는 코드입니다. 인자 중 64개의  최대 활성 커넥션 수는 사용 용도와 쿼리 빈도에 맞춰 최적화 할 수 있습니다.

```java
JAsyncSql jasyncSql = new JAsyncSql(new com.github.jasync.sql.db.Configuration(
                                    "gameanvil",
                                    "127.0.0.1",
                                    13306,
                                    "%gameanvil1",
                                    "GameDB_1"), 64));  // 64개의 최대 활성 커넥션
```

JasyncSql 객체를 통해 비동기 쿼리를 요청한 후 CompletableFuture를 반환 받을 수 있습니다. 일반적인 future 기반의 비동기 코드입니다.

```java
CompletableFuture<QueryResult> future = jasyncSql.executeAsync("SELECT * FROM UserInfo");

... // do something others

Async.awaitFuture(future); // 해당 파이버 상에서 비동기로 future를 대기
```

또한 쿼리 결과를 바로 획득하기 위해 해당 파이버에 대한 대기를 내포하는 동기화 API도 제공합니다.

```java
QueryResult result = jasyncSql.execute("SELECT * FROM UserInfo");
```

이 코드는 앞서 살펴본 future 기반의 비동기 코드를 하나로 함축한 것과 동일합니다. 이 모든 코드는 스레드 단위로 비동기화를 하는 것이 아니라 파이버 단위로 동작합니다.

### 4.3 넌블러킹 비동기 쿼리 vs 블러킹 쿼리

이 두 방식 사이에는 그 사용법과 코드 생산성 차이 뿐만 아니라 성능도 확연하게 차이가 납니다.  동일한 환경에서 두 가지 쿼리 방식의 성능을 측정한 결과는 아래의 그림과 같습니다.

![](https://static.toastoven.net/prod_gameanvil/images/mysql-async-performance.png)

jasync-sql 기반의 비동기 쿼리가 가장 성능이 높습니다. 이는 Mapper나 ORM을 사용한 블러킹 쿼리에 비해 약 2배의 성능 향상을 보여줍니다.  그런 측면에서 GameAnvil은 사용자들로 하여금 특별한 이유가 없다면 이러한 비동기 쿼리의 사용을 지향할 것을 제안합니다.
