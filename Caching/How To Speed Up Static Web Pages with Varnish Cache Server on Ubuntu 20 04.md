# How To Speed Up Static Web Pages with Varnish Cache Server on Ubuntu 20 04

```Ubuntu``` ```Caching```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Varnish is a versatile reverse HTTP proxy that caches responses from backend servers in memory so they are served quickly when requested again. It uses HTTP headers to determine whether to cache the responses to a particular request. By default, it does not cache responses with cookies because those are considered client-specific requests; however you can change this setting in the configuration file.


Besides acting as a caching server, Varnish can be used as a:


- Web application firewall
- DDoS attack defender
- Load balancer
- Quick fix for unstable backends
- HTTP router

There are three locations where the HTTP cache can be saved:


- Browser: This cache is saved on users’ browsers. It is user-specific and can be used to serve content instead of sending requests to web sites.
- Proxy: A proxy is an intermediate server that sits between users and servers. It is usually deployed by ISPs and can be used to cache responses that will be requested by multiple users.
- Reverse Proxy: This kind of proxy is created by the web site’s administrator and can be used to serve content from the network’s edge instead of sending requests to back end servers. This is the kind of cache you will create in this tutorial.


Note: For more information about HTTP caching, see this tutorial on HTTP headers and caching strategies.

In this tutorial, you will set up Varnish as a caching reverse proxy server. You’ll then test the setup with Varnish against a non-caching configuration using wrk.


# Prerequisites


To complete this tutorial, you will need:


- One Ubuntu 20.04 server with at least 2 GB of RAM
- A non-root user with sudo privileges as described in this Ubuntu 20.04 initial server setup guide

# Step 1 — Installing Varnish And Apache


To start, you’ll install Apache and Varnish. First update apt-get, and then install Apache with these commands:


```
sudo apt-get update
sudo apt-get install apache2 -y


```


You’ll see output indicating that Apache is being installed.


After the Apache installation process is complete, install Varnish with this command:


```
sudo apt-get install varnish -y


```


You’ll see output indicating that Varnish is being installed.


Next, make sure both packages installed correctly. First, use this command to check the status of Apache:


```
sudo systemctl status apache2


```


The output will look similar to this:


```
Outputroot@ubuntu-s-1vcpu-2gb-fra1-01:~# sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-08-04 18:58:39 UTC; 4min 10s ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 2279 (apache2)
      Tasks: 55 (limit: 2344)
     Memory: 5.0M
     CGroup: /system.slice/apache2.service
             ├─2279 /usr/sbin/apache2 -k start
             ├─2281 /usr/sbin/apache2 -k start
             └─2282 /usr/sbin/apache2 -k start

Aug 04 18:58:39 ubuntu-s-1vcpu-2gb-fra1-01 systemd[1]: Starting The Apache HTTP Server...
Aug 04 18:58:39 ubuntu-s-1vcpu-2gb-fra1-01 apachectl[2278]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' di>
Aug 04 18:58:39 ubuntu-s-1vcpu-2gb-fra1-01 systemd[1]: Started The Apache HTTP Server.

```


Press the Q key to exit the status command.


Next, check the status of Varnish with this command:


```
sudo systemctl status varnish


```


The output will look similar to this:


```
Outputroot@ubuntu-s-1vcpu-2gb-fra1-01:~# sudo systemctl status varnish
● varnish.service - Varnish HTTP accelerator
     Loaded: loaded (/lib/systemd/system/varnish.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-08-04 18:59:09 UTC; 4min 41s ago
       Docs: https://www.varnish-cache.org/docs/6.1/
             man:varnishd
   Main PID: 3423 (varnishd)
      Tasks: 217 (limit: 2344)
     Memory: 10.7M
     CGroup: /system.slice/varnish.service
             ├─3423 /usr/sbin/varnishd -j unix,user=vcache -F -a :6081 -T localhost:6082 -f /etc/varnish/default.vcl -S /etc/varnish/secret -s malloc,256m
             └─3447 /usr/sbin/varnishd -j unix,user=vcache -F -a :6081 -T localhost:6082 -f /etc/varnish/default.vcl -S /etc/varnish/secret -s malloc,256m

Aug 04 18:59:09 ubuntu-s-1vcpu-2gb-fra1-01 systemd[1]: Started Varnish HTTP accelerator.
Aug 04 18:59:10 ubuntu-s-1vcpu-2gb-fra1-01 varnishd[3423]: Debug: Version: varnish-6.2.1 revision 9f8588e4ab785244e06c3446fe09bf9db5dd8753
Aug 04 18:59:10 ubuntu-s-1vcpu-2gb-fra1-01 varnishd[3423]: Version: varnish-6.2.1 revision 9f8588e4ab785244e06c3446fe09bf9db5dd8753
Aug 04 18:59:10 ubuntu-s-1vcpu-2gb-fra1-01 varnishd[3423]: Debug: Platform: Linux,5.4.0-73-generic,x86_64,-junix,-smalloc,-sdefault,-hcritbit
Aug 04 18:59:10 ubuntu-s-1vcpu-2gb-fra1-01 varnishd[3423]: Platform: Linux,5.4.0-73-generic,x86_64,-junix,-smalloc,-sdefault,-hcritbit
Aug 04 18:59:10 ubuntu-s-1vcpu-2gb-fra1-01 varnishd[3423]: Debug: Child (3447) Started
Aug 04 18:59:10 ubuntu-s-1vcpu-2gb-fra1-01 varnishd[3423]: Child (3447) Started
Aug 04 18:59:10 ubuntu-s-1vcpu-2gb-fra1-01 varnishd[3423]: Info: Child (3447) said Child starts
Aug 04 18:59:10 ubuntu-s-1vcpu-2gb-fra1-01 varnishd[3423]: Child (3447) said Child starts

```


If you do not see both services up and running, wait a few minutes until they are fully loaded, and keep both of them running.


Now that you have Apache2 Varnish, installed, you’ll give Varnish something to serve, in this case Apache’s static web page.


# Step 2 — Configuring Varnish To Serve Apache’s Static Web Page


In the previous step, you installed Varnish, and next you’ll need to configure it. By default, Varnish listens on port 6081 and connects to a local web server on port 8080. You’ll change that to serve the Apache static site from Apache server.


First, you’ll change Varnish’s listening port to 8080. Usually you would want the listening port to be 80, but because you are running Apache and Varnish on the same server, you’ll use port 8080 for Varnish and port 80 for Apache.


There is no configuration option to change the listening port for Varnish, so you’ll do it using the command line.  You’ll create a file called customexec.conf in a new directory called varnish.service.d in /etc/systemd/system/ that will change the default ports.


Use the mkdir command to create the new directory:


```
sudo mkdir /etc/systemd/system/varnish.service.d


```


Use your favorite text editor to create a new file called customexec.conf :


```
sudo nano /etc/systemd/system/varnish.service.d/customexec.conf


```


In customexec.conf, add the following content:


```
/etc/systemd/system/varnish.service.d/customexec.conf file[Service]
ExecStart=
ExecStart=/usr/sbin/varnishd -j unix,user=vcache -F -a :8080 -T localhost:6082 -f /etc/varnish/default.vcl -S /etc/varnish/secret -s malloc,256m

```


In this file you are changing the Service section of the Varnish configuration. First you remove the old value for the ExecStart option, and then you assign a new value for it.


The new value specifies the binary file used to run Varnish with the following options:


- 
-j: Specifies the jailing mechanism to use. Varnish jails are used to reduce the permissions for the varnish process over various platform-specific methods. Here you’re using the unix mechanism and the user vcache to limit the permissions. This is default for varnish on Ubuntu systems.

- 
-F: Indicates that the server should run in the foreground, because systemd expects the main process to keep running so it can track it, and not fork a new process and die.

- 
-a: This flag is used to specify the IP address and port for accepting client connections. The IP in this case is empty, which means the server will accept all IPs.  The port is set to 8080.

- 
-T: This flag specifies the IP address and port for management interface, in this case localhost and port 6082.

- 
-f: This flag specifies the default VCL file for Varnish configuration. You will edit this file later in this tutorial to configure Varnish to connect to the Apache server.

- 
-S: This flag specifies a shared secret file for authorizing access to the management interface. The /etc/varnish/secret value is the default for Varnish on Ubuntu. You will not use the secret file in this tutorial.

- 
-s: This flag indicates where and how to store objects. The value malloc,256m is the default one for Vanish. It means to store various Varnish objects in memory using the malloc system call and a maximum size of 256 megabytes. Other possible values are default, which uses umem when malloc is not available, or file, which stores objects in a file on the disk.


Save and close the customexec.conf file. Then execute this command to reload the systemd services file from disk:


```
sudo systemctl daemon-reload


```


Then restart Varnish for changes to take effect.


```
sudo systemctl restart varnish


```


You won’t see any output from these last two commands. To make sure that Varnish is now listening on port 8080, use the netstat command to display all listening TCP sockets on the server.


```
sudo netstat -ltnp | grep 8080


```


You’ll see output that looks like this:


```
Outputtcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      18689/varnishd
tcp6       0      0 :::8080                 :::*                    LISTEN      18689/varnishd

```


Now that Varnish is running and listening to port 8080, you need to edit the default configuration file located at /etc/varnish/default.vcl:


```
sudo nano /etc/varnish/default.vcl


```


Navigate to the backend default block, and then change .port to 80, as shown here:


```
default.vcl file
# Default backend definition. Set this to point to your content server.
backend default {
    .host = "127.0.0.1";
    .port = "80";
}

```


Save and close the default.vcl file, then restart Varnish with this command:


```
sudo systemctl restart varnish


```


If everything is fine, there won’t be any output. Open http://your_server_ip:8080 in your browser, and you’ll see the Apache static site, opened using Varnish.


You now have Apache and Varnish running together on the same Droplet, with Apache listening to port 80 and Varnish to port 8080. Next, you’ll compare the response times of both servers using the wrk tool.


# Step 3 — Testing Varnish Using wrk


wrk is a modern HTTP benchmarking tool. It is written in C, and can be used to load test web servers with many requests per second. In this step, you’ll use wrk to run tests against Apache and Varnish and then compare the results.


First you’ll need to install wrk by building it from source. Start by installing some build tools for C and git, which are required for building wrk from source:


```
sudo apt-get install build-essential libssl-dev git unzip -y


```


Then clone the git repository for wrk into the wrk directory:


```
git clone https://github.com/wg/wrk.git wrk


```


Change to that new directory:


```
cd wrk


```


Build the wrk executable with the make command:


```
make


```


Copy wrk to the /usr/local/bin directory so you can access it from anywhere in your directory structure:


```
sudo cp wrk /usr/local/bin


```


Now that you have wrk installed, use it to test the responsiveness of Apache with this command:


```
wrk -t2 -c1000 -d30s --latency http://server_ip/


```


This command uses the following arguments:


- -t2: This means run two threads.
- -c1000: Keep 1000 HTTP connections open.
- -d30s: Run the test for 30 seconds.
- --latency: Print latency statistics.

Wait 30 seconds until the test is done, and you’ll see output similar to this:


```
outputRunning 30s test @ http://68.183.115.151/
  2 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    44.45ms  104.50ms   1.74s    91.20%
    Req/Sec     8.29k     1.07k   12.40k    71.00%
  Latency Distribution
     50%   11.59ms
     75%   22.73ms
     90%  116.16ms
     99%  494.90ms
  494677 requests in 30.04s, 5.15GB read
  Socket errors: connect 0, read 8369, write 0, timeout 69
Requests/sec:  16465.85
Transfer/sec:    175.45MB

```


In this test, the average latency is 44.45ms, there were 494,677 total requests, 8,369 read errors, and 69 timeout errors. The exact numbers will vary in your installation.


Now run the same test again for the Varnish server using this command:


```
wrk -t2 -c1000 -d30s --latency http://server_ip:8080/


```


Wait 30 seconds until the test is done, and you’ll see output similar to this:


```
outputRunning 30s test @ http://68.183.115.151:8080/
  2 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    14.41ms   13.70ms 602.49ms   90.05%
    Req/Sec     6.67k   401.10     8.74k    83.33%
  Latency Distribution
     50%   13.03ms
     75%   17.69ms
     90%   24.72ms
     99%   58.22ms
  398346 requests in 30.06s, 4.18GB read
  Socket errors: connect 0, read 19, write 0, timeout 0
Requests/sec:  13253.60
Transfer/sec:    142.48MB

```


The output you see will likely be somewhat different, but the latency will be lower for Varnish than for Apache. In this case, the average latency is 14.41ms, there were 398,346 total requests, and no errors.


In these tests, with Apache the average response time was 44.45ms with 8,438 errors, while Varnish achieved an increase in speed to 14.41ms, and also had no errors. This is because Varnish cached the response in memory and served it for later requests, unlike Apache, which needs to read from disk almost every time the resource is requested.


# Conclusion


In this tutorial, you configured Varnish as a reverse proxy caching server for a static web site. You saw how to use basic HTTP caching to improve performance, and you used wrk to run load tests for the Apache and Varnish servers to compare the results.


You’ve seen that the Varnish cache server speeds up your static site by serving content from main memory and not requesting it from the back end Apache server every time a new request arrives. For more information about other uses of Varnish, see the official documentation.


