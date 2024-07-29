# Authentication / Bearer (JWT)

### Startup.cs
```csharp
using Bla.WebApi.Extensions;

var builder = WebApplication.CreateBuilder(args);

builder.Services.ConfigureServices(builder.Configuration);
// ...
app.ConfigureApp();
```

### Extensions.cs

```csharp
public static class Extensions 
{
    public static IServiceCollection ConfigureServices(
        this IServiceCollection services,
        IConfiguration configuration
    )
    {
        // services.AddAnyService();
        
        return services;
    }
    
    public static WebApplication ConfigureApp(
        this WebApplication app
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

<hr>

## Add `AuthService` for generating JWT Token

We imagine that we have `Account` model. For example, like this:

### Account.cs
```csharp
public class Account
{
    public string UserId { get; set; }
    // Other properties
    // ...
}
```

And also we need to create `JwtOptions` class for JWT token options.

### JwtOptions.cs
```csharp
public class JwtOptions
{
    public string Issuer { get; set; }
    public string Audience { get; set; }
    
    public int ExpireHours { get; set; }

    // 32+ characters
    public string Key { get; set; }
}
```

P.S. We need to add `JwtOptions` in `appsettings.json` file.
For example, like this:

### appsettings.json
```json
{
    "JwtOptions": {
        "Issuer": "ExampleProjectServer",
        "Audience": "ExampleProjectClient",
        "ExpireHours": 1,
        "Key": "12345678901234567890123456789012"
    }
}
```

When we create `IAuthService` interface and `AuthService` class, 
we can generate JWT token for `Account` model.

### IAuthService.cs
```csharp
public interface IAuthService
{
    string GenerateToken(Account account);
}
```

### JWTService.cs
```csharp
public class JwtService: IAuthService
{
    private readonly JwtOptions _options;
    
    public JwtService(IOptions<JwtOptions> options)
    {
        _options = options.Value;
    }
    
    public string GenerateToken(Account account)
    {
        var claims = new List<Claim>
        {
            new Claim(CustomClaimTypes.UserId, account.UserId.ToString())
        };
        
        var jwtToken = new JwtSecurityToken
        (
            issuer: _options.Issuer,
            audience: _options.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddHours(_options.ExpireHours),
            signingCredentials: new SigningCredentials
            (
                key: new SymmetricSecurityKey(Encoding.ASCII.GetBytes(_options.Key)), 
                algorithm: SecurityAlgorithms.HmacSha256
            )
        );

        return new JwtSecurityTokenHandler().WriteToken(jwtToken);
    }
}

public static class CustomClaimTypes
{
    public const string UserId = "UserId";
    // You can use other claims, for example:
    // public const string UserName = "UserName";
}
```

And now configure all in `Extensions.cs` file.

### Extensions.cs
```csharp
using Bla.WebApi.Services;
using Microsoft.Extensions.Configuration;
using Microsoft.IdentityModel.Tokens;

public static class Extensions 
{
    public static IServiceCollection ConfigureServices(
        this IServiceCollection services,
        IConfiguration configuration
    )
    {
        // ...
        services.AddJwtAuth(configuration);
    }
    
    // ...
    
    private static IServiceCollection AddJwtAuth(
        this IServiceCollection services,
        IConfiguration configuration
    )
    {
        // 1. AuthService (generate JWT)
        
        // Configure JWT Options
        services
            .AddOptions<JwtOptions>()
            .Bind(configuration.GetSection(nameof(JwtOptions)));
        
        // Add JWT Service
        services
            .AddScoped<IAuthService, JwtService>();
        
        return services;
    }
}
```

## Add Authentication Middleware for validating JWT Token

### Extensions.cs
```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

public static class Extensions 
{
    public static IServiceCollection ConfigureServices(
        this IServiceCollection services,
        IConfiguration configuration
    )
    {
        // ...
        services.AddJwtAuth(configuration);
        
        return services;
    }
    
    public static WebApplication ConfigureApp(
        this WebApplication app
    )
    {
        // ...
        app.UseJwtAuth();
        
        return app;
    }
    
    // ...
    
    private static IServiceCollection AddJwtAuth(
        this IServiceCollection services,
        IConfiguration configuration
    )
    {
        // 1. AuthService (generate JWT)
        // ...
        
        // 2. Add Authentication Middleware (validate JWT)
        
        // Get JWT Options as instance
        var jwtOptions = configuration.GetSection(nameof(JwtOptions)).Get<JwtOptions>();
        
        // Add Authentication Middleware
        services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                options.RequireHttpsMetadata = false;
                options.TokenValidationParameters = new TokenValidationParameters
                {
                    ValidateIssuer = true,
                    ValidIssuer = jwtOptions.Issuer,

                    ValidateAudience = true,
                    ValidAudience = jwtOptions.Audience,

                    ValidateLifetime = true,

                    IssuerSigningKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(jwtOptions.Key)),
                    ValidateIssuerSigningKey = true,
                };
            }
        );
        
        return services;
    }
    
    private static WebApplication UseJwtAuth(this WebApplication app)
    {
        app.UseAuthentication();
        app.UseAuthorization();
        
        return app;
    }
}
```

## Add Bearer Authentication in Swagger

### Extensions.cs
```csharp
using Microsoft.OpenApi.Models;

public static class Extensions 
{
    public static IServiceCollection ConfigureServices(this IServiceCollection services)
    {
        // ...
        services.AddBearerAuthToSwagger();
        
        return services;
    }
    
    // ...
    
    private static IServiceCollection AddBearerAuthToSwagger(this IServiceCollection services)
    {
        services.AddSwaggerGen(
            (option) =>
            {
                option.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
                {
                    In = ParameterLocation.Header,
                    Description = "Please enter a valid token",
                    Name = "Authorization",
                    Type = SecuritySchemeType.Http,
                    BearerFormat = "JWT",
                    Scheme = "Bearer"
                });
                option.AddSecurityRequirement(new OpenApiSecurityRequirement
                {
                    {
                        new OpenApiSecurityScheme
                        {
                            Reference = new OpenApiReference
                            {
                                Type=ReferenceType.SecurityScheme,
                                Id="Bearer"
                            }
                        },
                        new string[]{}
                    }
                });
            }
        );
        
        return services;
    }
}
```
