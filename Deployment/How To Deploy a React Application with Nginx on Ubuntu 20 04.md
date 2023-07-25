# How To Deploy a React Application with Nginx on Ubuntu 20 04

```JavaScript``` ```React``` ```Ubuntu``` ```Deployment```

The author selected Creative Commons to receive a donation as part of the Write for DOnations program.


## Introduction


You can quickly deploy React applications to a server using the default Create React App build tool. The build script compiles the application into a single directory containing all of the JavaScript code, images, styles, and HTML files. With the assets in a single location, you can deploy to a web server with minimal configuration.


In this tutorial, you’ll deploy a React application on your local machine to an Ubuntu 20.04 server running Nginx. You’ll build an application using Create React App, use an Nginx config file to determine where to deploy files, and securely copy the build directory and its contents to the server. By the end of this tutorial, you’ll be able to build and deploy a React application.


# Prerequisites


- 
On your local machine, you will need a development environment running Node.js; this tutorial was tested on Node.js version 10.22.0 and npm version 6.14.6. To install this on macOS or Ubuntu 20.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 20.04.

- 
One Ubuntu 20.04 server for deployment, set up by following this initial server setup for Ubuntu 20.04 tutorial, including a sudo-enabled non-root user, a firewall, and SSH access from your local machine. To gain SSH access on a DigitalOcean Droplet, read through How to Connect to Droplets with SSH.

- 
A registered domain name. This tutorial will use your_domain throughout. You can purchase a domain name from Namecheap, get one for free with Freenom, or use the domain registrar of your choice.

- 
Both of the following DNS records set up for your server. If you are using DigitalOcean, please see our DNS documentation for details on how to add them.

An A record with your_domain pointing to your server’s public IP address.
An A record with www.your_domain pointing to your server’s public IP address.


- An A record with your_domain pointing to your server’s public IP address.
- An A record with www.your_domain pointing to your server’s public IP address.
- 
Nginx installed by following How To Install Nginx on Ubuntu 20.04. Be sure that you have a server block for your domain. This tutorial will use /etc/nginx/sites-available/your_domain as an example.

- 
It is recommended that you also secure your server with an HTTPS certificate. You can do this with the How To Secure Nginx with Let’s Encrypt on Ubuntu 20.04 tutorial.

- 
You will also need a basic knowledge of JavaScript, HTML, and CSS, which you can find in our How To Build a Website With HTML series, How To Build a Website With CSS series, and in How To Code in JavaScript.


# Step 1 — Creating a React Project


In this step, you’ll create an application using Create React App and build a deployable version of the boilerplate app.


To start, create a new application using Create React App in your local environment. In a terminal, run the command to build an application. In this tutorial, the project will be called react-deploy:


```
npx create-react-app react-deploy


```


The npx command will run a Node package without downloading it to your machine. The create-react-app script will install all of the dependencies needed for your React app and will build a base project in the react-deploy directory. For more on Create React App, check out out the tutorial How To Set Up a React Project with Create React App.


The code will run for a few minutes as it downloads and installs the dependencies. When it is complete, you will receive a success message. Your version may be slightly different if you use yarn instead of npm:


```
OutputSuccess! Created react-deploy at your_file_path/react-deploy
Inside that directory, you can run several commands:

  npm start
    Starts the development server.

  npm build
    Bundles the app into static files for production.

  npm test
    Starts the test runner.

  npm eject
    Removes this tool and copies build dependencies, configuration files
    and scripts into the app directory. If you do this, you can’t go back!

We suggest that you begin by typing:

  cd react-deploy
  npm start

Happy hacking!

```


Following the suggestion in the output, first move into the project folder:


```
cd react-deploy


```


Now that you have a base project, run it locally to test how it will appear on the server. Run the project using the npm start script:


```
npm start


```


When the command runs, you’ll receive output with the local server info:


```
OutputCompiled successfully!

You can now view react-deploy in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://192.168.1.110:3000

Note that the development build is not optimized.
To create a production build, use npm build.

```


Open a browser and navigate to http://localhost:3000. You will be able to access the boilerplate React app:





Stop the project by entering either CTRL+C or ⌘+C in a terminal.


Now that you have a project that runs successfully in a browser, you need to create a production build. Run the create-react-app build script with the following:


```
npm run build


```


This command will compile the JavaScript and assets into the build directory. When the command finishes, you will receive some output with data about your build. Notice that the filenames include a hash, so your output will be slightly different:


```
OutputCreating an optimized production build...
Compiled successfully.

File sizes after gzip:

  41.21 KB  build/static/js/2.82f639e7.chunk.js
  1.4 KB    build/static/js/3.9fbaa076.chunk.js
  1.17 KB   build/static/js/runtime-main.1caef30b.js
  593 B     build/static/js/main.e8c17c7d.chunk.js
  546 B     build/static/css/main.ab7136cd.chunk.css

The project was built assuming it is hosted at /.
You can control this with the homepage field in your package.json.

The build folder is ready to be deployed.
You may serve it with a static server:

  serve -s build

Find out more about deployment here:

  https://cra.link/deployment

```


The build directory will now include compiled and minified versions of all the files you need for your project. At this point, you don’t need to worry about anything outside of the build directory. All you need to do is deploy the directory to a server.


In this step, you created a new React application. You verified that the application runs locally and you built a production version using the Create React App build script. In the next step, you’ll log onto your server to learn where to copy the build directory.


# Step 2 — Determining Deployment File Location on your Ubuntu Server


In this step, you’ll start to deploy your React application to a server. But before you can upload the files, you’ll need to determine the correct file location on your deployment server. This tutorial uses Nginx as a web server, but the approach is the same with Apache. The main difference is that the configuration files will be in a different directory.


To find the directory the web server will use as the root for your project, log in to your server using ssh:


```
ssh username@server_ip


```


Once on the server, look for your web server configuration in /etc/nginx/sites-enabled. There is also a directory called sites-allowed; this directory includes configurations that are not necessarily activated. Once you find the configuration file, display the output in your terminal with the following command:


```
cat /etc/nginx/sites-enabled/your_domain


```


If your site has no HTTPS certificate, you will receive a result similar to this:


```
Outputserver {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


If you followed the Let’s Encrypt prerequisite to secure your Ubuntu 20.04 server, you will receive this output:


```
Outputserver {

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}
server {
    if ($host = www.your_domain) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = your_domain) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80;
        listen [::]:80;

        server_name your_domain www.your_domain;
    return 404; # managed by Certbot




}


```


In either case, the most important field for deploying your React app is root. This points HTTP requests to the /var/www/your_domain/html directory. That means you will copy your files to that location. In the next line, you can see that Nginx will look for an index.html file. If you look in your local build directory, you will see an index.html file that will serve as the main entry point.


Log off the Ubuntu 20.04 server and go back to your local development environment.


Now that you know the file location that Nginx will serve, you can upload your build.


# Step 3 — Uploading Build Files with scp


At this point, your build files are ready to go. All you need to do is copy them to the server. A quick way to do this is to use scp to copy your files to the correct location. The scp command is a secure way to copy files to a remote server from a terminal. The command uses your ssh key if it is configured. Otherwise, you will be prompted for a username and password.


The command format will be scp files_to_copy username@server_ip:path_on_server. The first argument will be the files you want to copy. In this case, you are copying all of the files in the build directory. The second argument is a combination of your credentials and the destination path. The destination path will be the same as the root in your Nginx config: /var/www/your_domain/html.


Copy all the build files using the * wildcard to /var/www/your_domain/html:


```
scp -r ./build/* username@server_ip:/var/www/your_domain/html


```


When you run the command, you will receive output showing that your files are uploaded. Your results will be slightly different:


```
Outputasset-manifest.json                                                                                          100% 1092    22.0KB/s   00:00
favicon.ico                                                                                                  100% 3870    80.5KB/s   00:00
index.html                                                                                                   100% 3032    61.1KB/s   00:00
logo192.png                                                                                                  100% 5347    59.9KB/s   00:00
logo512.png                                                                                                  100% 9664    69.5KB/s   00:00
manifest.json                                                                                                100%  492    10.4KB/s   00:00
robots.txt                                                                                                   100%   67     1.0KB/s   00:00
main.ab7136cd.chunk.css                                                                                      100%  943    20.8KB/s   00:00
main.ab7136cd.chunk.css.map                                                                                  100% 1490    31.2KB/s   00:00
runtime-main.1caef30b.js.map                                                                                 100%   12KB  90.3KB/s   00:00
3.9fbaa076.chunk.js                                                                                          100% 3561    67.2KB/s   00:00
2.82f639e7.chunk.js.map                                                                                      100%  313KB 156.1KB/s   00:02
runtime-main.1caef30b.js                                                                                     100% 2372    45.8KB/s   00:00
main.e8c17c7d.chunk.js.map                                                                                   100% 2436    50.9KB/s   00:00
3.9fbaa076.chunk.js.map                                                                                      100% 7690   146.7KB/s   00:00
2.82f639e7.chunk.js                                                                                          100%  128KB 226.5KB/s   00:00
2.82f639e7.chunk.js.LICENSE.txt                                                                              100% 1043    21.6KB/s   00:00
main.e8c17c7d.chunk.js                                                                                       100% 1045    21.7KB/s   00:00
logo.103b5fa1.svg                                                                                            100% 2671    56.8KB/s   00:00

```


When the command completes, you are finished. Since a React project is built of static files that only need a browser, you don’t have to configure any further server-side language. Open a browser and navigate to your domain name. When you do, you will find your React project:





In this step, you deployed a React application to a server. You learned how to identify the root web directory on your server and you copied the files with scp. When the files finished uploading, you were able to view your project in a web browser.


# Conclusion


Deploying React applications is a quick process when you use Create React App. You run the build command to create a directory of all the files you need for a deployment. After running the build, you copy the files to the correct location on the server, pushing your application live to the web.


If you would like to read more React tutorials, check out our React Topic page, or return to the How To Code in React.js series page.


