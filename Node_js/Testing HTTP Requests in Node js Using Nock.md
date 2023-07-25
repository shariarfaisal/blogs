# Testing HTTP Requests in Node js Using Nock

```Node.js```

Writing tests can be a very important part of your development cycle. Most code is easy to test, data goes in, write tests to make sure the right data comes out. Things start to get difficult when you introduce network requests. It’s easy enough to stand up an API server and writing integration tests, but what if you don’t want that much overhead or you are trying to test against a third-party API? nock has got you covered.


nock, or “network mock”, is a library for mocking HTTP server requests. It not only works great with Node’s built-in http and https requests, but also plays nice with other request libraries that use those interfaces, like superagent or axios.


# Getting Started


Getting started with nock is easy, all we need to do is add it to our project. I recommend installing it as a development dependency since most likely you won’t be using it on your production site.


## Via yarn


```
$ yarn add Jock --dev

```


## Via npm


```
$ npm install nock --save-dev

```


Don’t forget to require it in your test script:


```
const nock = require('nock');

```


# The Basics


nock works by allowing us to define mocked network requests, or interceptors.


The nock object itself will receive a hostname and then we can chain the request method (.get(), .post(), .put(), or .delete()) we want to mock to it.


When chaining the request method, we need to pass-in the URI of the endpoint as the first argument and optionally, the request body after that.


The last thing we need is to chain the mocked reply from the server. The .reply() accepts an HTTP status code as the first argument and then optionally, the response.


When we put it all together, it will look something like this:


```
nock('http://httpbin.org')
  .post('/post', { id: '123' })
  .reply(200, { status: 'OK' });

```


# The Interceptor


Before executing code that will make a network request that we want to test, we have to setup a mocked network request that will be used in place of the network request in your code:


```
nock('http://httpbin.org')
  .get('/get')
  .reply(200, {
    "args": {},
    "headers": {
      "Accept":
      "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
      "Accept-Encoding": "gzip, deflate",
      "Accept-Language": "en-US,en;q=0.9",
      "Connection": "close",
      "Cookie": "_gauges_unique_hour=1;
      _gauges_unique_day=1; _gauges_unique_month=1;
      _gauges_unique_year=1; _gauges_unique=1",
      "Host": "httpbin.org",
      "Upgrade-Insecure-Requests": "1",
      "User-Agent": "Mozilla/5.0 (X11; Linux
      x86_64) AppleWebKit/537.36 (KHTML, like
        Gecko) Chrome/70.0.3538.102 Safari/537.36"
      },
      "origin": "0.0.0.0",
      "url": "http://httpbin.org/get"
    }
  });

// Some code that GETs http://httpbin.org/get

```


The nock above will return successfully (200 OK) and return a payload that mimics the data that is returned by httpbin.org. All without actually making the external network request!


While we may be losing out on having a full fledged integration test, this method does speed things up substantially and means your test harness isn’t relying on the third-party service being online.


This is especially useful if your tests have to pass before code is deployed as part of your CI/CD pipeline.


Internally, nock keeps a list of these interceptors and as they are used, they are removed from the list.


The interceptors are used in the order they are created and we can define multiple interceptors that go to the same hostname and URI, but return different payloads.


Because interceptors are removed after they are used, any time we want to test a network call, we have to make sure that we have an interceptor available.


# Mocking Server Errors


Sadly, a lot of APIs out there don’t include a mechanism for being able to simulate errors being returned by the API. This shortcoming usually leads to writing a bunch of happy path tests that give no indication as to how well the system will respond to network failures or errors.


Because we’re mocking the requests, it doesn’t matter if the API we’re working with can simulate errors, we can just simulate those bad requests ourselves!


Let’s say we want to test how our code handles a 5xx server error:


```
nock('http://httpbin.org')
  .post('/post')
  .reply(500);

// Code we want to make sure handles errors...

```


Being able to test all our exception paths means we’re that much closer to the mythical 100% code coverage! 🦄


Happy nocking!


