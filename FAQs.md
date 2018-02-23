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

## 2. Add FAQ here.

## 3. Add FAQ here.

