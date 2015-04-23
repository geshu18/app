Add NuGet feed http://appinsights-aspnet.azurewebsites.net/nuget/. It has NuGet: Microsoft.ApplicationInsights.AspNet. 

Currently NuGet built against VS 2015 RC.

For standard Asp.Net template you need to modify four files (this will be the default template instrumentation in future).

***project.json*** 
Add new reference:
```
"Microsoft.ApplicationInsights.AspNet": "0.30.0.1-beta"
```

***config.json*** 
Configure instrumentation key:
```
 "ApplicationInsights": {
 	"InstrumentationKey": "11111111-2222-3333-4444-555555555555"
 }
```

***Startup.cs***
Add service:
```
services.AddApplicationInsightsTelemetry(Configuration);
```

Add middleware and configure developer mode: 

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
Define using and injection:

```
@using Microsoft.ApplicationInsights.AspNet
@inject Microsoft.ApplicationInsights.DataContracts.RequestTelemetry RequestTelelemtry
```

And insert HtmlHelper to the end of ```<head>``` section:

```
	@Html.ApplicationInsightsJavaScriptSnippet(RequestTelelemtry.Context.InstrumentationKey);
</head>
```
