# Utilizing npm Scripts During Development

```Node.js```

As developers in modern times we have access to increasingly more tools for improving the speed and efficiency at which we develop. Perhaps one of the most influential factors in the rising popularity of Node.js and other backend Javascript tools are that they allow us to use a familiar syntax at every stage of development.


Taskrunners, for example, are programs whose sole purpose is to automate mundane aspects of the development process. There are a variety of incredibly useful Javascript Taskrunners and Build tools to check out such as Grunt.js, Gulp, and webpack. Though powerful and invaluable to know, there is another tool that doesn’t get as much attention but behaves similarly and can be equally valuable to know in depth, npm’s built in scripts feature.


Let’s take a look at the various ways we can use npm scripts to help support our development needs.


# Package.json


Every project in Node has a package.json file that contains metadata about the project. In this file we find things such as the title, a description, version, and dependencies. The main purpose of this file is to allow publishing of our project on npm. When someone wants to install our program from npm, their system needs to know what other programs ours depends on to run properly. It also needs to know certain things about the behavior and configuration of the program. It retrieves all of this information from the package.json file.


A typical package.json file might look something like the following:


```
{
  "name": "Crispy",
  "version": "1.0.0",
  "description": "Cooked to perfection!",
  "main": "index.js",
  "scripts": {
    "test": "mocha"
  },
  "author": "AlligatorIO",
  "license": "ISC",
  "dependencies": {
    "ws": "^3.3.2"
  },
  "devDependencies": {
    "webpack": "^3.8.1"
  }
}

```


As you can see, it’s nothing more than a file containing a JSON object with information relevant to our project.


One piece of this information is the scripts section. These scripts are commands that will be triggered at various moments throughout the development and publishing lifecycle. Two of the most common npm scripts are the start script, and the test script. You’ll notice in the previous example that there is no start script defined. This is because npm has a default value for the start script which is node server.js. This means that if we don’t choose to define our own custom start script, typing npm start into the command line will automatically look for a file named server.js and run that file in Node if found.


Also notice that our test script’s value is simply “mocha”. Mocha is a common JavaScript testing toolkit, and once installed it can be run simply by using the mocha command in the command line. In this case our test script is doing nothing more than just that.


There are many different npm scripts that cover a wide range of use cases, but what we’re most interested in right now is the various ways we can use these scripts to make our lives easier during the development of a new project.


# Understanding the Lifecycle


The first key to unlocking the power of npm scripts is to understand what npm is doing when you run a script. The first thing it does is check the package.json file to see if you’ve defined a value for that script. If it finds that you have, it then looks for two other versions of the script. A ‘pre’ version and a ‘post’ version. If it finds either of these, it will run them in respect to the specified script.


Example:


```
{
  "scripts": {
    "prestart": "node loadJaw.js",
    "start": "node Gator.js",
    "poststart": "node bite.js"
  }
}

```


If this is what npm were to find in our package.json file it would run the scripts in the order prestart, start, then poststart. All we would be doing to trigger this is typing npm start in the command line. This is incredibly useful for a variety of situations where you need something to happen immediately before or immediately after a primary action. In development, we might need to connect to launch a local copy of our database or bundle our files and we need to make sure these things happen before our server launches to avoid errors.


If we set up our scripts properly then simply running the command npm start will be able to handle all of our needs in the proper order.


# Development Mode vs Production Mode


One of the most useful ways to utilize npm scripts is to alter the value of Node’s Node_ENV variable which is accessible in any Node program through the global Process variable. Simply reference process.env.Node_ENV to find its value.


```
function getEnvironment(){
  return process.env.Node_ENV;
}

getEnvironment(); // returns 'production';

```


Many frameworks use the value of the Node_ENV variable to determine whether to run in development mode or in production mode. The difference being that oftentimes production mode is optimized for performance, where as development mode is optimized for debugging. We can make use of this common mechanism in our own projects by writing programs that are configured differently depending on the value of the Node_ENV variable.


For example, imagine we want to log visitors to our site. In production mode we might just want to log the IP address of each new visitor to a log file for later reference. In development mode we might be analyzing traffic in real time to sort out bugs, so we’ll want to log information about new visitors to a different file as well as log the traffic to the console so we can watch it as it happens. This could be as simple as:


```
const fs = require('fs');
const inDevelopmentMode = process.env.Node_ENV === development;

function newVisitor(ip){
  let logMessage = "New Visitor IP: " + ip;

  if(inDevelopmentMode){
    fs.writeFile("development_log.txt", logMessage, (err) => {
      if(err) throw err;
    });
    console.log(logMessage);
  }
  else{
    fs.writeFile("production_log.txt", logMessage, (err) => {
      if(err) throw err;
    });
  }
}

```


Here we are using the fs module to interact with our file system. We are referencing the Node_ENV variable to determine if we are in development mode and if so logging both to the development_log.txt file and the console. If we are in production mode it will simply log to our production_log.txt file without logging to the console.


But how do we actually alter the Node_ENV variable? One way is to use npm scripts! In our package.json file:


```
{
  "scripts": {
    "start": "SET Node_ENV=development& node server.js",
  }
}

```


The syntax might look a little funny and conjure up a few questions on first glance. Here’s an explanation of why it’s done this way. There are a few important yet unpredictable things to watch out for when attempting to do this.


- 
Setting the Node_ENV variable at a step other than that which intends to interact with it can lead to errors. This is because each time an npm script is run it is considered another “process”. If we were to change the Node_ENV value in the prestart script, it would not carry over to the start scripts process. Therefore we have to change it in the start script if we intend to interact with it in our server.js file.

- 
There is a slight difference between the way Linux and Windows handle changing the variable. In Linux you can omit the SET command and simply use Node_ENV=development& node server.js. In windows you have to use the SET command to change the variable.

- 
It is important to make sure that there is no space between development and & in this line of code, otherwise that whitespace will be part of the value of the variable and lead to false when checking process.env.Node_ENV === "development".


Of course the difference between the way Linux and Windows sets the environment variable can lead to incompatibility if a team is working on a project from both operating systems. There are a few ways to handle this. You could use a script or module such as cross-env that tries to determine what operating system the program is running on before running the command, or you could have different npm scripts to use for each operating system and leave it up to the developers to remember which to use. Both ways are valid, but how would we go about creating alternate npm scripts that do similar things for different contexts?


# Arbitrary Scripts


Luckily npm provides a built in way to do this already by allowing us define our own custom arbitrary scripts. We can name these scripts whatever we want and npm will still run them exactly as we expect. It will still attempt to run all lifecycle variations of the script when triggered, by prepending “pre” and “post” to the name provided. This allows us to have any amount of custom scripts we can use while developing. There’s only one key difference and that is in the way we trigger the script from the command line.


Where for every natively supported npm script we type npm <script-name>, to trigger an arbitrary script we need to use npm run <script-name>. That’s the only difference. If we needed a modified start script for Linux users to work with we could create a script that omits the SET command from the start script and name it whatever we choose.


And if we wanted to have a custom script for launching a local MongoDB instance, we could simply do something like this:


```
{
  "scripts": {
    "start": "SET Node_ENV=development& node server.js",
    "mongo": "mongod --dbpath=./pathToDatabase/db"
  }
}

```


Now we can use npm run mongo to launch the database.


Arbitrary scripts are great for triggering different sequences of commands at will. Take for example:


```
{
  "scripts": {
    "test": "mocha",
    "start": "SET Node_ENV=development& node server.js",
    "mongo": "mongod --dbpath=./pathToDatabase/db",
    "launch": "npm test && npm run mongo && npm start",
  }
}

```


Here we have a launch script which triggers our test, mongo, and start scripts in that order. We can also utilize packages such as concurrently in our scripts to run actions simultaneously.


```
{
  "scripts": {
    "start": "concurrently \"node grip.js bacon\" \"node bite.js bacon\""
  }
}

```


# Final Thoughts


Hopefully reading this has given you some ideas on how you can make use of npm scripts to assist your development workflow! I personally prefer to have my start script trigger a production environment and create an arbitrary dev script to trigger a development environment. This way I don’t need to alter anything else in the package.json to switch back to production mode when I’m ready, it’s as simple as using npm start.


