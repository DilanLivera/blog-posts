# HTTP logging in ASP.Net Core

![cover-image](./cover-image.jpg)

I have played around with .Net 6 for the past few days and remembered [this](https://twitter.com/davidfowl/status/1388201662364348416) tweet I saw a while back from [David Fowler](https://twitter.com/davidfowl) regarding new HTTP logging middleware in ASP.Net Core. Before .Net 6, I created an HTTP request and response logging middleware template, which I can modify and use when I need it. But having HTTP logging as part of the ASP.Net Core framework is super convenient. Let see how we can add HTTP logging to an ASP.Net Core application.

First, we need to add the middleware to our web application. We used to add middleware to our application in the `Configure` method in the `Startup.cs` file.

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    //Enable HTTP logging
    //Where you place the middleware matters.
    //For example, if you place this middleware after UseHttpsRedirection **middleware
    //you would not get any logs if the request are made with http scheme**
    app.UseHttpLogging();

    app.UseHttpsRedirection();

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

If you are using .NET 6, then we could do this in the `Program.cs` file

```csharp
//.Net 6 API with Controllers
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

//Enable HTTP logging
//Where you place the middleware matters.
//For example, if you place this middleware after UseHttpsRedirection **middleware
//you would not get any logs if the request are made with http scheme**
app.UseHttpLogging();

app.UseHttpsRedirection();

app.MapControllers();

app.Run();
```

According to **[HTTP Logging in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/http-logging/) document**

> By default, HTTP Logging logs common properties such as path, status-code, and headers for requests and responses. The output is logged as a single message at `LogLevel.Information`.

![Logs without configuring the `UseHttpLogging` middleware](./logs-without-configuring-the-http-logging-middleware.png)

Logs without configuring the `UseHttpLogging` middleware.

If we need to configure HTTP logging, first, we need to add the `AddHttpLogging` service to the service collection of our web application. Then we can use `HttpLoggingOptions` to customize the logging.

```csharp
builder.Services.AddHttpLogging(options =>
{
    options.LoggingFields = HttpLoggingFields.RequestProperties;
});
```

![Logs when the `LoggingFields` option is `RequestProperties`.](./logs-when-logging-fields-option-is-request-properties.png)

Logs when the `LoggingFields` option is `RequestProperties`.

You can find a list of lodging options at [LoggingFields](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/http-logging#loggingfields).

Please note, when you create an ASP.Net Core web project in Visual Studio(or using dotnet CLI) using a template, logging for `Microsoft.AspNetCore` is `Warning` in `appsettings.json` and `appsettings.Development.json` files. Because `Warning` is a higher level than `Information`, you will not see any logs from HTTP logging middleware until you change the logging level of `Microsoft.AspNetCore` to `Information`. If you want to keep the setting as it is, you could add `"Microsoft.AspNetCore.HttpLogging": "Information"` to `LogLevel` section to get the logs from HTTP logging middleware.

You can find the HTTP request and response logging middleware template I use [here](https://gist.github.com/DilanLivera/95632953ce7488b2230f7c84db8b833e).

Thanks for reading.

## Credits

[HTTP Logging in .NET Core and ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/http-logging)

## Resources

[aspnetcore/HttpLoggingMiddleware.cs at a450cb69b5e4549f5515cdb057a68771f56cefd7 · dotnet/aspnetcore](https://github.com/dotnet/aspnetcore/blob/a450cb69b5e4549f5515cdb057a68771f56cefd7/src/Middleware/HttpLogging/src/HttpLoggingMiddleware.cs)
