AspNet 5 team is working on [EventNotification](https://github.com/aspnet/eventnotification) API that may be useful to replace existing automatic data collection for Asp.Net 5 applications.

#Problem statement
We want to eliminate the need for customer to inject middleware for simple request tracking:
``` csharp
app.UseApplicationInsightsRequestTelemetry();
app.UseApplicationInsightsExceptionTelemetry();
```
This injection may be error-prone and requires customer to understand pipeline quite good. There will be a risk that some of the telemetry would not be collected.

#Requirements

##Request tracking

##Exception tracking
- Receive exception object from ErrorHandling middleware
- Receive exception object when exception was unhandled by ANY middleware

- Associate exception with the current request object