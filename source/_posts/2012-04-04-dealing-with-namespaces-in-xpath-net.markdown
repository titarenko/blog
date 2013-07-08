---
layout: post
comments: true
title: "Dealing with Namespaces in XPath (.NET)"
categories: .NET XPath
---

This post is kind of short but useful note for those who are dealing with namespaces in XPath, especially if there is default namespace (with empty prefix).

Normally, when doing queries to document *with namespaces*, you involve `XmlNamespaceManager` like this:

```csharp
var namespaceManager = new XmlNamespaceManager(document.NameTable);

var namespaces = document
    .DocumentElement
    .Attributes
    .Cast<XmlAttribute>()
    .Where(x => x.Name.StartsWith("xmlns"))
    .Select(x => new
        {
            Prefix = x.Name.Length > 5
                            ? x.Name.Substring(6) // case when name is like "xmlns:something"
                            : string.Empty, // case when name is "xmlns" - default namespace
            Uri = x.Value
        });

foreach (var ns in namespaces)
{
    namespaceManager.AddNamespace(ns.Prefix, ns.Uri);
}

var nodes = document.SelectNodes("//node/subnode", namespaceManager);
```

But wait! This query *will return no results*! The reason is not so obvious: you *must use prefix for default namespace*, it can't be omitted! So, to make such queries working, we need to change the initial code as well as XPath query:

```csharp
...
            Prefix = x.Name.Length > 5
                            ? x.Name.Substring(6) // case when name is like "xmlns:something"
                            : "default", // case when name is "xmlns" - default namespace
            Uri = x.Value
...
var nodes = document.SelectNodes("//default:node/default:subnode", namespaceManager);
```

Good, now it's OK, we've got our results.
