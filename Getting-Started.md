For standard ASP.NET 5 template you need to modify four files (this will be the default template instrumentation in future) - add Application Insights NuGet package, configure it, initialize and inject tracking javascript on base layout.

##project.json
Add new entry in dependencies section:
``` json
"Microsoft.ApplicationInsights.AspNet": "0.32.0-beta4"
```

##config.json
If config.json file is not already part of your application, create it on the root level of your project.

Configure instrumentation key. You need azure subscription to get instrumentation key. [This instruction](http://azure.microsoft.com/en-us/documentation/articles/app-insights-java-get-started/) explains how to get instrumentation key:
Once you have your instrumentation key, add the following section in the root of your config.json file.
``` json
 "ApplicationInsights": {
 	"InstrumentationKey": "11111111-2222-3333-4444-555555555555"
 }
```

##Startup.cs
If you don't already have the code that parses config.json file and initializes configuration variable, create Configuration variable
``` C#
public IConfiguration Configuration { get; set; }
```

 add it to your Startup method:

``` C#
  // Setup configuration sources.
            var configuration = new Configuration()
                .AddJsonFile("config.json")
                .AddJsonFile($"config.{env.EnvironmentName}.json", optional: true);
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

##_Layout.cshtml
Define using and injection in the very top of the file:

``` html
@inject Microsoft.ApplicationInsights.Extensibility.TelemetryConfiguration TelemetryConfiguration 
```

And insert HtmlHelper to the end of ```<head>``` section but before any other script. Any custom javascript telemetry you want to report from the page should be injected after this snippet:

``` html
	@Html.ApplicationInsightsJavaScript(TelemetryConfiguration) 
</head>
```
