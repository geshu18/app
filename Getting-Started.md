[Application Insights](https://azure.microsoft.com/documentation/articles/app-insights-overview/) is an extensible analytics platform that monitors the performance and usage of your live ASP.NET Core web applications.

Follow the details in this article to make the following changes:
- Create an Application Insights resource
- Add Application Insights NuGet package dependency to `project.json`
- Add Application Insights instrumentation key to the `appsettings.json`
- Add Application Insights instrumentation code to the `startup.cs`
- Add Application Insights JavaScript instrumentation to the `_ViewImports.cshtml`, `_Layout.cshtml`


## Create an Application Insights resource

1. Sign in to the [Microsoft Azure portal](https://portal.azure.com). (Need to [sign up](https://azure.microsoft.com/pricing/free-trial/)?)
2. Create a new Application Insights resource. (Click **New**, **Developer Services**, **Application Insights**). Select the ASP.NET Core application type.
3. In your new resource, open the **Essentials** drop-down so that you can copy the Instrumentation Key - you'll need it shortly. 

## Add Application Insights NuGet package dependency to `project.json`
Add the following entry to the `dependencies`` section. 

``` json
{
  "dependencies": {
    "Microsoft.ApplicationInsights.AspNetCore": "1.0.0-rc2-final"
  }
}
```
Verify the version number: get the latest from the [Release page](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases). 

## Add the instrumentation key to the `appsettings.json`

Add the instrumentation key of your Application Insights web application resource to the `ApplicationInsights` section of the `appsettings.json`. 

``` json
{
  "ApplicationInsights": {
    "InstrumentationKey": "11111111-2222-3333-4444-555555555555"
  }
}
```

## Add Application Insights instrumentation code to `startup.cs`

If you don't already have the code that parses appsettings.json file and initializes configuration variable, create Configuration variable:

``` C#
public IConfigurationRoot Configuration { get; set; }
```

Then add the following dependency entries to your project.json if they are not defined there already.
``` json
    "Microsoft.Extensions.Configuration.Abstractions": "1.0.0-rc2-final",
    "Microsoft.Extensions.Configuration.Json":  "1.0.0-rc2-final"
```

(Verify the version number.)

Then add the code that parses configuration if you don't have it already.

``` C#
  // Setup configuration sources.
            var builder = new ConfigurationBuilder()
                .SetBasePath(appEnv.ApplicationBasePath)
                .AddJsonFile("appsettings.json")
                .AddEnvironmentVariables();
            Configuration = builder.Build();
```


In the method ```Startup``` make sure to set Application Insights settings overrides. Specifically, set developer mode to true in development environment:

``` C#
if (env.IsDevelopment())
{
    builder.AddApplicationInsightsSettings(developerMode: true);
}
```

In the method ```ConfigureServices``` add Application Insights service. You'll need to add namespace ```Microsoft.ApplicationInsights.AspNetCore``` in the using list:
``` c#
services.AddApplicationInsightsTelemetry(Configuration);
```

In the method ```Configure``` add Application Insights request and exception tracking middleware. Please note that request tracking middleware should be added as the very first middleware in pipeline:

``` c#
// Add Application Insights monitoring to the request pipeline as a very first middleware.
app.UseApplicationInsightsRequestTelemetry();
```
Exception middleware should be added after error page and any other error handling middleware:

``` c#
// Add Application Insights exceptions handling to the request pipeline.
app.UseApplicationInsightsExceptionTelemetry();
```
If you don't have any error handling middleware defined, just add this method right after UseApplicationInsightsRequestTelemetry method.

## Add Application Insights JavaScript instrumentation to the `_ViewImports.cshtml`, `_Layout.cshtml`
 (if present)

In `_ViewImports.cshtml`, add injection:
``` html
@inject Microsoft.ApplicationInsights.Extensibility.TelemetryConfiguration TelemetryConfiguration 
```

In `_Layout.cshtml`, insert HtmlHelper to the end of ```<head>``` section but before any other script. Any custom javascript telemetry you want to report from the page should be injected after this snippet:

``` html
	@Html.ApplicationInsightsJavaScript(TelemetryConfiguration) 
</head>
```

## Run your app and view your telemetry

* Use [Search Explorer](https://azure.microsoft.com/documentation/articles/app-insights-diagnostic-search/) to investigate individual requests, exceptions, log traces and other events.
* Use [Metrics Explorer](https://azure.microsoft.com/documentation/articles/app-insights-metrics-explorer/) to slice and dice metrics and counts such as response times, page views and user counts.
* [Set alerts](https://azure.microsoft.com/documentation/articles/app-insights-alerts/) to notify you when your app goes off-limits.

## Get more telemetry

* [Monitor dependencies](https://azure.microsoft.com/documentation/articles/app-insights-dependencies/) to see if REST, SQL or other external resources are slowing you down.
* [Use the API](https://azure.microsoft.com/documentation/articles/app-insights-api-custom-events-metrics/) to send your own events and metrics for a more detailed view of your app's performance and usage.
* [Availability tests](https://azure.microsoft.com/documentation/articles/app-insights-monitor-web-app-availability/) check your app constantly from around the world. 
