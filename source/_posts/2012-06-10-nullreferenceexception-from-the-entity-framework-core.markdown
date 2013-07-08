---
layout: post
comments: true
title: "NullReferenceException from the Entity Framework Core"
categories: .NET ORM
---

Running web application with Entity Framework 4.3 I got very strange (at first glance) exception saying: `"Object reference not set to an instance of an object."`. Following is the controller action caused this error:

```csharp
public ActionResult Index()
{
    return View(repository.Query<Story>().ToList());
    // repository.Query<Story>() here is nothing but (DbQuery<Story>) context.Stories
}
```

Here is the stack trace:

```
[NullReferenceException: Object reference not set to an instance of an object.]
   System.Data.Objects.ELinq.QueryParameterExpression.EvaluateParameter(Object[] arguments) +83
   System.Data.Objects.ELinq.ELinqQueryState.GetExecutionPlan(Nullable`1 forMergeOption) +779
   System.Data.Objects.ObjectQuery`1.GetResults(Nullable`1 forMergeOption) +149
   System.Data.Objects.ObjectQuery`1.System.Collections.Generic.IEnumerable<T>.GetEnumerator() +44
   System.Data.Entity.Internal.Linq.InternalQuery`1.GetEnumerator() +40
   System.Data.Entity.Infrastructure.DbQuery`1.System.Collections.Generic.IEnumerable<TResult>.GetEnumerator() +40
   System.Collections.Generic.List`1..ctor(IEnumerable`1 collection) +315
   System.Linq.Enumerable.ToList(IEnumerable`1 source) +58
   QTeam.PlanningBoard.Controllers.StoriesController.Index() in ...QTeam.PlanningBoard\Controllers\StoriesController.cs:26
   lambda_method(Closure , ControllerBase , Object[] ) +97
   System.Web.Mvc.ActionMethodDispatcher.Execute(ControllerBase controller, Object[] parameters) +17
   System.Web.Mvc.ReflectedActionDescriptor.Execute(ControllerContext controllerContext, IDictionary`2 parameters) +208
   System.Web.Mvc.ControllerActionInvoker.InvokeActionMethod(ControllerContext controllerContext, ActionDescriptor actionDescriptor, IDictionary`2 parameters) +27
   System.Web.Mvc.<>c__DisplayClass15.<InvokeActionMethodWithFilters>b__12() +55
   System.Web.Mvc.ControllerActionInvoker.InvokeActionMethodFilter(IActionFilter filter, ActionExecutingContext preContext, Func`1 continuation) +263
   System.Web.Mvc.<>c__DisplayClass17.<InvokeActionMethodWithFilters>b__14() +19
   System.Web.Mvc.ControllerActionInvoker.InvokeActionMethodWithFilters(ControllerContext controllerContext, IList`1 filters, ActionDescriptor actionDescriptor, IDictionary`2 parameters) +191
   System.Web.Mvc.ControllerActionInvoker.InvokeAction(ControllerContext controllerContext, String actionName) +343
   System.Web.Mvc.Controller.ExecuteCore() +116
   System.Web.Mvc.ControllerBase.Execute(RequestContext requestContext) +97
   System.Web.Mvc.ControllerBase.System.Web.Mvc.IController.Execute(RequestContext requestContext) +10
   System.Web.Mvc.<>c__DisplayClassb.<BeginProcessRequest>b__5() +37
   System.Web.Mvc.Async.<>c__DisplayClass1.<MakeVoidDelegate>b__0() +21
   System.Web.Mvc.Async.<>c__DisplayClass8`1.<BeginSynchronous>b__7(IAsyncResult _) +12
   System.Web.Mvc.Async.WrappedAsyncResult`1.End() +62
   System.Web.Mvc.<>c__DisplayClasse.<EndProcessRequest>b__d() +50
   System.Web.Mvc.SecurityUtil.<GetCallInAppTrustThunk>b__0(Action f) +7
   System.Web.Mvc.SecurityUtil.ProcessInApplicationTrust(Action action) +22
   System.Web.Mvc.MvcHandler.EndProcessRequest(IAsyncResult asyncResult) +60
   System.Web.Mvc.MvcHandler.System.Web.IHttpAsyncHandler.EndProcessRequest(IAsyncResult result) +9
   System.Web.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute() +8970061
   System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously) +184
```

Reading such stack trace can mislead you that cause is not in app code but somewhere in the EF itself. This is because lazy enumeration - after a little bit deeper analysis I found that I have such code:

```csharp
/// <summary>
/// Queries data for current tenant.
/// </summary>
private IQueryable<T> QueryFilteredInternal<T>(params Expression<Func<T, object>>[] eagerlyLoadedProperties) where T : class, IIdentifiable, IOwned
{
    return repository
        .Query(eagerlyLoadedProperties)
        .Where(x => x.OwnerId == userService.CurrentUser.Id);
}
```

Here `userService.CurrentUser` was null, so when EF tried to access `userService.CurrentUser.Id` for providing parameter value to query, `NullReferenceException` was occuring. 

So, summary is: don't fully rely on call stack investigating cause for exceptions while working with lazy evaluated things.

*P. S.* In the early beginning first thing I did was googling for stack trace. It gave me nothing but [links to failing sites](https://www.google.com.ua/search?sugexp=chrome,mod=7&sourceid=chrome&ie=UTF-8&q=at+System.Data.Objects.ELinq.QueryParameterExpression.EvaluateParameter%28Object%5B%5D+arguments%29at+System.Data.Objects.ELinq.ELinqQueryState.GetExecutionPlan%28Nullable%601+forMergeOption%29at+System.Data.Objects.ObjectQuery%601.GetResults%28Nullable%601+forMergeOption%29at+System.Data.Objects.ObjectQuery%601.System.Collections.Generic.IEnumerable%3CT%3E.GetEnumerator%28%29at+System.Data.Entity.Internal.Linq.InternalQuery%601.GetEnumerator%28%29at+System.Data.Entity.Infrastructure.DbQuery%601.System.Collections.Generic.IEnumerable%3CTResult%3E.GetEnumerator%28%29at+System.Collections.Generic.List%601..ctor%28IEnumerable%601+collection%29at+System.Linq.Enumerable.ToList%5BTSource%5D%28IEnumerable%601+source%29) and not to articles with solutions, as was expected, so I hope this post will be helpful for someone who will issue similar search query.
