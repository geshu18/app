[Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net) is an extensible analytics platform that monitors the performance and usage of your live ASP.NET Core web applications.

> **Visual Studio 2017 automatically configures ASP.NET Core web projects.** You probably don't need to follow these manual steps. Instead, right-click your project in Solution Explorer and look for **Configure Application Insights** or **Add > Application Insights**. [Learn more](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net).

Follow the details in this article to make the following changes:

* Create an Application Insights resource
* Add Application Insights NuGet package dependency to `project.json`  
* Add Application Insights instrumentation key to the `appsettings.json`  
* Add Application Insights instrumentation code to the `startup.cs`  
* Add Application Insights JavaScript instrumentation to the `_ViewImports.cshtml`,  `_Layout.cshtml`  

## Create an Application Insights resource

1. Sign in to the [Microsoft Azure portal](https://portal.azure.com). (Need to [sign up](https://azure.microsoft.com/pricing/free-trial/)?)
2. Create a new Application Insights resource. (Click **New -> Monitoring + management -> Application Insights**). Select the ASP.NET application type.
3. In your new resource, open the **Essentials** drop-down so that you can copy the Instrumentation Key - you'll need it shortly. 

## Add Application Insights NuGet package dependency to `project.json`

Add the following entry to the  `dependencies` section. 

```JSON
{
  "dependencies": {
    "Microsoft.ApplicationInsights.AspNetCore": "2.0.0"
  }
}
```

Verify the version number: get the latest from the [Releases page](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases). 

In the case of **.NET Core** applications, if you run into restore errors with respect to Application Insights dependency, please add "dnxcore50" and "portable-net45+win8" to the imports list (if it does not exist), under the frameworks section of  `project.json`, as shown below. Please see [Migrating from DNX](https://docs.microsoft.com/en-us/dotnet/articles/core/migrating-from-dnx) for more details.

```JSON
{
    "frameworks": {
        "netcoreapp1.0": { 
            "imports": ["dnxcore50", "portable-net45+win8"]
        }
    }
}
```


## Add the instrumentation key to `appsettings.json`

Add the instrumentation key of your Application Insights web application resource to the  ApplicationInsights  section of `appsettings.json`. 

```JSON
{
  "ApplicationInsights": {
    "InstrumentationKey": "11111111-2222-3333-4444-555555555555"
  }
}
```

## Add Application Insights instrumentation code to startup.cs 

If you don't already have the code that parses the appsettings.json file and initializes the configuration variable then create Configuration variable:

```

public IConfigurationRoot Configuration { get; set; }
```

Then add the following dependency entries to your project.json if they are not defined there already.

```JSON
    "Microsoft.Extensions.Configuration.Abstractions": "1.0.0",
    "Microsoft.Extensions.Configuration.Json":  "1.0.0"
```

(Verify the version number.)

Then add the code that parses configuration if you don't have it already in the method `Startup`.

```C#
            // Setup configuration sources.
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();

            Configuration = builder.Build();
```

If you have a `Main` method (e.g. in Program.cs) with a call to create a ```new WebHostBuilder()``` then make sure it has a chained call to ```UseApplicationInsights()``` like the example below:

```C#
        public static void Main(string[] args)
        {
            var host = new WebHostBuilder()
                .UseKestrel()
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseIISIntegration()
                .UseStartup<Startup>()
                .UseApplicationInsights()
                .Build();

            host.Run();
        }
```

Otherwise, in the method `ConfigureServices` add the Application Insights service. You'll need to add the namespace `Microsoft.ApplicationInsights.AspNetCore` to the using list:

```C#
services.AddApplicationInsightsTelemetry(Configuration);
```

Please note that `AddApplicationInsightsTelemetry` and `UseApplicationInsights` are intended to be mutually exclusive, only use one or the other and not both together.  The preferred default mechanism is to use the `UseApplicationInsights` extension method from the `WebHostBuilder` instance.  The older extension method `AddApplicationInsightsTelemetry` on the IServiceCollection is still available for customized configuration using one of its overloaded signatures.

## Add Application Insights JavaScript instrumentation to the  _ViewImports.cshtml ,  _Layout.cshtml  

(if present)

In `_ViewImports.cshtml`, add injection:

```
@inject Microsoft.ApplicationInsights.AspNetCore.JavaScriptSnippet JavaScriptSnippet
```

In `_Layout.cshtml`, insert HtmlHelper to the end of `<head>` section but before any other script. Any custom JavaScript telemetry you want to report from the page should be injected after this snippet:

```
    @Html.Raw(JavaScriptSnippet.FullScript)
</head>
```

## Run your app and view your telemetry

* Use [Search Explorer](https://azure.microsoft.com/documentation/articles/app-insights-diagnostic-search/) to investigate individual requests, exceptions, log traces and other events.
* Use [Metrics Explorer](https://azure.microsoft.com/documentation/articles/app-insights-metrics-explorer/) to slice and dice metrics and counts such as response times, page views and user counts.
* [Set alerts](https://azure.microsoft.com/documentation/articles/app-insights-alerts/) to notify you when your app goes off-limits.

## Get more telemetry

* [Monitor dependencies](https://azure.microsoft.com/documentation/articles/app-insights-dependencies/) to see whether REST, SQL or other external resources are slowing you down.
* [Use the API](https://azure.microsoft.com/documentation/articles/app-insights-api-custom-events-metrics/) to send your own events and metrics for a more detailed view of your app's performance and usage.
* [Availability tests](https://azure.microsoft.com/documentation/articles/app-insights-monitor-web-app-availability/) check your app constantly from around the world. 
