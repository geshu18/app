AspNet 5 team is working on [EventNotification](https://github.com/aspnet/eventnotification) API that may be useful to replace existing automatic data collection for Asp.Net 5 applications that is based on middleware injection.

#Problem statement
We want to eliminate the need for customer to inject middleware for simple request tracking:
``` csharp
app.UseApplicationInsightsRequestTelemetry();
app.UseApplicationInsightsExceptionTelemetry();
```
This injection may be error-prone and requires customer to understand pipeline quite good. There will be a risk that some of the telemetry would not be collected.

#Requirements

##Request tracking
- Get notification when request is started. Have access to the request headers
- If "MVC" request - get notification when name of action and controller were defined so we can use them as a name of request reported to Application Insights
- Add RequestTelemetry object to be accessible via DI for the duration of request
- Have notification when request complete. Have access to request and response headers. Ideally - ability to modify response headers

##Exception tracking
- Receive exception object from ErrorHandling middleware
- Receive exception object when exception was unhandled by ANY middleware

- Associate exception with the current request object