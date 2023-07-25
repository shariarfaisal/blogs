# How To Get Started with Node js and Express

```Node.js```

## Introduction


Express is a web application framework for Node.js that allows you to spin up robust APIs and web servers in a much easier and cleaner way. It is a lightweight package that does not obscure the core Node.js features.


In this article, you will install and use Express to build a web server.


# Prerequisites


If you would like to follow along with this article, you will need:


- A local development environment for Node.js. Follow How to Install Node.js and Create a Local Development Environment.

This tutorial was verified with Node v15.14.0, npm v7.10.0, express v4.17.1, and serve-index v1.9.1.


# Step 1 — Setting Up the Project


First, open your terminal window and create a new project directory:


```
mkdir express-example


```


Then, navigate to the newly created directory:


```
cd express-example


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


# Step 2 — Creating an Express Server


Now that Express is installed, create a new server.js file and open it with your code editor. Then, add the following lines of code:


server.js
```
const express = require('express');

const app = express();

```


The first line here is grabbing the main Express module from the package you installed. This module is a function, which we then run on the second line to create our app variable. You can create multiple apps this way, each with its own requests and responses.


server.js
```
const express = require('express');

const app = express();

app.get('/', (req, res) => {
  res.send('Successful response.');
});

```


These lines of code is where we tell our Express server how to handle a GET request to our server. Express includes similar functions for POST, PUT, etc. using app.post(...), app.put(...), etc.


These functions take two main parameters. The first is the URL for this function to act upon. In this case, we are targeting '/', which is the root of our website: in this case, localhost:3000.


The second parameter is a function with two arguments: req, and res. req represents the request that was sent to the server; We can use this object to read data about what the client is requesting to do. res represents the response that we will send back to the client.


Here, we are calling a function on res to send back a response: 'Successful response.'.


server.js
```
const express = require('express');

const app = express();

app.get('/', (req, res) => {
  res.send('Successful response.');
});

app.listen(3000, () => console.log('Example app is listening on port 3000.'));

```


Finally, once we’ve set up our requests, we must start our server! We are passing 3000 into the listen function, which tells the app which port to listen on. The function passed in as the second parameter is optional and runs when the server starts up. This provides us some feedback in the console to know that our application is running.


Revisit your terminal window and run your application:


```
node server.js


```


Then, visit localhost:3000 in your web browser. Your browser window will display: 'Successful response'. Your terminal window will display: 'Example app is listening on port 3000.'.


And there we have it, a web server! However, we definitely want to send more than just a single line of text back to the client. Let’s briefly cover what middleware is and how to set this server up as a static file server!


# Step 3 — Using Middleware


With Express, we can write and use middleware functions, which have access to all HTTP requests coming to the server. These functions can:


- Execute any code.
- Make changes to the request and the response objects.
- End the request-response cycle.
- Call the next middleware function in the stack.

We can write our own middleware functions or use third-party middleware by importing them the same way we would with any other package.


Let’s start by writing our own middleware, then we’ll try using some existing middleware to serve static files.


To define a middleware function, we call app.use() and pass it a function. Here’s a basic middleware function to print the current time in the console during every request:


server.js
```
const express = require('express');

const app = express();

app.use((req, res, next) => {
  console.log('Time: ', Date.now());
  next();
});

app.get('/', (req, res) => {
  res.send('Successful response.');
});

app.listen(3000, () => console.log('Example app is listening on port 3000.'));

```


The next() call tells the middleware to go to the next middleware function if there is one. This is important to include at the end of our function - otherwise, the request will get stuck on this middleware.


We can optionally pass a path to the middleware, which will only handle requests to that route. For example:


server.js
```
const express = require('express');

const app = express();

app.use((req, res, next) => {
  console.log('Time: ', Date.now());
  next();
});

app.use('/request-type', (req, res, next) => {
  console.log('Request type: ', req.method);
  next();
});

app.get('/', (req, res) => {
  res.send('Successful response.');
});

app.listen(3000, () => console.log('Example app is listening on port 3000.'));

```


By passing '/request-type' as the first argument to app.use(), this function will only run for requests sent to localhost:3000/request-type.


Revisit your terminal window and run your application:


```
node server.js


```


Then, visit localhost:3000/request-type in your web browser. Your terminal window will display the timestamp of the request and 'Request type:  GET'.


Now, let’s try using existing middleware to serve static files. Express comes with a built-in middleware function: express.static. We will also use a third-party middleware function, serve-index, to display an index listing of our files.


First, inside the same folder where the express server is located, create a directory named public and put some files in there.


Then, install the package serve-index:


```
npm install serve-index@1.9.1


```


First, import the serve-index package at the top of the server file.


Then, include the express.static and serveIndex middlewares and tell them the path to access from and the name of the directory:


server.js
```
const express = require('express');
const serveIndex = require('serve-index');

const app = express();

app.use((req, res, next) => {
  console.log('Time: ', Date.now());
  next();
});

app.use('/request-type', (req, res, next) => {
  console.log('Request type: ', req.method);
  next();
});

app.use('/public', express.static('public'));
app.use('/public', serveIndex('public'));

app.get('/', (req, res) => {
  res.send('Successful response.');
});

app.listen(3000, () => console.log('Example app is listening on port 3000.'));

```


Now, restart your server and navigate to localhost:3000/public. You will be presented with a listing of all your files!


# Conclusion


In this article, you installed and used Express to build a web server. You also used built-in and third-party middleware functions.


Continue your learning with How To Use the req Object in Express, How To Use the res Object in Express, and How To Define Routes and HTTP Request Methods in Express.


