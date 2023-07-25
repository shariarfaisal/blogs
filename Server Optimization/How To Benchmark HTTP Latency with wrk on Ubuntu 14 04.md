# How To Benchmark HTTP Latency with wrk on Ubuntu 14 04

```Docker``` ```Ubuntu``` ```Server Optimization```

## Introduction


This article focuses on an open-source HTTP benchmarking tool called wrk, which measures the latency of your HTTP services at high loads.


Latency refers to the time interval between the moment the request was made (by wrk) and the moment the response was received (from the service). This can be used to simulate the latency a visitor would experience on your site when visiting it using a browser or any other method that sends HTTP requests.


wrk is useful for testing any website or application that relies on HTTP, such as:


- Rails and other Ruby applications
- Express and other JavaScript applications
- PHP applications
- Static websites running on web servers
- Sites and applications behind load balancers like Nginx
- Your caching layer

Tests can’t be compared to real users, but they should give you a good estimate of expected latency so you can better plan your infrastructure. Tests can also give you insight into your performance bottlenecks.


wrk is open source and can be found on GitHub.


It’s very stable and allows simulating high loads thanks to its multithreaded nature. wrk’s greatest feature is its ability to integrate Lua scripts, which adds many possibilities like:


- Benchmarking requests with cookies
- Custom reporting
- Benchmarking multiple URLs - something the popular ab, the Apache HTTP server benchmarking tool, can’t

# Prerequisites


The infrastructure which we will use during this tutorial is presented in the diagram below:





As you can see we will use wrk in a very simple scenario. We will benchmark an Express on Node.js application.


We will spin up two Droplets: one for wrk, which generates the load, and the other one for the application. If they were on the same box, they would compete for resources and our results wouldn’t be reliable.


The machine which benchmarks should be strong enough to handle the stressed system, but in our case the application is so simple that we will use machines of the same size.


- Spin up two Droplets in the same region since they will communicate by private IP
- Call one Droplet wrk1 and the other app1, to follow along in this tutorial
- Select 2 GB of memory
- Select Ubuntu 14.04
- Select Private Networking in the Available Settings section
- Create a sudo user on each server

Smaller Droplets will work too, but you should expect more latency in the test results. In a real testing environment, your app server should be the same size you intend to use in production.





If you need help setting up Droplets, please refer to this article.


# Step 1 — Both Servers: Install Docker


To make our lives easier we will use Docker, so we can start wrk and our application inside containers. That lets us skip setting up a Node.js evironment, npm modules, and deb packages; all we need is to download and run the appropriate container. The saved time will be invested in learning wrk.


If you’re not familiar with Docker, you can read an introduction to it here.


Note: The commands in this section should be executed on both Droplets.


To install Docker, log in to your servers and execute the following commands. First, update the package lists:


```
sudo apt-get update


```


Install Wget and cURL:


```
sudo apt-get install -y wget curl


```


Download and install Docker:


```
sudo wget -qO- https://get.docker.com/ | sh


```


Add your user to the docker group so you can execute Docker commands without sudo:


```
sudo gpasswd -a ${USER} docker
sudo service docker restart
newgrp docker


```



If you are using a different Linux distribution, Docker has installation documentation that will probably cover your case.

To verify that docker is installed correctly use this command:


```
docker --version


```


You should get the following or similar output:


```
OutputDocker version 1.7.1, build 786b29d

```


# Step 2 — Prepare Test Application


Execute these commands on the app1 Droplet.


For testing purposes, the author published a Docker image in the public Docker registry. It contains a HTTP debugging application written in Node.js. It’s not a performance beast (we won’t break any records today) but it’s enough for testing and debugging. You can check out the source code here.


Of course, in a real-life scenario, you would want to test your own application.


Before we start the application, let’s save the Droplet’s private IP address to a variable called APP1_PRIVATE_IP:


```
export APP1_PRIVATE_IP=$(sudo ifconfig eth1 | egrep -o "inet addr:[^ ]*" | awk -F ":" '{print $2}')


```


You can view the private IP with:


```
echo $APP1_PRIVATE_IP


```


Output:


```
Output10.135.232.163

```


Your private IP address will be different, so please make a note of it.


You can also get the private IP from the digitalocean.com control panel. Simply choose your Droplet and then go to the Settings section as presented in the image below:





Now start the application simply by executing this command:


```
docker run -d -p $APP1_PRIVATE_IP:3000:3000 --name=http-debugging-application czerasz/http-debugger


```


The command above will first download the required Docker image, and then run a Docker container. The container is started in detached mode which simply means that it will run in the background. The option -p $APP1_PRIVATE_IP:3000:3000 will proxy all the traffic to and from the local container on port 3000, and to and from the host private IP on port 3000.


A diagram below describes this situation:





Now test with curl to see if the application is running:


```
curl -i -XPOST http://$APP1_PRIVATE_IP:3000/test -d 'test=true'


```


Expected output:


```
OutputHTTP/1.1 200 OK
X-Powered-By: Express
X-Debug: true
Content-Type: text/html; charset=utf-8
Content-Length: 2
ETag: W/"2-79dcdd47"
Date: Wed, 13 May 2015 16:25:37 GMT
Connection: keep-alive

ok

```


The application is very simple and returns just an ok message. So each time wrk requests this application, it will get a small ok message back.


The most important part is that we can see what requests are made by wrk to our application by analyzing the application logs.


View the application logs with the following command:


```
docker logs -f --tail=20 http-debugging-application


```


Your sample output should look like this:


```
Output[2015-05-13 16:25:37] Request 1

POST/1.1 /test on :::3000

Headers:
 - user-agent: curl/7.38.0
 - host: 0.0.0.0:32769
 - accept: */*
 - content-length: 9
 - content-type: application/x-www-form-urlencoded

No cookies

Body:
test=true

```


You can leave this running while you run your benchmark tests, if you’d like. Exit the tail with CTRL-C.


# Step 3 — Install wrk


Log in to the wrk1 server and get ready to install wrk.


Since we have Docker it’s very easy. Just download the williamyeh/wrk image from the Docker registry hub with this command:


```
docker pull williamyeh/wrk


```


The command above downloads a Docker image that contains wrk. We don’t need to build wrk, nor install any additional packages. To run wrk (inside a container) we only need to start a container based on this image, which we’ll do soon.


The download should take just few seconds because the image is very small - less than 3 MB. If you would like to install wrk directly on your favourite Linux distribution visit this wiki page and follow the instructions.


We’ll also set the APP1_PRIVATE_IP variable on this server. We need the private IP address from the app1 Droplet.


Export the variable with:


```
export APP1_PRIVATE_IP=10.135.232.163


```


Remember to change the 10.135.232.163 IP address to your app1 Droplet’s private IP. This variable will be saved in the current session only, so remember to re-set it the next time you log in to use wrk.


# Step 4 — Run a wrk Benchmark Test


In this section we will finally see wrk in action.


All the commands in this section should be executed on the wrk1 Droplet.


Let’s see the options wrk has available for us. Running the wrk container with just the --version flag will print out a brief summary of its usage:


```
docker run --rm williamyeh/wrk --version


```


Output:


```
Outputwrk 4.0.0 [epoll] Copyright (C) 2012 Will Glozer
Usage: wrk <options> <url>
  Options:
    -c, --connections <N>  Connections to keep open
    -d, --duration    <T>  Duration of test
    -t, --threads     <N>  Number of threads to use

    -s, --script      <S>  Load Lua script file
    -H, --header      <H>  Add header to request
        --latency          Print latency statistics
        --timeout     <T>  Socket/request timeout
    -v, --version          Print version details

  Numeric arguments may include a SI unit (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h)

```


Now that we have a good overview, let’s compose the command to run our test. Note that this command won’t do anything yet, since we’re not running it from inside the container.


The simplest case we could run with wrk is:


```
wrk -t2 -c5 -d5s -H 'Host: example.com' --timeout 2s http://$APP1_PRIVATE_IP:3000/

```


Which means:


- -t2: Use two separate threads
- -c5: Open six connections (the first client is zero)
- -d5s: Run the test for five seconds
- -H 'Host: example.com': Pass a Host header
- --timeout 2s: Define a two-second timeout
- http://$APP1_PRIVATE_IP:3000/ The target application is listening on $APP1_PRIVATE_IP:3000
- Benchmark the / path of our application

This can also be described as six users that request our home page repeatedly for five seconds.


The illustration below shows this situation:






Please remember that connections can’t be compared to real users because real users would also download CSS, image, and JavaScript files while viewing your home page.

Here’s the actual command for the test:


Let’s run the described scenario in our wrk Docker container:


```
docker run --rm williamyeh/wrk -t2 -c5 -d5s -H 'Host: example.com' --timeout 2s http://$APP1_PRIVATE_IP:3000/


```


Wait a few seconds for the test to run, and look at the results, which we’ll analyze in the next step.


# Step 5 — Evaluate the Output


Output:


```
OutputRunning 5s test @ http://10.135.232.163:3000
  2 threads and 5 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.82ms    2.64ms  26.68ms   85.81%
    Req/Sec   550.90    202.40     0.98k    68.00%
  5494 requests in 5.01s, 1.05MB read
Requests/sec:   1096.54
Transfer/sec:    215.24KB

```


- 
Current configuration summary:
  Running 5s test @ http://10.135.232.163:3000
    2 threads and 5 connections

Here we can see a brief summary of our benchmark configuration. The benchmark took 5 seconds, the benchmarked machine IP is 10.135.232.163, and the test used two threads.

- 
Normal distribution parameters for the latency and req/sec statistics:
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.82ms    2.64ms  26.68ms   85.81%
    Req/Sec   550.90    202.40     0.98k    68.00%

This part shows us the normal distribution details for our benchmark - what parameters a Gaussian function would have.
Benchmarks don’t always have normal distributions, and that’s why these results might be misleading. Therefore always look at the Max and +/- Stdev values. If those values are high, then you can expect that your distribution might have a heavy tail.

- 
Statistics about the request numbers, transferred data, and throughput:
    5494 requests in 5.01s, 1.05MB read
  Requests/sec:   1096.54
  Transfer/sec:    215.24KB

Here we see that during the time of 5.01 seconds, wrk could do 5494 requests and transfer 1.05MB of data. Combined with simple math (total number of requrests/benchmark duration) we get the result of 1096.54 requests per second.


Generally, the more clients you set, the fewer requests per second you should get. The latency will also grow. This is because the application will be under heavier load.


What results are best?


Your goal is to keep the Requests/sec as high as possible and the Latency as low as possible.


Ideally, the latency shouldn’t be too high, at least for web pages. The limit for a page load time with assets is optimal when it’s around two seconds or less.


Now you will probably ask yourself: Is 550.90 Requests/sec with a latency of 3.82ms a good result? Unfortunately, there is no easy answer. It depends on many factors like:


- Number of clients, as we discussed before
- Server resources - is it a big or small instance?
- Number of machines which serve the application
- Type of your service - is it a cache which serves static files or an ad server which serves dynamic responses?
- Database type, database cluster size, type of database connection
- Request and response type - is it a small AJAX request or a fat API call?
- And many others

# Step 6 — Take Action to Improve Latency


If you are not satisfied with your service performance, you can:


- Tune your service - check your code and see what can be done more efficiently
- Check your database to see if it’s your bottleneck
- Scale vertically - add resources to your machine
- Scale horizontally - add another instance of your service and add it to the load balancer
- Add a caching layer

For a more detailed discussion of application improvements, check out 5 Ways to Improve your Production Web Application Server Setup.


Remember to benchmark your service after applying changes to it - only then can you be sure that your service has improved.


That’s it, you might think, if there wasn’t this Lua thing . . .


# Simulate Advanced HTTP Requests with Lua Scripts


Because wrk has a built-in LuaJIT (Just-In-Time Compiler for Lua) it can be extended with Lua scripts. As mentioned in the introduction, this adds a lot of functionality to wrk.


Using a Lua script with wrk is simple. Just append the file path to the -s flag.


Because we use wrk inside of Docker, we have to first share this file with the container. This can be achieved with Docker’s -v option.


## Parts of a Lua script for wrk


In generic form, using a script called test.lua, the whole command could look like this:


```
docker run --rm -v `pwd`/scripts:/scripts williamyeh/wrk -c1 -t1 -d5s -s /scripts/test.lua http://$APP1_PRIVATE_IP:3000


```


We explained the wrk command and its options in an earlier step. This command doesn’t add too much more; just the path to the script and some extra commands to tell Docker how to find it outside the container.


The --rm flag will automatically remove the container after it has stopped.


But do we actually know how to write a Lua script? Don’t fear; you will learn it with ease. We’ll go over a simple example here, and you can run your own more advanced scripts on your own.


First let’s talk about the predetermined script structure which reflects wrk’s internal logic. The diagram below illustrates it for one thread:





wrk performs the following phases of execution:


- Resolve the IP address of the domain
- Begin with the thread setup
- Perform the stress testing phase, which is called the running phase
- The last step is called simply done

When using several threads, you’ll have one resolution phase and one done phase, but two setup phases and two running phases:





Additionally, the running phase can be split into three steps: init, request, and response.





According to the presented diagrams and the documentation we can use the following methods inside a Lua script:


- 
setup(thread): Executed when all threads have been initialized but not yet started. Used to pass data to threads

- 
init(args): Called when each thread is initialized
This function receives extra command line arguments for the script which must be separated from wrk arguments with --.
Example:
  wrk -c3 -d1s -t2 -s /scripts/debug.lua http://$APP1_PRIVATE_IP:3000 -- debug true


- 
request(): Needs to return the HTTP object for each request. In this function we can modify the method, headers, path, and body
Use the wrk.format helper function to shape the request object.
Example:
  return wrk.format(method, path, headers, body)


- 
response(status, headers, body): Called when the response comes back

- 
done(summary, latency, requests): Executed when all requests are finished and statistics are computed
Inside this function the following properties are available:



Property
Description




summary.duration
run duration in microseconds


summary.requests
total completed requests


summary.bytes
total bytes received


summary.errors.connect
total socket connection errors


summary.errors.read
total socket read errors


summary.errors.write
total socket write errors


summary.errors.status
total HTTP status codes > 399


summary.errors.timeout
total request timeouts


latency.min
minimum latency value reached during test


latency.max
maximum latency value reached during test


latency.mean
average latency value reached during test


latency.stdev
latency standard deviation


latency:percentile(99.0)
99th percentile value


latency[i]
raw latency data of request i





Each thread has its own Lua context and in it its own local variables.


Now we’ll go through a few practical examples, but you can find many more useful benchmarking scripts in the wrk project’s scripts directory.


## Example: POST requests


Let’s start with the simplest example, where we simulate a POST request.


POST requests are usually used to send data to the server. This could be used to benchmark:


- 
HTML form-handlers: Use the address which is in the action attribute of your HTML form:
  <form action="/login.php">
  ...
  </form>


- 
POST API endpoints: If you have a restful API, use the endpoint where you create your article:
  POST /articles



Begin with creating a scripts/post.lua file on the wrk1 Droplet.


```
cd ~
mkdir scripts
nano scripts/post.lua


```


Add to it the following content:


post.lua
```
wrk.method = "POST"
wrk.body   = "login=sammy&password=test"
wrk.headers["Content-Type"] = "application/x-www-form-urlencoded"

```


This script is very simple and we didn’t even use any of the mentioned methods yet. We just modified the global wrk object properties.


We changed the request method to POST, added some login parameters, and specified the Content-Type header to the MIME type HTML forms use.


Before we start the benchmark, here is a diagram to help you visualize how the script, the Docker container, and the app server relate:





Now the moment of truth - benchmark the application with this command (execute on the wrk1 Droplet):


```
docker run --rm -v `pwd`/scripts:/scripts williamyeh/wrk -c1 -t1 -d5s -s /scripts/post.lua http://$APP1_PRIVATE_IP:3000


```


Output:


```
OutputRunning 5s test @ http://10.135.232.163:3000
  1 threads and 1 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.04ms  718.38us  12.28ms   90.99%
    Req/Sec     1.02k   271.31     1.52k    66.00%
  5058 requests in 5.00s, 0.97MB read
Requests/sec:   1011.50
Transfer/sec:    198.55KB

```


The output is similar to the one we saw earlier.


Notice that we are benchmarking here with only one connection. This corresponds to a situation where only one user would like to continously log in, passing the username and password. This is not requesting any CSS, image, or JavaScript files.


For a more realistic scenario you should increase the number of clients and threads, while simultaneously observing the latency parameter, to see how quickly the application can validate the user credentials.


## Example: Multiple URL Paths


Another common need is to test multiple paths of an application simultaneously.


Let’s create a file called paths.txt in a data directory and add all the paths which we want to use during our benchmark.


```
cd ~
mkdir data
nano data/paths.txt


```


Find an example of the data/paths.txt below:


paths.txt
```
/feed.xml
/contact/
/about/
/blog/
/2015/04/21/nginx-maintenance-mode/
/2015/01/06/vagrant-workflows/
/2014/12/10/top-vagrant-plugins/

```


Then grab this simple script and save it as scripts/multiple-url-paths.lua:


multiple-url-paths.lua
```
-- Load URL paths from the file
function load_url_paths_from_file(file)
  lines = {}

  -- Check if the file exists
  -- Resource: http://stackoverflow.com/a/4991602/325852
  local f=io.open(file,"r")
  if f~=nil then
    io.close(f)
  else
    -- Return the empty array
    return lines
  end

  -- If the file exists loop through all its lines
  -- and add them into the lines array
  for line in io.lines(file) do
    if not (line == '') then
      lines[#lines + 1] = line
    end
  end

  return lines
end

-- Load URL paths from file
paths = load_url_paths_from_file("/data/paths.txt")

print("multiplepaths: Found " .. #paths .. " paths")

-- Initialize the paths array iterator
counter = 0

request = function()
  -- Get the next paths array element
  url_path = paths[counter]

  counter = counter + 1

  -- If the counter is longer than the paths array length then reset it
  if counter > #paths then
    counter = 0
  end

  -- Return the request object with the current URL path
  return wrk.format(nil, url_path)
end

```


While this tutorial isn’t trying to teach Lua scripting in detail, if you read the comments in the script, you can get a good idea of what it does.


The multiple-url-paths.lua script opens the /data/paths.txt file and if this file contains paths, they are saved into a internal paths array. Then, with each request, the next path is taken.


To run this benchmark, use the following command (execute on the wrk1 Droplet). You’ll notice we’re adding some line breaks to make it easier to copy:


```
docker run --rm \
           -v `pwd`/scripts:/scripts \
           -v `pwd`/data:/data \
           williamyeh/wrk -c1 -t1 -d5s -s /scripts/multiple-url-paths.lua http://$APP1_PRIVATE_IP:3000


```


Output:


```
Outputmultiplepaths: Found 7 paths
multiplepaths: Found 7 paths
Running 5s test @ http://10.135.232.163:3000
  1 threads and 1 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.92ms  466.59us   4.85ms   86.25%
    Req/Sec     1.10k   204.08     1.45k    62.00%
  5458 requests in 5.00s, 1.05MB read
Requests/sec:   1091.11
Transfer/sec:    214.17KB

```


## Avanced Requests with JSON and YAML


Now you might think that other benchmarking tools can do these types of test as well. However, wrk also has the ability to process advanced HTTP requests using JSON or YAML formatting.


For example, you could load a JSON or YAML file which describes each request in detail.


The author has published an advanced example with a JSON request on the author’s tech blog.


You can benchmark any kind of HTTP request you can think of, with wrk and Lua.


# Conclusion


After reading this article, you should be able to use wrk to benchmark your applications. As a side note, you can also see the beauty of Docker and how it can greatly minimize setting up your application and testing environments.


Finally, you can go the extra mile with advanced HTTP requests using Lua scripts with wrk.


