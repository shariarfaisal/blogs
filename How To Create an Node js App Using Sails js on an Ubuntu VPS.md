# How To Create an Node js App Using Sails js on an Ubuntu VPS

```Ubuntu``` ```Node.js```

# What the Red means


The lines that the user needs to enter or customize will be in red in this tutorial!


The rest should mostly be copy-and-pastable.


## What is Sails.js?


Sails.js makes it easy to build custom, enterprise-grade Node.js apps. It is designed to mimic the MVC pattern of frameworks like Ruby on Rails, but with support for the requirements of modern apps: data-driven APIs with scalable, service-oriented architecture. It's especially good for building chat, realtime dashboards, or multiplayer games.


In other words: Sails.js allows you to easily create apps with Node.js using the Model-View-Controller pattern to organize your code so it is easier to maintain.
Sails.js provides various commands to automate the creation of models and controllers, saving you time and allowing you to create the app faster. (Views have to be created manually in the /views directory in the template /views/:controller/:method.ejs). You are able to use various templating languages, however EJS is the default, and it would be safest and easiest to just stick with EJS.


Sails.js also has various "adapters", allowing you to use virtually any database you want to with your app. This gives you the maximum amount of flexibility, differing from other MVC frameworks that insist on you using MongoDB.


All of these and the fact that on deployment, all of your files are concatenated and minified means that you don't have to spend as much time setting the main framework up to build your app on top of, as it is all ready and easy to use.


# Installing Node.js on an Ubuntu VPS


1. 
    Installing the pre-requisites:
    sudo apt-get install python-software-properties python g++ make
    If you are using Ubuntu 12.10, you will need to do the following as well:
    sudo apt-get install software-properties-common
  
2. 
    Add the PPA Repository, which is recommended by Joyent (the maintainers of Node.js):
    sudo add-apt-repository ppa:chris-lea/node.js
  
3. 
    Update the package list:
    sudo apt-get update
  
4. 
    Install Node.js:
    sudo apt-get install nodejs
  

# Installing Sails.js


To install the latest stable release of sails, you will need to run:


```
sudo npm install sails -g
```


The -g flag ensures that sails is installed globally and can be used as a command-line tool.


# Creating your Sails app


You will want to navigate to the directory in which you would like your app to be located, e.g. /var/www and then run:


```
sails new project-name
```


This will add the required files to your project, and create a directory named project-name.


# Starting up the Sails.js server


To view your boilerplate app, you will need to change directory into the project directory and then start the server:


```
cd project-name
```


then:


```
sails lift
```


This will create a server running at 123.456.78.90:1337, and the page may look something like this (They have changed it a few times, so depending on when you are reading this article, it may be different):


![](https://assets.digitalocean.com/tutorial_images/Wun9wyy.png)
# Creating Controllers


Creating a controller is easy, the sails CLI does all the hard stuff for you.
e.g. To create a controller called user with the methods "index, show, edit, delete", all you have to do is run the following command:


```
sails generate controller user index show edit delete
```


This will create a file in api/controllers called UserController.js that looks something like this (giving you helpful hints on what the function does, and how it works):


```
/*---------------------
        :: User
        -> controller
---------------------*/
var UserController = {
  
  // To trigger this action locally, visit: 'http://localhost:port/user/index'
  index: function(req,res) {
      
        // This will render the view: /var/www/sails-test/views/user/index.ejs
        res.view();

  },

  // To trigger this action locally, visit: 'http://localhost:port/user/show'
  show: function(req,res) {
      
        // This will render the view: /var/www/sails-test/views/user/show.ejs
        res.view();

  },

  // To trigger this action locally, visit: 'http://localhost:port/user/edit'
  edit: function(req,res) {
      
        // This will render the view: /var/www/sails-test/views/user/edit.ejs
        res.view();

  },

  // To trigger this action locally, visit: 'http://localhost:port/user/delete'
  delete: function(req,res) {
      
        // This will render the view: /var/www/sails-test/views/user/delete.ejs
        res.view();

  },

};
module.exports = UserController;

```


# Creating Models


Creating a model is as easy as creating a controller with Sails.js. You have no database migrations to worry about, Sails.js does all of that for you intelligently. You are able to use the default in-file database, MySQL or many other database types via "adapters" which can be found by searching around, or looking through the creator's GitHub repositories.


When creating a model, you can specify fields to be added to that model by adding them afterwards, in the format of [name]:[type].


e.g. To create a model called user with the fields "name, email, password", all you have to do is run the following command:


```
sails generate model user name:string email:string password:string
```


This will create a file in api/models called User.js that looks something like this:


```
/*---------------------
        :: User
        -> model
---------------------*/
module.exports = {
    attributes: {
        
        // Simple attribute:
        // name: 'STRING',

        // Or for more flexibility:
        // phoneNumber: {
        //    type: 'STRING',
        //    defaultsTo: '555-555-5555'
        // }

        name: {
            type: 'STRING'
        },

        email: {
            type: 'STRING'
        },

        password: {
            type: 'STRING'
        },

    }
};

```


# Creating a Blueprint API


You can generate the Controller and Model at the same time, also generating a Blueprint API that allows you to visit /user and view the raw json representation of the data stored.


```
sails generate user
```


The Blueprint API saves you time in the short term by creating connections between your model and controller, allowing you to add new records to the database using that route, e.g. going to http://localhost/user/create?name=John+Smith will create a new user with the name of "John Smith" and print out a JSON array of all of the records created in that model, so the previous URL would print out:


```
{"name":"John Smith","createdAt":"2013-07-10T20:10:01.038Z","updatedAt":"2013-07-10T20:10:01.038Z","id":1}
```


In practice, you would want to change or add new methods in the controller, however every new Controller has create(), find(), findAll(), update(), and destroy() methods by default. These can be overridden though, to disable them or just customise them.


The data can also be populated by performing a POST request with a JSON string of the data that you want to insert, e.g.


```
{
  name: 'John Smith'
}

```


# Adding Routes


Routes can be added by opening config/routes.js. The file is very well documented using comments, so I feel it is only necessary to add an image rather than describe it, if anything.


![](https://assets.digitalocean.com/tutorial_images/4nFZSzj.png)
# Setting the Server to Production Mode


When you are ready to deploy an app to production, moving it from port 1337 to 80, Sails.js allows you to do this easily.


1. Open up config/application.js.
2. If this is the ONLY app on the server: On line 7, Change:
  port: 1337,
  to
  port: 80,
3. If there are multiple apps on the server and if you are running NGINX:
    You will need to modify the server block config to proxy the port that your Sails.js app is being run on by adding:
    location / {
    proxy_pass         http://127.0.0.1:1337/;
    proxy_redirect     off;
proxy_set_header   Host             $host;
proxy_set_header   X-Real-IP        $remote_addr;
proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;

}

(Leave the Port for Sails.js as 1337)
  
4. If there are multiple apps on the server and if you are running Apache:
    You will need to modify the vhost config to proxy the port that your Sails.js app is being run on by adding:
    ProxyRequests Off
<Proxy *>
Order deny,allow
Allow from all
</Proxy>
ProxyPass / http://127.0.0.1:1337
ProxyPassReverse / http://127.0.0.1:1337

(Leave the Port for Sails.js as 1337)
  
5. On line 15, Change:
  environment: 'development',
  to
  environment: 'production',
6. Install forever by running:
sudo npm install -g forever
    Forever is a node.js package that allows your app to run in the background, without you having to keep terminal open all of the time.
7. Run:
forever start /path/to/app.js
to start the background process. You can check that your app is running in the background by typing in forever list. This will list all running node.js processes (each app is run as a process on the server)

Article Submitted by: Rob Brazier 
