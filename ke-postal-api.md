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
* [Try the API console](http://bitly.com/api500px)
* Familiarize yourself with API functionality
* Read the Kenya-Postal-Codes-API [API Terms of Use][]
* [Register your application][] and get a token
* Hack away

***

## Basics

- **[Formats and Terms](https://github.com/500px/api-documentation/blob/master/basics/formats_and_terms.md)**
- **[API Terms of Use](https://github.com/500px/api-documentation/blob/master/basics/terms_of_use.md)**


## Examples (Consuming the API)

- **[Angular](http://500px.github.com/500px-js-sdk)**
- **[Java](https://github.com/500px/api-documentation/blob/master/examples/iOS/API%20Tutorials.md)**


## Changes

* 2018-04-10 first API release

## SDK

## Endpoints

#### Codes Resources

- **[<code>GET</code> codes](https://github.com/500px/api-documentation/blob/master/endpoints/photo/GET_photos.md)**
- **[<code>GET</code> code/:code](https://github.com/500px/api-documentation/blob/master/endpoints/photo/GET_photos_search.md)**
- **[<code>GET</code> names/:name](https://github.com/500px/api-documentation/blob/master/endpoints/photo/GET_photos_id.md)**

## Authentication
- **[<code>GET</code> key](https://github.com/500px/api-documentation/blob/master/endpoints/user/GET_users.md)**
- **[<code>GET</code> verify/key](https://github.com/500px/api-documentation/blob/master/endpoints/user/GET_users_show.md)**


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
[quick tutorial]: http://www.webmonkey.com/2010/02/get_started_with_json/
[Register your application]: http://500px.com/settings/applications
[API Terms of Use]: https://github.com/500px/api-documentation/blob/master/basics/terms_of_use.md
[See if the concepts used by the API are familiar to you]: https://github.com/500px/api-documentation#what-do-i-need-to-know-before-i-start-using-the-api
