[Telemetry Processors](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#filtering-itelemetryprocessor) are enabled in ASP.NET Core on both .NET Framework and .NET Core.

Following shows the process of adding additional Telemetry processors to the `TelemetryConfiguration`.


# Adding a Custom Telemetry Processor to telemetry pipeline

Custom telemetry processors can be created to enable the process of filtering the telemetry, as described in [Create Custom Telemetry Processor](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#filtering-itelemetryprocessor). Custom telemetry processors can be added to the `TelemetryConfiguration` by using the extension method `AddApplicationInsightsTelemetryProcessor` on `IServiceCollection`. Use the following example in `ConfigureServices` method of Application's Startup class to add TelemetryProcessors.

``` c#
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddApplicationInsightsTelemetry();
    services.AddApplicationInsightsTelemetryProcessor<MyFirstCustomTelemetryProcessor>();

    // If you have more processors:
    services.AddApplicationInsightsTelemetryProcessor<MySecondCustomTelemetryProcessor>();

    // ...
}
```

# Sampling

[Sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling) is achieved through telemetry processors. Two basic types of sampling are available:

* [Fixed rate sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling/#fixed-rate-sampling-for-aspnet-web-sites)
* [Adaptive sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling/#adaptive-sampling-at-your-web-server)

Adaptive sampling is by default enabled for all applications. Users can turn off this or customize the sampling behavior.

### Turning off Adaptive Sampling
The default sampling feature can be disabled when we add Application Insights service, in the method ```ConfigureServices```, using ```ApplicationInsightsServiceOptions```:

``` c#
public void ConfigureServices(IServiceCollection services)
{
// ...

var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
aiOptions.EnableAdaptiveSampling = false;
services.AddApplicationInsightsTelemetry(aiOptions);

//...
}
```
The above code will disable sampling feature. Follow steps below to add sampling with more customization options.

### Configure Sampling settings
Use extension methods of ```TelemetryProcessorChainBuilder``` as shown below to customize sampling behavior.

``` c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
var configuration = app.ApplicationServices.GetService<TelemetryConfiguration>();

var builder = configuration .TelemetryProcessorChainBuilder;

// Using adaptive sampling
builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:10);
 
// OR Using fixed rate sampling   
double fixedSamplingPercentage = 50;
builder.UseSampling(fixedSamplingPercentage);

builder.Build();

// ...
}

```

**If using the above method to configure sampling, please make sure to use ```aiOptions.EnableAdaptiveSampling = false;``` settings with AddApplicationInsightsTelemetry().** Without this, there would be multiple sampling processors in the TelemetryProcessor chain leading to unintended consequences.


## Metrics Stream

[Metrics Stream](https://azure.microsoft.com/en-us/blog/live-metrics-stream/) captures the live metrics and provides the current working scenario of the application.

Live metrics feature is enabled in both .NET Framework and .NET Core from 2.2.0 of SDK. Earlier versions support Live Metrics only for .NET Framework.

Metrics Stream is now enabled by default. This can be disabled when we add Application Insights service, in the method ```ConfigureServices```, using ```ApplicationInsightsServiceOptions```:

``` c#
var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
aiOptions.EnableQuickPulseMetricStream = false;
services.AddApplicationInsightsTelemetry(aiOptions);
```