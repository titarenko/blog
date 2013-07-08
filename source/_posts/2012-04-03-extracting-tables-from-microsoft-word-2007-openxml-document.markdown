---
layout: post
comments: true
title: "Extracting Tables from Microsoft Word 2007+ (OpenXML) Document"
categories: .NET MSOffice DataExtraction XPath
---
Let's suppose we have huge Word 2007+ document (*.docx) with lots of tables and we want to extract that information for further processing. Since our document is based on OpenXML format I would say "No problem! Let's do it!".

First, we need to open zip archive (each *.docx is zipped set of XML files). For this purpose we can use great library DotNetZip (just use NuGet to obtain it). Right after this we'll read word/document.xml entry.

```csharp
static void Main(string[] args)
{
    using (var file = ZipFile.Read(args[0]))
    using (var inputStream = file["word/document.xml"].OpenReader())
    {
        var document = new XmlDocument();
        document.Load(inputStream);

        var tables = ExtractTables(document);

        using (var outputStream = File.Create(args[1]))
        {
            new XmlSerializer(typeof(Table[])).Serialize(outputStream, tables);
        }
    }
}
```

Then we'll extract neccessary pieces using XPath: our target is `w:tbl` nodes (tables) with such descendants as `w:tr` (rows), `w:tc` (cells) and `w:t` (text pieces).

```csharp
private static Table[] ExtractTables(XmlNode document)
{
    return document.Select("//w:tbl", tableNode => new Table
    {
        Rows = tableNode.Select(".//w:tr", rowNode => new Row
        {
            Cells = rowNode.Select(".//w:tc", cellNode => new Cell
            {
                Text = cellNode.Select(".//w:t", textNode => textNode.InnerText)
            })
        })
    });
}
```

Classes `Table`, `Row` and `Cell` are simple DTOs used for simplifying of result serialization.

One more thing is `.Select()` extension method - it's implemented in the following way:

```csharp
public static T[] Select<T>(this XmlNode node, string xpath, Func<XmlNode, T> selector)
{
    return node
        .SelectNodes(xpath, GetOrCreateNamespaceManager((node as XmlDocument ?? node.OwnerDocument).NameTable))
        .Cast<XmlNode>()
        .Select(selector)
        .ToArray();
}
```

Method .GetOrCreateNamespaceManager() returns namespace manager as it's required for making of XPath queries with use of namespaces. There is only one parameter for it - name table instance.

```csharp
private static XmlNamespaceManager GetOrCreateNamespaceManager(XmlNameTable nameTable)
{
    if (manager == null)
    {
        manager = new XmlNamespaceManager(nameTable);
        manager.AddNamespace("w", "http://schemas.openxmlformats.org/wordprocessingml/2006/main");
    }

    return manager;
}
```

That's it! Due to openness of new Word document's format our initial task is so simple in implementation that it takes no more than couple of hours to implement it.
