Set your Instrumentation Key
============================
There are two ways to set instrumentation key - using configuration or via code

Configuration-based approach
----------------------------
If you are using json configuration provider - add the following into config.json
```
"ApplicationInsights": {
    "InstrumentationKey": "11111111-2222-3333-4444-555555555555"
}
```
If you are using environment variables:
```
SET ApplicationInsights:InstrumentationKey=11111111-2222-3333-4444-555555555555
```
***note:*** *Environment variable that is set by azure website (APPINSIGHTS_INSTRUMENTATIONKEY) is not supported.*

Or any other configuration provider format you defined in your application (configuration setting name is ApplicationInsights:InstrumentationKey) before the call to add insights telemetry in `Startup.cs`:
```
services.AddApplicationInsightsTelemetry(Configuration);
```

Code-based approach
-------------------
The recommended way to modify Application Insights Configuration is by obtaining the instance of TelemetryConfiguration from DI and then modifying it.
```
var tConfig = app.ApplicationServices.GetRequiredService<TelemetryConfiguration>();
tConfig.InstrumentationKey = "mynewikeyhere";
```
Note: It is not recommended to modify the `TelemetryConfiguration.Active` as changes may not be picked up by the auto collection modules.

Track custom trace/event/metric
===============================
Please refer to [Application Insights custom metrics API reference](http://azure.microsoft.com/en-us/documentation/articles/app-insights-custom-events-metrics-api/) for description of custom data reporting in Application Insights.

Get TelemetryClient using constructor dependency injection: 
``` c#
public class HomeController : Controller
{
    private TelemetryClient telemetry;

    public HomeController(TelemetryClient telemetry)
    {
        this.telemetry = telemetry;
    }
```
Track event or any methods on `TelemetryClient`:
```
    public IActionResult Index()
    {
        this.telemetry.TrackEvent("HomePageRequested");
        return View();
    }
```

Add request-level properties for all telemetry items
---------------------------------------------------- 
In order to set request level context properties for all telemetry items you need to configure your telemetry initializer.

First, define new class with IHttpContextAccessor dependency. This class will get property from the current http context and set corresponding property on telemetry data item:
``` c#
public class PropertyTelemetryInitializer : ITelemetryInitializer
{
    IHttpContextAccessor httpContextAccessor;

    public PropertyTelemetryInitializer(IHttpContextAccessor httpContextAccessor)
    {
        this.httpContextAccessor = httpContextAccessor;
    }

    public void Initialize(ITelemetry telemetry)
    {
        telemetry.Context.Properties["tenantName"] = httpContextAccessor.Value.Items["tenantName"].ToString();
    }
}
```
Secondly, register your telemetry initializer into global collection:
```
public void ConfigureServices(IServiceCollection services)
{
services.AddSingleton<ITelemetryInitializer, PropertyTelemetryInitializer>();
services.AddApplicationInsightsTelemetry()
...
}
``` 

Configure Telemetry initializers
=====================================
## Adding new TelemetryInitializer.
Create new Initializer.

Register it into DI Containers before `AddApplicationInsightsTelemetry()` is called in the ConfigureServices of your Startup class.
For eg:
```
// Use this if MyCustomTelemetryInitializer can be constructed without DI injected parameters
services.AddSingleton<ITelemetryInitializer>(new MyCustomTelemetryInitializer());
```
OR
```
// Use this if MyCustomTelemetryInitializer constructor has parameters which are DI injected.
services.AddSingleton<ITelemetryInitializer, MyCustomTelemetryInitializer>();
```
This will ensure the telemetry initializer will be part of the TelemetryConfiguration object.

Similar approach can be used to remove all or specific TelemetryInitializers. Use the following logic **after** the call to `AddApplicationInsightsTelemetry()` is made

``` c#
public void ConfigureServices(IServiceCollection services)
{
..
services.AddApplicationInsightsTelemetry("ikey");
// Remove a specific built-in TelemetryInitializer
var tiToRemove = services.FirstOrDefault<ServiceDescriptor>(t => t.ImplementationType == typeof(AspNetCoreEnvironmentTelemetryInitializer));
            if (tiToRemove != null)
            {
                services.Remove(tiToRemove);
            }

// or Remove all 
services.RemoveAll(typeof(ITelemetryInitializer)); // this requires importing namespace using Microsoft.Extensions.DependencyInjection.Extensions;
..
}
```

Redirect traffic to the different endpoint
==========================================

Configuration-based approach
----------------------------
If you are using json configuration provider - add the following into config.json
```
"ApplicationInsights": {
    "InstrumentationKey": "ikey",
    "TelemetryChannel": {
      "EndpointAddress": "http://localhost:8888/v2/track"
    }
  }
```
Then use UseApplicationInsights() method to add application insights.

If using AddApplicationInsightsTelemetry(), construct Configuration first and pass it to AddApplicationInsightsTelemetry in ConfigureServices method of Startup class as shown in below example.
```C#
var configBuilder = new ConfigurationBuilder()
               .SetBasePath(this.HostingEnvironment.ContentRootPath)
               .AddJsonFile("appsettings.json", true)               
               .AddEnvironmentVariables();            
            services.AddApplicationInsightsTelemetry(configBuilder.Build());
```

Configuring Telemetry Channel
==========================================
Add the required channel to DI in ConfigureServices() method of your application's Startup class before `AddApplicationInsightsTelemetry()` call.
For eg:, the following sets up `InMemoryChannel` with the specific `MaxTelemetryBufferCapacity`
```
services.AddSingleton(typeof(ITelemetryChannel), new InMemoryChannel() {MaxTelemetryBufferCapacity = 19898 });
```
If no channel is explicitly added by user, then SDK by default sets up ServerTelemetryChannel. Adding any ITelemetryChannel to DI overrides default channel.



