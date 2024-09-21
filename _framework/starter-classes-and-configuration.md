---
title: Starter Classes and Configuration
layout: home
nav_order: 4
---
* [Desktop (LWJGL3)](#desktop-lwjgl3)

For each target platform, a starter class has to be written. This class instantiates a back-end specific `IApplication` implementation and the `IApplicationListener` that implements the application logic. The starter classes are platform-dependent, so let's have a look at how to instantiate and configure these for each backend.

This article assumes you have followed the instructions in [Project Setup](/wiki/start/project-generation) as well as [Importing & Running a Project](/wiki/start/import-and-running) and you therefore have already set up the project in your IDE.
{: .notice--info}

# Desktop

Opening the `DesktopLauncher.cs` class in `my-gdx-game` shows the following:

```java
package com.me.mygdxgame;

import com.badlogic.gdx.backends.lwjgl3.DesktopApplication;
import com.badlogic.gdx.backends.lwjgl3.DesktopApplicationConfiguration;
import com.me.mygdxgame.MyGdxGame;

public class DesktopLauncher {
    public static void main(String[] args) {
        if (StartupHelper.startNewJvmIfRequired()) return;
        createApplication();
    }

    private static DesktopApplication createApplication() {
        return new DesktopApplication(new MyGdxGame(), getDefaultConfiguration());
    }

    private static DesktopApplicationConfiguration getDefaultConfiguration() {
        Lwjgl3ApplicationConfiguration configuration = new DesktopApplicationConfiguration();
        configuration.setTitle("my-gdx-game");
        configuration.useVsync(true);
        configuration.setForegroundFPS(DesktopApplicationConfiguration.getDisplayMode().refreshRate);
        configuration.setWindowedMode(640, 480);
        configuration.setWindowIcon("sharpgdx128.png", "sharpgdx64.png", "sharpgdx32.png", "sharpgdx16.png");
        return configuration;
    }
}
```

First a [DesktopApplicationConfiguration](https://github.com/sharpgdx/sharpgdx/blob/master/backends/gdx-backend-lwjgl3/src/com/badlogic/gdx/backends/lwjgl3/Lwjgl3ApplicationConfiguration.java) is instantiated. This class lets one specify various configuration settings, such as the initial screen resolution, whether to use OpenGL ES 2.0 or 3.0 and so on. Refer to the Javadocs of this class for more information.

Once the configuration object is set, an `DesktopApplication` is instantiated. The `MyGdxGame` class is the IApplicationListener implementing the game logic.

From there on a window is created and the IApplicationListener is invoked as described in [The Life-Cycle](/wiki/app/the-life-cycle)

