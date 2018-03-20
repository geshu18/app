## 1.What is the different between using UseApplicationInsights() vs AddApplicationInsightsTelemetry()
UseApplicationInsights() is a newly added extension method on ```IWebHostBuilder``` to easily enable Application Insights with all default settings. This is the recommended way to add Application Insights for most cases.

AddApplicationInsightsTelemetry() extension method on  ```IServiceCollection``` allows customization of Application Insights settings. ```ApplicationInsightsServiceOptions``` provides quick access to most frequently used settings. 
```
			ApplicationInsightsServiceOptions aiOptions = new ApplicationInsightsServiceOptions();
		            aiOptions.InstrumentationKey = "ikeygoeshere";
		            aiOptions.EnableAdaptiveSampling = false;
		            aiOptions.EnableQuickPulseMetricStream = false;
            services.AddApplicationInsightsTelemetry(aiOptions);
```

## 2. Accessing TelemetryConfiguration via TelemetryConfiguration.Active vs using DI
Telemetry configuration can be accessed either using Active
```var telemetryConfiguration = TelemetryConfiguration.Active```
or by getting it through application builder services (DI Approach)
```
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerfactory)
{
    var configuration= app.ApplicationServices.GetService<TelemetryConfiguration>();
}
```
However it is strongly recommended to use the DI based approach. Other methods could potentially introduce hard-to-track bugs like the ones mentioned here:
https://github.com/Microsoft/ApplicationInsights-dotnet/issues/613

This is especially true from 2.3.0-beta1 onwards as this PR introduces changes such that modifications to TelemetryConfiguration.Active will not be picked up by SDK's own modules.
https://github.com/Microsoft/ApplicationInsights-aspnetcore/pull/622

## 3. Add FAQ here.