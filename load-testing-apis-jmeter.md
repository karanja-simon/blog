## Load Testing API's with JMeter
##### *How perfomant is your API?*
###### [@admin](/whoami)
###### Dec 26, 2020 09:30PM
###### [#api]() [#jmeter]()

You have developed a shiny API that you are proud of, but you don't know how it will handle the real life traffic? This is always a concern for any developer, but you can ease some of these worries by stress testing your API. There are many tools out there for loading testing API's but perhaps the most popular is [Apache's JMeter](https://jmeter.apache.org/). From their website:-
> The Apache JMeter™ application is open source software, a 100% pure Java application designed to load test functional behavior and measure performance. It was originally designed for testing Web Applications but has since expanded to other test functions.
JMeter can be used to test performance on both static &amp; dynamic resources, by simulating heavy load on the material server/resource.

###### Ok. Let's do this

Head over to [download](https://jmeter.apache.org/download_jmeter.cgi) section of JMeter &amp; download it. You will need to have Java 8+ installed to run the latest version. Uncompress and launch the JMeter.
---
For this walk-through, I will be using a simple API living on the test machine. This is how the API resource structure looks:-
ACTION | RESOURCE
------ | --------
POST | /api/login
GET | /api/prices
GET | /api/prices/annual

We will start by creating a test plan. By default when you launch the JMeter, it will create a test case that you can rename or leave it as is. For my case I will call it *Fuel API Test Plan*.

##### The plan

We will create 3 *Thread Groups* each correspoding to the actions above. The first *Thread Group* will be tasked with login &amp; setting the *auth token* globally so the other threads can use it. Here the *Number of Threads/users* will be just one since all we need is just the token. The second &amp; third *Thread Group* is where the all action will be. We will crank-up the *Number of Threads/users* to simulate a real world usage.

##### Step 1: Acquiring API token

> If your API has no Auth flow, you can skip this section.
Let's deal with aquiring a token so we can consume our API. Our Fuel API provides a login interface on the */api/login* route. All it requires is a valid email and a password and in return, provides us with an access token. On JMeter interface, right click on your test plan and add a *Thread Group*. I will call this *Login Thread Group*. Leave everything as default.
![Login Thread Group](/images/blog/jmeter/02.png)




