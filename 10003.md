## React Testing: Mocking Axios with axios-mock-adapter
##### *An intro to Axios mocking*
###### [@admin](/whoami)
###### Nov 30, 2020 08:50PM
###### [#react]() [#axios]() [#axios-mock-adapter]()
###### Article first published Nov, 30 2020 on [Medium](https://simonkkaranja.medium.com/react-testing-mocking-axios-with-axios-mock-adapter-e24752a55923)

![Passing Tests](/images/blog/10003/10003.png)

Unless you fancy the built-in fetch API, chances are that you are using or have used axios as your HTTP client. If so, when writing tests you may need to mock API requests. You could of course use mock functions provided by your test library in this case Jest, but I have found a nifty package called [axios-mock-adapter](https://www.npmjs.com/package/axios-mock-adapter) to be excellent and natural when testing codes with axios implementation.
Lets write a test for a simple component that fetches some random todos. A fully working implementation can be be found here. You can also see the code in action on this codesandbox.
### Assumptions
1. Familiarity with Jest
2. Familiarity with `@testing-library/react`

To follow along, clone project off github here
```sh
git clone https://github.com/karanja-simon/axios-mocking.git
// Then switch to the clean branch (no tests yet). 
git checkout clean
```
You can then run `yarn install` to install the required dependencies.

---

This is the piece of component that will be testing. I have also off-loaded the the axios fetch into a custom hook `useAxios(url)` that lives in *axios.hook.js* file.

```js
const App = () => {
  const {loading, todos, error} = useAxios("/todos");
  
  if (loading) {
    return <p>Loading...</p>;
  }
  if (error) {
    return <p>Error: {error}</p>;
  }

  return (
    <div className="todo" data-testid="todos">
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </div>
  );
}

export default App;
```
### Test Setup
For the test setup, the only dependency we need is `axios-mock-adapter`. (This is the beauty of React as most of testing tools you need will come pre-configured with create-react-app 😄).
```sh
yarn add -D axios-mock-adapter
```
Open the *App.test.js* and add the following.
```js
import React from "react";
import axiosInstance from "./axios.instance";
import MockAdapter from "axios-mock-adapter";
import { cleanup } from "@testing-library/react";

const mock = new MockAdapter(axiosInstance, { onNoMatch: "throwException" });

beforeAll(() => {
  mock.reset();
});

afterEach(cleanup);

```
Here we create an instance of `MockAdapter` and pass our axios instance, followed by `onNoMatch` option with `throwException` (This throws an exception when a request is made without matching any handler. It's very helpful when debugging your test mocks. See gotchas section below).
> Pro Tip I: Rather than pass axios directly from the axios package, off-load axios setup in a separate file and export the configured instance to be consumed anywhere you need axios. See *axios.instance.js* file
We will need to reset both registered mock handlers and history items with `mock.reset()` before running any test case and finally, we cleanup.
Add the following on the *App.test.js*
```js
import { render, waitFor, cleanup } from "@testing-library/react";
import App from "./App";

......

const todos = [
  {
    id: 1,
    title: "Buy milk",
    completed: false,
  },
  {
    id: 2,
    title: "Walk the dog",
    completed: true,
  },
];

const renderComponent = () => render(<App />);

describe("axios mocking test", () => {
  it("should render loading followed by todos", async () => {
    mock.onGet("/todos").reply(200, todos);

    const { queryByText, getByTestId } = renderComponent();

    expect(queryByText(/Loading/i)).toBeInTheDocument();
    expect(queryByText(/Walk the dog/i)).not.toBeInTheDocument();

    await waitFor(() => getByTestId("todos"));
    expect(queryByText(/Loading/i)).not.toBeInTheDocument();
    expect(queryByText(/Walk the dog/i)).toBeInTheDocument();
  });
});
```
Here, we define our dummy todos, followed by the component we wish to test (In our case `<App/>`) passed into `@testing-library/react` `render()` function.
Of importance here, is:
```js
mock.onGet("/todo").reply(200, todos)
```
This will mock the axios `get()` and return the passed data (In our case a list of our dummy todos) whenever we hit `/todos` endpoint. The `reply()` takes the following arguments:- (`status`, `data`, `headers`) in that order. If you run the tests i.e.
```sh
yarn test
```
The test should pass.

![test_1](/images/blog/10003/test_1.png)

**What's in case of a network error you ask?** Well let's add the following to the *App.test.js*
```js
it("should render loading followed by error message", async () => {
    mock.onGet("/todos").networkError();

    const { queryByText, getByText } = renderComponent();

    expect(queryByText(/Loading/i)).toBeInTheDocument();
    expect(queryByText(/Walk the dog/i)).not.toBeInTheDocument();

    await waitFor(() => getByText(/Error:/i));
    expect(queryByText(/Loading/i)).not.toBeInTheDocument();
    expect(queryByText(/Error: /i)).toBeInTheDocument();
  });
```
Of interest is line 2:
```js
mock.onGet("/todos").networkError()
```
This will return a failed promise with `Error('Network Error')`. If say we wanted a network timeout, we could have done:-
```js
mock.onGet("/todos").timeout();
```
This would have returned a failed promise with `Error` code set to `ECONNABORTED`.
Now running the tests again, they should all be in green.

![test_2](/images/blog/10003/test_2.png)

That's how easy it is to mock axios requests with `axios-mock-adapter`. Of course we have only touched one HTTP verb, `GET` in our tests above. That's not all the library is capable of. It has all HTTP methods covered in case your needs goes past `GET` or `POST`. You can leaf through their [documentation](https://www.npmjs.com/package/axios-mock-adapter) for more use cases.

### Gotchas
When mocking with `axios-mock-adapter`, you may run into a couple of issues. Lets reproduce by far the most common one. On mock.onGet() lets provide a resource that does not match any of our axios handler, say we made a typo `/todo` instead of `/todos`
```js
mock.onGet("/todo").reply(200, todos)
```
Now run the tests again. You should get something similar to this.

![gotcha](/images/blog/10003/gotcha.png)

Notice we have a helpful error message telling us the mock for `/todos` could not be found. Now, If you recall when we initialized the MockAdapter, we passed a second argument. What if we have omitted this. Lets replace
```js
const mock = new MockAdapter(axiosInstance, { onNoMatch: "throwException" });
```
with
```js
const mock = new MockAdapter(axiosInstance);
```
and run the tests again.

![gotcha_2](/images/blog/10003/gotcha_2.png)

We get a vague 404 error. This is the default behavior of the library if it cannot match any handler.
> Pro Tip II: Always turn on the `{ onNoMatch: "throwException" }` when mocking with axios-mock-adapter. It will save hours of head scratching with subtle bugs.
That's all for today. *Gracias & Happy Coding ❤️*.
