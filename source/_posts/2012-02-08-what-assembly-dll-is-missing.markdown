---
layout: post
comments: true
title: "What Assembly (DLL) is Missing?"
categories: Windows .NET
---

Being involved into development of huge enterprise system, while working on distribution package building subsystem, I faced next problem - just deployed application crashed because of missing third-party assembly. Despite I had name of that DLL, I couldn't figure out what's wrong because exactly this DLL was present in the package.

The possible reason for error, obviously, was some another DLL, which was required by original one. So having lots of dependencies pushed me to find right way of investigating such kind of problems - that is using of Fusion Log Viewer (`fuslogvw.exe`) which is usually located in `%Program Files%\Microsoft SDKs\Windows\v7.0A\Bin`.

![Fusion Log Viewer (fuslogvw.exe)](http://ctitarenko.files.wordpress.com/2012/02/fuslogvw.png)

Algorithm is very clear: run viewer, clean current log, run your application and then see what is causing the problem. One note: please don't forget to verify your settings: "Log bind failures to disk" should be chosen.

![Fusion Log Viewer Settings](http://ctitarenko.files.wordpress.com/2012/02/fuslogvwsettings.png)
