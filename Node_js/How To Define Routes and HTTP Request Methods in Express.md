# How To Define Routes and HTTP Request Methods in Express

```Node.js```

## Introduction


This article will examine how to handle routes and HTTP request methods within an Express project. Routes handle user navigation to various URLs throughout your application. HTTP, short for Hyper Text Transfer Protocol, communicates and facilitates data from your Express server to the web browser.


You will learn how to define routes and use the HTTP request methods GET, POST, PUT, and DELETE to manipulate data.


# Prerequisites


To complete this tutorial, an understanding of Node.js is helpful but not required. If you’d like to learn more about Node.js, check out the How To Code in Node.js series.


# Setting Up Your Project


As Express is a Node.js framework, ensure that you have Node.js installed from Node.js before following the next steps. Run the following in your terminal:


Create a new directory named node-express-routing for your project:


```
mkdir node-express-routing


```


Change into the new directory:


```
cd node-express-routing


```


Initialize a new Node project with defaults. This will include your package.json file to access your dependencies:


```
npm init -y


```


Create your entry file, index.js. This is where you will handle your routes and HTTP request methods:


```
touch index.js


```


Install both Express and nodemon as dependencies. You will use nodemon to continually restart your Express project whenever the index.js file changes.


```
npm install express --save
npm install nodemon --save-dev


```


Open your package.json file with your preferred text editor and update your start script to include nodemon and your index.js file:


```
[label package.json] 
{
  "name": "node-express-routing",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "nodemon index.js"
  },
  "keywords": [],
  "author": "Paul Halliday",
  "license": "MIT"
}

```


This will allow you to use the npm start command in your terminal to launch your Express server and update changes.


Now that you’ve set up your project and configured nodemon to refresh when it detects changes in your index.js file, you are ready to start your Express server.


# Starting Your Express Server


Your Express server is where you will handle logic to integrate your routes and HTTP request methods. You will set up and run your server to visualize your project in the browser.


To start your Express server, require Express in your index.js file and store an instance into the app variable. Then, declare a PORT variable and specify the address :3000.


index.js
```
const express = require('express');

const app = express();
const PORT = 3000;

app.use(express.json()); 

app.listen(PORT, () => console.log(`Express server currently running on port ${PORT}`));

```


Next, append .listen() to app and insert PORT as the first argument, then a callback function. The Express middleware .listen() creates a local browser from the address in your PORT variable to visualize your changes.


Also include express.json() as the argument to app.use(). This is to parse through incoming data through your HTTP requests. An earlier version relied on the body-parser dependency, but since then Express has included a built-in middleware for parsing data.


Run the following command in your terminal to start the project:


```
npm start


```


Your project is loaded on http://localhost:3000. Navigate to your browser, and notice the following:





This is the beginning of a running Express instance. Let’s work on defining HTTP methods to populate your browser.


## Defining Your HTTP GET Request Method


In order to view your project, you can send information from your Express server through a GET request, an HTTP method.


In index.js, append .get() to your app variable, specify an anonymous route, and include a callback that accesses the request and response arguments:


```
[label index.js] 
app.get('/', (request, response) => {
  response.send('Hello');
});

```


The request argument contains information about the GET request, while response.send() dispatches data to the browser. The data within response.send() can be a string, object, or an array.


Now that you’ve implemented a GET request, let’s look at routes and other HTTP methods.


# Understanding Routes


Create new GET requests in your index.js file, and define the routes '/accounts' and '/accounts/:id'. Declare an accounts array with some mock data:


index.js
```
let accounts = [
  {
    "id": 1,
    "username": "paulhal",
    "role": "admin"
  },
  {
    "id": 2,
    "username": "johndoe",
    "role": "guest"
  },
  {
    "id": 3,
    "username": "sarahjane",
    "role": "guest"
  }
];

app.get('/accounts', (request, response) => {
  response.json(accounts);
});

app.get('/accounts/:id', (request, response) => {
  const accountId = Number(request.params.id);
  const getAccount = accounts.find((account) => account.id === accountId);

  if (!getAccount) {
    response.status(500).send('Account not found.')
  } else {
    response.json(getAccount);
  }
});

```


If you navigate to http://localhost:3000/accounts you’ll receive all of the accounts in your array:


```
Output[
  {
    "id": 1,
    "username": "paulhal",
    "role": "admin"
  },
  {
    "id": 2,
    "username": "johndoe",
    "role": "guest"
  },
  {
    "id": 3,
    "username": "sarahjane",
    "role": "guest"
  }
]

```


You’re also able to filter account ID’s using the /:id endpoint. Express considers the ID in the endpoint /:id as a placeholder for a user parameter, and matches that value.


Once you navigate to http://localhost:3000/accounts/3, you’ll get one account that matches the /:id parameter:


```
Output{
  "id": 3,
  "username": "sarahjane",
  "role": "guest"
}

```


# Designing POST, PUT, and DELETE HTTP Request Methods


HTTP methods provide additional functionality to your data using the POST, PUT, and DELETE requests. The POST request method creates new data in your server, while PUT updates existing information. The DELETE request method removes the specified data.


## POST


To create new data in the accounts array, you can integrate a POST request method.


In index.js, append .post() to your app variable, and include the route'/accounts' as the first argument:


index.js
```
app.post('/accounts', (request, response) => {
  const incomingAccount = request.body;

  accounts.push(incomingAccount);

  response.json(accounts);
})

```


You will push any incoming data from your POST request into the accounts array and send the response as a JSON object.


Your accounts array now holds a new user:


```
Output[
  {
    "id": 1,
    "username": "paulhal",
    "role": "admin"
  },
  {
    "id": 2,
    "username": "johndoe",
    "role": "guest"
  },
  {
    "id": 3,
    "username": "sarahjane",
    "role": "guest"
  },
  {
    "id": 4,
    "username": "davesmith",
    "role": "admin"
  }
]

```


## PUT


You can edit and update a particular account through a PUT request.


In index.js, append .put() to your app variable and include the route '/accounts/:id' as the first argument. You will find the inputted account ID, and set a conditional to update with new data:


index.js
```
app.put('/accounts/:id', (request, response) => {
  const accountId = Number(request.params.id);
  const body = request.body;
  const account = accounts.find((account) => account.id === accountId);
  const index = accounts.indexOf(account);

  if (!account) {
    response.status(500).send('Account not found.');
  } else {
    const updatedAccount = { ...account, ...body };

    accounts[index] = updatedAccount;

    response.send(updatedAccount);
  }
});

```


You’re now able to update data in the accounts array. If a user changes their "role":


```
{
    "role": "guest"
}

```


Notice that "role" updates to guest from admin in http://localhost:3000/accounts/1:


```
Output{
  "id": 1,
  "username": "paulhal",
  "role": "guest"
}

```


## DELETE


You can delete users and data using the DELETE request method.


Within index.js, append .delete() to your app variable, and include '/accounts/:id' as the first argument. You will filter through the accounts array and return the account to delete.


```
[label index.js] 
app.delete('/accounts/:id', (request, response) => {
  const accountId = Number(request.params.id);
  const newAccounts = accounts.filter((account) => account.id != accountId);

  if (!newAccounts) {
    response.status(500).send('Account not found.');
  } else {
    accounts = newAccounts;
    response.send(accounts);
  }
});

```


If you send a DELETE request to http://localhost:3000/accounts/1, this will remove the account with the /:id of 1 from the accounts array.


# Conclusion


Specifying routes and using HTTP request methods aid to generate additional performance when creating, updating, and deleting users and data in your Express server.


