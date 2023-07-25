# How To Run Parse Server on Ubuntu 14 04

```Ubuntu``` ```Node.js``` ```MongoDB``` ```NoSQL```

## Introduction


Parse is a Mobile Backend as a Service platform, owned by Facebook since 2013.  In January of 2016, Parse announced that its hosted services would shut down in January of 2017.


In order to help its users transition away from the service, Parse has released an open source version of its backend, called Parse Server, which can be deployed to environments running Node.js and MongoDB.


This guide supplements the official documentation with detailed instructions for installing Parse Server on an Ubuntu 14.04 system, such as a DigitalOcean Droplet.  It is intended first and foremost as a starting point for Parse developers who are considering migrating their applications, and should be read in conjunction with the official Parse Server Guide.


# Prerequisites


This guide assumes that you have a clean Ubuntu 14.04 system, configured with a non-root user with sudo privileges for administrative tasks.  You may wish to review the guides in the New Ubuntu 14.04 Server Checklist series.


Additionally, your system will need a running instance of MongoDB.  You can start by working through How to Install MongoDB on Ubuntu 14.04.  MongoDB can also be installed automatically on a new Droplet by adding this script to its User Data when creating it. Check out this tutorial to learn more about Droplet User Data.


Once your system is configured with a sudo user and MongoDB, return to this guide and continue.


# Step 1 — Install Node.js and Development Tools


Begin by changing the current working path to your sudo user’s home directory:


```
cd ~


```


NodeSource offers an Apt repository for Debian and Ubuntu Node.js packages.  We’ll use it to install Node.js.  NodeSource offers an installation script for the the latest stable release (v5.5.0 at the time of this writing), which can be found in the installation instructions.  Download the script with curl:


```
curl -sL https://deb.nodesource.com/setup_5.x -o nodesource_setup.sh


```


You can review the contents of this script by opening it with nano, or your text editor of choice:


```
nano ./nodesource_setup.sh


```


Next, run nodesource_setup.sh.  The -E option to sudo tells it to preserve the user’s environment variables so that they can be accessed by the script:


```
sudo -E bash ./nodesource_setup.sh


```


Once the script has finished, NodeSource repositories should be available on the system.  We can use apt-get to install the nodejs package.  We’ll also install the build-essential metapackage, which provides a range of development tools that may be useful later, and the Git version control system for retrieving projects from GitHub:


```
sudo apt-get install -y nodejs build-essential git


```


# Step 2 — Install an Example Parse Server App


Parse Server is designed to be used in conjunction with Express, a popular web application framework for Node.js which allows middleware components conforming to a defined API to be mounted on a given path.  The parse-server-example repository contains a stubbed-out example implementation of this pattern.


Retrieve the repository with git:


```
git clone https://github.com/ParsePlatform/parse-server-example.git


```


Enter the parse-server-example directory you just cloned:


```
cd ~/parse-server-example


```


Use npm to install dependencies, including parse-server, in the current directory:


```
npm install


```


npm will fetch all of the modules required by parse-server and store them in ~/parse-server-example/node_modules.


# Step 3 — Test the Sample Application


Use npm to start the service.  This will run a command defined in the start property of package.json.  In this case, it runs node index.js:


```
npm start


```


```
Output> parse-server-example@1.0.0 start /home/sammy/parse-server-example
> node index.js

DATABASE_URI not specified, falling back to localhost.
parse-server-example running on port 1337.

```


You can terminate the running application at any time by pressing Ctrl-C.


The Express app defined in index.js will pass HTTP requests on to the parse-server module, which in turn communicates with your MongoDB instance and invokes functions defined in ~/parse-server-example/cloud/main.js.


In this case, the endpoint for Parse Server API calls defaults to:


http://your_server_IP/parse


In another terminal, you can use curl to test this endpoint.  Make sure you’re logged into your server first, since these commands reference localhost instead of a specific IP address.


Create a record by sending a POST request with an X-Parse-Application-Id header to identify the application, along with some data formatted as JSON:


```
curl -X POST \
  -H "X-Parse-Application-Id: myAppId" \
  -H "Content-Type: application/json" \
  -d '{"score":1337,"playerName":"Sammy","cheatMode":false}' \
  http://localhost:1337/parse/classes/GameScore

```


```
Output{"objectId":"fu7t4oWLuW","createdAt":"2016-02-02T18:43:00.659Z"}

```


The data you sent is stored in MongoDB, and can be retrieved by using curl to send a GET request:


```
curl -H "X-Parse-Application-Id: myAppId" http://localhost:1337/parse/classes/GameScore


```


```
Output{"results":[{"objectId":"GWuEydYCcd","score":1337,"playerName":"Sammy","cheatMode":false,"updatedAt":"2016-02-02T04:04:29.497Z","createdAt":"2016-02-02T04:04:29.497Z"}]}

```


Run a function defined in ~/parse-server-example/cloud/main.js:


```
curl -X POST \
  -H "X-Parse-Application-Id: myAppId" \
  -H "Content-Type: application/json" \
  -d '{}' \
  http://localhost:1337/parse/functions/hello

```


```
Output{"result":"Hi"}

```


# Step 4 — Configure Sample Application


In your original terminal, press Ctrl-C to stop the running version of the Parse Server application.


As written, the sample script can be configured by the use of six environment variables:





Variable
Description




DATABASE_URI
A MongoDB connection URI, like mongodb://localhost:27017/dev


CLOUD_CODE_MAIN
A path to a file containing Parse Cloud Code functions, like cloud/main.js


APP_ID
A string identifier for your app, like myAppId


MASTER_KEY
A secret master key which allows you to bypass all of the app’s security mechanisms


PARSE_MOUNT
The path where the Parse Server API should be served, like /parse


PORT
The port the app should listen on, like 1337




You can set any of these values before running the script with the export command.  For example:


```
export APP_ID=fooApp


```


It’s worth reading through the contents of index.js, but in order to get a clearer picture of what’s going on, you can also write your own shorter version of the example .  Open a new script in your editor:


```
nano my_app.js


```


And paste the following, changing the highlighted values where desired:


~/parse-server-example/my_app.js
```
var express = require('express');
var ParseServer = require('parse-server').ParseServer;

// Configure the Parse API
var api = new ParseServer({
  databaseURI: 'mongodb://localhost:27017/dev',
  cloud: __dirname + '/cloud/main.js',
  appId: 'myOtherAppId',
  masterKey: 'myMasterKey'
});

var app = express();

// Serve the Parse API on the /parse URL prefix
app.use('/myparseapp', api);

// Listen for connections on port 1337
var port = 9999;
app.listen(port, function() {
    console.log('parse-server-example running on port ' + port + '.');
});

```


Exit and save the file, then run it with Node.js:


```
node my_app.js


```


```
Outputparse-server-example running on port 9999.

```


Again, you can press Ctrl-C at any time to stop my_app.js.  As written above, the sample my_app.js will behave nearly identically to the provided index.js, except that it will listen on port 9999, with Parse Server mounted at /myparseapp, so that the endpoint URL looks like so:


http://your_server_IP:9999/myparseapp


And it can be tested with curl like so:


```
curl -H "X-Parse-Application-Id: myOtherAppId" http://localhost:9999/myparseapp/classes/GameScore`


```


# Conclusion


You should now know the basics of running a Node.js application like Parse Server in an Ubuntu environment.  Fully migrating an app from Parse is likely to be a more involved undertaking, requiring code changes and careful planning of infrastructure.


For much greater detail on this process, see the second guide in this series, How To Migrate a Parse App to Parse Server on Ubuntu 14.04.  You should also reference the official Parse Server Guide, particularly the section on migrating an existing Parse app.


