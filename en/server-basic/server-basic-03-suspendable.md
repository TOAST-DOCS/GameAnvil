## Game > GameAnvil > Server Concept Description > Suspendable



## Suspendable

Below is a couple of things to be avoided when writing Fiber-based code. Users do not have to concern with the Fiber unit, the things below must be avoided.

- **Suspend** means that Fiber code yielded code execution to another Fiber and enter into the waiting status. This can be expressed as Fiber blocking.
- Methods that can be Suspended must have the **throws SuspendExecution** exception signature added to them. This is a promise with the engine that the method contains blocking code used to stop Fiber.

```
void someSuspendableMethod() throws SuspendExecution {

// some fiber-blocking call can be here

}
```

- If, for particular reasons, you cannot specify throws SuspendExecution exceptions signatures, you can use **@Suspendable** annotations. Otherwise, be sure to use throwSuspendExecution exception signatures first.

```
@Suspendable
void someSuspendableMethod() {

// some blocking call can be here

}
```

- Naturally, methods that call these suspendable methods are also suspendable. For example, if there is a method called someCaller() that calls someSuspendableMethod(), someCaller() must also be declared suspendable.

```
void someCaller() throws SuspendExecution {

    someSuspendableMethod();

}
```

- SuspendExecution is a special technique used by the Quasar library used by engine to process fibers. This is a kind of bypass technique used by Quasar library because they are not yet the standard for Java, and it is not an actual exception. Therefore, you must never catch SuspendExecution exceptions. This is a common error, so please be careful.

```
// !! Wrong usage!! 

  void someCaller() {  

    try {  

        someSuspendableMethod();  

    } catch (SuspendExecution e) {  
        // Never explicitly catch SuspendExecution!

    } 
}
```

- However, catching entire exceptions is no problem. It is handled automatically.

```
void someCaller() throws SuspendExeuction {  
  
    try {  
  
        someSuspendableMethod();  
  
    } catch (Exception e) {  
        // No problem  
    }  
}
```

