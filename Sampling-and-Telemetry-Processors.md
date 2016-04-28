[Sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#sampling) and [Telemetry Processors](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#filtering-itelemetryprocessor) are enabled in full framework (currently not supported in .NET Core), when Application Insights (>= [1.0.0-RC1-Update4](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases/tag/v1.0.0-rc1-update4)) is installed to monitor your live ASP.NET 5 web app.

## Telemetry Processors

Telemetry processors, are configured as a part of telemetry configuration. From (>= [1.0.0-RC1-Update4](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases/tag/v1.0.0-rc1-update4)), we have enabled ```TelemetryConfiguration.Active``` as the default telemetry configuration to configure modules and telemetry initializers. Once we have the telemetry configuration instance ready, we can add our own custom telemetry processors or use existing telemetry processors (adaptive sampling and fixed sampling). Following describes the process of accessing telemetry configuration, using custom telemetry processors, enabling sampling, and controlling default adaptive sampling behavior for AspNet5 applications.

### Accessing Telemetry Configuration

```services.AddApplicationInsightsTelemetry(Configuration)``` function call in the method ```ConfigureServices``` ([Getting Started](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Getting-Started)) of ```startup.cs```, will register the modules and telemetry intializers with the telemetry configuration. As a result, we can access telemetry configuration either through 

```var telemetryConfiguration = TelemetryConfiguration.Active``` 

or by getting it from services 

```var telemetryConfiguration = services.GetRequiredService<TelemetryConfiguration>();```

### Using Custom Telemetry Processor

Custom telemetry processors can be used to enable the process of filtering the telemetry, as described in [Create custom telemetry processor](https://azure.microsoft.com/en-us/documentation/articles/app-insights-api-filtering-sampling/#filtering-itelemetryprocessor). We can then ```use``` the custom telemetry processor through code as follows:

``` c#
var builder = telemetryConfiguration.TelemetryProcessorChainBuilder;
builder.Use((next) => new CustomTelemetryProcessor(next));

// If you have more processors:
builder.Use((next) => new AnotherCustomTelemetryProcessor(next));

builder.Build();
```

## Sampling

[Sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling) is achieved through telemetry processors, and will be supported in AspNet 5 applications. We support two type of sampling today:
* [Fixed rate sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling/#fixed-rate-sampling-for-aspnet-web-sites)
* [Adaptive sampling](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling/#adaptive-sampling-at-your-web-server)

Adaptive sampling is by default added to the applications. The default sampling feature can be disabled using ```ApplicationInsightsServiceOptions``` when ```AddApplicationInsightsTelemetry``` is invoked in ```ConfigureServices```, as described below:

``` c#
var aiOptions = new Microsoft.ApplicationInsights.AspNet.Extensions.ApplicationInsightsServiceOptions();
aiOptions.EnableAdaptiveSampling = false;

services.AddApplicationInsightsTelemetry(Configuration, aiOptions);
```

It is best advised to add telemetry processors in the method ```ConfigureServices()``` of application ```startup.cs```. Sampling can be enabled using extension methods of telemetry processor chain builder as described below:

``` c#
var builder = telemetryConfiguration.TelemetryProcessorChainBuilder;
// Using adaptive sampling
builder.UseAdaptiveSampling();
 
// Using fixed rate sampling   
double fixedSamplingPercentage = 50;
builder.UseSampling(fixedSamplingPercentage);

builder.Build();
```

## Quick Pulse

[Quick pulse](https://azure.microsoft.com/en-us/documentation/articles/app-insights-sampling) captures the live metrics and provides the current working scenario of the application. Quick pulse is not enabled by default in the applications. In order to enable the feature, we need to register the ```QuickPulseTelemetryProcessor``` as well as ```QuickPulseTelemetryModule``` with the telemetry configuration. It is best advised to have ```QuickPulseTelemetryProcessor``` as the first telemetry processor, which can ensure that all the telemetry (before filtered/sampled) is gone through live metrics. So, the default adaptive sampling has to be disabled first, register the quick pulse module and processor and then add adaptive sampling telemetry processor back to the configuration, as described below:

The function call 

``` c#
services.AddApplicationInsightsTelemetry(Configuration);
``` 

should be replaced by

``` c#
// Disable the default adaptive sampling feature
var aiOptions = new Microsoft.ApplicationInsights.AspNet.Extensions.ApplicationInsightsServiceOptions();
aiOptions.EnableAdaptiveSampling = false;
services.AddApplicationInsightsTelemetry(Configuration, aiOptions);

// Initialize QuickPulseTelemetryModule
var module = new QuickPulseTelemetryModule();
module.Initialize(TelemetryConfiguration.Active);

// Use and Register QuickPulseTelemetryProcessor
QuickPulseTelemetryProcessor processor = null; 
var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;
builder.Use((next) => {
    processor = new QuickPulseTelemetryProcessor(next);
    module.RegisterTelemetryProcessor(processor);
    return processor;
} );

// Re-enable Adaptive sampling
builder.UseAdaptiveSampling();

// Build the processors
builder.Build();
```