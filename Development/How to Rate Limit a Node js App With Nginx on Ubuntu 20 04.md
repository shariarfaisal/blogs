# How to Rate Limit a Node js App With Nginx on Ubuntu 20 04

```Development``` ```Nginx```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Rate limiters are essential for production environments since they help prevent malicious brute force attacks from accessing sensitive resources that are usually placed behind web logins. They also play an active role in stopping Distributed Denial of Service (DDoS) attacks. Web servers like Nginx provide native tooling to implement rate limiting and they use fewer resources to process requests.


In this tutorial, you will set up Nginx to limit the amount of traffic a Node.js simplified login will receive. You will configure Nginx to define a maximum amount of requests over a period of time, test the rate limiter, and customize the 429 error page displayed to users.


# Prerequisites


To complete this tutorial, you will need:


- 
An Ubuntu 20.04 server running Node.js with 1 CPU and 1GB of RAM configured with a non-root user with sudo privileges and a firewall. Note that Nginx rate limiters will benefit from having more available CPU over RAM. To set up a non-root user and configure a firewall, please see the guide, Initial Server Setup with Ubuntu 20.04.

- 
A Node.js app with application manager PM2 set up on your server, which you can do by following the tutorial, How To Set Up a Node.js Application for Production on Ubuntu 20.04.

- 
Nginx installed with SSL certificates set up, which you can do by following the tutorials, How To Install Nginx on Ubuntu 20.04 and How To Secure Nginx with Let’s Encrypt on Ubuntu 20.04.

- 
A registered domain. You can get one for free from Freenom or use an existing domain name you own.

- 
A DNS A record with your application and domain name pointing to your Nginx rate-limited application’s IPv4 address. Follow the guide How To Point to DigitalOcean Nameservers From Common Domain Registrars to set this up. This tutorial will use login.your_domain.com.

- 
Some familiarity with HTML and CSS will be useful to build the login and error web pages. If you want to learn more about HTML and CSS, consider visiting How To Build a Website with HTML and How To Build a Website With CSS to learn more.


# Step 1 — Setting Up the Node.js App


Rate limiters usually sit in front of an API or application. In this tutorial, you will use a Node.js application that will be a login situated behind a rate limiter.


As a first step, you will install the project’s dependencies, create a login form (with optional styling), and test that the application is running. You’ll begin by creating a new project folder where source files and npm dependencies will be stored.


Create a new folder for the application files using the following command:


```
sudo mkdir /srv/rate-limited-login


```


Assign ownership of the folder to your non-root user, sammy, which you created in the prerequisite tutorial:


```
sudo chown sammy /srv/rate-limited-login


```


Since sammy now owns the folder, you won’t need to use sudo every time for creating and modifying files inside the directory.


Switch to the /srv/rate-limited-login directory you created:


```
cd /srv/rate-limited-login


```


Initialize a new npm repository:


```
npm init


```


Type the information npm asks you (or press enter to leave the default values).


Before moving on to writing any code, you will need to install and save Express as an npm dependency, to help route HTTP requests within the Node.js application:


```
npm install express --save


```


Express will be installed under the node_modules folder, managed by npm. Using the --save flag will add Express to the package.json configuration file.


With Express set up as a dependency, you will now create an HTTP route that will be put behind a rate limit later on in this article. The project folder, /srv/rate-limited-login, will contain the source code that will enable Express to handle HTTP requests for login purposes as well as serving the website containing the login form itself. /srv/rate-limited-login will contain three files:


- index.js will host code related to the Express endpoint to serve static files and handle POST / requests to your login application route.
- public/index.html will contain the HTML login form.
- public/login.css will have login form CSS styles (optional).

First, create and edit the index.js file:


```
nano index.js 


```


Add the code below to create an application to serve the HTTP route POST /  and static files from the folder public on port 8080:


/srv/rate-limited-login/index.js
```
const express = require('express');
const path = require('path');
const app = express();
const listeningPort = 8080;

app.use('/', express.static(path.join(__dirname, 'public')));

app.post('/', (request, response) => {
    response.send("Logging in...");
});

app.listen(listeningPort, () => {
    console.log(`Rate Limited Login listening on port ${listeningPort}!`);
});

```


Express will process HTTP requests whenever index.js is executed using Node.js. Every file that is stored under /srv/rate-limited-login/public will be reachable through the web server and HTTP POST requests to http://localhost:8080 will return a website with the phrase Logging in….


Next, create a public folder that will host the HTML and CSS for the login form:


```
mkdir /srv/rate-limited-login/public


```


Move to the new folder:


```
cd /srv/rate-limited-login/public


```


Create a file named index.html:


```
nano index.html


```


Add an HTML form to the file for your login:


/srv/rate-limited-login/public/index.html
```
<html>

    <head>
        <title>Login</title>
        <link href="https://fonts.googleapis.com/css2?family=Quicksand:wght@600&display=swap" rel="stylesheet">
        <link rel="stylesheet" href="login.css">
    </head>

    <body>
        <div id="login-container">
            <h1>Rate Limited Login</h1>
            <form id="login-form" method="POST">
                <label for="user">Username</label>
                <input type="text" name="user" id="user">
                <label for="password">Password</label>
                <input type="password" name="password" id="password">
                <button type="submit">Login</button>
            </form>
        </div>
    </body>

</html>

```


Now that you have created a login form, every time a user completes it and presses the submit button, a request will be fired towards the POST / route that you previously configured within the index.js file.


If you would like to add styling, you can create a login.css file:


```
nano login.css


```


Using the following code, you can add a gradient as background, change the default font, center the text, and allow the body to take 100% of the browser height:


/srv/rate-limited-login/public/login.css
```
body {
    background: linear-gradient(132deg, rgba(184,231,209,1) 35%, rgba(0,255,190,1) 100%);
    font-family: 'Quicksand', sans-serif;
    overflow: hidden;
    height:100%;
    text-align: center;
}

```


You can increase the size of the main header text:


/srv/rate-limited-login/public/login.css
```
...
h1 {
    font-size: 1.5em;
}
...

```


You can add a gradient as background of the login form container while adding a width, padding, and margin:


/srv/rate-limited-login/public/login.css
```
...
#login-container {
    background: linear-gradient(132deg, rgba(225,255,249,1) 35%, rgba(182,230,231,1) 100%);
    margin: 2em auto;
    padding: 1em;
    width: 25em;
}
...

```


To position labels on the left side of the form, you can override body-centered text:


/srv/rate-limited-login/public/login.css
```
...
#login-form label {
    display: block;
    font-size: 1.1em;
    margin-left: 1.2em;
    text-align: left;
}
...

```


And finally, you can define a border, height of line, width, and spacing for form fields:


/srv/rate-limited-login/public/login.css
```
...
#login-form input, button{
    border: 0.05em solid rgb(184,231,209);
    font-size: 1em;
    line-height: 1.7em;
    margin: 1em 1em 1.5em 1em;
    padding: 0.2em;
    width: 90%;
}
...

```


You now have all the necessary files to run the login application.


Start the application using this command:


```
node /srv/rate-limited-login/index.js 


```


The output will look similar to this:


```
OutputRate Limited Login listening on port 8080!

```


To stop the application, press CTRL+C in Linux and Windows users or CMD+C for Mac environments.


The application now runs on port 8080. It will listen to POST / HTTP requests and will serve the files under the /public folder to /. The POST / endpoint will return a string Logging in..., while GET / will serve the login HTML form that you will be creating next.


You now have the application running. However, it will stop running if you close your current session to the server. To keep your Node.js instance running even after you log out from the server, you will use PM2, a process manager for Node.js applications.


In the prerequisites for this tutorial, you created a Node.js application that runs PM2. Now, you will use PM2 to manage the login application so that it can run even after you log out from the server.


Start the application using PM2:


```
pm2 start /srv/rate-limited-login/index.js --name rate-limited-login 


```


With the --name flag, you can give the PM2 process a label (in this case, rate-limited-login).


Add the new application to your existing list of saved PM2 applications that will run after booting up:


```
pm2 save


```


To test that the application is running, make a POST request to it:


```
curl -X POST http://localhost:8080/


```


If everything is configured correctly, the output should look similar to this:


```
OutputLogging in...

```



Note: Bear in mind that a real-world login would have additional security that is not being taken into consideration in this tutorial. You can learn more about authentication security guidelines by following OWASP Authentication Cheat Sheet. Third-party libraries such as Passport can also be used for authentication purposes and they already have built-in security mechanisms to help prevent malicious attacks.

Now that you have set up the application, you’re ready to set up the rate limiter.


# Step 2 — Setting up the Rate Limiter


At this point, Nginx redirects traffic to the login application. In this step, you will implement a rate limit using three Nginx directives: limit_req_zone, limit_req, and limit_req_status.


The first, limit_req_zone, specifies the criteria to limit requests, the amount of memory you are giving Nginx to keep track of previous requests’ data, and the rate limit over a period of time.


Rate limiters are set up in the same configuration blocks that you previously created in the prerequisites section of this tutorial. Open the block configuration file for login.your_domain.com:


```
sudo nano /etc/nginx/sites-available/login.your_domain.com


```


Add a new limit_req_zone Nginx directive:


/etc/nginx/sites-available/login.your_domain.com
```
...
limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/s
server {
...

```


The preceding line indicates that Nginx’s requests should be based on IP addresses at a rate of 5 requests per second and should allocate 10 megabytes to maintain a record of previous IPs.


To limit requests based on other criteria (rather than using client’s IP address as a key), you can use  $geoip_country_code to allow requests by country. You can also sort traffic based on other Nginx variables.


The next directive you’ll use is limit_req, which describes the scope of where the rate limit should be applied and if there are additional settings such as bursts and delays that may be beneficial for the application. (In a later step, you will learn more about these extra parameters.)


Once again, open the block configuration file for login.your_domain.com:


```
sudo nano /etc/nginx/sites-available/login.your_domain.com


```


Define the context where the rate limit should be executed by adopting the limit_req Nginx directive:


/etc/nginx/sites-available/login.your_domain.com
```
...
server {
    ...
    location / {
    limit_req zone=login_limit;
    ....
}
...

```


Depending on your application needs, limit_req directive can live inside a server or location context in an Nginx configuration block file.


The last directive you’ll add is limit_req_status. By default, Nginx will serve a 503 Service Unavailable HTTP code when a user triggers a rate limiter threshold. The message may be misleading as it can give the impression that the server is not working as expected when, in fact, it is the client who is making more requests than Nginx is set up to allow. A more transparent alternative is to use the directive limit_req_status to trigger the rate limit and return a 429 Too Many Requests HTTP code.


Open the block configuration file for login.your_domain.com:


```
sudo nano /etc/nginx/sites-available/login.your_domain.com


```


Change the default HTTP response code by adding the Nginx directive: limit_req_status:


/etc/nginx/sites-available/login.your_domain.com
```
...
limit_req_status 429
server {
...

```


After adding the limit_req_zone, limit_req and limit_req_status Nginx directives to the block configuration file for login.your_domain.com, you will have something similar to the following content:


/etc/nginx/sites-available/login.your_domain.com
```
limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/s;
limit_req_status 429;

server {
        root /var/www/login.your_domain.com/html;
        index index.html index.htm index.nginx-debian.html;
        server_name login.your_domain.com;

        location / {
          limit_req zone=login_limit;
          proxy_pass http://localhost:8080;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection 'upgrade';
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/login.your_domain.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/login.your_domain.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = login.your_domain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    listen [::]:80;
    server_name login.your_domain.com;
    return 404; # managed by Certbot

}

```


Finally, reload your current Nginx configuration:


```
sudo service nginx reload

```


You could test that rate limits are in place by visiting login.your_domain.com and refreshing many times until you get a 429 error page. However, this method may not scale when you are trying to trigger dozens of requests per second instead of just a few. In the next step, you will use a load testing tool to effectively test when rate limiters start to kick in.


# Step 3 — Verifying Rate Limit Works Using Apache Benchmark


A browser can only load a page per tab, which is not ideal for testing concurrent connections and seeing when a rate limit gets triggered. Instead, you can use load testing tools such as k6 or Apache Benchmark. In this step, you will use Apache Benchmark because its CLI requires no scripting code to hit a single page.


Install Apache Benchmark with the following command:


```
sudo apt install apache2-utils -y


```


To run the test, fire up 100 concurrent connections and a total of 100 requests to your login:


```
ab -c 100 -n 100 https://login.your_domain.com/


```


After Apache Benchmark finishes with the test, it will display many data points. To evaluate if rate limiters are working as expected, focus on the following line:


```
Output...
Time taken for tests:   0.180 seconds 
...
Non-2xx responses:      99
...

```


Setting a rate limit of 5 requests per second means that, at most, a request every 200ms will be allowed.


To check if these requests are indeed errors with a 429 status, display Nginx’s error log while running the load test in another terminal.


Open a new terminal and run the following command:


```
sudo tail -f /var/log/nginx/error.log


```


The output will look similar to this:


```
Output[error] 17133#17133: *6346 limiting requests, excess: 0.540 by zone "login_limit", client: 203.0.113.0, server: login.your_domain.com, request: "GET / HTTP/1.0", host: "login.your_domain.com"

```


Highlighting some of the output entries above, excess represents the number of requests per milliseconds after passing the rate-limit threshold that the request embodies.


login_limit represents the rate limit group that you specified within your Nginx block configuration file. If you’d happen to have two or more rate limiters with different settings, you’d set up different zones to distinguish them.


client tells you the visitor’s IP address, while host and request represent the domain that was accessed and what kind of request was made, respectively.


During the next steps you can keep this terminal window open to see how changes to the rate limiter’s behavior is affected over configuration changes.


You can now be certain that the rate limits are functioning as expected. In the next step, you will use additional configuration to tweak Nginx for more realistic workloads.


# Step 4 — Setting up Bursts and Delays


As mentioned previously, having a rate limit of 5-requests-per-second implies that a request every 200ms will be allowed. This may be inadequate for an application like a website with assets such as images, HTML, CSS, and JS files that may make requests more often.


One solution provided by Nginx is having a burst queue that will take in a number of extra requests before returning an error. However, relying on this queue to allow multiple requests in a short period can cause performance issues, as requests are queued to be processed, but not simultaneously. To overcome this issue, you can configure a nodelay directive to prevent Nginx from having to wait between each request to be processed before starting the next one.


There is also a hybrid approach using delay in conjunction with burst. This allows you to have a burst queue so that requests don’t get dropped by Nginx, but only allow a small fraction of those queued items to be processed simultaneously.


In this step, you will explore all of these approaches in order to see how they can be combined to get different behaviors while configuring your rate limiter. You will set up an Nginx burst queue, and you will also use more advanced configuration allowing a limited number of requests to go through the rate limiter without time penalties. This enables users to have a better overall experience accessing your website.


You’ll begin by setting up a burst queue. Adding burst to the limit_req directive lets you define how many requests to queue for later processing. This way, you can tell Nginx to accept up to 10 additional asset requests instead of returning 429 errors. By default, images, JS files, and CSS files will be served at the same speed that the rate limiter dictates (in this case, 200ms per request).


First, open the configuration file for your site:


```
sudo nano /etc/nginx/sites-available/login.your_domain.com


```


Configure Nginx to queue up to 10 requests:


/etc/nginx/sites-available/login.your_domain.com
```
...
limit_req zone=login_limit burst=10;
...

```


Refresh the Nginx configuration:


```
sudo service nginx reload


```


Test the rate limiter changes by running ab again:


```
ab -c 100 -n 100 https://login.your_domain.com/


```


You will see the following output:


```
Output...
Time taken for tests:   2.004 seconds
...
Non-2xx responses:      89
...

```


With a rate limit of 5 requests per second (or an equivalent 300 requests per minute), 10 concurrent web assets requests would take 2 seconds (one every 200ms), which detracts from the users’ experience.


However, using nodelay within limit_req enables Nginx to process the requests in the burst queue without waiting 200ms between each one.


To set this configuration, open your Nginx block configuration file:


```
sudo nano /etc/nginx/sites-available/login.your_domain.com


```


Enable nodelay to serve requests in the burst queue without waiting between requests:


/etc/nginx/sites-available/login.your_domain.com
```
...
limit_req zone=login_limit burst=10 nodelay;
...

```


Reload Nginx with this command:


```
sudo service nginx reload


```


Finally, verify that queued requests in the burst queue are not delayed:


```
ab -c 100 -n 100 https://login.your_domain.com/


```


The output will look similar to the following:


```
Output...
Time taken for tests:   0.233 seconds
...
Non-2xx responses:      89
...

```


The same number of 10 requests won’t be locked waiting for their turn to be served, and the 2 seconds users previously had to wait will be reduced only to the time network and server processing takes to deliver them.


If the same user tried to retrieve the 11th asset before Nginx could clear a slot in the burst queue, the request would be rejected.


Using nodelay enhances user experience since simultaneous requests are served as fast as possible. On the other hand, not using nodelay provides a more controlled flow of requests to the application since a wait interval is enforced between requests.


Beginning with Nginx 1.15.7, delay lets you combine these two approaches. It lets you define how many requests can bypass delays out of the overall burst queue.


To use this approach, begin by opening your Nginx block configuration file:


```
sudo nano /etc/nginx/sites-available/login.your_domain.com


```


Replace nodelay with delay to allow up to 4 requests to be processed without delays:


/etc/nginx/sites-available/login.your_domain.com
```
...
limit_req zone=login_limit burst=10 delay=4;
...

```


Renew the current Nginx configuration:


```
sudo service nginx reload


```


Finally, check that delay is working as expected:


```
ab -c 100 -n 100 https://login.your_domain.com/


```


The output will look similar to this:


```
Output...
Time taken for tests:   1.205 seconds
...
Non-2xx responses:      89
...

```


A maximum of 4 requests will be able to avoid the rate limit delay between requests, just like if they were using nodelay.


The remaining 6 requests in the burst queue will be accepted with the waiting time that the rate limiter specifies (in the example, 200ms between those 6 requests).


You can now quickly deliver first requests while throttling excessive usage and denying serving too much traffic with Nginx. To improve users’ experience even more, you can provide better 429 error pages that are more aligned to your website’s design.


# Step 5 — Customizing a 429 Error Page (Optional)


Every time a visitor exceeds the number of allowed requests to your application, they will meet a 429 error response that includes an HTML page provided by Nginx. To improve user experience on your application, you might want 429 error pages to match the format of regular pages. In this step, you will create a personalized error page that will be served by Nginx.


Make a folder for the 429 error:


```
sudo mkdir -p /usr/share/nginx/errors/429


```


Create a new 429.html file:


```
sudo nano /usr/share/nginx/errors/429/429.html


```


Define the content that you want to return when 429 errors occur:


/usr/share/nginx/errors/429/429.html
```
<html>

<head>
    <title>Error 429</title>
    <link href="https://fonts.googleapis.com/css2?family=Bangers&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="/errors/429/429.css">
    <script></script>
</head>

<body>
    <div id="login-container">
        <h1>Error 429 - Too many requests</h1>
        <p>You have been rate limited.</p>
    </div>
</body>

</html>

```


Add some CSS styling to the 429 error page:


```
sudo nano /usr/share/nginx/errors/429/429.css


```


You can add some styling to define a gradient as background, change the default font, center the text, and allow the body to take 100% of browser height:


/usr/share/nginx/errors/429/429.css
```
body {
    background: linear-gradient(132deg, rgba(184,231,209,1) 35%, rgba(0,255,190,1) 100%);
    font-family: Arial, Helvetica, sans-serif;
    height: 100%;
    overflow: hidden;
    text-align: center;
}

```


You can also increase the size and change header’s font:


/usr/share/nginx/errors/429/429.css
```
...
h1 {
    font-size: 2em;
    font-family: 'Bangers', cursive;
}
...

```


Now that you have the files ready to be served on 429 errors, configure your Nginx block configuration:


```
sudo nano /etc/nginx/sites-available/login.your_domain.com


```


Add the Nginx directive error_page to describe the HTML file path you previously created and make /usr/share/nginx/errors available to the public under /errors. Your final Nginx configuration block should look similar to this:


/etc/nginx/sites-available/login.your_domain.com
```
...
server {
        ...
        error_page 429 /errors/429/429.html;
        ...
}
...

```


Reload Nginx to reflect the new changes:


```
sudo service nginx reload


```


To finish, visit login.your_domain.com and refresh it until you trigger the rate limiter and see the new 429 error page. To test it, consider temporarily removing  burst, delay, and lowering the number of allowed requests to trigger the 429 error more easily.


# Conclusion


Throughout this article, you configured and tweaked Nginx to rate-limit incoming requests to the login to prevent too much traffic from accessing it from the Internet. You can apply what you learned in this tutorial to enhance the security of your web application.


As a next step, consider checking out the Nginx product documentation, Limiting Access to Proxied HTTP Resources, to learn how to limit bandwidth or number of connections per IP. You can also create an allow list based on geolocalization by visiting Advanced Configuration Examples. For the latest Nginx developments, consider reading Nginx’s blog.


For more about security, check out DigitalOcean’s security tutorials, Megan Snyder’s talk on Securing Your Deploy, and Mason Egger’s presentation on Foundations of Computer Security.


