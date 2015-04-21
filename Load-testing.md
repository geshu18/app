# Environment

```
RAM: 12 GB
```
```
CPU: 6 Cores 3.2. GHz
```
```
OS: Windows Server 2012 R2 Standard x64
```
```
Components: Web Server/Application Server: IIS/.NET 4.0/4.5/4.6
```

# Application used for testing
Sources: https://github.com/Microsoft/ApplicationInsights-aspnetv5/tree/master/test/PerfTest/Aspnet5_EmptyApp_46.

Application was created from Aspnet v5 empty template, framework 4.6. It has 2 startup classes that can be used when you want to measure baseline without AI and measure AI noise.
Application has 1 controller with 1 method that emulates some activity for an internal provided as an action parameter.


