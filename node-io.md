## How much does Nodejs suck at CPU intesive computations?
##### *A look at the weakness of js single thread model*
###### [@admin](/whoami)
###### Oct 5, 2021 04:13PM
###### [#nodejs]() [#non-blocking]() [#threads]()

Javascript is inherently a sigle-threaded language. This makes it increadibly easy to build applications since developers don't need to think or handle the complex multi-thread environment, it's also the biggest weakness of the language. Performing CPU intensive tasks will block the main thread and render your application unresponsive.

Nodejs executes JavaScript code in the Event Loop (main thread) just like the browsers do, but it also offers a Worker Pool to handle expensive tasks like I/O. If the Event Loop (main thread) or a Worker thread is held by a long running task, say a CPU intensive task, then it cannot respond to requests from other clients and it's said to be blocked. This obviously leads to subsequent requests waiting for the *greedy* task and will lead to a bad user experience, or in a worst case; a Denial of Service.

##### Let's see this in action.
Let's build a simple Nodejs/Express server that calculates the Fibonacci of an n'th term. I will implement a worst-case time-complexity algorithm for calculating Fibonacci using recursion.
Let's look at the recursive function:

```js
export const fib = (n) => {
    if (n < 2) {
        return n;
    } else {
        return fib(n - 1) + fib(n - 2);
    }
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

If 

##### How do you use it then?
Install their official SDK
```sh
yarn add emailjs-com
```
Initialize the SDK
```ts
import emailjs from 'emailjs-com';

emailjs.init("YOUR_USER_ID");
```
`YOUR_USER_ID` will be available on [intergration](https://dashboard.emailjs.com/admin/integration) section on your dashboard.
Now, sending the email is quite easy. Emailjs provides two ways of sending an email.

> Note: Both method are rate limited at 1 request/sec.

#### Option 1

```js
emailjs.send(serviceID, templateID, templateParams, userID);
```
`serviceID` will be assigned when configuring your service and `templateID` when setting up your template on your [admin dashboard](https://dashboard.emailjs.com/admin/).
`useID` is optional if you used `init()`.

Example:

```js
const params = {
name: 'John',
reply_email: 'john@example.com',
message: 'This is awesome!'
};
emailjs.send('YOUR_SERVICE_ID', 'YOUR_TEMPLATE_ID', templateParams)
    .then(function(response) {
       console.log('SUCCESS!', response.status, response.text);
    }, function(error) {
       console.log('FAILED...', error);
    });
```
#### Option 2

This will automatically collect the values of the form and pass them to the specified template.

```js
emailjs.sendForm(serviceID, templateID, templateParams, userID);
```

Example:

```js
import React from 'react';
import emailjs from 'emailjs-com';

const ContactMe = () => {

  const sendEmail = (e) => {
    e.preventDefault();

    emailjs.sendForm('YOUR_SERVICE_ID', 'YOUR_TEMPLATE_ID', e.target, 'YOUR_USER_ID')
      .then((result) => {
          console.log(result.text);
      }, (error) => {
          console.log(error.text);
      });
  }

  return (
    <form onSubmit={sendEmail}>
      <label>Email</label>
      <input type="email" name="user_email" />
      <label>Message</label>
      <textarea name="message" />
      <input type="submit" value="Send" />
    </form>
  );
}
```

**The input names on the form should correspond to the names on the template.**
If everything is setup properly, you should recieve an email from the mail address you provided when setting up your template.

*Happy coding*


