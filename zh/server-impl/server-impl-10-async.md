## Game > GameAnvil > Server Development Guide > Asynchronous Support

## Asynchronous Support

GameAnvil provides asynchronous support for the following purposes

![async-goal.png](https://static.toastoven.net/prod_gameanvil/images/async-goal.png)

For asynchronous processing on the fiber, GameAnvil provides the Async class. Using the Async class, you can fiberize all common blocking/non-blocking calls using the following import statement.

```java
import com.nhn.gameanvil.async.Async;
```



The API for invocation is broadly divided into call and run, which are used when there is a return value and when there isn't, respectively. Otherwise, it converts thread-based futures to fiber-based ones. The usage of each is covered in more detail in the next part of the documentation.

### **Handling Blocking Calls

A typical blocking call is thread blocking, which means that it blocks not only the fiber that the current code is doing, but also the thread that is scheduling this fiber. You should never use blocking calls directly, as this means that your node will freeze. GameAnvil provides an Async API that converts these thread blocking calls into fiber blocking. This API uses an external executor to handle those blocking calls and then re-fiberizes the code flow after completion. You can use either runBlocking() or callBlocking(), depending on whether you want a return value or not. Also, as described in the basic concepts, these fiber blocking APIs are suspendable, so API call methods must specify the SuspendExecution exception signature.

```java
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() throws SuspendExecution {

    Async.runBlocking(executor, runnable); // Convert thread blocking calls to fiber blocking calls

}
} import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() throws SuspendExecution {

    return Async.callBlocking(executor, callable); // Convert a thread blocking call to a fiber blocking call

}
```

### Future Handling

Waiting on Future causes thread blocking. For example, the code below blocks the calling thread.

```java
Future<SomeObject> future = someAsyncJob();

SomeObject ret = future.get(); // cause thread blocking
```

GameAnvil provides APIs to switch waiting for these futures from thread-blocking to fiber-blocking. However, these APIs only support Java's CompletableFuture and Guava's ListenableFuture. Fortunately, most libraries support asynchrony based on these two futures, so you should be able to adapt without too much trouble. The code below is an example of using these Async APIs to fiber-block waiting for a future.

```java
// lettuce future, jdk CompletableFuture, etc. 
CompletionStage<SomeObject> future = someAsyncJob();

SomeObject ret = Async.awaitFuture(future); // block only that fiber
```

### Delegate Blocking Handling

The process for blocking calls is covered earlier. The runBlocking() or callBlocking () of Async API is used only when continuing the execution flow of the corresponding Fiber after blocking process is finished. However, if the user does not need to concern after delegating blocking calls to an external thread, the execution flow can be continued. In this case, use the API below. This API does not wait for the result of blocking call and cannot be suspended.

```java
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() { // NOT suspendable
    Async.exec(executor, runnable); // This fiber is not blocked because we delegated the blocking call to an external thread.
}
```

## Asynchronous Redis Support

Previously, it was entirely GameAnvil user's responsibility to determine which type of Redis client to be used. However, it is learned that the type and complexity of the issues related to Redis increase proportionate to the difference in the type and usage of the Redis client selected by the user. To prevent this, we decided to include the information on how to use the Redis client to the basic guideline of GameAnvil. This guideline aims to consolidate the Redis clients supported by GameAnvil by turning the Redis clients and the basic usage into API. Of course, it is possible to use a Redis client other than the provided API, this should be avoided if possible.

> [Note]
>
> In the following, to distinguish between the Lettuce classes provided by GameAnvil and the product name "Lettuce", the former will be referred to as "Lettuce classes" whenever possible, and simply as "Lettuce" in some cases. To distinguish between the two, the product name is written in all capital letters "LETTUCE". The [LETTUCE](https://github.com/lettuce-io/lettuce-core)described in this article focuses on how to use it in GameAnvil, so if you need more information, please refer to the [official LETTUCE page](https://github.com/lettuce-io/lettuce-core).*


> GameAnvil recommends using LETTUCE as the Redis client. The Redis wrapping API provided by GameAnvil uses LETTUCE as well. For reference, LETTUCE is as an asynchronous Redis client and the most of asynchronous APIs are based on CompletableFuture. This means that GameAnvil's Async API can be converted into Fiber-based asynchronization.

The Redis wrapping API provided by GameAnvil can be categorized into the Lettuce, RedisCluster, and RedisSingle classes. Lettuce provides the most common usage and it is a static class that does not internally manage the LETTUCE object. Therefore, if the user wants to use LETTUCE in the most common form, this Lettuce class is the most proper. RedisCluster and RedisSingle are the classes for responding to the Redis cluster and standalone and they internally manage the LETTUCE objects.

### Lettuce

The Lettuce class provides the most essential static APIs used to process Fiber-units. Internally, no status related to Redis are stored, it can be used without having to create a separate object. If the user is familiar with the Lettuce library, it is best to use the Lettuce class directly.

```java
import com.nhn.gameanvil.async.redis.Lettuce;
```

The basic Lettuce usage can be maintained except the following three cautions.

- First, connect must use the Lettuce.connect() or Lettuce.connectAsync() of GameAnvil. As connection basically blocks threads, it includes the Fiber process.
- Second, shutdown must use Lettuce.shutdown as well for the same reason with connection.
- Third, the wait for RedisFuture must use Lettuce.awaitFuture() to block Fiber.

The code that is used to connect to Redis using the Lettuce class.

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

Wraps the API for Redis Cluster. The usage is basically not different from the usage for Lettuce. However, this class manages the objects related to Lettuce (e.g. RedisClusterClient, StatefulRedisClusterConnection etc.)on its own. Consider using this API if the user wants to manage these Lettuce objects through RedisCluster.

The cautions to be observed are the same with Lettuce. Below is the code that is used to connect to Redis using RedisCluster.

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

When compared to RedisCluster, the only difference is that the target Redis is standalone.

The cautions to be observed are the same with Lettuce. Below is the code that is used to connect to Redis using RedisCluster.

```java
redisCluster = new RedisCluster<>(IP_ADDRESS, 7500);
redisCluster.connect(RpsConfig.DB_THREAD_POOL, StringCodec.UTF8);
if (redisCluster.isConnected()) {
    logger.warn("============= Connected to Redis using Lettuce =============");
}
```

### Using RedisFuture on Fiber

Lettuce, RedisCluster, and RedisSingle provide the API that can make the RedisFuture supported by the Lettuce library wait on Fiber. As the internal implementation uses the Async.awaitFuture() provided by the engine, they can be interchangeably used. The four code below are the same code. The get() of the RedisFuture on the Fiber of GameAnvil must use one of the four methods.

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

- **Incorrect usage: If directly wait for Future, the Node (Thread) is blocked, so never use the code below.

```java
try {
    RedisFuture future = clusterAsyncCommands.mget("testKey", getUserId()));
    future.get(); // Causes thread blocking
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

### set/get

Set and get, the most basic components, are provided by RedisCluster and RedisSingle by default.

- Set/get examples with RedisCluster

```java
String setResult = redisCluster.set(key, value);
String getResult = redisCluster.get(key);
```

- Set/get examples with RedisSingle

```java
String setResult = redisSingle.set(key, value);
String getResult = redisSingle.get(key);
```

- Example using RedisAsyncCommands object in LETTUCE directly

```java
RedisFuture<String> setFuture = redisAsyncCommands.set(key, value);RedisFuture<String> getFuture = redisAsyncCommands.get(key);String setResult = Async.awaitFuture(setFuture);String getResult = Async.awaitFuture(getFuture);
```

### Asynchronous Process of LETTUCE

The various commands provided by Redis can be used through the Commands object of LETTUCE. Basically, LETTUCE provides the Commands object in Sync and the Commands object in Async. GameAnvil recommends using the Async method. Basically, AsyncCommands for Redis Cluster and StandAlone are as below:

- RedisAdvancedClusterAsyncCommands
- RedisAsyncCommands

The examples below are examples of using these AsyncCommands objects to perform mget. Asynchronous processing in LETTUCE uses RedisFuture by default, and this RedisFuture is a CompletableFuture. You can learn more about CompletableFuture in the [official Java reference](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html). Note that the examples below show only a small subset of asynchronous handling for LETTUCE, so don't use them verbatim, but tailor them to the code you're developing. For complete control of asynchronous code, you must familiarize yourself with [LETTUCE and](https://github.com/lettuce-io/lettuce-core) [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html).

> [Note]
>
> thenApply(), thenAccept() and others are called by arbitrary external thread, do not access the internal resources managed by Node or use Lock on such resources.

- Example 1> Obtain values for key1 and key2 asynchronously
>
> ```java
> Lettuce.awaitFuture(asyncCommands.mget("key1", "key2"));
> ```
> 
> - Example 2> Delegate processing to an external thread with a future chain when it is irrelevant to the subsequent code flow (i.e., when you don't need to wait for mget to finish acquiring the value)
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
> - Example 3> Wait for that future and process it if it's relevant to the later code flow
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
>    // Remember to use Lettuce.awaitFuture() to wait for that future on the fiber
>     Map<String, String> map = Lettuce.awaitFuture(cs);
>
>     for (Entry<String, String> entry : map.entrySet())
>         logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
> } catch (TimeoutException e) {
>     logger.error("GameUser::onLogin()", e);
> }
> ```

## How to Use Asynchronous HttpRequest & HttpResponse

As with Redis, GameAnvil provides a basic API and guidelines for handling HTTP. You can of course choose to use other kinds of Http, but we recommend avoiding them unless you have a specific reason to do so. GameAnvil uses [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)internally for asynchronous Http usage. We recommend that you use [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)directly rather than the API we provide if you are beyond the scope of the API and its usage. Like LETTUCE, AsyncHttpClient uses CompletableFuture internally, so all you need to do is fiber the wait for the future with Async.awaitFuture() and the rest is exactly the same as using it on a normal thread.

```java
Async.awaitFuture(future.get()); // Wait for that future on the fiber.
```

The Http API provided by GameAnvil consists of the HttpRequest and HttpResponse classes for requests and responses, and the HttpResultTemplate class for general handling of results. These classes allow you to handle Http requests and responses in a simple and intuitive way, and the results can take any form you want. Also, all code is asynchronous, so no special handling is required. Here is some example code that uses them.

### Using HttpReqeust & HttpResponse
The HttpRequest library has been used in GameAnvil for a long time, but the related libraries have not been updated, causing some issues when using it. To address these issues, the HttpRequest2 class exists, which has changed the internal Http library, and if you are experiencing issues while using HttpRequest, we recommend changing to the HttpRequest2 configuration. In future releases, if HttpReuqest2 does not cause issues, the existing HttpRequest may be removed.

- Example 1> The most basic usage This is the most intuitive, as it takes care of handling future on a per-fiber basis internally. Unless you have a specific reason to do otherwise, this basic usage should suffice.

```java
HttpRequest request = new HttpRequest(URL);
HttpResponse response = request.GET();
```

- Example 2> Asynchronous HTTP requests based on future If you want to do something else between requesting and waiting for a response, you can use future directly as shown below.

```java
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

// Do something here

HttpResponse response = new HttpResponse(Async.awaitFuture(future, 10000, TimeUnit.MILLISECONDS));
```

- Example 3> Configuring HTTP request headers As shown in the example below, AsyncHttpClient provides a variety of APIs. For more information on how to use AsyncHttpClient, see the [official page](https://github.com/AsyncHttpClient/async-http-client).

```java
HttpRequest request = new HttpRequest(url);
request.getBuilder()
    .addHeader("if-none-check-node", "false")
    .setRequestTimeout(timeout);
    .addQueryParam("serviceId", serviceId);

HttpResponse httpResponse = request.GET();
```

- Example 4> Delegate processing to an external thread with a future chain if it is irrelevant to the subsequent flow of code (same way as in the case of Lettuce)

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

- Example 5> Wait for that future and process it if it's relevant to the later code flow

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
    // Remember to use Async.awaitFuture() to wait for that future on the fiber.
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

### Using HttpReqeust2

The above HttpRequest library has been used in GameAnvil for a long time, but the related libraries have not been updated, causing some issues when using it. To solve these issues, the HttpRequest2 class exists, which changes the internal Http library, and if you encounter issues while using HttpRequest, we recommend changing to the HttpRequest2 configuration. In future releases, if HttpReuqest2 does not cause issues, the existing HttpRequest may be removed.

- Example 1> Wait for that future and process it if it's relevant to the subsequent code flow

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

## RDBMS asynchronous processing

The query for RDBMS is generally blocking. The way to process this blocking query on GameAnvil is not that different from the usage of other Asyncs. Regardless of the type of RDBMS used, the code for the SQL query can be implemented in the same way. And engine users can freely choose from SQL Mapper, ORM, or others to access DB.

On the other hand, there are asynchronous DB drivers such as the [MySQL X](https://dev.mysql.com/doc/x-devapi-userguide/en/)DevAPI or [jasync-sql](https://github.com/jasync-sql/jasync-sql)that provide native, non-blocking asynchronous processing of these queries. GameAnvil supports both of these. However, MySQL X DevAPI has been found to have some flaws and is only available as a standalone library in beta. That said, GameAnvil has full support for asynchronous queries based on jasync-sql, and we suggest that you use jasync-sql as well, if for no other reason.

### Blocking queries

Because blocking queries block the calling thread, they must be handled asynchronously. Asynchronous handling of these blocking queries can be divided into two main categories. The only difference between these two cases is whether the result of the query is needed or not, and the fiber waits until the entire query execution is complete. This means that the asynchronously requested query completes and then proceeds to the next code, so engine users can implement it as if they were writing normal blocking code.

> [Note]
> 
> The most important thing while implementing the query for DB but often overlooked is the size of CP (ConnectionPool) for DB, the number of TPs (ThreadPool) to asynchronously process them, and understanding of the relationship between them. These two values are usually set the same, considering the amount of queries to be processed or set TP slightly larger than CP. For your information, in a large scale performance test using GameAnvil, setting TP for 6000~8000 people and CP for 250 per server process showed the optimal result. It is important to find the optimal value by running as many tests as possible considering the complex elements such as the complexity and frequency of query.

> First, when the user wants to obtain the query result, they need to use the callBlocking API of the Async class as shown in the example below. callBlocking calls an arbitrary blocking and returns the result.

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

At this time, the thread pool for processing asynchronously can be created during the Bootstrap step.

```
gameAnvilServer.createExecutorService("MyThreadPool", 250);
```

Or if it is necessary, engine users may use the external thread pool directly created.

```
gameAnvilServer.createExecutorService(myExecutorService, 250);
```

Second, when the user does not need to obtain the query result, they need to use the runBlocking API of the Async class as shown in the example below. runBlocking calls arbitrary blocking on Fiber.

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

In this case, the arbitrary thread pool can be passed to runBlocking API as a parameter.

### Non-Blocking Asynchronous Queries

As mentioned earlier, GameAnvil uses jasync-sql as its default asynchronous DB driver, which is very intuitive and easy to use, resulting in more productive code and much better performance than traditional blocking queries.  First of all, to use the Jasync-sql provided by GameAnvil, add the following import statement

```java
import com.nhn.gameanvil.async.db.JAsyncSql
```

The JasyncSql class provides functionality for asynchronous queries to work flexibly over the GameAnvil fiber. In general, it is best to create and use one JasyncSql object per node unless there is a specific reason to do so. And unlike blocking, asynchronous queries do not require the user to create a separate thread pool or connection pool when using them.

Here is the code to create a JasyncSql object. The maximum number of active connections of 64 in the argument can be optimized for your usage and query frequency.

```java
JAsyncSql jasyncSql = new JAsyncSql(new com.github.jasync.sql.db.Configuration(
                                    "gameanvil",
                                    "127.0.0.1",
                                    13306,
                                    "%gameanvil1",
                                    "GameDB_1"), 64)); // 64 maximum active connections
```

You can request an asynchronous query through a JasyncSql object and get a CompletableFuture returned. This is typical future-based asynchronous code.

```java
CompletableFuture<QueryResult> future = jasyncSql.executeAsync("SELECT * FROM UserInfo");

... // do something else

Async.awaitFuture(future); // wait for future asynchronously on that fiber
```

It also provides a synchronization API that implicitly waits on that fiber to get the query results immediately.

```java
QueryResult result = jasyncSql.execute("SELECT * FROM UserInfo");
```

This code is the same as the future-based asynchronous code we saw earlier, all rolled into one. All of this code works on a per-fiber basis, rather than asynchronizing on a per-thread basis.

### No-blocking asynchronous queries vs. blocking queries

In addition to the usage and code productivity differences between the two methods, there is a significant difference in performance.  We measured the performance of the two query methods in the same environment, and the results are shown in the figure below.

![](https://static.toastoven.net/prod_gameanvil/images/mysql-async-performance.png)

Asynchronous queries based on jasync-sql are the most performant, showing about a 2x performance improvement over blocking queries using Mapper or the ORM. In that respect, GameAnvil suggests that users should strive to use these asynchronous queries unless there is a specific reason not to.
