# How To Create a Custom Middleware in Express js

```Node.js```

## Introduction


Middleware is a function that executes the lifecycle method to an Express server, and utilizes the request and response cycles. Express.js offers built-in middleware, and allows you to produce custom versions for precise functionality such as preventing a user from performing a certain operation or logging the path for an incoming request to your application.


In this article, you will learn about how to create a custom middleware in Express.js.


# Prerequisites


To follow along with this article, you will need:


- A general understanding of Node.js is suggested, but not required. To learn more about Node.js, check out our How To Code in Node.js series.
- A general understanding of the request and response cycles. Check out our tutorials on How To Use the req Object in Express and How To Use the res Object in Express.
- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.

# Analyzing an Express Middleware


All middleware functions in Express.js accept three arguments following the request, response, and next lifecycle methods. In your index.js file, define a function with the three lifecycle methods as arguments:


index.js
```
function myCustomMiddleware(req, res, next) {
  // ...
}

```


The first argument, req, is shorthand for the request object with built-in properties to access data from the client side and facilitate HTTP requests. The res argument is the response object with built-in methods to send data to the client side through HTTP requests. The argument, next, is a function that tells Express.js to continue on to the following middleware you have configured for your application.


Middleware has the ability to modify the req and res objects, run any code you wish, end the request and response cycle, and move onto the following functions.


Note the order of your middleware, as invoking the next() function is required in each preceding middleware.


Now that you’ve reviewed the three arguments that build a middleware, let’s look at how to assemble a custom middleware.


# Using the req Object


To identify the currently logged in user, you can construct a custom middleware that can fetch the user through authentication steps. In your setCurrentUser.js file, define a function that accepts the three lifecycle methods as arguments:


middleware/setCurrentUser.js
```
// Require in logic from your authentication controller
const getUserFromToken = require("../getUserFromToken");

module.exports = function setCurrentUser(req, res, next) {
  const token = req.header("authorization");

  // look up the user based on the token
  const user = getUserFromToken(token).then(user => {
    // append the user object the the request object
    req.user = user;

    // call next middleware in the stack
    next();
  });
};

```


Within your setCurrentUser() function, the req object applies the built-in .header() method to return the access token from a user. Using the authentication controller method, getUserFromToken(), your req.header() logic passes in as an argument to look up the user based on their token. You can also use the req object to define a custom property, .user to store the user’s information. Once your middleware is complete, export the file.


You can enable your custom middleware in your Express server by applying the built-in Express.js middleware, .use().


In your server.js file, instantiate an instance of Express and require in your setCurrentUser() custom middleware:


server.js
```
const express = require('express');

const setCurrentUser = require('./middleware/setCurrentUser.js');

const app = express();

app.use(setCurrentUser);

// ...

```


The app.use() middleware accepts your custom middleware as an argument, and authorizes your logic in your Express server.


# Applying the res Object


You can also create a custom middleware to handle functionality on your response object, such as designing a new header.


In your addNewHeader.js file, define a function and utilize the .setHeader() method on the res object:


middleware/addNewHeader.js
```
module.exports = function addNewHeader(req, res, next) {
  res.setHeader("X-New-Policy", "Success");
  next();
};

```


Here, the .setHeader() method will apply the new header, Success, on each function call. The next() method will tell Express.js to move on to following middleware once the execution completes.


# Ending the Request and Response Cycle


Express.js also allows you to end the request and response cycle in your custom middleware. A common use case for a custom middleware is to validate a user’s data set on your req object.


In your isLoggedIn.js file, define a function and set a conditional to check if a user’s data exists on the req object:


middleware/isLoggedIn.js
```
module.exports = function isLoggedIn(req, res, next) {
  if (req.user) {
    next();
  } else {
    // return unauthorized
    res.send(401, "Unauthorized");
  }
};

```


If the user’s data exists in the req object, your custom middleware will move on to following functions. If a particular user’s data is not in the object, the .send() method on the res object will forward the error status code 401 and a message to the client side.


Once you’ve set your middleware, export the file and navigate to your Express.js server. In your server.js file, require in and insert your custom middleware as an argument in a GET request to authenticate a user through a single route:


server.js
```
const express = require("express");

const setCurrentUser = require("./middleware/setCurrentUser.js");
const isLoggedIn = require("./middleware/isLoggedIn.js");

const app = express();

app.use(setCurrentUser);

app.get("/users", isLoggedIn, function(req, res) {
  // ...
});

```


The route /users handles the logic within your isLoggedIn custom middleware. Based on the order of your Express server, the route can also access the setCurrentUser middleware as it is defined before your GET request.


# Conclusion


Express.js provides you the ability to customize your middleware outside of the built-in methods to parse through a user authentication and their data.


For further reading on writing custom Express.js middleware visit the official documentation on the Express.js site.


