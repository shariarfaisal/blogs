# How To Implement Browser Caching with Nginx s header Module on Ubuntu 20 04

```Ubuntu``` ```Nginx```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


The faster a website loads, the more likely a visitor is to stay. When websites are full of images and interactive content run by scripts loaded in the background, opening a website is not a simple task. It consists of requesting many different files from the server one by one. Minimizing the quantity of these requests is one way to speed up your website.


One method for improving website performance is browser caching. Browser caching tells the browser that it can reuse local versions of downloaded files instead of requesting the server for them again and again. To do this, you must introduce new HTTP response headers that tell the browser how to behave.


Nginx’s header module can help you accomplish browser caching. You can use this module to add any arbitrary headers to the response, but its major role is to properly set caching headers. In this tutorial, we will use Nginx’s header module to implement browser caching.


# Prerequisites


To follow this tutorial, you will need:


- 
One Ubuntu 20.04 server with a regular, non-root user with sudo privileges. You can learn how to prepare your server by following this Initial Server Setup tutorial.

- 
Nginx installed on your server by following the How To Install Nginx on Ubuntu 20.04 tutorial.


# Step 1 — Creating Test Files


In this step, we will create several test files in the default Nginx directory. We’ll use these files later to check Nginx’s default behavior and then to test that browser caching is working.


To infer what kind of file is served over the network, Nginx does not analyze the file contents; that would be prohibitively slow. Instead, it looks up the file extension to determine the file’s MIME type, which denotes its purpose.


Because of this behavior, the content of our test files is irrelevant. By naming the files appropriately, we can trick Nginx into thinking that, for example, one entirely empty file is an image and another is a stylesheet.


Create a file named test.html in the default Nginx directory using truncate. This extension denotes that it’s an HTML page:


```
sudo truncate -s 1k /var/www/html/test.html


```


Let’s create a few more test files in the same manner: one jpg image file, one css stylesheet, and one js JavaScript file:


```
sudo truncate -s 1k /var/www/html/test.jpg
sudo truncate -s 1k /var/www/html/test.css
sudo truncate -s 1k /var/www/html/test.js


```


The next step is to check how Nginx behaves with respect to sending caching control headers on a fresh installation with the files we have just created.


# Step 2 — Checking the Default Behavior


By default, all files will have the same default caching behavior. To explore this, we’ll use the HTML file we created in step 1, but you can run these tests with any example files.


So, let’s check if test.html is served with any information regarding how long the browser should cache the response. The following command requests a file from our local Nginx server and shows the response headers:


```
curl -I http://localhost/test.html


```


You will see several HTTP response headers:


```
Output: Nginx response headersHTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 02 Feb 2021 19:03:21 GMT
Content-Type: text/html
Content-Length: 1024
Last-Modified: Tue, 02 Feb 2021 19:02:58 GMT
Connection: keep-alive
ETag: "6019a1e2-400"
Accept-Ranges: bytes

```


In the second to last line you will find the ETag header, which contains a unique identifier for this particular revision of the requested file. If you execute the previous curl command repeatedly, you will find the exact same ETag value.


When using a web browser, the ETag value is stored and sent back to the server with the If-None-Match request header when the browser wants to request the same file again — for example, when refreshing the page.


We can simulate this on the command line with the following command. Make sure you change the ETag value in this command to match the ETag value in your previous output:


```
curl -I -H 'If-None-Match: "6019a1e2-400"' http://localhost/test.html


```


The response will now be different:


```
Output: Nginx response headersHTTP/1.1 304 Not Modified
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 02 Feb 2021 19:04:09 GMT
Last-Modified: Tue, 02 Feb 2021 19:02:58 GMT
Connection: keep-alive
ETag: "6019a1e2-400"

```


This time, Nginx will respond with 304 Not Modified. It won’t send the file over the network again; instead, it will tell the browser that it can reuse the file it already has downloaded locally.


This is useful because it reduces network traffic, but it’s not good enough to achieve good caching performance. The problem with ETag is that browsers always send a request to the server asking if it can reuse its cached file. Even though the server responds with a 304 instead of sending the file again, it still takes time to make the request and receive the response.


In the next step, we will use the headers module to append caching control information. This will make the browser cache some files locally without explicitly asking the server if its fine to do so.


# Step 3 — Configuring Cache-Control and Expires Headers


In addition to the ETag file validation header, there are two caching control response headers: Cache-Control and Expires. Cache-Control is the newer version, with more options than Expires and is generally more useful if you want finer control over your caching behavior.


If these headers are set, they can tell the browser that the requested file can be kept locally for a certain amount of time (including forever) without requesting it again. If the headers are not set, browsers will always request the file from the server, expecting either 200 OK or 304 Not Modified responses.


We can use the header module to set these HTTP headers. The header module is a core Nginx module, which means it doesn’t need to be installed separately to be used.


To add the header module, open the default Nginx configuration file in nano or your favorite text editor:


```
sudo nano /etc/nginx/sites-available/default


```


Find the server configuration block:


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


Add the following two new sections here: one before the server block, to define how long to cache different file types, and one inside it, to set the caching headers appropriately:


Modified /etc/nginx/sites-available/default
```
. . .
# Default server configuration
#

# Expires map
map $sent_http_content_type $expires {
    default                    off;
    text/html                  epoch;
    text/css                   max;
    application/javascript     max;
    ~image/                    max;
    ~font/                     max;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    expires $expires;
. . .

```


The section before the server block is a new map block that defines the mapping between the file type and how long that kind of file should be cached.


We’re using several different settings in this map:


- 
The default value is set to off, which will not add any caching control headers. It’s a safe bet for the content, we have no particular requirements on how the cache should work.

- 
For text/html, we set the value to epoch. This is a special value that results explicitly in no caching, which forces the browser to always ask if the website itself is up to date.

- 
For text/css and application/javascript, which are stylesheets and JavaScript files, we set the value to max. This means the browser will cache these files for as long as possible, reducing the number of requests considerably given that there are typically many of these files.

- 
The last two settings are for ~image/ and ~font/, which are regular expressions that will match all file types containing image/ or font/ in their MIME type name (like image/jpg, image/png or font/woff2). Like stylesheets, both pictures and web fonts on websites can be safely cached to speed up page-loading times, so we set this to max as well.



Note: These are just a few examples of the most common MIME types used on websites. You can get acquainted with the more extensive list of such types on Common MIME types site and add others to the map that you might find useful in your case.

Inside the server block, the expires directive (a part of the headers module) sets the caching control headers. It uses the value from the $expires variable set in the map. This way, the resulting headers will be different depending on the file type.


Save and close the file to exit.


To enable the new configuration, restart Nginx:


```
sudo systemctl restart nginx


```


Next, let’s make sure our new configuration works.


# Step 4 — Testing Browser Caching


Execute the same request as before for the test HTML file:


```
curl -I http://localhost/test.html


```


This time the response will be different. You will see two additional HTTP response headers:


```
OutputHTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 02 Feb 2021 19:10:13 GMT
Content-Type: text/html
Content-Length: 1024
Last-Modified: Tue, 02 Feb 2021 19:02:58 GMT
Connection: keep-alive
ETag: "6019a1e2-400"
Expires: Thu, 01 Jan 1970 00:00:01 GMT
Cache-Control: no-cache
Accept-Ranges: bytes

```


The Expires header shows a date in the past and Cache-Control is set with no-cache, which tells the browser to always ask the server if there is a newer version of the file (using the ETag header, like before).


You’ll find a difference in response with the test image file:


```
curl -I http://localhost/test.jpg


```


Note the new output:


```
OutputHTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 02 Feb 2021 19:10:42 GMT
Content-Type: image/jpeg
Content-Length: 1024
Last-Modified: Tue, 02 Feb 2021 19:03:02 GMT
Connection: keep-alive
ETag: "6019a1e6-400"
Expires: Thu, 31 Dec 2037 23:55:55 GMT
Cache-Control: max-age=315360000
Accept-Ranges: bytes

```


In this case, Expires shows the date in the distant future, and Cache-Control contains max-age information, which tells the browser how long it can cache the file in seconds. This tells the browser to cache the downloaded image for as long as it can, so any subsequent appearances of this image will use local cache and not send a request to the server at all.


The result should be similar for both test.js and test.css, as both JavaScript and stylesheet files are set with caching headers too.


This means the cache control headers have been configured properly and your website will benefit from the performance gain and less server requests due to browser caching. You should customize the caching settings based on your website’s content, but the defaults in this article are a reasonable place to start.


# Conclusion


The headers module can be used to add any arbitrary headers to the response, but properly setting caching control headers is one of its most useful applications. It increases performance for the website users, especially on networks with higher latency, like mobile carrier networks. It can also lead to better results on search engines that factor speed tests into their results. Setting browser caching headers is a crucial recommendation from Google’s PageSpeed and similar performance testing tools.


You can find more detailed information about the headers module in Nginx’s official headers module documentation.


