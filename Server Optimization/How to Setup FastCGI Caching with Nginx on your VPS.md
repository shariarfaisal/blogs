# How to Setup FastCGI Caching with Nginx on your VPS

```Caching``` ```Server Optimization``` ```Nginx``` ```PHP```

## Prelude



Nginx includes a FastCGI module which has directives for caching dynamic content that are served from the PHP backend. Setting this up removes the need for additional page caching solutions like reverse proxies (think Varnish) or application specific plugins. Content can also be excluded from caching based on the request method, URL, cookies, or any other server variable.


# Enabling FastCGI Caching on your VPS



This article assumes that you’ve already setup and configured Nginx with PHP on your droplet. Edit the Virtual Host configuration file for which caching has to be enabled.


```
nano /etc/nginx/sites-enabled/vhost

```


Add the following lines to the top of the file outside the server { } directive:


```
fastcgi_cache_path /etc/nginx/cache levels=1:2 keys_zone=MYAPP:100m inactive=60m;
fastcgi_cache_key "$scheme$request_method$host$request_uri";

```


The “fastcgi_cache_path” directive specifies the location of the cache (/etc/nginx/cache), its size (100m), memory zone name (MYAPP), the subdirectory levels, and the inactive` timer.


The location can be anywhere on the hard disk; however, the size must be less than your droplet’s RAM + Swap or else you’ll receive an error that reads “Cannot allocate memory.” We will look at the “levels” option in the purging section-- if a cache isn’t accessed for a particular amount of time specified by the “inactive” option (60 minutes here), then Nginx removes it.


The “fastcgi_cache_key” directive specifies how the the cache filenames will be hashed. Nginx encrypts an accessed file with MD5 based on this directive.


Next, move the location directive that passes PHP requests to php5-fpm. Inside “location ~ .php$ { }” add the following lines.


```
fastcgi_cache MYAPP;
fastcgi_cache_valid 200 60m;

```


The “fastcgi_cache” directive references to the memory zone name which we specified in the “fastcgi_cache_path” directive and stores the cache in this area.


By default Nginx stores the cached objects for a duration specified by any of these headers: X-Accel-Expires/Expires/Cache-Control.


The “fastcgi_cache_valid” directive is used to specify the default cache lifetime if these headers are missing. In the statement we entered above, only responses with a status code of 200 is cached. Other response codes can also be specified.


Do a configuration test


```
service nginx configtest

```


Reload Nginx if everything is OK


```
service nginx reload

```


The complete vhost file will look like this:


```
fastcgi_cache_path /etc/nginx/cache levels=1:2 keys_zone=MYAPP:100m inactive=60m;
fastcgi_cache_key "$scheme$request_method$host$request_uri";

server {
    listen   80;
    
	root /usr/share/nginx/html;
	index index.php index.html index.htm;

	server_name example.com;

	location / {
	    try_files $uri $uri/ /index.html;
    }

	location ~ \.php$ {
	    try_files $uri =404;
	    fastcgi_pass unix:/var/run/php5-fpm.sock;
	    fastcgi_index index.php;
	    include fastcgi_params;
	    fastcgi_cache MYAPP;
	    fastcgi_cache_valid 200 60m;
    }
}

```


Next we will do a test to see if caching works.


# Testing FastCGI Caching on your VPS



Create a PHP file which outputs a UNIX timestamp.


```
 /usr/share/nginx/html/time.php

```


Insert


```
<?php
echo time();
?>

```


Request this file multiple times using curl or your web browser.


```
root@droplet:~# curl http://localhost/time.php;echo
1382986152
root@droplet:~# curl http://localhost/time.php;echo
1382986152
root@droplet:~# curl http://localhost/time.php;echo
1382986152

```


If caching works properly, you should see the same timestamp on all requests as the response is cached. </br></br>


Do a recursive listing of the cache location to find the cache of this request.


```
root@droplet:~# ls -lR /etc/nginx/cache/
/etc/nginx/cache/:
total 0
drwx------ 3 www-data www-data 60 Oct 28 18:53 e

/etc/nginx/cache/e:
total 0
drwx------ 2 www-data www-data 60 Oct 28 18:53 18

/etc/nginx/cache/e/18:
total 4
-rw------- 1 www-data www-data 117 Oct 28 18:53 b777c8adab3ec92cd43756226caf618e

```


The naming convention will be explained in the purging section.


We can also make Nginx add a “X-Cache” header to the response, indicating if the cache was missed or hit.


Add the following above the server { } directive:


```
add_header X-Cache $upstream_cache_status;

```


Reload the Nginx service and do a verbose request with curl to see the new header.


```
root@droplet:~# curl -v http://localhost/time.php
* About to connect() to localhost port 80 (#0)
*   Trying 127.0.0.1...
* connected
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET /time.php HTTP/1.1
> User-Agent: curl/7.26.0
> Host: localhost
> Accept: */*
>
* HTTP 1.1 or later with persistent connection, pipelining supported
< HTTP/1.1 200 OK
< Server: nginx
< Date: Tue, 29 Oct 2013 11:24:04 GMT
< Content-Type: text/html
< Transfer-Encoding: chunked
< Connection: keep-alive
< X-Cache: HIT
<
* Connection #0 to host localhost left intact
1383045828* Closing connection #0

```


# Setting Cache Exceptions



Some dynamic content such as authentication required pages shouldn’t be cached. Such content can be excluded from being cached based on server variables like “request_uri,” “request_method,” and “http_cookie.”


Here is a sample configuration which must be used in the server{ } context.


```
#Cache everything by default
set $no_cache 0;

#Don't cache POST requests
if ($request_method = POST)
{
    set $no_cache 1;
}

#Don't cache if the URL contains a query string
if ($query_string != "")
{
    set $no_cache 1;
}

#Don't cache the following URLs
if ($request_uri ~* "/(administrator/|login.php)")
{
    set $no_cache 1;
}

#Don't cache if there is a cookie called PHPSESSID
if ($http_cookie = "PHPSESSID")
{
    set $no_cache 1;
}

```


To apply the “$no_cache” variable to the appropriate directives, place the following lines inside location ~ .php$ { }


```
fastcgi_cache_bypass $no_cache;
fastcgi_no_cache $no_cache;

```


The “fasctcgi_cache_bypass” directive ignores existing cache for requests related to the conditions set by us previously. The “fastcgi_no_cache” directive doesn’t cache the request at all if the specified conditions are met.


# Purging the Cache



The naming convention of the cache is based on the variables we set for the “fastcgi_cache_key” directive.


```
fastcgi_cache_key "$scheme$request_method$host$request_uri";

```


According to these variables, when we requested “http://localhost/time.php” the following would’ve been the actual values:


```
fastcgi_cache_key "httpGETlocalhost/time.php";

```


Passing this string through MD5 hashing would output the following string:


```
b777c8adab3ec92cd43756226caf618e

```


This will form the filename of the cache as for the subdirectories we entered “levels=1:2.”  Therefore, the first level of the directory will be named with 1 character from the last of this MD5 string which is e; the second level will have the last 2 characters after the first level i.e. 18. Hence, the entire directory structure of this cache is as follows:


```
/etc/nginx/cache/e/18/b777c8adab3ec92cd43756226caf618e

```


Based on this cache naming format you can develop a purging script in your favorite language. For this tutorial, I’ll provide a simple PHP script which purges the cache of a __POST__ed URL.


/usr/share/nginx/html/purge.php


Insert


```
<?php
$cache_path = '/etc/nginx/cache/';
$url = parse_url($_POST['url']);
if(!$url)
{
    echo 'Invalid URL entered';
    die();
}
$scheme = $url['scheme'];
$host = $url['host'];
$requesturi = $url['path'];
$hash = md5($scheme.'GET'.$host.$requesturi);
var_dump(unlink($cache_path . substr($hash, -1) . '/' . substr($hash,-3,2) . '/' . $hash));
?>

```


Send a POST request to this file with the URL to purge.


```
curl -d 'url=http://www.example.com/time.php' http://localhost/purge.php

```


The script will output true or false based on whether the cache was purged to not. Make sure to exclude this script from being cached and also restrict access.


Submitted by: <a rel=“author” href=“http://jesin.tk/”>Jesin A</a>


