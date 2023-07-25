# How To Setup a Node js Development Environment on a VPS for a Team

```Ubuntu``` ```Node.js``` ```Nginx```

# Introduction



In this tutorial, we will setup a Node.js development environment, which allows you to include rapidly new team members into the development process of your Node.js applications. This method can also be applied if a developer wants to create several versions of an application simultaneously.


This method is based on Node.js interacting with Nginx over Unix sockets instead of ports. Let’s assume that you have your development versions of the application at login.dev.nodeapp.com. In addition, we will hold the sockets for every developer in the /tmp directory, like /tmp/login.dev.nodeapp.com.sock.


# Requirements



You will need to have Nginx and Node.js installed. Additionally, we will assume that you already have a domain (for example nodeapp.com) linked to your VPS. Note: you should also setup wildcard CNAME record for your domain. There are already well written tutorials about these topics on DigitalOcean:


- How To Install Nginx on Ubuntu 
- How To Install Node.js with NVM on a VPS.
- How To Set Up a Host Name with DigitalOcean

# Setting Up Nginx



We should create a new Nginx configuration file /etc/nginx/sites-available/dev.nodeapp.com which contains:


```
server {

    listen 80; 

    server_name ~^(?<login>[a-z]+)\.dev\.nodeapp\.com$;

    location / {

        proxy_pass http://unix:/tmp/$login.dev.nodeapp.com.sock:$uri$is_args$args;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }   
}

```


Link this configuration to sites-enabled folder and restart Nginx:


```
ln -nfs /etc/nginx/sites-available/dev.nodeapp.com /etc/nginx/sites-enabled/dev.nodeapp.com
/etc/init.d/nginx restart

```


Now Nginx is ready to accept user requests and guide them to the developer’s app copy, depending on URL. For example:


```
http://ivan.dev.nodeapp.com -> /tmp/ivan.dev.nodeapp.com.sock
http://anna.dev.nodeapp.com -> /tmp/anna.dev.nodeapp.com.sock

```


# Modifying Node.js App



We will use a minimal webserver example from the Node.js, but the same modifications can be applicable for any Node.js server (like express).


The thing is, we need to change the default port-listening behavior into socket-listening:


```
var fs = require('fs');
var http = require('http');

var mask = process.umask(0);
var socket = '/tmp/' + process.env.USER + '.dev.nodeapp.com.sock';

if (fs.existsSync(socket)) {
	fs.unlinkSync(socket);
}
    
http.createServer(function (req, res) {
	res.writeHead(200, {'Content-Type': 'text/plain'});
  	res.end('Hello World\n');
}).listen(socket, function() {
	if (mask) {
		process.umask(mask);
		mask = null;
	}
});

console.log('Server running at ' + socket);

```


Now you can run your application node app.js and access it on http://yourlogin.dev.nodeapp.com.


Note: when Node.js starts listening for sockets, it creates a specified file. But if the socket file already exists, Node.js will fail to start listening. So we should make sure we remove the socket from the previous run.


The other thing is, we should create a socket with full access for all, so Nginx will be able to use it. It’s fine for development, but might not be good for production.


And that’s it, congratulations! All you need now to introduce a new developer to create user on your VPS.


<div class=“author”>Submitted by: <a href=“https://github.com/artjock”>Artur Burtsev</a></div>


