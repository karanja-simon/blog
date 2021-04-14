## SSL Clients Authentication
##### *RESTful API clients authentication via client certificates*
###### [@admin](/whoami)
###### Mar 14, 2021 12:20PM
###### [#ssl]() [#rest]()

In traditional RESTful systems, authentication is quite straight-forward. A client registers in the platform and then uses these credentials stored on the platform to authenticate themselves. Now, what if you don't have a prior knowledge of a client, i.e the client doesn't exist on your platform? How would you know a genuine client vs a malicious one? The answer lies on ssl client certificates.

#### SSL Mutual Authentication 
For the most part, SSL is used that to facilitate encryption and establish trust between clients and web browsers. This is the basic SSL authentication i.e, the server presents a certificate that the client verifies against its trusted certificate authorities. Whilst this is what is known by most 
people, SSL can be use to authenticate clients, called SSL mutual authetication. Generally the above applies, but in addition, the server challenges the client to provide a certificate during the TLS handshake. This in turn forces the client to provide a valid certificate before the server can provide any resource. We can implement our custom checks here to see if the incoming certificate is valid and grant access, otherwise deny.

| ![SSL Mutual Authentication](/images/blog/ssl/mutual-auth.png) | 
|:--:| 
| *SSL Mutual Authentication. Image credit of [cloudboost.io](https://blog.cloudboost.io/implementing-mutual-ssl-authentication-fc20ab2392b3)* |


#### Generating SSL certificates
For this, we will use OpenSSl to generate our client certificate, of course for secure/production systems you can 
buy certificates from trusted authority like Sentigo. Here we will be our own Certificate Authority (CA) and issue both the server and client certificates. Assuming you have OpenSSL already setup.
*Create two folders/directories, i.e server and client where the correspoding certificates files will be stored.*
##### 1.0 Generate a certificate authority

```sh
openssl req -x509 -newkey rsa:4096 -keyout server/ca-key.pem -out server/ca-crt.pem -nodes -days 365
```
##### 2.0 Generate server private key

```sh
openssl genrsa -out server/server-key.pem 4096
```
##### 2.1 Generate server certificate signing request

```sh
openssl req -new -sha256 -key server/server-key.pem -out server/server-csr.pem
```
##### 2.2 Generate server certificate

```sh
openssl x509 -req -days 365 -in server/server-csr.pem -CA server/ca-crt.pem -CAkey server/ca-key.pem -CAcreateserial -out server/server-crt.pem
```
##### 3.0 Generate client private key

```sh
openssl genrsa -out client/client-key.pem 4096
```
##### 3.1 Generate client certificate signing request

```sh
openssl req -new -sha256 -key client/client-key.pem -out client/client-csr.pem
```
##### 3.2 Generate client certificate

```sh
openssl x509 -req -days 365 -in client/client-csr.pem -CA server/ca-crt.pem -CAkey server/ca-key.pem -CAcreateserial -out client/client-crt.pem
```

#### Some Code
Now, for the Node.js part, lets build a simple https server with express
```js
const express = require("express");
const https = require("https");
const path = require("path");
const fs = require("fs");
const authenticate = require("./authenticate");

const PORT = process.env.PORT || 3000;
const app = express();
const privateRouter = express.Router();

const __DIR = path.join(__dirname, './ssl/server');

const opts = {
	key: fs.readFileSync(path.join(__DIR, 'server-key.pem')),
	cert: fs.readFileSync(path.join(__DIR, 'server-crt.pem')),
	requestCert: true,
	rejectUnauthorized: true,
	ca: [
		fs.readFileSync(path.join(__DIR, 'ca-crt.pem'))
	]
};

// Here, we register our authentication middleware. It will
// be responsible for checking and validating client certificates.
app.use("/private", authenticate, privateRouter);

https.createServer(opts, app).listen(PORT, () => {
    console.log(`⚡️[server]: Server is running at https://localhost:${PORT}`);
});

```
Of importance here is line 77, i.e

```js 
rejectUnauthorized: true 
```

With this, we reject any unauthenticated traffic. Ofcourse for debug purposes, you can set it to false since any unauthenticated requests will be reject silently.
Now for the fun part, let look at our authentication middleware.

```js 
const authenticate = (req, res, next) => {
    const cert = req.socket.getPeerCertificate();

    console.log('Cert: ', cert);

	if (req.client.authorized) {
		res.send(`Hello ${cert.subject.CN}, your certificate was issued by ${cert.issuer.CN}!`);

	} else if (cert.subject) {
		res.status(403)
			 .send(`Sorry ${cert.subject.CN}, certificates from ${cert.issuer.CN} are not welcome here.`);

	} else {
		res.status(401)
		   .send(`Sorry, but you need to provide a client certificate to continue.`);
	}
}

module.exports = authenticate;
```

