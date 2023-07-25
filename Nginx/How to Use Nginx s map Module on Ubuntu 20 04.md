# How to Use Nginx s map Module on Ubuntu 20 04

```Ubuntu``` ```System Tools``` ```Nginx```

The author selected the Open Internet/Free Speech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Nginx’s map module lets you create variables in Nginx’s configuration file whose values are conditional — that is, they depend on other variables’ values. In this guide, we will look at how to use Nginx’s map module to implement two examples: setting up a list of redirects from old website URLs to new ones and creating an allowlist of countries to control traffic to your website.


# Prerequisites


To follow this tutorial, you will need:


- 
One Ubuntu 20.04 server with a regular, non-root user with sudo privileges. You can learn how to prepare your server by following this initial server setup tutorial.

- 
Nginx installed on your server by following the How To Install Nginx on Ubuntu 20.04 tutorial.


# Step 1 — Creating and Testing an Example Webpage


First, we will create a test file representing a newly published website. We’ll use this file to test our configuration.


Let’s create a simple page, index.html, in the default Nginx website directory. This file will just have plain text describing what’s inside: Home:


```
sudo sh -c 'echo "Home" > /var/www/html/index.html'


```


With this test file in place, we’ll check that it’s being served correctly with curl. We don’t need to specify index.html for this command because that file is served by default if no exact filename is provided:


```
curl http://localhost/


```


In response, you should see a single word saying Home just like below:


```
OutputHome

```


Now let’s try to access a file that doesn’t exist in /var/www/html/, like old.html:


```
curl -L http://localhost/old.html


```


The response will be a system error message, 404 Not Found, meaning the page does not exist:


```
Output<html>
<head><title>404 Not Found</title></head>
<body bgcolor="white">
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>

```


We’re just using a dummy website in this tutorial, but if old.html was a page on a real website that used to exist and was deleted, returning a 404 would mean that all links to that page are broken. This is less than ideal because those links may have been indexed by Google, printed out or written down, or shared by any other means.


In the next step, we’ll use the map module to make sure this old address will work again by redirecting viewers to the new replacements automatically.


# Step 2 — Configuring the Redirects


For small websites with only a few pages, simple if conditional statements can be used for redirects and similar things. However, such a configuration is not easy to maintain or extend in the long run as the list of conditions grows longer.


The map module is a more elegant, concise solution. It allows you to compare Nginx variable values against a list of conditions and then associate a new value with the variable depending on the match. In this example, we’ll be comparing the requested URL with the list of old pages that we want to redirect to their new counterparts. For each old address, we’ll associate the new one.


The map module is a core Nginx module, which means it doesn’t need to be installed separately. To create the necessary map and redirect configuration, open the default server block Nginx configuration file in nano or your favorite text editor:


```
sudo nano /etc/nginx/sites-available/default


```


Find the server configuration block, which looks like this:


/etc/nginx/sites-available/default
```
. . .
# Default server configuration
#

server {
    listen 80 default_server;
    listen [::]:80 default_server;

. . .

```


We’ll be adding two new sections: one before the server block, and one inside it.


The section before the server block is a new map block, which defines the mapping between the old URLs and the new ones using the map module. The section inside the server block is the redirect itself:


/etc/nginx/sites-available/default
```
. . .
# Default server configuration
#

# Old website redirect map
#
map $uri $new_uri {
    /old.html /index.html;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # Old website redirect
    if ($new_uri) {
        rewrite ^ $new_uri permanent;
    }
. . .

```


The map $uri $new_uri directive takes the contents of system $uri variable, which contains the URL address of the requested page, and then compares it against the list of conditions in the curly brackets. Each item in the list of conditions has two sections: the value to match against and the new value to assign to the variable if it matches.


The line /old.html /index.html inside the map block means that if $uri’s value is /old.html, $new_uri will be changed to /index.html. If it doesn’t match, it’s not changed. Here, we only define one condition, but you can define as many conditions as you want in a map.


Using a conditional if statement inside the server block, we check if the $new_uri variable’s value is set. If it is, it means the condition in the map was satisfied, and we should redirect to the new website using the rewrite command. The permanent keyword ensured that the redirect will be a 301 Moved Permanently HTTP redirect, which means that the old address is no longer valid and will not come back online.


Save and close the file to exit.


To enable the new configuration, restart Nginx:


```
sudo systemctl restart nginx


```


To test the new configuration, execute the same request as before:


```
curl -L http://localhost/old.html


```


This time there will be no 404 Not Found error in the output. Instead, you’ll see the simple home page we created in Step 1:


```
OutputHome

```


This means the map has been appropriately configured, and you can use it to redirect URLs by adding more entries to the map.


Redirecting URLs is one useful application of the map module. Another, which we’ll explore in the next step, is filtering traffic based on the visitors’ geographical location.


# Step 3 — Restricting Website Access to Certain Countries


Sometimes, a server might receive an excessive quantity of automated, malicious requests. This could be a DDoS attack, an attempt to brute-force passwords to website administrative panels, or an attempt to exploit known vulnerabilities in software to attack the website and use it to send spam or modify the site contents.


Such automated attacks may come from many different distributed servers in many different countries, making it difficult to block. One solution to mitigate the effects of an attack like this is to create an allowlist of countries that can access the website.


It’s not a perfect solution, but in situations where restricting access to the website based on the visitor’s geographical location is a sensible choice and does not limit the website’s audience, this solution has the benefit of being fast and less error-prone.


Filtering at the server level is faster than filtering at the website level and also covers all requests (including static files, like images). This kind of filtering also prevents requests from reaching the website software directly, making vulnerabilities harder to exploit.


To make use of the geographical filtering, we must first install the Nginx GeoIP module as well as the GeoIP database containing the mappings between visitors’ IP addresses and their respective countries. To do so, let’s execute:


```
sudo apt install libnginx-mod-http-geoip geoip-database


```


Now, let’s first create a new configuration file:


```
sudo nano /etc/nginx/conf.d/geoip.conf


```


Paste the following contents into the file. This tells Nginx where to find the GeoIP database to identify countries based on the visitors’ IP addresses:


/etc/nginx/conf.d/geoip.conf
```
# GeoIP database path
#

geoip_country /usr/share/GeoIP/GeoIP.dat;

```


The next step is to create the necessary map and restriction configuration. Open the default server block Nginx configuration:


```
sudo nano /etc/nginx/sites-available/default


```


Find the server configuration block which, after the modifications in steps 1 and 2, looks like this:


/etc/nginx/sites-available/default
```
. . .
# Default server configuration
#

# Old website redirect map
#
map $uri $new_uri {
    /old.html /index.html;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # Old website redirect
    if ($new_uri) {
        rewrite ^ $new_uri permanent;
    }
. . .

```


We’ll be adding two new sections: one before the server block and one inside it.


The section before the server block is a new map block, which defines the default action (access disallowed) as well as the list of country codes allowed to access the website. The section inside the server block denies access to the website if the map result says so:


Modified /etc/nginx/sites-available/default
```
. . .
# Default server configuration
#

# Allowed countries
#
map $geoip_country_code $allowed_country {
    default no;
    country_code_1 yes;
    country_code_2 yes;
}

# Old website redirect map
#
map $uri $new_uri {
    /old.html /index.html;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;

	# Disallow access based on GeoIP
	if ($allowed_country = no) {
        return 444;
    }

    # Old website redirect
    if ($new_uri) {
        rewrite ^ $new_uri permanent;
    }
. . .

```


Save and close the file to exit.


Here, we used country_code_1 and country_code_2 as placeholders. Replace these variables with the two-character country code for the country or countries you want to allow. You can use the ISO’s full, searchable list of all country codes to find. For example, the code for the United States is US.


Unlike the first example, in this map block, the $allowed_country variable will always be set to something. By default, it’s set to no; if the $geoip_country_code variable matches one of the country codes in the block, it’s set to yes. If the $allowed_country variable is no, we return a 444 Connection Closed Without Response instead of serving the actual website.


To enable the new configuration, restart Nginx:


```
sudo systemctl restart nginx


```


If you didn’t add your country to the allowlist, when you try to visit http://your_server_ip, you’ll see an error message like The page isn’t working or The page didn’t send any data. If you did add your country to the allowlist, you’ll see Home as before.


# Conclusion


The map module not only allows simple comparisons but also supports regular expressions allowing more complex matches. It is a great way to make configuration files clearer if multiple conditions must be evaluated.


More detailed information can be found in Nginx’s official map module documentation.


