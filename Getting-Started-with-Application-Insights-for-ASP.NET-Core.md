Please follow official docs here: https://docs.microsoft.com/en-us/azure/azure-monitor/app/asp-net-core-no-visualstudio

The contents of this is wiki is no longer updated.


[Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net) is an extensible analytics platform that monitors the performance and usage of your live ASP.NET Core web applications.

**With Asp.Net Core 2.0, ApplicationInsights is included by default when running from Visual Studio 2017.**
You only need to configure the instrumentation key so that telemetry is sent to Application Insights service.
If instrumentation key is not added:
* Application Insights will be enabled while debugging under 
Visual Studio 2017.
* Application Insights telemetry will be shown locally in Visual Studio 2017. 
* Nothing gets sent to the Azure Application Insights service, unless an instrumentation key is added.

**If running from outside Visual Studio 2017, like from a command-line (using `dotnet run myapp.dll`) or other IDE like Visual Studio Code, then Application Insights need to be explicitly added/configured.**

Follow the steps below, skipping the ones as indicated:

1. * Create an Application Insights resource
2. * Add Application Insights NuGet package dependency to `.csproj`
3. * Add Application Insights instrumentation key
4. * Add Application Insights instrumentation in code  
5. * Add Application Insights JavaScript instrumentation to the `_ViewImports.cshtml`,  `_Layout.cshtml`  
6. * Customizing Application Insights Settings.
## 1. Create an Application Insights resource

1. Sign in to the [Microsoft Azure portal](https://portal.azure.com). (Need to [sign up](https://azure.microsoft.com/pricing/free-trial/)?)
2. Create a new Application Insights resource. (Click **New -> Monitoring + management -> Application Insights**). Select the ASP.NET application type.
3. In your new resource, open the **Essentials** drop-down so that you can copy the Instrumentation Key - you'll need it shortly. 

## 2. Add Application Insights NuGet package dependency to `.csproj`

If the following package reference already exists in your `.csproj`, then there is no need to explicitly add Application Insights package. This is a meta package
and it contains the Application Insights package.
```
<PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />
```
If the above package reference is not present, then add the following to explicitly add Application Insights.

```
<PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.3.0-beta1" />
```

Verify the version number: get the latest from the [Releases page](https://github.com/Microsoft/ApplicationInsights-aspnetcore/releases). 

As of now, the version of Microsoft.ApplicationInsights.AspNetCore included in Microsoft.AspNetCore.All (v2.0.0) meta package is 2.1.1.
You can explicitly add a reference to Microsoft.ApplicationInsights.AspNetCore when you want a version higher than the one shipped with Microsoft.AspNetCore.All metapackage. Also, starting with Asp.Net Core 2.1, this meta package will no longer contain Microsoft.ApplicationInsights.AspNetCore, so you need to add it manually to `.csproj`.

## 3. Add the instrumentation key. 

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
### Option 2: Environment Variable
Set instrumentation key into environment variable `APPINSIGHTS_INSTRUMENTATIONKEY`.

Both the above options work only if Application Insights is enabled is code with UseApplicationInsights() extension. As of now its not supported with AddApplicationInsightsTelemetry() as mentioned here - Update 2.3.0 fixes this issue.

### Option 3: Via Code
Add instrumentation key in code while enabling ApplicationInsights instrumentation. See the following section. 

## 4. Add Application Insights instrumentation in code.

There are two options for instrumenting your code. These are mutually exclusive, only use one or the other and not both!

Both options below allow passing an instrumentation key as a parameter to the call `AddApplicationInsightsTelemetry()` or `UseApplicationInsights()`. This is required if the key
is not set via options mentioned previously.

### Option 1: `UseApplicationInsights` (Preferred if using default Application Insights settings)

The easiest way is to use the `UseApplicationInsights` extension method from the `WebHostBuilder` instance.

Starting with Asp.Net Core 2.0, all project templates include a call to `WebHost.CreateDefaultBuilder()` in `Program.cs`.
Add `UseApplicationInsights()` here to configure ApplicationInsights.
This will automatically read settings from appsettings.json/environment variables and initialize configuration variables.


```C#
public static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseApplicationInsights() // Enable Application Insights
        .Build();
```

### Option 2: `AddApplicationInsightsTelemetry` (Preferred if customizing Application Insights settings)

The extension method `AddApplicationInsightsTelemetry` on the `IServiceCollection` is recommended for customized configuration using one of its overloaded signatures.

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

## 5. Add Application Insights JavaScript instrumentation to the  _ViewImports.cshtml ,  _Layout.cshtml  

This section is applicable only if you have client-side components and need to collect client side telemetry as well (like page views etc)

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

## 6. Customizing Application Insights
TODO: Add link to configuration

## Get more telemetry

* [Monitor dependencies](https://azure.microsoft.com/documentation/articles/app-insights-dependencies/) to see whether REST, SQL or other external resources are slowing you down.
* [Use the API](https://azure.microsoft.com/documentation/articles/app-insights-api-custom-events-metrics/) to send your own events and metrics for a more detailed view of your app's performance and usage.
* [Availability tests](https://azure.microsoft.com/documentation/articles/app-insights-monitor-web-app-availability/) check your app constantly from around the world.