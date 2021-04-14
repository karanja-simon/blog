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
*Generate server certificate*


 
