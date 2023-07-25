# How To Add Advanced Photo Uploads in Node and Express

```Node.js``` ```Applications```

## Introduction


At one time or another when building our Node application we have been faced with uploading a photo (usually from a form) to be used as a profile photo for a user in our app. In addition, we usually have to store the photo in the local filesystem (during development) or even in the cloud for easy access. Since this is a very common task, there are lots of tools available which we can leverage to handle the individual parts of the process.


In this tutorial, we will see how to upload a photo and manipulate it (resize, crop, greyscale, etc) before writing it to storage. We will limit ourselves to storing files in the local filesystem for simplicity.


# Prerequisites


We will be using the following packages to build our application:


- express: A very popular Node server.
- lodash: A very popular JavaScript library with lots of utility functions for working with arrays, strings, objects and functional programming.
- multer: A package for extracting files from multipart/form-data requests.
- jimp: An image manipulation package.
- dotenv: A package for adding .env variables to process.env.
- mkdirp: A package for creating nested directory structure.
- concat-stream: A package for creating a writable stream that concatenates all the data from a stream and calls a callback with the result.
- streamifier: A package to convert a Buffer/String into a readable stream.

# Project Goals


We want to take over the uploaded file stream from Multer and then manipulate the stream buffer (image) however we wish using Jimp, before writing the image to storage (local filesystem). This will require us to create a custom storage engine to use with Multer — which we will be doing in this tutorial.


Here is the end result of what we will be building in this tutorial:



    <a href="https://www.youtube.com/watch?v=oedBD8Bnyls" target="_blank">View YouTube video</a>

# Step 1 — Getting Started


We will begin by creating a new Express app using the Express generator. If you don’t have the Express generator already you will need to install it first by running the following command on your command line terminal:


```
npm install express-generator -g


```


Once you have the Express generator, you can now run the following commands to create a new Express app and install the dependencies for Express. We will be using ejs as our view engine:


```
express --view=ejs photo-uploader-app
cd photo-uploader-app
npm install


```


Next, we will install the remaining dependencies we need for our project:


```
npm install --save lodash multer jimp dotenv concat-stream streamifier mkdirp


```


# Step 2 — Configuring the Basics


Before we continue, our app will need some form configuration. We will create a .env file on our project root directory and add some environment variables. The .env file should look like the following snippet.


```
AVATAR_FIELD=avatar
AVATAR_BASE_URL=/uploads/avatars
AVATAR_STORAGE=uploads/avatars

```


Next, we will load our environment variables into process.env using dotenv so that we can access them in our app. To do this, we will add the following line to the app.js file. Ensure you add this line at the point where you are loading the dependencies. It must come before all route imports and before creating the Express app instance.


app.js
```
var dotenv = require('dotenv').config();

```


Now we can access our environment variables using process.env. For example: process.env.AVATAR_STORAGE should contain the value uploads/avatars. We will go ahead to edit our index route file routes/index.js to add some local variables we will be needing in our view. We will add two local variables:


- title: The title of our index page: Upload Avatar
- avatar_field: The name of the input field for our avatar photo. We will be getting this from process.env.AVATAR_FIELD

Modify the GET / route as follows:


routes/index.js
```
router.get('/', function(req, res, next) {
res.render('index', { title: 'Upload Avatar', avatar_field: process.env.AVATAR_FIELD });
});

```


# Step 3 — Preparing the View


Let’s begin by creating the basic markup for our photo upload form by modifying the views/index.ejs file. For the sake of simplicity we will add the styles directly on our view just to give it a slightly nice look. See the following code for the markup of our page.


views/index.ejs
```
<html class="no-js">
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title><%= title %></title>
<style type="text/css">
* {
font: 600 16px system-ui, sans-serif;
}
form {
width: 320px;
margin: 50px auto;
text-align: center;
}
form > legend {
font-size: 36px;
color: #3c5b6d;
padding: 150px 0 20px;
}
form > input[type=file], form > input[type=file]:before {
display: block;
width: 240px;
height: 50px;
margin: 0 auto;
line-height: 50px;
text-align: center;
cursor: pointer;
}
form > input[type=file] {
position: relative;
}
form > input[type=file]:before {
content: 'Choose a Photo';
position: absolute;
top: -2px;
left: -2px;
color: #3c5b6d;
font-size: 18px;
background: #fff;
border-radius: 3px;
border: 2px solid #3c5b6d;
}
form > button[type=submit] {
border-radius: 3px;
font-size: 18px;
display: block;
border: none;
color: #fff;
cursor: pointer;
background: #2a76cd;
width: 240px;
margin: 20px auto;
padding: 15px 20px;
}
</style>
</head>
<body>
<form action="/upload" method="POST" enctype="multipart/form-data">
<legend>Upload Avatar</legend>
<input type="file" name="<%= avatar_field %>">
<button type="submit" class="btn btn-primary">Upload</button>
</form>
</body>
</html>


```


Notice how we have used our local variables on our view to set the title and the name of the avatar input field. You will notice that we are using enctype="multipart/form-data" on our form since we will be uploading a file. You will also see that we have set the form to make a POST request to the /upload route (we will implement later) on submission.


Now let’s start the app for the first time using npm start.


```
npm start


```


If you have been following correctly everything should run without errors. Just visit localhost:3000 on your browser. The page should look like the following screenshot:





# Step 4 — Creating the Multer Storage Engine


So far, trying to upload a photo through our form will result in an error because we’ve not created the handler for the upload request. We are going to implement the /upload route to actually handle the upload and we will be using the Multer package for that. If you are not already familiar with the Multer package you can check the Multer package on Github.


We will have to create a custom storage engine to use with Multer. Let’s create a new folder in our project root named helpers and create a new file AvatarStorage.js inside it for our custom storage engine. The file should contain the following blueprint code snippet:


helpers/AvatarStorage.js
```
// Load dependencies
var _ = require('lodash');
var fs = require('fs');
var path = require('path');
var Jimp = require('jimp');
var crypto = require('crypto');
var mkdirp = require('mkdirp');
var concat = require('concat-stream');
var streamifier = require('streamifier');

// Configure UPLOAD_PATH
// process.env.AVATAR_STORAGE contains uploads/avatars
var UPLOAD_PATH = path.resolve(__dirname, '..', process.env.AVATAR_STORAGE);

// create a multer storage engine
var AvatarStorage = function(options) {

// this serves as a constructor
function AvatarStorage(opts) {}

// this generates a random cryptographic filename
AvatarStorage.prototype._generateRandomFilename = function() {}

// this creates a Writable stream for a filepath
AvatarStorage.prototype._createOutputStream = function(filepath, cb) {}

// this processes the Jimp image buffer
AvatarStorage.prototype._processImage = function(image, cb) {}

// multer requires this for handling the uploaded file
AvatarStorage.prototype._handleFile = function(req, file, cb) {}

// multer requires this for destroying file
AvatarStorage.prototype._removeFile = function(req, file, cb) {}

// create a new instance with the passed options and return it
return new AvatarStorage(options);

};

// export the storage engine
module.exports = AvatarStorage;


```


Let’s begin to add the implementations for the listed functions in our storage engine. We will begin with the constructor function.


```

// this serves as a constructor
function AvatarStorage(opts) {

var baseUrl = process.env.AVATAR_BASE_URL;

var allowedStorageSystems = ['local'];
var allowedOutputFormats = ['jpg', 'png'];

// fallback for the options
var defaultOptions = {
storage: 'local',
output: 'png',
greyscale: false,
quality: 70,
square: true,
threshold: 500,
responsive: false,
};

// extend default options with passed options
var options = (opts && _.isObject(opts)) ? _.pick(opts, _.keys(defaultOptions)) : {};
options = _.extend(defaultOptions, options);

// check the options for correct values and use fallback value where necessary
this.options = _.forIn(options, function(value, key, object) {

switch (key) {

case 'square':
case 'greyscale':
case 'responsive':
object[key] = _.isBoolean(value) ? value : defaultOptions[key];
break;

case 'storage':
value = String(value).toLowerCase();
object[key] = _.includes(allowedStorageSystems, value) ? value : defaultOptions[key];
break;

case 'output':
value = String(value).toLowerCase();
object[key] = _.includes(allowedOutputFormats, value) ? value : defaultOptions[key];
break;

case 'quality':
value = _.isFinite(value) ? value : Number(value);
object[key] = (value && value >= 0 && value <= 100) ? value : defaultOptions[key];
break;

case 'threshold':
value = _.isFinite(value) ? value : Number(value);
object[key] = (value && value >= 0) ? value : defaultOptions[key];
break;

}

});

// set the upload path
this.uploadPath = this.options.responsive ? path.join(UPLOAD_PATH, 'responsive') : UPLOAD_PATH;

// set the upload base url
this.uploadBaseUrl = this.options.responsive ? path.join(baseUrl, 'responsive') : baseUrl;

if (this.options.storage == 'local') {
// if upload path does not exist, create the upload path structure
!fs.existsSync(this.uploadPath) && mkdirp.sync(this.uploadPath);
}

}


```


Here, we defined our constructor function to accept a couple of options. We also added some default (fallback) values for these options in case they are not provided or they are invalid. You can tweak this to contain more options depending on what you want, but for this tutorial we will stick with the following options for our storage engine.


- storage: The storage filesystem. Only allowed value is 'local' for local filesystem. Defaults to 'local'. You can implement other storage filesystems (like Amazon S3) if you wish.
- output: The image output format. Can be 'jpg' or 'png'. Defaults to 'png'.
- greyscale: If set to true, the output image will be greyscale. Defaults to false.
- quality: A number between 0: 100 that determines the quality of the output image. Defaults to 70.
- square: If set to true, the image will be cropped to a square. Defaults to false.
- threshold: A number that restricts the smallest dimension (in px) of the output image. The default value is 500. If the smallest dimension of the image exceeds this number, the image is resized so that the smallest dimension is equal to the threshold.
- responsive: If set to true, three output images of different sizes (lg, md and sm) will be created and stored in their respective folders. Defaults to false.

Let’s implement the methods for creating the random filenames and the output stream for writing to the files:


```

// this generates a random cryptographic filename
AvatarStorage.prototype._generateRandomFilename = function() {
// create pseudo random bytes
var bytes = crypto.pseudoRandomBytes(32);

// create the md5 hash of the random bytes
var checksum = crypto.createHash('MD5').update(bytes).digest('hex');

// return as filename the hash with the output extension
return checksum + '.' + this.options.output;
};

// this creates a Writable stream for a filepath
AvatarStorage.prototype._createOutputStream = function(filepath, cb) {

// create a reference for this to use in local functions
var that = this;

// create a writable stream from the filepath
var output = fs.createWriteStream(filepath);

// set callback fn as handler for the error event
output.on('error', cb);

// set handler for the finish event
output.on('finish', function() {
cb(null, {
destination: that.uploadPath,
baseUrl: that.uploadBaseUrl,
filename: path.basename(filepath),
storage: that.options.storage
});
});

// return the output stream
return output;
};


```


Here, we use crypto to create a random md5 hash to use as filename and appended the output from the options as the file extension. We also defined our helper method to create writable stream from the given filepath and then return the stream. Notice that a callback function is required, since we are using it on the stream event handlers.


Next we will implement the _processImage() method that does the actual image processing. Here is the implementation:


```

// this processes the Jimp image buffer
AvatarStorage.prototype._processImage = function(image, cb) {

// create a reference for this to use in local functions
var that = this;

var batch = [];

// the responsive sizes
var sizes = ['lg', 'md', 'sm'];

var filename = this._generateRandomFilename();

var mime = Jimp.MIME_PNG;

// create a clone of the Jimp image
var clone = image.clone();

// fetch the Jimp image dimensions
var width = clone.bitmap.width;
var height = clone.bitmap.height;
var square = Math.min(width, height);
var threshold = this.options.threshold;

// resolve the Jimp output mime type
switch (this.options.output) {
case 'jpg':
mime = Jimp.MIME_JPEG;
break;
case 'png':
default:
mime = Jimp.MIME_PNG;
break;
}

// auto scale the image dimensions to fit the threshold requirement
if (threshold && square > threshold) {
clone = (square == width) ? clone.resize(threshold, Jimp.AUTO) : clone.resize(Jimp.AUTO, threshold);
}

// crop the image to a square if enabled
if (this.options.square) {

if (threshold) {
square = Math.min(square, threshold);
}

// fetch the new image dimensions and crop
clone = clone.crop((clone.bitmap.width: square) / 2, (clone.bitmap.height: square) / 2, square, square);
}

// convert the image to greyscale if enabled
if (this.options.greyscale) {
clone = clone.greyscale();
}

// set the image output quality
clone = clone.quality(this.options.quality);

if (this.options.responsive) {

// map through the responsive sizes and push them to the batch
batch = _.map(sizes, function(size) {

var outputStream;

var image = null;
var filepath = filename.split('.');

// create the complete filepath and create a writable stream for it
filepath = filepath[0] + '_' + size + '.' + filepath[1];
filepath = path.join(that.uploadPath, filepath);
outputStream = that._createOutputStream(filepath, cb);

// scale the image based on the size
switch (size) {
case 'sm':
image = clone.clone().scale(0.3);
break;
case 'md':
image = clone.clone().scale(0.7);
break;
case 'lg':
image = clone.clone();
break;
}

// return an object of the stream and the Jimp image
return {
stream: outputStream,
image: image
};
});

} else {

// push an object of the writable stream and Jimp image to the batch
batch.push({
stream: that._createOutputStream(path.join(that.uploadPath, filename), cb),
image: clone
});

}

// process the batch sequence
_.each(batch, function(current) {
// get the buffer of the Jimp image using the output mime type
current.image.getBuffer(mime, function(err, buffer) {
if (that.options.storage == 'local') {
// create a read stream from the buffer and pipe it to the output stream
streamifier.createReadStream(buffer).pipe(current.stream);
}
});
});

};


```


A lot is going on in this method but here is a summary of what it is doing:


- Generates a random filename, resolves the Jimp output image mime type and gets the image dimensions.
- Resize the image if required, based on the threshold requirements to ensure that the smallest dimension does not exceed the threshold.
- Crop the image to a square if enabled in the options.
- Convert the image to greyscale if enabled in the options.
- Set the image output quality from the options.
- If responsive is enabled, the image is cloned and scaled for each of the responsive sizes (lg, md and sm) and then an output stream is created using the _createOutputStream() method for each image file of the respective sizes. The filename for each size takes the format [random_filename_hash]_[size].[output_extension]. Then the image clone and the stream are put in a batch for processing.
- If responsive is disabled, then only the current image and an output stream for it is put in a batch for processing.
- Finally, each item in the batch is processed by converting the Jimp image buffer into a readable stream using streamifier and then piping the readable stream to the output stream.

Now we will implement the remaining methods and we will be done with our storage engine.


```

// multer requires this for handling the uploaded file
AvatarStorage.prototype._handleFile = function(req, file, cb) {

// create a reference for this to use in local functions
var that = this;

// create a writable stream using concat-stream that will
// concatenate all the buffers written to it and pass the
// complete buffer to a callback fn
var fileManipulate = concat(function(imageData) {

// read the image buffer with Jimp
// it returns a promise
Jimp.read(imageData)
.then(function(image) {
// process the Jimp image buffer
that._processImage(image, cb);
})
.catch(cb);
});

// write the uploaded file buffer to the fileManipulate stream
file.stream.pipe(fileManipulate);

};

// multer requires this for destroying file
AvatarStorage.prototype._removeFile = function(req, file, cb) {

var matches, pathsplit;
var filename = file.filename;
var _path = path.join(this.uploadPath, filename);
var paths = [];

// delete the file properties
delete file.filename;
delete file.destination;
delete file.baseUrl;
delete file.storage;

// create paths for responsive images
if (this.options.responsive) {
pathsplit = _path.split('/');
matches = pathsplit.pop().match(/^(.+?)_.+?\.(.+)$/i);

if (matches) {
paths = _.map(['lg', 'md', 'sm'], function(size) {
return pathsplit.join('/') + '/' + (matches[1] + '_' + size + '.' + matches[2]);
});
}
} else {
paths = [_path];
}

// delete the files from the filesystem
_.each(paths, function(_path) {
fs.unlink(_path, cb);
});

};


```


Our storage engine is now ready for use with Multer.


# Step 5 — Implementing the POST /upload Route


Before we define the route, we will need to setup Multer for use in our route. Let’s go ahead to edit the routes/index.js file to add the following:


routes/index.js
```

var express = require('express');
var router = express.Router();

/**
 * CODE ADDITION
 * 
 * The following code is added to import additional dependencies
 * and setup Multer for use with the /upload route.
 */

// import multer and the AvatarStorage engine
var _ = require('lodash');
var path = require('path');
var multer = require('multer');
var AvatarStorage = require('../helpers/AvatarStorage');

// setup a new instance of the AvatarStorage engine 
var storage = AvatarStorage({
square: true,
responsive: true,
greyscale: true,
quality: 90
});

var limits = {
files: 1, // allow only 1 file per request
fileSize: 1024 * 1024, // 1 MB (max file size)
};

var fileFilter = function(req, file, cb) {
// supported image file mimetypes
var allowedMimes = ['image/jpeg', 'image/pjpeg', 'image/png', 'image/gif'];

if (_.includes(allowedMimes, file.mimetype)) {
// allow supported image files
cb(null, true);
} else {
// throw error for invalid files
cb(new Error('Invalid file type. Only jpg, png and gif image files are allowed.'));
}
};

// setup multer
var upload = multer({
storage: storage,
limits: limits,
fileFilter: fileFilter
});

/* CODE ADDITION ENDS HERE */


```


Here, we are enabling square cropping, responsive images and setting the threshold for our storage engine. We also add limits to our Multer configuration to ensure that the maximum file size is 1 MB and to ensure that non-image files are not uploaded.


Now let’s add the POST /upload route as follows:


```
/* routes/index.js */

/**
 * CODE ADDITION
 * 
 * The following code is added to configure the POST /upload route
 * to upload files using the already defined Multer configuration
 */

router.post('/upload', upload.single(process.env.AVATAR_FIELD), function(req, res, next) {

var files;
var file = req.file.filename;
var matches = file.match(/^(.+?)_.+?\.(.+)$/i);

if (matches) {
files = _.map(['lg', 'md', 'sm'], function(size) {
return matches[1] + '_' + size + '.' + matches[2];
});
} else {
files = [file];
}

files = _.map(files, function(file) {
var port = req.app.get('port');
var base = req.protocol + '://' + req.hostname + (port ? ':' + port : '');
var url = path.join(req.file.baseUrl, file).replace(/[\\\/]+/g, '/').replace(/^[\/]+/g, '');

return (req.file.storage == 'local' ? base : '') + '/' + url;
});

res.json({
images: files
});

});

/* CODE ADDITION ENDS HERE */


```


Notice how we passed the Multer upload middleware before our route handler. The single() method allows us to upload only one file that will be stored in req.file. It takes as first parameter, the name of the file input field which we access from process.env.AVATAR_FIELD.


Now let’s start the app again using npm start.


```
npm start


```


visit localhost:3000 on your browser and try to upload a photo. Here is a sample screenshot I got from testing the upload route on Postman using our current configuration options:





You can tweak the configuration options of the storage engine in our Multer setup to get different results.


# Conclusion


In this tutorial, we have been able to create a custom storage engine for use with Multer which manipulates uploaded images using Jimp and then writes them to storage. For a complete code sample of this tutorial, checkout the advanced-multer-node-sourcecode repository on Github.


