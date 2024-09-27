---
title: Querying
layout: home
nav_order: 5
---

{: .important }
> This is a copy of the Java documentation for libGDX. It is not finished being ported.

The `Application` interface provides various methods to query properties of the run-time environment.

### Getting the Application Type
Sometimes it is necessary to implement certain functionality differently depending on the platform it is running on. The `Application.getType()` method returns the platform the application is currently running on:

```csharp
switch (Gdx.app.getType()) 
{
    case Desktop:
        // desktop specific code
        break;
    default:
        // Other platforms specific code
}
```

### Memory Consumption
For debugging and profiling purposes it is often necessary to know the memory consumption, for both the Java heap and the native heap:

```csharp
long javaHeap = Gdx.app.getJavaHeap();
long nativeHeap = Gdx.app.getNativeHeap();
```

Both methods return the number of bytes currently in use on the respective heap.