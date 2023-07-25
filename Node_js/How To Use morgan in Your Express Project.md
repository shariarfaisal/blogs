# How To Use morgan in Your Express Project

```Node.js```

## Introduction


morgan is a Node.js and Express middleware to log HTTP requests and errors, and simplifies the process. In Node.js and Express, middleware is a function that has access to the request and response lifecycle methods, and the next() method to continue logic in your Express server.


In this article you will explore how to implement morgan in your Express project.


# Prerequisites


To follow along with this article, you will need:


- A general understanding of Node.js is suggested, but not required. To learn more about Node.js, check out our How To Code in Node.js series.
- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.

# Step 1 – Setting Up the Project


As Express.js is a Node.js framework, ensure you have the latest version of Node.js from Node.js before moving forward.


To include morgan in your Express project, you will need to install it as a dependency.


Create a new directory named express-morgan for your project:


```
mkdir express-morgan


```


Change into the new directory:


```
cd express-morgan


```


Initialize a new Node project with defaults. This will include your package.json file to access your dependencies:


```
npm init -y


```


Install morgan as a dependency:


```
npm install morgan --save


```


Create your entry file, index.js. This is where you will handle logic in your Express server:


```
touch index.js


```


Now that you’ve added morgan to your project, let’s include it in your Express server. In your index.js file, instantiate an Express instance and require morgan:


index.js
```
const express = require('express');
const morgan = require('morgan');

const app = express();

app.listen(3000, () => {
    console.debug('App listening on :3000');
});

```


With your Express server now set up, let’s look at using morgan to add request logging.


# Step 2 – Using morgan in Express


To use morgan in your Express server, you can invoke an instance and pass as an argument in the .use() middleware before your HTTP requests. morgan comes with a suite of presets, or predefined format strings, to create a new logger middleware with built-in format and options. The preset tiny provides the minimal output when logging HTTP requests.


In your index.js file, invoke the app.use() Express middleware and pass morgan() as an argument:


index.js
```
const app = express();

app.use(morgan('tiny'));

```


Including the preset tiny as an argument to morgan() will use its built-in method, identify the URL, declare a status, and the request’s response time in milliseconds.


Alternatively, morgan reads presets like tiny in a format string defined below:


```
morgan(':method :url :status :res[content-length] - :response-time ms');

```


This tends to the same functionality contained in the tiny preset in a format that morgan parses. Following the : symbol are morgan functions called tokens. You can use the format string to define tokens create your own custom morgan middleware.


# Step 3 – Creating Your Own Tokens


Tokens in morgan are functions identified following the : symbol. morgan allows you to create your own tokens with the .token() method.


The .token() method accepts a type, or the name of the token as the first argument, following a callback function. morgan will run the callback function each time a log occurs using the token. As a middleware, morgan applies the req and res objects as arguments.


In your index.js file, employ the .token() method, and pass a type as the first argument following an anonymous function:


index.js
```
morgan.token('host', function(req, res) {
    return req.hostname;
});

```


The anonymous callback function will return the hostname on the req object as a new token to use in an HTTP request in your Express server.


# Step 4 – Designing Tokens with Custom Arguments


To denote custom arguments, you can use square brackets to define arguments passed to a token. This will allow your tokens to accept additional arguments. In your index.js file apply a custom argument to the morgan format string in the :param token:


index.js
```
app.use(morgan(':method :host :status :param[id] :res[content-length] - :response-time ms'));

morgan.token('param', function(req, res, param) {
    return req.params[param];
});

```


The custom argument id on the :param token in the morgan invocation will include the ID in the parameter following the .token() method.


# Conclusion


morgan allows flexibility when logging HTTP requests and updates precise status and response time in custom format strings or in presets. For further reading, check out the morgan documentation.


