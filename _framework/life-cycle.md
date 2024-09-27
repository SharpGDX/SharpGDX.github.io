---
title: Life Cycle
layout: home
nav_order: 2
---

{: .important }
> This is a copy of the Java documentation for libGDX. It is not finished being ported.

A SharpGDX application has a well defined life-cycle, governing the states of an application, like creating, pausing and resuming, rendering and disposing the application.

## IApplicationListener
An application developer hooks into these life-cycle events by implementing the [ApplicationListener](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/ApplicationListener.html) interface and passing an instance of that implementation to the `Application` implementation of a specific back-end (see [The Application Framework](/wiki/app/the-application-framework)). From there on, the `Application` will call the `ApplicationListener` every time an application level event occurs. A bare-bones `ApplicationListener` implementation may look like this:

```csharp
public class MyGame : IApplicationListener
{
	public void Create() 
	{
	}
	
	public void Render() 
	{
	}

	public void Resize(int width, int height)
	{
	}
	
	public void Pause() 
	{
	}
	
	public void Resume() 
	{
	}
	
	public void Dispose() 
	{
	}
}
```

One can also derive from the [ApplicationAdapter](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/ApplicationAdapter.html) class, which provides empty default implementations for those methods.

Once passed to the `IApplication`, the `IApplicationListener` methods will be called as follows:

| Method signature | Description |
| ---------------- | ----------- |
| `Create()` | Method called once when the application is created.|
| `Resize(int width, int height)` | This method is called every time the game screen is re-sized and the game is not in the paused state. It is also called once just after the `create()` method.<br/> The parameters are the new width and height the screen has been resized to in pixels.|
| `Render()` | Method called by the game loop from the application every time rendering should be performed. Game logic updates are usually also performed in this method.|
| `Pause()` | On Android this method is called when the Home button is pressed or an incoming call is received. On desktop this is called when the window is minimized and just before `dispose()` when exiting the application.<br/> A good place to save the game state.|
| `Resume()` | This method is called on Android, when the application resumes from a paused state, and on desktop when unminimized.|
| `Dispose()` | Called when the application is destroyed. It is preceded by a call to `pause()`.|

The following diagram illustrates the life-cycle visually:

![images/70efff32-dd28-11e3-9fc4-1eb57143aee6.png](/assets/images/70efff32-dd28-11e3-9fc4-1eb57143aee6.png)

## Where is the main loop?
SharpGDX is event driven by nature, mostly due to the way Android and Javascript work. An explicit main loop does not exist, however, the `IApplicationListener.Render()` method can be regarded as the body of such a main loop.