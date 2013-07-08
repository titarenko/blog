---
layout: post
comments: true
title: "When IList Has Indexer But Does Not Have Enumerator"
categories: .NET DevExpress
---
What can you expect from type implementing `IList`? Surely, availability of enumerator, which give you a power of LINQ! But do not be so fast, sometimes this assumption can be wrong! For example, yesterday I've found that some types in XAF (eXpress Application Framework by DevExpress) can implement `IList` having the indexer but at the same time no enumerator!

Imagine you are implementing handler for some controller's action where you need to access all items from `ListView`:

```csharp
...
var records = listView.CollectionSource.List.Cast<MyType>().ToList();
...
```

Instance `listView.CollectionSource.List` is of type implementing `IList`, so you can expect reference to list of `MyType` instances in the `records` variable, but you'll receive `System.NotSupportedException`!

```
  System.NotSupportedException was unhandled by user code
  Message=Specified method is not supported.
  Source=DevExpress.Xpo.v10.2
  StackTrace:
       at DevExpress.Xpo.Helpers.XpoServerCollectionAdderRemover.GetEnumerator()
       at DevExpress.Xpo.Helpers.XpoServerCollectionWrapperBase.GetEnumerator()
       at System.Linq.Enumerable.<CastIterator>d__aa`1.MoveNext()
       at System.Collections.Generic.List`1..ctor(IEnumerable`1 collection)
       at System.Linq.Enumerable.ToList[TSource](IEnumerable`1 source)
       at MyProject.MyController.HandleMyAction(Object sender, SimpleActionExecuteEventArgs e)
       at DevExpress.ExpressApp.Actions.SimpleAction.RaiseExecute(ActionBaseEventArgs eventArgs)
       at DevExpress.ExpressApp.Actions.ActionBase.ExecuteCore(Delegate handler, ActionBaseEventArgs eventArgs)
```

So, what should you do? Use indexer!

```csharp
...
var records = Enumerable.Range(0, listView.CollectionSource.List.Count).Select(
  index => listView.CollectionSource.List[index] as MyType).ToList();
...
```

Is it so hard for framework developers to implement enumerator if type already has indexer? I think no. So it's very strange that we should bother ourselves with such things. But anyway the solution is not so hard, so generally everything is OK.

*P. S.* Also it's quite strange and unintuitive that we should access items of list view through `listView.CollectionSource.List` instead of `listView.List` or `listView.Items`, especially, when such properties do exist, but intended for another purposes (internal). But that is another story.
