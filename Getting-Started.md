For web applications created from the standard ASP.NET 5 project template in Visual Studio 2015, you don't need to make any changes if you selected "Add Application Insights" during project creation. Otherwise, the following changes need to be made
- Add Application Insights NuGet package dependency to `project.json`
- Add Application Insights instrumentation key to the `appsettings.json`
- Add Application Insights instrumentation code to the `startup.cs`
- Add Application Insights JavaScript instrumentation to the `_ViewImports.cshtml`, `_Layout.cshtml`

## Add Application Insights NuGet package dependency to `project.json`
Add the following entry to the `dependencies`` section. 

``` json
{
  "dependencies": {
    "Microsoft.ApplicationInsights.AspNet": "1.0.0-beta8"
  }
}
```
where "0.*" is a Nuget number, for example "1.0.0-beta8". Use the latest version number from [Release page](https://github.com/Microsoft/ApplicationInsights-aspnet5/releases). 

## Add Application Insights instrumentation key to the `appsettings.json`
Add the instrumentation key of an existing Application Insights web application resource to the `ApplicationInsights` section of the `appsettings.json`. 
``` json
{
  "ApplicationInsights": {
    "InstrumentationKey": "11111111-2222-3333-4444-555555555555"
  }
}
```

If you don't have the instrumentation key, follow instructions on this [page](http://azure.microsoft.com/en-us/documentation/articles/app-insights-start-monitoring-app-health-usage) to get it.

## Add Application Insights instrumentation code to the `startup.cs`
If you don't already have the code that parses appsettings.json file and initializes configuration variable, create Configuration variable
``` C#
public IConfigurationRoot Configuration { get; set; }
```

Then add the following dependency entries to your project.json if they are not defined there already.
``` json
    "Microsoft.Framework.Configuration.Abstractions": "1.0.0-beta8",
    "Microsoft.Framework.Configuration.Json":  "1.0.0-beta8"
```

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

In the method ```ConfigureServices``` add Application Insights service. You'll need to add namespace ```Microsoft.ApplicationInsights.AspNet``` in the using list:
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
