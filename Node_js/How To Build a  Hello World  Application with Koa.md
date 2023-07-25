# How To Build a  Hello World  Application with Koa

```Node.js```

## Introduction


Koa is a new web framework created by the team behind Express. It aims to be a modern and more minimalist version of Express.


Some of its characteristics are its support and reliance on new JavaScript features such as generators and async/await. Koa also does not ship with any middleware though it can be extended using custom and existing plugins.


In this article, you will learn more about the Koa framework and build an app to get familiar with its functionality and philosophy.


# Prerequisites


If you would like to follow along with this article, you will need:


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.
- You also need to have a working knowledge of JavaScript and ES6 syntax.


Note: This tutorial has been revised from Koa 1.0 to Koa 2.0. Refer to the migration documentation for updating your 1.0 implementations.

This tutorial was verified with Node v15.14.0, npm v7.10.0, koa v2.13.1, @koa/router v10.0.0, and koa-ejs v4.3.0.


# Step 1 — Setting Up the Project


To begin, create a new directory for your project. This can be done by copying and running the command below in your terminal:


```
mkdir koa-example


```



Note: You can give your project any name, but this article will be using koa-example as the project name and directory.

At this point, you have created your project directory koa-example. Navigate to the newly created project directory.


```
cd koa-example


```


Then, initialize your Node project from inside the directory.


```
npm init -y


```


After running the npm init command, you will have a package.json file with the default configuration.


Next, run this command to install Koa:


```
npm install koa@2.13.1


```


Your application is now ready to use Koa.


# Step 2 — Creating a Koa Server


First, create the index.js file. Then, using your code editor of choice, open the index.js file and add the following lines of code:


index.js
```
'use strict';

const Koa = require('koa');
const app = new Koa();

app.use(ctx => {
  ctx.body = 'Hello World';
});

app.listen(1234);

```


In the code above, you created a Koa application that runs on port 1234. You can run the application using the command:


```
node index.js


```


And visit the application on http://localhost:1234.


# Step 3 — Adding Routing and View Rendering


As mentioned earlier, Koa.js does not ship with any contained middleware and unlike its predecessor, Express, it does not handle routing by default.


In order to implement routes in your Koa app, you will install a middleware library for routing in Koa, Koa Router.


Open your terminal window and run the following command:


```
npm install @koa/router@10.0.0


```



Note: Previously koa-router was the recommended package, but the @koa/router is now the officially supported package.

To make use of the router in your application, amend your index.js file:


index.js
```
'use strict';

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

router.get('koa-example', '/', (ctx) => {
  ctx.body = 'Hello World';
});

app
  .use(router.routes())
  .use(router.allowedMethods());

app.listen(1234);

```


This code defines a route on the base URL of your application (http://localhost:1234) and registers this route to your Koa application.


For more information on route definition in Koa.js applications, visit the Koa Router library documentation.


As previously established, Koa comes as a minimalistic framework, therefore, to implement view rendering with a template engine you will have to install a middleware library. There are several libraries to choose from but in this article, you will use Koa ejs.


Open your terminal window and run the following command:


```
npm install koa-ejs@4.3.0


```


Next, amend your index.js file to register your templating with the snippet below:


index.js
```
'use strict';

const Koa = require('koa');
const Router = require('@koa/router');
const render = require('koa-ejs');
const path = require('path');

const app = new Koa();
const router = new Router();

render(app, {
  root: path.join(__dirname, 'views'),
  layout: 'index',
  viewExt: 'html',
  cache: false,
  debug: true
});

router.get('koa-example', '/', (ctx) => {
  ctx.body = 'Hello World';
});

app
  .use(router.routes())
  .use(router.allowedMethods());

app.listen(1234);

```


In your template registering, you defined the root directory of your view files, the extension of the view files, and the base view file (which other views extend).


Now that you have registered your template middleware, amend your route definition to render a template file:


index.js
```
// ...

router.get('koa-example', '/', (ctx) => {
  let koalaFacts = [];

  koalaFacts.push({
    meta_name: 'Color',
    meta_value: 'Black and white'
  });

  koalaFacts.push({
    meta_name: 'Native Country',
    meta_value: 'Australia'
  });

  koalaFacts.push({
    meta_name: 'Animal Classification',
    meta_value: 'Mammal'
  });

  koalaFacts.push({
    meta_name: 'Life Span',
    meta_value: '13 - 18 years'
  });

  koalaFacts.push({
    meta_name: 'Are they bears?',
    meta_value: 'No'
  });

  return ctx.render('index', {
    attributes: koalaFacts
  });
})

// ...

```


Your base route renders the index.html file found in the views directory.


Now, create this directory and file. The open index.html and add the following lines of code:


views/index.html
```
<h2>Koala - a directory Koala of attributes</h2>
<ul class="list-group">
  <% attributes.forEach( function(attribute) { %>
    <li class="list-group-item">
      <%= attribute.meta_name %> - <%= attribute.meta_value %>
    </li>
  <% }) %>
</ul>

```


Now, when running the application and observing it in a web browser will display the following:


```
OutputKoala - a directory Koala of attributes
Color - Black and white
Native Country - Australia
Animal Classification - Mammal
Life Span - 13 - 18 years
Are they bears? - No

```


For more options with using the koa-ejs template middleware, visit the library documentation.


# Step 4 — Handling Errors and Responses


Koa handles errors by defining an error middleware early in your entry point file. The error middleware must be defined early because only errors from middleware defined after the error middleware will be caught.


Using your index.js file as an example, make the following changes to the code:


index.js
```
'use strict';

const Koa = require('koa');
const Router = require('@koa/router');
const render = require('koa-ejs');
const path = require('path');

const app = new Koa();
const router = new Router();

app.use(async (ctx, next) => {
  try {
    await next()
  } catch(err) {
    console.log(err.status)
    ctx.status = err.status || 500;
    ctx.body = err.message;
  }
});

// ...

```


This block of code will catch any error thrown during the execution of your application.


You can test this by throwing an error in the function body of the route you defined:


index.js
```
// ...

router.get('error', '/error', (ctx) => {
  ctx.throw(500, 'internal server error');
});

app
  .use(router.routes())
  .use(router.allowedMethods());

app.listen(1234);

```


Now, when running the application and observing /error in a web browser will display the following:


```
Outputinternal server error

```


The Koa response object is usually embedded in its context object. Using route definition, let’s show an example of setting responses:


index.js
```
// ...

router.get('status', '/status', (ctx) => {
  ctx.status = 200;
  ctx.body   = 'ok';
})

app
  .use(router.routes())
  .use(router.allowedMethods());

app.listen(1234);

```


Now, when running the application and observing /status in a web browser will display the following:


```
Outputok

```


Your application now handles errors and responses.


# Conclusion


In this article, you had a brief introduction to Koa and how to implement some common functionalities in a Koa project. Koa is a minimalist and flexible framework that can be extended to more functionality than this article has shown. Because of its futuristic similarity to Express, some have even described it as Express 5.0 in spirit.


