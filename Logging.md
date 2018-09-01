Application Insights SDK is capable of collecting Trace telemetry from logs generated via Asp.Net Core's [logging infrastructure](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging).
When running the application from Visual Studio IDE, all traces logged via `ILogger` interface are automatically captured. If an instrumentation key is configured, then these traces are sent to the Application Insights service. To limit the level of logging captured by Application Insights when run from Visual Studio see this [issue](https://github.com/Microsoft/ApplicationInsights-aspnetcore/issues/603)

Following shows how to configure Application Insights to collect traces logged via `ILogger` interface when running outside of Visual Studio.

This document assumes that application insights instrumentation key is configured already using instructions [here](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Getting-Started-for-a-ASP.NET-CORE-2.0-WebApp) 

Application Insights SDK for Asp.Net Core provides an extension method `AddApplicationInsights` on `ILoggerFactory` to configure logging. Modify the `Startup.cs` class of your application as follows to enable Application Insights logging.

``` csharp
public void Configure(
    IApplicationBuilder app, 
    IHostingEnvironment env, 
    ILoggerFactory loggerFactory)
{
  /*...existing code..*/
  loggerFactory.AddApplicationInsights(app.ApplicationServices, LogLevel.Warning);
}
```

This above snippet causes all messages (Warning or above) to be sent to Application Insights. 

As of today, this is the only way to enable application insights traces. We will be adding new extension methods on `ILoggingBuilder` [soon](https://github.com/Microsoft/ApplicationInsights-aspnetcore/issues/536).

# Include EventId in logs

It is possible to include `EventId` in telemetry properties. Simply setup `ApplicationInsightsLoggerOptions` instance in `Startup.ConfigureServices` method.

``` csharp
services
    .AddOptions<ApplicationInsightsLoggerOptions>()
    .Configure(o => o.IncludeEventId = true);
```

Also it is possible to read configuration from configuration file.

At first add code into `Startup.ConfigureServices` method.

``` csharp
services
    .AddOptions<ApplicationInsightsLoggerOptions>()
    .Bind(Configuration.GetSection("ApplicationInsightsLogger"));
```

Then change your configuration file:

``` json
{
  "ApplicationInsightsLogger": {
    "IncludeEventId": true
  }
}
```