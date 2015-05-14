#Request processing and telemetry
Application Insights SDK for ASP.NET 5 applications allows to track requests and exception information. For every request it collects:

- request name, 
- url, 
- timestamp, 
- duration, 
- status code, 
- whether request is successful or not, 
- exception happened during processing of this request

This article explains how this data collection works and how to configure it for different scenarios.

##Request tracking middleware
Request telemetry middleware measures request duration and then reads request properties like name and status code and sends ```RequestTelemetry``` item to backend.

By default request will be marked as failed if response status code is 400+. However if unhandled exception passes this middleware - request will be marked as failed independently from status code. Typically unhandled exception in request telemetry middleware indicates that error handling middleware wasn't configured for the application.

##Exception tracking middleware
Exception tracking middleware catches and re-throws exceptions from the next middleware down the request processing pipeline. 

##Typical application
Typical application pipeline starts with error handling middleware followed by static files, identity and then middleware implementing business logic.

For this pipeline you need to insert request tracking middleware (```UseApplicationInsightsRequestTelemetry```) as a very first middleware. Exception tracking middleware should be inserted right after error handling middleware such as ```ErrorPage``` middleware. 

This configuration ensures that every exception thrown by identity and business logic middleware will be caught and reported by exception tracking middleware and request tracking middleware will report accurate request duration and status code.

##Exceptional cases
There are situations when telemetry reported by Application Insights may not look correct:

- If application has no error handling middleware Application Insights will report response status code ```200``` when unhandled exception is thrown. Actual status code will be ```500```.
- If middleware [UseStatusCodePagesWithRedirects](https://github.com/aspnet/Diagnostics/blob/b1643b438aa947370868b4d5ee7727c27f2d78cb/src/Microsoft.AspNet.Diagnostics/StatusCodePagesExtensions.cs#L76) is used - failed requests will be reported as successful with the status code ```302```.

You also may need to fine tune certain telemetry data:
- Do not report expected exceptions
- Mark some of ```404``` requests as successful so failed requests count will not be too high

###Runaway exceptions


