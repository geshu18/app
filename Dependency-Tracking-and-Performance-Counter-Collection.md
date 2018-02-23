[Dependency tracking](https://azure.microsoft.com/documentation/articles/app-insights-dependencies/) module is enabled by default when application is targeting both .NET Core and .NET Framework.
[Performance counter collection](https://azure.microsoft.com/documentation/articles/app-insights-web-monitor-performance/) is enabled by default in both .NET Core and .NET Framework - However there will be no perf counter collection in .NET Core as perf counters are not supported in .NET Core. Live metrics is enabled in both cases though.

## Telemetry Modules and Default Tracking

Application Insights references [ApplicationInsights-server-dotnet](https://github.com/Microsoft/ApplicationInsights-server-dotnet/releases/tag/InitialCommit) that includes telemetry modules (```DependencyTrackingTelemetryModule``` and ```PerformanceCollectorModule```), which are responsible for enabling default dependency tracking and performance counter collection. Both the telemetry modules are added along with telemetry initializers, when Application Insights is added to the application.

## Disabling Telemetry Module Services

In order to disable the services, you need to manually remove the modules from the existing list of services in the method ```ConfigureServices```. Please note that telemetry modules should be removed only after adding Application Insights to the services.

``` c#
// Removing dependency tracking telemetry module - to disable default dependency tracking -- This will not work currently as DependencyTrackingTelemetryModule will not be instantiated until end of ConfigureServices method.
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

## Configuring Telemetry Module Services
Inorder to configure any properties of TelemetyModules, obtain the module from DI Container first. Then modify the required properties, and re-initialize the module with currently active Telemetry Configuration.

For eg:, to modify DependencyTrackingTelemetryModule to disable setting Correlationheaders, the following code is required in the Configure method of you Startup class.
```
DependencyTrackingTelemetryModule dep;
            var modules = app.ApplicationServices.GetServices<ITelemetryModule>();
            foreach(var module in modules)
            {
                if (module is DependencyTrackingTelemetryModule)
                {
                    dep = module as DependencyTrackingTelemetryModule;
                    dep.SetComponentCorrelationHttpHeaders = false;                    
                    dep.Initialize(TelemetryConfiguration.Active);                    
                }
            }
```
