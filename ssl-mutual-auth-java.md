## SpringBoot: SSL Mutual Authentication
##### *RESTful API clients authentication via client certificates (The Java Way)*
###### [@admin](/whoami)
###### Jul, 2021 06:30PM
###### [#ssl]() [#rest]() [#springboot]()

## *`This is a draft. It's incomplete and may contain errors or materials that might be removed on final copy.`*

 I have been refreshing my Java/Spring Boot skills for the last few months, and nothing could be better than rewriting a [blog entry](/blog/ssl-client-authentication), I had done previously using Node.js. One of the things I 
 like about Java is how security is intergrated in to the core framework. By simply including the Spring Security, you are literally half-way there to building secure applications.
 In this entry, will look at how easy and robust it is to implement SSL mutual authentication in Java.
#### SSL Mutual Authentication 
For mutual SSL authentication or 2-way-SSL authentication, both the server and client will verify each other's certificate in a 12 steps digital handshake. You read more 
[here](https://www.codeproject.com/Articles/326574/An-Introduction-to-Mutual-SSL-Authentication). For most part, the process is similar to server authentication, except the 
server will request and verify the client's certificate.

#### Generating SSL certificates
This will be identical to how we generated the SSL certificates on the previous entry. We will use OpenSSl to generate our client certificate, of course for secure/production systems you can 
buy certificates from trusted authority like Sentigo. We will be our own Certificate Authority (CA) and issue both the server and client certificates. Assuming you have OpenSSL already setup.
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

#### 4.0 The Java way
Here is where Java does things a bit differently. For this to work, Java needs something called a keystore and a truststore, both of which can be generated with a keytool (It's already included in your JDK).
Now, what are these stores I hear? A keystore is repository that holds your server's key and certificate, while the truststore, well, it holds the Certificate Authority i.e the certificates that will be used to identify others. Java has a proprietary format: *Java Key Store (.jks)* although it also supports the standard *Public-Key Cryptography Standards (PKCS#12) .p12 or .pfx*.

##### 4.1 Import to keystore
We will now import the generated keys and certificates into the keystore. We will PKCS to package our key and certificate together.
Let's start with the server's:

```sh
openssl pkcs12 -export -out server.p12 -name "server" -inkey server/server-key.pem -in server/server-crt.pem
```
Now that we have combined the certificate and it's correspoding key, let's create the keystore:

```sh
keytool -importkeystore -srckeystore server.p12 -srcstoretype PKCS12 -destkeystore server-keystore.jks -deststoretype JKS
```

#### Some Code
Go ahead to [Spring Initializr](https://start.spring.io/) and generate a boilerplate with `Spring Web`, `Thymeleaf` and `Spring Security`, download and extract the files. Next let's create a simple controller

```java

@Controller
public class UserController {
    @RequestMapping(value = "/user")
    public String user(Model model, Principal principal) {
        
        UserDetails currentUser 
          = (UserDetails) ((Authentication) principal).getPrincipal();
        model.addAttribute("username", currentUser.getUsername());
        return "user";
    }
}

```
For the configuration, add the following to `application.properties`

```js 
server.ssl.key-store=../store/keystore.jks
server.ssl.key-store-password=${PASSWORD}
server.ssl.key-alias=localhost
server.ssl.key-password=${PASSWORD}
server.ssl.enabled=true
server.port=8443
spring.security.user.name=Admin
spring.security.user.password=admin
```

#### Testing
Am using Postman to simulate an API request. To set the client certificates on Postman, go to Settings > Certificates then add the CA and the client certificates.  
| ![Postman certificate](/images/blog/ssl/postman.png) |
|:--:|
| *Client certificate, Postman* |

Running this and hitting the `/private` end point, I get a success message:-

```json
{
    "message": "Welcome to private section"
}
```

Try providing an invalid certificate, and the request should fail with a `401` status code.

#### Sources

Get the project source code here: [https://github.com/karanja-simon/node.js_ssl_client_authentication](https://github.com/karanja-simon/node.js_ssl_client_authentication)

#### References
1. [https://techcommunity.microsoft.com/t5/iis-support-blog/client-certificate-authentication-part-1/ba-p/324623](https://techcommunity.microsoft.com/t5/iis-support-blog/client-certificate-authentication-part-1/ba-p/324623)
2. [https://www.matteomattei.com/client-and-server-ssl-mutual-authentication-with-nodejs/](https://www.matteomattei.com/client-and-server-ssl-mutual-authentication-with-nodejs/)

