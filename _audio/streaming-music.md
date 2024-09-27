---
title: Streaming Music
layout: home
nav_order: 5
---

{: .important }
> This is a copy of the Java documentation for libGDX. It is not finished being ported.

For any sound that's longer than a few seconds it is preferable to stream it from disk instead of fully loading it into RAM. SharpGDX provides an IMusic interface that lets you do that.

To load an IMusic instance we can do the following:

```csharp
var music = Gdx.Audio.NewMusic(GDX.Files.Internal("data/mymusic.wav"));
```

This loads a WAV file called `"mymusic.wav"` from the internal directory `data`.

Playing back the music instance works as follows:

```csharp
music.Play();
```

Of course you can set various playback attributes of the `IMusic` instance:

```csharp
music.SetVolume(0.5f);			// sets the volume to half the maximum volume
music.SetLooping(true);			// will repeat playback until music.stop() is called
music.Stop();				// stops the playback
music.Pause();				// pauses the playback
music.Play();				// resumes the playback
var isPlaying = music.IsPlaying;	// obvious :)
var isLooping = music.IsLooping;	// obvious as well :)
var position = music.Position;		// returns the playback position in seconds
```

`IMusic` instances are heavy on some backends (such as Android), you should usually not have more than about 10 loaded and more than 1 or 2 playing at the same time.

An `IMusic` instance needs to be disposed if it is no longer needed, to free up resources.

```csharp
music.Dispose();
```