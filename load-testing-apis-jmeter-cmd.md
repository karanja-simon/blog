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

#### Running Tests

Remember when using the JMeter GUI mode, you saved your test plan (*jmx*) somewhere in your file system? We will need to pass this file as one of the JMeter console arguments.
 
