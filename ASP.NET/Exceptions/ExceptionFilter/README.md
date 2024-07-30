# Exceptions / Exception Filter

## 1. Create Exception Response Model

At first, we need to create model for exception response:

### ApiError.cs
```csharp
public class ApiError
{
    public string Message { get; set; }
    public string Detail { get; set; }
    // You can add other properties
}
```

## 2. Add `ExceptionFilter`

Now we can create `ExceptionFilter` with JSON response.
Also, we added `IWebHostEnvironment` for checking environment (Development or Production).

### ApiExceptionFilter.cs
```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;

public class JsonExceptionFilter: IExceptionFilter
{
    IWebHostEnvironment _env;
    
    public JsonExceptionFilter(IWebHostEnvironment env)
    {
        _env = env;
    }
    
    public void OnException(ExceptionContext context)
    {
        var error = new ApiError();
        
        if (_env.IsDevelopment())
        {
            error.Message = context.Exception.Message;
            error.Detail = context.Exception.StackTrace;
        }
        else
        {
            error.Message = "A server error occurred.";
            error.Detail = context.Exception.Message;
        }
        
        context.Result = new ObjectResult(error)
        {
            StatusCode = 500
        };
    }
}
```

## 3. Add `ExceptionFilter` in `Extensions.cs`:

### Extensions.cs
```csharp
using Bla.WebApi;

public static class Extensions 
{
    public static IServiceCollection ConfigureServices(
        this IServiceCollection services
    )
    {
        // ...
        services.AddJsonExceptionFilter(configuration);
    }
    
    // ...
    
    private static IServiceCollection AddJsonExceptionFilter(
        this IServiceCollection services
    )
    {
        // 1. ExceptionFilter
        services.AddMvc(
            options =>
            {
                options.Filters.Add<JsonExceptionFilter>();
            }
        );
        
        // 2. You can add problem details (optional)
        services
            .AddProblemDetails();
        
        return services;
    }
}
```

P.S. We don't need to add `IWebHostEnvironment` in `Extensions.cs`, 
because it will be resolved automatically by DI container.
