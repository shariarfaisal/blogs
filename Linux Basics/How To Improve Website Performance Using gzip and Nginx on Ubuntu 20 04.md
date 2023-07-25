# How To Improve Website Performance Using gzip and Nginx on Ubuntu 20 04

```Linux Basics``` ```Ubuntu``` ```Open Source``` ```Nginx```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


A website’s performance depends partially on the size of all the files that a user’s browser must download. Reducing the size of those transmitted files can make your website faster. It can also make your website cheaper for those who pay for their bandwidth usage on metered connections.


gzip is a popular data compression program. You can configure Nginx to use gzip to compress the files it serves on the fly. Those files are then decompressed by the browsers that support it upon retrieval with no loss whatsoever, but with the benefit of a smaller amount of data to transfer between the web server and browser. The good news is that compression support is ubiquitous among all major browsers, and there is no reason not to use it.


Because of the way compression works in general and how gzip works, certain files compress better than others. For example, text files compress very well, often ending up over two times smaller. On the other hand, images such as JPEG or PNG files are already compressed by their nature, and second compression using gzip yields little or no results. Compressing files use up server resources, so it is best to compress only files that will benefit from the size reduction.


In this tutorial, you will configure Nginx to use gzip compression. This will reduce the size of content sent to your website’s visitors and improve performance.


# Prerequisites


To follow this tutorial, you will need:


- 
One Ubuntu 20.04 server with a regular, non-root user with sudo privileges. You can learn how to prepare your server by following this initial server setup tutorial.

- 
Nginx installed on your server by following our tutorial, How To Install Nginx on Ubuntu 20.04.


# Step 1 — Creating Test Files


In this step, we will create several test files in the default Nginx directory. We’ll use these files later to check Nginx’s default behavior for gzip’s compression and test that the configuration changes have the intended effect.


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


The next step is to check how Nginx behaves with respect to compressing requested files on a fresh installation with the files we have just created.


# Step 2 — Checking the Default Behavior


Let’s check if the HTML file named test.html is served with compression. The command requests a file from our Nginx server and specifies that it is fine to serve gzip compressed content by using an HTTP header (Accept-Encoding: gzip):


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.html


```


In response, you should see several HTTP response headers:


```
OutputHTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 09 Feb 2021 19:04:25 GMT
Content-Type: text/html
Last-Modified: Tue, 09 Feb 2021 19:03:41 GMT
Connection: keep-alive
ETag: W/"6022dc8d-400"
Content-Encoding: gzip

```


In the last line, you can see the Content-Encoding: gzip header. This tells us that gzip compression was used to send this file. That’s because Nginx has gzip compression enabled automatically even on the fresh Ubuntu 20.04 installation.


However, by default, Nginx compresses only HTML files. Every other file will be served uncompressed, which is less than optimal. To verify that, you can request our test image named test.jpg in the same way:


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.jpg


```


The result should be slightly different than before:


```
OutputHTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 09 Feb 2021 19:05:49 GMT
Content-Type: image/jpeg
Content-Length: 1024
Last-Modified: Tue, 09 Feb 2021 19:03:45 GMT
Connection: keep-alive
ETag: "6022dc91-400"
Accept-Ranges: bytes

```


There is no Content-Encoding: gzip header in the output, which means the file was served without any compression.


You can repeat the test with the test CSS stylesheet:


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.css


```


Once again, there is no mention of compression in the output:


```
OutputHTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 09 Feb 2021 19:06:04 GMT
Content-Type: text/css
Content-Length: 1024
Last-Modified: Tue, 09 Feb 2021 19:03:45 GMT
Connection: keep-alive
ETag: "6022dc91-400"
Accept-Ranges: bytes

```


In the next step, we’ll tell Nginx to compress all sorts of files that will benefit from using gzip.


# Step 3 — Configuring Nginx’s gzip Settings


To change the Nginx gzip configuration, open the main Nginx configuration file in nano or your favorite text editor:


```
sudo nano /etc/nginx/nginx.conf


```


Find the gzip settings section, which looks like this:


/etc/nginx/nginx.conf
```
. . .
##
# `gzip` Settings
#
#
gzip on;
gzip_disable "msie6";

# gzip_vary on;
# gzip_proxied any;
# gzip_comp_level 6;
# gzip_buffers 16 8k;
# gzip_http_version 1.1;
# gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
. . .

```


You can see that gzip compression is indeed enabled by the gzip on directive, but several additional settings are commented out with # sign and have no effect. We’ll make several changes to this section:


- Enable the additional settings by uncommenting all of the commented lines (i.e., by deleting the # at the beginning of the line)
- Add the gzip_min_length 256; directive, which tells Nginx not to compress files smaller than 256 bytes. Very small files barely benefit from compression.
- Append the gzip_types directive with additional file types denoting web fonts, icons, XML feeds, JSON structured data, and SVG images.

After these changes have been applied, the settings section should look like this:


/etc/nginx/nginx.conf
```
. . .
##
# `gzip` Settings
#
#
gzip on;
gzip_disable "msie6";

gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_min_length 256;
gzip_types
  application/atom+xml
  application/geo+json
  application/javascript
  application/x-javascript
  application/json
  application/ld+json
  application/manifest+json
  application/rdf+xml
  application/rss+xml
  application/xhtml+xml
  application/xml
  font/eot
  font/otf
  font/ttf
  image/svg+xml
  text/css
  text/javascript
  text/plain
  text/xml;
. . .

```


Save and close the file to exit.


To enable the new configuration, restart Nginx:


```
sudo systemctl restart nginx


```


Next, let’s make sure our new configuration works.


# Step 4 — Verifying the New Configuration


Execute the same request as before for the test HTML file:


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.html


```


The response will stay the same since compression has already been enabled for that filetype:


```
OutputHTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 09 Feb 2021 19:04:25 GMT
Content-Type: text/html
Last-Modified: Tue, 09 Feb 2021 19:03:41 GMT
Connection: keep-alive
ETag: W/"6022dc8d-400"
Content-Encoding: gzip

```


However, if we request the previously uncompressed CSS stylesheet, the response will be different:


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.css


```


Now gzip is compressing the file:


```
OutputHTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 09 Feb 2021 19:21:54 GMT
Content-Type: text/css
Last-Modified: Tue, 09 Feb 2021 19:03:45 GMT
Connection: keep-alive
Vary: Accept-Encoding
ETag: W/"6022dc91-400"
Content-Encoding: gzip

```


From all test files created in step 1, only the test.jpg image file should stay uncompressed. We can test this the same way:


```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.jpg


```


There is no gzip compression:


```
OutputHTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 09 Feb 2021 19:25:40 GMT
Content-Type: image/jpeg
Content-Length: 1024
Last-Modified: Tue, 09 Feb 2021 19:03:45 GMT
Connection: keep-alive
ETag: "6022dc91-400"
Accept-Ranges: bytes

```


Here the Content-Encoding: gzip header is not present in the output as expected.


If that is the case, you have configured gzip compression in Nginx successfully.


# Conclusion


Changing Nginx configuration to utilize gzip compression is easy, but the benefits can be immense. Not only will visitors with limited bandwidth receive the site faster, but all other users will also see noticeable speed gains. Search engines will be happy about the site loading quicker too. Loading speed is now a crucial metric in how the search engines rank websites, and using gzip is one big step to improve it.


