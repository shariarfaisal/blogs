# How To Serve Static Files in Express

```Node.js```

## Introduction


In this article, you will learn how to serve static files in Express. A Node.js framework, Express facilitates data in a server and includes rendering your static files on the client-side such as images, HTML, CSS, and JavaScript.


If you’re new to Express, check out our Introduction to Express to get caught up on the basics.


# Prerequisites


To complete this tutorial, you will need the following:


- An understanding of Node.js is suggested but not required. If you’d like to learn more about Node.js, check out the How To Code in Node.js series.
- As Express is a Node.js framework, ensure you have Node.js installed from Node.js prior to following the next steps.

# Step 1 — Setting up Express


To begin, run the following in your terminal:


Create a new directory for your project named express-static-file-tutorial:


```
mkdir express-static-file-tutorial


```


Change into your new directory:


```
cd express-static-file-tutorial


```


Initialize a new Node project with defaults. This will set a package.json file to access your dependencies:


```
npm init -y


```


Create your entry file, index.js. This is where you will store your Express server:


```
touch index.js


```


Install Express as a dependency:


```
npm install express --save


```


Within your package.json, update your start script to include node and your index.js file.


package.json
```
{
  "name": "express-static-file-tutorial",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "keywords": [],
  "author": "Paul Halliday",
  "license": "MIT"
}

```


This will allow you to use the npm start command in your terminal to launch your Express server.


# Step 2 — Structuring Your Files


To store your files on the client-side, create a public directory and include an index.html file along with an image. Your file structure will look like this:


```
express-static-file-tutorial
  |- index.js
  |- public
    |- shark.png
    |- index.html

```


Now that your files are set up let’s begin your Express server.


# Step 3 — Creating Your Express Server


In your index.js file, require in an Express instance and implement a GET request:


index.js
```
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
    res.send('Hello World!');
});

app.listen(PORT, () => console.log(`Server listening on port: ${PORT}`));

```


Now let’s tell Express to handle your static files.


# Step 4 — Serving Your Static Files


Express provides a built-in method to serve your static files:


```
app.use(express.static('public'));

```


When you call app.use(), you’re telling Express to use a piece of middleware. Middleware is a function that Express passes requests through before sending them to your routing functions, such as your app.get('/') route. express.static() finds and returns the static files requested. The argument you pass into express.static() is the name of the directory you want Express to serve files. Here, the public directory.


In index.js, serve your static files below your PORT variable. Pass in your public directory as the argument:


index.js
```
const express = require('express');
const app = express();
const PORT = 3000;

app.use(express.static('public'));

app.get('/', (req, res) => {
    res.send('Hello World!');
});

app.listen(PORT, () => console.log(`Server listening on port: ${PORT}`));

```


With your Express server set, let’s focus on the client-side.


# Step 5 — Building Your Web Page


Navigate to your index.html file in the public directory. Populate the file with body and image elements:


```
[label index.html] 
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
    <img src="shark.png" alt="shark">
  </body>
</html>

```


Notice the image element source to shark.png. Since you’ve served the public directory through Express, you can add the file name as your image source’s value.


# Step 6 — Running Your Project


In your terminal, launch your Express project:


```
npm start


```


```
Server listening on port: 3000

```


Open your web browser, and navigate to http://localhost:3000. You will see your project:





# Conclusion


Express offers a built-in middleware to serve your static files and modularizes content within a client-side directory in one line of code.


