# An introduction to the hapi Node js Framework

```Node.js```

The great thing about the Node.js ecosystem is the fact that if you’re looking to create an application, there’s most likely a module/framework that can help with that! In this article, we’ll be creating a basic REST API with hapi.js.



You may also be interested in API Development and Routing with Node.js and Express!

Let’s start off by creating a new project and install hapi. Run the following in your terminal to get started:


```
# Create a new directory for the project
$ mkdir hapi-api && cd hapi-api

# Initialise a new Node project
$ npm init -y

# Install hapi.js
$ npm install hapi

# Create a new server file
$ touch index.js

# Install nodemon
$ npm i nodemon -g

# Run server with nodemon
$ nodemon index.js

```


We’re making use of nodemon to start our server in watch mode.


The first thing we need to do is create a server. Thankfully, this is easy with Node.js!


# Creating a server


```
const Hapi = require('hapi');

const server = Hapi.server({
  port: 3000,
  host: 'localhost'
});

const start = async () => {
  await server.start();
};

start();

```


Our server is now waiting on localhost:3000. Next up, we’ll set up routing to respond on the / route.


```
server.route({
  path: '/',
  method: 'GET',
  handler: (request, h) => {
    return 'Hello, hapi!';
  }
});

```


We can check to see whether this returns what we expect either by using curl or a GUI based project such as Postman.


```
$ curl http://localhost:3000/

> Hello, hapi!

```





## Parameters


We’re also able to take this further with the request and h parameters. Let’s add the ability to pass parameters into our URL:


```
server.route({
  path: '/',
  method: 'GET',
  handler: (request, h) => {
    return 'Hello, hapi!';
  }
});

server.route({
  path: '/{id}',
  method: 'GET',
  handler: (request, h) => {
    return `Product ID: ${encodeURIComponent(request.params.id)}`;
  }
});

```


In our small example, we’re imagining that we have an API that returns a particular product. Whenever the user requests http://localhost:3000/123, they’ll get back:


Product ID: 123


This is because the request.params object contains any params that we’ve set-up in our path.


Notice that we surrounded the id inside of two braces: {id}, this tells hapi that we intend for a user to replace that part of the URL as a param.


At the same time, we also kept the original route without the id. This shows that we can have multiple routes that target a similar base pattern and they won’t override one another. Each one gets more specific, and if it doesn’t match a particular route, it’ll look back in the stack until one is matched.



Although we’ve looked at GET in this example, handling other HTTP verbs is done similarly. Check out routing inside of the hapi documentation for more details.

# Plugins


We can also use plugins within hapi. Let’s use the good plugin to increase the power of our logging capabilities. Run the following in your terminal to install the necessary packages:


```
$ npm install good good-console good-squeeze

```


We’ll then need to create an consoleLogging object which can be used to initialize our plugins.


```
const consoleLogging = {
  plugin: require('good'),
  options: {
    ops: {
      interval: 1000
    },
    reporters: {
      consoleReporter: [
        {
          module: 'good-squeeze',
          name: 'Squeeze',
          args: [{ response: '*', log: '*' }]
        },
        { module: 'good-console' },
        'stdout'
      ]
    }
  }
};

```


We can then register this inside of our start function:


```
const start = async () => {
  await server.register([consoleLogging]);

  await server.start();
};

```


This now means that any time we access our API it’ll log an event to the console. We can try this out by navigating to http://localhost:3000/ inside of our browser, or alternatively, we can use curl:


```
$ curl http://localhost:3000/

```


This gives us the following result:


```
(Pauls-MacBook-Pro) [response] http://localhost:3000 : get / {} 200 (16ms)

```


Tada! 🎉 Now we have automatic logging for every action that happens on our hapi server.


# Serving files


How do we serve files? Good question! Let’s make ourselves a new index.html page:


```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>hapi Todo List</title>
  <style>
  body {
    background-color: #6624fb;
    color: white;
  }

  .container {
    display: flex;
    height: 93vh;
    justify-content: center;
    flex-wrap: wrap;
    flex-direction: column;
    align-items: center;
  }

  .completed {
    text-decoration: line-through;
  }

  ul {
    padding: 0px;
    margin: 0px;
  }

  li {
    font-size: 24px;
    list-style:none;
  }
  </style>
</head>
<body>
  <div class="container">
    <h1>Todo List</h1>
    <ul>
      <li class="completed">Learn about Hapi.js</li>
      <li>Read more articles on Alligator.io</li>
      <li>Drink less coffee</li>
    </ul>
  </div>
</body>
</html>

```


We can then serve this from our index route (or another route or your choosing). To do this, we’ll first need to install the inert module which is used to serve static files.


Thankfully, we already learned how to register plugins within our hapi app! Install inert by running the following in your terminal:


```
$ npm install inert

```


Then register inert like so:


```
const start = async () => {
  /** 
    Note: You can also require inert as a variable like so:
    const Inert = require('inert');

    await server.register([Inert]);
  **/
  await server.register([consoleLogging, require('inert')]);

  await server.start();
};

```


When we navigate to /, not only do we get a log inside of our console, but we also get our Todo list:





# Summary


We now know just enough to get up and running with hapi. Stay tuned for further articles which will look at concepts discussed inside of this article in more detail!


You can find the code for this article in this repo


