## Moleculer Microservices I
##### *An intro to microservices on Node.js*
###### [@admin](/whoami)
###### Dec 19, 2020 02:50PM
###### [#node.js]() [#moleculer]() [#microservices]()

There are a couple of microservice frameworks available for Node.js platform. The 3 most known would be [Cote](https://cote.com), [Moleculer](https://moleculer) and [Seneca](https://seneca.com). For this part series, I will be exploring [Microservices](https://) in Node.js platform using Moleculer.

#### 1.0 Basic Concepts

Microservices is a way of buidling (architecture) software as a collection of small services, that are loosely coupled, running on thier own processes and communicating via a lightweight mechanisms like HTTP, message broker etc. This is in contrast to the traditional systems that are built as a single unit, often called a monolith.
Alot has been writen about microservices by scholars &amp; pioneers/early adopters like [Amazon](https://aws.amazon.com/microservices/#:~:text=Microservices%20are%20an%20architectural%20and,small%2C%20self%2Dcontained%20teams.) &amp; [Netflix](https://netflixtechblog.com/tagged/microservices) (See footnotes for more sources), so I will not regugitate this informtion. However, there are some universal concepts that any microservice
regardless of the language implementation, must fulfill. These are, but not limited to:-
* Service registry &amp; discover
* Networking
* Load balancing
* Fault tolerance

We will look at this &amp; and see how they are implemented on the [Moleculer](https://moleculer.services/) framework, but first let's see what we will be building.

#### 2.0 Moleculer

Moleculer is a fast & powerful microservices framework with support &amp; excellent documentation. It offers alot of capabilities out of the box &amp; it's also easy to extend functionality via custom plugins. Some of the features offered are:-

* Built-in service registry & dynamic service discovery
* Load balancing
* Fault tolerance features (Circuit Breaker, Bulkhead, Retry, Timeout, Fallback)
* Built-in caching solution (Memory, MemoryLRU, Redis)
* Pluggable loggers (Console, File, Pino, Bunyan, Winston, Debug, Datadog, Log4js)

You can view all available features [here](https://moleculer.services/docs/0.14/). As I outlined above, it's clear to see that Moleculer has many if not all features that any microservice architecture requires.

#### 2.1 Some definations

There are some concepts that you will require to understand how Moleculer works. Thier [documentation](https://moleculer.services/docs/0.14/) has all covered but here are the basic concepts behind this great framework.

1. **Service**: Is a basic Javascript object that holds your part or all of your business logic.
2. **Node**: Is an OS process running on local or a remote network.
3. **Local service**: 2 or more service running on the single node.
4. **Remote services**: Services running across multiple nodes.
5. **Service Broker**: Does service management &amp; communication between services. Each Node must have an instance of a Service Broker.
6. **Transporter**: Is a communication bus used by services to exchange messages.
7. **API Gateway**: Exposes  services to outside world. It's usually a regular service running a HTTP server.

