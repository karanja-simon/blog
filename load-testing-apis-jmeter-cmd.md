## Load Testing API's with JMeter: CMD Tools &amp; Results Interpretation
##### *JMeter CMD Tools &amp; Interpreting the results*
###### [@admin](/whoami)
###### Dec 30, 2020 11:50AM
###### [#api]() [#jmeter]()

On the [previous](/blog/load-testing-apis-jmeter-gui) article, we saw how to setup JMeter in GUI mode. Although the GUI mode is a poweful tool for configuring and debugging your tests, it's a poor choice when it comes to load testing. On this entry, we will look at how we can use the command-line tools to stress our API, and then use the JMeter GUI mode to interprete the results.

#### 1.0 Configuring JMeter CMD mode

You will need to add JMeter on your system path or enviroment variables on windows. On linux move the extracted JMeter directory to `/usr/local/bin/` or wherever you feel convinient. Then add it to the PATH variables:-

```sh
export PATH=$PATH:/usr/local/bin/apache-jmeter-5.4/bin/
```
This is temporal, to persist you can add an entry to `/etc/environment` or user shell specific entry.

```sh
nano ~/.bashrc
```
 And add the entry on the bottom
 
 ```sh
export PATH=$PATH:/usr/local/bin/apache-jmeter-5.4/bin/
```
To apply the effect on the current shell:-

```sh
source ~/.bashrc
```

#### 2.0 Running the Tests

Remember when using the JMeter GUI mode, you saved your test plan (*jmx*) somewhere in your file system? We will need to pass this file as one of the JMeter console arguments. But first, let's see how we execute the commands.

 ```sh
 jmeter -n -t path_to_.jmx -l path_to_.jtl
 ```
 
Where:-
* -n  Specifies JMeter is to run in non-gui mode
* -t  Name of JMX file that contains the Test Plan
* -l  Name of JTL/CSV/XML file to log sample results to
* -j  Name of JMeter run log file

Since we are no longer running on GUI mode, edit your *.jmx* and remove all the GUI related stuff. The easiest way is to to load your *.jmx* on the JMeter GUI &amp; delete all the listeners we had added previously. Now, let's run our tests. Launch your command terminal and point it to the directory where your test plan lives and execute the JMeter command.

```sh
jmeter -n -t fuel-api-cmd.jmx -l fuel-api-results.jtl
```
This will take a couple of minutes depedending on the number of threads/users you have on your *.jmx*. In my case I have 6000 threads on `/api/v1/prices` route. After the run is complete, it will generate a *.jtl* file. We will need this file for the next section.

#### 3.0 Interpreting the results

For this walk-through, I will use the JMeter *Aggregate Report*.
Launch the JMeter GUI, right click on the default *Test Plan > Add > Listener > Aggregate Report*. Load your *.jtl* file. This will take sometime depending on the number of threads/users you have configured. 


| ![Reaults](/images/blog/jmeter/results.png) | 
|:--:| 
| *Test run results* |


Now, how do I make sense of this data? I will try to explain the results, but have a look at the links provided on the footnotes to better understand the numbers. I will just cover the basics, with the hope that you will read more from the links provided below.

**Label**: It is the name/URL for the specific HTTP(s) Request. In our case we just have 2 correspoding to `/api/v1/login` and `/api/v1/prices`.

**Samples**: This indicates the number of virtual users/threads per request.

**Average**: It is the average time taken by all the samples to execute specific label. In our case, average time for *Prices Request* is 41164 milliseconds & total average time is 44175 milliseconds.

**Min**: The shortest time taken by a sample for specific label. Out of 6000 samples sent for *Prices Request*, the shortest response time one of the sample had was 90 milliseconds.

**Max**: The longest time taken by a sample for specific label. Out of 6000 samples sent for *Prices Request*, the longest response time one of the sample had was 110285 milliseconds.

**Error%**: Percentage of Failed requests per Label.

**Throughput**: Throughput is the number of request that are processed per time unit(seconds, minutes, hours) by the server. This time is calculated from the start of first sample to the end of the last sample. Larger throughput is better.

**KB/Sec**: This indicates the amount of data downloaded from server during the performance test execution. In short, it is the Throughput measured in Kilobytes per second.

**90% Line**: 90% of the samples took no more than this time. The remaining samples took at least as long as this. (90th percentile)

**95% Line**: 95% of the samples took no more than this time. The remaining samples took at least as long as this. (95th percentile)

**99% Line**: 99% of the samples took no more than this time. The remaining samples took at least as long as this. (99th percentile)

**Median**: It is the time in the middle of a set of samples result. It indicates that 50% of the samples took no more than this time i.e the remainder took at least as long.

Thank you for getting here. Hope you have learned something today. Cheers!

### 4.0 Footnotes
---
##### 4.1 Disclaimer
I am no authority on JMeter testing, nor is this a complete look at JMeter capabilities infact it's just one of many capabilities of the JMeter. If you need to learn more about JMeter, please follow the links provided below as they do a great deal of covering JMeter.

##### 4.2 References

1. [http://www.testingjournals.com/what-is-throughput-in-apache-jmeter/](http://www.testingjournals.com/what-is-throughput-in-apache-jmeter/)
2. [http://www.testingjournals.com/understand-aggregate-report-jmeter/](http://www.testingjournals.com/understand-aggregate-report-jmeter/)
3. [https://www.vinsguru.com/jmeter-understanding-aggregate-summary-reports-results/](https://www.vinsguru.com/jmeter-understanding-aggregate-summary-reports-results/)
 
