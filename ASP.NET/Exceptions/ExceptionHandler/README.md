# Exceptions / Exception Handler (or Middleware)

## 1. Create Exception Response Model

At first, we need to create model for exception response:

### ErrorResponse.cs
```csharp
public class ErrorResponse
{
    public string Title { get; set; }
    public int StatusCode { get; set; }
    public string Message { get; set; }
    // You can add other properties
}
```

## 2. Add `ExceptionHandler` Middleware

Now we can create `ExceptionHandler` middleware:

### MyExceptionHandler.cs
```csharp
using System.Net;
using Microsoft.AspNetCore.Diagnostics;

public class MyExceptionHandler: IExceptionHandler
{ 
    private readonly ILogger<MyExceptionHandler> _logger;
    
    public MyExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        ArgumentNullException.ThrowIfNull(logger, nameof(logger));
        
        _logger = logger;
    }
    
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError($"An error occurred while processing your request: {exception.Message}");
        
        var errorResponse = new ErrorResponse
        {
            Message = exception.Message
        };
        
        switch (exception)
        {
            case FileNotFoundException:
                errorResponse.StatusCode = (int)HttpStatusCode.NotFound;
                errorResponse.Title = exception.GetType().Name;
                break;
            case ArgumentNullException:
            case ApplicationException:
            case InvalidOperationException:
            // Add other exceptions
                errorResponse.StatusCode = (int)HttpStatusCode.BadRequest;
                errorResponse.Title = exception.GetType().Name;
                break;
            // Add other cases
            default:
                errorResponse.StatusCode = (int)HttpStatusCode.InternalServerError;
                errorResponse.Title = "Internal Server Error";
                break;
        }
        
        httpContext.Response.StatusCode = errorResponse.StatusCode;
        httpContext.Response.ContentType = "application/problem+json";
        
        await httpContext
            .Response
            .WriteAsJsonAsync(errorResponse, cancellationToken);
        
        return true;
    }
}
```

## 3. Add `ExceptionHandler` in `Extensions.cs`:

And now configure it in `Extensions.cs` file.

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
        services.AddMyExceptionHandler(configuration);
    }
    
    // ...
    
    private static IServiceCollection AddMyExceptionHandler(
        this IServiceCollection services
    )
    {
        // 1. MyExceptionHandler
        services
            .AddExceptionHandler<MyExceptionHandler>();
        
        // 2. You can add problem details (optional)
        services
            .AddProblemDetails();
        
        return services;
    }
}
```
