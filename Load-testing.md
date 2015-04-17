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
Components: Web Server/Application Server: IIS/.NET 4.0/4.5
```
# AI SDK version

Used prerelease version complied in release from master 4/17/2015

# Application used for testing
Sources: https://github.com/Microsoft/ApplicationInsights-aspnetv5/tree/master/test/PerfTest/Aspnet5_EmptyApp_46.

Application was created from Aspnet v5 empty template, framework 4.6. It has 2 startup classes that can be used when you want to measure baseline without AI and measure AI noise.
Application has 1 controller with 1 method that emulates some activity for an internal provided as an action parameter.

# Load test results:
Activity internal was 50 milliseconds.
Test run is 10 minutes with 1 minute warm up interval.

**Baseline**
			
Box CPU|w3wp CPU|RPS|RPS/CPU
--------|--------|--------|--------
81.9 | 490 | 78.7 | 0.160612245
84.7 | 488 | 78.2 | 0.160245902
82.4 | 494 | 79.1 | 0.160121457
			
**Performance Run**			

Box CPU|w3wp CPU|RPS|RPS/CPU
--------|--------|--------|--------
83.1|496|79.5|0.160282258
83|497|79.6|0.160160966
83.8|497|79.6|0.160160966

**Noise:	0.078052036**
