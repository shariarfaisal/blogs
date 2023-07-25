# How To Upload Images with a Node js Backend in Multer and Express

```Node.js```

## Introduction


While you may upload images on the frontend, you would need to implement an API and database on the backend to receive them. With Multer and Express, a Node.js framework, you can establish file and image uploads in one setting.


In this article, you will learn how to upload images with a Node.js backend using Multer and Express.


# Prerequisites


- 
An understanding of Node.js is recommended. To learn more about Node.js, check out our How To Code in Node.js series.

- 
A general understanding of HTTP request methods in Express is suggested. To learn more about HTTP request methods, check out our How To Define Routes and HTTP Request Methods in Express tutorial.


# Step 1 — Setting Up the Project


As Express is a Node.js framework, ensure that you have Node.js installed from Node.js prior to following the next steps. Run the following in your terminal:


Create a new directory named node-multer-express for your project:


```
mkdir node-multer-express


```


Change into the new directory:


```
cd node-multer-express


```


Initialize a new Node.js project with defaults. This will include your package.json file to access your dependencies:


```
npm init


```


Create your entry file, index.js. This is where you will handle your Express logic:


```
touch index.js


```


Install Multer, Express, and morgan as dependencies:


```
npm install multer express morgan --save


```


Multer is your image upload library and manages accessing form data from an Express request. morgan is Express middleware for logging network requests.


## Applying Multer in Your Project


To set up your Multer library, use the .diskStorage() method to tell Express where to store files to the disk. In your index.js file, require Multer and declare a storage variable and assign its value the invocation of the .diskStorage() method:


index.js
```
const multer = require('multer');

const storage = multer.diskStorage({
  destination: function(req, file, callback) {
    callback(null, '/src/my-images');
  },
  filename: function (req, file, callback) {
    callback(null, file.fieldname);
  }
});

```


The destination property on the diskStorage() method determines which directory the files will store. Here, the files will store in the directory, my-images. If you’ve not applied a destination, the operating system will default to a directory for temporary files.


The property filename indicates what to name your files. If you do not set a filename, Multer will return a randomly generated name for your files.



Note: Multer does not add extensions to file names, and it’s recommended to return a filename complete with a file extension.

With your Multer setup complete, let’s combine it within your Express server.


# Step 2 — Handling the Express Server


Your Express server is where you handle the logic for HTTP request methods, the request and response lifecycle methods, and where you can implement the dependencies Multer and morgan for file and image transfer.


In your index.js file, declare an app variable and assign its value an Express instance. Require in Multer and morgan, and declare an upload variable to store a Multer instance:


index.js
```
import morgan from 'morgan';
import express from 'express';
const app = express();
const multer = require('multer');
const upload = multer({dest: 'uploads/'});

app.use(express.json());
app.use(express.urlencoded({extended: true}));
app.use(morgan('dev'));

app.use(express.static(__dirname, 'public'));

```


You’ll operate the Express middleware, .use(), to pass in the .json() middleware to parse your incoming responses as a JSON object. As well, .use() accepts an invocation of morgan and the argument 'dev'. This tells Express to use morgan’s development environment to alert you of response status. To create static files, transfer in the Express middleware .static() to .use() and define the directory containing your images as an argument.


Once you’ve set your global variables, set a POST request that accepts an anonymous route, and the req and response callback to receive new files and images:


```
app.post('/', upload.single('file'), (req, res) => {
  if (!req.file) {
    console.log("No file received");
    return res.send({
      success: false
    });

  } else {
    console.log('file received');
    return res.send({
      success: true
    })
  }
});

```


When the anonymous route receives a file or image, Multer will save them to your specified directory. The second argument in your POST request, upload.single() is a built-in Multer method to save a file with a fieldname property and store it in the Express req.file object. The fieldname property is defined on your Multer .diskStorage() method.


Should you integrate a database, you can require the filename in your index.js file:


index.js
```
const host = req.host;
const filePath = req.protocol + "://" + host + '/' + req.file.path;

```


Save the variable filePath to the database, and operate your database with the incoming file names.


# Conclusion


Express provides you a process to save and store incoming files and images into your server. The middleware dependency Multer streamlines your form data to handle multiple file uploads.


If you’d like to learn more about Node.js, take a look at our How To Code in React.js series, or check out our Node.js topic page for exercises and programming projects.


