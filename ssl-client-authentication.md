## SSL Clients Authentication
##### *RESTful API clients authentication via client certificates*
###### [@admin](/whoami)
###### Mar 14, 2021 12:20PM
###### [#ssl]() [#rest]()

In traditional RESTful systems, authentication is quite straight-forward. A client registers in the platform and then uses these credentials stored on the platform to authenticate themselves. Now, what if you don't have a prior knowledge of a client, i.e the client doesn't exist on your platform? How would you know a genuine client vs a malicious one? The answer lies on ssl client certificates.

#### 1.0 Some background 
For the most part, SSL is used that to facilitate encryption and establish trust between clients and web browsers. Whilst this is what is known by most 
people, SSL can be use to authenticate clients, a good example is SSH authentication keys. 
For this, we will use Open SSl to generate our client certificate, of course for secure/production systems you can 
buy certificates from trusted authority like Sentigo. 

 
