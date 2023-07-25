# How To Customize Your Nginx Server Name After Compiling From Source In CentOS

```Security``` ```Nginx``` ```CentOS```

## Introduction



As a follow up to the article on how to compile nginx from source, this tutorial helps you customize the name of the server on your host. Usually, companies modify the server names for security reasons. If a vulnerability is found in a particular version of the webserver, hackers can replicate it to exploit the behaviour.


Customizing the name of your nginx server requires modifying source code (this tutorial will guide you step by step) and requires re-compilation from the previous article.


# Finding your server’s version



```
curl -I http://example.com/

HTTP/1.1 200 OK
Server: nginx/1.5.6 # <-- this is the version of nginx you currently use
Date: Thu, 17 Nov 2013 20:40:18 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Thu, 17 Nov 2013 20:37:02 GMT
Connection: keep-alive
ETag: "51f18c6e-264"
Accept-Ranges: bytes

```


# Change the Nginx Server Strings



Go back to the nginx source directory from the last tutorial.  You should look at the previous tutorial on compiling from source after the “Downloading the source code” section.


```
cd ~/src/nginx/
vi +49 src/http/ngx_http_header_filter_module.c

```


Find the lines:


```
static char ngx_http_server_string[] = "Server: nginx" CRLF;
static char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;

```


and modify to:


```
static char ngx_http_server_string[] = "Server: the-ocean" CRLF;
static char ngx_http_server_full_string[] = "Server: the-ocean" CRLF;

```


# Recompile Nginx with New Options



You need to follow this guide to look at the configure options or search from your command line history:


```
./configure ... 
make
make install

```


# Stop Displaying Server Version in Configuration



```
vi +19 /etc/nginx/nginx.conf

```


Add the line under http configuration. Repeat for https if you have the section


```
http {
...
server_tokens off;
....

```


# Restart the Nginx Service



We need to restart nginx since the nginx file has changed:


```
service nginx restart

```


# Verify the Results



Let’s verify if we see the server information now:


```
curl -I http://example.com/

HTTP/1.1 200 OK
Server: the-ocean
Date: Thu, 17 Nov 2013 20:50:17 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Thu, 17 Nov 2013 20:37:02 GMT
Connection: keep-alive
ETag: "51f18c6e-264"
Accept-Ranges: bytes

```


<div class=“author”><a href=“http://sair.am/”>By Sairam Kunala</a></div>


