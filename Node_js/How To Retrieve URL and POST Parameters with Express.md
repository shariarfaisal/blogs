# How To Retrieve URL and POST Parameters with Express

```Node.js```

## Introduction


Often when you are building applications using Express, you will need to get information from your users. Two of the most popular methods are URL parameters and POST parameters.


In this article, you will learn how to use Express to retrieve URL parameters and POST parameters from requests.


# Prerequisites


To complete this tutorial, you will need:


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.
- Downloading and installing a tool like Postman will be required for sending POST requests.


Note: Previously, this tutorial recommended using req.param. This is deprecated as of v4.11.0. This tutorial also recommended installing body-parser. This is no longer necessary as of v4.16.0.

This tutorial was verified with Node v15.4.0, npm v7.10.0, and express v4.17.1.


# Step 1 – Setting Up the Project


First, open your terminal window and create a new project directory:


```
mkdir express-params-example


```


Then, navigate to the newly created directory:


```
cd express-params-example


```


At this point, you can initialize a new npm project:


```
npm init -y


```


Next, you will need to install the express package:


```
npm install express@4.17.1


```


At this point, you have a new project ready to use Express.


Create a new server.js file and open it with your code editor:


server.js
```
const express = require('express');

const app = express();
const port = process.env.PORT || 8080;

// routes will go here

app.listen(port);
console.log('Server started at http://localhost:' + port);

```


Revisit your terminal window and run your application:


```
node server.js


```



You will have to restart the node server every time you edit server.js. If this gets tedious, see How To Restart Your Node.js Apps Automatically with nodemon.

Now let’s create two routes now to test grabbing parameters.


# Step 2 – Using req.query with URL Parameters


req.query can be used to retrieve values for URL parameters.


Consider the following example:


```
http://example.com/api/users?id=4&token=sdfa3&geo=us

```


This URL includes parameters for id, token, and geo (geolocation):


```
id: 4
token: sdfa3
geo: us

```


Revisit server.js with your code editor and add the following lines of code for req.query.id, req.query.token, and req.query.geo:


server.js
```
// ...

// routes will go here
// ...

app.get('/api/users', function(req, res) {
  const user_id = req.query.id;
  const token = req.query.token;
  const geo = req.query.geo;

  res.send({
    'user_id': user_id,
    'token': token,
    'geo': geo
  });
});

app.listen(port);
console.log('Server started at http://localhost:' + port);

```


With the server running, use the URL http://localhost:8080/api/users?id=4&token=sdfa3&geo=us in either a web browser or with Postman.


The server will respond back with the user_id, token, and geo values.


# Step 3 – Using req.params with Routes


req.params can be used to retrieve values from routes.


Consider the following URL:


```
http://localhost:8080/api/1

```


This URL includes routes for api and :version (1).


Revisit server.js with your code editor and add the following lines of code for req.params.version:


server.js
```
// ...

// routes will go here
// ...

app.get('/api/:version', function(req, res) {
  res.send(req.params.version);
});

app.listen(port);
console.log('Server started at http://localhost:' + port);

```


With the server running, use the URL http://localhost:8080/api/1 in either a web browser or with Postman.


The server will respond back with the version value.


# Step 4 – Using .param with Route Handlers


Next up, you are using the Express .param function to grab a specific parameter. This is considered middleware and will run before the route is called.


This can be used for validations (like checking if a user exists) or grabbing important information about that user or item.


Consider the following URL:


```
http://localhost:8080/api/users/sammy

```


This URL includes routes for users and :name (Sammy).


Revisit server.js with your code editor and add the following lines of code for modifying the name:


server.js
```
// ...

app.param('name', function(req, res, next, name) {
  const modified = name.toUpperCase();

  req.name = modified;
  next();
});

// routes will go here
// ...

app.get('/api/users/:name', function(req, res) {
  res.send('Hello ' + req.name + '!');
});

app.listen(port);
console.log('Server started at http://localhost:' + port);

```


With the server running, use the URL http://localhost:8080/api/users/sammy in either a web browser or with Postman.


The server will respond back with:


```
OutputHello SAMMY!

```


You can use this param middleware for validations and making sure that information passed through is valid and in the correct format.


Then save the information to the request (req) so that the other routes will have access to it.


# Step 5 – Using req.body with POST Parameters


express.json() and express.urlencoded() are built-in middleware functions to support JSON-encoded and URL-encoded bodies.


Open server.js with your code editor and add the following lines of code:


server.js
```
const express = require('express');

const app = express();
const port = process.env.PORT || 8080;

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// ...

```


Next, add app.post with req.body.id, req.body.token, and req.body.geo:


server.js
```
// ...

// routes will go here
// ...

app.post('/api/users', function(req, res) {
  const user_id = req.body.id;
  const token = req.body.token;
  const geo = req.body.geo;

  res.send({
    'user_id': user_id,
    'token': token,
    'geo': geo
  });
});

app.listen(port);
console.log('Server started at http://localhost:' + port);

```


With the server running, generate a POST request with Postman.



Note: If you need assistance navigating the Postman interface for requests, consult the official documentation.

Set the request type to POST and the request URL to http://localhost:8080/api/users. Then set Body to x-www-form-urlencoded.


Then, provide the following values:





Key
Value




id
4


token
sdfa3


geo
us




After submitting the response, the server will respond back with the user_id, token, and geo values.


# Conclusion


In this article, you learned how to use Express to retrieve URL parameters and POST parameters from requests. This was achieved with req.query, req.params, and req.body.


Continue your learning with Learn to Use the Express 4.0 Router and How To Deliver HTML Files with Express.


