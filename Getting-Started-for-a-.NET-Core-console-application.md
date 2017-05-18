This page is about getting started with setting up application insights for a **.NET Core console application**.  If you are interested in getting started with an **ASP.NET Core Application** please refer to this [Getting Started](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Getting-Started) guide instead.

# Walkthrough
## Create an Application Insights Resource
Please refer to [Create an Application Insights resource](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-create-new-resource) to create an application insights resource, keep a copy of the instrumentation key.

## Build the App
### 1. Add nuget package(s):

The base AI nuget package is the minimum to get Application Insights to work with your .NET Core console application:

```
 PM> Install-Package Microsoft.ApplicationInsights -Version 2.3.0
```
Find the latest package version on [nuget.org](https://www.nuget.org/packages/Microsoft.ApplicationInsights/).

### 2. Write code to configure and use application insights:

The following example code shows how to set your instrumentation key on the Application Insights default configuration instance and create and use a ```TelemetryClient``` directly. The default TelemetryConfiguration is a singleton that will live until the application terminates and you can create a telemetry client anywhere you want using the no argument constructor and it will use the default config instance.  You may call any of the ```Track*``` API's as desired/appropriate in your code.

```csharp
using System;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Extensibility;

namespace Microsoft.ApplicaitonInsights.Examples.NetCoreConsole
{
    public class Program
    {
        public static void Main(string[] args)
        {
            TelemetryConfiguration.Active.InstrumentationKey = 
                "11111111-2222-3333-4444-555555555555";
            TelemetryClient client = new TelemetryClient();
            client.TrackTrace("Demo application starting up.");

            for (int i = 0; i < 10; i++)
            {
                client.TrackEvent("Testing " + i);
            }

            client.TrackException(new Exception("Demo exception."));
            client.TrackTrace("Demo application exiting.");
            client.Flush();
        }
    }
}
```

The above example is a quite simplistic and hardcoded way to set the instrumentation key.  A better alternative would be to read the value from configuration.  Below is an example of reading from an environment variable:

```csharp
            // Adding all environment variables into IConfiguration.
            IConfiguration config =  new ConfigurationBuilder()
                .AddEnvironmentVariables()
                .Build();

            // Read instrumentation key from IConfiguration.
            string iKey = config["AI_KEY"];
            if (string.IsNullOrEmpty(iKey))
            {
                return -1;
            }
```
Remember to set the environment variable of **AI_KEY** as needed.

Or you can read from the appsettings.json file:
```json
{
  "ApplicationInsights": {
    "InstrumentationKey": "11111111-2222-3333-4444-555555555555"
  }
}
```
```csharp
            // Adding JSON file into IConfiguration.
            IConfiguration config =  new ConfigurationBuilder()
                .AddJsonFile("appsettings.json", true, true)
                .Build();

            // Read instrumentation key from IConfiguration.
            string ikey = config["ApplicationInsights:InstrumentationKey"];
```
