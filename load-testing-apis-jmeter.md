## Load Testing API's with Apache's JMeter
##### *How good is your API?*
###### [@admin](/whoami)
###### Dec 26, 2020 09:30PM
###### [#api]() [#jmeter]()

Unless you fancy the built-in fetch API, chances are that you are using or have used axios as your HTTP client. If so, when writing tests you may need to mock API requests. You could of course use mock functions provided by your test library in this case Jest, but I have found a nifty package called [axios-mock-adapter](https://www.npmjs.com/package/axios-mock-adapter) to be excellent and natural when testing codes with axios implementation.
Lets write a test for a simple component that fetches some random todos. A fully working implementation can be be found here. You can also see the code in action on this codesandbox.
