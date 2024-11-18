---
title: "A Simple Game"
layout: home
nav_order: 0
---

Let's make a game! Game design is hard, but if you break up the process into small, achievable goals, you'll be able to produce wonders. In this simple game tutorial, you will learn how to make a basic game from scratch. These are the essential skills that you will build on in future projects.

![](/assets/a-simple-game/screenshot.png)

As you can see from the screenshot, we're going to make a basic game where you control a bucket to collect water droplets falling from the sky. There is no score or end goal. Just enjoy the experience! Here are the steps that we will use to split up the game design process:

  * [Prerequisites](#prerequisites)
  * [Loading Assets](#loading-assets)
  * [The Game Life Cycle](#the-game-life-cycle)
  * [Rendering](#rendering)
  * [Input Controls](#input-controls)
  * [Game Logic](#game-logic)
  * [Sound and Music](#sound-and-music)
  * [Further Learning](#further-learning)
  * [Full Example Code](#full-example-code)

## Prerequisites
There are a few things that you need to do before you begin this tutorial.

  * Make sure to follow the [setup instructions](/wiki/start/setup) on SharpGDX.com to configure your IDE.
  * Create a project with the following settings:
    * Project Template: Console App
    * Project Name: Drop
    * Framework: .NET 8.0 (Long Term Support)
    * Do not use top-level statements: selected
  * Add a reference to the SharpGDX.Desktop NuGet package.
  
If you have knowledge of C# code, your experience in SharpGDX will be a lot easier. However, you should still be able to follow along in this tutorial even if you only have a minimal understanding of how code works.

Comments will be used throughout the code examples to explain the facets of this tutorial. You do not have to copy these comments when writing your code:

```csharp
// This is a comment. It will be ignored by the compiler.
```

However, it is good practice to make your own comments to explain confusing code or to label parts of your design.

```csharp
// Restrict bucket movement to the width of the viewport.
```

Using directives are an important part of C# programming. Thankfully, they are automatically added by modern IDE's as you type your code. They are omitted in the examples, but assume they are necessary in your code. They typically appear at the top of the file.

```csharp
using SharpGDX;
using SharpGDX.Audio;
using SharpGDX.Graphics;
using SharpGDX.Graphics.G2D;
using SharpGDX.Mathematics;
using SharpGDX.Utils;
using SharpGDX.Utils.Viewports;
```

There are times where something named in SharpGDX is named the same as something found in another package. As you type `Rectangle`, for example, most IDEs will prompt you with a list of options to autocomplete the using directive. Always opt for the SharpGDX named class in the subsequent menu. These are usually in the parent namespace `SharpGDX`.

![SharpGDX Rectangle class](/assets/a-simple-game/multiple-namespaces.png)

Decimal numbers in OpenGL based games (like those made with SharpGDX) are usually described with floating point variables like `22.5f`. Forgetting the "f" can cause build errors. `float` is preferred over `double` (such as `22.5`) because it uses less memory, it's what's supported by most hardware, and it's what OpenGL usually expects. You can check what parameters a method expects by pressing `CTRL+K, CTRL+P` in Visual Studio.

![method parameters](/assets/a-simple-game/method-parameters.png)

Whenever you see ellipses `...` in the code examples below, assume that other code has been removed for brevity. Use the context of the lines you can see to figure out where you should be in the file. If you're completely lost, the complete example is listed at the bottom.

Before we can test our game, we should set the size of the desktop window. Any configuration that needs to happen for the desktop version of your game needs to be set in the DesktopLauncher class. Find this file in the project folder:

![DesktopLauncher](/assets/a-simple-game/desktop-launcher-class.png)

```csharp
...

private static DesktopApplicationConfiguration getDefaultConfiguration() 
{
    DesktopApplicationConfiguration configuration = new DesktopApplicationConfiguration();

    configuration.setTitle("Drop");
    configuration.useVsync(true);
    configuration.setForegroundFPS(DesktopApplicationConfiguration.getDisplayMode().refreshRate + 1);
    configuration.setWindowedMode(800, 500); // this line changes the size of the window
    configuration.setWindowIcon("sharpgdx128.png", "sharpgdx64.png", "sharpgdx32.png", "sharpgdx16.png");

    return configuration;
}
```

## Loading Assets
2D games made in SharpGDX need assets: images, audio, and other resources that comprise the project. In this case, we'll need a bucket, a raindrop, a background, a water drop sound effect, and music. If you're pretty resourceful, you can make these on your own. For simplicity's sake, you can download these examples which are optimized for this tutorial.

<a href="/assets/simple-game/bucket.png?nomagnify" download="bucket.png">bucket.png</a><br>
<a href="/assets/simple-game/drop.png?nomagnify" download="drop.png">drop.png</a><br>
<a href="/assets/simple-game/background.png?nomagnify" download="background.png">background.png</a><br>
<a href="/assets/simple-game/drop.mp3?nomagnify" download="drop.mp3">drop.mp3</a><br>
<a href="/assets/simple-game/music.mp3?nomagnify" download="music.mp3">music.mp3</a>

Just having these saved on your computer is not enough. These files need to be placed in the assets folder of your project. Look inside the project folder:

![assets folder](/assets/images/dev/a-simple-game/4.png)

There are many folders in here for the different backends that SharpGDX supports. Assets is a folder shared by all the backends. Whatever you save in here gets distributed with your game. For example, your desktop game will include these files inside your JAR distributable. This is what you give your users so they can play your game.

Note that file name casing and extensions are important in SharpGDX. There is a difference between `drop.png`, `drop.mp3`, and `Drop.mp3`. And annoyingly, this usually doesn't break your game until you try to make a release. Always refer to the in-editor project browser as file extensions are hidden by default in the Windows File Explorer.

SharpGDX has an emphasis on code. Every asset you use must be loaded through code before you can use it in the rest of your game. This needs to happen when the game starts. Open the Core project > Main.cs. This file is the main file we're going to work in.

![Drop class](/assets/images/dev/a-simple-game/5.png)

Declare your variables at the top of the file right underneath the `public class Drop` line. You'll need a variable for every asset you plan to use:

```csharp
public class Main : IApplicationListener
{
    private Texture _backgroundTexture = null!;
    private Texture _bucketTexture     = null!;
    private ISound  _dropSound         = null!;
    private Texture _dropTexture       = null!;
    private IMusic  _music             = null!;
```

However, you should not create these objects at the constructor or init level. This would fail because SharpGDX needs to load first. Enter the following code while implementing th create method:

```csharp
    public void Create()
    {
        _backgroundTexture = new Texture("background.png");
        _bucketTexture = new Texture("bucket.png");
        _dropTexture = new Texture("drop.png");
    }
```

This loads the assets into memory after SharpGDX has started. See that the background image is loaded as a texture. Textures are the way that games keep images in video ram. It's actually not efficient to have different textures for each element in the game. It should be one big Texture for them all. Learn about [TexturePacker](/wiki/tools/texture-packer) in the wiki. For now, we'll keep these as separate textures because it's easier to explain this way.

Similarly, we have Sound to handle the raindrop audio file in our project. Sounds are loaded completely into memory so they can be played quickly and repeatedly. Music, on the other hand, is too large to keep entirely in memory. It's streamed from the file in chunks. There is no precise rule about what should or should not be a sound versus a music, but you may consider any audio shorter than 10 seconds to be a sound.

```csharp
    public void Create()
    {
        ...
    
        _dropSound = Gdx.Audio.NewSound(Gdx.Files.Internal("drop.mp3"));
        _music = Gdx.Audio.NewMusic(Gdx.Files.Internal("music.mp3"));
    }
```

We have many assets to manage now. We should use an [AssetManager](/wiki/managing-your-assets) to handle them. That can be found in the wiki too. Again, that's outside the scope of this tutorial. Let's keep it simple.

## The Game Life Cycle
The work we've done so far is in the create method. Create executes immediately when the game is run, so of course it makes sense to load our assets there. What are these other methods for? The SharpGDX [life cycle](/wiki/app/the-life-cycle) lists these in more detail.

`Create()`, `Render()`, `Resize(int width, int height)`, `Pause()`, `Resume()`, and `Dispose()` are all included in your class because you are implementing the `ApplicationListener` interface. You will use these methods to create your game and all future games you make in SharpGDX. You'll find that a lot of advanced systems you can use abstract the direct use of these methods, however these remain at the foundation of your code.

Most of the code in this example will be in the create and render methods. For example, we won't be using `Dispose` because it's not necessary for us to clean up resources in a simple app like this. This is more relevant for games with multiple screens.

## Rendering
Now let's talk about rendering. For the most part, all modern games just manipulate textures, drawing them to the screen to give you the final image you see: the frame.

This process is repeated many times per second to give the illusion of motion. That's what we're going to do here. Let's start with some boilerplate code. What is meant by boilerplate code is that you'll use this same code again and again without much change. And you'll see this pattern in all the games you make. Declare new variables:

```csharp
public class Main : ApplicationListener
{
    ...

    private SpriteBatch _spriteBatch = null!;
    private FitViewport _viewport    = null!;
```

Initialize these variables in the create method:

```csharp
    public void Create()
    {
        ...

        _spriteBatch = new SpriteBatch();
        _viewport = new FitViewport(8, 5);
    }
```

A viewport controls how we see the game. It's like a window from our world into another: the game world. The viewport controls how big this "window" is and how it's placed on our screen. There are many kinds of viewports you can use. A simple one to understand is the FitViewport which will ensure that no matter what size our window is, the full game view will always be visible. The parameters determine how large our visible game world will be in game units. It will "fit" into the window. Each viewport also has a camera which controls what part of the game world is visible and at what zoom. Learn more about [viewports and cameras](/wiki/graphics/viewports) in the wiki.

You should always remember to update the viewport in the resize method:

```csharp
    public void Resize(int width, int height)
    {
        _viewport.Update(width, height, true); // true centers the camera
    }
```

Our code is going to get more complex now. A good practice is to divide your code into separate methods. We'll prepare for this by adding new methods to be used inside the render method:

```csharp
    public void Render()
    {
        // organize code into three methods
        Input();
        Logic();
        Draw();
    }

    private void Input()
    {

    }

    private void Logic()
    {

    }

    private void Draw()
    {

    }
```

We will focus on the `Draw()` method for now.

```csharp
    private void Draw()
    {
        ScreenUtils.Clear(Color.Black);
        _viewport.Apply();
        _spriteBatch.SetProjectionMatrix(_viewport.GetCamera().Combined);
        _spriteBatch.Begin();

        _spriteBatch.End();
    }
```

`ScreenUtils.clear(Color.BLACK);` clears the screen. It's a good practice to clear the screen every frame. Otherwise, you'll get weird graphical errors. You can use any color you want, but we'll just settle on Black this time.

Ever wonder why your favorite games sometimes have poor FPS or Frames Per Second? Stuttering gameplay is often related to the number of textures being rendered and the capabilities of your player's graphics card. There are tricks to alleviate this. For one, it is more efficient to send all your draw calls at once to the graphics processing unit (GPU). The process of drawing an individual texture is called a draw call. The SpriteBatch is how SharpGDX combines these draw calls together.

`spriteBatch.setProjectionMatrix(viewport.getCamera().combined);` shows how the Viewport is applied to the SpriteBatch. This is necessary for the images to be shown in the correct place.

It is important to order the begin and end lines appropriately. You should never draw from a SpriteBatch outside of a begin and an end. If you do, you will get an error message.

```csharp
        _spriteBatch.Begin();
        // add lines to draw stuff here
        _spriteBatch.End();
```

So, let's do that. The coordinates we provide determine where the bucket will be drawn on the screen. The coordinates begin in the bottom left and grow to the right and up. Our game world is described in imaginary units best defined as meters. For reference our bucket is 100 pixels wide and 100 pixels tall. For simplicity, we will decide that 100 pixels will equal 1 meter, making our bucket 1x1 meters. This ratio of pixels per meter can be anything you want, but make sure whatever you choose is a simple value that makes sense in your game world. This is typically the size of your tiles or the height of the player character. Your game logic should really know nothing about pixels.

![Coordinate Plane](/assets/images/dev/a-simple-game/6.png)

Add the line to draw the bucket:

```csharp
    private void Draw()
    {
        ScreenUtils.Clear(Color.Black);
        _viewport.Apply();
        _spriteBatch.SetProjectionMatrix(_viewport.GetCamera().Combined);
        _spriteBatch.Begin();

        _spriteBatch.Draw(_bucketTexture, 0, 0, 1, 1); // draw the bucket with width/height of 1 meter

        _spriteBatch.End();
    }
```

This code should now draw our bucket at the bottom of the screen in the lower left corner. You can run this game by calling the appropriate Gradle command in IDEA or implement whatever steps are needed for your chosen development environment as listed in the [setup guide](/wiki/start/setup). If all things have gone well, you should see our brave, lone bucket sitting in the darkness of the void.

![bucket in void](/assets/images/dev/a-simple-game/7.png)

Let's cheer up this scene with the background. Drawing the background is similar to drawing the bucket. It is drawn at the width/height of the viewport to ensure that the entire view is covered:

```csharp
    private void Draw()
    {
        ScreenUtils.Clear(Color.Black);
        _viewport.Apply();
        _spriteBatch.SetProjectionMatrix(_viewport.GetCamera().Combined);
        _spriteBatch.Begin();

        // store the worldWidth and worldHeight as local variables for brevity
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        _spriteBatch.Draw(_bucketTexture, 0, 0, 1, 1); // draw the bucket
        _spriteBatch.Draw(_backgroundTexture, 0, 0, worldWidth, worldHeight); // draw the background

        _spriteBatch.End();
    }
```

You can run the game and you'll see the background.

![background with no bucket](/assets/images/dev/a-simple-game/8.png)

But what happened to our bucket? We need to talk about draw order. Drawing happens consecutively in the order you list it in code. This is what really happens:

1. The screen is cleared.
2. The bucket is drawn to the back buffer.
3. The background is drawn over everything. This final image is then shown on the screen. 

This process is repeated every frame. To resolve this problem, we simply we need to reorder our draw calls.

```csharp
    private void Draw()
    {
        ScreenUtils.Clear(Color.Black);
        _viewport.Apply();
        _spriteBatch.SetProjectionMatrix(_viewport.GetCamera().Combined);
        _spriteBatch.Begin();

        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        // rearrange these two lines
        _spriteBatch.Draw(_backgroundTexture, 0, 0, worldWidth, worldHeight); // draw the background
        _spriteBatch.Draw(_bucketTexture, 0, 0, 1, 1); // draw the bucket

        _spriteBatch.End();
    }
```

![background with bucket](/assets/images/dev/a-simple-game/9.png)

Ensure that your game shows the background and the bucket. We'll skip rendering the droplets for now and return to it when we have the logic to create them.

## Input Controls
It's not fun to have a game without some sort of movement or action on screen. Let's enable the player's ability to control the bucket. As you know, there are all sorts of ways to get input from a user. We'll focus on a few: the keyboard, mouse, and touch.

We need some way of keeping track of where the player bucket is in the game world. Texture does not store any position state. Sure, you can tell SpriteBatch where to draw it every frame by using the provided overloaded methods. What if you want to rotate it? Resize it? These methods get incredibly complicated the more you want to do.

![long method parameters](/assets/images/dev/a-simple-game/10.png)

Let's use a Sprite instead. Sprite is capable of doing all these things and keeping state. This means that it will remember its properties instead of you having to define them every frame.

```csharp
public class Main : IApplicationListener 
{
    ...
    private Sprite _bucketSprite = null!; // Declare a new Sprite variable
```

```csharp
    public void Create()
    {
        ...
        _bucketSprite = new Sprite(_bucketTexture); // Initialize the sprite based on the texture
        _bucketSprite.SetSize(1, 1); // Define the size of the sprite
    }
```

Erase the `_spriteBatch.Draw(_bucketTexture, 0, 0, 1, 1);` line for the bucket. The Sprite draw code is written in a different way:

```csharp
    private void Draw()
    {
        ScreenUtils.Clear(Color.Black);
        _viewport.Apply();
        _spriteBatch.SetProjectionMatrix(_viewport.GetCamera().Combined);
        _spriteBatch.Begin();

        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        _spriteBatch.Draw(_backgroundTexture, 0, 0, worldWidth, worldHeight);
        _bucketSprite.Draw(_spriteBatch); // Sprites have their own draw method

        _spriteBatch.End();
    }
```
If you run the game again, there shouldn't be any difference in the output. That's good! If you don't see the bucket, make sure you adjusted the line indicated above correctly.

### Keyboard
Now to capturing player input. This is how you detect if a player is pressing keys on the keyboard. This needs to happen in the input method

```csharp
    private void Input()
    {
        if (Gdx.Input.IsKeyPressed(IInput.Keys.Right))
        {
            // TODO: Do something when the user presses the right arrow
        }
    }
```

This is known as keyboard polling. Every time a frame is drawn, we're going to check if a key is pressed. There is a list of pretty much every conceivable key in `Gdx.input.Input.Keys`. We want to react to the user pressing the right arrow key.

![keys list](/assets/images/dev/a-simple-game/11.png)

That's great, but what is supposed to happen when the key is pressed? We need to move the coordinates of the bucket sprite.

```csharp
    private void Input()
    {
        const float speed = .25f;

        if (Gdx.Input.IsKeyPressed(IInput.Keys.Right))
        {
            _bucketSprite.TranslateX(speed); // Move the bucket right
        }
    }
```

The variable `speed` dictates how fast the bucket moves. Adding to the x makes the bucket move to the right. This basically means "set the bucket x to what it currently is right now plus a little bit more". Subtracting from the x makes the bucket move to the left.

An unfortunate side effect of having our logic inside the render method is that our code behaves differently on different hardware. This is because of differences in framerate. More frames per second means more movement per second.

![fps comparison](/assets/images/dev/a-simple-game/12.png)

To counteract this, we need to use delta time. Delta time is the measured time between frames. If we multiply our movement by delta time, the movement will be consistent no matter what hardware we run this game on.

```csharp
    private void Input()
    {
        const float speed = .25f;
        var delta = Gdx.Graphics.GetDeltaTime(); // retrieve the current delta

        if (Gdx.Input.IsKeyPressed(IInput.Keys.Right))
        {
            _bucketSprite.TranslateX(speed * delta); // Move the bucket right
        }
    }
```

This effectively means that the number we select here is how far the bucket moves in one second. Remember to use delta time whenever you are calculating something that happens over time. Adjust the number to a value that moves the bucket at an acceptable speed like `4f`. Now let's copy the code for movement to the left. Flip the plus to a minus to move to the left.

```csharp
    private void Input()
    {
        const float speed = 4f;
        var delta = Gdx.Graphics.GetDeltaTime();

        if (Gdx.Input.IsKeyPressed(IInput.Keys.Right))
        {
            _bucketSprite.TranslateX(speed * delta); // move the bucket right
        }
        else if (Gdx.Input.IsKeyPressed(IInput.Keys.LEFT))
        {
            _bucketSprite.TranslateX(-speed * delta); // move the bucket left
        }
    }
```

Run the game again to make sure you can move the bucket left and right with the keyboard.

### Mouse and Touch Controls
Mouse and Touch controls are related. To react to the user clicking or tapping the screen, call the following method:

```csharp
    private void Input()
    {
        const float speed = 4f;
        var delta = Gdx.Graphics.GetDeltaTime();

        if (Gdx.Input.IsKeyPressed(IInput.Keys.Right))
        {
            _bucketSprite.TranslateX(speed * delta);
        }
        else if (Gdx.Input.IsKeyPressed(IInput.Keys.LEFT))
        {
            _bucketSprite.TranslateX(-speed * delta);
        }

        if (Gdx.Input.IsTouched()) // If the user has clicked or tapped the screen
        {
            // TODO: React to the player touching the screen
        }
    }
```

Now the player has clicked the screen, but where did they click? We can use the methods Gdx.input.getX() and Gdx.input.getY() for this. Unfortunately, these values are in window coordinates which don't correlate to our selected pixels per meter. The coordinates are also upside down because Window coordinates start from the top left. We need to declare a Vector2 object to do some math.

```csharp
public class Main : IApplicationListener
{
    ...
    private Vector2 _touchPos = null!;
```

Initialize the Vector2:

```csharp
@Override
    public void Create()
    {
        ...

        _touchPos = new Vector2();
    }
```

Notice that we have created a single instance variable for the Vector2 instead of creating it locally. By reusing this Vector2, we prevent the game from triggering the garbage collector frequently which causes lag spikes in the game. This is how we use the Vector2 to move the bucket:

```csharp
    private void Input()
    {
        const float speed = 4f;
        var delta = Gdx.Graphics.GetDeltaTime();

        if (Gdx.Input.IsKeyPressed(IInput.Keys.Right))
        {
            _bucketSprite.TranslateX(speed * delta);
        }
        else if (Gdx.Input.IsKeyPressed(IInput.Keys.LEFT))
        {
            _bucketSprite.TranslateX(-speed * delta);
        }

        if (Gdx.Input.IsTouched())
        {
            _touchPos.Set(Gdx.Input.GetX(), Gdx.Input.GetY()); // Get where the touch happened on screen
            _viewport.Unproject(_touchPos); // Convert the units to the world units of the viewport
            _bucketSprite.SetCenterX(_touchPos.x); // Change the horizontally centered position of the bucket
        }
    }
```

This converts the window coordinates to coordinates in our world space. This code actually supports mobile devices as well, however you should read about [some other input features](/wiki/input/event-handling) that SharpGDX provides you. Run the game and click the screen to move the bucket.

## Game Logic
The player can move left and right now, but they can go completely off the screen. We need to prevent the player from doing that. SharpGDX provides some helpful methods in the `MathUtils` class. We can achieve our goal by using the `clamp()` method. Remember that the left side of the screen starts at 0. This code detects if the bucket goes too far left. If it does, it snaps its position at the farthest it's allowed to go. Add the following lines to the logic method:

```csharp
    private void Logic()
    {
        // Store the worldWidth and worldHeight as local variables for brevity
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        // Clamp x to values between 0 and worldWidth
        _bucketSprite.SetX(MathUtils.Clamp(_bucketSprite.GetX(), 0, worldWidth));
    }
```

Try the game out. This kind of works, but it lets the bucket go just a little too far right. In fact, it's one whole unit too far to the right. This is because the bucket sprite has an origin on the bottom left of the image. To resolve this, we need to subtract the width of the bucket from the right edge.

```csharp
    private void Logic()
    {
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        // Store the bucket size for brevity
        var bucketWidth = _bucketSprite.GetWidth();
        var bucketHeight = _bucketSprite.GetHeight();

        // Subtract the bucket width
        _bucketSprite.SetX(MathUtils.Clamp(_bucketSprite.GetX(), 0, worldWidth - bucketWidth));
    }
```

Now to spawn the rain drops. We will have more than one raindrop, so we need a list to keep track of them. Thankfully, SharpGDX has many useful [collections](/wiki/utils/collections) to help with this. Declare the list:

```csharp
public class Main : IApplicationListener
{
    ...
    private Array<Sprite> _dropSprites       = null!;
```

Initialize the list:

```csharp
    public void Create()
    {
        ...

        _dropSprites = [];
    }
```

As before, it is advised to organize your code into methods. Create a new method to create a droplet. Place it after your draw method.

```csharp
    private void Draw()
    {
        ...
    }

    private void CreateDroplet()
    {
        // create local variables for convenience
        const float dropWidth = 1;
        const float dropHeight = 1;
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        // create the drop sprite
        var dropSprite = new Sprite(_dropTexture);
        dropSprite.SetSize(dropWidth, dropHeight);
        dropSprite.SetX(0);
        dropSprite.SetY(worldHeight);
        _dropSprites.Add(dropSprite); // Add it to the list
    }

    public void Pause()
    {

    }
```

The size is the same as the player. Setting the y position at the top of the screen will make it appear as if it's falling from the sky. The `dropSprites.add(dropSprite);` line adds the drop to the list of drops that we can manage in our render loop. 

We'll call `CreateDroplet()` in the create method.

```csharp
    public void Create()
    {
        ...
        _dropSprites = [];

        CreateDroplet();
    }
```

Drawing each drop is pretty simple. Add the sprite drawing code to the draw method:

```csharp
    private void Draw()
    {
        ScreenUtils.Clear(Color.Black);
        _viewport.Apply();
        _spriteBatch.SetProjectionMatrix(_viewport.GetCamera().Combined);
        _spriteBatch.Begin();

        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        _spriteBatch.Draw(_backgroundTexture, 0, 0, worldWidth, worldHeight);
        _bucketSprite.Draw(_spriteBatch);

        // draw each sprite
        foreach (var dropSprite in _dropSprites)
        {
            dropSprite.Draw(_spriteBatch);
        }

        _spriteBatch.End();
    }
```

If you run the program now, you'll see nothing happen. That's because the droplet doesn't have any movement code. Begin coding the logic for the droplets after the bucket logic:

```csharp
    private void Logic()
    {
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();
        var bucketWidth = _bucketSprite.GetWidth();
        var bucketHeight = _bucketSprite.GetHeight();

        _bucketSprite.SetX(MathUtils.Clamp(_bucketSprite.GetX(), 0, worldWidth - bucketWidth));

        var delta = Gdx.Graphics.GetDeltaTime(); // retrieve the current delta

        // loop through each drop
        foreach (var dropSprite in _dropSprites)
        {
            dropSprite.TranslateY(-2f * delta); // move the drop downward every frame
        }
    }
```

We have a problem here. The rain drop only spawns on the left side every time. You could change the position, of course, but there is no variability. No randomness. We need it to be a random position between 0 and the width of the world.

```csharp
    private void CreateDroplet()
    {
        const float dropWidth = 1;
        const float dropHeight = 1;
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        var dropSprite = new Sprite(_dropTexture);
        dropSprite.SetSize(dropWidth, dropHeight);
        dropSprite.SetX(MathUtils.random(0f, worldWidth - dropWidth)); // Randomize the drop's x position
        dropSprite.SetY(worldHeight);
        _dropSprites.Add(dropSprite);
    }
```

Again, we're subtracting the width of the sprite so none of the raindrops appear outside of the view. Success!

That's only one droplet though. When it rains, we should have multiple droplets over the course of time. Let's move the droplet spawning code to the logic method. This will make droplets repeatedly every frame. Cut the line from the create method:

![Cut the droplet](/assets/images/dev/a-simple-game/13.png)

Paste it into the logic method:

```csharp
    private void Logic()
    {
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();
        var bucketWidth = _bucketSprite.GetWidth();
        var bucketHeight = _bucketSprite.GetHeight();

        _bucketSprite.SetX(MathUtils.Clamp(_bucketSprite.GetX(), 0, worldWidth - bucketWidth));

        var delta = Gdx.Graphics.GetDeltaTime();

        foreach (var dropSprite in _dropSprites)
        {
            dropSprite.TranslateY(-2f * delta);
        }

        // paste the line here
        CreateDroplet();
    }
```

If you run this, you'll see that we have a catastrophe! There are too many droplets.

![too many droplets](/assets/images/dev/a-simple-game/14.png)

There should be a delay between each spawn. Whenever we need something to be done repeatedly over time with a delay, we can create a timer. Declare a new variable to store the time:

```csharp
public class Main : IApplicationListener
{
    ...
    private float _dropTimer = 0.0f;
```

`dropTimer` will keep track of how much time has elapsed between each spawn. Modify the code in the logic method:

```csharp
    private void Logic()
    {
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();
        var bucketWidth = _bucketSprite.GetWidth();
        var bucketHeight = _bucketSprite.GetHeight();

        _bucketSprite.SetX(MathUtils.Clamp(_bucketSprite.GetX(), 0, worldWidth - bucketWidth));

        var delta = Gdx.Graphics.GetDeltaTime();

        foreach (var dropSprite in _dropSprites)
        {
            dropSprite.TranslateY(-2f * delta);
        }

        _dropTimer += delta; // Adds the current delta to the timer
        
        if (_dropTimer > 1f)
        { // Check if it has been more than a second
            _dropTimer = 0; // Reset the timer
            CreateDroplet(); // Create the droplet
        }
    }
```

`_dropTimer` accumulates the time that passes between every frame. If it's been more than a second, it will update the recorded time and proceed to create the droplet. This works as expected now.

![3 droplets](/assets/images/dev/a-simple-game/15.png)

These droplets will fall off the screen never to be seen again. C# doesn't forget though. These droplets will remain in memory forever. If you [profile](https://visualvm.github.io/) your game you'll see that we have a memory leak.

![memory profile](/assets/images/dev/a-simple-game/16.png)

If the player would leave the game on for a really long time, it will crash. So, we should remove the drop sprite from the list when it falls off screen. We need to make some considerable modifications to the logic for loop. Erase your loop in the logic method:

![erase lines](/assets/images/dev/a-simple-game/17.png)

Replace it with the loop indicated below:

```csharp
    private void Logic()
    {
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();
        var bucketWidth = _bucketSprite.GetWidth();
        var bucketHeight = _bucketSprite.GetHeight();

        _bucketSprite.SetX(MathUtils.Clamp(_bucketSprite.GetX(), 0, worldWidth - bucketWidth));

        var delta = Gdx.Graphics.GetDeltaTime();

        // Loop through the sprites backwards to prevent out of bounds errors
        for (var i = _dropSprites.size - 1; i >= 0; i--)
        {
            var dropSprite = _dropSprites.Get(i); // Get the sprite from the list
            var dropWidth = dropSprite.GetWidth();
            var dropHeight = dropSprite.GetHeight();

            dropSprite.TranslateY(-2f * delta);

            // if the top of the drop goes below the bottom of the view, remove it
            if (dropSprite.GetY() < -dropHeight)
            {
                _dropSprites.RemoveIndex(i);
            }
        }

        _dropTimer += delta;

        if (_dropTimer > 1f)
        {
            _dropTimer = 0;
            CreateDroplet();
        }
    }
```

Removing items in a list while you are iterating through it can cause some unforeseen bugs. That's why we are iterating through the list backwards so you don't skip any indexes. Make sure to learn about other [collections](/wiki/utils/collections#specialized-lists) available like the SnapshotArray and the DelayedRemovalArray for more complex projects.

We have made great progress, however the drops don't interact with the bucket. This is where we incorporate some rudimentary collision detection. This can be achieved with the Rectangle class. We need two rectangles to make comparisons. One for the bucket and one to be reused with every drop.

```csharp
public class Main : IApplicationListener
{
    ...
    private Rectangle     _bucketRectangle   = null!;
    Rectangle             _dropRectangle     = null!;
```

```csharp
    public void Create()
    {
        ...

        _bucketRectangle = new Rectangle();
        _dropRectangle = new Rectangle();
    }
```

The code now sets the rectangles to the position and dimensions of the Sprites.

```csharp
    private void Logic()
    {
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();
        var bucketWidth = _bucketSprite.GetWidth();
        var bucketHeight = _bucketSprite.GetHeight();

        _bucketSprite.SetX(MathUtils.Clamp(_bucketSprite.GetX(), 0, worldWidth - bucketWidth));

        var delta = Gdx.Graphics.GetDeltaTime();
        // Apply the bucket position and size to the bucketRectangle
        _bucketRectangle.Set(_bucketSprite.GetX(), _bucketSprite.GetY(), bucketWidth, bucketHeight);

        for (var i = _dropSprites.size - 1; i >= 0; i--)
        {
            var dropSprite = _dropSprites.Get(i);
            var dropWidth = dropSprite.GetWidth();
            var dropHeight = dropSprite.GetHeight();

            dropSprite.TranslateY(-2f * delta);
            // Apply the drop position and size to the dropRectangle
            _dropRectangle.Set(dropSprite.GetX(), dropSprite.GetY(), dropWidth, dropHeight);

            if (dropSprite.GetY() < -dropHeight)
            {
                _dropSprites.RemoveIndex(i);
            }
            else if (_bucketRectangle.Overlaps(_dropRectangle)) // Check if the bucket overlaps the drop
            {
                _dropSprites.RemoveIndex(i); // Remove the drop
            }
        }

        _dropTimer += delta;

        if (_dropTimer > 1f)
        {
            _dropTimer = 0;
            CreateDroplet();
        }
    }
```

`_bucketRectangle.Overlaps(_dropRectangle)` checks if the bucket overlaps the drop. If it does, the Sprite will be removed from the list of drop Sprites. That means it will no longer be drawn or acted upon. It simply doesn't exist anymore, making it look like the bucket collected it. Make sure your game works as expected. You're almost done!

## Sound and Music
It's very easy to add a line to play a sound effect now that we are at the end of our workflow. We want the drop sound (the sound effect loaded at the beginning of this tutorial) to play when the bucket collides with the drop. It should not play when the drop falls out of the level.

```csharp
    private void Logic()
    {
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();
        var bucketWidth = _bucketSprite.GetWidth();
        var bucketHeight = _bucketSprite.GetHeight();

        _bucketSprite.SetX(MathUtils.Clamp(_bucketSprite.GetX(), 0, worldWidth - bucketWidth));

        var delta = Gdx.Graphics.GetDeltaTime();
        _bucketRectangle.Set(_bucketSprite.GetX(), _bucketSprite.GetY(), bucketWidth, bucketHeight);

        for (var i = _dropSprites.size - 1; i >= 0; i--)
        {
            var dropSprite = _dropSprites.Get(i);
            var dropWidth = dropSprite.GetWidth();
            var dropHeight = dropSprite.GetHeight();

            dropSprite.TranslateY(-2f * delta);
            _dropRectangle.Set(dropSprite.GetX(), dropSprite.GetY(), dropWidth, dropHeight);

            if (dropSprite.GetY() < -dropHeight)
            {
                _dropSprites.RemoveIndex(i);
            }
            else if (_bucketRectangle.Overlaps(_dropRectangle))
            {
                _dropSprites.RemoveIndex(i);
                _dropSound.Play(); // Play the sound
            }
        }

        _dropTimer += delta;
        
        if (_dropTimer > 1f)
        {
            _dropTimer = 0;
            CreateDroplet();
        }
    }
```

The music should play at the beginning of the game. It needs to loop continuously until the player is done playing the game. The file is also a little loud, so it should play at half volume. Volume is a value from 0f to 1f with 1f being the normal volume of the file.

```csharp
    public void Create()
    {
        ...

        _music.SetLooping(true);
        _music.SetVolume(.5f);
        _music.Play();
    }
```

Make sure your speakers are on and try it out.

## Further learning
So, you're at the final steps of making a game. You should test the game out. Tweak values to make the game easier or harder. This can be done by changing how fast the bucket moves and what rate the droplets spawn.

If you want to let your friends and colleagues try your game out, you'll need to make a distributable that they can play. No one is going to want set up an IDE and copy your entire project just to play it. See the page on [Deploying your application](/wiki/deployment/deploying-your-application).

Now that you've completed the simple game, it's time to [extend the simple game](/wiki/start/simple-game-extended). This project managed to put all of its code in a single class. This was in the service of making it simple, but it is a terrible way to organize code. The next tutorial will teach you about the Game class and how to implement Screen to arrange your project. It will also cover other important improvements to your game. For example, these instructions skipped the use of the dispose() method because it's not relevant for single page project. When working with multiple screens, you may want to dispose of resources from the last screen to release the memory for new resources in your game.

Game design is a constant journey of learning. The [wiki](/wiki/) goes further in depth regarding all the subjects you have learned here. Look into [collections](/wiki/utils/collections), [TexturePacker](/wiki/tools/texture-packer), [AssetManager](/wiki/managing-your-assets), [audio](/wiki/audio/audio), [memory management](/wiki/articles/memory-management), and [user input](/wiki/input/input-handling).

This tutorial focused entirely on desktop development. There are many more considerations you must make before you explore Android, iOS, and HTML5 development. There is an extensive article on [considerations for HTML5](/wiki/html5-backend-and-gwt-specifics), for example. C# is not truly "write once, run anywhere" but SharpGDX takes you pretty close to that goal.

## Full Example Code
The following is the full example code of the game described throughout this tutorial. It's here for reference if you get stuck on any of the steps. Don't cheat yourself by copying the whole thing! Learning how to program is mainly you asking yourself questions and trying to resolve issues on your own first.

```csharp
using SharpGDX;
using SharpGDX.Audio;
using SharpGDX.Graphics;
using SharpGDX.Graphics.G2D;
using SharpGDX.Mathematics;
using SharpGDX.Utils;
using SharpGDX.Utils.Viewports;

namespace Drop;

public class Main : IApplicationListener
{
    private Texture       _backgroundTexture = null!;
    private Rectangle     _bucketRectangle   = null!;
    private Sprite        _bucketSprite      = null!;
    private Texture       _bucketTexture     = null!;
    private Rectangle     _dropRectangle     = null!;
    private ISound        _dropSound         = null!;
    private Array<Sprite> _dropSprites       = null!;
    private Texture       _dropTexture       = null!;
    private float         _dropTimer         = 0.0f;
    private IMusic        _music             = null!;
    private SpriteBatch   _spriteBatch       = null!;
    private Vector2       _touchPos          = null!;
    private FitViewport   _viewport          = null!;

    public void Create()
    {
        _backgroundTexture = new Texture("background.png");
        _bucketTexture = new Texture("bucket.png");
        _dropTexture = new Texture("drop.png");
        _dropSound = Gdx.Audio.NewSound(Gdx.Files.Internal("drop.mp3"));
        _music = Gdx.Audio.NewMusic(Gdx.Files.Internal("music.mp3"));
        _spriteBatch = new SpriteBatch();
        _viewport = new FitViewport(8, 5);
        _bucketSprite = new Sprite(_bucketTexture);
        _bucketSprite.SetSize(1, 1);
        _touchPos = new Vector2();
        _dropSprites = new Array<Sprite>();
        _bucketRectangle = new Rectangle();
        _dropRectangle = new Rectangle();
        _music.SetLooping(true);
        _music.SetVolume(.5f);
        _music.Play();
    }

    public void Dispose()
    {
    }

    public void Pause()
    {
    }

    public void Render()
    {
        Input();
        Logic();
        Draw();
    }

    public void Resize(int width, int height)
    {
        _viewport.Update(width, height, true);
    }

    public void Resume()
    {
    }

    private void CreateDroplet()
    {
        const float dropWidth = 1;
        const float dropHeight = 1;
        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        var dropSprite = new Sprite(_dropTexture);
        dropSprite.SetSize(dropWidth, dropHeight);
        dropSprite.SetX(MathUtils.random(0f, worldWidth - dropWidth));
        dropSprite.SetY(worldHeight);
        _dropSprites.Add(dropSprite);
    }

    private void Draw()
    {
        ScreenUtils.Clear(Color.Black);
        _viewport.Apply();
        _spriteBatch.SetProjectionMatrix(_viewport.GetCamera().Combined);
        _spriteBatch.Begin();

        var worldWidth = _viewport.GetWorldWidth();
        var worldHeight = _viewport.GetWorldHeight();

        _spriteBatch.Draw(_backgroundTexture, 0, 0, worldWidth, worldHeight);
        _bucketSprite.Draw(_spriteBatch);

        foreach (var dropSprite in _dropSprites)
        {
            dropSprite.Draw(_spriteBatch);
        }

        _spriteBatch.End();
    }

    private void Input()
    {
        const float speed = 4f;
        var delta = Gdx.Graphics.GetDeltaTime();

        if (Gdx.Input.IsKeyPressed(IInput.Keys.RIGHT))
        {
            _bucketSprite.TranslateX(speed * delta);
        }
        else if (Gdx.Input.IsKeyPressed(IInput.Keys.LEFT))
        {
            _bucketSprite.TranslateX(-speed * delta);
        }

        if (Gdx.Input.IsTouched())
        {
            _touchPos.Set(Gdx.Input.GetX(), Gdx.Input.GetY());
            _viewport.Unproject(_touchPos);
            _bucketSprite.SetCenterX(_touchPos.x);
        }
    }

    private void Logic()
    {
        var worldWidth = _viewport.GetWorldWidth();
        var bucketWidth = _bucketSprite.GetWidth();
        var bucketHeight = _bucketSprite.GetHeight();

        _bucketSprite.SetX(MathUtils.Clamp(_bucketSprite.GetX(), 0, worldWidth - bucketWidth));

        var delta = Gdx.Graphics.GetDeltaTime();
        _bucketRectangle.Set(_bucketSprite.GetX(), _bucketSprite.GetY(), bucketWidth, bucketHeight);

        for (var i = _dropSprites.size - 1; i >= 0; i--)
        {
            var dropSprite = _dropSprites.Get(i);
            var dropWidth = dropSprite.GetWidth();
            var dropHeight = dropSprite.GetHeight();

            dropSprite.TranslateY(-2f * delta);
            _dropRectangle.Set(dropSprite.GetX(), dropSprite.GetY(), dropWidth, dropHeight);

            if (dropSprite.GetY() < -dropHeight)
            {
                _dropSprites.RemoveIndex(i);
            }
            else if (_bucketRectangle.Overlaps(_dropRectangle))
            {
                _dropSprites.RemoveIndex(i);
                _dropSound.Play();
            }
        }

        _dropTimer += delta;

        if (_dropTimer > 1f)
        {
            _dropTimer = 0;
            CreateDroplet();
        }
    }
}
```