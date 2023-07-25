# How To Install and Configure Naxsi on Ubuntu 14 04

```Security``` ```Ubuntu``` ```Nginx``` ```Firewall```

## Introduction


Naxsi is a third party Nginx module which provides web application firewall features. It brings additional security to your web server and protects you from various web attacks such as XSS and SQL injections.


Naxsi is flexible and powerful. You can use readily available rules for popular web applications such as WordPress. At the same time, you can also create your own rules and fine tune them by using Naxsi’s learning mode.


Naxsi is similar to ModSecurity for Apache. Thus, if you are already acquainted with ModSecurity and/or seek similar functionality for Nginx, Naxsi will certainly be of interest to you. However, you may not find all of ModSecurity’s features in Naxsi.


This tutorial shows you how to install Naxsi, understand the rules, create a whitelist, and where to find rules already written for commonly-used web applications.


# Prerequisites


Before following this tutorial, please make sure you complete the following prerequisites:


- An Ubuntu 14.04 Droplet
- A non-root sudo user. Check out Initial Server Setup with Ubuntu 14.04 for details.

Except otherwise noted, all of the commands that require root privileges in this tutorial should be run as a non-root user with sudo privileges.


# Step 1 — Installing Naxsi


To install Naxsi you will have to install an Nginx server compiled with it. For this purpose you will need the package nginx-naxsi. You can install it in the usual Ubuntu way with the apt-get command:


```
sudo apt-get update
sudo apt-get install nginx-naxsi


```


This will install Naxsi along with Nginx and all of its dependencies. It will also make sure the service starts and stops automatically on the Droplet.



Note: If you already have Nginx installed without Naxsi, you will need to replace the package nginx-core, or another flavor of Nginx you might have, with the package nginx-naxsi. The other Nginx packages do not support loadable modules, and you cannot just load Naxsi into an existing Nginx server.
In most cases replacing nginx-core with nginx-naxsi is not a problem, and you can continue using your previous configuration. Still, it’s always a good idea with such upgrade to create a backup of your existing /etc/nginx/ directory first. After that, follow the instructions for a new installation, and simply confirm you agree to remove the existing Nginx package on your system.

The default installation of Nginx provides a basic, working Nginx environment, which is sufficient for getting acquainted with Naxsi. We will not spend time customizing Nginx, but instead we will go straight to configuring Naxsi. However, if you have no experience with Nginx it is a good idea to check How To Install Nginx on Ubuntu 14.04 LTS and its related articles, especially How To Set Up Nginx Server Blocks (Virtual Hosts) on Ubuntu 14.04 LTS.


# Step 2 — Enabling Naxsi


First, to enable Naxsi we have to load its core rules found in the file /etc/nginx/naxsi_core.rules. This file contains generic signatures for detecting malicious attacks. We’ll discuss these rules in greater details later. For now, we’ll just include the rules in Nginx’s main configuration file /etc/nginx/nginx.conf in the HTTP listener part. So, open the latter file for editing with nano:


```
sudo nano /etc/nginx/nginx.conf


```


Then find the http section and uncomment the include part for Naxsi’s rules by removing the # character at the beginning of the line. It should now look like this:


/etc/nginx/nginx.conf
```
http {
...
        # nginx-naxsi config
        ##
        # Uncomment it if you installed nginx-naxsi
        ##

        include /etc/nginx/naxsi_core.rules;
...

```


Save the file and exit the editor.


Second, we have to enable the previous rules and configure some basic options for Naxsi. By default, the basic Naxsi configuration is found in the file /etc/nginx/naxsi.rules. Open this file:


```
sudo nano /etc/nginx/naxsi.rules


```


Change only the value for DeniedUrl to an error file that already exists by default, and leave the rest unchanged:


/etc/nginx/naxsi.rules
```
# Sample rules file for default vhost.
LearningMode;
SecRulesEnabled;
#SecRulesDisabled;
DeniedUrl "/50x.html";

## check rules
CheckRule "$SQL >= 8" BLOCK;
CheckRule "$RFI >= 8" BLOCK;
CheckRule "$TRAVERSAL >= 4" BLOCK;
CheckRule "$EVADE >= 4" BLOCK;
CheckRule "$XSS >= 8" BLOCK;

```


Save the file and exit.


Here are the configuration directives from above with their meaning:


- LearningMode - Start Naxsi in learning mode. This means that no request will actually be blocked. Only security exceptions will be raised in the Nginx error log. Such a non-blocking initial behavior is important because the default rules are rather aggressive. Later, based on these exceptions, we will create whitelist for legitimate traffic.
- SecRulesEnabled - Enable Naxsi for a server block / location. Similarly, you can disable Naxsi for a site or part of a site by uncommenting SecRulesDisabled.
- DeniedUrl - URL to which denied requests will be sent internally. This is the only setting you should change. You can use the readily available 50x.html error page found inside the default document root (/usr/share/nginx/html/50x.html), or you can create your own custom error page.
- CheckRule - Set the threshold for the different counters. Once this threshold is passed (e.g. 8 points for the SQL counter) the request will be blocked. To make these rules more aggressive, decrease their values and vice versa.

The file naxsi.rules has to be loaded on a per location basis for a server block. Let’s load it for the root location (/) of the default server block. First open the server block’s configuration file /etc/nginx/sites-enabled/default:


```
sudo nano /etc/nginx/sites-enabled/default


```


Then, find the root location / and make sure that it looks like this:


```
    location / {
            # First attempt to serve request as file, then
            # as directory, then fall back to displaying a 404.
            try_files $uri $uri/ =404;
            # Uncomment to enable naxsi on this location
            include /etc/nginx/naxsi.rules;
    }

```



Warning: Make sure to add a semicolon at the end of the include statement for naxsi.rules because there is no such by default. Thus, if you only uncomment the statement, there will be a syntax error in the configuration.

Once you have made the above changes you can reload Nginx for the changes to take effect:


```
sudo service nginx reload


```


The next step explains how to check if the changes have been successful and how to read the logs.


# Step 3 — Checking the Logs


To make sure Naxsi works, even though still in learning mode, let’s access a URL that should throw an exception and watch the error log for the exception.


We’ll see later how this rule works exactly. For now, tail Nginx’s error log to find the exception (the -f option keeps the output open and appends new content to it:


```
sudo tail -f /var/log/nginx/error.log


```


Try to access your Droplet at the URL http://Your_Droplet_IP/index.html?asd=----. This should trigger a Naxsi security exception because of the dashes, which are used for comments in SQL, and thus are considered parts of SQL injections.


In the output of sudo tail -f /var/log/nginx/error.log, you should now see the following new content:


```
Output of nginx's error log2015/11/14 03:58:35 [error] 4088#0: *1 NAXSI_FMT: ip=X.X.X.X&server=Y.Y.Y.Y&uri=/index.html&learning=1&total_processed=24&total_blocked=1&zone0=ARGS&id0=1007&var_name0=asd, client: X.X.X.X, server: localhost, request: "GET /index.html?asd=---- HTTP/1.1", host: "Y.Y.Y.Y"

```


The most important part of the above line is highlighted: zone0=ARGS&id0=1007&var_name0=asd. It gives you the zone (the part of the request), the id of the triggered rule, and the variable name of the suspicious request.


Furthermore, X.X.X.X is your local computer’s IP, and Y.Y.Y.Y is the IP of your Droplet. The URI also contains the filename of the request (index.htm), the fact that Naxsi is still working in learning mode (learning=1), and the total number of all processed requests (total_processed=24).


Also, right after the above line, there should follow a message about the redirect to the DeniedUrl:


```
Output of nginx's error log2015/11/14 03:58:35 [error] 4088#0: *1 rewrite or internal redirection cycle while internally redirecting to "/50x.html" while sending response to client, client: X.X.X.X, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "Y.Y.Y.Y", referrer: "http://Y.Y.Y.Y/index.html?asd=----"

```


When Naxsi is in learning mode, this redirect will only show in the logs but will not actually happen.


Press CTRL-C to exit tail and stop the output of the error log file.


Later on, we’ll learn more about Naxsi’s rules, and then it’ll be important to have this basic understanding of the logs.


# Step 4 — Configuring Naxsi Rules


The most important part of Naxsi’s configuration is its rules. There are two types of rules — main rules and basic rules. The main rules (identified by MainRule) are applied globally for the server, and thus are part of the http block of the main Nginx’s configuration. They contain generic signatures for detecting malicious activities.


The basic rules (identified by BasicRule) are used mainly for whitelisting false positive signatures and rules. They are applied per location and thus should be part of the server block (vhost) configuration.


Let’s start with the main rules, and take a look at the default ones provided by the nginx-naxsi package in the file /etc/nginx/naxsi_core.rules. Here is a sample line:


/etc/nginx/naxsi_core.rules
```
...
MainRule "str:--" "msg:mysql comment (--)" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:4" id:1007;
...

```


From the rule above we can outline the following parts, which are universal and present in every rule:


- MainRule is the directive to begin every rule with. Similarly, every rule ends with the rule’s id number.
- str: is found in the second part of the rule. If it is str: it means that the signature will be a plain string, as per the example above. Regular expressions can be also matched with the directive rx:.
- msg: gives some clarification about the rule.
- mz: stands for match zone, or which part of the request will be inspected. This could be the body, the URL, the arguments, etc.
- s: determines the score which will be assigned when the signature is found. Scores are added to different counters such as SQL (SQL attacks), RFI (remote file inclusion attacks), etc.

Essentially, the above rule (id 1007) with comment mysql comments means that if the string -- is found in any part of a request (body, arguments, etc.), 4 points will be added to the SQL counter.


If we go back to the example URI (http://Your_Droplet_IP/index.html?asd=----) that triggered the SQL exception in the log, you will notice that to trigger rule 1007, we needed 2 pairs of dashes (--). This is because for each pair we get 4 points and the SQL chain needs 8 points to block a request. Thus, only one pair of dashes would not be problematic, and in most cases legitimate traffic will not suffer.


One special rule directive is negative. It applies scores if the signature is not matched, i.e. you suspect malicious activity when something in the request is missing.


For example, let’s look at the rule with id 1402 from the same file /etc/nginx/naxsi_core.rules:


/etc/nginx/naxsi_core.rules
```
...
MainRule negative "rx:multipart/form-data|application/x-www-form-urlencoded" "msg:Content is neither mulipart/x-www-form.." "mz:$HEADERS_VAR:Content-type" "s:$EVADE:4" id:1402;
...

```


The above rule means that 4 points will be added to the EVADE counter if the Content-type  request header has neither multipart/form-data, nor application/x-www-form-urlencoded in it. This rule is also an example of how regular expressions (rx:) can be used for the signature description.


# Step 5 — Whitelisting Rules


The default Naxsi rules will almost certainly block some legitimate traffic on your site, especially if you have a complex web application supporting a wide variety of user interactions. That’s why there are whitelists to resolve such problems.


Whitelists are created with the second type of rules, Naxsi’s basic rules. With a basic rule you can whitelist either a whole rule or parts of it.


To demonstrate how basic rules work, let’s go back to the SQL comment rule (id 1007). Imagine that you have a file with two dashes in the filename, e.g. some--file.html on your site. With rule 1007 in place, this file will increase the SQL counter with 4 points. This filename alone and the resulting score is not sufficient to block a request, but it is still a false positive which could cause problems. For example, if we also have an argument with two dashes in it, then the request will trigger rule 1007.


To test it, tail the error log like before:


```
sudo tail -f /var/log/nginx/error.log


```


Try accessing http://Your_Droplet_IP/some--file.html?asd=--. You don’t need to have this file on your website for the test.


You should see a familiar exception similar to this one in the output of the error log:


```
Output of nginx's error log2015/11/14 14:43:36 [error] 5182#0: *10 NAXSI_FMT: ip=X.X.X.X&server=Y.Y.Y.Y&uri=/some--file.html&learning=1&total_processed=10&total_blocked=6&zone0=URL&id0=1007&var_name0=&zone1=ARGS&id1=1007&var_name1=asd, client: X.X.X.X, server: localhost, request: "GET /some--file.html?asd=-- HTTP/1.1", host: "Y.Y.Y.Y"

```


Press CTRL-C to stop showing the error log output.


To address this false positive trigger we’ll need a whitelist which looks like this one:


```
BasicRule wl:1007 "mz:URL";

```


The important keyword is wl for whitelist, followed by the rule ID. To be more precise what we are whitelisting, we have also specified the match zone — the URL.


To apply this whitelist, first create a new file for whitelists:


```
sudo nano /etc/nginx/naxsi_whitelist.rules


```


Then, paste the rule into the file:


/etc/nginx/naxsi_whitelist.rules
```
BasicRule wl:1007 "mz:URL";

```


If you have other whitelists, they can go in this file too, each one on a new row.


The file with whitelists has to be included in your server block. To include it in the default server block, use again nano:


```
sudo nano /etc/nginx/sites-enabled/default


```


Then add the new include right after the previous one for Naxsi like this:


/etc/nginx/sites-enabled/default
```

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
                # Uncomment to enable naxsi on this location
                include /etc/nginx/naxsi.rules;
                include /etc/nginx/naxsi_whitelist.rules;
        }

```


For this change to take effect, reload Nginx:


```
sudo service nginx reload


```


Now if you try again the same request in your browser to Your_Droplet_IP/some--file.html?asd=-- only the asd parameter equaling two dashes will trigger 4 points for the SQL counter, but the uncommon filename will not. Thus, you will not see this request in the error log as an exception.


Writing all the necessary whitelists can be a tedious task and a science of its own. That’s why at the beginning you can use readily available Naxsi whitelists. There are such for most popular web applications. You just have to download them and include them in the server block like we just did.


Once you have made sure you don’t see any exceptions for legitimate requests in the error log, you can disable the learning mode of Naxsi. For this purpose open the file /etc/nginx/naxsi.rules with nano:


```
sudo nano /etc/nginx/naxsi.rules


```


Comment out the LearningMode directive by adding the # character in front of it like this:


/etc/nginx/naxsi.rules
```
...
#LearningMode;
SecRulesEnabled;
#SecRulesDisabled;
...

```


Finally, reload Nginx for the change to take effect:


```
sudo service nginx reload


```


Now, Naxsi will block any suspicious requests, and your site will be more secure.


# Conclusion


That’s how easy it is to have a web application firewall with Nginx and Naxsi. That’s enough for a beginning and hopefully you will be interested in learning more of what the powerful Naxsi module has to offer. Now you can make your Nginx server not only fast, but also secure.


