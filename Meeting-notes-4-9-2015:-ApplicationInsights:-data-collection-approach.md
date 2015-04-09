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


```
// TODO: Register if customer did not register
app.UseRequestServices();
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

Ideally, I think that we want to work with the ASP.NET team and implement this in the RouterMiddleware so that we can access IScopedInstance<RouteContext> in our components instead of relying on platform-specific workarounds.



