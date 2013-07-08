---
layout: post
title: "ASP.NET MVC - Preparing for Deployment"
categories: .NET Deployment
---

Development process of several of my pet projects reached stage when robust deployment process is needed. So, I decided to consider list of options and started googling.

I found that msbuild itself allows to do packaging for deployment (but very few developers were recommending to use it for real-life deployment).

Other option was use of NAnt - but disadvantages were that it needs to write configuration in XML, which is quite verbose way of expressing things, and also it has little or no development activity as for now.

But then I spotted [psake](https://github.com/psake/psake) which seemed to be like rake, but for PowerShell, not Ruby. I even read nice post with real life example from [Ayende](http://ayende.com/blog/4156/on-psake").

So, decision was made - packaging will be done by script written in PowerShell. Advantages:

- no need in additional interpreter/dependencies (as in case with rake)
- ability to use .NET types from script
- quite expressive language and decent syntax
- no need to write tons of XML

So, for one of projects, I even didn't use psake and expressed everything in plain PowerShell with only dependency on 7-zip for creating zip archive and it's only few lines of clean and understandable code - you can look at it [here, on Github](https://github.com/titarenko/ClientManager/blob/master/BinaryStudio.ClientManager.WebUi/Deployment/Package.ps1) (it's well commented, so it's good place to start if you haven't written such things before).
