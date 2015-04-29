Currently NuGet built against VS 2015 RC.

For standard Asp.Net template you need to modify four files (this will be the default template instrumentation in future).

***project.json*** 
Add new reference:
```
"Microsoft.ApplicationInsights.AspNet": "0.31.0-beta4"
```

***config.json*** 
Configure instrumentation key. You need azure subscription to get instrumentation key. [This instruction](http://azure.microsoft.com/en-us/documentation/articles/app-insights-java-get-started/) explains how to get instrumentation key:
```
 "ApplicationInsights": {
 	"InstrumentationKey": "11111111-2222-3333-4444-555555555555"
 }
```

***Startup.cs***
In the method ```ConfigureServices``` add Application Insights service. You'll need to add namespace ```Microsoft.ApplicationInsights.AspNet``` in the using list:
```
services.AddApplicationInsightsTelemetry(Configuration);
```

In the method ```Configure``` add application insights request and exception tracking middleware. Please note that request tracking middleware should be added very as a very first middleware in pipeline, exception middleware should be added after error page and any other error handling middleware:

```
// Add Application Insights monitoring to the request pipeline as a very first middleware.
app.UseApplicationInsightsRequestTelemetry();
...
// Add the following to the request pipeline only in development environment.
if (string.Equals(env.EnvironmentName, "Development", StringComparison.OrdinalIgnoreCase))
{
	app.SetApplicationInsightsTelemetryDeveloperMode();
}
...
// Add Application Insights exceptions handling to the request pipeline.
app.UseApplicationInsightsExceptionTelemetry();
```

***_Layout.cshtml***
Define using and injection in the very top of the file:

```
@using Microsoft.ApplicationInsights.AspNet
@inject Microsoft.ApplicationInsights.DataContracts.RequestTelemetry RequestTelelemtry
```

And insert HtmlHelper to the end of ```<head>``` section. Any custom javascript telemetry you want to report from the page should be injected after this snippet:

```
	@Html.ApplicationInsightsJavaScriptSnippet(RequestTelelemtry.Context.InstrumentationKey);
</head>
```
