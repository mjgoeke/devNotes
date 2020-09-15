Working with asp.net core controllers, and all the underlying code they call, it's useful to have a declarative approach in handling exceptions and responses.  
By carving out a few poco exceptions types (e.g. NotFoundException, BadRequestException, ValidationException) the underlying code can use standard throwing mechanisms, but these will just result in 500 server responses.  
It would be valuable to configure mappings for these exception types -> http response type.  

Insert asp.net core middleware.

It looks like `Microsoft.AspNetCore.Diagnostics` has a `ExceptionHandlerMiddleware` class already.  I should be able to wrap the options to have the interface look how I'd like.


This worked, but if `UseDeveloperExceptionPage` is called anywhere it will override the other ExceptionHandler settings, regardless which is called first/last.

ended up with extension method that looked like this:
``` c#
public static IApplicationBuilder UseExceptionHandlerMapping(this IApplicationBuilder app, Func<Exception, int?> mapping)
{
  return app.UseExceptionHandler(new ExceptionHandlerOptions
  {
    ExceptionHandler = async context =>
    {
      var exceptionHandler = context.Features.Get<IExceptionHandlerFeature>();
      var e = exceptionHandler.Error;
      var statusCode = mapping(e);
      if (statusCode == null) return;

      context.Response.ContentType = "text/plain";
      context.Response.StatusCode = statusCode.Value;
      await context.Response.WriteAsync(e.Message);
    }
  });
}
```

for the `Func<Exception, int?> mapping` function, c# 8 recursive pattern matching is really slick:
``` c#
app.UseExceptionHandlerMapping(e => e switch
  {
	    DomainValidationException _ => StatusCodes.Status400BadRequest,
	    ValidationException _       => StatusCodes.Status400BadRequest,
	    NotFoundException _         => StatusCodes.Status404NotFound,
	    _ => null
  });
```
