Using [sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#sampling) and [telemetry processors](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#filtering-itelemetryprocessor) is enabled in full framework (currently not supported in .NET Core), when Application Insights (>= [1.0.0-RC1-Update4](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases/tag/v1.0.0-rc1-update4)) is installed to monitor your live ASP.NET 5 web app.

## Accessing Telemetry Processors

Telemetry processors, are configured as a part of telemetry configuration. From (>= [1.0.0-RC1-Update4](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases/tag/v1.0.0-rc1-update4)), we have enabled ```TelemetryConfiguration.Active``` as the default telemetry configuration to configure modules and telemetry initializers. Once we have the telemetry configuration instance ready, we can add our own custom telemetry processors or use existing telemetry processors (adaptive sampling and fixed sampling). Following describes the process of accessing telemetry configuration, using custom telemetry processors, enabling sampling, and controlling default adaptive sampling behavior for AspNet5 applications.

### Accessing Telemetry Configuration

```services.AddApplicationInsightsTelemetry(Configuration)``` function call in the method ```ConfigureServices``` ([Getting Started](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Getting-Started)) of ```startup.cs```, will register the modules and telemetry intializers with the telemetry configuration. As a result, we can access telemetry configuration either through ```var telemetryConfiguration = TelemetryConfiguration.Active``` or by getting it from services (```var telemetryConfiguration = services.GetRequiredService<TelemetryConfiguration>();```).

### Using Custom Telemetry Processor

Custom telemetry processors can be used to enable the process of filtering the telemetry, as described in [Create custom telemetry processor](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#filtering-itelemetryprocessor). We can then ```use``` the custom telemetry processor through code as follows:

``` c#
var builder = telemetryConfiguration.TelemetryProcessorChainBuilder;
builder.Use((next) => new CustomTelemetryProcessor(next));

// If you have more processors:
builder.Use((next) => new AnotherCustomTelemetryProcessor(next));

builder.Build();
```

### Using Sampling

We today, provide two types of sampling extensions for telemetry processor chain builder. 
* Fixed sampling
* [Adaptive sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling/#adaptive-sampling-at-your-web-server)

These extensions of builder can be used to enable a specific sampling, as follows:

``` c#
var builder = telemetryConfiguration.TelemetryProcessorChainBuilder;
// Using adaptive sampling
builder.UseAdaptiveSampling();
 
// Using fixed rate sampling   
double fixedSamplingPercentage = 50;
builder.UseSampling(fixedSamplingPercentage);

builder.Build();
```