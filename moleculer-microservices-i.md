## Moleculer Microservices I
##### *An intro to microservices on Node.js*
###### [@admin](/whoami)
###### Dec 19, 2020 02:50PM
###### [#node.js]() [#moleculer]() [#microservices]()

There are a couple of microservice frameworks available for Node.js platform. The 3 most known would be [Cote](https://cote.com), [Moleculer](https://moleculer) and [Seneca](https://seneca.com). For this part series, I will be exploring [Microservices](https://) in Node.js platform using Moleculer.

#### Basic Concepts

Microservices is a way of buidling (architecture) software as a collection of small services, that are loosely coupled, running on thier own processes and communicating via a lightweight mechanisms like HTTP, message broker etc. This is in contrast to the traditional systems that are built as a single unit, often called a monolith.
Alot has been writen about microservices by scholars &amp; pioneers/early adopters like [Amazon](https://aws.amazon.com/microservices/#:~:text=Microservices%20are%20an%20architectural%20and,small%2C%20self%2Dcontained%20teams.) &amp; [Netflix](https://netflixtechblog.com/tagged/microservices) (See footnotes for more sources), so I will not regugitate this informtion. However, there are some universal concepts that any microservice
regardless of the language implementation, must fulfill. These are, but not limited to:-
* Service registry &amp; discover
* Networking
* Load balancing
* Fault tolerance

We will look at this &amp; and see how they are implemented on the Moleculer framework, but first let's see what we will be building.

#### The Strategy

Am no expert on microservices, but the idea here is to walk with me as I explore microservices on Node.js platform. We will biuld a fictitious **Takser**
