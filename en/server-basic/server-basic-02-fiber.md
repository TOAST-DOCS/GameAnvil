## Game > GameAnvil > Server Concept Description > Fiber



## Fiber

Fiber is a kind of lightweight user thread, which is the basic flow unit of GameAnvil server code. The node's single thread earlier is again divided into multiple fibers to effectively handle large numbers of sessions, users, and rooms at the same time. In other words, GameAnvil supports fiber-based continuation.

Multiple fibers are scheduled on thread pool (Executor). At this time, if you fix the thread pool size to 1, it immediately becomes a model for the GameAnvil node. In other words, the node is an annotation scheduler for processing multiple fibers simultaneously. This is illustrated as follows.

![image.png](https://static.toastoven.net/prod_gameanvil/images/FiberConcept.png)

The advantage of using fiber is that it allows for sequential code writing. Server code becomes very similar to writing regular blocking code. There is no need to worry about separate callback processing or notification of completion. In addition to the advantages of these fibers, GameAnvil users don't have to worry too much about these fiber units. All the fibers are managed by the GameAnvil engine end, so you can develop them as if you were writing a typical single threading code.

![fiber-context-switching.png](https://static.toastoven.net/prod_gameanvil/images/fiber-context-switching.png)

GameAnvil server code is based on asynchronous processing. For this purpose, [Asynchronous Support API](../server-impl/server-impl-10-async) is provided. Blocking calls on any fiber using these asynchronous APIs result in only that fiber being suspended. This will be covered in more detail below this document.



## Asynchronous fiber-based processing (Asynchronous on fibers)

Codes should be able to be processed asynchronously on a fiber. In other words, if one fiber makes an I/O call that takes any time, that fiber can yield execution rights to the other fiber until the call is complete. Even a thread blocking call provides an API that allows it to be asynchronized by switching to a fiber blocking call. To put it another way, thread blocking calls must be handled using [Asynchronous Support API](../server-impl/server-impl-10-async), if you break it and call directly, the thread will be blocked, which will result in the blocking of all fibers on the thread, resulting in outage of entire node.

```
    void someProblematicMethod() {  
  
        someThreadBlockingCall(); // Not just that fibre, but the whole thread is blocked! 
  
    }
```

A detailed description of this is covered in detail in [Asynchronous Support](../server-impl/server-impl-10-async) in the implementation method.

