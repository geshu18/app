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

For this pipeline you need to insert request telemetry middleware (```UseApplicationInsightsRequestTelemetry```) as a very first middleware. 




