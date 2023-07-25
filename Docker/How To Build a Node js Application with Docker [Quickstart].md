# How To Build a Node js Application with Docker [Quickstart]

```Docker``` ```Node.js``` ```Applications``` ```Quickstart```

## Introduction


This tutorial will walk you through creating an application image for a static website that uses the Express framework and Bootstrap. You will then build a container using that image, push it to Docker Hub, and use it to build another container, demonstrating how you can recreate and scale your application.


For a more detailed version of this tutorial, with more detailed explanations of each step, please refer to How To Build a Node.js Application with Docker.


# Prerequisites


To follow this tutorial, you will need:


- A sudo user on your server or in your local environment.
- Docker.
- Node.js and npm.
- A Docker Hub account.

# Step 1 — Installing Your Application Dependencies


First, create a directory for your project in your non-root user’s home directory:


```
mkdir node_project


```


Navigate to this directory:


```
cd node_project


```


This will be the root directory of the project.


Next, create a package.json with your project’s dependencies:


```
nano package.json


```


Add the following information about the project to the file; be sure to replace the author information with your own name and contact details:


~/node_project/package.json
```
{
  "name": "nodejs-image-demo",
  "version": "1.0.0",
  "description": "nodejs image demo",
  "author": "Sammy the Shark <sammy@example.com>",
  "license": "MIT",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "nodejs",
    "bootstrap",
    "express"
  ],
  "dependencies": {
    "express": "^4.16.4"
  }
}

```


Install your project’s dependencies:


```
npm install


```


# Step 2 — Creating the Application Files


We will create a website that offers users information about sharks.


Open app.js in the main project directory to define the project’s routes:


```
nano app.js


```


Add the following content to the file to create the Express application and Router objects, define the base directory, port, and host as variables, set the routes, and mount the router middleware along with the application’s static assets:


~/node_project/app.js
```
var express = require("express");
var app = express();
var router = express.Router();

var path = __dirname + '/views/';

// Constants
const PORT = 8080;
const HOST = '0.0.0.0';

router.use(function (req,res,next) {
  console.log("/" + req.method);
  next();
});

router.get("/",function(req,res){
  res.sendFile(path + "index.html");
});

router.get("/sharks",function(req,res){
  res.sendFile(path + "sharks.html");
});

app.use(express.static(path));
app.use("/", router);

app.listen(8080, function () {
  console.log('Example app listening on port 8080!')
})

```


Next, let’s add some static content to the application. Create the views directory:


```
mkdir views


```


Open index.html:


```
nano views/index.html


```


Add the following code to the file, which will import Boostrap and create a jumbotron component with a link to the more detailed sharks.html info page:


~/node_project/views/index.html
```
<!DOCTYPE html>
<html lang="en">
   <head>
      <title>About Sharks</title>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
      <link href="css/styles.css" rel="stylesheet">
      <link href='https://fonts.googleapis.com/css?family=Merriweather:400,700' rel='stylesheet' type='text/css'>
      <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
      <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
   </head>
   <body>
      <nav class="navbar navbar-inverse navbar-static-top">
         <div class="container">
            <div class="navbar-header">
               <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
               <span class="sr-only">Toggle navigation</span>
               <span class="icon-bar"></span>
               <span class="icon-bar"></span>
               <span class="icon-bar"></span>
               </button>
               <a class="navbar-brand" href="#">Everything Sharks</a>
            </div>
            <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
               <ul class="nav navbar-nav mr-auto">
                  <li class="active"><a href="/">Home</a></li>
                  <li><a href="/sharks">Sharks</a></li>
               </ul>
            </div>
         </div>
      </nav>
      <div class="jumbotron">
         <div class="container">
            <h1>Want to Learn About Sharks?</h1>
            <p>Are you ready to learn about sharks?</p>
            <br>
            <p><a class="btn btn-primary btn-lg" href="/sharks" role="button">Get Shark Info</a></p>
         </div>
      </div>
      <div class="container">
         <div class="row">
            <div class="col-md-6">
               <h3>Not all sharks are alike</h3>
               <p>Though some are dangerous, sharks generally do not attack humans. Out of the 500 species known to researchers, only 30 have been known to attack humans.</p>
            </div>
            <div class="col-md-6">
               <h3>Sharks are ancient</h3>
               <p>There is evidence to suggest that sharks lived up to 400 million years ago.</p>
            </div>
         </div>
      </div>
   </body>
</html>

```


Next, open a file called sharks.html:


```
nano views/sharks.html


```


Add the following code, which imports Bootstrap and the custom style sheet and offers users detailed information about certain sharks:


~/node_project/views/sharks.html
```
<!DOCTYPE html>
<html lang="en">
   <head>
      <title>About Sharks</title>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
      <link href="css/styles.css" rel="stylesheet">
      <link href='https://fonts.googleapis.com/css?family=Merriweather:400,700' rel='stylesheet' type='text/css'>
      <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
      <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
   </head>
   <nav class="navbar navbar-inverse navbar-static-top">
      <div class="container">
         <div class="navbar-header">
            <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="#">Everything Sharks</a>
         </div>
         <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
            <ul class="nav navbar-nav mr-auto">
               <li><a href="/">Home</a></li>
               <li class="active"><a href="/sharks">Sharks</a></li>
            </ul>
         </div>
      </div>
   </nav>
   <div class="jumbotron text-center">
      <h1>Shark Info</h1>
   </div>
   <div class="container">
      <div class="row">
         <div class="col-md-6">
            <p>
            <div class="caption">Some sharks are known to be dangerous to humans, though many more are not. The sawshark, for example, is not considered a threat to humans.</div>
            <img src="https://assets.digitalocean.com/articles/docker_node_image/sawshark.jpg" alt="Sawshark">
            </p>
         </div>
         <div class="col-md-6">
            <p>
            <div class="caption">Other sharks are known to be friendly and welcoming!</div>
            <img src="https://assets.digitalocean.com/articles/docker_node_image/sammy.png" alt="Sammy the Shark">
            </p>
         </div>
      </div>
    </div>
   </body>
</html>

```


Finally, create the custom CSS style sheet that you’ve linked to in index.html and sharks.html by first creating a css folder in the views directory:


```
mkdir views/css


```


Open the style sheet and add the following code, which will set the desired color and font for our pages:


~/node_project/views/css/styles.css
```
.navbar {
        margin-bottom: 0;
}

body {
        background: #020A1B;
        color: #ffffff;
        font-family: 'Merriweather', sans-serif;
}
h1,
h2 {
        font-weight: bold;
}
p {
        font-size: 16px;
        color: #ffffff;
}


.jumbotron {
        background: #0048CD;
        color: white;
        text-align: center;
}
.jumbotron p {
        color: white;
        font-size: 26px;
}

.btn-primary {
        color: #fff;
        text-color: #000000;
        border-color: white;
        margin-bottom: 5px;
}

img, video, audio {
        margin-top: 20px;
        max-width: 80%;
}

div.caption: {
        float: left;
        clear: both;
}

```


Start the application:


```
npm start


```


Navigate your browser to http://your_server_ip:8080 or localhost:8080 if you are working locally. You will see the following landing page:





Click on the Get Shark Info button. You will see the following information page:





You now have an application up and running. When you are ready, quit the server by typing CTRL+C.


# Step 3 — Writing the Dockerfile


In your project’s root directory, create the Dockerfile:


```
nano Dockerfile


```


Add the following code to the file:


~/node_project/Dockerfile
```

FROM node:10-alpine

RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app

WORKDIR /home/node/app

COPY package*.json ./

USER node

RUN npm install

COPY --chown=node:node . .

EXPOSE 8080

CMD [ "node", "app.js" ]

```


This Dockerfile uses an alpine base image and ensures that application files are owned by the non-root node user that is provided by default by the Docker Node image.


Next, add your local node modules, npm logs, Dockerfile, and .dockerignore to your .dockerignore file:


~/node_project/.dockerignore
```
node_modules
npm-debug.log
Dockerfile
.dockerignore

```


Build the application image using the docker build command:


```
docker build -t your_dockerhub_username/nodejs-image-demo .


```


The . specifies that the build context is the current directory.


Check your images:


```
docker images


```


You will see the following output:


```
OutputREPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
your_dockerhub_username/nodejs-image-demo          latest              1c723fb2ef12        8 seconds ago       895MB
node                                               10                  f09e7c96b6de        17 hours ago        893MB

```


Run the following command to build a container using this image:


```
docker run --name nodejs-image-demo -p 80:8080 -d your_dockerhub_username/nodejs-image-demo 


```


Inspect the list of your running containers with docker ps:


```
docker ps


```


You will see the following output:


```
OutputCONTAINER ID        IMAGE                                                   COMMAND             CREATED             STATUS              PORTS                  NAMES
e50ad27074a7        your_dockerhub_username/nodejs-image-demo               "npm start"         8 seconds ago       Up 7 seconds        0.0.0.0:80->8080/tcp   nodejs-image-demo

```


With your container running, you can now visit your application by navigating your browser to http://your_server_ip or localhost. You will see your application landing page once again:





Now that you have created an image for your application, you can push it to Docker Hub for future use.


# Step 4 — Using a Repository to Work with Images


The first step to pushing the image is to log in to the your Docker Hub account:


```
docker login -u your_dockerhub_username -p your_dockerhub_password


```


Logging in this way will create a ~/.docker/config.json file in your user’s home directory with your Docker Hub credentials.


Push your image up using your own username in place of your_dockerhub_username:


```
docker push your_dockerhub_username/nodejs-image-demo


```


If you would like, you can test the utility of the image registry by destroying your current application container and image and rebuilding them.


First, list your running containers:


```
docker ps

```


You will see the following output:


```
OutputCONTAINER ID        IMAGE                                       COMMAND             CREATED             STATUS              PORTS                  NAMES
e50ad27074a7        your_dockerhub_username/nodejs-image-demo   "npm start"         3 minutes ago       Up 3 minutes        0.0.0.0:80->8080/tcp   nodejs-image-demo

```


Using the CONTAINER ID listed in your output, stop the running application container. Be sure to replace the highlighted ID below with your own CONTAINER ID:


```
docker stop e50ad27074a7


```


List your all of your images with the -a flag:


```
docker images -a


```


You will see the following output with the name of your image, your_dockerhub_username/nodejs-image-demo, along with the node image and the other images from your build:


```
OutputREPOSITORY                                           TAG                 IMAGE ID            CREATED             SIZE
your_dockerhub_username/nodejs-image-demo            latest              1c723fb2ef12        7 minutes ago       895MB
<none>                                               <none>              e039d1b9a6a0        7 minutes ago       895MB
<none>                                               <none>              dfa98908c5d1        7 minutes ago       895MB
<none>                                               <none>              b9a714435a86        7 minutes ago       895MB
<none>                                               <none>              51de3ed7e944        7 minutes ago       895MB
<none>                                               <none>              5228d6c3b480        7 minutes ago       895MB
<none>                                               <none>              833b622e5492        8 minutes ago       893MB
<none>                                               <none>              5c47cc4725f1        8 minutes ago       893MB
<none>                                               <none>              5386324d89fb        8 minutes ago       893MB
<none>                                               <none>              631661025e2d        8 minutes ago       893MB
node                                                 10                  f09e7c96b6de        17 hours ago        893MB

```


Remove the stopped container and all of the images, including unused or dangling images, with the following command:


```
docker system prune -a

```


With all of your images and containers deleted, you can now pull the application image from Docker Hub:


```
docker pull your_dockerhub_username/nodejs-image-demo


```


List your images once again:


```
docker images


```


You will see your application image:


```
OutputREPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
your_dockerhub_username/nodejs-image-demo      latest              1c723fb2ef12        11 minutes ago      895MB

```


You can now rebuild your container using the command from Step 3:


```
docker run --name nodejs-image-demo -p 80:8080 -d your_dockerhub_username/nodejs-image-demo


```


List your running containers:


```
docker ps


```


```
OutputCONTAINER ID        IMAGE                                                   COMMAND             CREATED             STATUS              PORTS                  NAMES
f6bc2f50dff6        your_dockerhub_username/nodejs-image-demo               "npm start"         4 seconds ago       Up 3 seconds        0.0.0.0:80->8080/tcp   nodejs-image-demo

```


Visit http://your_server_ip or localhost once again to view your running application.


# Related Tutorials


Here are links to more detailed guides related to this tutorial:


- How To Install Docker Compose on Ubuntu 18.04.
- How To Provision and Manage Remote Docker Hosts with Docker Machine on Ubuntu 18.04.
- How To Share Data between Docker Containers.
- How To Share Data Between the Docker Container and the Host.

You can also look at the longer series on From Containers to Kubernetes with Node.js, from which this tutorial is adapated.


Additionally, see our full library of Docker resources for more on Docker.


