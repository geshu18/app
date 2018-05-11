The following auto-collection modules are enabled by default for Asp.Net Core Apps with either `UseApplicationInsights()` or `AddApplicationInsightsTelemetry()` methods of enabling Application Insights.

RequestTrackingTelemetryModule - Collects information about Requests incoming to your application. This module works from subscribing to `DiagnosticSource` events publishing by the Asp.Net Core Host.

[DependencyTrackingTelemetryModule](https://azure.microsoft.com/documentation/articles/app-insights-dependencies/)

[PerformanceCollectorModule](https://azure.microsoft.com/documentation/articles/app-insights-web-monitor-performance/) only if the application is targeting the full .NET Framework as there is no perf counter collection in .NET Core

[QuickPulseTelemetryModule](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-live-stream)

AppServicesHeartbeatTelemetryModule - Collects additional HeartBeart properties if application is deployed to Azure Web Apps

AzureInstanceMetadataTelemetryModule - Collects additional HeartBeart properties if application is deployed to Azure VMs

## Disabling Telemetry Module

In order to disable any auto-collection module, you need to manually remove the modules from the DI. Use the following example code in the method ```ConfigureServices``` of your application's Startup.cs class. Please note that telemetry modules should be removed only after adding Application Insights to the services (i.e after `AddApplicationInsights()`).

``` c#
public void ConfigureServices(IServiceCollection services)
{
.....
services.AddApplicationInsightsTelemetry("ikey");

// Removing performance collector module - to disable default performance counter collection
// This can be replaced with any of the auto-collection module.
var performanceCounterService = services.FirstOrDefault<ServiceDescriptor>(t => t.ImplementationType == typeof(PerformanceCollectorModule));
if (performanceCounterService != null)
{
 services.Remove(performanceCounterService);
}
```

## Configuring Telemetry Module Services
In order to configure any of the TelemetryModules, use the extension method `ConfigureTelemetryModule<T>` on `IServiceCollection` (available from 2.3.0-beta1 onwards)
Use the following code in the ConfigureServices method of your application's Startup class to configure any module.
```
public void ConfigureServices(IServiceCollection services)
{
......
services.AddApplicationInsightsTelemetry("ikey");
services.ConfigureTelemetryModule<DependencyTrackingTelemetryModule>( module => module.SetComponentCorrelationHttpHeaders = false);

//sets AutheticationApiKey for QuickPulse
services.ConfigureTelemetryModule<QuickPulseTelemetryModule>( module => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
```
