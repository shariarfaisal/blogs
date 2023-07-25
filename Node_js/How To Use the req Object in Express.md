# How To Use the req Object in Express

```Node.js```

## Introduction


Short for request, the req object is one half of the request and response cycle to examine calls from the client side, make HTTP requests, and handle incoming data whether in a string or JSON object.


In this article, you will learn about the req object in Express.


# Prerequisites


To follow along with this article, you will need:


- A general understanding of Node.js is suggested, but not required. To learn more about Node.js, check out our How To Code in Node.js series.
- A general knowledge of HTTP requests.

# Managing Client-Side Data


Express servers receive data from the client side through the req object in three instances: the req.params, req.query, and req.body objects.


The req.params object captures data based on the parameter specified in the URL. In your index.js file, implement a GET request with a parameter of '/:userid':


index.js
```
// GET https://example.com/user/1

app.get('/:userid', (req, res) => {
  console.log(req.params.userid) // "1"
})

```


The req.params object tells Express to output the result of a user’s id through the parameter '/:userid'. Here, the GET request to https://example.com/user/1 with the designated parameter logs into the console an id of "1".


To access a URL query string, apply the req.query object to search, filter, and sort through data. In your index.js file, include a GET request to the route '/search':


index.js
```
// GET https://example.com/search?keyword=great-white

app.get('/search', (req, res) => {
  console.log(req.query.keyword) // "great-white"
})

```


Utilizing the req.query object matches data loaded from the client side in a query conditional. In this case, the GET request to the '/search' route informs Express to match keywords in the search query to https://example.com. The result from appending the .keyword property to the req.query object logs into the console, "great-white".


The req.body object allows you to access data in a string or JSON object from the client side. You generally use the req.body object to receive data through POST and PUT requests in the Express server.


In your index.js file, set a POST request to the route '/login':


```
// POST https://example.com/login
//
//      {
//        "email": "user@example.com",
//        "password": "helloworld"
//      }

app.post('/login', (req, res) => {
  console.log(req.body.email) // "user@example.com"
  console.log(req.body.password) // "helloworld"
})

```


When a user inputs their email and password on the client side, the req.body object stores that information and sends it to your Express server. Logging the req.body object into the console results in the user’s email and password.


Now that you’ve examined ways to implement the req object, let’s look at other approaches to integrate into your Express server.


# Examining the URL With req Properties


Properties on the req object can also return the parts of a URL based on the anatomy. This includes the protocol, hostname, path, originalUrl, and subdomains.


In your index.js file, set a GET request to the '/creatures' route:


index.js
```
// https://ocean.example.com/creatures?filter=sharks

app.get('/creatures', (req, res) => {
  console.log(req.protocol)     // "https"
  console.log(req.hostname)     // "example.com"
  console.log(req.path)         // "/creatures"
  console.log(req.originalUrl)  // "/creatures?filter=sharks"
  console.log(req.subdomains)   // "['ocean']"
})

```


You can access various parts of the URL using built-in properties such as .protocol and .hostname. Logging the req object with each of the properties results in the anatomy of the URL.


# Analyzing Additional req Properties


The res object consists of properties to maximize your calls to HTTP requests.


To access the HTTP method, whether a GET, POST, PUT, or DELETE, utilize the .method property to your req object. In your index.js file, implement a DELETE request to an anonymous endpoint:


index.js
```
app.delete('/', (req, res) => {
  console.log(req.method) // "DELETE"
})

```


The .method property returns the current HTTP request method. In this case, the console logs a DELETE method.


For headers sent into your server, append the method .header() to your req object. Set a POST request to the route '/login' in your index.js file:


index.js
```
app.post('/login', (req, res) => {
  req.header('Content-Type')  // "application/json"
  req.header('user-agent')    // "Mozilla/5.0 (Macintosh Intel Mac OS X 10_8_5) AppleWebKi..."
  req.header('Authorization') // "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
})

```


The req.header() method will return the header type such as Content-Type and Authorization. The argument for req.header() is case-insensitive so you can use req.header('Content-Type') and req.header('content-type') interchangeably.


If you’ve added cookie-parser as a dependency in your Express server, the req.cookies property will store values from your parser. In your index.js file, set an instance of req.cookies and apply the sessionDate property:


index.js
```
// Cookie sessionDate=2019-05-28T01:49:11.968Z

req.cookies.sessionDate // "2019-05-28T01:49:11.968Z"

```


Notice the result returned from the a cookie’s session date when called from the req object.


# Conclusion


Express provides built-in properties to utilize the req object as part of the request cycle to handle HTTP requests and data from the client side. If you’d like to view the official documentation on req please visit the official Express.js docs.


