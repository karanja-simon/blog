## Nodejs and CPU intensive tasks?
##### *A look at Nodejs CPU-bound task handling*
###### [@admin](/whoami)
###### Nov 05, 2021 04:13PM
###### [#nodejs]() [#non-blocking]() [#threads]()

Nodejs executes JavaScript code in the Event Loop (main thread) just like the browsers do, but it also offers a Worker Pool to handle expensive tasks like I/O. If the Event Loop (main thread) or a Worker thread is held by a long running task, say a CPU intensive task, then it cannot respond to requests from other clients and it's said to be blocked. This obviously leads to subsequent requests waiting for the *greedy* task to yeild, therefore leading to a bad user experience, or in a worst case; a Denial of Service.

It's up to the developer to offload these CPU-bound task to a Worker Pool by using a [Child Process](https://nodejs.org/api/child_process.html) or a [Cluster](https://nodejs.org/api/cluster.html). These will create new processes with their own memory, Event Loop & V8 instance. This is an expensive operation in terms OS resources and this is why the [Worker Threads](https://nodejs.org/api/worker_threads.html) were born. (We will look at Worker Threads in another article)

##### How do we block the Event Loop?
Let's build a simple Nodejs/Express server that calculates the Fibonacci of an n'th term. I will implement a linear time-complexity algorithm for calculating Fibonacci. As n becomes larger, so do the time to calculate the Fibonacci number.
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

I will use Postman to interact with this simple API. Running the server and making a `GET` request for a 45'th term of the Fibonacci i.e `n = 45` on the `/fib` endpoint: 

```bash
http://localhost:4001/fib/45
```
Since the computation takes time, the Event Loop is held captive by our recursive task. Opening another tab on Postman and making another `GET` request on the `/hello` endpoint, we get a waiting indicator. This means our first request is still being processed by the Event Loop/Main Thread and until it completes and release the CPU, our second request will keep waiting.
This is a classic CPU-bound operation that blocks the Event Loop and makes our server non responsive. Let's look at the competition.

##### How would a Multi-threaded system cope?
I will implement the same algorithm using Java Spring Boot with Apache Tomcat and see how it copes with the same. Below is the same recursive method in Java:

```java
public int fib(int n){
    if (n <= 1)
         return n;
    return fib(n-1) + fib(n-2);
}
```

And below is a simple Rest Controller to handle our request:

```java
@RestController
public class FibonacciController {

    @Autowired
    private Fib fibService;

    @GetMapping("/hello")
    public ResponseEntity<String> getHello() {
        return ResponseEntity.ok("Hello!");
    }
    
    @GetMapping("/fib/{n}")
    public ResponseEntity<Result> getFib(@PathVariable int n) {
        int fib = fibService.fib(n);
        return ResponseEntity.ok(new Result(n, fib));
    }
}
```

Again, running this server and making a `GET` request for a 45'th term of the Fibonacci i.e `n = 45` on the `/fib` endpoint: 

```bash
http://localhost:4002/fib/45
```
Opening another tab on Postman and making another `GET` request on the `/hello` endpoint, we immediately get a `Hello!` response. This means our first request is handled by a different thread, meaning for every request, a new thread is spun to handle it. This is so called one-thread-per client system, where each client is assigned its own thread. This is the implementation in many servers like Apache.
Since we don't have the liberty of running our js code here, let's see what we can do on Nodejs platform to improve our situation.

### What can we do to improve this on Nodejs?
#### Offloading: The Worker Pool
We know Nodejs excels in I/O-bound tasks, but suffers on CPU-bound operations. One approach to overcome this, is by offloading task to Worker Pool, the second kind of threading the Nodejs environment offers. 

We will look at two approaches; a [Cluster](https://nodejs.org/api/cluster.html) from the `cluster` module and a [Child Process](https://nodejs.org/api/child_process.html) from `child_process` module. 

#### Child Process
This allow for spwaning indepedent subprocess. These subprocesses are indepedent processes with their own memory and Event Loop. The `child_process` API offers various methods of spwaning new processes. Of interest here is the `child_process.fork()` which spawns a new Nodejs process with all the neccessary mechanisms (IPC) for passing messages between itself and the parent process.

The `child_process.fork()` takes 3 arguments, but the mandatory is the modulePath, which is the module to run in the child. 
First, we will create the child module which will contain our recursive function. I will call it `fib-fork.js`. Here is it's contents:

```js

process.send('ready');

process.on("message", (message) => {
  console.log(`Message from Parent: ${message}`);
  process.send(fib(message));
  process.exit(0);
});

process.on("error", (err) => {
  console.log('Process encountered error: ', err);
});

process.on("exit", (message) => {
  console.log('Process exited');
});

const fib = (n) => {
  if (n < 2)
    return n;
  return fib(n - 1) + fib(n - 2);
}
```

We need to modify the server file to accomodate the new changes:

```js
import express from 'express';
import { fork } from 'child_process';
import path from 'path';


const __dirname = path.resolve();

const PORT = process.env.PORT || 8000;

const app = express();
app.use(express.json());

const router = express.Router();

app.use(router);

router.get('/hello', (req, res) => {
    res.status(200).json({ message: 'Hello!' });
});

router.get('/fib/:n', (req, res) => {
    let { n } = req.params;

    const child = fork(path.join(__dirname, 'fib-fork.js'));

    child.on('message', message => {
        // Check if the child is ready to receive 
        // messages. This is neccessary when using
        // es6 module
        if (message === 'ready') {
            child.send(n);
        } else {
            res.status(200).json({ message: 'Hello!', fib: `fib(${n}) = ${message}` });
        }
    });

    child.on('error', message => {
        console.log(`Err: ${message}`);
    });

    child.on('close', code => {
        console.log("child process exited with code " + code);
    });
});


app.listen(PORT, () => {
    console.log(`Server running @: http://localhost:${PORT}`);
});

```
> If using ES6 module, like I have done, then it's necessary for the child to message the parent (although also recommended on commonjs) once it's ready to recieve messages. You can check this issue [here](https://github.com/nodejs/node/issues/34785).

Runnig this, and making a `GET` request for a 45'th term of the Fibonacci i.e `n = 45` on the `/fib` endpoint: 

```bash
http://localhost:4002/fib/45
```
And opening another tab on Postman and making another `GET` request on the `/hello` endpoint, we now immediately get a `Hello!` response as the first request keeps calculating. This means our requests has spwaned a new process to handle our task and free the Event Loop to respond to other request. If you hit the `/fin/n` endpoint again whilst the first request is still running, another process will be spawned to handle that request, and once done, it will message the parent will the result of the computation and then exit.

If you look keenly, you might see why this approach although noble might not be kind enough on system resources as requests increases. With every request to `fib/n` endpoint, a new Nodejs process with it's own memory, Event Loop and all the bells and whistles will be spwaned and eat into our OS resources. A better approach is the use of Worker Threads, that we will look at on another article.

#### Cluster

Perhaps this is the most easiest and by far the most familiar implementation on the internet. If you searched for Nodejs concurrency/parallelism, you will probably see this. The cluster module allows easy creation of child processes that all share server port. 
Form official Nodejs, 

> A single instance of Node.js runs in a single thread. To take advantage of multi-core systems, the user will sometimes want to launch a cluster of Node.js processes to handle the load.

Basically, a cluster will spwan as many processes as are the CPUs and load balance requests to these processes. Care has to be taken not to spawn more processes than the number of cores availabe on your CPU.

Now rewriting our server code we have:

```js
import express from 'express';
import axios from 'axios';
import cluster from 'cluster';
import { cpus } from 'os';
import { fib } from './fib.js';

const PORT = process.env.PORT || 4001;
const CPUs = cpus().length;

if (cluster.isMaster) {
    console.log(`Master ${process.pid} is running`);

    for (let i=0; i<CPUs; i++) {
        cluster.fork();
    }

    cluster.on('exit', (worker, code, signal) => {
        cluster.fork();
    });
} else {

const app = express();
const router = express.Router();

app.use(express.json());
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
}
```
That's all. Runnig this, the server creates as many process as are CPU cores. Now, making a `GET` request for a 45'th term of the Fibonacci i.e `n = 45` on the `/fib` endpoint: 

```bash
http://localhost:4002/fib/45
```
And opening another tab on Postman and making another `GET` request on the `/hello` endpoint, we now immediately get a `Hello!` response as the first request keeps calculating. This means our requests are being handled by different *processes*.

As noted above, this approach is not cheap, since processes are not kind to OS resources pool. Nodejs team has since introduces a better approach called the Worker Threads from the `worker_threads` module, which allows execution in parallel, and are light-weight. We will look at this on the next article.

*Cheers. Happy coding*


