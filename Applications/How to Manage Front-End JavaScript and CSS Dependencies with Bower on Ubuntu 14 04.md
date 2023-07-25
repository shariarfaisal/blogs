# How to Manage Front-End JavaScript and CSS Dependencies with Bower on Ubuntu 14 04

```Ubuntu``` ```Applications``` ```Nginx```

## Introduction


Long gone are the days when we had to manually search for, download, unpack, and figure out installation directories for our front-end frameworks, libraries, and assets.


Bower is a package manager for front-end modules that are usually comprised of JavaScript and/or CSS. It lets us easily search for, install, update, or remove these front-end dependencies.


The advantage to using Bower is that you do not have to bundle external dependencies with your project when you distribute it. Bower will take care of the third-party code when you run bower install and get those dependencies to the right locations. It also makes the final project package smaller for distribution.





In this tutorial you’ll learn how to install and use Bower on an Ubuntu 14.04 server. We’ll use to Bower to install Bootstrap and AngularJS and illustrate them running a simple application on an Nginx web server.


## Prerequisites


Before we begin, there are some important steps that you need to complete:


- 
Droplet with a clean Ubuntu 14.04 installation. For this purpose the size of the Droplet really doesn’t matter, so you can safely go with the smallest version. If you haven’t yet created your Droplet, you can do so by following the steps in the How To Create Your First DigitalOcean Droplet Virtual Server tutorial

- 
Connect to your server with SSH

- 
Create a user with sudo privileges by following our Ubuntu 14.04 Initial Server Setup tutorial. In our example, this user is called sammy

- 
For the web server, we will use Nginx, a powerful and efficient web server that has seen wide adoption due to its performance capabilities. Follow the How To Install Nginx on Ubuntu 14.04 LTS tutorial to install Nginx


Also, Bower needs Git, Node.js and npm.


- 
Install Git on your server with the following command:
sudo apt-get install git


If you want a more in-depth tutorial about setting up Git, you can take a look at How To Install Git on Ubuntu 14.04.

- sudo apt-get install git

- 
Install Node.js on your server with the following command:
sudo apt-get install nodejs


If you want a more in-depth tutorial about setting up Node.js, you can take a look at How To Install Node.js on an Ubuntu 14.04 server.

- sudo apt-get install nodejs

- 
Install npm on your server with the following command:
sudo apt-get install npm


If you want a more in-depth tutorial about setting up npm, you can take a look at How To Use npm to Manage Node.js Packages on a Linux Server.

- sudo apt-get install npm

- 
Since we installed Node.js from a package manager your binary may be called nodejs instead of node. Because npm relies on the fact that your Node.js binary will be called node, you just need to symlink it like so:
sudo ln -s /usr/bin/nodejs /usr/bin/node

You can read more about this issue on Github, and you can learn more about the symlinking from this StackExchange question.


When you are finished with these steps, you can continue with this guide.


# Step 1 — Installing Bower


Install Bower using npm:


```
sudo npm install bower -g


```


The -g switch is used to install Bower globally on your system.


Now that we have Bower installed we will continue with a practical example. In the next steps, we’ll


- Make a new Bower project
- Install Bootstrap with Bower
- Install AngularJS with Bower
- Serve the website via Nginx

At the end of this tutorial, in the Bower Reference section, you can read more about each of the Bower options.


# Step 2 — Preparing Project Directory


We’ll create our Bower project in the /usr/share/nginx/html directory so we can easily access our application as a website. This is Nginx’s default document root directory.


So, we need to change to this directory with the cd command:


```
cd /usr/share/nginx/html


```


By default, Nginx on Ubuntu 14.04 has one server block enabled by default. It is configured to serve documents out of the aforementioned /usr/share/nginx/html directory.


In our quick example we’ll use the default site.


For a production application, though, you should probably set up a server block for your specific domain.


Before we can do any work in the /usr/share/nginx/html directory, we have to give our sudo user rights to it.


Change the ownership of the directory with the following command:


```
sudo chown -R sammy:sammy /usr/share/nginx/html/


```


Instead of sammy you would use your own sudo user which you created in the prerequisite Ubuntu 14.04 Initial Server Setup tutorial.


# Step 3 — Initializing Bower Project


Now, inside the directory /usr/share/nginx/html, execute the following command to make a new Bower project:


```
bower init      


```


You will be asked a series of questions. For this quick example project you can just press ENTER to select all the defaults.


See a detailed breakdown of the answers below, marked in red:


```
Interactive? May bower anonymously report usage statistics to improve the tool over time? Yes
? name: BowerTest
? version: 0.0.0
? description: Testing Bower
? main file: index.html
? what types of modules does this package expose? Just press ENTER
? keywords: bower angular bootstrap
? authors: Nikola Brežnjak
? license: MIT
? homepage: http://bower.example.com
? set currently installed components as dependencies? Yes
? add commonly ignored files to ignore list? Yes
? would you like to mark this package as private which prevents it from being accidentally published to the registry? No

{
  name: 'BowerTest',
  version: '0.0.0',
  description: 'Testing Bower',
  main: 'index.html',
  keywords: [
    'bower',
    'angular',
    'bootstrap'
  ],
  authors: [
    'Nikola Brežnjak'
  ],
  license: 'MIT',
  homepage: 'http://bower.example.com',
  ignore: [
    '**/.*',
    'node_modules',
    'bower_components',
    'test',
    'tests'
  ]
}

? Looks good? Yes

```


A few notes about these options:


- Just to revisit the note from before, you don’t need to enter any of the options when running the bower init command for this example project
- In the What types of modules does this package expose? question, you can select or deselect the options by pressing SPACEBAR. Pressing ENTER confirms the selection. By default none are selected, and for this simple example we don’t need any of them. You can read more about them from the official GitHub issue
- For a production project, you would want to fill out the authors field and other settings so that other people know more about the project
- The homepage setting is used only to show your own website, and has nothing to do with the settings of the actual server where you’re running this application

Now you should have a bower.json file in your working directory (/usr/share/nginx/html/) with the JSON content shown in the output above.


# Step 4 — Installing AngularJS


AngularJS is a JavaScript framework for web applications. To install AngularJS with Bower, run the following command:


```
bower install angularjs --save


```


You can see the output of the command below:


```
[secondary_label Output]                                                
bower angularjs#*               cached git://github.com/angular/bower-angular.git#1.3.14
bower angularjs#*             validate 1.3.14 against git://github.com/angular/bower-angular.git#*
bower angularjs#*                  new version for git://github.com/angular/bower-angular.git#*
bower angularjs#*              resolve git://github.com/angular/bower-angular.git#*
bower angularjs#*             download https://github.com/angular/bower-angular/archive/v1.4.3.tar.gz
bower angularjs#*             progress received 0.1MB of 0.5MB downloaded, 20%
bower angularjs#*             progress received 0.1MB of 0.5MB downloaded, 24%
bower angularjs#*             progress received 0.5MB of 0.5MB downloaded, 98%
bower angularjs#*              extract archive.tar.gz
bower angularjs#*             resolved git://github.com/angular/bower-angular.git#1.4.3
bower angularjs#~1.4.3         install angularjs#1.4.3

angularjs#1.4.3 bower_components/angularjs

```


If you get slightly different output than the one shown above, that can be due to the fact that Bower caches packages for faster download and your package was installed from cache.


We now have AngularJS installed in the bower_components/angular directory (or possibly the bower_components/angularjs) directory, with the path to the minified version (which we will be using) being: bower_components/angular/angular.min.js.


# Step 5 — Installing Bootstrap


Bootstrap is a CSS framework. To install Bootstrap with Bower, run the following command:


```
bower install bootstrap --save


```


You can see the output of the command below:


```
Outputbower angularjs#~1.4.3          cached git://github.com/angular/bower-angular.git#1.4.3
bower angularjs#~1.4.3        validate 1.4.3 against git://github.com/angular/bower-angular.git#~1.4.3
bower bootstrap#*               cached git://github.com/twbs/bootstrap.git#3.3.5
bower bootstrap#*             validate 3.3.5 against git://github.com/twbs/bootstrap.git#*
bower jquery#>= 1.9.1           cached git://github.com/jquery/jquery.git#2.1.4
bower jquery#>= 1.9.1         validate 2.1.4 against git://github.com/jquery/jquery.git#>= 1.9.1
bower angularjs#~1.4.3         install angularjs#1.4.3
bower bootstrap#~3.3.5         install bootstrap#3.3.5
bower jquery#>= 1.9.1          install jquery#2.1.4

angularjs#1.4.3 js/angularjs

bootstrap#3.3.5 js/bootstrap
└── jquery#2.1.4

jquery#2.1.4 js/jquery

```


We now have Bootstrap installed in the bower_components/bootstrap directory with the path to the minified version (which we will be using) being: bower_components/bootstrap/dist/js/bootstrap.min.js for the JavaScript file and bower_components/bootstrap/dist/css/bootstrap.min.css for the CSS file.


Notice how jQuery was installed too, as it’s a dependency required by Bootstrap.


# Step 6 — Creating a Hello World Application


Inside the /usr/share/nginx/html/ folder edit, let’s replace the default index.html file with our own content:


```
mv /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html.orig


```


Open the file for editing:


```
vim /usr/share/nginx/html/index.html


```


You can enter this content exactly:


/usr/share/nginx/html/index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <title>Bootstrap 101 Template</title>

    <!-- Bootstrap -->
    <link href="bower_components/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container" ng-app>
        <form class="form-signin">
            <h2 class="form-signin-heading">What you type here:</h2>

            <input ng-model="data.input" type="text" class="form-control" autofocus>

            <h2 class="form-signin-heading">It will also be shown below:</h2>
            <input type="text" class="sr-only">{{data.input}}</label>
        </form>
    </div>

    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="bower_components/bootstrap/dist/js/bootstrap.min.js"></script>
    
    <script src="bower_components/angular/angular.min.js"></script>
</body>
</html>

```



Depending on how Bower installs AngularJS on your system, the path to the script might be bower_components/angularjs/angular.min.js rather than bower_components/angular/angular.min.js.

Now we have a simple Hello World type example application using Boostrap with AngularJS, running on Nginx.


(This is basically the example Signin Template from Bootstrap where the content inside the <body> tag has a simple form with two input fields.)


To view this example app, you should navigate in your browser to the IP of your Droplet; something like http://your_server_ip/. You should see something like the image below:





If you type something in the text box field, the exact same content will appear below, using the AngularJS two-way data binding.


If you don’t get any output, try restarting Nginx with the following command:


```
sudo service nginx restart


```


If you want to learn more about AngularJS, visit the official documentation at https://docs.angularjs.org/tutorial. If you wish to learn more about Bootstrap, visit the official documentation at http://getbootstrap.com/getting-started/.


If you want to be able to access your web server via a domain name, instead of its public IP address, purchase a domain name then follow these tutorials:


- How To Set Up a Host Name with DigitalOcean
- How to Point to DigitalOcean Nameservers From Common Domain Registrars

# Bower Reference


Now that we’ve gone through a practical example with Bower, let’s look at some of its general capabilities.


## Installing Packages


To install a package (for example, AngularJS or Bootstrap) we would need to run the following command:


```
bower install package


```


Instead of package, enter the exact name of the package you want to install. The package can be a GitHub shorthand, a Git endpoint, a URL, and a lot more.


You can also install a specific version of a certain package.


Find out more about all the available options for installing via Bower’s official documentation on installing.


## Searching Packages


You can search for packages via this online tool or by using the Bower CLI. To search for packages using the Bower CLI, use the following command:


```
bower search package


```


For example, if we would like to install AngularJS, but we’re unsure about the correct package name, or we would like to see all the available packages for AngularJS, we can execute the following command:


```
bower search angularjs


```


We would get an output similar to this:


```
OutputSearch results:
    angularjs-nvd3-directives git://github.com/cmaurer/angularjs-nvd3-directives.git
    AngularJS-Toaster git://github.com/jirikavi/AngularJS-Toaster.git
    angularjs git://github.com/angular/bower-angular.git
    angular-facebook git://github.com/Ciul/Angularjs-Facebook.git
    angularjs-file-upload git://github.com/danialfarid/angular-file-upload-bower.git
    angularjs-rails-resource git://github.com/FineLinePrototyping/dist-angularjs-rails-resource
    angularjs-geolocation git://github.com/arunisrael/angularjs-geolocation.git

```


In order to install AngularJS you would then simply execute the following command:


```
bower install angularjs


```


## Saving Packages


When starting a project with Bower, it’s standard to start with running the init command:


```
bower init


```


This will guide you through creating a bower.json file, which Bower uses for project configuration. The process might look like this:


```
Output? name: howto-bower
? version: 0.0.0
? description:
? main file:
? what types of modules does this package expose?
? keywords:
? authors: Nikola Breznjak <nikola.breznjak@gmail.com>
? license: MIT
? homepage: https://github.com/Hitman666/jsRockstar
? set currently installed components as dependencies? Yes
? add commonly ignored files to ignore list? Yes
? would you like to mark this package as private which prevents it from being accidentally published to the registry? No

{
  name: 'howto-bower',
  version: '0.0.0',
  homepage: 'https://github.com/Hitman666/jsRockstar',
  authors: [
    'Nikola Breznjak <nikola.breznjak@gmail.com>'
  ],
  license: 'MIT',
  ignore: [
    '**/.*',
    'node_modules',
    'bower_components',
    'test',
    'tests'
  ]
}

? Looks good? Yes

```


Now, if you install any of the packages using the --save switch, they will be saved to the bower.json file, in the dependencies object. For example, if we installed AngularJS with the following command:


```
bower install angularjs --save


```


Then our bower.json file would look something like this (note the dependencies object):


bower.json
```
{
  "name": "howto-bower",
  "version": "0.0.0",
  "homepage": "https://github.com/Hitman666/jsRockstar",
  "authors": [
    "Nikola Breznjak <nikola.breznjak@gmail.com>"
  ],
  "license": "MIT",
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "dependencies": {
    "angularjs": "~1.4.3"
  }
}

```


## Uninstalling Packages


To uninstall a Bower package, simply run the following command:


```
bower uninstall package


```


This will uninstall a package from your bower_component directory (or any other directory you’ve defined in the .bowerrc file (more about configuration in the next section).


## Configuring Bower with .bowerrc


To configure Bower, you have to create a file called .bowerrc. (Notice the dot - it means it’s a hidden file in a Linux environment.)


Create the .bowerrc file in the root directory of your project (alongside the bower.json file). You can have one .bowerrc file per project, with different settings.


Bower lets you configure many options using this file, which you can read more about in the configuration options from the official documentation.


One useful option is the directory option, which lets you to customize the folder in which Bower saves all its packages.


To set this simple option, create a .bowerrc file that looks like this:


.bowerrc
```
{
  "directory": "js/"  
}

```


# Conclusion


After completing this tutorial, you should know how to use Bower to install the dependencies for a simple AngularJS application.


You should also have an idea of how to use Bower for your own custom applications.


