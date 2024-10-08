---
title: Audio
layout: home
nav_order: 1
---

{: .important }
> This is a copy of the Java documentation for libGDX. It is not finished being ported.

# Introduction

SharpGDX provides methods to playback small sound effects as well as stream larger music pieces directly from disk. It also provides convenient read and write access to the audio hardware.

All access to the audio facilities is done through the [audio module](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/Audio.html), referenced by:

```csharp
var audio = GDX.Audio;
```

SharpGDX will automatically pause and resume all audio playback for you if your application is paused and resumed.


# Audio on Android

SharpGDX Android backend uses the `SoundPool` API to play `Sound`s and the `MediaPlayer` for `Music`. These API have some limitations and known issues in certain scenarios:
- Latency is not great and the default implementation is not recommended for latency sensitive apps like rhythm games.
- Playing several sounds at the same time may cause performance issues on some devices. An easy way to fix it (with the limitation some methods are unsupported) is using the alternative Android implementation `AsynchronousAndroidAudio` by implementing `createAudio()`on `AndroidLauncher` like this:

```csharp
public override AndroidAudio createAudio(Context context, AndroidApplicationConfiguration config) {
	return new AsynchronousAndroidAudio(context, config);
}
```

Generally speaking, Audio on Android is problematic and there may be other scenarios or device especific issues.

## Alternatives

In an attempt to fix some of these issues Google created [Oboe](https://github.com/google/oboe) that can be used on SharpGDX projects thanks to [SharpGDX Oboe](https://github.com/barsoosayque/sharpgdx-oboe).

Another alternative is [MiniAudio](https://miniaud.io/) through [gdx-miniaudio](https://github.com/rednblackgames/gdx-miniaudio) project which is an actively mantained cross-platform audio engine already used in production by some SharpGDX games.

The SharpGDX setup tool gdx-liftoff has options to load SharpGDX-Oboe and gdx-miniaudio.  You'll find these under the Third-Party Section of gdx-liftoff.