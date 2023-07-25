# How To Use the Ambassador Pattern to Dynamically Configure Services on CoreOS

```Apache``` ```Docker``` ```Configuration Management``` ```CoreOS```

## Introduction


The Docker Links feature enables a method of dynamically configuring network connections between containers, known as the ambassador pattern. The ambassador pattern promotes service portability between provider and consumer containers. In CoreOS, etcd can be leveraged to implement the ambassador pattern distributed across multiple machines in a cluster.


In this tutorial, we will demonstrate deploying an Apache HTTP container that is registered with etcd. The Apache container will represent our provider container, and we will use HAProxy as our consumer container. We will be using Docker images from this CoreOS ambassador demo for our ambassador containers, and we will create our own Apache and HAProxy Docker images from scratch.


# Prerequisites


You must have a CoreOS cluster on DigitalOcean that consists of at least three machines. Here is a tutorial on how to set that up: How To Create and Run a Service on a CoreOS Cluster


You must have basic knowledge of using CoreOS, etcdctl, fleetctl, setting up services, and running Docker containers. These topics are covered in the Getting Started with CoreOS tutorial series.


You must have a Docker Hub account or a private Docker registry. This is covered in the Creating the Docker Container section of the How To Create and Run a Service on a CoreOS Cluster tutorial.


For full details about how the ambassador pattern works, check out the Link via an Ambassador Container article from Docker. Also, check out this article posted on the CoreOS blog: Dynamic Docker links with an ambassador powered by etcd.


# Our Goal


At the end of this tutorial we will have six containers running on two machines. This section will provide a brief description of each, and how they will be grouped together. This exact setup is not useful for most people, but it can be adapted to allow dynamic service discovery for your own services.


## Machine A


Machine A will run the provider container, i.e. Apache web server, and a couple other containers that will support it.


- Apache Web Server: A basic Apache container that we will create from scratch, similar to the one described in the How To Create and Run a Service on a CoreOS Cluster tutorial. This is our producer
- polvi/docker-register: A registration container that will read the IP address and port of Apache via the Docker API and write it to etcd
- polvi/simple-amb: A simple ambassador container that will forward traffic to a specified location. In this case, we will forward traffic to etcd and link it to the docker-register container to provide that container access to etcd. In CoreOS, because the location of etcd is static, this could be removed if docker-register was modified to access etcd directly

## Machine B


Machine B is a CoreOS machine that will run the consumer container, i.e. HAProxy, and the main ambassador container.


- HAProxy Reverse Proxy: A basic HAProxy container, which we will create from scratch, which will represent our consumer. This will be used to demonstrate that the ambassador setup works
- polvi/dynamic-etcd-amb: The main ambassador container. A dynamic proxy that watches a specified etcd key for the IP address and port of provider container and routes all traffic to the provider container. The value of the key can be updated, and the proxy will update itself
- polvi/simple-amb: The same container used on the other machine, but used to link dynamic-etcd-amb to etcd

# Create Apache Docker Image


SSH to one of your CoreOS machines, and pass your SSH agent (substitute in the public IP address):


```
ssh -A core@coreos-1_public_IP

```


Then log in to Docker:


```
docker login

```


Enter your user_name, password, and email address when prompted.


Next, create a new directory to write your Apache Dockerfile to:


```
mkdir -p ambassador/apache

```


Now change to the directory and open Dockerfile for editing:


```
cd ambassador/apache
vi Dockerfile

```


Based on the Apache container setup from How To Create and Run a Service on a CoreOS Cluster, we can create the following Dockerfile (substitute user_name with your own Docker username):


```
FROM ubuntu:14.04
MAINTAINER user_name

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y install apache2 && \
    echo "<h1>Running from Docker on CoreOS</h1>" > /var/www/html/index.html

EXPOSE 80

ENTRYPOINT ["/usr/sbin/apache2ctl"]
CMD ["-D", "FOREGROUND"]

```


Save and quit.


Now that we have a Dockerfile that installs Apache and replaces index.html with a basic message, build your Docker image and name it “apache” with the following command (substitute your own username):


```
docker build --tag="user_name/apache" .

```


Now, to make the image available to your other CoreOS machines, push it to your Docker registry with the following command:


```
docker push user_name/apache

```


Now your Apache image is ready for use. Let’s move on to creating an HAProxy image.


# Create HAProxy Docker Image


We will create an HAProxy Docker image, based on the HAproxy Dockerfile for trusted automated Docker builds. We will slightly modify the provided haproxy.cfg and start.bash files.


In the ambassador directory, use git to clone the HAProxy repository:


```
cd ~/ambassador
git clone https://github.com/dockerfile/haproxy.git

```


This will create an haproxy directory, with Dockerfile, haproxy.cfg, and start.bash files.


The Dockerfile basically installs HAProxy and exposes ports 80 and 443, so we can leave it as is.


We will modify the haproxy.cfg file to add a frontend and backend. Open haproxy.cfg for editing:


```
cd haproxy
vi haproxy.cfg

```


Now find and delete the following lines:


```
listen stats :80
  stats enable
  stats uri /

```


Then add the following lines at the end of the file:


```
frontend www-http
        bind :80
        default_backend www-backend

backend www-backend
        server apache private_ipv4:80 check

```


This configures HAProxy to listen on port 80 and forward incoming traffic to www-backend, which consists of a single server. We will use the start.bash script to substitute private_ipv4 with the private IP address of the CoreOS machine that this container will run on, when the HAProxy container starts. Our dynamic ambassador container, which HAProxy will forward traffic through to the Apache container, will run on the same machine.


Open the start.bash file for editing:


```
vi start.bash

```


At the bottom of the file, you will find a line that will start the HAProxy process in this container. It looks like this:


```
haproxy -f /etc/haproxy/haproxy.cfg -p "$PIDFILE"

```


Directly above this line, insert the following lines:


```
# Set backend IP address to machine's private IP address
PRIVATE_IPV4=$(curl -sw "\n" http://169.254.169.254/metadata/v1/interfaces/private/0/ipv4/address)
sed -i -e "s/server apache private_ipv4:80 check/server apache ${PRIVATE_IPV4}:80 check/g" $HAPROXY/$CONFIG

```


Save and exit. The curl command will retrieve the private IP address of the machine that the container will run on via the DigitalOcean Metadata service. The sed command replaces the private_ipv4 string in haproxy.cfg with the actual IP address that was retrieved from Metadata. This script runs from inside the HAProxy container, so the private IP address will be configured at runtime.


Now we are ready to build the HAProxy docker image. Build your Docker image and name it “haproxy” with the following command (substitute your own username):


```
docker build --tag="user_name/haproxy" .

```


Now, to make the image available to your other CoreOS machines, push it to your Docker registry with the following command:


```
docker push user_name/haproxy

```


Your HAProxy image is ready for use. We are ready to write our fleet service unit files!


# Fleet Service Unit Files


Now that all of the required Docker images are available to our CoreOS cluster, let’s start working on the files that are required for deploying our containers. Because we are using a CoreOS cluster, we can create and schedule all of our fleet service unit files from a single CoreOS machine.


We will create all of the service files in the ~/ambassador directory that we created earlier, so change to that directory now:


```
cd ~/ambassador

```


## apache.service


The apache.service unit will run on Host A.


The first service file we will create is for the Apache web server container, user_name/apache. Open a file called apache.service for editing now:


```
vi apache.service

```


Add the following lines (substitute your Docker username in both places):


```
[Unit]
Description=Apache web server service

[Service]
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker kill %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull user_name/apache
ExecStart=/usr/bin/docker run --rm --name %n -p ${COREOS_PRIVATE_IPV4}::80 user_name/apache
ExecStop=/usr/bin/docker stop -t 3 %n

```


Save and exit. This is a fairly straightforward service file that starts Apache in foreground mode. Of particular note is that we are binding port 80 inside the container to a dynamic port on the private network interface (-p ${COREOS_PRIVATE_IPV4}::80).


## etcd-amb-apache.service


The etcd-amb-apache.service unit will run on Host A.


Next we will want to create a service file for our simple ambassador container (simple-amb) that will allow the Apache registration container to access etcd. Open a file called etcd-amb-apache.service now:


```
vi etcd-amb-apache.service

```


Add the following lines:


```
[Unit]
Description=Simple Apache ambassador
After=apache.service
BindsTo=apache.service

[Service]
ExecStartPre=-/usr/bin/docker kill %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStart=/usr/bin/docker run --rm --name %n polvi/simple-amb 172.17.42.1:4001
ExecStop=/usr/bin/docker stop -t 3 %n

[X-Fleet]
X-ConditionMachineOf=apache.service

```


Save and exit.


The simple-amb container forwards all traffic it receives on port 10000 to the argument provided when it is started, i.e. 172.17.42.1:4001, which is etcd’s standard location in CoreOS.


X-ConditionMachineOf=apache.service tells fleet to schedule this on the same machine as the Apache container, which is critical since it is used by the docker-register container to register the IP address and port that Apache is using to etcd.


## apache-docker-reg.service


The apache-docker-reg.service unit will run on Host A.


Let’s create the service file for our container that will register Apache’s IP address and port in etcd, docker-register. Open a file called apache-docker-reg.service now:


```
vi apache-docker-reg.service

```


Insert the following lines:


```
[Unit]
Description=Register Apache
After=etcd-amb-apache.service
BindsTo=etcd-amb-apache.service

[Service]
ExecStartPre=-/usr/bin/docker kill %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStart=/usr/bin/docker run --link etcd-amb-apache.service:etcd -v /var/run/docker.sock:/var/run/docker.sock --rm polvi/docker-register apache.service 80 apache-A

[X-Fleet]
X-ConditionMachineOf=etcd-amb-apache.service

```


Save and exit. Here is a breakdown of the notable parts of the docker run command:


- --link etcd-amb-apache.service:etcd links this container to the simple ambassador, which will be used to pass Apache’s connection information to etcd
- -v /var/run/docker.sock:/var/run/docker.sock allows this container to determine the dynamic port that Apache is binding to via the Docker API of the machine it will run on.
- apache.service 80 apache-A passes these arguments to the container. The first two arguments specify the name and port of the docker container to look up, and the third argument specifies the name of the etcd key to write to. After this container starts, it will write the dynamic port and IP address of apache.service into the /services/apache-A/apache.service key.

X-ConditionMachineOf=etcd-amb-apache.service tells fleet to schedule this on the same machine as the simple ambassador container which is critical since they are linked with a Docker link, to provide the registration container a way to find etcd.


## etcd-amb-apache2.service


The etcd-amb-apache2.service unit will run on Host B.


Create a service file for our second simple ambassador container (simple-amb) that will allow the dynamic ambassador container to access etcd. Open a file called etcd-amb-apache2.service now:


```
vi etcd-amb-apache2.service

```


Add the following lines:


```
[Unit]
Description=Simple Apache ambassador 2

[Service]
ExecStartPre=-/usr/bin/docker kill %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStart=/usr/bin/docker run --rm --name %n polvi/simple-amb 172.17.42.1:4001
ExecStop=/usr/bin/docker stop -t 3 %n

[X-Fleet]
X-Conflicts=apache.service

```


Save and exit.


This is pretty much the same service file as etcd-amb-apache.service except X-Conflicts=apache.service tells fleet to schedule it on a different machine than the Apache container, and it will be used to link the dynamic ambassador to etcd.


## apache-dyn-amb.service


The apache-dyn-amb.service unit will run on Host B.


Create a service file for our dynamic ambassador container (dynamic-etd-amb) that will allow the dynamic ambassador container to access etcd. Open a file called apache-dyn-amb.service now:


```
vi apache-dyn-amb.service

```


Add the following lines:


```
[Unit]
Description=Dynamic ambassador for Apache
After=etcd-amb-apache2.service
BindsTo=etcd-amb-apache2.service

[Service]
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker kill %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull polvi/dynamic-etcd-amb
ExecStart=/usr/bin/docker run --link etcd-amb-apache2.service:etcd --rm --name %n -p ${COREOS_PRIVATE_IPV4}:80:80 polvi/dynamic-etcd-amb apache-A 80
ExecStop=/usr/bin/docker stop -t 3 %n

[X-Fleet]
X-ConditionMachineOf=etcd-amb-apache2.service

```


Save and exit. Here is a breakdown of the notable parts of the docker run command:


- --link etcd-amb-apache2.service:etcd links this container to the second simple ambassador, which will be used to retrieve Apache’s connection information from etcd
- -p ${COREOS_PRIVATE_IPV4}:80:80 exposes port 80 on the container and the machine’s private network interface
- apache-A 80 are two arguments that specify that port 80 traffic (i.e. port 80 on the private network interface) should be proxied to the service registered as apache-A in etcd

X-ConditionMachineOf=etcd-amb-apache2.service tells fleet to schedule this on the same machine as the second simple ambassador container which is critical since they are linked with a Docker link, to provide the dynamic ambassador container a way to find etcd.


## haproxy.service


The haproxy.service unit will run on Host B.


Create a service file for our HAProxy container (haproxy) that will be used to connect to the Apache container, through the dynamic ambassador container. Open a file called haproxy.service now:


```
vi haproxy.service

```


Add the following lines (substitute your Docker username in both places):


```
[Unit]
Description=HAProxy consumer

[Service]
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker kill %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull user_name/haproxy
ExecStart=/usr/bin/docker run --name %n -p ${COREOS_PUBLIC_IPV4}:80:80 user_name/haproxy
ExecStop=/usr/bin/docker stop -t 3 %n

[X-Fleet]
X-ConditionMachineOf=apache-dyn-amb.service

```


Save and exit. This is a straightforward service file, that starts HAProxy and exposes port 80 on its host machine’s public IP address. Remember that the backend server will be configured to the private IP address of the host machine on port 80, which happens to be where the dynamic ambassador is listening for traffic to proxy to the Apache service.


X-ConditionMachineOf=apache-dyn-amb.service tells fleet to schedule this on the same machine as the dynamic ambassador container which is important because the dynamic ambassador provides the HAProxy container with a route to get to the Apache container.


# Deploy With Fleet


Now that we have all of the necessary fleet service files, we can finally deploy our ambassador setup. In the directory that contains all of your service files, run the following commands:


```
fleetctl start apache.service
fleetctl start etcd-amb-apache.service
fleetctl start apache-docker-reg.service
fleetctl start etcd-amb-apache2.service
fleetctl start apache-dyn-amb.service
fleetctl start haproxy.service

```


You should see messages saying that each service is loaded. To check the status of your fleet units, run the following command:


```
fleetctl list-units

```


You should see output that is similar to the following:


```
UNIT                       MACHINE                      ACTIVE   SUB
apache-docker-reg.service  ceb3ead2.../10.132.233.107   active   running
apache-dyn-amb.service     3ce87ca7.../10.132.233.106   active   running
apache.service             ceb3ead2.../10.132.233.107   active   running
etcd-amb-apache.service	   ceb3ead2.../10.132.233.107   active   running
etcd-amb-apache2.service   3ce87ca7.../10.132.233.106   active   running
haproxy.service	           3ce87ca7.../10.132.233.106   active   running

```


All of the statuses should be active and running. Another thing to note is that the “Machine A” units should be on the same machine and the “Machine B” units should be on a different machine–just look at the IP addresses of each unit to confirm this.


# Testing Your Setup


## Ensure HAProxy Can Reach Apache


Because we did not specify that the HAProxy container should run on a specific machine, we need to find where it is running. An easy way to do this is using the fleetctl ssh command:


```
fleetctl ssh haproxy.service

```


This will connect you to the machine that is running the haproxy.service container. Now you can source the /etc/environment file to get the public IP address of the CoreOS machine that is running HAProxy:


```
. /etc/environment
echo $COREOS_PUBLIC_IPV4

```


Take the resulting IP address and go to it using a web browser. You will see the following image:





Note that you are accessing HAProxy, and HAProxy is accessing Apache through the dynamic ambassador proxy.


Now you can exit your current SSH session to go back to your original SSH session:


```
exit

```


## Test Failover


Now that you have confirmed the ambassador setup works, let’s see what happens when the provider service (apache.service) changes its IP address and port.


Use fleetctl to connect to the machine that is running apache.service:


```
fleetctl ssh apache.service

```


Now reboot the machine that Apache is running on:


```
sudo reboot

```


Note: If apache.service was running on the machine that you originally connected to via SSH, you will be disconnected. If this is the case, simply SSH to another one of your machines in the same CoreOS cluster.


Now wait a minute and check which units are running:


```
fleetctl list-units

```


Depending on how long you waited, you may see that the three units related to “Host A” (apache.service, etcd-amb-apache.service, and apache-docker-reg.service) are restarting or active. Eventually, they should all go back to the active state. Once they do, note that they are now running on a different machine than before.


Now go back to your web browser that was connecting to HAProxy, and hit refresh. You should see the same test page as before, indicating that HAProxy is still able to connect to Apache via the dynamic ambassador!


# Conclusion


Now that you have set up your own ambassador pattern, you should be able to adapt the concepts presented in this tutorial to your own services. It is a unique way to configure your consumer services at runtime, which allows you to move your backend provider services amongst machines easily. In a more realistic setup, you would probably replace the Apache service with one or more application containers, and you might configure HAProxy with multiple backend servers (or use an entirely different consumer service).


