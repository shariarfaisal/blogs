# How To Install Express  a Node js Framework  and Set Up Socket io on a VPS

```Node.js``` ```Development```

## What You Will Learn In This Article



- How to install NodeJS 0.10.16 using Node Version Manager (NVM)
- How to install the Express web application framework for NodeJS
- How to setup a simple Express project
- How to setup socket.io in Express
- Sending simple messages to the client in real-time
- How to listen for messages using client-side javascript

# Step 1: Setting up NodeJS



Note: You may skip this step if you already know you have NodeJS installed v0.10.16


Node Version Manager (NVM) is a tool to help install various versions of NodeJS on your linux machine. In order to use NVM ensure you have git and curl installed.


Connect to your VPS (droplet) using SSH.


If you do not have these installed, use your system’s package manager to install them. For example, on an Ubuntu or Debian install you would run:


```
```
sudo apt-get install curl git
```

```


Now you must run the NVM install script:


```
curl https://raw.github.com/creationix/nvm/master/install.sh | sh

```


IMPORTANT: You must now logout of your box and reconnect using SSH.


Test to make sure the nvm command works by typing nvm at the terminal. If you do not get a command not found error, then you have correctly setup NVM.


To install the latest version of Node (which is 0.10.16 at the time of this article), simply type:


```
nvm install 0.10.16

```


Then wait for the installation to complete. If the install was successful, you should get an output reading: Now using node v0.10.16.


Type node -v at the terminal to ensure you are using the specified version. You should get the output: v0.10.16


# Step 2: Setting Up Express



Express is a web application framework for Node. It is minimal and flexible. In order to start using Express, you need to use NPM to install the module. Simple type:


```
npm install -g express

```


This will install the Express command line tool, which will aid in creating a basic web application. Once you have Express installed, follow these steps to create an empty Express project:


```
mkdir socketio-test

cd socketio-test

express

npm install

```


These commands will create an empty Express project in the directory we just created socketio-test. We then run npm install to get all the dependencies that are needed to run the app. To test the empty application, run node app then navigate your browser to http://yourDropletIp-or-domainName:3000. You should get a simple welcome message saying: “Welcome to Express”.


If you see the welcome message then you have a basic express application ready and running!


Be sure to kill your VPS with the Ctrl+C keyboard command before you keep going.


# Step 3: Installing Socket.io Into Your Express Application



First, a quick summary of what Socket.io is. Socket.io is a real-time JavaScript library. In short, it’s a WebSocket API that will determine the correct type of connection to make depending on the browser’s capabilities, whether it be AJAX Long Polling, Flash, or even just plain WebSockets.


So how do you get started with this? First you need a Socket.io server. We already have an Express server ready and waiting, all we need to do is add on the socket library. To do that we have to add it to the package.json file.


Your initial file might look something like this:


```
{
  "name": "application-name",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "3.3.5",
    "jade": "*"
  }
}

```


Now add a new field to the “dependencies” area:


```
"socket.io": "latest",

```


Your resulting file should look something like this:


```
{
  "name": "application-name",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "socket.io": "latest",
    "express": "3.3.5",
    "jade": "*"
  }
}

```


Now run npm install once more to install the socket library.


# Part 4: The Server Code



Now that we have all the dependencies set up, we can start the code! Go and open up the app.js file in the Express application folder. Inside you’ll a bunch of auto-generated code, delete all of it and use the following example instead:


```
/**
 * Module dependencies.
 */
 
var express = require('express')
  , routes = require('./routes')
  , http = require('http');
 
var app = express();
var server = app.listen(3000);
var io = require('socket.io').listen(server); // this tells socket.io to use our express server
 
app.configure(function(){
  app.set('views', __dirname + '/views');
  app.set('view engine', 'jade');
  app.use(express.favicon());
  app.use(express.logger('dev'));
  app.use(express.static(__dirname + '/public'));
  app.use(express.bodyParser());
  app.use(express.methodOverride());
  app.use(app.router);
});
 
app.configure('development', function(){
  app.use(express.errorHandler());
});
 
app.get('/', routes.index);
 
 
console.log("Express server listening on port 3000");

```


All this example file has done has reorganized the auto-generated code and added the var io = require('socket.io').listen(server); line which tells socket.io to listen on and use our Express server. If you run your node app you should see it output: info - socket.io started.


Now how do we transport a message to the user?


Add the following lines to your app.js just below the last line.


```
io.sockets.on('connection', function (socket) {
    console.log('A new user connected!');
    socket.emit('info', { msg: 'The world is round, there is no up or down.' });
});

```


This will send out a new socket message whenever a new user connects to the server. Now we just need a way to interact with the VPS on the client side.


# Part 5: The Client Side Code



Naviagate to the public/javascripts folder inside your Express application and add a new file called client.js and put this code into it:


```
// connect to the socket server
var socket = io.connect(); 

// if we get an "info" emit from the socket server then console.log the data we recive
socket.on('info', function (data) {
    console.log(data);
});

```


The code is simple but it demonstrates what you can do with Socket.io. Now we just need to include the scripts into our main page.


Navigate to your views folder in the Express app and open layout.jade. Express does not use plain HTML to render its pages. It uses a templating engine called Jade. (More information about Jade can be found here) Jade is simple and clean compared to plain HTML. To add our client script and add the Socket.io client library we simple need to add these to lines just below the link(rel='stylesheet', href='/stylesheets/style.css'):


```
script(type='text/javascript', src="/socket.io/socket.io.js")
script(type='text/javascript', src='javascripts/client.js')

```


Make sure they line up on the same indent level and do NOT mix tabs and spaces. This will cause Jade to throw an error.


Switch back into the socketio-test directory:


```
cd socketio-test

```


Save the layout file and start your Express app once again using: node app.


# Part 6: Testing It



Now navigate your browser once more to http://yourDropletIp-or-domainName:3000 and open the developer tools console. You should see something like this in the log:


```
Object {msg: "The world is round, there is no up or down."}

```


This is a message sent directly from the VPS to the client in real-time.


<div class=“author”>Submitted by: <a href=“https://twitter.com/aaron524web”>Aaron Shea</a></div>


