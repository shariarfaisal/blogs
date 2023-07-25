# How To Build a Rate Limiter With Node js on App Platform

```Security``` ```Node.js``` ```Development``` ```DigitalOcean App Platform```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Rate limiting manages your network’s traffic and limits the number of times someone repeats an operation in a given duration, such as using an API. A service without a layer of security against rate limit abuse is prone to overload and hampers your application’s proper operation for legitimate customers.


In this tutorial, you will build a Node.js server that will check the IP address of the request and also calculate the rate of these requests by comparing the timestamp of requests per user. If an IP address crosses the limit you have set for the application, you will call Cloudflare’s API and add the IP address to a list. You will then configure a Cloudflare Firewall Rule that will ban all requests with IP addresses in the list.


By the end of this tutorial, you will have built a Node.js project deployed on DigitalOcean’s App Platform that protects a Cloudflare routed domain with rate limiting.


# Prerequisites


Before you begin this guide, you will need:


- A Cloudflare account. The free plan of Cloudflare is sufficient for the tutorial. If you are creating a new account, choose the Free plan. This guide on creating a Cloudflare account and adding a website can help you.
- A registered domain added to your Cloudflare account. The guide on how to mitigate DDoS attacks against your website with Cloudflare can help you set this up. This article on introduction to DNS terminology, components, and concepts can also be of assistance.
- Basic Express server with Node.js. Follow How To Get Started with Node.js and Express article up to Step 2.
- A GitHub account and git installed on your local machine. A GitHub account and having git installed is necessary as you will push the code to GitHub to deploy from DigitalOcean App Platform.
- A DigitalOcean account.

# Step 1 — Setting Up the Node.js Project and Deploying to DigitalOcean’s App Platform


In this step, you will expand on your basic Express server, push your code to a GitHub repository, and deploy your application to App Platform.


Open the project directory of the basic Express server with your code editor. Create a new file by the name .gitignore in the root directory of the project. Add the following lines to the newly created .gitignore file:


.gitignore
```
node_modules/
.env

```


The first line in your .gitignore file is a directive to git not to track the node_modules directory. This will enable you to keep your repository size small. The node_modules can be generated when required by running the command npm install. The second line prevents the environment variable file from being tracked. You will create the .env file in further steps.


Navigate to your server.js in your code editor and modify the following lines of code:


server.js
```
...
app.listen(process.env.PORT || 3000, () => {
    console.log(`Example app is listening on port ${process.env.PORT || 3000}`);
});

```


The change to conditionally use PORT as an environment variable enables the application to dynamically have the server running on the assigned PORT or use 3000 as the fallback one.



Note: The string in console.log() is wrapped within backticks(`) and not within quotes. This enables you to use template literals, which provides the capability to have expressions within strings.

Visit your terminal window and run your application:


```
node server.js


```


Your browser window will display Successful response. In your terminal, you will see the following output:


```
OutputExample app is listening on port 3000

```


With your Express server running successfully, you’ll now deploy to App Platform.


First, initialize git in the root directory of the project and push the code to your GitHub account. Navigate to the App Platform dashboard in the browser and click on the Create App button. Choose the GitHub option and authorize with GitHub, if necessary. Select your project’s repository from the dropdown list of projects you want to deploy to App Platform. Review the configuration, then give a name to the application. For the purpose of this tutorial, select the Basic plan as you’ll work in the application’s development phase. Once ready, click Launch App.


Next, navigate to the Settings tab and click on the section Domains. Add your domain routed via Cloudflare into the field Domain or Subdomain Name. Select the bullet You manage your domain to copy the CNAME record that you’ll use to add to your domain’s Cloudflare DNS account.


With your application deployed to App Platform, head over to your domain’s dashboard on Cloudflare in a new tab as you will return to App Platform’s dashboard later. Navigate to the DNS tab. Click on the Add Record button and select CNAME as your Type, @ as the root, and paste in the CNAME you copied from the App Platform. Click on the Save button, then navigate to the Domains section under the Settings tab in your App Platform’s Dashboard and click on the Add Domain button.


Click the Deployments tab to see the details of the deployment. Once deployment finishes, you can open your_domain to view it on the browser. Your browser window will display: Successful response. Navigate to the Runtime Logs tab on the App Platform dashboard, and you will get the following output:


```
OutputExample app is listening on port 8080


```



Note: The port number 8080 is the default assigned port by the App Platform. You can override this by changing the configuration while reviewing the app before deployment.

With your application now deployed to App Platform, let’s look at how to outline a cache to calculate requests to the rate limiter.


# Step 2 — Caching User’s IP Address and Calculating Requests Per Second


In this step, you will store a user’s IP address in a cache with an array of timestamps to monitor the requests per second of each user’s IP address. A cache is temporary storage for data frequently used by an application. The data in a cache is usually kept in quick access hardware like RAM (Random-Access Memory). The fundamental goal of a cache is to improve data retrieval performance by decreasing the need to visit the slower storage layer underneath it. You will use three npm packages: node-cache, is-ip, and request-ip to aid in the process.


The request-ip package captures the user’s IP address used to request the server. The node-cache package creates an in-memory cache which you will use to keep track of user’s requests. You’ll use the is-ip package used to check if an IP Address is IPv6 Address. Install the node-cache, is-ip, and request-ip package via npm on your terminal.


```
npm i node-cache is-ip request-ip


```


Open the server.js file in your code editor and add following lines of code below const express = require('express');:


server.js
```
...
const requestIP = require('request-ip');
const nodeCache = require('node-cache');
const isIp = require('is-ip');
...

```


The first line here grabs the requestIP module from request-ip package you installed. This module captures the user’s IP address used to request the server. The second line grabs the nodeCache module from the node-cache package. nodeCache creates an in-memory cache, which you will use to keep track of user’s requests per second. The third line takes the isIp module from the is-ip package. This checks if an IP address is IPv6 which you will format as per Cloudflare’s specification to use CIDR notation.


Define a set of constant variables in your server.js file. You will use these constants throughout your application.


server.js
```
...
const TIME_FRAME_IN_S = 10;
const TIME_FRAME_IN_MS = TIME_FRAME_IN_S * 1000;
const MS_TO_S = 1 / 1000;
const RPS_LIMIT = 2;
...

```


TIME_FRAME_IN_S is a constant variable that will determine the period over which your application will average the user’s timestamps. Increasing the period will increase the cache size, hence consume more memory. The TIME_FRAME_IN_MS constant variable will also determine the period of time your application will average user’s timestamps, but in milliseconds. MS_TO_S is the conversion factor you will use to convert time in milliseconds to seconds. The RPS_LIMIT variable is the threshold limit of the application that will trigger the rate limiter, and change the value as per your application’s requirements. The value 2 in the RPS_LIMIT variable is a moderate value that will trigger during the development phase.


With Express, you can write and use middleware functions, which have access to all HTTP requests coming to your server. To define a middleware function, you will call app.use() and pass it a function. Create a function named ipMiddleware as middleware.


server.js
```
...
const ipMiddleware = async function (req, res, next) {
    let clientIP = requestIP.getClientIp(req);
    if (isIp.v6(clientIP)) {
        clientIP = clientIP.split(':').splice(0, 4).join(':') + '::/64';
    }
    next();
};
app.use(ipMiddleware);

...

```


The getClientIp() function provided by requestIP takes the request object, req from the middleware, as parameter. The .v6() function comes from the is-ip module and returns true if the argument passed to it is an IPv6 address. Cloudflare’s Lists requires the IPv6 address in /64 CIDR notation. You need to format the IPv6 address to follow the format: aaaa:bbbb:cccc:dddd::/64. The .split(':') method creates an array from the string containing the IP address splitting them by the character :. The .splice(0,4) method returns the first four elements of the array. The .join(':') method returns a string from the array combined with the character :.


The next() call directs the middleware to go to the next middleware function if there is one. In your example, it will take the request to the GET route /. This is important to include at the end of your function. Otherwise, the request will not move forward from the middleware.


Initialize an instance of node-cache by adding the following variable below the constants:


server.js
```
...
const IPCache = new nodeCache({ stdTTL: TIME_FRAME_IN_S, deleteOnExpire: false, checkperiod: TIME_FRAME_IN_S });
...

```


With the constant variable IPCache, you are overriding the default parameters native to nodeCache with the custom properties:


- stdTTL: The interval in seconds after which a key-value pair of cache elements will be evicted from the cache. TTL stands for Time To Live, and is a measure of time after which cache expires.
- deleteOnExpire: Set to false as you will write a custom callback function to handle the expired event.
- checkperiod: The interval in seconds after which an automatic check for expired elements is triggered. The default value is 600, and as your application’s element expiry is set to a lesser value, the check for expiry will also happen sooner.

For more information on the default parameters of node-cache, you will find the node-cache npm package’s docs page useful. The following diagram will help you to visualise how a cache stores data:





You will now create a new key-value pair for the new IP address and append to an existing key-value pair if an IP address exists in the cache. The value is an array of timestamps corresponding to each request made to your application. In your server.js file, create the updateCache() function below the IPCache constant variable to add the timestamp of the request to cache:


server.js
```
...
const updateCache = (ip) => {
    let IPArray = IPCache.get(ip) || [];
    IPArray.push(new Date());
    IPCache.set(ip, IPArray, (IPCache.getTtl(ip) - Date.now()) * MS_TO_S || TIME_FRAME_IN_S);
};
...

```


The first line in the function gets the array of timestamps for the given IP address, or if null, initializes with an empty array. In the following line, you are pushing the present timestamp caught by the new Date() function into the array. The .set() function provided by node-cache takes three arguments: key, value and the TTL. This TTL will override the standard TTL set by replacing the value of stdTTL from the IPCache variable. If the IP address already exists in the cache, you will use the existing TTL; else, you will set TTL as TIME_FRAME_IN_S.


The TTL for the current key-value pair is calculated by subtracting the present timestamp from the expiry timestamp. The difference is then converted to seconds and passed as the third argument to the .set() function. The .getTtl() function takes a key and IP address as an argument and returns the TTL of the key-value pair as a timestamp. If the IP address does not exist in the cache, it will return undefined and use the fallback value of TIME_FRAME_IN_S.



Note: You require the conversion timestamps from milliseconds to seconds as JavaScript stores them in milliseconds while the node-cache module uses seconds.

In the ipMiddleware middleware, add the following lines after the if code block if (isIp.v6(clientIP)) to calculate the requests per second of the IP address calling your application:


server.js
```
...
    updateCache(clientIP);
    const IPArray = IPCache.get(clientIP);
    if (IPArray.length > 1) {
        const rps = IPArray.length / ((IPArray[IPArray.length - 1] - IPArray[0]) * MS_TO_S);
        if (rps > RPS_LIMIT) {
            console.log('You are hitting limit', clientIP);
        }
    }
...

```


The first line adds the timestamp of the request made by the IP address to the cache by calling the updateCache() function you declared. The second line collects the array of timestamps for the IP address. If the number of elements in the array of timestamps is greater than one (calculating requests per second needs a minimum of two timestamps), and the requests per second are more than the threshold value you defined in the constants, you will console.log the IP address. The rps variable calculates the requests per second by dividing the number of requests with a time interval difference, and converts the units to seconds.


Since you had defaulted the property deleteOnExpire to the value false in the IPCache variable, you will now need to handle the expired event manually. node-cache provides a callback function that triggers on expired event. Add the following lines of code below the IPCache constant variable:


server.js
```
...
IPCache.on('expired', (key, value) => {
    if (new Date() - value[value.length - 1] > TIME_FRAME_IN_MS) {
        IPCache.del(key);
    }
});
...

```


.on() is a callback function that accepts key and value of the expired element as the arguments. In your cache, value is an array of timestamps of requests. The highlighted line checks if the last element in the array is at least TIME_FRAME_IN_S in the past than the present time. As you are adding elements to your array of timestamps, if the last element in value is at least TIME_FRAME_IN_S in the past than the present time, the .del() function takes key as an argument and deletes the expired element from the cache.


For the instances when some elements of the array are at least TIME_FRAME_IN_S in the past than the present time, you need to handle it by removing the expired items from the cache. Add the following code in the callback function after the if code block if (new Date() - value[value.length - 1] > TIME_FRAME_IN_MS).


server.js
```
...
    else {
        const updatedValue = value.filter(function (element) {
            return new Date() - element < TIME_FRAME_IN_MS;
        });
        IPCache.set(key, updatedValue, TIME_FRAME_IN_S - (new Date() - updatedValue[0]) * MS_TO_S);
    }
...

```


The filter()  array method native to JavaScript provides a callback function to filter the elements in your array of timestamps. In your case, the highlighted line checks for elements that are least TIME_FRAME_IN_S in the past than the present time. The filtered elements are then added to the updatedValue variable. This will update your cache with the filtered elements in the updatedValue variable and a new TTL. The TTL that matches the first element in the updatedValue variable will trigger the .on('expired') callback function when the cache removes the following element. The difference of TIME_FRAME_IN_S and the time expired since the first request’s timestamp in updatedValue calculates the new and updated TTL.


With your middleware functions now defined, visit your terminal window and run your application:


```
node server.js


```


Then, visit localhost:3000 in your web browser. Your browser window will display: Successful response. Refresh the page repeatedly to hit the RPS_LIMIT. Your terminal window will display:


```
OutputExample app is listening on port 3000
You are hitting limit ::1

```



Note: The IP address for localhost is shown as ::1. Your application will capture the public IP of a user when deployed outside localhost.

Your application is now able to able to track the user’s requests and store the timestamps in the cache. In the next step, you will integrate Cloudflare’s API to set up the Firewall.


# Step 3 — Setting Up the Cloudflare Firewall


In this step, you will set up Cloudflare’s Firewall to block IP Addresses when hitting the rate limit, create environment variables, and make calls to the Cloudflare API.


Visit the Cloudflare dashboard in your browser, log in, and navigate to your account’s homepage. Open Lists under Configurations tab. Create a new List with your_list as the name.



Note: The Lists section is available on your Cloudflare account’s dashboard page and not your Cloudflare domain’s dashboard page.

Navigate to the Home tab and open your_domain's dashboard. Open the Firewall tab and click on Create a Firewall rule under the Firewall Rules section. Give your_rule_name to the Firewall to identify it. In the Field, select IP Source Address from the dropdown, is in list for the Operator, and your_list for the Value. Under the dropdown for Choose an action, select Block and click Deploy.


Create a .env file in the project’s root directory with the following lines to call Cloudflare API from your application:


.env
```
ACCOUNT_MAIL=your_cloudflare_login_mail
API_KEY=your_api_key
ACCOUNT_ID=your_account_id
LIST_ID=your_list_id

```


To get a value for API_KEY, navigate to the API Tokens tab on the My Profile section of your Cloudflare dashboard. Click View in the Global API Key section and enter your Cloudflare password to view it. Visit the Lists section under the Configurations tab on the account’s homepage. Click on Edit beside your_list list you created. Get the ACCOUNT_ID and LIST_ID from the URL of your_list in the browser. The URL is of the format below:
https://dash.cloudflare.com/your_account_id/configurations/lists/your_list_id



Warning: Make sure the content of .env is kept confidential and not made public. Make sure you have the .env file listed in the .gitignore file you created in Step 1.
<$>
Install the axios and dotenv package via npm on your terminal.
npm i axios dotenv


Open the server.js file in your code editor and the add following lines of code below the nodeCache constant variable:
server.js
...
const axios = require('axios');
require('dotenv').config();
...

The first line here grabs the axios module from axios package you installed. You will use this module to make network calls to Cloudflare’s API. The second line requires and configures the dotenv module to enable the process.env global variable that will define the values you placed in your .env file to server.js.
Add the following to the if (rps > RPS_LIMIT) condition within ipMiddleware above console.log('You are hitting limit', clientIP) to call Cloudflare API.
server.js
...
    const url = `https://api.cloudflare.com/client/v4/accounts/${process.env.ACCOUNT_ID}/rules/lists/${process.env.LIST_ID}/items`;
    const body = [{ ip: clientIP, comment: 'your_comment' }];
    const headers = {
        'X-Auth-Email': process.env.ACCOUNT_MAIL,
        'X-Auth-Key': process.env.API_KEY,
        'Content-Type': 'application/json',
    };
    try {
        await axios.post(url, body, { headers });
    } catch (error) {
        console.log(error);
    }
...

You are now calling the Cloudflare API through the URL to add an item, in this case an IP address, to your_list. The Cloudflare API takes your ACCOUNT_MAIL and API_KEY in the header of the request with the key as X-Auth-Email and X-Auth-Key. The body of the request takes an array of objects with ip as the IP address to add to the list, and a comment with the value your_comment to identify the entry. You can modify value of comment with your own custom comment. The POST request made via axios.post() is wrapped in a try-catch block to handle errors if any, that may occur. The axios.post function takes the url, body and an object with headers to make the request.
Change the clientIP variable within the ipMiddleware function when testing out the API requests with a test IP address like 198.51.100.0/24 as Cloudflare does not accept the localhost’s IP address in its Lists.
server.js
...
let clientIP = '198.51.100.0/24';
...

Visit your terminal window and run your application:
node server.js


Then, visit localhost:3000 in your web browser. Your browser window will display: Successful response. Refresh the page repeatedly to hit the RPS_LIMIT. Your terminal window will display:
OutputExample app is listening on port 3000
You are hitting limit ::1

When you have hit the limit, open the Cloudflare dashboard and navigate to the your_list's page. You will see the IP address you put in the code added to your Cloudflare’s List named your_list. The Firewall page will display after pushing your changes to GitHub.
<$>[warning]
Warning:  Make sure to change the value in your clientIP variable to requestIP.getClientIp(req) before deploying or pushing the code to GitHub.

Deploy your application by committing the changes and pushing the code to GitHub. As you have set up auto-deploy, the code from GitHub will automatically deploy to your DigitalOcean’s App Platform. As your .env file is not added to GitHub, you will need to add it to App Platform via the Settings tab at App-Level Environment Variables section. Add the key-value pair from your project’s .env file so your application can access its contents on the App Platform. After you save the environment variables, open your_domain in your browser after deployment finishes and refresh the page repeatedly to hit the RPS_LIMIT. Once you hit the limit, the browser will show Cloudflare’s Firewall page.





Navigate to the Runtime Logs tab on the App Platform dashboard, and you will view the following output:


```
Output...
You are hitting limit your_public_ip

```


You can open your_domain from a different device or via VPN to see that the Firewall bans only the IP address in your_list. You can delete the IP address from your_list through your Cloudflare dashboard.



Note: Occasionally, it takes few seconds for the Firewall to trigger due to the cached response from the browser.

You have set up Cloudflare’s Firewall to block IP Addresses when users are hitting the rate limit by making calls to the Cloudflare API.


# Conclusion


In this article, you built a Node.js project deployed on DigitalOcean’s App Platform connected to your domain routed via Cloudflare. You protected your domain against rate limit misuse by configuring a Firewall Rule on Cloudflare. From here, you can modify the Firewall Rule to show JS Challenge or CAPTCHA instead of banning the user. The Cloudflare documentation details the process.


