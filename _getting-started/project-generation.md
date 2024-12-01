---
title: "Creating a Project"
description: "The SharpGDX extension and templates take care of all the steps involved in setting up a SharpGDX project."
layout: home
nav_order: 1
---

To setup your first project and download the necessary dependencies, SharpGDX offers templates.

{: important}
> The usage of the provided templates is completely optional, but will require a bit of manual setup. It is recommended that you use the provided templates to gain an understanding of how SharpGDX works before attempting to go it alone.

{% include setup_flowchart.html current='1' %}

## 1. Visual Studio

1. Open a command prompt.
2. Run the following command to install the Visual Studio templates.
```
dotnet new install SharpGDX.Templates
```
3. Launch Visual Studio 2022
4. Create a **New Project**.
5. Select the **Templates > Visual C# > SharpGDX** category, and then select **SharpGDX Cross-Platform Desktop Application**. You will be asked for the following:
    * Project Name:  The name of the application. This can contain letters, numbers, underscore and dashes, e.g. `YourProjectName`.
    * Location: The destination folder for the project.
    * Solution Name: May be specified, if desired, if you uncheck **Place solution and project in the same directory**.
6. Click **Create**. Your project will be created and opened.