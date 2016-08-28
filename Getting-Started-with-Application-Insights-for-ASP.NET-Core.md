Monitor your live ASP.NET Core applications with [Visual Studio Application Insights](https://azure.microsoft.com/documentation/articles/app-insights-overview/). Application Insights is an extensible analytics platform that monitors the performance and usage of your live web applications. With the feedback you get about the performance and effectiveness of your app in the wild, you can make informed choices about the direction of the design in each development lifecycle.

Follow the details in this article to add Application Insights to your app. You'll make the following changes:

- Create an Application Insights resource
- Add Application Insights NuGet package dependency to `project.json`
- Add Application Insights instrumentation key to the `appsettings.json`
- Add Application Insights instrumentation code to the `startup.cs`
- Add Application Insights JavaScript instrumentation to the `_ViewImports.cshtml`, `_Layout.cshtml`

If you prefer, you can use Visual Studio to perform the first three steps by using the "Add Application Insights" menu command on your web project. But you still need to [make some changes manually](#manual).

## Create an Application Insights resource

1. Sign in to the [Microsoft Azure portal](https://portal.azure.com). (Need to [sign up](https://azure.microsoft.com/pricing/free-trial/)?)
2. Create a new Application Insights resource. (Click **New**, **Developer Services**, **Application Insights**). Select the **ASP.NET web application** type.

    ![Create a resource](https://acom.azurecomcdn.net/80C57D/cdn/mediahandler/docarticles/dpsmedia-prod/azure.microsoft.com/en-us/documentation/articles/app-insights-create-new-resource/20160826050250/01-new.png)
3. In your new resource, open the **Essentials** drop-down so that you can copy the Instrumentation Key - you'll need it shortly. 

## Add Application Insights NuGet package dependency to `project.json`
Add the following entry to the `dependencies`` section. 

``` json
{
  "dependencies": {
    "Microsoft.ApplicationInsights.AspNetCore": "1.0.0"
  }
}
```

Verify the version number: get the latest from the [Release page](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases). 

In case of **.NET Core** applications, if you run into restore errors with respect to application insights dependency, please add ```"dnxcore50"``` and ```"portable-net45+win8" ``` to the imports list (if it does not exist), under ```frameoworks``` section of ```project.json```, as described below. Please visit [Migrating from DNX](http://dotnet.github.io/docs/core-concepts/dnx-migration.html) for more details.
``` json
{
    "frameworks": {
        "netcoreapp1.0": { 
            "imports": ["dnxcore50", "portable-net45+win8"]
        }
    }
}
```

## Add the instrumentation key to the `appsettings.json`

Add the instrumentation key of your Application Insights web application resource to the `ApplicationInsights` section of the `appsettings.json`. 

``` json
{
  "ApplicationInsights": {
    "InstrumentationKey": "11111111-2222-3333-4444-555555555555"
  }
}
```
<a name="manual"></a>
## Add Application Insights instrumentation code to `startup.cs`

If you don't already have the code that parses appsettings.json file and initializes configuration variable, create Configuration variable:

``` C#
public IConfigurationRoot Configuration { get; set; }
```

Then add the following dependency entries to your project.json if they are not defined there already.
``` json
    "Microsoft.Extensions.Configuration.Abstractions": "1.0.0",
    "Microsoft.Extensions.Configuration.Json":  "1.0.0"
```

(Verify the version number.)

Then add the code that parses configuration if you don't have it already in the method ```Startup```.

``` C#
  // Setup configuration sources.
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json")

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
* [Analytics](https://azure.microsoft.com/documentation/articles/app-insights-analytics/) is a powerful query language that you can use to extract performance and usage data from your telemetry.
* [Set alerts](https://azure.microsoft.com/documentation/articles/app-insights-alerts/) to notify you when your app goes off-limits.

## Get more telemetry

* [Monitor dependencies](https://azure.microsoft.com/documentation/articles/app-insights-dependencies/) to see if REST, SQL or other external resources are slowing you down.
* [Use the API](https://azure.microsoft.com/documentation/articles/app-insights-api-custom-events-metrics/) to send your own events and metrics for a more detailed view of your app's performance and usage.
* [Availability tests](https://azure.microsoft.com/documentation/articles/app-insights-monitor-web-app-availability/) check your app constantly from around the world. 
