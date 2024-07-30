# CSharpUsefulSheets

Any useful c# practices which I found

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
