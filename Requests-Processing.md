#Request processing and telemetry
Application Insights SDK for ASP.NET Core web applications allows to track requests and exception information. For every request it collects:

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

- *Runaway exceptions*: If application has no error handling middleware Application Insights will report response status code ```200``` when unhandled exception is thrown. Actual status code will be ```500```.
- *Redirect to error page*: If middleware [UseStatusCodePagesWithRedirects](https://github.com/aspnet/Diagnostics/blob/b1643b438aa947370868b4d5ee7727c27f2d78cb/src/Microsoft.AspNet.Diagnostics/StatusCodePagesExtensions.cs#L76) is used - failed requests will be reported as successful with the status code ```302```.
- *SignalR*: SignalR requests are very long running and their duration can affect aggregated request duration metrics.

You also may need to fine tune certain telemetry data:
- *Expected exceptions*: Do not report expected exceptions
- *Expected error codes*: Mark some of ```404``` requests as successful so failed requests count will not be too high


###Runaway exceptions
Not handling exceptions and rely on framework behavior is considered bad practice. You will need to implement your own middleware to properly report response status code in this case.

###Redirect to error page
If your error processing logic redirects request (returns ```302```) to error page in case of exception you may want to insert Request Tracking Module after error-redirection module.

###SignalR
For long running requests that can affect aggregations you need to implement telemetry initializer that will reset duration of such requests to zero. See [Configure](https://github.com/Microsoft/ApplicationInsights-aspnet5/wiki/Configure/) section to find out how to create a telemetry initializer.

###Expected exceptions
If expected exception is handled by dedicated middleware - consider adding exception tracking middleware before this middleware so it will not see this exception.

###Expected error codes
For error codes like ```404``` you may want to report request as successful so "failed requests" metric will not be affected. You need to implement telemetry initializer to reset successful flag to true for such requests. See [Configure](https://github.com/Microsoft/ApplicationInsights-aspnet5/wiki/Configure/) section to find out how to create a telemetry initializer.
