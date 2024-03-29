### Oct 2021
---

#### [Node.js Worker Threads](/blog/nodejs-worker-threads)
##### [@admin](/whoami)
###### Oct 06, 2021 02:10PM
###### [#nodejs]() [#threads]()
[Previously](/blog/nodejs-cpu-bound-tasks), we looked at offloading CPU-bound task to a Worker Pool by using a [Child Process](https://nodejs.org/api/child_process.html) or a [Cluster](https://nodejs.org/api/cluster.html). In this article, we will look at [Worker Threads](https://nodejs.org/api/worker_threads.html) and why they are more desirable than previous approach.

Worker threads are provided by the `worker_thread` module, introduced in Node.js version 10. These threads executes in parallel, and unlike `child_process` or `cluster`, they can share memory. They, therefore do not need the expensive IPC mechanism to communicate with their parent process. Each worker is connected to it's parent worker via a message channel. The child worker writes to the message channel using `parentPort.postMessage()` function and the parent worker writes to the message channel by invoking `worker.postMessage()`. [Read more](/blog/nodejs-worker-threads)


### Sep 2021
---

#### [Node.js and CPU intensive tasks](/blog/nodejs-cpu-bound-tasks)
##### [@admin](/whoami)
###### Sep 05, 2021 04:13PM
###### [#nodejs]() [#cpu-bound]() [#threads]()
Javascript is inherently a sigle-threaded language. This makes it increadibly easy to build applications since developers don't need to think or handle the complex multi-thread environment, it's also the biggest weakness of the language per se. Performing a CPU intensive tasks will block the main thread and render your application unresponsive.

Node.js executes JavaScript code in the Event Loop (main thread) just like the browsers do, but it also offers a Worker Pool to handle expensive tasks like I/O. If the Event Loop (main thread) or a Worker thread is held by a long running task, say a CPU intensive task, then it cannot respond to requests from other clients and it's said to be blocked. This obviously leads to subsequent requests waiting for the *greedy* task to yeild, therefore leading to a bad user experience, or in a worst case; a Denial of Service. [Read more](/blog/nodejs-cpu-bound-tasks)

### Apr 2021
---

#### [A developer PSR?](/blog/psr)
##### [@admin](/whoami)
###### Apr 13, 2021 02:20PM
###### [#csr]() [#freebie]()
We all know of Corporate Social Responsibility (CSR), but there exists another form of social responsibility called Individual/Personal Social Responsibilty (PSR). It's defined as the primary responsibility of an individual toward family, workplace, community, and environment (both ecological and social). [See this entry from Jayashree](https://www.linkedin.com/pulse/personal-social-responsibility-psr-jayashree-venugopala). As developer, I see this as a call to build a solution/software that is free, easily acessible and relevant. It's with this philosophy that I decided to an application that could immortalize the wisdom of the Agikuyu people. Here is my effort to create a collection of 1000 Kikuyu proverbs [Read more](/blog/psr)

### Mar 2021
---

#### [SSL Client Authentication](/blog/ssl-client-authentication)
##### [@admin](/whoami)
###### Mar 14, 2021 12:20PM
###### [#nodejs]() [#ssl]() [#authentication]()
In traditional RESTful systems, authentication is quite straight-forward. A client registers in the platform and then uses these credentials stored on the platform to authenticate themselves. Now, what if you don't have a prior knowledge of a client, i.e the client doesn't exist on your platform? How would you know a genuine client vs a malicious one? 
[Read more](/blog/ssl-client-authentication)

### Jan 2021
---

#### [Microservices with Moleculer: Part II](/blog/moleculer-microservices-ii)
##### [@admin](/whoami)
###### Jan 09, 2021 02:50PM
###### [#nodejs]() [#moleculer]() [#microservices]()
In the [last](/blog/moleculer-microservices-i) series, we covered the basics of microservices and moleculer. Today we will build the *Tasker* application using Moleculer framework. For storage, we will use a simple sqlite database with [Moleculer Sequelize-Adapter](https://www.npmjs.com/package/moleculer-db-adapter-sequelize) ORM.
[Read more](/blog/moleculer-microservices-ii)

### Dec 2020
---

#### [Loading Testing API's with JMeter: CMD Tools](/blog/load-testing-apis-jmeter-cmd)
##### [@admin](/whoami)
###### Dec 30, 2020 11:50AM
###### [#api]() [#jmeter]()
On the [previous](/blog/load-testing-apis-jmeter-gui) article, we saw how to setup JMeter with GUI mode. Although the GUI mode is a poweful tool for configuring and debugging your tests, it's a poor choice when it comes to load testing. On this entry, we will look at how we can use the command-line tools to stress our API, and then use the JMeter GUI mode to interprete the results.
[Read more](/blog/load-testing-apis-jmeter-cmd)

#### [Loading Testing API's with JMeter: GUI](/blog/load-testing-apis-jmeter-gui)
##### [@admin](/whoami)
###### Dec 26, 2020 09:30PM
###### [#api]() [#jmeter]()
You have developed a shiny API that you are proud of, but you don't know how it will handle real life traffic? This is always a concern for any developer, but you can ease some of these worries by stress testing your API. There are many tools out there for loading testing API's but perhaps the most popular is [Apache's JMeter](https://jmeter.apache.org/)
[Read more](/blog/load-testing-apis-jmeter-gui)

#### [Microservices with Moleculer: Part I](/blog/moleculer-microservices-i)
##### [@admin](/whoami)
###### Dec 20, 2020 02:13PM
###### [#nodejs]() [#moleculer]() [#microservices]()
There are a couple of microservice frameworks available for Node.js platform. The 3 most known would be [Cote](https://cote.com), [Moleculer](https://moleculer) and [Seneca](https://seneca.com). For this part series, I will be exploring [Microservices](https://) in Node.js platform using Moleculer.
[Read more](/blog/moleculer-microservices-i)

#### [Server-less emailing](/blog/serverless-emailing)
##### [@admin](/whoami)
###### Dec 15, 2020 10:13PM
###### [#react]() [#email]() [#serverless]()
Can you really send an email from Javascript (React) without a backend server? Well, you could.. sort of. In my quest of building a [server-less]() blog, I went looking
for a mailing solution that didn't require a server setup. What I found was [emailjs](https://www.emailjs.com/).
[Read more](/blog/serverless-emailing)

#### [Node, Express Testing: Jest + Supertest](/blog/node-express-jest-testing)
##### [@admin](/whoami)
###### Dec 13, 2020 12:13PM
###### [#react]() [#express]() [#jest]() [#supertest]()
With a multitude of testing framework available for JavaScript and Node.js in specific, it can quickly become overwhelming. For the seasoned, the choices usually narrows down to [Mocha](https://mochajs.org/) paired with assertion library like [Chai](https://www.chaijs.com/). Although Mocha can also be used for testing on the browser side, I have found having a uniform test framework across the Node.js and browser (React) environment to be more rewarding, and there’s no other choice that can rival [Jest](https://jestjs.io/). The beauty of Jest is the out-of-the-box support on React and zero configuration almost elsewhere
[Read more](/blog/node-express-jest-testing)

#### [React Hooks I: An In-depth Look](/blog/react-hooks-usestate)
##### [@admin](/whoami)
###### Dec 8, 2020 12:13PM
###### [#react]() [#hooks]()
React hooks is a god's sent feature that was introduced in React 16.8. In old days, if your component did not handle any local state, then you would normally write it as a stateless functional component. Simple and elegant. Now the problem came if you decided to introduce state to the said component. You would need to refactor your functional component to a class based component to enjoy the state & the lifecycles. Now with the Hook API, you can easily use state, introduce side effects and so much more. React provides a handful of hooks, but the most widely used are:-
[Read more](/blog/react-hooks-usestate)

### Nov 2020
---

#### [React Testing: Mocking axios with axios-mock-adapter](/blog/axios-mocking)
##### [@admin](/whoami)
###### Nov 30, 2020 11:13PM
###### [#react]() [#axios]() [#axios-mock-adapter]()
Unless you fancy the built-in `fetch` API, chances are that you are using or have used axios as your HTTP client. If so, when writing tests you may need to mock API requests. You could of course use mock functions provided by your test library in this case Jest, but I have found a nifty package called `axios-mock-adapter` to be excellent and natural when testing codes with axios implementation.
[Read more](/blog/axios-mocking)

### Oct 2020
---
## [Lo-fi Beats compilation](/blog/lo-fi)
###### *Some beats to chill to as you code.*
###### [@admin](/whoami)
###### Oct 10, 2020 9:50PM
###### [#music]() [#lofi]()
So, some days back I went looking for the best Low Fidelify sounds aka *Lo-fi* beats. (I am lover of lofi, if can't tell already and do enjoy them especially 
when coding). Fist, I wanted to know more about their origin and was suprised that they go way back to 50's. I will not bother you will all the history, but 
here is a nice [entry in Wikipedia](https://en.wikipedia.org/wiki/Lo-fi_music) you can read. I didn't know Lo-fi is a *music quality* in which
[Read more](/blog/lo-fi)

### Very Old
---
## [JavaFX Multi-screen Manager](/blog/javafx-multiscreen-framework)
###### [@admin](/whoami)
###### Oct 10, 2016 9:50PM
###### [#java]() [#javafx]()
Long, long time ago while working with JavaFX, I decided to create a small framework/library to managing multiple screens or views (.fxml files). Here is my effort, 
and although
[Read more](/blog/javafx-multiscreen-framework)
