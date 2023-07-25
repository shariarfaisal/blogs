# How To Deliver HTML Files with Express

```Node.js```

## Introduction


In Node.js and Express applications, res.sendFile() can be used to deliver files. Delivering HTML files using Express can be useful when you need a solution for serving static pages.



Note: Prior to Express 4.8.0, res.sendfile() was supported. This lowercase version of res.sendFile() has since been deprecated.

In this article, you will learn how to use res.sendFile().


# Prerequisites


To complete this tutorial, you will need:


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.

This tutorial was verified with Node v16.0.0, npm v7.11.1, and express v4.17.1.


# Step 1 – Setting Up the Project


First, open your terminal window and create a new project directory:


```
mkdir express-sendfile-example


```


Then, navigate to the newly created directory:


```
cd express-sendfile-example


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

// sendFile will go here

app.listen(port);
console.log('Server started at http://localhost:' + port);

```


Revisit your terminal window and run your application:


```
node server.js


```


After verifying your project is working as expected, you can use res.sendFile().


# Step 2 – Using res.sendFile()


Revisit server.js with your code editor and add path, .get() and res.sendFile():


server.js
```
const express = require('express');
const path = require('path');

const app = express();
const port = process.env.PORT || 8080;

// sendFile will go here
app.get('/', function(req, res) {
  res.sendFile(path.join(__dirname, '/index.html'));
});

app.listen(port);
console.log('Server started at http://localhost:' + port);

```


When a request is made to the server, an index.html file is served.


Create a new index.html file and open it with your code editor:


index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sample Site</title>
  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
  <style>
    body { padding-top: 50px; }
  </style>
</head>
<body>

  <div class="container">
    <div class="jumbotron">
      <h1>res.sendFile() Works!</h1>
    </div>
  </div>
    
</body>
</html>

```


This code will display the message: res.sendFile() Works!.



Note: This tutorial makes use of BootstrapCDN for styling, but it is not required.

Save your changes. Then, open your terminal window again and re-run the server.


```
node server.js


```


With the server running, visit http://localhost:8080 in a web browser:





Your application now uses res.sendFile() to serve HTML files.


# Conclusion


In this article, you learned how to use res.sendFile().


Continue your learning with Learn to Use the Express 4.0 Router and How To Retrieve URL and POST Parameters with Express.


