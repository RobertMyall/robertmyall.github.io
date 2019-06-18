---
layout: post
title:  "Azure DevOps Yaml Pipeline and .Net Core"
date:   2019-06-18 21:00:00 +0100
categories: dev
tags: [azure, devops, nuget, dotnetcore]
---

I ran into a minor issue with Azure DevOps when using the standard yaml pipeline to build .Net Core applications. In this case my application was compiled against .Net Core 2.1 but when the DevOps pipeline ran I got the following error, which suggested that it was trying to build against .Net Core 1.0.


> Error : NETSDK1061: The project was restored using Microsoft.NETCore.App version 1.0.0, but with current settings, version 2.1.0 would be used instead.

The first task in the default Azure Pipelines yaml installs the correct version of NuGet, so that it can restore the solution correctly. For some reason the default task (at least for me) was using version 4.3.0 which you can see in the following snippet from the NuGetToolInstaller task output:

> Found tool in cache: NuGet 4.3.0 x64  
  Resolved from tool cache: 4.3.0  
  Using version: 4.3.0  

If you look at the [NuGet Tool Installer Task documentation][nuget-docs] you can see that it is possible to add a version spec to the task, which allows for either a specific version (e.g: 5.1.0) or a minimum (e.g: >=5.0). In this case I'd rather specify a minimum and allow the CI to choose the most up to date version. If a future version of nuget breaks my project then that is something I'd like to know about and either fix or have the option to choose a static version at that point. You can change the yml task to something similar to the following: 

```yaml
- task: NuGetToolInstaller@0
  inputs:
    versionSpec: '>=5.0'
```

After that is committed, and built I now get the following output from the NugetToolInstaller task, helpfully warning me that this might break in the future: 

> You are using a query match on the version string. Behavior changes or breaking changes might occur as NuGet updates to a new version.  
  Downloading: https://dist.nuget.org/win-x86-commandline/v5.1.0/nuget.exe  
  Caching tool: NuGet 5.1.0 x64  
  Using version: 5.1.0  
  Found tool in cache: NuGet 5.1.0 x64  
  Using tool path: C:\hostedtoolcache\windows\NuGet\5.1.0\x64  


But more importantly the build no longer complains about being built against the wrong framework.

[nuget-docs]: https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/tool/nuget?view=azure-devops