# Add health checks in ASP.Net Core

![cover-image](./cover-image.jpg)

Health checks help us find the status of a service or other services on which our service is dependent. For example, Health checks are helpful if you have a load balancer in front of your services to distribute the load across multiple service nodes. If there are various nodes, the load balancer can use the node's health status to route requests to the healthy node. We could also use health checks to find the status of the dependent services. For example, if our service uses another API or a database server, we could use the health status of the API or database to calculate the health status of our service. In this case, if the dependent service is in an unhealthy or degraded status, we could downgrade the health status of our service to degraded or unhealthy. In ASP.Net Core APIs, Health checks are endpoints that expose the service health to other services.

To add a basic health check to an ASP.Net Core application, we first need to register health check services with [AddHealthChecks](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.healthcheckservicecollectionextensions.addhealthchecks) in the `ConfigureServices` method of the `Startup` class. Then we need to add the `EndpointMiddleware` to the `IApplicationBuilder` and use `MapHealthChecks` extension method to add a health check endpoint. We are free to name the endpoint whatever we want.

## ASP.Net Core 3.1 - 5

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddHealthChecks();
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapHealthChecks("/health");
        });
    }
}
```

## ASP.Net Core 6

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHealthChecks();

var app = builder.Build();

app.MapHealthChecks("/health");

app.Run();
```

## Create a custom health check

To create a custom health check, first, we need to implement the `IHealthCheck` interface. Specifically, we need to implement the `CheckHealthAsync` method of the `IHealthCheck` interface. Inside the method, we can perform any tasks to decide the `HealthCheckResult`'s health status.

```csharp
Using Microsoft.Extensions.Diagnostics.HealthChecks;

namespace WeatherApi;

public class MemoryHealthCheck : IHealthCheck
{
    private const long Threshold = 1024L * 1024L * 1024L;

    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        var allocatedBytesInManagedMemory = GC.GetTotalMemory(forceFullCollection: false);
        var data = new Dictionary<string, object>()
        {
            { "AllocatedBytesInManagedMemory", allocatedBytesInManagedMemory },
            { "NumberOfTimesGarbageCollectionOccurredForGen0Collection", GC.CollectionCount(0) },
            { "NumberOfTimesGarbageCollectionOccurredForGen1Collection", GC.CollectionCount(1) },
            { "NumberOfTimesGarbageCollectionOccurredForGen2Collection", GC.CollectionCount(2) }
        };

        if (allocatedBytesInManagedMemory > Threshold)
        {
            var result = new HealthCheckResult(
                status: context.Registration.FailureStatus,
                description: $"Allocated bytes in managed memory is > {Threshold}",
                data: data);

            return Task.FromResult(result);
        }

        var healthyResult = new HealthCheckResult(
            status: HealthStatus.Healthy,
            description: $"Allocated bytes in managed memory is <= {Threshold}",
            data: data);

        return Task.FromResult(healthyResult);
    }
}
```

I only use a condition in the above health check to decide if the service is healthy. If it is not healthy, `MemoryHealthCheck` returns the default failure status(The Default failure status is `Unhealthy`). We can change the default failure status for the health check when we add the health check to the `IHealthChecksBuilder`.

```csharp
builder.Services
    .AddHealthChecks()
    .AddCheck<MemoryHealthCheck>(name: "memory", failureStatus: HealthStatus.Degraded);
```

## Customize the health check HTTP status codes and response

By default, when we call a health check endpoint, the endpoint will return a `200 OK` status code regardless of the health check status. Sometimes, it will be helpful to produce a unique status code depending on the health check status. We can achieve this by setting the `ResultStatusCodes` property of `HealthCheckOptions`. If the health status is degraded in the following example, the endpoint will return a `500 InternalServerError`. When health status is unhealthy, the endpoint will return `503 ServiceUnavailable`.

```csharp
app.MapHealthChecks(pattern: "/ready", options: new HealthCheckOptions
{
    ResponseWriter = WriteHealthCheckResponseAsync,
    ResultStatusCodes =
    {
        [HealthStatus.Degraded] = StatusCodes.Status500InternalServerError,
        [HealthStatus.Healthy] = StatusCodes.Status200OK,
        [HealthStatus.Unhealthy] = StatusCodes.Status503ServiceUnavailable,
    },
    Predicate = _ => true
});
```

The default responses(report) for health checks is `Healthy`, `Unhealthy` or `Degraded`. We can customize the response(report) by providing a delegate to the `ResponseWriter` of `HealthCheckOptions`.

```csharp
static Task WriteHealthCheckResponseAsync(
    HttpContext httpContext, HealthReport healthReport)
{
    httpContext.Response.ContentType = "application/json";

    var dependencyHealthChecks = healthReport.Entries.Select(entry => new
    {
        Name = entry.Key,
        Discription = entry.Value.Description,
        Status = entry.Value.Status.ToString(),
        DurationInSeconds = entry.Value.Duration.TotalSeconds.ToString("0:0.00"),
        Data = entry.Value.Data,
        Exception = entry.Value.Exception?.Message
    });

    var healthCheckResponse = new
    {
        Status = healthReport.Status.ToString(),
        TotalCheckExecutionTimeInSeconds = healthReport.TotalDuration.TotalSeconds.ToString("0:0.00"),
        DependencyHealthChecks = dependencyHealthChecks
    };

    var serializerOptions = new JsonSerializerOptions
    {
        WriteIndented = true,
        DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase
    };

    var responseString = JsonSerializer.Serialize(healthCheckResponse, serializerOptions);

    return httpContext.Response.WriteAsync(responseString);
}
```

The above delegate creates the following report.

```json
{
  "status": "Healthy",
  "totalCheckExecutionDurationInSeconds": "0:0.12",
  "dependencyHealthChecks": [
    {
      "name": "memory",
      "discription": "Reports memory status of the application.",
      "status": "Healthy",
      "durationInSeconds": "0:0.02",
      "data": {
        "AllocatedBytesInManagedMemory": 1907368,
        "NumberOfTimesGarbageCollectionOccurredForGen0Collection": 2,
        "NumberOfTimesGarbageCollectionOccurredForGen1Collection": 2,
        "NumberOfTimesGarbageCollectionOccurredForGen2Collection": 0
      }
    }
  ]
}
```

## Filter health checks

Depending on your application, you might have several health checks. If you do not need to return the status of all the health checks, you can filter out the health checks you need using tags and the `Predicate` property of `HealthCheckOptions`. `Predicate` property takes a parameter of type `HealthCheckRegistration`. We can use the `Tags` property of `HealthCheckRegistration` to filter the health checks. We can set the tag/s using the `tags` parameter of the `AddCheck` extension when adding the health checks.

```csharp
builder.Services
    .AddHealthChecks()
    .AddCheck<MemoryHealthCheck>(
        name: "memory",
        failureStatus: HealthStatus.Unhealthy,
        tags: new[] { "memory" });
```

```csharp
app.MapHealthChecks(pattern: "/ready", options: new HealthCheckOptions
{
    Predicate = registration => registration.Tags.Contains("memory")
}); ;
```

Thanks for reading.

## Resources

- [Microsoft Docs: Health checks in ASP.NET Core](https://docs.microsoft.com/aspnet/core/host-and-deploy/health-checks)
- [Microsoft Docs: HealthCheckOptions Class](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.diagnostics.healthchecks.healthcheckoptions)
- [Microsoft Docs: HealthCheckRegistration Class](https://docs.microsoft.com/dotnet/api/microsoft.extensions.diagnostics.healthchecks.healthcheckregistration)
- [Microsoft Docs: HealthCheckEndpointRouteBuilderExtensions.MapHealthChecks Method](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.builder.healthcheckendpointroutebuilderextensions.maphealthchecks)
- [Microsoft Docs: HealthCheckRegistration.Tags Property](https://docs.microsoft.com/dotnet/api/microsoft.extensions.diagnostics.healthchecks.healthcheckregistration.tags)
