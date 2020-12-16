![ke-postal-api](/images/kepostalapi/01.png)
### [KE Postal API](/portfolio/ke-postal-api)
##### Jul 05, 2018 05:22 PM
###### [#node.js]() [#express]() [#rest]()

Kenya-Postal-Codes-API API provides a lookup for the Kenya Postal Corporation codes and addresses.
Version 1 of the API is limited to the essentials: listing all postal codes, search by code or name.

The API is [REST API](http://en.wikipedia.org/wiki/Representational_State_Transfer "RESTful")
and uses [JWT](http://oauth.net/ "OAuth") tokens for user authentication purposes.
Currently, return format for all endpoints is [JSON](http://json.org/ "JSON").

***

## Checklist
* Familiarize yourself with API functionality
* Read the Kenya-Postal-Codes-API [API Terms of Use][]
* [Register your application][] and get a token
* Hack away

***

## Changes

* 2018-04-10 first API release

## SDK

## Endpoints

#### Codes Resources

- **[`GET` codes]()**
- **[`GET` code/:code]()**
- **[`GET` names/:name]()**

## Authentication
- **[`GET` key]()**
- **[`GET` verify/key]()**


## FAQ
### What do I need to know before I start using the API?
Here are the docs you might need to get started:

- HTTPS protocol
- [REST software pattern][]
- Authentication with [OAuth][] (or the official [Beginner’s Guide][])
- Data serialization with [JSON][] (or see a [quick tutorial][])

### How do I connect to the Kenya-Postal-Codes-API?
The API is only available to authenticated clients. Clients should authenticate users using [JWT][]. Once authenticated, you need to request a resource from one of the endpoints using HTTPS. Generally, reading any data is done through a request with GET method. If you want our server to create, update or delete a given resource, POST or PUT methods are required.

### What return formats do you support?
Kenya-Postal-Codes-API currently returns data in [JSON](http://json.org/ "JSON") format.

### What kind of authentication is required?
Applications must identify themselves to access any resource.


### Is there a request rate limit?
There is no limit.

[REST software pattern]: http://en.wikipedia.org/wiki/Representational_State_Transfer
[OAuth]: http://oauth.net/core/1.0a/
[Beginner’s Guide]: http://hueniverse.com/oauth/
[JSON]: http://json.org
