## Load Testing API's with JMeter: CMD Tools
##### *JMeter CMD Tools &amp; Interpreting the results*
###### [@admin](/whoami)
###### Dec 30, 2020 11:50AM
###### [#api]() [#jmeter]()

On the [previous](/blog/load-testing-apis-jmeter-gui) article, we saw how to setup JMeter with GUI mode. Although the GUI mode is a poweful tool for configuring and debugging your tests, it's a poor choice when it comes to load testing. On this entry, we will look at how we can use the command-line tools to stress our API, and then use the JMeter GUI mode to interprete the results.

#### Configuring JMeter CMD mode

You will need to JMeter on system path or enviroment variables on windows. On linux move the extracted JMeter directory to `/usr/local/bin/` or wherever you feel convinient. Then add it to the PATH variables:-

```sh
export PATH=$PATH:/usr/local/bin/apache-jmeter-5.4/bin/
```
The above is temporal, to persist you can add an entry to `/etc/environment` or user shell specific.

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

#### Running the Tests

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
This will take a couple of minutes depedending on the number of threads/users you specified. In my case I have 5000 threads on `/api/v1/prices` on route. After the run is complete, it will generate a *.jtl* file. We will need this file for the next section.

#### Interpreting the results

Launch the JMeter GUI 


#### References
---
1. [http://www.testingjournals.com/what-is-throughput-in-apache-jmeter/](http://www.testingjournals.com/what-is-throughput-in-apache-jmeter/)
2. [http://www.testingjournals.com/understand-aggregate-report-jmeter/](http://www.testingjournals.com/understand-aggregate-report-jmeter/)
3. [https://www.vinsguru.com/jmeter-understanding-aggregate-summary-reports-results/](https://www.vinsguru.com/jmeter-understanding-aggregate-summary-reports-results/)
 
