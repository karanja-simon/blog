## Load Testing API's with JMeter
##### *How perfomant is your API?*
###### [@admin](/whoami)
###### Dec 26, 2020 09:30PM
###### [#api]() [#jmeter]()

You have developed a shiny API that you are proud of, but you don't know how it will handle the real-world traffic? This is always a concern for any serious developer. For a sizeable enterprise, you may of course have a QA/Testing team that handles this. But if you are working solo or your team is small enough that you are still the QA or just for the heck of it, you may want to get some preliminary numbers from your API. There are many tools out there for loading testing API's but perhaps the most popular is [Apache's JMeter](https://jmeter.apache.org/). From their website:-
> The Apache JMeter™ application is open source software, a 100% pure Java application designed to load test functional behavior and measure performance. It was originally designed for testing Web Applications but has since expanded to other test functions.

JMeter can be used to test performance on both static &amp; dynamic resources, by simulating heavy load on the material server/resource. 

#### 1.0 Setting up JMeter

Head over to [download](https://jmeter.apache.org/download_jmeter.cgi) section of JMeter &amp; download it. You will need to have Java 8+ installed to run the latest version. Uncompress and launch the JMeter.

#### 2.0 Preparations

For this walk-through, I will be using a simple API living on the test machine. This is how the API resource structure looks:-

ACTION | RESOURCE
------ | --------
POST | /api/v1/login
GET | /api/v1/prices
GET | /api/v1/prices/annual

#### 3.0 The plan

We will start by creating a test plan. By default when you launch JMeter, it will create a test case that you can rename or leave it as is. For my case I will call it *Fuel API Test Plan*.
We will create 3 *Thread Groups* each correspoding to the actions above. The first *Thread Group* will be tasked with login &amp; setting the *auth token* globally so the other threads can use it. Here the *Number of Threads/users* will be just one since all we need is just the token. The second &amp; third *Thread Group* is where the all action will be. We will crank-up the *Number of Threads/users* to simulate a real world usage.

> Note: It's highly discouraged to use JMeter GUI mode for load testing. For the sake of this walkthrough we will use it sparingly to do some load tests. On the [next](/blog/load-testing-apis-jmeter-cmd) article, we will look at JMeter command-line tools for load stressing our API. Keep tuned.

##### 3.1 Acquiring the API access token

> If your API has no Auth flow, you can skip this section.
Let's deal with aquiring a token so we can consume our API. Our Fuel API provides a login interface on the */api/login* resource. All that is required is a valid email and a password and in return, we get ourselves an access token. On JMeter interface, right click on your test plan and *Add > Threads(Users) > Thread Group*. I will call this *Login Thread Group*. Leave everything else as default.

| ![Login Thread Group](/images/blog/jmeter/02.png) | 
|:--:| 
| *The Login Thread Group* |

We need to add a sampler, in our case a *HTTP Request* sampler to the *Thread Group* we just created. Right click on your *Thread Group* Then *Add > Sampler > HTTP Request*. I will call mine *Login Request*. Here you need to configure your server IP &amp; your API login resource.

| ![Login Request](/images/blog/jmeter/03.png) | 
|:--:| 
| *HTTP Request* |

We would want to see the server response once we hit send. We do this by adding a listener. So, right click on your *Thread Group* then *Add > Listener > View Results Tree*. If you are ready, you can hit *Run > Start* or click the green button on the toolbar.

| ![View Results](/images/blog/jmeter/05.png) | 
|:--:| 
| *View Results Tree* |

If everything was set right, you should see the response on *View Results Tree*. I have the *auth token* which I can use to make API requests.

##### 3.2 Sharing our API access token

Now that we have the token, we need a way of sharing it with the other threads that we will create shortly. First, let's extract the token from the response. We will need a *Post Processor* for this, specifically the *JSON Extractor* post processor. Right click on your *HTTP Request*, mine is *Login Request* then *Add > Post Processor > JSON Extractor*. Fill it like so:-

| ![JSON Extractor](/images/blog/jmeter/06.png) | 
|:--:| 
| *JSON Extractor* |

While at it, let's add a *Debug Sampler* to see what's happening under the hood. Right click on your *Thread Group* the *Add > Sampler > Debug Sampler*

| ![Debug](/images/blog/jmeter/07.png) | 
|:--:| 
| *Debug Sampler* |

Hit run, and check *View Result Tree* on *Debug Sampler*, you should see the extracted token.

| ![Debug Sampler](/images/blog/jmeter/08.png) | 
|:--:| 
| *API Token on Debug Sampler* |

Now, we need to set this token as a global. We will use an *Assertion* specifically, the *Bean Shell Assertion*. Right click on your *Thread Group* and *Add > Assertion > BeanShell*. Add the following:-

```sh
${__setProperty(token, ${token})};
```

| ![Bean Shell](/images/blog/jmeter/09.png) | 
|:--:| 
| *BeanShell Assertion* |

##### 3.3 Reusing the API token

Create a new *Thread Group*, Add a *HTTP Request*. This will correspond to the `/api/v1/prices` route which requires authentication. Add a *View Results Sampler*. We will need to configure our API token since we are now on a private resource. Right click on your *HTTP Request* then *Add > Config Element > HTTP Header Manager*. On the bottom of the window click add &amp; input as follows:-

| ![Header manager](/images/blog/jmeter/10.png) | 
|:--:| 
| *Header Manager* |

Of importance here is:

```sh
${__property(token)}
```
This is how we extract the token we had earlier set on the *Login Thread Group*. Running this on my setup, I get the expected data from the API:-

| ![API Data](/images/blog/jmeter/11.png) | 
|:--:| 
| *API Results* |

##### 3.4 Stressing the API

Now that everything is configured and working, let's stress load our API. Let's add another listener. Right click on the *Thread Group* then *Add > Listener > View Resilts in Table*. This will allow us to see our API metrics like Sample Time, Latency etc. Remember to add to add a .csv file path where the results will be dumped. We now add more threads count to our *Thread Group*. Click on the *Thread Group* and add the desired *Number of Threads(users)*. I will start with 50 for my case. 

| ![Results](/images/blog/jmeter/12.png) | 
|:--:| 
| *A reasonable thread count* |

Hitting run again:-

| ![Results](/images/blog/jmeter/13.png) | 
|:--:| 
| *A Sample results* |

You can keep adjusting the thread count to see how your API behaves **But read the note below**. In the next section, we will see how we can interpret the results &amp; and make informed decisions. 

> Note: If you rump-up the *Number of threads* to a huge number, you will run into memory issues, with a nasty *OutOfMemoryError* error. The JMeter GUI mode is not recommeded for running load tests. Rather it's meant for configuring and debugging tests. On the next section will see how we can avoid this by using JMeter command-line tools.

Next -> [Load Testing APIs with JMeter: CMD Tools + Interpretation](/blog/load-testing-apis-jmeter-cmd)

##### References
---
1. [https://octoperf.com/blog/2018/04/23/jmeter-rest-api-testing/](https://octoperf.com/blog/2018/04/23/jmeter-rest-api-testing/)
2. [https://www.blazemeter.com/blog/how-get-started-jmeter-installation-test-plans](https://www.blazemeter.com/blog/how-get-started-jmeter-installation-test-plans)







