For web applications created from the standard ASP.NET 5 project template in Visual Studio 2015 RC, you need to make the following changes 
- Add Application Insights NuGet package dependency to `project.json`
- Add Application Insights instrumentation key to the `config.json`
- Add Application Insights instrumentation code to the `startup.cs`
- Add Application Insights JavaScript instrumentation to the `_Layout.cshtml`

## project.json
Add the following entry to the `dependencies`` section. 
``` json
{
  "dependencies": {
    "Microsoft.ApplicationInsights.AspNet": "0.32.0-beta4"
  }
}
```

## config.json
Add the instrumentation key of an existing Application Insights web application resource to the `ApplicationInsights` section of the `config.json`. 
``` json
{
  "ApplicationInsights": {
    "InstrumentationKey": "11111111-2222-3333-4444-555555555555"
  }
}
```

If you don't have the instrumentation key, follow instructions on this [page](http://azure.microsoft.com/en-us/documentation/articles/app-insights-start-monitoring-app-health-usage) to get it.

##Startup.cs
If you don't already have the code that parses config.json file and initializes configuration variable, create Configuration variable
``` C#
public IConfiguration Configuration { get; set; }
```

Then add the following dependency entries to your project.json if they are not defined there already.
``` json
    "Microsoft.Framework.ConfigurationModel.Interfaces": "1.0.0-beta4",
    "Microsoft.Framework.ConfigurationModel.Json":  "1.0.0-beta4"
```

Then add the code that parses configuration if you don't have it already.

``` C#
  // Setup configuration sources.
            var configuration = new Configuration()
                .AddJsonFile("config.json")
                .AddJsonFile($"config.{env.EnvironmentName}.json", optional: true);
            configuration.AddEnvironmentVariables();
            Configuration = configuration;
```


In the method ```Startup``` make sure to set Application Insights settings overrides. Specifically, set developer mode to true in development environment:

``` C#
if (env.IsEnvironment("Development"))
{
    configuration.AddApplicationInsightsSettings(developerMode: true);
}
```

In the method ```ConfigureServices``` add Application Insights service. You'll need to add namespace ```Microsoft.ApplicationInsights.AspNet``` in the using list:
``` c#
services.AddApplicationInsightsTelemetry(Configuration);
```

In the method ```Configure``` add Application Insights request and exception tracking middleware. Please note that request tracking middleware should be added very as a very first middleware in pipeline:

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

##_Layout.cshtml (if present)
Define using and injection in the very top of the file:

``` html
@inject Microsoft.ApplicationInsights.Extensibility.TelemetryConfiguration TelemetryConfiguration 
```

And insert HtmlHelper to the end of ```<head>``` section but before any other script. Any custom javascript telemetry you want to report from the page should be injected after this snippet:

``` html
	@Html.ApplicationInsightsJavaScript(TelemetryConfiguration) 
</head>
```
