---
title: Logging
layout: home
nav_order: 6
---
The `IApplication` interface provides simple logging facilities that give granular control over which messages should be logged.

A message can be a normal **info message**, an **error message** with an optional exception or a **debug message**:

```csharp
GDX.App.Log("MyTag", "my informative message");
GDX.App.Error("MyTag", "my error message", exception);
GDX.App.Debug("MyTag", "my debug message");
```

On desktop, the messages are logged to the console.

Logging can be limited to a specific logging level:

```csharp
GDX.App.SetLogLevel(logLevel);
```

Where `logLevel` can be one of the following values:

  * `Application.LOG_DEBUG`: logs all messages.
  * `Application.LOG_INFO`: logs error and normal messages.
  * `Application.LOG_ERROR`: logs only error messages.
  * `Application.LOG_NONE`: mutes all logging.