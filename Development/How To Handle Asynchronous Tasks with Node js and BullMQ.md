# How To Handle Asynchronous Tasks with Node js and BullMQ

```Development``` ```JavaScript``` ```Node.js``` ```Redis```

The author selected the Society of Women Engineers to receive a donation as part of the Write for DOnations program.


## Introduction


Web applications have request/response cycles. When you visit a URL, the browser sends a request to the server running an app that processes data or runs queries in the database. As this happens, the user is kept waiting until the app returns a response. For some tasks, the user can get a response quickly; for time-intensive tasks, such as processing images, analyzing data, generating reports, or sending emails, these tasks take a long time to finish and can slow down the request/response cycle. For example, suppose you have an application where users upload images. In that case, you might need to resize, compress, or convert the image to another format to preserve your server’s disk space before showing the image to the user. Processing an image is a CPU-intensive task, which can block a Node.js thread until the task is finished. That might take a few seconds or minutes. Users have to wait for the task to finish to get a response from the server.


To avoid slowing down the request/response cyrcle, you can use bullmq, a distributed task (job) queue that allows you to offload time-consuming tasks from your Node.js app to bullmq, freeing up the request/response cycle. This tool enables your app to send responses to the user quickly while bullmq executes the tasks asynchronously in the background and independently from your app. To keep track of jobs, bullmq uses Redis to store a short description of each job in a queue. A bullmq worker then dequeues and executes each job in the queue, marking it complete once done.


In this article, you will use bullmq to offload a time-consuming task into the background, which will enable an application to respond quickly to users. First, you will create an app with a time-consuming task without using bullmq. Then, you will use bullmq to execute the task asynchronously. Finally, you will install a visual dashboard to manage bullmq jobs in a Redis queue.


# Prerequisites


To follow this tutorial, you will need the following:


- 
Node.js development environment set up. For Ubuntu 22.04, follow our tutorial on How To Install Node.js on Ubuntu 22.04. For other systems, see How to Install Node.js and Create a Local Development Environment.

- 
Redis installed on your system. On Ubuntu 22, follow Steps 1 through 3 in our tutorial on How To Install and Secure Redis on Ubuntu 22.04. For other systems, see our tutorial on How To Install and Secure Redis.

- 
Familiarity with promises and async/await functions, which you can develop in our tutorial Understanding the Event Loop, Callbacks, Promises, and Async/Await in JavaScript.

- 
Basic knowledge of how to use Express. See our tutorial on How To Get Started with Node.js and Express.

- 
Familiarity with Embedded JavaScript (EJS). Check out our tutorial on How To Use EJS to Template Your Node Application for more details.

- 
Basic understanding of how to process images with sharp, which you can learn in our tutorial on How To Process Images in Node.js with Sharp.


# Step 1 — Setting Up the Project Directory


In this step, you will create a directory and install the necessary dependencies for your application. The application you’ll build in this tutorial will allow users to upload an image, which is then processed using the sharp package. Image processing is time-intensive and can slow the request/response cycle, making the task a good candidate for bullmq to offload into the background. The technique you will use to offload the task will also work for other time-intensive tasks.


To begin, create a directory called image_processor and navigate into the directory:


```
mkdir image_processor && cd image_processor


```


Then, initialize the directory as an npm package:


```
npm init -y


```


The command creates a package.json file. The -y option tells npm to accept all the defaults.


Upon running the command, your output will match the following:


```
OutputWrote to /home/sammy/image_processor/package.json:

{
  "name": "image_processor",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```


The output confirms that the package.json file has been created. Important properties include the name of your app (name), your application version number (version), and the starting point of your project (main). If you want to learn more about the other properties, you can review npm’s package.json documentation.


The application you will build in this tutorial will require the following dependencies:


- express: a web framework for building web apps.
- express-fileupload: a middleware that allows your forms to upload files.
- sharp: an image processing library.
- ejs: a template language that allows you to generate HTML markup with Node.js.
- bullmq: a distributed task queue.
- bull-board: a dashboard that builds upon bullmq and displays the status of the jobs with a nice User Interface(UI).

To install all these dependencies, run the following command:


```
npm install express express-fileupload sharp ejs bullmq @bull-board/express


```


In addition to the dependencies you installed, you will also use the following image later in this tutorial:





Use curl to download the image to the location of your choice on your local computer


```
curl -O https://deved-images.nyc3.digitaloceanspaces.com/CART-68886/underwater.png


```


You have the necessary dependencies to build a Node.js app that does not have bullmq, which you will do next.


# Step 2 — Implementing a Time-Intensive Task Without bullmq


In this step, you will build an application with Express that allows users to upload images. The app will start a time-intensive task using sharp to resize the image into multiple sizes, which are then displayed to the user after a response is sent. This step will help you understand how time-intensive tasks affect the request/response cycle.


Using nano, or your preferred text editor, create the index.js file:


```
nano index.js


```


In your index.js file, add the following code to import dependencies:


image_processor/index.js
```
const path = require("path");
const fs = require("fs");
const express = require("express");
const bodyParser = require("body-parser");
const sharp = require("sharp");
const fileUpload = require("express-fileupload");

```


In the first line, you import the path module for computing file paths with Node. In the second line, you import the fs module for interacting with directories. You then import the express web framework. You import the body-parser module to add middleware to parse data in HTTP requests. Following that, you import the sharp module for image processing. Finally, you import express-fileupload for handling uploads from an HTML form.


Next, add the following code to implement middleware in your app:


image_processor/index.js
```
...
const app = express();
app.set("view engine", "ejs");
app.use(bodyParser.json());
app.use(
  bodyParser.urlencoded({
    extended: true,
  })
);

```


First, you set the app variable to an instance of Express. Second, using the app variable, the set() method configures Express to use the ejs template language. You then add the body-parser module middleware with the use() method to transform JSON data in HTTP requests into variables that can be accessed with JavaScript. In the following line, you do the same with URL-encoded input.


Next, add the following lines to add more middleware to handle file uploads and serve static files:


image_processor/index.js
```
...
app.use(fileUpload());
app.use(express.static("public"));

```


You add middleware to parse uploaded files by calling the fileUpload() method, and you set a directory where Express will look at and serve static files, such as images and CSS.


With the middleware set, create a route that displays an HTML form for uploading an image:


image_processor/index.js
```
...
app.get("/", function (req, res) {
  res.render("form");
});

```


Here, you use the get() method of the Express module to specify the / route and the callback that should run when the user visits the homepage or / route. In the callback, you invoke res.render() to render the form.ejs file in the views directory. You have not yet created the form.ejs file or the views directory.


To create it, first, save and close your file. In your terminal, enter the following command to create the views directory in your project root directory:


```
mkdir views


```


Move into the views directory:


```
cd views


```


Create the form.ejs file in your editor:


```
nano form.ejs


```


In your form.ejs file, add the following code to create the form:


image_processor/views/form.ejs
```
<!DOCTYPE html>
<html lang="en">
  <%- include('./head'); %>
  <body>
    <div class="home-wrapper">
      <h1>Image Processor</h1>
      <p>
        Resizes an image to multiple sizes and converts it to a
        <a href="https://en.wikipedia.org/wiki/WebP">webp</a> format.
      </p>
      <form action="/upload" method="POST" enctype="multipart/form-data">
        <input
          type="file"
          name="image"
          placeholder="Select image from your computer"
        />
        <button type="submit">Upload Image</button>
      </form>
    </div>
  </body>
</html>

```


First, you reference the head.ejs file, which you haven’t created yet. The head.ejs file will contain the HTML head element you can reference in other HTML pages.


In the body tag, you create a form with the following attributes:


- action specifies the route where the form data should be sent when the form is submitted.
- method specifies the HTTP method for sending data. The POST method embeds the data in an HTTP request.
- encytype specifies how the form data should be encoded. The value multipart/form-data enables the HTML input elements to upload file data.

In the form element, you create an input tag to upload files. Then you define the button element with the type attribute set to submit, which lets you submit forms.


Once finished, save and close your file.


Next, create a head.ejs file:


```
nano head.ejs


```


In your head.ejs file, add the following code to create the head section of the app:


image_processor/views/head.ejs
```
<head>
  <meta charset="UTF-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Image Processor</title>
  <link rel="stylesheet" href="css/main.css" />
</head>

```


Here, you reference the main.css file, which you will create in the public directory later in this step. That file will contain the styles for this application. For now, you will continue setting up the processes for static assets.


Save and close the file.


To handle data submitted from the form, you must define a post method in Express. To do that, return to the root directory of your project:


```
cd ..


```


Open your index.js file again:


```
nano index.js


```


In your index.js file, add the highlighted lines to define a method for handling form submissions on route /upload:


image_processor/index.js
```
app.get("/", function (req, res) {
  ...
});

app.post("/upload", async function (req, res) {
  const { image } = req.files;

  if (!image) return res.sendStatus(400);

});

```


You use the app variable to call the post() method, which will handle the submitted form on the /upload route. Next, you extract the uploaded image data from the HTTP request into the image variable. After that, you set a response to return a 400 status code if the user does not upload an image.


To set the process for the uploaded image, add the following highlighted code:


image_processor/index.js
```
...
app.post("/upload", async function (req, res) {
  const { image } = req.files;

  if (!image) return res.sendStatus(400);
  const imageName = path.parse(image.name).name;
  const processImage = (size) =>
    sharp(image.data)
      .resize(size, size)
      .webp({ lossless: true })
      .toFile(`./public/images/${imageName}-${size}.webp`);

  sizes = [90, 96, 120, 144, 160, 180, 240, 288, 360, 480, 720, 1440];
  Promise.all(sizes.map(processImage));
});

```


These lines represent how your app will process the image. First, you remove the image extension from the uploaded image and save the name in the imageName variable. Next, you define the processImage() function. This function takes the size parameter, whose value will be used to determine the image dimensions during resizing. In the function, you invoke sharp() with image.data, which is a buffer containing the binary data for the uploaded image. sharp resizes the image according to the value in the size parameter. You use the webp() method from sharp to convert the image to the webp image format. Then, you save the image in the public/images/ directory.


The subsequent list of numbers defines the sizes that will be used to resize the uploaded image. You then use JavaScript’s map() method to invoke processImage() for each element in the sizes array, after which it will return a new array. Every time the map() method calls the processImage() function, it returns a promise to the new array. You use the Promise.all() method to resolve them.


Computer processing speeds vary, as will the size of images a user can upload, which might affect the image processing speed. To delay this code for demonstration purposes, insert the highlighted lines to add a CPU-intensive increment loop and a redirect to a page that will display the resized images with the highlighted lines:


image_processor/index.js
```
...
app.post("/upload", async function (req, res) {
  ...
  let counter = 0;
  for (let i = 0; i < 10_000_000_000; i++) {
    counter++;
  }

  res.redirect("/result");
});

```


The loop will run 10 billion times to increment the counter variable. You invoke the res.redirect() function to redirect the app to the /result route. The route will render an HTML page that will display the images in the public/images directory.


The /result route doesn’t exist yet. To create it, add the highlighted code in your index.js file:


image_processor/index.js
```
...

app.get("/", function (req, res) {
 ...
});

app.get("/result", (req, res) => {
  const imgDirPath = path.join(__dirname, "./public/images");
  let imgFiles = fs.readdirSync(imgDirPath).map((image) => {
    return `images/${image}`;
  });
  res.render("result", { imgFiles });
});

app.post("/upload", async function (req, res) {
  ...
});

```


You define the /result route with the app.get() method. In the function, you define the imgDirPath variable with the full path to the public/images directory. You use the readdirSync() method of the fs module to read all the files in the given directory. From there, you chain the map() method to return a new array with the images paths prefixed with images/.


Finally, you call res.render() to render the result.ejs file, which doesn’t exist yet. You pass the imgFiles variable, which contains an array of all the image’s relative paths, to the result.ejs file.


Save and close your file.


To create the result.ejs file, return to the views directory:


```
cd views


```


Create and open the result.ejs file in your editor:


```
nano result.ejs


```


In your result.ejs file, add the following lines to display images:


image_processor/views/result.ejs
```
<!DOCTYPE html>
<html lang="en">
  <%- include('./head'); %>
  <body>
    <div class="gallery-wrapper">
      <% if (imgFiles.length > 0){%>
      <p>The following are the processed images:</p>
      <ul>
        <% for (let imgFile of imgFiles){ %>
        <li><img src=<%= imgFile %> /></li>
        <% } %>
      </ul>
      <% } else{ %>
      <p>
        The image is being processed. Refresh after a few seconds to view the
        resized images.
      </p>
      <% } %>
    </div>
  </body>
</html>

```


First, you reference the head.ejs file. In the body tag, you check if the imgFiles variable is empty. If it has data, you iterate over each file and create an image for each array element. If imgFiles is empty, you print a message that tells the user to Refresh after a few seconds to view the resized images..


Save and close your file.


Next, return to the root directory and create the public directory that will contain your static assets:


```
cd .. && mkdir public


```


Move into the public directory:


```
cd public


```


Create an images directory that will keep the uploaded images:


```
mkdir images


```


Next, create the css directory and navigate to it:


```
mkdir css && cd css


```


In your editor, create and open the main.css file, which you referenced earlier in the head.ejs file:


```
nano main.css


```


In your main.css file, add the following styles:


image_processor/public/css/main.css
```
body {
  background: #f8f8f8;
}

h1 {
  text-align: center;
}

p {
  margin-bottom: 20px;
}

a:link,
a:visited {
  color: #00bcd4;
}

/** Styles for the "Choose File"  button **/
button[type="submit"] {
  background: none;
  border: 1px solid orange;
  padding: 10px 30px;
  border-radius: 30px;
  transition: all 1s;
}

button[type="submit"]:hover {
  background: orange;
}

/** Styles for the "Upload Image"  button **/
input[type="file"]::file-selector-button {
  border: 2px solid #2196f3;
  padding: 10px 20px;
  border-radius: 0.2em;
  background-color: #2196f3;
}

ul {
  list-style: none;
  padding: 0;
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.home-wrapper {
  max-width: 500px;
  margin: 0 auto;
  padding-top: 100px;
}

.gallery-wrapper {
  max-width: 1200px;
  margin: 0 auto;
}

```


These lines will style elements in the app. Using HTML attributes, you style the Choose File button background with the hex code #2196f3 (a shade of blue) and the Upload Image button border to orange. You also style the elements on the /result route to make them more presentable.


Once finished, save and close your file.


Return to the project root directory:


```
cd ../..


```


Open index.js in your editor:


```
nano index.js


```


In your index.js, add the following code, which will start the server:


image_processor/index.js
```
...
app.listen(3000, function () {
  console.log("Server running on port 3000");
});

```


The complete index.js file will now match the following:


image_processor/index.js
```
const path = require("path");
const fs = require("fs");
const express = require("express");
const bodyParser = require("body-parser");
const sharp = require("sharp");
const fileUpload = require("express-fileupload");

const app = express();
app.set("view engine", "ejs");
app.use(bodyParser.json());
app.use(
  bodyParser.urlencoded({
    extended: true,
  })
);

app.use(fileUpload());

app.use(express.static("public"));

app.get("/", function (req, res) {
  res.render("form");
});

app.get("/result", (req, res) => {
  const imgDirPath = path.join(__dirname, "./public/images");
  let imgFiles = fs.readdirSync(imgDirPath).map((image) => {
    return `images/${image}`;
  });
  res.render("result", { imgFiles });
});

app.post("/upload", async function (req, res) {
  const { image } = req.files;

  if (!image) return res.sendStatus(400);
  const imageName = path.parse(image.name).name;
  const processImage = (size) =>
    sharp(image.data)
      .resize(size, size)
      .webp({ lossless: true })
      .toFile(`./public/images/${imageName}-${size}.webp`);

  sizes = [90, 96, 120, 144, 160, 180, 240, 288, 360, 480, 720, 1440];
  Promise.all(sizes.map(processImage));
  let counter = 0;
  for (let i = 0; i < 10_000_000_000; i++) {
    counter++;
  }

  res.redirect("/result");
});

app.listen(3000, function () {
  console.log("Server running on port 3000");
});

```


Once you are finished making the changes, save and close your file.


Run the app using the node command:


```
node index.js


```


You will receive an output like so:


```
OutputServer running on port 3000

```


This output confirms the server is running without any issues.


Open your preferred browser and visit http://localhost:3000/.



Note: If you are following the tutorial on a remote server, you can access the app in your local browser using port forwarding.
While the Node.js server is running, open another terminal and enter the following command:
ssh -L 3000:localhost:3000  your-non-root-user@yourserver-ip


Once you have connected to the server, run node index.js and then navigate to http://localhost:3000/ on your local machine’s web browser.

When the page loads, it will match the following:





Next, press the Choose File button and select the underwater.png image on your local machine. The display will switch from No file chosen to underwater.png. After that, press the Upload Image button. The app will load for a while as it processes the image and runs the incrementing loop.


Once the task finishes, the /result route will load with the resized images:





You can stop the server now with CTRL+C. Node.js does not automatically reload the server when files are changed, so you will need to stop and restart the server whenever you update the files.


You now know how a time-intensive task can affect an application’s request/response cycle. You will execute the task asynchronously next.


# Step 3 — Executing Time-Intensive Tasks Asynchronously with bullmq


In this step, you will offload a time-intensive task to the background using bullmq. This adjustment will free the request/response cycle and allow your app to respond to users immediately while the image is being processed.


To do that, you need to create a succinct description of the job and add it to a queue with bullmq. A queue is a data structure that works similarly to how a queue works in real life. When people line up to enter a space, the first person on the line will be the first person to enter the space. Anyone who comes later will line up at the end of the line and will enter the space after everyone who precedes them in line until the last person enters the space. With the queue data structure’s First-In, First-Out (FIFO) process, the first item added to the queue is the first item to be removed (dequeue). With bullmq, a producer will add a job in a queue, and a consumer (or worker) will remove a job from the queue and execute it.


The queue in bullmq is in Redis. When you describe a job and add it to the queue, an entry for the job is created in a Redis queue. A job description can be a string or an object with properties that contain minimal data or references to the data that will allow bullmq to execute the job later. Once you define the functionality to add jobs to the queue, you move the time-intensive code into a separate function. Later, bullmq will call this function with the data you stored in the queue when the job is dequeued. Once the task has finished, bullmq will mark it completed, pull another job from the queue, and execute it.


Open index.js in your editor:


```
nano index.js


```


In your index.js file, add the highlighted lines to create a queue in Redis with bullmq:


image_processor/index.js
```
...
const fileUpload = require("express-fileupload");
const { Queue } = require("bullmq");

const redisOptions = { host: "localhost", port: 6379 };

const imageJobQueue = new Queue("imageJobQueue", {
  connection: redisOptions,
});

async function addJob(job) {
  await imageJobQueue.add(job.type, job);
}
...

```


You start by extracting the Queue class from bullmq, which is used to create a queue in Redis. You then set the redisOptions variable to an object with properties that the Queue class instance will use to establish a connection with Redis. You set the host property value to localhost because Redis is running on your local machine.



Note: If Redis were running on a remote server separate from your app, you would update the host property value to the IP address of the remote server. You also set the port property value to 6379, the default port that Redis uses to listen for connections.
If you have set up port forwarding to a remote server running Redis and the app together, you do not need to update the host property, but you will need to use the port forwarding connection every time you log in to your server to run the app.

Next, you set the imageJobQueue variable to an instance of the Queue class, taking the queue’s name as its first argument and an object as a second argument. The object has a connection property with the value set to an object in the redisOptions variable. After instantiating the Queue class, a queue called imageJobQueue will be created in Redis.


Finally, you define the addJob() function that you will use to add a job in the imageJobQueue. The function takes a parameter of job containing the information about the job ⁠(you will call the addJob() function with the data you want to save in a queue). In the function, you invoke the add() method of the imageJobQueue, taking the name of the job as the first argument and the job data as the second argument.


Add the highlighted code to call the addJob() function to add a job in the queue:


image_processor/index.js
```
...
app.post("/upload", async function (req, res) {
  const { image } = req.files;

  if (!image) return res.sendStatus(400);
  const imageName = path.parse(image.name).name;
  ...
  await addJob({
    type: "processUploadedImages",
    image: {
      data: image.data.toString("base64"),
      name: image.name,
    },
  });

  res.redirect("/result");
});
...

```


Here, you call the addJob() function with an object that describes the job. The object has the type attribute with a value of the name of the job. The second property, image, is set to an object containing the image data the user has uploaded. Because the image data in image.data is in a buffer (binary form), you invoke JavaScript’s toString() method to convert it to a string that can be stored in Redis, which will set the data property as a result. The image property is set to the name of the uploaded image (including the image extension).


You have now defined the information needed for bullmq to execute this job later. Depending on your job, you may add more job information or less.



Warning: Since Redis is an in-memory database, avoid storing large amounts of data for jobs in the queue. If you have a large file that a job needs to process, save the file on the disk or the cloud, then save the link to the file as a string in the queue. When bullmq executes the job, it will fetch the file from the link saved in Redis.

Save and close your file.


Next, create and open the utils.js file that will contain the image processing code:


```
nano utils.js


```


In your utils.js file, add the following code to define the function for processing an image:


image_processor/utils.js
```
const path = require("path");
const sharp = require("sharp");

function processUploadedImages(job) {
}

module.exports = { processUploadedImages };

```


You import the modules necessary to process images and compute paths in the first two lines. Then you define the processUploadedImages() function, which will contain the time-intensive image processing task. This function takes a job parameter that will be populated when the worker fetches the job data from the queue and then invokes the processUploadedImages() function with the queue data. You also export the processUploadedImages() function so that you can reference it in other files.


Save and close your file.


Return to the index.js file:


```
nano index.js


```


Copy the highlighted lines from the index.js file, then delete them from this file. You will need the copied code momentarily, so save it to a clipboard. If you are using nano, you can highlight these lines and right-click with your mouse to copy the lines:


image_processor/index.js
```
...
app.post("/upload", async function (req, res) {
  const { image } = req.files;

  if (!image) return res.sendStatus(400);
  const imageName = path.parse(image.name).name;
  const processImage = (size) =>
    sharp(image.data)
      .resize(size, size)
      .webp({ lossless: true })
      .toFile(`./public/images/${imageName}-${size}.webp`);

  sizes = [90, 96, 120, 144, 160, 180, 240, 288, 360, 480, 720, 1440];
  Promise.all(sizes.map(processImage))
  let counter = 0;
  for (let i = 0; i < 10_000_000_000; i++) {
    counter++;
  };
...
  res.redirect("/result");
});

```


The post method for the upload route will now match the following:


image_processor/index.js
```
...
app.post("/upload", async function (req, res) {
  const { image } = req.files;

  if (!image) return res.sendStatus(400);

  await addJob({
    type: "processUploadedImages",
    image: {
      data: image.data.toString("base64"),
      name: image.name,
    },
  });

  res.redirect("/result");
});
...

```


Save and close this file, then open the utils.js file:


```
nano utils.js


```


In your utils.js file, paste the lines you just copied for the /upload route callback into the processUploadedImages function:


image_processor/utils.js
```
...
function processUploadedImages(job) {
  const imageName = path.parse(image.name).name;
  const processImage = (size) =>
    sharp(image.data)
      .resize(size, size)
      .webp({ lossless: true })
      .toFile(`./public/images/${imageName}-${size}.webp`);

  sizes = [90, 96, 120, 144, 160, 180, 240, 288, 360, 480, 720, 1440];
  Promise.all(sizes.map(processImage));
  let counter = 0;
  for (let i = 0; i < 10_000_000_000; i++) {
    counter++;
  };
}
...

```


Now that you have moved the code for processing an image, you need to update it to use the image data from the job parameter of the processUploadedImages() function you defined earlier.


To do that, add and update the highlighted lines below:


image_processor/utils.js
```

function processUploadedImages(job) {
  const imageFileData = Buffer.from(job.image.data, "base64");
  const imageName = path.parse(job.image.name).name;
  const processImage = (size) =>
    sharp(imageFileData)
      .resize(size, size)
      .webp({ lossless: true })
      .toFile(`./public/images/${imageName}-${size}.webp`);
  ...
}

```


You convert the stringified version of the image data back to binary with the Buffer.from() method. Then you update path.parse() with a reference to the image name saved in the queue. After that, you update the sharp() method to take the image binary data stored in the imageFileData variable.


The complete utils.js file will now match the following:


image_processor/utils.js
```
const path = require("path");
const sharp = require("sharp");

function processUploadedImages(job) {
  const imageFileData = Buffer.from(job.image.data, "base64");
  const imageName = path.parse(job.image.name).name;
  const processImage = (size) =>
    sharp(imageFileData)
      .resize(size, size)
      .webp({ lossless: true })
      .toFile(`./public/images/${imageName}-${size}.webp`);

  sizes = [90, 96, 120, 144, 160, 180, 240, 288, 360, 480, 720, 1440];
  Promise.all(sizes.map(processImage));
  let counter = 0;
  for (let i = 0; i < 10_000_000_000; i++) {
    counter++;
  };
}

module.exports = { processUploadedImages };

```


Save and close your file, then return to the index.js:


```
nano index.js


```


The sharp variable is no longer needed as a dependency since the image is now processed in the utils.js file. Delete the highlighted line from the file:


image_processor/index.js
```
const bodyParser = require("body-parser");
const sharp = require("sharp");
const fileUpload = require("express-fileupload");
const { Queue } = require("bullmq");
...

```


Save and close your file.


You have now defined the functionality to create a queue in Redis and add a job. You also defined the processUploadedImages() function to process uploaded images.


The remaining task is to create a consumer (or worker) that will pull a job from the queue and call the processUploadedImages() function with the job data.


Create a worker.js file in your editor:


```
nano worker.js


```


In your worker.js file, add the following code:


image_processor/worker.js
```
const { Worker } = require("bullmq");

const { processUploadedImages } = require("./utils");

const workerHandler = (job) => {
  console.log("Starting job:", job.name);
  processUploadedImages(job.data);
  console.log("Finished job:", job.name);
  return;
};

```


In the first line, you import the Worker class from bullmq; when instantiated, this will start a worker that dequeues jobs from the queue in Redis and executes them. Next, you reference the processUploadedImages() function from the utils.js file so that the worker can call the function with the data in the queue.


You define a workerHandler() function that takes a job parameter containing the job data in the queue. In the function, you log that the job has started, then invoke processUploadedImages() with the job data. After that, you log a success message and return null.


To allow the worker to connect to Redis, dequeue a job from the queue, and call the workerHandler() with the job data, add the following lines to the file:


image_processor/worker.js
```
...
const workerOptions = {
  connection: {
    host: "localhost",
    port: 6379,
  },
};

const worker = new Worker("imageJobQueue", workerHandler, workerOptions);

console.log("Worker started!");

```


Here, you set the workerOptions variable to an object containing Redis’s connection settings. You set the worker variable to an instance of the Worker class that takes the following parameters:


- imageJobQueue: the name of the job queue.
- workerHandler: the function that will run after a job has been dequeued from the Redis queue.
- workerOptions: the Redis config settings that the worker uses to establish a connection with Redis.

Finally, you log a success message.


After adding the lines, save and close your file.


You have now defined the bullmq worker functionality to dequeue jobs from the queue and execute them.


In your terminal, remove the images in the public/images directory so that you can start fresh for testing your app:


```
rm public/images/*


```


Next, run the index.js file:


```
node index.js


```


The app will start:


```
OutputServer running on port 3000

```


You’ll now start the worker. Open a second terminal session and navigate to the project directly:


```
cd image_processor/


```


Start the worker with the following command:


```
node worker.js


```


The worker will start:


```
OutputWorker started!

```


Visit http://localhost:3000/ in your browser. Press the Choose File button and select the underwater.png from your computer, then press the Upload Image button.


You may receive an instant response that tells you to refresh the page after a few seconds:





Alternatively, you might receive an instant response with some processed images on the page while others are still being processed:





You can refresh the page a few times to load all the resized images.


Return to the terminal where your worker is running. That terminal will have a message that matches the following:


```
OutputWorker started!
Starting job: processUploadedImages
Finished job: processUploadedImages

```


The output confirms that bullmq ran the job successfully.


Your app can still offload time-intensive tasks even if the worker is not running. To demonstrate this, stop the worker in the second terminal with CTRL+C.


In your initial terminal session, stop the Express server and remove the images in public/images:


```
rm public/images/*


```


After that, start the server again:


```
node index.js


```


In your browser, visit http://localhost:3000/ and upload the underwater.png image again. When you are redirected to the /result path, the images will not show on the page because the worker is not running:





Return to the terminal where you ran the worker and start the worker again:


```
node worker.js


```


The output will match the following, which lets you know that the job has started:


```
OutputWorker started!
Starting job: processUploadedImages

```


After the job has been completed and the output includes a line that reads Finished job: processUploadedImages, refresh the browser. The images will now load:





Stop the server and the worker.


You now can offload a time-intensive task to the background and execute it asynchronously using bullmq. In the next step, you will set up a dashboard to monitor the status of the queue.


# Step 4 — Adding a Dashboard to Monitor bullmq Queues


In this step, you will use the bull-board package to monitor the jobs in the Redis queue from a visual dashboard. This package will automatically create a user interface (UI) dashboard that displays and organizes the information about the bullmq jobs that are stored in the Redis queue. Using your browser, you can monitor the jobs that are completed, are waiting, or have failed without opening the Redis CLI in the terminal.


Open the index.js file in your text editor:


```
nano index.js


```


Add the highlighted code to import bull-board:


image_processor/index.js
```
...
const { Queue } = require("bullmq");
const { createBullBoard } = require("@bull-board/api");
const { BullMQAdapter } = require("@bull-board/api/bullMQAdapter");
const { ExpressAdapter } = require("@bull-board/express");
...

```


In the preceding code, you import the createBullBoard() method from bull-board. You also import BullMQAdapter, which allows bull-board access to bullmq queues, and ExpressAdapter, which provides functionality for Express to display the dashboard.


Next, add the highlighted code to connect bull-board with bullmq:


image_processor/index.js
```
...
async function addJob(job) {
  ...
}

const serverAdapter = new ExpressAdapter();
const bullBoard = createBullBoard({
  queues: [new BullMQAdapter(imageJobQueue)],
  serverAdapter: serverAdapter,
});
serverAdapter.setBasePath("/admin");

const app = express();
...

```


First, you set the serverAdapter to an instance of the ExpressAdapter. Next, you invoke createBullBoard() to initialize the dashboard with the bullmq queue data. You pass the function an object argument with queues and serverAdapter properties. The first property, queues, accepts an array of the queues you defined with bullmq, which is the imageJobQueue here. The second property, serverAdapter, contains an object that accepts an instance of the Express server adapter. After that, you set the /admin path to access the dashboard with the setBasePath() method.


Next, add the serverAdapter middleware for the /admin route:


image_processor/index.js
```
app.use(express.static("public"))

app.use("/admin", serverAdapter.getRouter());

app.get("/", function (req, res) {
  ...
});

```


The complete index.js file will match the following:


image_processor/index.js
```
const path = require("path");
const fs = require("fs");
const express = require("express");
const bodyParser = require("body-parser");
const fileUpload = require("express-fileupload");
const { Queue } = require("bullmq");
const { createBullBoard } = require("@bull-board/api");
const { BullMQAdapter } = require("@bull-board/api/bullMQAdapter");
const { ExpressAdapter } = require("@bull-board/express");

const redisOptions = { host: "localhost", port: 6379 };

const imageJobQueue = new Queue("imageJobQueue", {
  connection: redisOptions,
});

async function addJob(job) {
  await imageJobQueue.add(job.type, job);
}

const serverAdapter = new ExpressAdapter();
const bullBoard = createBullBoard({
  queues: [new BullMQAdapter(imageJobQueue)],
  serverAdapter: serverAdapter,
});
serverAdapter.setBasePath("/admin");

const app = express();
app.set("view engine", "ejs");
app.use(bodyParser.json());
app.use(
  bodyParser.urlencoded({
    extended: true,
  })
);
app.use(fileUpload());

app.use(express.static("public"));

app.use("/admin", serverAdapter.getRouter());

app.get("/", function (req, res) {
  res.render("form");
});

app.get("/result", (req, res) => {
  const imgDirPath = path.join(__dirname, "./public/images");
  let imgFiles = fs.readdirSync(imgDirPath).map((image) => {
    return `images/${image}`;
  });
  res.render("result", { imgFiles });
});

app.post("/upload", async function (req, res) {
  const { image } = req.files;

  if (!image) return res.sendStatus(400);

  await addJob({
    type: "processUploadedImages",
    image: {
      data: Buffer.from(image.data).toString("base64"),
      name: image.name,
    },
  });

  res.redirect("/result");
});

app.listen(3000, function () {
  console.log("Server running on port 3000");
});

```


After you are done making changes, save and close your file.


Run the index.js file:


```
node index.js


```


Return to your browser and visit http://localhost:3000/admin. The dashboard will load:





In the dashboard, you can review the job type, the data it consumes, and more information about the job. You can also switch to other tabs, such as the Completed tab for information about the completed jobs, the Failed tab for more information about the jobs that failed, and the Paused tab for more information about the jobs that have been paused.


You can now use the bull-board dashboard to monitor queues.


# Conclusion


In this article, you offloaded a time-intensive task to a job queue using bullmq. First, without using bullmq, you created an app with a time-intensive task that has a slow request/response cycle. Then you used bullmq to offload the time-intensive task and execute asynchronously, which boosts the request/response cycle. After that, you used bull-board to create a dashboard to monitor bullmq queues in Redis.


You can visit the bullmq documentation to learn more about bullmq features not covered in this tutorial, such as scheduling, prioritizing or retrying jobs, and configuring concurrency settings for workers. You can also visit the bull-board documentation to learn more about the dashboard features.


