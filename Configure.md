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

Or any other configuration provider format you defined in your application (configuration setting name is ApplicationInsights:InstrumentationKey).

Code-based approach
-------------------


Track custom trace/event/metric
===============================

Add additional context properties
=================================

Add additional telemetry item properties
========================================

Remove default telemetry initializers
=====================================

Redirect traffic to the different endpoint
==========================================


