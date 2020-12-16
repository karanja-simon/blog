## Serverless emailing
##### *Sending emails with client-side only*
###### [@admin](/whoami)
###### Dec 15, 2020 10:13PM
###### [#react]() [#email]() [#serverless]()

Can you really send an email from Javascript (React) without a backend server? Well, you could.. sort of. In my quest of building a [server-less]() blog, I went looking
for a mailing solution that didn't require a server setup. What I found was [emailjs](https://www.emailjs.com/).
##### But how does it work?
From the official emailjs docs, this is what they say:
> EmailJS helps sending emails using client side technologies only. No server is required – just connect EmailJS to one of the supported email services, create an email template, > and use our Javascript library to trigger an email.

```js
const params = {
name: 'John',
reply_email: 'john@example.com',
message: 'This is awesome!'
};
emailjs.send( 'gmail', 'feedback', params );
```
[Read more](/blog/10002)
