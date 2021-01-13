## Moleculer Microservices II
##### *An intro to microservices on Node.js*
###### [@admin](/whoami)
###### Jan 9, 2021 02:50PM
###### [#node.js]() [#moleculer]() [#microservices]()

In the [last](/blog/moleculer-microservices-i) series, we covered the basics of microservices and moleculer. Today we will build the *Tasker* application using Moleculer framework. For storage, we will use a simple sqlite database with [Moleculer Sequelize-Adapter](https://www.npmjs.com/package/moleculer-db-adapter-sequelize) ORM. 
*If you are new here, you check the [previous](/blog/moleculer-microservices-i) introductory article, where we covered Moleculer basics &amp; the architecture of the *Tasker* API that we will build on this entry.*

#### Environment

Initialize an empty project &amp; install Molecular via `npm` or `yarn`

```sh
yarn add moleculer moleculer-web
```

We will also require Moleculer REPL. This will allow us to interact with running services, benchmark &amp; importantly, it will allow us to start services with `moleculer-runner`. Install it:

```sh
yarn add moleculer-repl
```
Since will be running remote services, we will use `nats` as our `transporter`. Download [nats server](https://nats.io/) &amp; start it. We will need to install `nats` npm module:

```sh
yarn add nats
```
We will need to create a moleculer config for our broker. Remember from our last article that every node must have a broker. We could of course create the broker manually, but since we're using the `moleculer-runner` we will use the config option. I will have 2 broker configs, one for dev &amp; the other for production. On the root of your project create a `moleculer.dev.config.js` &amp; add the following:

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

  metrics: true,
};
```
You can create another config for production i.e. `moluculer.config.js` with the relevant features turned on/off. You can check all supported options [here](https://moleculer.services/docs/0.14/configuration.html).
We need to tell `moleculer-runner` where to find and how to run our services. On our `package.json` add the following:

```json
  "scripts": {
    "dev": "moleculer-runner --repl --hot --config moleculer.dev.config.js ./src/services",
    "start": "moleculer-runner --instances=1 ./src/services"
  }
```


**Note: As per my project structure, my servcices lives inside */src/services* directory.**

#### The Code

Create a *src* directory. In it, create a *services* directory. This is where `moleculer-runner` will pick our services. Inside the *services* create *api.service.js* file &amp; add the following:

```js
const APIService = require("moleculer-web");

const TaskerAPIGatewayService = {
  name: "tasker-api",
  mixins: [APIService],

  settings: {
    routes: [
      {
        path: "/api",
        aliases: {
          "REST /task": "task",
        },
      },
      {
        path: "/api",
        aliases: {
          "REST /users": "users",
        },
      },
    ],
  },

  actions: {},
};

module.exports = TaskerAPIGatewayService;
```

This is our gateway service. Here, we load `APIService` from `molculer-web`, then create our gateway service. I have `REST` aliases to generate the correspoding `REST` actions. E.g, for `REST /users` this will generate:-

```sh
   GET /api/users => users.list
   GET /api/users/:id => users.get
   POST /api/users => users.create
   PUT /api/users/:id => users.update
   PATCH /api/users/:id => users.patch
   DELETE /api/users/:id => users.remove
```

We will then need to implement these actions on our users service. (We will do this shortly. Bear with me). Now lets fire our application:

```sh
yarn run dev
```
