---
layout:     post
title:      "Returning HTTP status codes from exceptions via OWIN"
date:       2016-02-17 14:44:12 +0000
comments:   true
categories: OWIN ASP.NET C#
---
In the past when writing APIs using ASP.NET, I have used Exceptions as a method of breaking out of the current call chain and returning an error response to the client. I have recently been investigating exception handling in ASP.NET Core 1.0 and how to return a status code for a specific exception.

The "standard" middleware from Microsoft has mechanisms for handling unhandled exceptions and returning a 500 response with re-execution of the failing request on a separate pipeline (see the ExceptionHandler middleware in the Microsoft.AspNetCore.Diagnostics package). This gave me the idea to create my own middleware. Whilst this may not be the most ideal place in the pipeline to perform such functionality it would be a good lesson in creating middleware.

Firstly I needed to create a method of mapping exceptions to a HTTP status code. I created a class that would handle this in a fluent syntax whilst populating a dictionary of exception type to status code. The resultant syntax looks as follows:

{% highlight csharp %}
var transformations = new TransformCollectionBuilder()
  .Return(404)
  .For<CustomerNotFoundException>()
  .Transformations;
{% endhighlight %}

It was then a case of creating a middleware class that wraps the request pipeline in a big try...catch. In the catch block it attempts to map the caught exception to a status code using the configured transformations. If the exception has not been mapped a 500 (Internal Server Error) code is assigned to the response.

The repository for the middleware can be found [here][middleware-repository], feel free to raise an issue or submit a pull request.

[middleware-repository]: https://github.com/dotnetprogrammr/Dnp.AspNetCore.Diagnostics