---
layout:     post
title:      "Interpreting exceptions as HTTP status codes in MVC"
date:       2016-02-17 14:44:12 +0000
comments:   false
categories: C# ASP.NET
tags:       [C#, ASP.NET, MVC]
---
In a [related post]({% post_url 2016-02-17-returning-http-status-codes-from-exceptions-via-owin %}) I detailed a 
method of catching exceptions then setting the HTTP status code on the response based on the handled exceptions 
type. It relied on creating custom middleware that caught all un-handled exceptions and set the status code on the 
response. I had reservations about the solution whilst writing the middleware and post, I felt that 
middleware was not the right point in the request pipeline to handle exceptions thrown from MVC. Also, if I wanted 
to return "pretty" response bodies I couldn't as I was outside of the MVC request pipeline.

A quick search on the web took me to [this][damienbod-post] helpful post. I had briefly looked at exception filters 
in ASP.NET 4.6 (but had not used them prefering to implement IExceptionHandler) but had not identified them within 
ASP.NET Core. An implementation of my previous post as an exception filter satisifed my grievances with my 
previous solution:

* Handled exceptions thrown from business logic can now be handled separately from 
  those exceptions thrown from the OWIN pipeline.
* As the exception will now be handled within the ASP.NET pipeline "pretty" responses 
  could be returned to the client.

When implementing a custom exception filter it is important to remember to "handle" the exception. By this I mean 
that we need to stop filters/handlers further down the pipeline from receiving the exception. In an exception 
filter this can be done in two ways:

* By setting the Exception property of the ExceptionContext to Null.
* By setting the Result property of the ExceptionContext.

As this implementation is only setting the HTTP status code of the response it will set the Result property to a 
HttpStatusCodeResult instance. The status code passed to the IActionResult instance will be the status code inferred 
from the handled exception. I personally feel that nulling the Exception property is messy and the property should 
not be mutable.

The repository for the filter can be found [here][repository], feel free to raise an issue or submit a pull 
request.

[damienbod-post]: http://damienbod.com/2015/09/30/asp-net-5-exception-filters-and-resource-filters/
[repository]: https://github.com/dotnetprogrammr/Dnp.AspNetCore.Mvc