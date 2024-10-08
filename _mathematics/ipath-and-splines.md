---
title: "IPath and Splines"
layout: home
nav_order: 2
---

{: .important }
> This is a copy of the Java documentation for libGDX. It is not finished being ported.

The [IPath interface](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/math/Path.html) [(code)](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/math/Path.html) has implementations that allows you to traverse smoothly through a set of defined points (in some cases, tangents too).

Paths can be defined to be bi-dimensional or tri-dimensional, because it is a template that takes a derived of the Vector class, thus you can either use it with a set of Vector2's or Vector3's.

Mathematically, it is defined as F(t) where t is [0, 1], where 0 is the start of the path, and 1 is the end of the path.

## Types

There are currently 3 implementations of the IPath interface (the splines), these are:
* [Bezier](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/math/Bezier.html) [(code)](https://github.com/sharpgdx/sharpgdx/blob/master/gdx/src/com/badlogic/gdx/math/Bezier.java)
* [BSpline](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/math/BSpline.html) [(code)](https://github.com/sharpgdx/sharpgdx/blob/master/gdx/src/com/badlogic/gdx/math/BSpline.java)
* [CatmullRomSpline](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/math/CatmullRomSpline.html) [(code)](https://github.com/sharpgdx/sharpgdx/blob/master/gdx/src/com/badlogic/gdx/math/CatmullRomSpline.java)

## Use

Splines can be used statically like this:

```csharp
    var o = new Vector2();
    var tmp = new Vector2();
    var dataSet = new Vector2[size];

    /* fill dataSet with path points */
    CatmullRomSpline.Calculate(o, t, dataSet, continuous, tmp); // stores in the vector o the point of the catmullRom path of the dataSet in the time t. Uses tmp as a temporary vector. if continuous is true, the path is a loop.
    CatmullRomSpline.Derivative(o, t, dataSet, continuous, tmp);    // the same as above, but stores the derivative of the time t in the vector o.
```

or they can be stored like this:

```csharp
    var myCatmull = new CatmullRomSpline<Vector2>(dataSet, true);
    var o = new Vector2();
    
    myCatmull.ValueAt(o, t);
    myCatmull.DerivativeAt(o, t);
```

It is preferred that you do the second way, since it's the only way guaranteed by the Path interface.

## Snippets

### Caching a Spline

This is often done at loading time, where you load the data set, calculates the spline and stores the points calculated. So you only evaluates the spline once. Trading memory for CPU time. (You usually should always cache if drawing static things)

```csharp
/*members*/
    private int _k = 100;   //increase _k for more fidelity to the spline
    private Vector2 _points = new Vector2[k];

/*Init()*/
    var myCatmull = new CatmullRomSpline<Vector2>(dataSet, true);
    
    for(var i = 0; i < k; ++i)
    {
        _points[i] = new Vector2();
        myCatmull.valueAt(_points[i], ((float)i) / ((float)_k - 1));
    }
```

### Rendering cached spline

How to render the spline previously cached

```csharp
/*members*/
    private int _k = 100;   // increase _k for more fidelity to the spline
    private Vector2[] _points = new Vector2[_k];
    private ShapeRenderer _shapeRenderer;

/*Render()*/
    _shapeRenderer.Begin(ShapeType.Line);
    
    for (int i = 0; i < _k - 1; ++i)
    {
        _shapeRenderer.Line(_points[i], _points[i + 1]);
    }

    _shapeRenderer.End();
```

### Calculating on the fly

Do everything at render stage

```csharp
/*members*/
    private int _k = 100;    //increase k for more fidelity to the spline
    private CatmullRomSpline<Vector2> _myCatmull = new CatmullRomSpline<Vector2>(dataSet, true);
    private Vector2 _out = new Vector2();
    private ShapeRenderer _shapeRenderer;

/*render()*/
    _shapeRenderer.begin(ShapeType.Line);
    
    for (int i = 0; i < _k - 1; ++i)
    {
        _shapeRenderer.line(_myCatmull.valueAt(_points[i], ((float)i)/((float)_k - 1)), _myCatmull.valueAt(_points[i+1], ((float)(i + 1))/((float)_k - 1)));
    }

    _shapeRenderer.end();
```

### Make sprite traverse through the cached path

This way uses a LERP through the cached points. It looks roughly sometimes but is very fast.

```java
/*members*/
    float speed = 0.15f;
    float current = 0;
    int k = 100; //increase k for more fidelity to the spline
    Vector2[] points = new Vector2[k];

/*render()*/
    current += Gdx.graphics.getDeltaTime() * speed;
    if(current >= 1)
        current -= 1;
    float place = current * k;
    Vector2 first = points[(int)place];
    Vector2 second;
    if(((int)place+1) < k)
    {
        second = points[(int)place+1];
    }
    else
    {
        second = points[0]; //or finish, in case it does not loop.
    }
    float t = place - ((int)place); //the decimal part of place
    batch.draw(sprite, first.x + (second.x - first.x) * t, first.y + (second.y - first.y) * t);
```

### Make sprite traverse through path calculated on the fly

Calculate sprite position on the path every frame, so it looks much more pleasant (you usually should always calculate on the fly if drawing dynamic things)

```java
/*members*/
    float speed = 0.15f;
    float current = 0;
    Vector2 out = new Vector2();
/*render()*/
    _current += GDX.Graphics.GetDeltaTime() * _speed;
    
    if (current >= 1)
    {
        current -= 1;
    }

    _myCatmull.valueAt(_out, _current);
    batch.draw(sprite, _out.x, _out.y);
```

### Make sprite look at the direction of the spline

The angle can be found when applying the atan2 function to the normalised tangent(derivative) of the curve.

```csharp
    _myCatmull.derivativeAt(_out, _current);
    float angle = _out.angle();
```

### Make the sprite traverse at constant speed

As the arc-length of the spline through the dataSet points is not constant, when going from 0 to 1 you may notice that the sprite sometimes goes faster or slower, depending on some factors. To cancel this, we will change the rate of change of the time variable.
We can easily do this by dividing the speed by the length of the rate of change.
Instead of

```csharp
    _current += GDX.Graphics.GetDeltaTime() * _speed;
```

change to:

```java
    myCatmull.derivativeAt(out, current);
    current += (Gdx.graphics.getDeltaTime() * speed / myCatmull.spanCount) / out.len();
```

You should change the speed variable too, since it doesn't take a "percent per second" value anymore, but a "meter(pixel?) per second") now. The spanCount is necessary since the derivativeAt method takes into account the current span only.