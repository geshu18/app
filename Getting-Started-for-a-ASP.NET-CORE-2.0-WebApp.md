[Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net) is an extensible analytics platform that monitors the performance and usage of your live ASP.NET Core web applications.

**With Asp.Net Core 2.0, ApplicationInsights is included by default.**
You only need to configure the instrumentation key so that telemetry is sent to ApplicationInsights service.
If instrumentation key is not added:
* ApplicationInsights will be enabled while debugging under 
Visual Studio 2017.
* ApplicationInsights telemetry will be shown in Visual Studio 2017. 
* Nothing gets sent to the ApplicationInsights service, unless an instrumentation key is added.

You probably don't need to follow these manual steps below (except the instrumentation key section) unless you upgraded from a previous version.

Follow the steps below, skippings the ones as indicated:

* Create an Application Insights resource
* Add Application Insights NuGet package dependency to `.csproj`
* Add Application Insights instrumentation key
* Add Application Insights instrumentation in code  
* Add Application Insights JavaScript instrumentation to the `_ViewImports.cshtml`,  `_Layout.cshtml`  

## Create an Application Insights resource

1. Sign in to the [Microsoft Azure portal](https://portal.azure.com). (Need to [sign up](https://azure.microsoft.com/pricing/free-trial/)?)
2. Create a new Application Insights resource. (Click **New -> Monitoring + management -> Application Insights**). Select the ASP.NET application type.
3. In your new resource, open the **Essentials** drop-down so that you can copy the Instrumentation Key - you'll need it shortly. 

## Add Application Insights NuGet package dependency to `.csproj`

If the following package reference already exists in you `.csproj`, then there is no need to explicitly add ApplicationInsights package. This is a meta package
and it contains the ApplicationInsights package.
```
<PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />
```
If the above package reference is not present, then add the following to explicitly add ApplicationInsights.

```
<PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.2.0" />
```

Verify the version number: get the latest from the [Releases page](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases). 

As of now, the version of Microsoft.ApplicationInsights.AspNetCore included in Microsoft.AspNetCore.All (v2.0.0) meta package is 2.1.1.
You can explicitly add a reference to Microsoft.ApplicationInsights.AspNetCore when you want a version higher than the one shipped with Microsoft.AspNetCore.All
metapackage.

## Add the instrumentation key. 

### Option 1: Use `appsettings.json`. 

You can control whether telemetry is sent to ApplicationInsights service by setting/not setting instrumentation key. You
may have separate `appsettings.json` for each environment, and each can contain different instrumentation keys, or do not contain anything at all if you do not want
telemetry to be sent to ApplicationInsights service.

```JSON
{
  "ApplicationInsights": {
    "InstrumentationKey": "11111111-2222-3333-4444-555555555555"
  }
}
```
### Option 2:
Set instrumentation key into environment variable  ApplicationInsights:InstrumentationKey
eg:set ApplicationInsights:InstrumentationKey=ikeygoeshere

Both the above options work only if Application Insights is enabled is code with UseApplicationInsights() extension. As of now its not supported with AddApplicationInsightsTelemetry() as mentioned [here](https://github.com/Microsoft/ApplicationInsights-aspnetcore/issues/605)

### Option 3:
Add instrumentation key in code while enabling ApplicationInsights instrumentation. See the following section. 

## Add Application Insights instrumentation in code.

There are two options for instrumenting your code. These are intended to be mutually exclusive, only use one or the other and not both together!

Both options below allow passing an instrumentation key as a parameter to the call `AddApplicationInsightsTelemetry()` or `UseApplicationInsights()`. This is required if the key
is not set via options mentioned at the beginning.

### Option 1: Program.cs (Recommended)

The preferred default mechanism is to use the `UseApplicationInsights` extension method from the `WebHostBuilder` instance.  

Starting with Asp.Net Core 2.0, all project templates include a call to `WebHost.CreateDefaultBuilder()` in `Program.cs` which automatically does everything needed to read settings from appsettings.json/environment variables and initialize configuration variables.

```C#            
public static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .Build();       
```

Modify the above code as follows to configure ApplicationInsights.

```C#
public static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseApplicationInsights()
        .Build();
```

### Option 2: Startup.cs

The older extension method `AddApplicationInsightsTelemetry` on the `IServiceCollection` is still available for customized configuration using one of its overloaded signatures.

In the method `ConfigureServices` of your Startup class, add the Application Insights service as follows:

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddApplicationInsightsTelemetry();
}
```

This method has an override which can accept configuration as well. 
```C#
var configBuilder = new ConfigurationBuilder()
                .SetBasePath(this.hostingEnvironment.ContentRootPath)
                .AddEnvironmentVariables();
var Configuration= configBuilder.Build();
services.AddApplicationInsightsTelemetry(Configuration);
```



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