[Telemetry Processors](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#filtering-itelemetryprocessor) are enabled in ASP.NET Core on both .NET Framework and .NET Core.

Telemetry processors, are configured as a part of telemetry configuration. Once telemetry configuration instance is available, telemetry processors can be made available to the configuration. Following describes the process of [accessing telemetry configuration](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Telemetry-Processors:-Sampling-and-Quick-Pulse#accessing-telemetry-configuration), [using custom telemetry processors](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Telemetry-Processors:-Sampling-and-Quick-Pulse#using-custom-telemetry-processor), [enabling sampling](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Telemetry-Processors:-Sampling-and-Quick-Pulse#sampling) and [activating Metrics Stream](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Telemetry-Processors:-Sampling-and-Quick-Pulse#quick-pulse).

> Note: It is best advised to use telemetry processors in the method ```ConfigureServices``` of application ```startup.cs```.

## Accessing Telemetry Configuration

Telemetry configuration can be accessed either using ```Active```

``` c#
var telemetryConfiguration = TelemetryConfiguration.Active
``` 

or by getting it through application builder services

``` c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerfactory)
{
    var configuration= app.ApplicationServices.GetService<TelemetryConfiguration>();
```

## Using Custom Telemetry Processor

Custom telemetry processors can be created to enable the process of filtering the telemetry, as described in [Create Custom Telemetry Processor](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#filtering-itelemetryprocessor). Custom telemetry processors can be made available to configuration through ```TelemetryProcessorChainBuilder``` as described below:

``` c#
public void ConfigureServices(IServiceCollection services)
{
    // ...
    var telemetryConfiguration =
        services.BuildServiceProvider().GetService<TelemetryConfiguration>();
    var builder = telemetryConfiguration.TelemetryProcessorChainBuilder;
    builder.Use((next) => new CustomTelemetryProcessor(next));

    // If you have more processors:
    builder.Use((next) => new AnotherCustomTelemetryProcessor(next));

    builder.Build();
    // ...
}
```

## Sampling

[Sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling) is achieved through telemetry processors. Two basic types of sampling are available:

* [Fixed rate sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling/#fixed-rate-sampling-for-aspnet-web-sites)
* [Adaptive sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling/#adaptive-sampling-at-your-web-server)

Adaptive sampling is by default enabled for the applications. The default sampling feature can be disabled when we add Application Insights service, in the method ```ConfigureServices```, using ```ApplicationInsightsServiceOptions```:

``` c#
var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
aiOptions.EnableAdaptiveSampling = false;
services.AddApplicationInsightsTelemetry(aiOptions);
```

Sampling can also be enabled using extension methods of ```TelemetryProcessorChainBuilder``` as described below:

``` c#
var builder = telemetryConfiguration.TelemetryProcessorChainBuilder;
// Using adaptive sampling
builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:10);
 
// Using fixed rate sampling   
double fixedSamplingPercentage = 50;
builder.UseSampling(fixedSamplingPercentage);

builder.Build();
```

If using the above method to configure sampling, please make sure to use ```aiOptions.EnableAdaptiveSampling = false;``` settings with AddApplicationInsightsTelemetry(). Without this, there would be multiple sampling processors in the TelemetryProcessor chain leading to unintended consequences.


## Metrics Stream

[Metrics Stream](https://azure.microsoft.com/en-us/blog/live-metrics-stream/) captures the live metrics and provides the current working scenario of the application.

Live metrics feature is enabled in both .NET Framework and .NET Core from 2.2.0 of SDK. Earlier versions support Live Metrics only for .NET Framework.

Metrics Stream is now enabled by default. This can be disabled when we add Application Insights service, in the method ```ConfigureServices```, using ```ApplicationInsightsServiceOptions```:

``` c#
var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
aiOptions.EnableQuickPulseMetricStream = false;
services.AddApplicationInsightsTelemetry(aiOptions);
```