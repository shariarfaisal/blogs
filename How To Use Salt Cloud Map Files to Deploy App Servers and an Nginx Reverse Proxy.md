# How To Use Salt Cloud Map Files to Deploy App Servers and an Nginx Reverse Proxy

```Configuration Management``` ```Node.js``` ```Scaling``` ```Nginx``` ```CentOS```

## Introduction


You have your app written, and now you need to deploy it. You could make a production environment and set your app up on a VM, but how do you scale it when it gets popular? How do you roll out new versions? What about load balancing? And, most importantly, how can you be certain the configuration is correct? We can automate all of this to save ourselves a lot of time.


In this tutorial, we’ll show you how to define your application in a Salt Cloud map file, including the use of custom Salt grains to assign roles to your servers and dynamically configure a reverse proxy.


At the end of this tutorial, you will have two basic app servers, an Nginx reverse proxy with a dynamically-built configuration, and the ability to scale your application in minutes.


# Prerequisites


To follow this tutorial, you will need:


- 
One 1 GB CentOS 7 Droplet. All commands in this tutorial will be exeucted as root, so you don’t need to create a sudo non-root user.

- 
An SSH key on the Droplet for the root user (i.e. a new keypair created on the Droplet). Add this SSH key in the DigitalOcean control panel so you can use it to log into other DigitalOcean Droplets from the master Droplet. You can use the  How To Use SSH Keys with DigitalOcean Droplets tutorial for instructions.
Make a note of the name you assign to the key in the DigitalOcean control panel. In this tutorial, we’re using the name salt-master-root-key. You should also make a note of the private key location; by default, it’s /root/.ssh/id_rsa.

- 
A personal access token, which you can create by following the instructions in this step of How To Use the DigitalOcean APIv2. Be sure to set the scope to read and write.


# Step 1 — Installing Salt and Salt Cloud


To start, you’ll need to have Salt Cloud installed and configured on your server. For this tutorial, we’ll just use the Salt bootstrap script.


First, etch the Salt bootstrap script to install Salt.


```
wget -O install_salt.sh https://bootstrap.saltstack.com


```


Run the Salt bootstrap script. We use the -M flag to also install salt-master.


```
sh install_salt.sh -M


```


While the Salt Cloud codebase has been merged into the core Salt project, it’s still packaged separately for CentOS. Fortunately, the install_salt script configured the repos for us, so we can just install salt-cloud with yum:


```
yum install salt-cloud


```


Now we can check Salt Cloud’s version to confirm a successful installation:


```
salt-cloud --version


```


You should see output like this:


salt-cloud --version output
```
salt-cloud 2015.5.3 (Lithium)

```


Note that Salt is on a rolling release cycle, so your version may differ slightly from above.


# Step 2 — Configuring Salt Cloud


In this section, we’ll configure Salt Cloud to connect to DigitalOcean and define some profiles for our Droplets.


## Configuring the DigitalOcean Provider File


In Salt Cloud, provider files are where you define where the new VMs will be created. Providers are defined in the /etc/salt/cloud.providers.d directory


Create and open a DigitalOcean provider file using nano or your favorite text editor.


```
nano /etc/salt/cloud.providers.d/digital_ocean.conf


```


Insert the below text, replacing the variables in red with yours — namely the server IP and access token, and also the SSH key name and file if you customized them.


/etc/salt/cloud.providers.d/digital_ocean.conf
```
### /etc/salt/cloud.providers.d/digital_ocean.conf ###
######################################################
do:
  provider: digital_ocean
  minion:                      
    master: your_server_ip
                                
  # DigitalOcean Access Token
  personal_access_token: your_access_token
  
  # This is the name of your SSH key in your Digital Ocean account
  # as it appears in the control panel.          
  ssh_key_name: salt-master-root-key 
  
  # This is the path on disk to the private key for your Digital Ocean account                                                                    
  ssh_key_file: /root/.ssh/id_rsa

```


You need to lock down the permissions on your SSH key file. Otherwise, SSH will refuse to use it. Assuming yours is at the default location, /root/.ssh/id_rsa, you can do this with:


```
chmod 600 /root/.ssh/id_rsa


```


## Configuring the Profiles for Deployable Servers


In Salt Cloud, profiles are individual VM descriptions that are tied to a provider (e.g. “A 512 MB Ubuntu VM on DigitalOcean”). These are defined in the /etc/salt/cloud.profiles.d directory.


Create and open a profile file.


```
nano /etc/salt/cloud.profiles.d/digital_ocean.conf


```


Paste the following into the file. No modification is necessary:


/etc/salt/cloud.profiles.d/digital_ocean.conf
```
### /etc/salt/cloud.profiles.d/digital_ocean.conf ###
#####################################################

ubuntu_512MB_ny3:
  provider: do
  image: ubuntu-14-04-x64
  size: 512MB
  location: nyc3
  private_networking: True

ubuntu_1GB_ny3:
  provider: do
  image: ubuntu-14-04-x64
  size: 1GB
  location: nyc3
  private_networking: True

```


Save and close the file. This file defines two profiles:


- A Ubuntu 14.04 VM with 512 MB of memory, deployed in the New York 3 region.
- A Ubuntu 14.04 VM with 1 GB of memory, deployed in the New York 3 region.

If you want to use an image other than Ubuntu 14.04, you can use Salt Cloud to list all of the available image names on DigitalOcean with the following command:


```
salt-cloud --list-images do


```


This will show all of the standard DigitalOcean images, as well as custom images that you have saved on your account with the snapshot tool. You can replace the image name or region that we used in the provider file with a different image name from this list. If you do, make sure to use the slug field from this output in the image setting in the profile file.


Test your configuration with a quick query.


```
salt-cloud -Q


```


You should see something like the following.


Example salt-cloud -Q output
```
[INFO    ] salt-cloud starting
do:
    ----------
    digital_ocean:
        ----------
        centos-salt:
            ----------
            id:
                2806501
            image_id:
                6372108
            public_ips:
                192.241.247.229
            size_id:
                63
            state:
                active

```


This means Salt Cloud is talking to your DigitalOcean account, and you have two basic profiles configured.


# Step Three — Writing a Simple Map File


A map file is a YAML file that lists the profiles and number of servers you want to create. We’ll start with a simple map file, then build on it in the next section.


Using the above profiles, let’s say you want two 1 GB app servers fronted by a single 512 MB reverse proxy. We’ll make a mapfile named /etc/salt/cloud.maps.d/do-app-with-rproxy.map and define the app.


First, create the file:


```
nano /etc/salt/cloud.maps.d/do-app-with-rproxy.map


```


Insert the following text. No modification is necessary:


/etc/salt/cloud.maps.d/do-app-with-rproxy.map
```
### /etc/salt/cloud.maps.d/do-app-with-rproxy.map ####
######################################################
ubuntu_512MB_ny3:
  - nginx-rproxy
  
ubuntu_1GB_ny3:
  - appserver-01
  - appserver-02

```


That’s it! That’s about as simple as a Map File gets. Go ahead and deploy those servers with:


```
salt-cloud -m /etc/salt/cloud.maps.d/do-app-with-rproxy.map


```


Once the command finishes, confirm success with a quick ping:


```
salt '*' test.ping


```


You should see the following:


```
[label salt '*' test.ping
appserver-01:
    True
appserver-02:
    True
nginx-rproxy:
    True

```


Once you’ve successfully created the VMs in your map file, deleting them is just as easy:


```
salt-cloud -d -m /etc/salt/cloud.maps.d/do-app-with-rproxy.map


```


Be sure to use that command with caution, though! It will delete all the VMs specified in that map file.


# Step Four — Writing A More Realistic Map File


That map file worked fine, but even a shell script could spin up a set of VMs. What we need is to define the footprint of our application. Let’s go back to our map file and add a few more things; open the map file again.


```
nano /etc/salt/cloud.maps.d/do-app-with-rproxy.map


```


Delete the previous contents of the file and place the following into it. No modification is needed:


/etc/salt/cloud.maps.d/do-app-with-rproxy.map
```
### /etc/salt/cloud.maps.d/do-app-with-rproxy.map ###
#####################################################
ubuntu_512MB_ny3:
  - nginx-rproxy:
      minion:
        mine_functions:
          network.ip_addrs:
            interface: eth0
        grains:
          roles: rproxy
ubuntu_1GB_ny3:
  - appserver-01:
      minion:
        mine_functions:
          network.ip_addrs:
            interface: eth0
        grains:
          roles: appserver
  - appserver-02:
      minion:
        mine_functions:
          network.ip_addrs:
            interface: eth0
        grains:
          roles: appserver

```


Now we’re getting somewhere! It looks like a lot but we’ve only added two things. Let’s go over the two additions: the mine_functions section and the grains section.


We’ve told Salt Cloud to modify the Salt Minion config for these VMs and add some custom grains. Specifically, the grains give the reverse proxy VM the rproxy role and give the app servers the appserver role. This will come in handy when we need to dynamically configure the reverse proxy.


The mine_functions will also be added to the Salt Minion config. It instructs the Minion to send the IP address found on eth0 back to the Salt Master to be stored in the Salt mine. This means the Salt Master will automatically know the IP of the newly-created Droplet without us having to configure it. We’ll be using this in the next part.


# Step Five — Defining the Reverse Proxy


We have a common task in front of us now: install the reverse proxy web server and configure it. For this tutorial, we’ll be using Nginx as the reverse proxy.


## Writing the Nginx Salt State


It’s time to get our hands dirty and write a few Salt states. First, make the default Salt state tree location:


```
mkdir /srv/salt


```


Navigate into that directory and make one more directory just for nginx:


```
cd /srv/salt
mkdir /srv/salt/nginx


```


Go into that directory and, using your favorite editor, create a new file called rproxy.sls:


```
cd /srv/salt/nginx
nano /srv/salt/nginx/rproxy.sls


```


Place the following into that file. No modification is needed:


/srv/salt/nginx/rproxy.sls
```
### /srv/salt/nginx/rproxy.sls ###
##################################

### Install Nginx and configure it as a reverse proxy, pulling the IPs of
### the app servers from the Salt Mine.

nginx-rproxy:
  # Install Nginx
  pkg:
    - installed
    - name: nginx    
  # Place a customized Nginx config file
  file:
    - managed
    - source: salt://nginx/files/awesome-app.conf.jin
    - name: /etc/nginx/conf.d/awesome-app.conf
    - template: jinja
    - require:
      - pkg: nginx-rproxy
  # Ensure Nginx is always running.
  # Restart Nginx if the config file changes.
  service:
    - running
    - enable: True
    - name: nginx
    - require:
      - pkg: nginx-rproxy
    - watch:
      - file: nginx-rproxy
  # Restart Nginx for the initial installation.
  cmd:
    - run
    - name: service nginx restart
    - require:
      - file: nginx-rproxy

```


This state does the following:


- Installs Nginx.
- Places our custom config file into /etc/nginx/conf.d/awesome-app.conf.
- Ensures Nginx is running.

Our Salt state simply installs Nginx and drops a config file; the really interesting content is in the config.


## Writing the Nginx Reverse Proxy Config File


Let’s make one more directory for our config file:


```
mkdir /srv/salt/nginx/files
cd /srv/salt/nginx/files


```


And open the config file:


```
nano /srv/salt/nginx/files/awesome-app.conf.jin


```


Put the following in the config file. No modification is necessary, unless you are not using private networking; in that case, change the 1 to 0 as noted in-line:


/srv/salt/nginx/files/awesome-app.conf.jin
```
### /srv/salt/nginx/files/awesome-app.conf.jin ###
##################################################

### Configuration file for Nginx to act as a 
### reverse proxy for an app farm.

# Define the app servers that we're in front of.
upstream awesome-app {
    {% for server, addrs in salt['mine.get']('roles:appserver', 'network.ip_addrs', expr_form='grain').items() %}
    server {{ addrs[0] }}:1337;
    {% endfor %}
}

# Forward all port 80 http traffic to our app farm, defined above as 'awesome-app'.
server {
    listen       80;
    server_name  {{ salt['network.ip_addrs']()[1] }};  # <-- change the '1' to '0' if you're not using 
                                                       #     DigitalOcean's private networking.

    access_log  /var/log/nginx/awesome-app.access.log;
    error_log  /var/log/nginx/awesome-app.error.log;

    ## forward request to awesome-app ##
    location / {
     proxy_pass  http://awesome-app;
     proxy_set_header        Host            $host;
     proxy_set_header        X-Real-IP       $remote_addr;
     proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
   }
}

```


We use the .jin extension to tell ourselves that the file contains Jinja templating. Jinja templating allows us to put a small amount of logic into our text files so we can dynamically generate config details.


This config file instructs Nginx to take all port 80 HTTP traffic and forward it on to our app farm. It has two parts: an upstream (our app farm) and the configuration to act as a proxy between the user and our app farm.


Let’s talk about the upstream. A normal, non-templated upstream specifies a collection of IPs. However, we don’t know what the IP addresses of our minions will be until they exist, and we don’t edit config files manually. (Otherwise, there’d be no reason to use Salt!)


Remember the mine_function lines in our map file? The minions are giving their IPs to the Salt Master to store them for just such an occassion. Let’s look at that Jinja line a little closer:


Jinja excerpt
```
{% for server, addrs in salt['mine.get']('roles:appserver', 'network.ip_addrs', expr_form='grain').items() %}

```


This is a for-loop in Jinja, running an arbitrary Salt function. In this case, it’s running mine.get. The parameters are:


- roles:appserver - This says to only get the details from the minions who have the “appserver” role.
- network.ip_addrs - This is the data we want to get out of the mine. We specified this in our map file as well.
- expr_form='grain' - This tells Salt that we’re targeting our minions based on their grains. More on matching by grain at the Saltstack targeting doc.

Following this loop, the variable {{addrs}} contains a list of IP addresses (even if it’s only one address). Because it’s a list, we have to grab the first element with [0].


That’s the upstream. As for the server name:


```
server_name  {{ salt['network.ip_addrs']()[0] }};

```


This is the same trick as the Salt mine call (call a Salt function in Jinja). It’s just simpler. It’s calling network.ip_addrs and taking the first element of the returned list. This also lets us avoid having to manually edit our file.


# Step Six — Defining the App Farm


A reverse proxy doesn’t mean much if it doesn’t have an app behind it. Let’s make a small Node.js application that just reports the IP of the server it’s on (so we can confirm we’re reaching both machines).


Make a new directory called awesome-app and move there.


```
mkdir -p /srv/salt/awesome-app
cd /srv/salt/awesome-app


```


Create a new app state file called app.sls.


```
nano /srv/salt/awesome-app/app.sls


```


Place the following into the file. No modification is necessary:


/srv/salt/awesome-app/app.sls
```
### /srv/salt/awesome-app/app.sls ###
#####################################

### Install Nodejs and start a simple
### web application that reports the server IP.

install-app:
  # Install prerequisites
  pkg:
    - installed
    - names: 
      - node
      - npm
      - nodejs-legacy  # workaround for Debian systems
  # Place our Node code
  file: 
    - managed
    - source: salt://awesome-app/files/app.js
    - name: /root/app.js
  # Install the package called 'forever'
  cmd:
    - run
    - name: npm install forever -g
    - require:
      - pkg: install-app
   
run-app:
  # Use 'forever' to start the server
  cmd:
    - run
    - name: forever start app.js
    - cwd: /root

```


This state file does the following:


- Installs the nodejs, npm, and nodejs-legacy packages.
- Adds the JavaScript file that will be our simple app.
- Uses NPM to install Forever.
- Runs the app.

Now create the (small) app code:


```
mkdir /srv/salt/awesome-app/files
cd /srv/salt/awesome-app/files


```


Create the file:


```
nano /srv/salt/awesome-app/files/app.js


```


Place the following into it. No modification is needed:


/srv/salt/awesome-app/files/app.js
```
/* /srv/salt/awesome-app/files/app.js

   A simple Node.js web application that
   reports the server's IP.
   Shamefully stolen from StackOverflow:
   http://stackoverflow.com/questions/10750303/how-can-i-get-the-local-ip-address-in-node-js
*/

var os = require('os');
var http = require('http');

http.createServer(function (req, res) {
  var interfaces = os.networkInterfaces();
  var addresses = [];
  for (k in interfaces) {
      for (k2 in interfaces[k]) {
          var address = interfaces[k][k2];
          if (address.family == 'IPv4' && !address.internal) {
              addresses.push(address.address)
          }
      }
  }

  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end(JSON.stringify(addresses));
}).listen(1337, '0.0.0.0');
console.log('Server listening on port 1337');


```


This is a simple Node.js server that does one thing: accepts HTTP requests on port 1337 and respond with the server’s IPs.


At this point, you should have a file structure that looks like the following:


File structure
```
/srv/salt
         ├── awesome-app
         │   ├── app.sls
         │   └── files
         │       └── app.js
         └── nginx
             ├── rproxy.sls
             └── files
                 └── awesome-app.conf.jin

```


# Step Seven — Deploying the Application


All that’s left is to deploy the application.


## Deploy the Servers With Salt Cloud


Run the same deployment command from earlier, which will now use all the configurations we’ve been making.


```
salt-cloud -m /etc/salt/cloud.maps.d/do-app-with-rproxy.map


```


Wait for Salt Cloud to complete; this will take a while. Once it returns, confirm successful deployment by pinging the app servers:


```
salt -G 'roles:appserver' test.ping


```


You should see:


App server ping output
```
appserver-02:
    True
appserver-01:
    True

```


Ping the reverse proxy:


```
salt -G 'roles:rproxy' test.ping


```


You should see:


Reverse proxy ping output
```
nginx-rproxy:
    True

```


Once you have your VMs, it’s time to give them work.


## Build the Application


Next, issue the Salt commands to automatically build the app farm and the reverse proxy.


Build the app farm:


```
salt -G 'roles:appserver' state.sls awesome-app.app


```


There will be a fair amount of output, but it should end with the following:


```
Summary
------------
Succeeded: 6 (changed=6)
Failed:    0
------------
Total states run:     6


```


Build the reverse proxy:


```
salt -G 'roles:rproxy' state.sls nginx.rproxy


```


Again, there will be a fair amount of output, ending with the following:


```
Summary
------------
Succeeded: 4 (changed=4)
Failed:    0
------------
Total states run:     4


```


So what just happened here?


The first command (the one with the app servers) took the Salt state that we wrote earlier and executed it on the two app servers. This resulted in two machines with identical configurations running identical versions of code.


The second command (the reverse proxy) executed the Salt state we wrote for Nginx. It installed Nginx and  the configuration file, dynamically filling in the IPs of our app farm in the config file.


Once those Salt runs complete, you can test to confirm successful deployment. Find the IP of your reverse proxy:


```
salt -G 'roles:rproxy' network.ip_addrs


```


You may get back two IPs if you’re using private networking on your Droplet.


Plug the public IP into your browser and visit the web page! Hit refresh a few times to confirm that Nginx is actually proxying among the two app servers you built. You should see the IPs changing, confirming that you are, indeed, connecting to more than one app server.


If you get the same IP despite refreshing, it’s likely due to browser caching. You can try using curl instead to show that Nginx is proxying among your app servers. Run this command several times and observe the output:


```
curl http://ip-of-nginx-rproxy


```


We can take this a few steps further and completely automate the application deployment via OverState. This would let us build a single command to tell Salt to build, say, the app servers first before moving on to build the reverse proxy, guaranteeing the order of our build process.


# Step Eight — Scaling Up (Optional)


The point of using Salt is to automate your build process; the point of using Salt Cloud and map files is to easily scale your deployment. If you wanted to add more app servers (say, two more) to your deployment, you would update your map file to look like this:


/etc/salt/cloud.maps.d/do-app-with-rproxy.map
```
### /etc/salt/cloud.maps.d/do-app-with-rproxy.map ###
#####################################################
ubuntu_512MB_ny3:
  - nginx-rproxy:
      minion:
        mine_functions:
          network.ip_addrs:
            interface: eth0
        grains:
          roles: rproxy
ubuntu_1GB_ny3:
- appserver-01:
    minion:
      mine_functions:
        network.ip_addrs:
            interface: eth0
      grains:
        roles: appserver
- appserver-02:
    minion:
      mine_functions:
        network.ip_addrs:
            interface: eth0
      grains:
        roles: appserver
- appserver-03:
    minion:
      mine_functions:
        network.ip_addrs:
            interface: eth0
      grains:
        roles: appserver
- appserver-04:
    minion:
      mine_functions:
        network.ip_addrs:
            interface: eth0
      grains:
        roles: appserver        

```


After making that update, you would re-run the salt-cloud command and the two salt commands from Step 6:


```
salt-cloud -m /etc/salt/cloud.maps.d/do-app-with-rproxy.map
salt -G 'roles:appserver' state.sls awesome-app.app
salt -G 'roles:rproxy' state.sls nginx.rproxy


```


The existing servers wouldn’t be impacted by the repeat Salt run, the new servers would be built to spec, and the Nginx config would update to begin routing traffic to the new app servers.


# Conclusion


Deploying an app that just reports the server’s IP isn’t very useful. Fortunately, this approach is not limited to Node.js applications. Salt doesn’t care what language your app is written in.


If you wanted to take this framework to deploy your own app, you would just need to automate the task of installing your app on a server (either via a script or with Salt states) and replace our awesome-app example with your own automation.


Just as Salt automates processes on your Droplets, Salt Cloud automates processes on your cloud. Enjoy!


