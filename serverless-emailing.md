## Serverless emailing
##### *Sending emails with client-side js only*
###### [@admin](/whoami)
###### Dec 15, 2020 10:13PM
###### [#react]() [#email]() [#serverless]()

Can you really send an email from Javascript (React) without a backend server? Well, you could.. sort of. In my quest of building a [server-less]() blog, I went looking
for a mailing solution that didn't require a server setup. What I found was [emailjs](https://www.emailjs.com/).
##### But how does it work?
From the official emailjs docs, this is what they say:

> EmailJS helps sending emails using client side technologies only. No server is required – just connect EmailJS to one of the supported email services, create an email template and use our Javascript library to trigger an email.

If you ever used [Nodemailer](https://nodemailer.com/) on Node.js, you usually create an SMTP service, say from Gmail, then a create a transport from this SMTP service and finally send the mail with the defined transport object. The emailjs uses the same concept (at least, to my understanding), but now instead of building your own server to do this, they provide you with an API, via thier SDK to achieve the same. Of course their service is not free and comes with a risk of allowing a 3'rd party to send emails on your behalf, but it's a novel ideal. They do offer a free 200 emails credit every month should that suit your need.

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
`useID` is optional if used `init()`.

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


