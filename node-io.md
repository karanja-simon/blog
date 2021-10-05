## How much does Nodejs suck at CPU intesive computations?
##### *A look at Nodejs CPU-bound task handling*
###### [@admin](/whoami)
###### Oct 05, 2021 04:13PM
###### [#nodejs]() [#non-blocking]() [#threads]()

Javascript is inherently a sigle-threaded language. This makes it increadibly easy to build applications since developers don't need to think or handle the complex multi-thread environment, it's also the biggest weakness of the language. Performing CPU intensive tasks will block the main thread and render your application unresponsive.

Nodejs executes JavaScript code in the Event Loop (main thread) just like the browsers do, but it also offers a Worker Pool to handle expensive tasks like I/O. If the Event Loop (main thread) or a Worker thread is held by a long running task, say a CPU intensive task, then it cannot respond to requests from other clients and it's said to be blocked. This obviously leads to subsequent requests waiting for the *greedy* task and will lead to a bad user experience, or in a worst case; a Denial of Service.

##### Let's see this in action.
Let's build a simple Nodejs/Express server that calculates the Fibonacci of an n'th term. I will implement a worst-case time-complexity algorithm for calculating Fibonacci using recursion.
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

##### How would a Multi-threaded system cope?
I will implement the same algorithm using Java Spring Boot with Apache Tomcat and see how it copes with the same. Below is the recursive method in Java:

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

### What can we do to improve this on Nodejs?
#### Offloading: Worker Pool
We know Nodejs excels in I/O-bound tasks, but suffers on complex calculations. One approach to overcome this, is by offloading task to Worker Pool, the second kind of threading the Nodejs environment offers. The downside of this, is the communication overhead between the Worker and Event Loop. From [Nodejs docs](https://nodejs.org/api/cluster.html):

>  From a Worker, you cannot manipulate a JavaScript object in the Event Loop's namespace. Instead, you have to serialize and deserialize any objects you wish to share. 

To do the offloading, we will implement a [Cluster](https://nodejs.org/api/cluster.html). This allows creation of child processes that shares the server ports. Now rewriting our server code we have:

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
Runnig this, the server creates as many process as are CPU cores. Now, making a `GET` request for a 45'th term of the Fibonacci i.e `n = 45` on the `/fib` endpoint: 

```bash
http://localhost:4002/fib/45
```
And opening another tab on Postman and making another `GET` request on the `/hello` endpoint, we now immediately get a `Hello!` response as the first request keeps calculating. This means our requests are being handled by a different process. The biggest drawback to this is if you need to have a shared state or sessions.  

*Happy coding*


