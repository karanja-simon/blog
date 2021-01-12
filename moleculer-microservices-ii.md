## Moleculer Microservices II
##### *An intro to microservices on Node.js*
###### [@admin](/whoami)
###### Jan 9, 2021 02:50PM
###### [#node.js]() [#moleculer]() [#microservices]()

In the [last](/blog/moleculer-microservices-i) series, we covered the basics of microservices and moleculer. Today we will build the *Tasker* application using Moleculer framework. For storage, we will use a simple sqlite database with [Moleculer Sequelize-Adapter](https://www.npmjs.com/package/moleculer-db-adapter-sequelize) ORM. 
> If you are new here, you check the [previous](/blog/moleculer-microservices-i) introductory article, where we covered Moleculer basics &amp; the architecture of the *Tasker* API that we will build on this entry.

#### Environment

Initialize an empty project &amp; install Molecular via `npm` or `yarn`

```sh
npm i moleculer --save
```

We will also require Moleculer CLI. Install it globally:

```sh
npm i moleculer-cli -g
```
