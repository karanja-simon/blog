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
* Service registry &amp; discovery
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

There are some basic concepts that you will need to  to fully understand how Moleculer works. Thier [documentation](https://moleculer.services/docs/0.14/) has all this covered but here are the basic concepts behind this great framework.

1. **Service**: Is a basic Javascript object that holds your part or all of your business logic.
2. **Node**: Is an OS process running on local or a remote network.
3. **Local service**: 2 or more service running on the single node.
4. **Remote services**: Services running across multiple nodes.
5. **Service Broker**: Does service management &amp; communication between services. Each Node must have an instance of a Service Broker.
6. **Transporter**: Is a communication bus used by services to exchange messages.
7. **API Gateway**: Exposes  services to outside world. It's usually a regular service running a HTTP server.

#### 2.2 Architecture: Tasker

We will biuld an API for a fictitious application called *Tasker*. *Tasker* is a simple application that let's team leads assing tasks across the team members &amp; get notified once the task/s is/are complete. We will have 4 running services:-
* *Gateway Service*:  Will process external client requests &amp; maps them to appropriate service.
* *User Service*: Will provide storage &amp; management all the available users/team members.
* *Task Service*: Will manage tasks storage, assignment, statuses etc.
* *Notification Service*: Will handle dispatch of tasks statuses via email.

**But how does all this work?**

We will have 3 nodes. The first one, `node-1` will host the *Gateway Service* and *User Service*, `node-2` will hold the *Task Service*, while `node-3` will run the *Notification Service*. Let's look at a scenario where the lead creates a task and assigns it to member X. The `POST /task` request will go through `node-1` running a HTTP server. The incoming request is forwared to the *Gateway Service* which maps it's to the *Task Service* running on `node-2`. The request is then forwared to the broker which checks if the service is running locally or remote. Since the our *Task Service* is remote. i.e it's running on another node, the request is passed to a transporter. The transporter delivers the request to `node-2` successfully, since they are connected to the same communication bus. The broker of `node-2` parses the incoming request &amp; maps it to *User Service*. The *User Service* calls the `create` action, which in turn persist the task to a local database. The `node-2` broker emits a task created/assigned event, that is recived by the *Notification Service* running on `node-3`. 



