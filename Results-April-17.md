# AI SDK version

Used prerelease version complied in release from master 4/17/2015

# Load test results:
## Experiment 1 
One web test is used that calls the page that emulates some activity for 50 milliseconds.
([LoadTest](https://github.com/Microsoft/ApplicationInsights-aspnet5/blob/master/test/PerfTest/PerfTest/DoRequestLoad.loadtest))
50 ms is chosen as it is an average of all times from all customers we have right now.
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

## Experiment 2 
Same test but controller emulates activity for 10 ms 
10 ms is chosen because 50 percentile of all times from all customers is less than 10ms right now.
Test run is 10 minutes with 2 minute warm up interval.

**Baseline**
			
Box CPU|w3wp CPU|RPS|RPS/CPU
--------|--------|--------|--------
79.3 | 475 | 304 | 0.64
79.9 | 478 | 306 | 0.640167364
81.9 | 475 | 305 | 0.642105263

			
**Performance Run**			

Box CPU|w3wp CPU|RPS|RPS/CPU
--------|--------|--------|--------
81.7 | 477 | 306 | 0.641509434
79.9 | 478 | 306 | 0.640167364
80.2 | 480 | 307 | 0.639583333


**Noise:	0.052671814**


## Experiment 3 
Test noise when application throws exceptions on 30% requests. 
30% unhandled exceptions is very high for a real application. The goal is to check that if an application does not perform well AI does not completely kill it.
Test run is 10 minutes with 2 minute warm up interval.

**Baseline**
			
Box CPU|w3wp CPU|RPS|RPS/CPU
--------|--------|--------|--------
83.4 | 499 | 114 | 0.228456914
83.5 | 499 | 114 | 0.228456914
83.5 | 500 | 114 | 0.228

			
**Performance Run**			

Box CPU|w3wp CPU|RPS|RPS/CPU
--------|--------|--------|--------
85.3 | 499 | 114 | 0.228456914
85.2 | 499 | 114 | 0.228456914
83.5 | 500 | 113 | 0.226

**Noise:	0.292007537**

## Experiment 4 
Test noise when application throws exceptions on 5% requests. 

**Baseline**
			
Box CPU|w3wp CPU|RPS|RPS/CPU
--------|--------|--------|--------
85.1 | 499 | 84.1 | 0.168537074
85.1 | 499 | 84 | 0.168336673
84.9 | 499 | 84.2 | 0.168737475

			
**Performance Run**			

Box CPU|w3wp CPU|RPS|RPS/CPU
--------|--------|--------|--------
83.1 | 498 | 83.9 | 0.168473896
84.1 | 499 | 84.1 | 0.168537074
85.3 | 499 | 84 | 0.168336673

**Noise:	0.052130838**


## Experiment 5
One web test is used that calls the page that emulates some activity for 50 milliseconds.
([LoadTest](https://github.com/Microsoft/ApplicationInsights-aspnet5/blob/master/test/PerfTest/PerfTest/DoRequestLoad.loadtest))

Test run is 72 hours.

**Results**

No memory leaks or crashes were detected. No changes in RPS pattern or CPU spikes were observed. 


