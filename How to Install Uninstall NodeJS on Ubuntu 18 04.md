# How to Install Uninstall NodeJS on Ubuntu 18 04

```Ubuntu``` ```UNIX/Linux```

NodeJS is a JavaScript framework that allows you to build fast network applications with ease. In this guide, we delve in and see how you can How to install NodeJS on Ubuntu 18.04.


# Step 1: Adding the NodeJS PPA to Ubuntu 18.04


To start off, add the NodeJS PPA to your system using the following commands.


```
sudo apt-get install software-properties-common

```


Sample Output  Next, add the NodeJS PPA.


```
curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -

```


Sample Output  Great! In our next step, we are going to run the command for installing NodeJS.


# Step 2: Install NodeJS on Ubuntu


After successfully adding the NodeJS PPA, It’s time now to install NodeJS using the command below.


```
sudo apt-get install nodejs

```


Sample Output  This command not only installs NodeJS but also NPM (NodeJS Package Manager) and other dependencies as well.


# Step 3: Verfiying the version of NodeJS and NPM


After successful installation of NodeJS, you can test the version of NodeJS using the simple command below.


```
node -v

```


Sample Output  For NPM, run


```
npm -v

```


Sample Output 


# Step 4: Creating a Web Server demonstration


This is an optional step that you can use to test if NodeJS is working as intended. We are going to create a web server that displays the text “Congratulations! node.JS has successfully been installed !” Let’s create a NodeJS file and call it nodeapp.js


```
vim nodeapp.js

```


Add the following content


```
var http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Congratulations! node.JS has successfully been installed !\n');
}).listen(3000, " server-ip);
console.log('Server running at https://server-ip:3000/');

```


Save and exit the text editor Start the application using the command below


```
node nodeapp.js

```


Sample Output  This will display the content of the application on port 3000. Ensure the port is allowed on the firewall of your system.


```
ufw allow 3000/tcp
ufw reload

```


Now open your browser and browse the server’s address as shown


```
https://server-ip:3000

```





# Uninstall NodeJS from Ubuntu


If you wish to uninstall NodeJS from your Ubuntu system, run the command below.


```
sudo apt-get remove nodejs

```


The command will remove the package but retain the configuration files. To remove both the package and the configuration files run:


```
sudo apt-get purge nodejs

```


As a final step, you can run the command below to remove any unused files and free up the disk space


```
sudo apt-get autoremove

```


Great! We have successfully installed and tested the installation of NodeJS. We also learned how to uninstall NodeJS from Ubuntu and clean up space.


