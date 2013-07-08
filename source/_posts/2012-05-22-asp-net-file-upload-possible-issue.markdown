---
layout: post
comments: true
title: "ASP.NET File Upload Possible Issue"
categories: .NET Web
---

One of issues you can face when developing web application with use of ASP.NET MVC and IIS 7.5 is maximal request size. No doubt, after googling, you'll find that cause is request size limit, so you'll happily add to web.config following line and forget about this.

```xml
<httpRuntime maxRequestLength="100000"/>
```

But if under some circumstances you'll find that controller's action responsible for receiving of large file is not being triggered but instead application returns 404 code, try following solution. Please note, in this case content length should be provided in bytes, not kilobytes like in prevous example.

```xml
<security>
    <requestFiltering>
        <requestLimits maxAllowedContentLength="100000000" />
    </requestFiltering>
</security>
```

For me, while working with Windows integrated authentication, this helped a lot. Thanks to [StackOverflow](http://stackoverflow.com/a/6687932)!
