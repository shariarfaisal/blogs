# How To Set Up a Private Docker Registry on Ubuntu 20 04

```Docker``` ```Ubuntu 20.04``` ```Nginx```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Docker Registry is an application that manages storing and delivering Docker container images. Registries centralize container images and reduce build times for developers. Docker images guarantee the same runtime environment through virtualization, but building an image can involve a significant time investment. For example, rather than installing dependencies and packages separately to use Docker, developers can download a compressed image from a registry that contains all of the necessary components. Furthermore, developers can automate pushing images to a registry using continuous integration tools, such as TravisCI, to seamlessly update images during production and development.


Docker also has a free public registry, Docker Hub, that can host your custom Docker images, but there are situations where you will not want your image to be publicly available. Images typically contain all the code necessary to run an application, so using a private registry is preferable when using proprietary software.


In this tutorial, you will set up and secure your own private Docker Registry. You will use Docker Compose to define configurations to run your Docker containers and Nginx to forward server traffic from the internet to the running Docker container. Once you’ve completed this tutorial, you will be able to push a custom Docker image to your private registry and pull the image securely from a remote server.


# Prerequisites


- Two Ubuntu 20.04 servers set up by following the Ubuntu 20.04 Initial Server Setup Guide, including a sudo non-root user and a firewall. One server will host your private Docker Registry and the other will be your client server.
- Docker installed on both servers by following Step 1 and 2 of How To Install and Use Docker on Ubuntu 20.04.
- Docker Compose installed on the host server by following Step 1 of How To Install and Use Docker Compose on Ubuntu 20.04.
- Nginx installed on your host server by following the steps  in How To Install Nginx on Ubuntu 20.04.
- Nginx secured with Let’s Encrypt on your server for the private Docker Registry, by following the How To Secure Nginx with Let’s Encrypt on Ubuntu 20.04 tutorial. Make sure to redirect all traffic from HTTP to HTTPS in Step 4.
- A domain name that resolves to the server you’re using for the private Docker Registry. You will set this up as part of the Let’s Encrypt prerequisite. In this tutorial, we’ll refer to it as your_domain.

# Step 1 — Installing and Configuring the Docker Registry


Docker on the command line is useful when starting out and testing containers, but proves to be unwieldy for bigger deployments involving multiple containers running in parallel.


With Docker Compose, you can write one .yml file to set up each container’s configuration and information the containers need to communicate with each other. You can use the docker-compose command-line tool to issue commands to all the components that make up your application, and control them as a group.


Docker Registry is itself an application with multiple components, so you will use Docker Compose to manage it. To start an instance of the registry, you’ll set up a docker-compose.yml file to define it and the location on disk where your registry will be storing its data.


You’ll store the configuration in a directory called docker-registry on the main server. Create it by running:


```
mkdir ~/docker-registry


```


Navigate to it:


```
cd ~/docker-registry


```


Then, create a subdirectory called data, where your registry will store its images:


```
mkdir data


```


Create and open a file called docker-compose.yml by running:


```
nano docker-compose.yml


```


Add the following lines, which define a basic instance of a Docker Registry:


~/docker-registry/docker-compose.yml
```
version: '3'

services:
  registry:
    image: registry:2
    ports:
    - "5000:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - ./data:/data

```


First, you name the first service registry, and set its image to registry, version 2. Then, under ports, you map the port 5000 on the host to the port 5000 of the container. This allows you to send a request to port 5000 on the server, and have the request forwarded to the registry.


In the environment section, you set the REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY variable to /data, specifying in which volume it should store its data. Then, in the volumes section, you map the /data directory on the host file system to /data in the container, which acts as a passthrough. The data will actually be stored on the host’s file system.


Save and close the file.


You can now start the configuration by running:


```
docker-compose up


```


The registry container and its dependencies will be downloaded and started.


```
OutputCreating network "docker-registry_default" with the default driver
Pulling registry (registry:2)...
2: Pulling from library/registry
e95f33c60a64: Pull complete
4d7f2300f040: Pull complete
35a7b7da3905: Pull complete
d656466e1fe8: Pull complete
b6cb731e4f93: Pull complete
Digest: sha256:da946ca03fca0aade04a73aa94b54ff0dc614216bdd1d47585f97b4c1bdaa0e2
Status: Downloaded newer image for registry:2
Creating docker-registry_registry_1 ... done
Attaching to docker-registry_registry_1
registry_1  | time="2021-03-18T12:32:59.587157744Z" level=warning msg="No HTTP secret provided - generated random secret. This may cause problems with uploads if multiple registries are behind a load-balancer. To provide a shared secret, fill in http.secret in the configuration file or set the REGISTRY_HTTP_SECRET environment variable." go.version=go1.11.2 instance.id=119fe50b-2bb6-4a8d-902d-dfa2db63fc2f service=registry version=v2.7.1
registry_1  | time="2021-03-18T12:32:59.587912733Z" level=info msg="redis not configured" go.version=go1.11.2 instance.id=119fe50b-2bb6-4a8d-902d-dfa2db63fc2f service=registry version=v2.7.1
registry_1  | time="2021-03-18T12:32:59.598496488Z" level=info msg="using inmemory blob descriptor cache" go.version=go1.11.2 instance.id=119fe50b-2bb6-4a8d-902d-dfa2db63fc2f service=registry version=v2.7.1
registry_1  | time="2021-03-18T12:32:59.601503005Z" level=info msg="listening on [::]:5000" go.version=go1.11.2 instance.id=119fe50b-2bb6-4a8d-902d-dfa2db63fc2f service=registry version=v2.7.1
...

```


You’ll address the No HTTP secret provided warning message later in this tutorial. Notice that the last line of the output shows it has successfully started listening on port 5000.


You can press CTRL+C to stop its execution.


In this step, you have created a Docker Compose configuration that starts a Docker Registry listening on port 5000. In the next steps, you’ll expose it at your domain and set up authentication.


# Step 2 — Setting Up Nginx Port Forwarding


As part of the prerequisites, you’ve enabled HTTPS at your domain. To expose your secured Docker Registry there, you’ll only need to configure Nginx to forward traffic from your domain to the registry container.


You have already set up the /etc/nginx/sites-available/your_domain file, containing your server configuration. Open it for editing by running:


```
sudo nano /etc/nginx/sites-available/your_domain


```


Find the existing location block:


/etc/nginx/sites-available/your_domain
```
...
location / {
  ...
}
...

```


You need to forward traffic to port 5000, where your registry will be listening for traffic. You also want to append headers to the request forwarded to the registry, which provides additional information from the server about the request itself. Replace the existing contents of the location block with the following lines:


/etc/nginx/sites-available/your_domain
```
...
location / {
    # Do not allow connections from docker 1.5 and earlier
    # docker pre-1.6.0 did not properly set the user agent on ping, catch "Go *" user agents
    if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
      return 404;
    }

    proxy_pass                          http://localhost:5000;
    proxy_set_header  Host              $http_host;   # required for docker client's sake
    proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
    proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_read_timeout                  900;
}
...

```


The if block checks the user agent of the request and verifies that the version of the Docker client is above 1.5, as well as that it’s not a Go application that’s trying to access. For more explanation on this, you can find the nginx header configuration in Docker’s registry Nginx guide.


Save and close the file when you’re done. Apply the changes by restarting Nginx:


```
sudo systemctl restart nginx


```


If you get an error, double-check the configuration you’ve added.


To confirm that Nginx is properly forwarding traffic to your registry container on port 5000,  run it:


```
docker-compose up


```


Then, in a browser window, navigate to your domain and access the v2 endpoint, like so:


```
https://your_domain/v2

```


You will see an empty JSON object:


```
{}

```


In your terminal, you’ll receive output similar to the following:


```
Outputregistry_1  | time="2018-11-07T17:57:42Z" level=info msg="response completed" go.version=go1.7.6 http.request.host=cornellappdev.com http.request.id=a8f5984e-15e3-4946-9c40-d71f8557652f http.request.method=GET http.request.remoteaddr=128.84.125.58 http.request.uri="/v2/" http.request.useragent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/604.4.7 (KHTML, like Gecko) Version/11.0.2 Safari/604.4.7" http.response.contenttype="application/json; charset=utf-8" http.response.duration=2.125995ms http.response.status=200 http.response.written=2 instance.id=3093e5ab-5715-42bc-808e-73f310848860 version=v2.6.2
registry_1  | 172.18.0.1 - - [07/Nov/2018:17:57:42 +0000] "GET /v2/ HTTP/1.0" 200 2 "" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/604.4.7 (KHTML, like Gecko) Version/11.0.2 Safari/604.4.7"

```


You can see from the last line that a GET request was made to /v2/, which is the endpoint you sent a request to, from your browser. The container received the request you made, from the port forwarding, and returned a response of {}. The code 200 in the last line of the output means that the container handled the request successfully.


Press CTRL+C to stop its execution.


Now that you have set up port forwarding, you’ll move on to improving the security of your registry.


# Step 3 — Setting Up Authentication


Nginx allows you to set up HTTP authentication for the sites it manages, which you can use to limit access to your Docker Registry. To achieve this, you’ll create an authentication file with htpasswd and add username and password combinations to it that will be accepted.


You can obtain the htpasswd utility by installing the apache2-utils package. Do so by running:


```
sudo apt install apache2-utils -y


```


You’ll store the authentication file with credentials under ~/docker-registry/auth. Create it by running:


```
mkdir ~/docker-registry/auth


```


Navigate to it:


```
cd ~/docker-registry/auth


```


Create the first user, replacing username with the username you want to use. The -B flag orders the use of the bcrypt algorithm, which Docker requires:


```
htpasswd -Bc registry.password username


```


Enter the password when prompted, and the combination of credentials will be appended to registry.password.



Note: To add more users, re-run the previous command without -c, which creates a new file:
htpasswd -B registry.password username



Now that the list of credentials is made, you’ll edit docker-compose.yml to order Docker to use the file you created to authenticate users. Open it for editing by running:


```
nano ~/docker-registry/docker-compose.yml


```


Add the highlighted lines:


~/docker-registry/docker-compose.yml
```
version: '3'

services:
  registry:
    image: registry:2
    ports:
    - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - ./auth:/auth
      - ./data:/data

```


You’ve added environment variables specifying the use of HTTP authentication and provided the path to the file htpasswd created. For REGISTRY_AUTH, you have specified htpasswd as its value, which is the authentication scheme you are using, and set REGISTRY_AUTH_HTPASSWD_PATH to the path of the authentication file. REGISTRY_AUTH_HTPASSWD_REALM signifies the name of htpasswd realm.


You’ve also mounted the ./auth directory to make the file available inside the registry container. Save and close the file.


You can now verify that your authentication works correctly. First, navigate to the main directory:


```
cd ~/docker-registry


```


Then, run the registry by executing:


```
docker-compose up


```


In your browser, refresh the page of your domain. You’ll be asked for a username and password.


After providing a valid combination of credentials, you’ll see an empty JSON object:


```
{}

```


This means that you’ve successfully authenticated and gained access to the registry. Exit by pressing CTRL+C.


Your registry is now secured and can be accessed only after authentication. You’ll now configure it to run as a background process while being resilient to reboots by starting automatically.


# Step 4 — Starting Docker Registry as a Service


You can ensure that the registry container starts every time the system boots up, or after it crashes, by instructing Docker Compose to always keep it running. Open docker-compose.yml for editing:


```
nano docker-compose.yml


```


Add the following line under the registry block:


docker-compose.yml
```
...
  registry:
    restart: always
...

```


Setting restart to always ensures that the container will survive reboots. When you’re done, save and close the file.


You can now start your registry as a background process by passing in -d:


```
docker-compose up -d


```


With your registry running in the background, you can freely close the SSH session, and the registry won’t be affected.


Because Docker images may be very large in size, you’ll now increase the maximum file size that Nginx will accept for uploads.


# Step 5 — Increasing File Upload Size for Nginx


Before you can push an image to the registry, you need to ensure that your registry will be able to handle large file uploads.


The default size limit of file uploads in Nginx is 1m, which is not nearly enough for Docker images. To raise it, you’ll modify the main Nginx config file, located at /etc/nginx/nginx.conf. Open it for editing by running:


```
sudo nano /etc/nginx/nginx.conf


```


Find the http section, and add the following line:


/etc/nginx/nginx.conf
```
...
http {
        client_max_body_size 16384m;
        ...
}
...

```


The client_max_body_size parameter is now set to 16384m, making the maximum upload size equal to 16GB.


Save and close the file when you’re done.


Restart Nginx to apply the configuration changes:


```
sudo systemctl restart nginx


```


You can now upload large images to your Docker Registry without Nginx blocking the transfer or erroring out.


# Step 6 — Publishing to Your Private Docker Registry


Now that your Docker Registry server is up and running, and accepting large file sizes, you can try pushing an image to it. Since you don’t have any images readily available, you’ll use the ubuntu image from Docker Hub, a public Docker Registry, to test.


From your second, client server, run the following command to download the ubuntu image, run it, and get access to its shell:


```
docker run -t -i ubuntu /bin/bash


```


The -i and -t flags give you interactive shell access into the container.


Once you’re in, create a file called SUCCESS by running:


```
touch /SUCCESS


```


By creating this file, you have customized your container. You’ll later use it to check that you’re using exactly the same container.


Exit the container shell by running:


```
exit


```


Now, create a new image from the container you’ve just customized:


```
docker commit $(docker ps -lq) test-image


```


The new image is now available locally, and you’ll push it to your new container registry. First, you have to log in:


```
docker login https://your_domain


```


When prompted, enter in a username and password combination that you’ve defined in step 3 of this tutorial.


The output will be:


```
Output...
Login Succeeded

```


Once you’re logged in, rename the created image:


```
docker tag test-image your_domain/test-image


```


Finally, push the newly tagged image to your registry:


```
docker push your_domain/test-image


```


You’ll receive output similar to the following:


```
OutputThe push refers to a repository [your_domain/test-image]
420fa2a9b12e: Pushed
c20d459170d8: Pushed
db978cae6a05: Pushed
aeb3f02e9374: Pushed
latest: digest: sha256:88e782b3a2844a8d9f0819dc33f825dde45846b1c5f9eb4870016f2944fe6717 size: 1150

```


You’ve verified that your registry handles user authentication by logging in, and allows authenticated users to push images to the registry. You’ll now try pulling the image from your registry.


# Step 7 — Pulling From Your Private Docker Registry


Now that you’ve pushed an image to your private registry, you’ll try pulling from it.


On the main server, log in with the username and password you set up previously:


```
docker login https://your_domain


```


Try pulling the test-image by running:


```
docker pull your_domain/test-image


```


Docker should download the image. Run the container with the following command:


```
docker run -it your_domain/test-image /bin/bash


```


List the files present by running:


```
ls


```


You will see the SUCCESS file you’ve created earlier, confirming that its the same image you’ve created:


```
SUCCESS  bin  boot  dev  etc  home  lib  lib64  media   mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```


Exit the container shell by running:


```
exit


```


Now that you’ve tested pushing and pulling images, you’ve finished setting up a secure registry that you can use to store custom images.


# Conclusion


In this tutorial you set up your own private Docker Registry, and published a Docker image to it. As mentioned in the introduction, you can also use TravisCI or a similar CI tool to automate pushing to a private registry directly. By leveraging Docker containers in your workflow, you can ensure that the image containing the code will result in the same behavior on any machine, whether in production or in development. For more information on writing Docker files, you can visit the official docs on best practices.


