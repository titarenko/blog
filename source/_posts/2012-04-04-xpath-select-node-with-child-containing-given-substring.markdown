---
layout: post
comments: true
title: "XPath: Select Node with Child Containing Given Substring"
categories: XPath
---

XPath is very interesting topic, essentially it allows you to do quite sophisticated things. And in this short post, I'll give example how to create a little more complex than plain query: it will return all books, which contains "C++" in any of its childrens' texts (please, see [example of such XML here](http://msdn.microsoft.com/en-us/library/windows/desktop/ms762271%28v=vs.85%29.aspx)).

So, our query is following:

```
//*[contains(text(), 'C++')]/parent::book
```

It will return:

```xml
<book id="bk112">
    <author>Galos, Mike</author>
    <title>Visual Studio 7: A Comprehensive Guide</title>
    <genre>Computer</genre>
    <price>49.95</price>
    <publish_date>2001-04-16</publish_date>
    <description>Microsoft Visual Studio 7 is explored in depth,
    looking at how Visual Basic, Visual C++, C#, and ASP+ are 
    integrated into a comprehensive development 
    environment.</description>
</book>
```

That's it. Simple. But useful, especially if you haven't worked with XPath too much before.
