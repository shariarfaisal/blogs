# How To Install and Use Bower on your VPS

```Ubuntu``` ```Node.js``` ```Git```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


# Introduction


Bower is a package management solution for front-end packages such as javascript libraries and css frameworks. It runs on Node.js and uses Git to fetch and install most packages. You can find a list of all the packages you can install using Bower.


In this tutorial we will install Bower on a VPS running Ubuntu 12.04. You should already have your own virtual private server, as well as Node.js and NPM (Node Packaged Modules) installed. If you don’t, follow the steps outlined in this tutorial to get you set up.


# Install Bower on your VPS


First thing is to install Git. For more information about this step, there is a great tutorial that you can follow-- basically, you can install it with the following command:


```
sudo apt-get install git-core
```


To install Bower run the following command (assuming Node.js and NPM are installed):


```
npm install -g bower
```


Now that Bower is installed on your VPS, you can run the following for more information about available commands:


```
bower help
```


If you are using the root user, please note that you need to append the following option to the command in order for it to run: --allow-root:


```
bower help --allow-root
```


# Using Bower in your Project


Let’s see how we can use Bower to manage package dependency for your project.


Navigate to your web server’s root directory:


```
cd /var/www
```


Create a new folder and navigate inside:


```
mkdir project
cd project
```


This will be the project’s root directory. Let’s say it will use Twitter Bootstrap. We can use it by downloading the zip file and unpacking the files on the cloud server. However with Bower, we can turn our project into a package, declare Bootstrap as a dependency, and have Bower fetch it for us. For this we need a file called bower.json that will list the dependencies. In here, we will include Bootstrap.


So create a new file called bower.json in the root of your project:


```
nano /var/www/project/bower.json
```


and paste in the following:


```
{
  "name": "my-project",
  "dependencies": {
    "bootstrap": ">= 3.0.0"
  }
}
```


Save the file and exit. Our project now has Bootstrap 3.0 (or higher) set as a dependency. To have Bower fetch and install it, as well as all other dependencies you may want to declare, run the following command (remember to add --allow-root if you are operating as the root user):


```
bower install
```


You’ll now notice that a new folder was created in your project called bower_components where you’ll see two new folders: bootstrap and jquery (the latter is also installed because Bootstrap requires jQuery). So that’s pretty cool.


If you want to see which packages you have already installed on your VPS in the project, run the following command:


```
bower list
```


If you want this list to contain the paths to the files you need to include in the application you are building (i.e. the relevant .js or .css files), add the --paths option:


```
bower list --paths
```


Another way to install a package is by using the install command and specifying the actual package you want installed on your VPS. So let’s create another project folder and install Bootstrap there:


```
cd /var/www
mkdir project2
cd project2
```


Now that we are in the new project folder, run the following command:


```
bower install bootstrap
```


You’ll see the same effect happen and in a much faster way. Note that you can use Bower in this way to install virtually anything at the end of a git endpoint or URL not necessarily found on the Bower components website. So for instance, running the same command but replacing bootstrap with a git address would fetch that git project:


```
bower install git://github.com/someone/some-package.git
```


Or even a zip file that would get extracted into the folder:


```
bower install http://www.example.com/package.zip
```


To uninstall a package you had installed locally for your project, you can run the following command:


```
bower uninstall bootstrap
```


You should, of course, replace bootstrap with the name of the project you’d like uninstalled.


Article Submitted by: Danny
