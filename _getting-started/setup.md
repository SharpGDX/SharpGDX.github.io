---
title: "Setup an IDE"
description: "Before you can get your first SharpGDX project up and running, you need to set up your development environment. The first step in doing this is choosing an IDE: Visual Studio or Rider are among the most common choices for this."
layout: home
nav_order: 0
---

{: .important }
> This is a copy of the Java documentation for libGDX. It is not finished being ported.

If this is your first time using SharpGDX, you're at the right place. The following steps detail how you can get your first SharpGDX project up and running.

{% include setup_flowchart.html current='0' %}

Before you can get started with SharpGDX, you need to set up an IDE (Integrated Development Environment). It is basically an editor for your code files, which makes developing applications considerably more convenient in various ways. **If you already have an IDE installed, you can skip to the next [step](/documentation/start/project-generation).**

There are several IDEs available for C# development, the most common being Visual Studio and Rider, both which have free versions for open source or non-commercial projects.

## 1. Visual Studio

- SDK: is provided by Visual Studio
- IDE: [Visual Sutdio](https://www.visualstudio.com/downloads/) (the "Community" edition is sufficient)

## 2. Rider

- SDK: [.NET 8.0](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) (Build Apps - SDK, not runtime)
- IDE: [Rider](https://www.jetbrains.com/rider/download/)

{: .note}
> Rider has a 30 day free trial license, but after that you will need to purchase one to continue using it. It is worth trying to see how it compares to Visual Studio and Visual Studio Code.

## 3. Visual Studio Code

- SDK: [.NET 8.0](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) (Build Apps - SDK, not runtime)
- IDE: [Visual Studio Code](https://code.visualstudio.com/)

## 4. No IDE

It is possible to develop sharpGDX applications entirely without an iDE, just using a simple editor like Notepad. This approach is **not** recommended, and will not be outlined here.


<br/>

**Now that you have a development environment, you can create your very first SharpGDX project. SharpGDX offers templates and NuGet packages for that, which makes getting started easy. To get started with it, take a look [here](/getting-started/project-generation).**