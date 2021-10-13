## Node.js Worker Threads
##### *A look at Node.js parallelism with worker threads*
###### [@admin](/whoami)
###### OCT 06, 2021 02:10PM
###### [#nodejs]() [#threads]()

[Previously](/blog/nodejs-cpu-bound-tasks), we looked at offloading CPU-bound task to a Worker Pool by using a [Child Process](https://nodejs.org/api/child_process.html) or a [Cluster](https://nodejs.org/api/cluster.html). In this article, we will look at [Worker Threads](https://nodejs.org/api/worker_threads.html) and why they are more desirable than previous approach.

Worker threads are provided by the `worker_thread` module, introduced in Node.js version 10. These threads executes in parallel, and unlike `child_process` or `cluster`, they can share memory. They, therefore do not need the expensive IPC mechanism to communicate with their parent process. Each worker is connected to it's parent worker via a message channel. The child worker writes to the message channel using `parentPort.postMessage()` function and the parent worker writes to the message channel by invoking `worker.postMessage()` function on the worker instance. These threads have their own Event Loop and V8 instance and runs indepedently. Threads lives inside a process, therefore should it die, it will also terminate the threads it holds. This is different from Cluster or Child process, where if one process dies, others can keep executing.

> Note: Workers (threads) are useful for performing CPU-intensive tasks. They are not meant for I/O-intensive work, since Node.js built-in asynchronous I/O operations are more efficient.

##### Some code..
Let's borrow the code from the [previous](/blog/nodejs-cpu-bound-tasks) article. It's a simple Nodejs/Express server that calculates the Fibonacci of an n'th term with a linear time-complexity algorithm. As n becomes larger, so do the time to calculate the Fibonacci number.
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

We create a worker by calling the `Worker` class and passing a mandatory script or module we wish to run, followed by options, which could be `argv` arguments or the `workerData`. See the ref [here](https://nodejs.org/api/worker_threads.html#worker_threads_new_worker_filename_options) for complete list of available options. 

First, we will create the worker module which will contain our recursive function. I will call it `fib-worker.js`. Here is it's contents:

```js
import { Worker, isMainThread, workerData, parentPort } from "worker_threads";
import { fileURLToPath } from "url";

const __filename = fileURLToPath(import.meta.url);

const fib = (n) => {
    if (n < 2)
      return n;
    return fib(n - 1) + fib(n - 2);
}

export let fibWorker;

if (isMainThread) {
    console.log('On Main thread..');
    fibWorker =  (n) => {
        return new Promise((resolve, reject) => {
            const worker = new Worker(__filename, {
                workerData: n
            });
            worker.on('message', resolve);
            worker.on('error', reject);
            worker.on('exit', code => {
                if (code !== 0) {
                    reject(new Error(`Worker stopped with exit code ${code}`));
                }
            });
        });
    }
} else {
    console.log('On Worker thread..');
    const result = fib(workerData);
    parentPort.postMessage(result);
    parentPort.on('message', msg => {
        console.log('Msg: ', msg);
    });
    process.exit(0);
}

```

Now rewriting our server code we have:

```js
import express from 'express';
import { fibWorker } from './fib-worker.js';

const PORT = process.env.PORT || 4001;

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
Now, making a `GET` request for a 45'th term of the Fibonacci i.e `n = 45` on the `/fib` endpoint: 

```bash
curl http://localhost:4001/fib/45
```
And opening another terminal and making another curl `GET` request on the `/hello` endpoint, we now immediately get a `Hello!` response as the first request keeps calculating. This means our Event Loop is no longer blocking. Our CPU-intesive calculations has been offloaded to a Worker thread. If you do another request to the `/fib/n` endpoint, another thread will be spwaned to handle this request and so on.

As light as these threads are compared to processes, they are still expensive in terms of the system resources and should not be spwaned frequently. A better approach is using Worker thread pooling that can reduce the overhead of creating new threads. We will look at this in a future article. 

#### References

* https://blog.insiderattack.net/deep-dive-into-worker-threads-in-node-js-e75e10546b11
* https://nodejs.org/api/worker_threads.html

*Cheers. Happy coding*


