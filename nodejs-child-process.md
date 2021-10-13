## Node.js Worker Threads
##### *A look at Node.js parallelism with worker threads*
###### [@admin](/whoami)
###### OCT 06, 2021 02:10PM
###### [#nodejs]() [#threads]()

[Previously](/nodejs), we looked at offloading CPU-bound task to a Worker Pool by using a [Child Process](https://nodejs.org/api/child_process.html) or a [Cluster](https://nodejs.org/api/cluster.html). In this article, we will look at [Worker Threads](https://nodejs.org/api/worker_threads.html) and why they are more desirable than previous approach.

Worker threads are provided by the `worker_thread` module, introduced in Node.js version 10. These threads executes in parallel, and unlike `child_process` or `cluster`, they can share memory. They, therefore do not need the expensive IPC mechanism to communicate with their parent process. Each worker is connected to it's parent worker via a message channel. The child worker writes to the message channel using `parentPort.postMessage()` function and the parent worker writes to the message channel by invoking `worker.postMessage()` function on the worker instance. Threads lives inside a process, therefore should it die, it will also terminate the threads it holds. This is different from Cluster or Child process, where if one process dies, others can keep executing.

> Note: Workers (threads) are useful for performing CPU-intensive tasks. They are not meant for I/O-intensive work, since Node.js built-in asynchronous I/O operations are more efficient.

##### Some code..
Let's borrow the code from the [previous]() article. It's a simple Nodejs/Express server that calculates the Fibonacci of an n'th term with a linear time-complexity algorithm. As n becomes larger, so do the time to calculate the Fibonacci number.
Let's look at the recursive function:

```js
export const fib = (n) => {
    if (n < 2)
        return n;
    return fib(n - 1) + fib(n - 2);
}
```

And below is simple Nodejs/Express server.

```js
import express from 'express';
import axios from 'axios';
import { fib } from './fib.js';

const PORT = process.env.PORT || 4001;

const app = express();
app.use(express.json());

const router = express.Router();

app.use(router);

router.get('/hello', (req, res) => {
    res.status(200).json({message: 'Hello!'});  
});

router.get('/fib/:n', (req, res) => {
    let { n } = req.params;
    res.status(200).json({message: 'Hello!', fib: `fib(${n}) = ${fib(n)}`});  
});


app.listen(PORT, () => {
    console.log(`Server running @: http://localhost:${PORT}`);
});
```

Running the server and making a `GET` request for a 45'th term of the Fibonacci i.e `n = 45` on the `/fib` endpoint: 

```bash
curl http://localhost:4001/fib/45
```
Since the computation takes time, the Event Loop is held captive by our recursive task. Let's see this in action. Opening another terminal and making a curl `GET` request on the `/hello` endpoint, we get no immediate response. This means our first request is still being processed by the Event Loop/Main Thread and until it completes and release the CPU, our second request will keep waiting.
This is a classic CPU-bound operation that blocks the Event Loop and makes our server non responsive.
 

#### The Worker Threads.

The `child_process.fork()` takes 3 arguments, but the mandatory is the modulePath, which is the module to run in the child. 
First, we will create the child module which will contain our recursive function. I will call it `fib-fork.js`. Here is it's contents:


Now rewriting our server code we have:

```js
import express from 'express';
import { fibWorker } from './fib-worker.js';

const PORT = process.env.PORT || 8000;

const app = express();
app.use(express.json());

const router = express.Router();

app.use(router);

router.get('/hello', (req, res) => {
    res.status(200).json({ message: 'Hello!' });
});

router.get('/fib/:n', async (req, res) => {
    let { n } = req.params;

    const result = await fibWorker(n);

    res.status(200).json({ message: 'Hello!', fib: `fib(${n}) = ${result}` });

});


app.listen(PORT, () => {
    console.log(`Server running @: http://localhost:${PORT}`);
});
```
That's all. Runnig this, the server creates as many process as are CPU cores. Now, making a `GET` request for a 45'th term of the Fibonacci i.e `n = 45` on the `/fib` endpoint: 

```bash
http://localhost:4002/fib/45
```
And opening another tab on Postman and making another `GET` request on the `/hello` endpoint, we now immediately get a `Hello!` response as the first request keeps calculating. This means our requests are being handled by different *processes*.

As noted above, this approach is not cheap, since processes are not kind to OS resources pool. Nodejs team has since introduces a better approach called the Worker Threads from the `worker_threads` module, which allows execution in parallel, and are light-weight. We will look at this on the next article.

*Cheers. Happy coding*


