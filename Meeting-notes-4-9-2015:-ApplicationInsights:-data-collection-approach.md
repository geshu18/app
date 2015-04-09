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

4. Ship next week (without core support).
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


