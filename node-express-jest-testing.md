## Node, Express Testing: Jest + Supertest
##### *Testing Node.js, Express with Supertest & Jest testing framework.*
###### [@admin](/whoami)
###### Dec 13, 2020 6:50PM
###### [#react]() [#node.js]() [#express]() [#jest]() [#supertest]()
###### Article first published Dec, 8 2020 on [Medium](https://simonkkaranja.medium.com/node-testing-jest-supertest-343c9b650d89)

![Passing Tests](/images/blog/10001/10001.png)

With a multitude of testing framework available for JavaScript and Node.js in specific, it can quickly become overwhelming. For the seasoned, the choices usually narrows down to [Mocha](https://mochajs.org/) paired with assertion library like [Chai](https://www.chaijs.com/). Although Mocha can also be used for testing on the browser side, I have found having a uniform test framework across the Node.js and browser (React) environment to be more rewarding, and there’s no other choice that can rival [Jest](https://jestjs.io/). The beauty of Jest is the out-of-the-box support on React and zero configuration almost elsewhere.
Now, first things first. Let’s create a minimal RESTful API that returns a cliché of todos. I will be using TypeScript and for simplicity, the todos will be stored on a plain file rather than a database. We will also use supertest for HTTP assertion. To follow along, clone the demo repo & switch to clean branch:-
```sh 
git clone https://github.com/karanja-simon/node-jest-test.git
git checkout clean
```
Should you get lost anywhere as we go along, you can quick switch to working branch (git checkout master) for the fully working demo.
Install tests dependencies
We will need to install jest, ts-jest and supertest:-
```sh
yarn add -D jest ts-jest supertest
```
Perhaps worth mentioning, `ts-jest` is a TypeScript preprocessor for jest that allows it to transpile TypeScript on the fly. In short, it allows you to test codebase written in TypeScript.
Since we are using typescript, let quickly add types:-
```bash
yarn add -D @types/jest @types/supertest
```
We also need to create a configuration that Jest is going to use when running our tests:-
```sh
jest --init
```
Based on your project, Jest will ask you a few questions and will create a basic configuration file with a short description for each option. 
Will also need to tell jest to use `ts-jest`. Finally, your config should look some thing like this:-
```js
export default {
  clearMocks: true,
  coverageDirectory: "coverage",
  rootDir: ".",
  roots: ["<rootDir>/src"],

  testEnvironment: "node",
  testMatch: ["**/__tests__/**/*.[jt]s?(x)", "**/?(*.)+(spec|test).[tj]s?(x)"],

  transform: {
    "^.+\\.(ts|tsx)$": "ts-jest",
  },
};
```

### The tests
Of importance here is the `todo.controller.ts` that lives in *src/api/controllers* and our `todo.service.ts` in *src/api/services*. Since our controller is 
dependent on the service, lets first deal with the latter. Let’s see how our todo service looks like:-
```js
import { ITodo, todos } from ".";

export const allTodos = (): Promise<ITodo[]> => {
  return new Promise((resolve, reject) => {
    resolve(todos);
  });
};

export const todoById = (id: number): Promise<ITodo> => {
  return new Promise((resolve, reject) => {
    resolve(todos.filter((todo) => todo.id === id)[0]);
  });
};

export const todoByStatus = (status: string): Promise<ITodo[]> => {
  const todoStatus = status === "done" ? true : false;
  return new Promise((resolve, reject) => {
    resolve(todos.filter((todo) => todo.completed === todoStatus));
  });
};
```

This is a simple as any todo can get. With the spirit of unit testing, it’s clear that we need to test the three functions: `allTodos()`, `todoById(id: number)` and `todoByStatus(status: string)`.
Let’s fire our favorite editor and open the `todo.service.test.ts` in *src/api/services/__tests__* and add the following:-
```js
import { ITodo, allTodos } from "../";
import { todoById, todoByStatus } from "../";

const dummyTodo: ITodo = {
  id: 1,
  title: "Walk the dog",
  completed: true,
};

describe("Todo Service", () => {
  it("should return all todos", async () => {
    const todos = await allTodos();
    expect(todos.length).toEqual(3);
    expect(todos[0]).toMatchObject(dummyTodo);
  });

  it("should return a todo by id -> [id: 1]", async () => {
    const todo = await todoById(1);
    expect(todo).toBeDefined();
    expect(todo.id).toEqual(1);
    expect(todo).toMatchObject(dummyTodo);
  });

  it("should return todo(s) by status -> [status: done]", async () => {
    const todos = await todoByStatus("done");
    expect(todos).toBeDefined();
    expect(todos.length).toBeGreaterThan(0);
    expect(todos[0].completed).toBeTruthy();
  });
  
    it("should return todo(s) by status -> [status: pending]", async () => {
    const todos = await todoByStatus("pending");
    expect(todos).toBeDefined();
    expect(todos.length).toBeGreaterThan(0);
    expect(todos[0].completed).toBeFalsy();
  });
});
```

As with any Jest test, we wrap our test cases in `describe`, then implement our first test case, line 11–15. Since our `allTodos()` returns a promise, we pass `async` keyword in front of the function we pass to `it`. This tells jest to wait for the completion of the asynchronous code before executing the next test. We get the todo on line 18, then do our assertions with expect. If we run our tests:-
```sh
yarn test
```
We should see a healthy test run. You can also pass coverage flag to see the test coverage.

![Passing Tests](/images/blog/10001/test_1.png)

Notice we have 100% coverage, which is always a good sign.
Now, to the second file we need to test. Open `todo.controller.ts` inside *src/api/controllers*. Let’s have a look:-
```js
import request from "supertest";

import app from "../../../app";
import { ITodo } from "../../services";

const dummyTodo: ITodo = {
  id: 1,
  title: "Walk the dog",
  completed: true,
};

describe("Todo Controller", () => {
  it("should get all todos", async () => {
    const res = await request(app).get("/api/todos");
    expect(res.status).toEqual(200);
    expect(res.body).toBeDefined();
    expect(res.body.success).toBeTruthy();
    expect(res.body.todos).toBeDefined();
    expect(res.body.todos.length).toEqual(3);
  });

  it("should get a todo by id -> [id: 1]", async () => {
    const res = await request(app).get("/api/todo/1");
    expect(res.status).toEqual(200);
    expect(res.body).toBeDefined();
    expect(res.body.success).toBeTruthy();
    expect(res.body.todo).toBeDefined();
    expect(res.body.todo).toMatchObject(dummyTodo);
  });

  it("should get todos by status -> [status: 'done']", async () => {
    const res = await request(app).get("/api/todos/done");
    expect(res.status).toEqual(200);
    expect(res.body).toBeDefined();
    expect(res.body.success).toBeTruthy();
    expect(res.body.todos).toBeDefined();
    expect(res.body.todos.length).toBeGreaterThan(0);
  });

  it("should get todos by status -> [status: 'pending']", async () => {
    const res = await request(app).get("/api/todos/pending");
    expect(res.status).toEqual(200);
    expect(res.body).toBeDefined();
    expect(res.body.success).toBeTruthy();
    expect(res.body.todos).toBeDefined();
    expect(res.body.todos.length).toBeGreaterThan(0);
  });

  it("should return a 404 with an error message if id in not numeric [id: abc]", async () => {
    const res = await request(app).get("/api/todo/abc");
    expect(res.status).toEqual(400);
    expect(res.body).toBeDefined();
    expect(res.body.success).toBeFalsy();
    expect(res.body).toMatchObject({
      success: false,
      errors: [
        {
          value: expect.any(String),
          msg: "Todo id should be numeric",
          param: "id",
          location: "params",
        },
      ],
    });
  });

  it("should return a 404 with an error message if status is neither 'done' or 'pending' [status: waiting]", async () => {
    const res = await request(app).get("/api/todos/waiting");
    expect(res.status).toEqual(400);
    expect(res.body).toBeDefined();
    expect(res.body.success).toBeFalsy();
    expect(res.body).toMatchObject({
      success: false,
      errors: [
        {
          value: expect.any(String),
          msg: "Status should be either 'done' or 'pending'",
          param: "status",
          location: "params",
        },
      ],
    });
  });
});
```
The code is straight forward. We have three controller functions that corresponds to our service functions. I have also used `express-validator` for validating our API requests. As usual, open the `todo.controller.test.ts` inside *src/api/controllers/__tests__* and paste the following:-

This follows the same recipe as the tests we have covered above, but here we need a way to test our HTTP code. We need to simulate users hitting our `GET` end points. This is where [supertest](https://www.npmjs.com/package/supertest) comes to play. We import request from supertest on line 1 and create our first GET request on line 14.
> Notice the `request` function takes in our express app as the argument. It’s therefore a good practice to define your express as ‘exportable’ module. 
> See `app.ts`

We can then assert the response from request. Running this, again we get a nice green pass with 100% coverage.

![tests_2](/images/blog/10001/test_2.png)


### Gotchas
As you setup your tests and do your first run, you may get an error similar to this:-
![tests_2](/images/blog/10001/gotcha.png)
This happens if you have not configured your `transform` property in jest configuration properly. Remember as I mentioned above, jest requires the .ts files to be transpiled to .js. So, ensure or check that `transform` on `jest.config.ts` is configured as:-
```js
........
transform: {"^.+\\.(ts|tsx)$": "ts-jest"},
........
```
As you may have noticed, I have just covered R (Read = GET) of the CRUD (Create, Read, Update & Delete) possibilities of our API. The rest will follow the same approach, and is left as an exercise for the reader.

Keep Coding. ❤️. *Shukran!*
