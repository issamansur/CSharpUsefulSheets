# CSharpUsefulSheets

Any useful c# practices which I found

## Table of Contents

- [Prepare Project](#prepare-project)
- ASP.NET Core
- -  Authentication
- - - Bearer
- - - - [Add Bearer Service for generating JWT](./ASP.NET/Authentication/Bearer/README.md#1-add-authservice-for-generating-jwt-token)
- - - - [Add Bearer Middleware for validating JWT](./ASP.NET/Authentication/Bearer/README.md#2-add-authentication-middleware-for-validating-jwt-token)
- - - - [Add Bearer Authentication in Swagger](./ASP.NET/Authentication/Bearer/README.md#3-add-bearer-authentication-in-swagger)
- - [Exceptions](./ASP.NET/Exceptions/README.md)
- - - [Exception Filter](./ASP.NET/Exceptions/ExceptionFilter/README.md)
- - - [Exception Handler (or Middleware)](./ASP.NET/Exceptions/ExceptionHandler/README.md)


## Prepare Project

### Startup.cs
```csharp
using Bla.WebApi.Extensions;

var builder = WebApplication.CreateBuilder(args);

builder.Services.ConfigureServices(/*dependencies*/);
// ...
app.ConfigureApp(/*dependencies*/);
// ...
```

### Extensions.cs

```csharp
public static class Extensions 
{
    public static IServiceCollection ConfigureServices(
        this IServiceCollection services
        /*dependencies*/
    )
    {
        // services.AddAnyService();
        
        return services;
    }
    
    public static WebApplication ConfigureApp(
        this WebApplication app
        /*dependencies*/
    )
    {
        // app.UseAnyMiddleware(); or else
        
        return app;
    }
    
    // Private methods
}
```

### P. S.
You can use `ConfigureServices` and `ConfigureApp` methods in `Startup.cs` directly.

Or you can split logic in other layers by `Dependency Injection`.
