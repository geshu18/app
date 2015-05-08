For standard Asp.Net template you need to modify four files (this will be the default template instrumentation in future) - add Application Insights NuGet package, configure it, initialize and inject tracking javascript on base layout.

##project.json
Add new reference:
```
"Microsoft.ApplicationInsights.AspNet": "0.32.0-beta4"
```

##config.json
Configure instrumentation key. You need azure subscription to get instrumentation key. [This instruction](http://azure.microsoft.com/en-us/documentation/articles/app-insights-java-get-started/) explains how to get instrumentation key:
```
 "ApplicationInsights": {
 	"InstrumentationKey": "11111111-2222-3333-4444-555555555555"
 }
```

##Startup.cs
In the method ```Startup``` make sure to set Application Insights settings overrides. Specifically, set developer mode to true in development environment:

```
if (env.IsEnvironment("Development"))
{
    configuration.AddApplicationInsightsSettings(developerMode: true);
}
```

In the method ```ConfigureServices``` add Application Insights service. You'll need to add namespace ```Microsoft.ApplicationInsights.AspNet``` in the using list:
```
services.AddApplicationInsightsTelemetry(Configuration);
```

In the method ```Configure``` add application insights request and exception tracking middleware. Please note that request tracking middleware should be added very as a very first middleware in pipeline:

```
// Add Application Insights monitoring to the request pipeline as a very first middleware.
app.UseApplicationInsightsRequestTelemetry();
```
Exception middleware should be added after error page and any other error handling middleware:

```
// Add Application Insights exceptions handling to the request pipeline.
app.UseApplicationInsightsExceptionTelemetry();
```

##_Layout.cshtml
Define using and injection in the very top of the file:

```
@inject Microsoft.ApplicationInsights.Extensibility.TelemetryConfiguration TelemetryConfiguration 
```

And insert HtmlHelper to the end of ```<head>``` section but before any other script. Any custom javascript telemetry you want to report from the page should be injected after this snippet:

```
	@Html.ApplicationInsightsJavaScript(TelemetryConfiguration) 
</head>
```
