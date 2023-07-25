# How To Add the gzip Module to Nginx on CentOS 7

```Nginx``` ```Server Optimization``` ```CentOS```

## Introduction


How fast a website will load depends on the size of all of the files that have to be downloaded by the browser. Reducing the size of files to be transmitted can make the website not only load faster, but also cheaper to those who have to pay for their bandwidth usage.


gzip is a popular data compression program. You can configure Nginx to use gzip to compress files it serves on the fly. Those files are then decompressed by the browsers that support it upon retrieval with no loss whatsoever, but with the benefit of smaller amount of data being transferred between the web server and browser.


Because of the way compression works in general, but also how gzip works, certain files compress better than others. For example, text files compress very well, often ending up over two times smaller in result. On the other hand, images such as JPEG or PNG files are already compressed by their nature and second compression using gzip yields little or no results. Compressing files use up server resources, so it is best to compress only those files that will reduce its size considerably in result.


In this guide, we’ll discuss how to configure Nginx installed on your CentOS 7 server to utilize gzip compression to reduce the size of content sent to website visitors.


# Prerequisites


To follow this tutorial, you will need:


- 
One CentOS 7 server with a sudo non-root user

- 
Nginx installed on your server by following the How To Install Nginx on CentOS 7 tutorial


# Step 1 — Creating Test Files


In this step, we will create several test files in the default Nginx directory to text gzip’s compression.


To make a decision what kind of file is served over the network, Nginx does not analyze the file contents because it wouldn’t be fast enough. Instead, it just looks up the file extension to determine its MIME type, which denotes the purpose of the file.


Because of this behavior, the contents of the test files is irrelevant. By naming the files appropriately, we can trick Nginx into thinking that one entirely empty file is an image and the another, for example, is a stylesheet.


In our configuration, Nginx will not compress very small files, so we’re are going to create test files that are exactly 1 kilobyte in size. This will allow us to verify whether Nginx uses compression where it should, compressing one type of files and not doing so with the others.


Create a 1 kilobyte file named test.html in the default Nginx directory using truncate. The extension denotes that it’s an HTML page.


```
sudo truncate -s 1k /usr/share/nginx/html/test.html


```


Let’s create a few more test files in the same manner: one jpg image file, one css stylesheet, and one js JavaScript file.


```
sudo truncate -s 1k /usr/share/nginx/html/test.jpg
sudo truncate -s 1k /usr/share/nginx/html/test.css
sudo truncate -s 1k /usr/share/nginx/html/test.js


```


# Step 2 — Checking the Default Behavior


The next step is to check how Nginx behaves in respect to compression on a fresh installation with the files we have just created.


Let’s check if HTML file named test.html is served with compression. The command requests a file from our Nginx server, and specifies that it is fine to serve gzip compressed content by using an HTTP header (Accept-Encoding: gzip).


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.html


```


In response, you should see several HTTP response headers:


Nginx response headers
```
HTTP/1.1 200 OK
Server: nginx/1.6.3
Date: Fri, 11 Mar 2016 12:53:06 GMT
Content-Type: text/html
Content-Length: 1024
Last-Modified: Fri, 11 Mar 2016 12:48:02 GMT
Connection: keep-alive
ETag: "56e2be82-400"
Accept-Ranges: bytes

```


In the response, there is no mention of gzip whatsoever. This tells us that gzip compression is not enabled on the server. That’s beause on CentOS 7 the support for gzip is entirely disabled in default Nginx configuration. If the compression was enabled, we would see additional header in the output saying Content-Encoding: gzip.


Not only HTML pages, but also every other file on a fresh installation will be served uncompressed. To verify that, you can request our test image named test.jpg in the same way.


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.jpg


```


The result should be virtually identical as before:


Nginx response headers
```
HTTP/1.1 200 OK
Server: nginx/1.6.3
Date: Fri, 11 Mar 2016 12:58:03 GMT
Content-Type: image/jpeg
Content-Length: 1024
Last-Modified: Fri, 11 Mar 2016 12:48:05 GMT
Connection: keep-alive
ETag: "56e2be85-400"
Accept-Ranges: bytes

```


There is no Content-Encoding: gzip header in the output either, which means the file was served without compression.


You can repeat the test with test CSS stylesheet.


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.css


```


Once again, there is no mention of compression in the output.


Nginx response headers for CSS file
```
HTTP/1.1 200 OK
Server: nginx/1.6.3
Date: Fri, 11 Mar 2016 12:59:04 GMT
Content-Type: text/css
Content-Length: 1024
Last-Modified: Fri, 11 Mar 2016 12:48:05 GMT
Connection: keep-alive
ETag: "56e2be85-400"
Accept-Ranges: bytes

```


# Step 3 — Enabling and Configuring Nginx’s gzip Module


The next step is to configure Nginx to to enable compression for all file formats that can benefit from compression.


The gzip module is a core module in Nginx, which means it is already installed but must be enabled and configured.  In a fresh Nginx installation on CentOS 7, all files with the .conf extension from from the /etc/nginx/conf.d directory are automatically loaded. This allows for easy configuration of additional modules.


To enable the Nginx gzip module, create the configuration file named gzip.conf using nano or your favorite text editor.


```
sudo nano /etc/nginx/conf.d/gzip.conf


```


Paste in the following contents.


/etc/nginx/conf.d/gzip.conf
```
##
# `gzip` Settings
#
#
gzip on;
gzip_disable "msie6";

gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_http_version 1.1;
gzip_min_length 256;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;

```


Save and close the file to exit.


Let’s walk through the configuration settings applied here:


- 
gzip on directive enables the Gzip compression.

- 
gzip_disable "msie6" excludes Internet Explorer 6 from the browsers that will receive compressed files, because IE6 does not support gzip at all.

- 
gzip_vary and gzip_proxied settings make sure that proxy servers between the browser and the server will recognize compression correctly.

- 
gzip_comp_level 6 sets how much files will be compressed. The higher the number, the higher the compression level and the resources usage. 6 is a reasonable middle ground.

- 
gzip_http_version 1.1 is used to limit gzip compression to browsers supporting the HTTP/1.1 protocol. If the browser does not support it, there is a good chance it does not support gzip either.

- 
gzip_min_length 256 tells Nginx not to compress files smaller than 256 bytes. Very small files barely benefit from compression.

- 
gzip_types lists all of the MIME types that will be compressed. In this case, the list includes HTML pages, CSS stylesheets, Javascript and JSON files, XML files, icons, SVG images, and web fonts.


To enable the new configuration, restart Nginx.


```
sudo systemctl restart nginx


```


# Step 4 — Verifying the New Configuration


The next step is to check whether changes to the configuration have worked as expected.


We can test this just like we did in step 2, by using curl on each of the test files and examining the output for the Content-Encoding: gzip header.


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.html


```


In response, you should see Content-Encoding: gzip header that was not there before:


Nginx response headers
```
HTTP/1.1 200 OK
Server: nginx/1.6.3
Date: Fri, 11 Mar 2016 13:19:16 GMT
Content-Type: text/html
Last-Modified: Fri, 11 Mar 2016 12:48:02 GMT
Connection: keep-alive
Vary: Accept-Encoding
Content-Encoding: gzip

```


You can test all other files the same way:


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.jpg
curl -H "Accept-Encoding: gzip" -I http://localhost/test.css
curl -H "Accept-Encoding: gzip" -I http://localhost/test.js


```


Now only test.jpg, which is an image file, should stay uncompressed. In both other examples, you should be able to find Content-Encoding: gzip header in the output.


If that is the case, you have configured gzip compression in Nginx successfully!


# Conclusion


Changing Nginx configuration to fully use gzip compression is easy, but the benefits can be immense. Not only visitors with limited bandwidth will receive the site faster but also Google will be happy about the site loading faster. Speed is gaining traction as an important part of modern web and using gzip is one big step to improve it.


