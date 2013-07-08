---
layout: post
comments: true
title: "Snapping To Grid"
categories: .NET Snippet Math
---
Let's suppose user selects some value (real number) and you need to snap it to nearest point on discrete axis. No problem!

```csharp
public double SnapValue(double value, double step)
{
    var remainder = Math.IEEERemainder(value, step);
    return step/2 > remainder ? value - remainder : value + remainder;
}
```

So, assuming your you have an axis with step 0.5, value 10.2 will be snapped to 10 (closest node), and value 10.78, in turn, - to 11.
