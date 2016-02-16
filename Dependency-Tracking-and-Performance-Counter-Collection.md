[Dependency tracking](https://azure.microsoft.com/documentation/articles/app-insights-dependencies/) and [performance counter collection](https://azure.microsoft.com/documentation/articles/app-insights-web-monitor-performance/) are by default enabled in full framework (currently not supported in .NET Core), when Application Insights (>= [1.0.0-RC1-Update2](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases/tag/v1.0.0-rc1-update2)) is installed to monitor your live ASP.NET 5 web app.

## Telemetry Modules and Default Tracking

Application Insights references [ApplicationInsights-server-dotnet](https://github.com/Microsoft/ApplicationInsights-server-dotnet/releases/tag/InitialCommit) that includes telemetry modules (```DependencyTrackingTelemetryModule``` and ```PerformanceCollectorModule```), which are responsible for enabling default dependency tracking and performance counter collection. Both the telemetry modules are added along with telemetry initializers, when Application Insights is added to services in the method ```ConfigureServices``` ([Getting Started](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Getting-Started)). 

``` c#
services.AddApplicationInsightsTelemetry(Configuration);
```

## Disabling Telemetry Module Services

In order to disable the services, you need to manually remove the modules from the existing list of services in the method ```ConfigureServices```. Please note that telemetry modules should be removed only after adding Application Insights to the services.

``` c#
// Removing dependency tracking telemetry module - to disable default dependency tracking
var dependencyTrackingService = services.FirstOrDefault<ServiceDescriptor>(t => t.ImplementationType == typeof(DependencyTrackingTelemetryModule));
if (dependencyTrackingService!= null)
{
 services.Remove(dependencyTrackingService);
}

// Removing performance collector module - to disable default performance counter collection
var performanceCounterService = services.FirstOrDefault<ServiceDescriptor>(t => t.ImplementationType == typeof(PerformanceCollectorModule));
if (performanceCounterService != null)
{
 services.Remove(performanceCounterService);
}
```

