---
layout: post
comments: true
title: "Cookie Aware WebClient (.NET)"
categories: .NET Web Snippet
---
If you've used WebClient to access protected content on some site, then you probably had dealt with `403` error `"Not authorized"`. In case if that site has used cookies for persisting of authentication state then most probable reason is that WebClient doesn't support managing cookies by default. However, this is a matter of few lines of code to implement the solution which is given above.

```csharp
/// <summary>
/// Web client with cookies enabled.
/// </summary>
class CookieAwareWebClient : WebClient
{
    private readonly CookieContainer container = new CookieContainer();

    protected override WebRequest GetWebRequest(Uri address)
    {
        var request = base.GetWebRequest(address);
        if (request is HttpWebRequest)
        {
            (request as HttpWebRequest).CookieContainer = container;
        }

        return request;
    }

    protected override WebResponse GetWebResponse(WebRequest request)
    {
        var response = base.GetWebResponse(request);
        if (response is HttpWebResponse)
        {
            container.Add((response as HttpWebResponse).Cookies);
        }

        return response;
    }
}
```
