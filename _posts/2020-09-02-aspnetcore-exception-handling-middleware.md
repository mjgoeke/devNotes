Working with asp.net core controllers, and all the underlying code they call, it's useful to have a declarative approach in handling exceptions and responses.  
By carving out a few poco exceptions types (e.g. NotFoundException, BadRequestException, ValidationException) the underlying code can use standard throwing mechanisms, but these will just result in 500 server responses.  
It would be valuable to configure mappings for these exception types -> http response type.  

Insert asp.net core middleware.

It looks like `Microsoft.AspNetCore.Diagnostics` has a `ExceptionHandlerMiddleware` class already.  I should be able to wrap the options to have the interface look how I'd like.
