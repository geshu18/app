Agenda:

1. Integration points
    - [readme](https://github.com/Microsoft/AppInsights-aspnetv5/blob/master/Readme.md)
    - We need to add UseRequestServices(). Should we do it explicitly or implicitly inside "UseApplicationInsights()"?
2. DI design and Application Insights:
    - What is currently implemented
    - Discuss what we plan to implement
3. Getting routing information
    - Types of applications supported for routing?
    - Can/should we use mvc.core?
    - How can we access routing information?
4. Ship next week (without core support) - CTP6 or RC?
5. Build/test infra best practices


Current DI design
-----------------
This is how customers reports telemetry today:
```
TelemetryClient telemetry = new TelemetryClient();
telemetry.TrackEvent("WinGame");
```
```TelemetryClinet``` will be using configuration set in static singleton ```TelemetryConfiguration.Active``` (unless you pass new configuration as a parameter).

Every telemetry item should be associated with the list of context-specific properties. We are using mechanism of telemetry initializers to accomplish that. List of telemetry initializers is a property of ```TelemetryConfiguration.Active```.

```
// TODO: Register if customer did not register
app.UseRequestServices();
```


This is how we initialize application insights today:
```
public static void AddApplicationInsightsTelemetry(this IServiceCollection services, IConfiguration config)
{
  ActiveConfigurationManager.AddInstrumentationKey(TelemetryConfiguration.Active, config);

  services.AddSingleton((svcs) => {
    ActiveConfigurationManager.AddTelemetryInitializers(TelemetryConfiguration.Active, svcs);
    ActiveConfigurationManager.AddContextInitializers(TelemetryConfiguration.Active);

    return new TelemetryClient();
  });

  services.AddScoped<RequestTelemetry>((svcs) => {
    var rt = new RequestTelemetry();
    // this is workaround to inject proper instrumentation key into javascript:
    rt.Context.InstrumentationKey = svcs.GetService<TelemetryClient>().Context.InstrumentationKey;
    return rt;
  });
}
```

Here is how telemetry initializer will look like:
```
public class ClientIpHeaderTelemetryInitializer : ITelemetryInitializer
{
  public ClientIpHeaderTelemetryInitializer(IHttpContextAccessor httpContextAccessor)
    : base(httpContextAccessor)
  {
    this.httpContextAccessor = httpContextAccessor;
  }

  public void Initialize(ITelemetry telemetry)
  {
    var context = this.httpContextAccessor.Value;

    if (context == null)
    {
      //TODO: Diagnostics!
      return;
    }

    if (context.RequestServices == null)
    {
      //TODO: Diagnostics!
      return;
    }

    var request = context.RequestServices.GetService<RequestTelemetry>();

    if (request == null)
    {
      //TODO: Diagnostics!
       return;
     }

     this.OnInitializeTelemetry(context, request, telemetry);
  }
}
```

DI-based API
------------

Typical Usage
```
public class AccountController : Controller
{
    readonly ITelemetryClient telemetryClient;
    public AccountController(ITelemetryClient telemetryClient)
    {
        this.telemetryClient = telemetryClient;
    }
    public IActionResult Login()
    {
        this.telemetryClient.TrackEvent("User logged in");
        return View();
    }
}
```

Usage API
```
public interface ITelemetryClient
{
    void Track(ITelemetry telemetry);
}
public interface ITelemetry
{
    string InstrumentationKey { get; set; }
    TelemetryContext Context { get; }
}
public static class TelemetryExtensions
{
    public static void TrackEvent(this ITelemetryClient client, string name)
    {
        client.Track(new EventTelemetry { Name = name });
    }
}
public class EventTelemetry : ITelemetry
{
    public string InstrumentationKey { get; set; }
    public TelemetryContext Context { get; }
    public string Name { get; set; }
}
public class TelemetryContext
{
    public DeviceContext Device { get; }
    public OperationContext Operation { get; }
}
public class DeviceContext
{
    public string OSVersion { get; set; }
}
public class OperationContext
{
    public string Name { get; set; }
}
```
Configuration

```
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddInstance<ITelemetryInitializer>(new InstrumentationKeyInitializer("YOUR KEY"));
        services.AddSingleton<ITelemetryChannel, InMemoryTelemetryChannel>();
        services.AddSingleton<ITelemetryInitializer, OSVersionInitializer>();
        services.AddScoped<ITelemetryInitializer, OperationNameInitializer>();
        services.AddScoped<ITelemetryClient, TelemetryClient>();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory lf)
    {
        app.UseMiddleware<RequestTelemetryMiddleware>();
    }
}
```
Implementation

```
public class TelemetryClient : ITelemetryClient
{
    readonly ITelemetryChannel channel;
    readonly IEnumerable<ITelemetryInitializer> initializers;
    public TelemetryClient(ITelemetryChannel channel, IEnumerable<ITelemetryInitializer> initializers)
    {
        this.channel = channel;
        this.initializers = initializers;
    }
    public void Track(ITelemetry telemetry)
    {
        foreach (ITelemetryInitializer initializer in this.initializers)
            initializer.Initialize(telemetry);
        this.channel.Send(telemetry);
    }
}
```

Application-scoped Telemetry Initializers
```
public class OSVersionInitializer : ITelemetryInitializer
{
    public void Initialize(ITelemetry telemetry)
    {
        telemetry.Context.Device.OSVersion = Environment.OSVersion.VersionString;
    }
}
```

Request-scoped Telemetry Initializers
```
public class OperationNameInitializer : ITelemetryInitializer
{
    readonly IScopedInstance<ActionContext> actionContextAccessor;
    public OperationNameInitializer(IScopedInstance<ActionContext> actionContextAccessor)
    {
        this.actionContextAccessor = actionContextAccessor;
    }
    public void Initialize(ITelemetry telemetry)
    {
        telemetry.Context.Operation.Name = this.actionContextAccessor.Value.RouteData;
    }
}
```
Request-scoped Telemetry Modules
```
public class RequestTelemetryMiddleware
{
    readonly RequestDelegate next;
    public RequestTelemetryMiddleware(RequestDelegate next)
    {
        this.next = next;
    }
    public async Task Invoke(HttpContext context, ITelemetryClient client)
    {
        Stopwatch stopwatch = Stopwatch.StartNew();
        await this.next(context);
        var request = new RequestTelemetry();
        request.Duration = stopwatch.Elapsed;
        request.ResponseCode = context.Response.StatusCode.ToString();
        client.Track(request);
    }
}
```
Open Questions
```
- How to access application and request scoped services in 
    - Application-scoped Telemetry Modules (UnobservedTaskException)
    - Logging adapters (Trace.Listener, NLog.Target, Log4Net.Appender) 
- How to remove from services?
```

Routes
------

Here is how a request-scoped component can access MVC route data. This is an injection constructor, but it can also be resolved from ```HttpContext.RequestServices``` service provider.

```
public MyComponent(IScopedInstance<ActionContext> actionContextAccessor)
{
  RouteData foo = actionContextAccessor.Value.RouteData;
}
```

The code that makes it possible is in the ```InvokeActionAsync``` method of the ```MvcRouteHandler```. 

Notice, however, that ```ActionContext``` is MVC specific while ```RouteData``` is not. Ideally we’d want to avoid MVC dependencies and support other One ASP.NET frameworks, like WebForms, Dynamic Data, Web Pages, all of which support modern URL routing and would be a subject to the same cardinality problems with our service.

Ideally, I think that we want to work with the ASP.NET team and implement this in the ```RouterMiddleware``` so that we can access ```IScopedInstance<RouteContext>``` in our components instead of relying on platform-specific workarounds.


