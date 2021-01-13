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
yarn add moleculer moleculer-web
```

We will also require Moleculer REPL. This will allow us to interact with running services, benchmark &amp; importantly, it will allow us start services with `moleculer-runner`. Install it:

```sh
yarn add moleculer-repl
```
Since will be running remote services, we will use `nats` as our `transporter`. Download [nats server](https://nats.io/) &amp; start it. We will need to install `nats` npm module:

```sh
yarn add nats
```
We will need to create a moleculer config that `moleculer-runner` is going to use to run our services. On the root of your project create a `moleculer.config.js` &amp; add the following:

```js
module.exports = {
  nodeID: "tasker",
  logger: true,
  logLevel: "debug",

  transporter: "nats://localhost:4222",
  requestTimeout: 5 * 1000,

  circuitBreaker: {
    enabled: true,
  },

  metrics: false,
};
```

We need to tell `moleculer-runner` how &amp; where to find our services. On our `package.json` add the following:

```json
  "scripts": {
    "dev": "moleculer-runner --repl --hot --config moleculer.dev.config.js ./src/services",
    "start": "moleculer-runner --instances=1 ./src/services"
  }
```

> Note
