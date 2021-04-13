## A developer PSR?
##### *Curated list of 1000 Kikuyu proverbs*
###### [@admin](/whoami)
###### Apr 13, 2021 02:20PM
###### [#psr]() [#reference]()
We all know of Corporate Social Responsibility (CSR), but there exists another form of social responsibility called Individual/Personal Social Responsibilty (PSR). 
It's defined as the primary responsibility of an individual toward family, workplace, community, and environment (both ecological and social). 
[See this entry from Jayashree](https://www.linkedin.com/pulse/personal-social-responsibility-psr-jayashree-venugopala). As developer, 
I see this as a call to build a solution/software that is free, easily acessible and relevant. It's with this philosophy that I decided to an 
application that could immortalize the wisdom of the Agikuyu people. Here is my effort to create a collection of 1000 Kikuyu proverbs, based on G. Barra book of the 
same title.

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


---
##### 4.1 Disclaimer
I am no authority on JMeter testing, nor is this a complete look at JMeter capabilities infact it's just one of many capabilities of the JMeter. If you need to learn more about JMeter, please follow the links provided below as they do a great deal of covering JMeter.

##### 4.2 References

1. [http://www.testingjournals.com/what-is-throughput-in-apache-jmeter/](http://www.testingjournals.com/what-is-throughput-in-apache-jmeter/)
2. [http://www.testingjournals.com/understand-aggregate-report-jmeter/](http://www.testingjournals.com/understand-aggregate-report-jmeter/)
3. [https://www.vinsguru.com/jmeter-understanding-aggregate-summary-reports-results/](https://www.vinsguru.com/jmeter-understanding-aggregate-summary-reports-results/)
 
