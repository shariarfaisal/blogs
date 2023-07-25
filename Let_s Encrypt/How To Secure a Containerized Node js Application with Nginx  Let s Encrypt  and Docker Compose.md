# How To Secure a Containerized Node js Application with Nginx  Let s Encrypt  and Docker Compose

```Docker``` ```Security``` ```Node.js``` ```Let's Encrypt``` ```Ubuntu 18.04```

## Introduction


There are multiple ways to enhance the flexibility and security of your Node.js application. Using a reverse proxy like Nginx offers you the ability to load balance requests, cache static content, and implement Transport Layer Security (TLS). Enabling encrypted HTTPS on your server ensures that communication to and from your application remains secure.


Implementing a reverse proxy with TLS/SSL on containers involves a different set of procedures from working directly on a host operating system. For example, if you were obtaining certificates from Let’s Encrypt for an application running on a server, you would install the required software directly on your host. Containers allow you to take a different approach. Using Docker Compose, you can create containers for your application, your web server, and the Certbot client that will enable you to obtain your certificates. By following these steps, you can take advantage of the modularity and portability of a containerized workflow.


In this tutorial, you will deploy a Node.js application with an Nginx reverse proxy using Docker Compose. You will obtain TLS/SSL certificates for the domain associated with your application and ensure that it receives a high security rating from SSL Labs. Finally, you will set up a cron job to renew your certificates so that your domain remains secure.


# Prerequisites


To follow this tutorial, you will need:


- 
An Ubuntu 18.04 server, a non-root user with sudo privileges, and an active firewall. For guidance on how to set these up, please read this Initial Server Setup guide.

- 
Docker and Docker Compose installed on your server. For guidance on installing Docker, follow Steps 1 and 2 of How To Install and Use Docker on Ubuntu 18.04. For guidance on installing Compose, follow Step 1 of How To Install Docker Compose on Ubuntu 18.04.

- 
A registered domain name. This tutorial will use your_domain throughout. You can get one for free at Freenom, or use the domain registrar of your choice.

- 
Both of the following DNS records set up for your server. You can follow this introduction to DigitalOcean DNS for details on how to add them to a DigitalOcean account, if that’s what you’re using:

An A record with your_domain pointing to your server’s public IP address.
An A record with www.your_domain pointing to your server’s public IP address.


- An A record with your_domain pointing to your server’s public IP address.
- An A record with www.your_domain pointing to your server’s public IP address.

Once you have everything set up, you’re ready to begin the first step.


# Step 1 — Cloning and Testing the Node Application


As a first step, you’ll clone the repository with the Node application code, which includes the Dockerfile to build your application image with Compose. Then you’ll test the application by building and running it with the docker run command, without a reverse proxy or SSL.


In your non-root user’s home directory, clone the nodejs-image-demo repository from the DigitalOcean Community GitHub account. This repository includes the code from the setup described in How To Build a Node.js Application with Docker.


Clone the repository into a directory. This example uses node_project as the directory name. Feel free to name this directory to your liking:


```
git clone https://github.com/do-community/nodejs-image-demo.git node_project


```


Change into the node_project directory:


```
cd node_project


```


In this directory, there is a Dockerfile that contains instructions for building a Node application using the Docker node:10 image and the contents of your current project directory. You can preview the contents of the Dockerfile with the following:


```
cat Dockerfile


```


```
OutputFROM node:10-alpine

RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app

WORKDIR /home/node/app

COPY package*.json ./

USER node

RUN npm install

COPY --chown=node:node . .

EXPOSE 8080

CMD [ "node", "app.js" ]

```


These instructions build a Node image by copying the project code from the current directory to the container and installing dependencies with npm install. They also take advantage of Docker’s caching and image layering by separating the copy of package.json and package-lock.json, containing the project’s listed dependencies, from the copy of the rest of the application code. Finally, the instructions specify that the container will be run as the non-root node user with the appropriate permissions set on the application code and node_modules directories.


For more information about this Dockerfile and Node image best practices, please explore the complete discussion in Step 3 of How To Build a Node.js Application with Docker.


To test the application without SSL, you can build and tag the image using docker build and the -t flag. This example names the image node-demo, but you are free to name it something else:


```
docker build -t node-demo .


```


Once the build process is complete, you can list your images with docker images:


```
docker images


```


The following output confirms the application image build:


```
OutputREPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
node-demo           latest              23961524051d        7 seconds ago       73MB
node                10-alpine           8a752d5af4ce        3 weeks ago         70.7MB

```


Next, create the container with docker run. Three flags are included with this command:


- -p: This publishes the port on the container and maps it to a port on your host. You will use port 80 on the host in this example, but feel free to modify this as necessary if you have another process running on that port. For more information about how this works, review this discussion in the Docker documentation on port binding.
- -d: This runs the container in the background.
- --name: This allows you to give the container a memorable name.

Run the following command to build the container:


```
docker run --name node-demo -p 80:8080 -d node-demo


```


Inspect your running containers with docker ps:


```
docker ps


```


The following output confirms that your application container is running:


```
OutputCONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
4133b72391da        node-demo           "node app.js"       17 seconds ago      Up 16 seconds       0.0.0.0:80->8080/tcp   node-demo

```


You can now visit your domain to test your setup: http://your_domain. Remember to replace your_domain with your own domain name. Your application will display the following landing page:





Now that you have tested the application, you can stop the container and remove the images. Use docker ps to get your CONTAINER ID:


```
docker ps


```


```
OutputCONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
4133b72391da        node-demo           "node app.js"       17 seconds ago      Up 16 seconds       0.0.0.0:80->8080/tcp   node-demo

```


Stop the container with docker stop. Be sure to replace the CONTAINER ID listed here with your own application CONTAINER ID:


```
docker stop 4133b72391da


```


You can now remove the stopped container and all of the images, including unused and dangling images, with docker system prune and the -a flag:


```
docker system prune -a


```


Press y when prompted in the output to confirm that you would like to remove the stopped container and images. Be advised that this will also remove your build cache.


With your application image tested, you can move on to building the rest of your setup with Docker Compose.


# Step 2 — Defining the Web Server Configuration


With our application Dockerfile in place, you’ll create a configuration file to run your Nginx container. You can start with a minimal configuration that will include your domain name, document root, proxy information, and a location block to direct Certbot’s requests to the .well-known directory, where it will place a temporary file to validate that the DNS for your domain resolves to your server.


First, create a directory in the current project directory, node_project, for the configuration file:


```
mkdir nginx-conf


```


Create and open the file with nano or your favorite editor:


```
nano nginx-conf/nginx.conf


```


Add the following server block to proxy user requests to your Node application container and to direct Certbot’s requests to the .well-known directory. Be sure to replace your_domain  with your own domain name:


~/node_project/nginx-conf/nginx.conf
```
server {
        listen 80;
        listen [::]:80;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                proxy_pass http://nodejs:8080;
        }

        location ~ /.well-known/acme-challenge {
                allow all;
                root /var/www/html;
        }
}

```


This server block will allow you to start the Nginx container as a reverse proxy, which will pass requests to your Node application container. It will also allow you to use Certbot’s webroot plugin to obtain certificates for your domain. This plugin depends on the HTTP-01 validation method, which uses an HTTP request to prove that Certbot can access resources from a server that responds to a given domain name.


Once you have finished editing, save and close the file. If you used nano, you can do this by pressing CTRL + X, then Y, and ENTER. To learn more about Nginx server and location block algorithms, please refer to this article on Understanding Nginx Server and Location Block Selection Algorithms.


With the web server configuration details in place, you can move on to creating your docker-compose.yml file, which will allow you to create your application services and the Certbot container you will use to obtain your certificates.


# Step 3 — Creating the Docker Compose File


The docker-compose.yml file will define your services, including the Node application and web server. It will specify details like named volumes, which will be critical to sharing SSL credentials between containers, as well as network and port information. It will also allow you to specify commands to run when your containers are created. This file is the central resource that will define how your services will work together.


Create and open the file in your current directory:


```
nano docker-compose.yml


```


First, define the application service:


~/node_project/docker-compose.yml
```
version: '3'

services:
  nodejs:
    build:
      context: .
      dockerfile: Dockerfile
    image: nodejs
    container_name: nodejs
    restart: unless-stopped

```


The nodejs service definition includes the following:


- build: This defines the configuration options, including the context and dockerfile, that will be applied when Compose builds the application image. If you wanted to use an existing image from a registry like Docker Hub, you could use the image instruction instead, with information about your username, repository, and image tag.
- context: This defines the build context for the application image build. In this case, it’s the current project directory which is represented with the ..
- dockerfile: This specifies the Dockerfile that Compose will use for the build — the Dockerfile reviewed at in Step 1.
- image, container_name: These apply names to the image and container.
- restart: This defines the restart policy. The default is no, but in this example, the container is set to restart unless it is stopped.

Note that you are not including bind mounts with this service, since your setup is focused on deployment rather than development. For more information, please read the Docker documentation on bind mounts and volumes.


To enable communication between the application and web server containers, add a bridge network called app-network after the restart definition:


~/node_project/docker-compose.yml
```
services:
  nodejs:
...
    networks:
      - app-network

```


A user-defined bridge network like this enables communication between containers on the same Docker daemon host. This streamlines traffic and communication within your application, since it opens all ports between containers on the same bridge network, while exposing no ports to the outside world. Thus, you can be selective about opening only the ports you need to expose your frontend services.


Next, define the webserver service:


~/node_project/docker-compose.yml
```
...
 webserver:
    image: nginx:mainline-alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - web-root:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
      - certbot-var:/var/lib/letsencrypt
    depends_on:
      - nodejs
    networks:
      - app-network

```


Some of the settings defined here for the nodejs service remain the same, but some of the following changes were made:


- image: This tells Compose to pull the latest Alpine-based Nginx image from Docker Hub. For more information about alpine images, please read Step 3 of How To Build a Node.js Application with Docker.
- ports: This exposes port 80 to enable the configuration options you’ve defined in your Nginx configuration.

The following named volumes and bind mounts are also specified:


- web-root:/var/www/html: This will add your site’s static assets, copied to a volume called web-root, to the the /var/www/html directory on the container.
- ./nginx-conf:/etc/nginx/conf.d: This will bind mount the Nginx configuration directory on the host to the relevant directory on the container, ensuring that any changes you make to files on the host will be reflected in the container.
- certbot-etc:/etc/letsencrypt: This will mount the relevant Let’s Encrypt certificates and keys for your domain to the appropriate directory on the container.
- certbot-var:/var/lib/letsencrypt: This mounts Let’s Encrypt’s default working directory to the appropriate directory on the container.

Next, add the configuration options for the certbot container. Be sure to replace the domain and email information with your own domain name and contact email:


~/node_project/docker-compose.yml
```
...
  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - certbot-var:/var/lib/letsencrypt
      - web-root:/var/www/html
    depends_on:
      - webserver
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain  -d www.your_domain 

```


This definition tells Compose to pull the certbot/certbot image from Docker Hub. It also uses named volumes to share resources with the Nginx container, including the domain certificates and key in certbot-etc, the Let’s Encrypt working directory in certbot-var, and the application code in web-root.


Again, you’ve used depends_on to specify that the certbot container should be started once the webserver service is running.


The command option specifies the command to run when the container is started. It includes the certonly subcommand with the following options:


- --webroot: This tells Certbot to use the webroot plugin to place files in the webroot folder for authentication.
- --webroot-path: This specifies the path of the webroot directory.
- --email: Your preferred email for registration and recovery.
- --agree-tos: This specifies that you agree to ACME’s Subscriber Agreement.
- --no-eff-email: This tells Certbot that you do not wish to share your email with the Electronic Frontier Foundation (EFF). Feel free to omit this if you would prefer.
- --staging: This tells Certbot that you would like to use Let’s Encrypt’s staging environment to obtain test certificates. Using this option allows you to test your configuration options and avoid possible domain request limits. For more information about these limits, please read Let’s Encrypt’s rate limits documentation.
- -d: This allows you to specify domain names you would like to apply to your request. In this case, you’ve included your_domain and www.your_domain. Be sure to replace these with your own domains.

As a final step, add the volume and network definitions. Be sure to replace the username here with your own non-root user:


~/node_project/docker-compose.yml
```
...
volumes:
  certbot-etc:
  certbot-var:
  web-root:
    driver: local
    driver_opts:
      type: none
      device: /home/sammy/node_project/views/
      o: bind

networks:
  app-network:
    driver: bridge

```


Your named volumes include your Certbot certificate and working directory volumes, and the volume for your site’s static assets, web-root. In most cases, the default driver for Docker volumes is the local driver, which on Linux accepts options similar to the mount command. Thanks to this, you are able to specify a list of driver options with driver_opts that mount the views directory on the host, which contains your application’s static assets, to the volume at runtime. The directory contents can then be shared between containers. For more information about the contents of the views directory, please read Step 2 of How To Build a Node.js Application with Docker.


The following is the complete docker-compose.yml file:


~/node_project/docker-compose.yml
```
version: '3'

services:
  nodejs:
    build:
      context: .
      dockerfile: Dockerfile
    image: nodejs
    container_name: nodejs
    restart: unless-stopped
    networks:
      - app-network

  webserver:
    image: nginx:mainline-alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - web-root:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
      - certbot-var:/var/lib/letsencrypt
    depends_on:
      - nodejs
    networks:
      - app-network

  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - certbot-var:/var/lib/letsencrypt
      - web-root:/var/www/html
    depends_on:
      - webserver
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain  -d www.your_domain 

volumes:
  certbot-etc:
  certbot-var:
  web-root:
    driver: local
    driver_opts:
      type: none
      device: /home/sammy/node_project/views/
      o: bind

networks:
  app-network:
    driver: bridge  

```


With the service definitions in place, you are ready to start the containers and test your certificate requests.


# Step 4 — Obtaining SSL Certificates and Credentials


You can start the containers with docker-compose up. This will create and run your containers and services in the order you have specified. Once your domain requests succeed, your certificates will be mounted to the /etc/letsencrypt/live folder on the webserver container.


Create the services with docker-compose up with the -d flag, which will run the nodejs and webserver containers in the background:


```
docker-compose up -d


```


Your output will confirm that your services have been created:


```
OutputCreating nodejs ... done
Creating webserver ... done
Creating certbot   ... done

```


Use docker-compose ps to check the status of your services:


```
docker-compose ps


```


If everything was successful, your nodejs and webserver services will be Up and the certbot container will have exited with a 0 status message:


```
Output  Name                 Command               State          Ports
------------------------------------------------------------------------
certbot     certbot certonly --webroot ...   Exit 0
nodejs      node app.js                      Up       8080/tcp
webserver   nginx -g daemon off;             Up       0.0.0.0:80->80/tcp

```


If you notice anything other than Up in the State column for the nodejs and webserver services, or an exit status other than 0 for the certbot container, be sure to check the service logs with the docker-compose logs command. For example, if you wanted to check the Certbot log, you would run:


```
docker-compose logs certbot


```


You can now check that your credentials have been mounted to the webserver container with docker-compose exec:


```
docker-compose exec webserver ls -la /etc/letsencrypt/live


```


Once your request succeeds, your output will reveal the following:


```
Outputtotal 16
drwx------ 3 root root 4096 Dec 23 16:48 .
drwxr-xr-x 9 root root 4096 Dec 23 16:48 ..
-rw-r--r-- 1 root root  740 Dec 23 16:48 README
drwxr-xr-x 2 root root 4096 Dec 23 16:48 your_domain

```


Now that you know your request will be successful, you can edit the certbot service definition to remove the --staging flag.


Open the docker-compose.yml file:


```
nano docker-compose.yml


```


Find the section of the file with the certbot service definition, and replace the --staging flag in the command option with the --force-renewal flag. This will tell Certbot that you want to request a new certificate with the same domains as an existing certificate. The certbot service definition should have the following definitions:


~/node_project/docker-compose.yml
```
...
  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - certbot-var:/var/lib/letsencrypt
      - web-root:/var/www/html
    depends_on:
      - webserver
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --force-renewal -d your_domain -d www.your_domain
...

```


When you’re done editing, save and exit the file. You can now run docker-compose up to recreate the certbot container and its relevant volumes. By including the --no-deps option, you’re telling Compose that it can skip starting the webserver service, since it is already running:


```
docker-compose up --force-recreate --no-deps certbot


```


The following output indicates that your certificate request was successful:


```
OutputRecreating certbot ... done
Attaching to certbot
certbot      | Account registered.
certbot      | Renewing an existing certificate for your_domain and www.your_domain
certbot      |
certbot      | Successfully received certificate.
certbot      | Certificate is saved at: /etc/letsencrypt/live/your_domain/fullchain.pem
certbot      | Key is saved at:         /etc/letsencrypt/live/your_domain                               phd.com/privkey.pem
certbot      | This certificate expires on 2022-11-03.
certbot      | These files will be updated when the certificate renews.
certbot      | NEXT STEPS:
certbot      | - The certificate will need to be renewed before it expires. Cert                               bot can automatically renew the certificate in the background, but you may need                                to take steps to enable that functionality. See https://certbot.org/renewal-setu                               p for instructions.
certbot      | Saving debug log to /var/log/letsencrypt/letsencrypt.log
certbot      |
certbot      | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -                                - - - - - - -
certbot      | If you like Certbot, please consider supporting our work by:
certbot      |  * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/do                               nate
certbot      |  * Donating to EFF:                    https://eff.org/donate-le
certbot      | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -                                - - - - - - -
certbot exited with code 0

```


With your certificates in place, you can move on to modifying your Nginx configuration to include SSL.


# Step 5 — Modifying the Web Server Configuration and Service Definition


Enabling SSL in your Nginx configuration will involve adding an HTTP redirect to HTTPS and specifying your SSL certificate and key locations. It will also involve specifying the Diffie-Hellman group, which you will use for Perfect Forward Secrecy.


Since you are going to recreate the webserver service to include these additions, you can stop it now:


```
docker-compose stop webserver


```


Next, create a directory in your current project directory for the Diffie-Hellman key:


```
mkdir dhparam


```


Generate your key with the openssl command:


```
sudo openssl dhparam -out /home/sammy/node_project/dhparam/dhparam-2048.pem 2048


```


It will take a few moments to generate the key.


To add the relevant Diffie-Hellman and SSL information to your Nginx configuration, first remove the Nginx configuration file you created earlier:


```
rm nginx-conf/nginx.conf


```


Open another version of the file:


```
nano nginx-conf/nginx.conf


```


Add the following code to the file to redirect HTTP to HTTPS and to add SSL credentials, protocols, and security headers. Remember to replace your_domain with your own domain:


~/node_project/nginx-conf/nginx.conf
```

server {
        listen 80;
        listen [::]:80;
        server_name your_domain www.your_domain;

        location ~ /.well-known/acme-challenge {
          allow all;
          root /var/www/html;
        }

        location / {
                rewrite ^ https://$host$request_uri? permanent;
        }
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name your_domain www.your_domain;

        server_tokens off;

        ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem;

        ssl_buffer_size 8k;

        ssl_dhparam /etc/ssl/certs/dhparam-2048.pem;

        ssl_protocols TLSv1.2;
        ssl_prefer_server_ciphers on;

        ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

        ssl_ecdh_curve secp384r1;
        ssl_session_tickets off;

        ssl_stapling on;
        ssl_stapling_verify on;
        resolver 8.8.8.8;

        location / {
                try_files $uri @nodejs;
        }

        location @nodejs {
                proxy_pass http://nodejs:8080;
                add_header X-Frame-Options "SAMEORIGIN" always;
                add_header X-XSS-Protection "1; mode=block" always;
                add_header X-Content-Type-Options "nosniff" always;
                add_header Referrer-Policy "no-referrer-when-downgrade" always;
                add_header Content-Security-Policy "default-src * data: 'unsafe-eval' 'unsafe-inline'" always;
                #add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
                # enable strict transport security only if you understand the implications
        }

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;
}

```


The HTTP server block specifies the webroot for Certbot renewal requests to the .well-known/acme-challenge directory. It also includes a rewrite directive that directs HTTP requests to the root directory to HTTPS.


The HTTPS server block enables ssl and http2. To read more about how HTTP/2 iterates on HTTP protocols and the benefits it can have for website performance, please read the introduction to How To Set Up Nginx with HTTP/2 Support on Ubuntu 18.04. This block also includes a series of options to ensure that you are using the most up-to-date SSL protocols and ciphers and that OSCP stapling is turned on. OSCP stapling allows you to offer a time-stamped response from your certificate authority during the initial TLS handshake, which can speed up the authentication process.


The block also specifies your SSL and Diffie-Hellman credentials and key locations.


Finally, you’ve moved the proxy pass information to this block, including a location block with a try_files directive, pointing requests to your aliased Node.js application container, and a location block for that alias, which includes security headers that will enable you to get A ratings on things like the SSL Labs and Security Headers server test sites. These headers include X-Frame-Options, X-Content-Type-Options, Referrer Policy, Content-Security-Policy, and X-XSS-Protection. The HTTP Strict Transport Security (HSTS) header is commented out — enable this only if you understand the implications and have assessed its “preload” functionality.


Once you have finished editing, save and close the file.


Before recreating the webserver service, you need to add a few things to the service definition in your docker-compose.yml file, including relevant port information for HTTPS and a Diffie-Hellman volume definition.


Open the file:


```
nano docker-compose.yml


```


In the webserver service definition, add the following port mapping and the dhparam named volume:


~/node_project/docker-compose.yml
```
...
 webserver:
    image: nginx:latest
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - web-root:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
      - certbot-var:/var/lib/letsencrypt
      - dhparam:/etc/ssl/certs
    depends_on:
      - nodejs
    networks:
      - app-network

```


Next, add the dhparam volume to your volumes definitions. Remember to replace the sammy and node_project directories to match yours:


~/node_project/docker-compose.yml
```
...
volumes:
  ...
  webroot:
  ...
  dhparam:
    driver: local
    driver_opts:
      type: none
      device: /home/sammy/node_project/dhparam/
      o: bind

```


Similarly to the web-root volume, the dhparam volume will mount the Diffie-Hellman key stored on the host to the webserver container.


Save and close the file when you are finished editing.


Recreate the webserver service:


```
docker-compose up -d --force-recreate --no-deps webserver


```


Check your services with docker-compose ps:


```
docker-compose ps


```


The following output indicates that your nodejs and webserver services are running:


```
Output  Name                 Command               State                     Ports
----------------------------------------------------------------------------------------------
certbot     certbot certonly --webroot ...   Exit 0
nodejs      node app.js                      Up       8080/tcp
webserver   nginx -g daemon off;             Up       0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp

```


Finally, you can visit your domain to ensure that everything is working as expected. Navigate your browser to https://your_domain, making sure to substitute your_domain with your own domain name:





A lock icon should appear in your browser’s security indicator. If you would like to, you can navigate to the SSL Labs Server Test landing page or the Security Headers server test landing page. The configuration options included should earn your site an A rating on the SSL Labs Server Test. In order to get an A rating on the Security Headers server test, you would have to uncomment the Strict Transport Security (HSTS) header in your nginx-conf/nginx.conf file:


~/node_project/nginx-conf/nginx.conf
```
…
location @nodejs {
                proxy_pass http://nodejs:8080;
                add_header X-Frame-Options "SAMEORIGIN" always;
                add_header X-XSS-Protection "1; mode=block" always;
                add_header X-Content-Type-Options "nosniff" always;
                add_header Referrer-Policy "no-referrer-when-downgrade" always;
                add_header Content-Security-Policy "default-src * data: 'unsafe-eval' 'unsafe-inline'" always;
                add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
                # enable strict transport security only if you understand the implications
        }
…

```


Again, enable this option only if you understand the implications and have assessed its “preload” functionality.


# Step 6 — Renewing Certificates


Let’s Encrypt certificates are valid for 90 days. You can set up an automated renewal process to ensure that they do not lapse. One way to do this is to create a job with the cron scheduling utility. You can schedule a cron job using a script that will renew your certificates and reload your Nginx configuration.


Open a script called ssl_renew.sh in your project directory:


```
nano ssl_renew.sh


```


Add the following code to the script to renew your certificates and reload your web server configuration:


~/node_project/ssl_renew.sh
```
#!/bin/bash

COMPOSE="/usr/local/bin/docker-compose --ansi never"
DOCKER="/usr/bin/docker"

cd /home/sammy/node_project/
$COMPOSE run certbot renew --dry-run && $COMPOSE kill -s SIGHUP webserver
$DOCKER system prune -af

```


This script first assigns the docker-compose binary to a variable called COMPOSE, and specifies the --no-ansi option, which will run docker-compose commands without ANSI control characters. It then does the same with the docker binary. Finally, it changes to the ~/node_project directory and runs the following docker-compose commands:


- docker-compose run: This will start a certbot container and override the command provided in the certbot service definition. Instead of using the certonly subcommand use the renew subcommand, which will renew certificates that are close to expiring. Also included is the --dry-run option to test the script.
- docker-compose kill: This will send a SIGHUP signal to the webserver container to reload the Nginx configuration.

It then runs docker system prune to remove all unused containers and images.


Close the file when you are finished editing, then make it executable:


```
chmod +x ssl_renew.sh


```


Next, open your root crontab file to run the renewal script at a specified interval:


```
sudo crontab -e 

```


If this is your first time editing this file, you will be asked to choose an editor:


crontab
```
no crontab for root - using an empty one
Select an editor.  To change later, run 'select-editor'.
  1. /bin/ed
  2. /bin/nano        <---- easiest
  3. /usr/bin/vim.basic
  4. /usr/bin/vim.tiny
Choose 1-4 [2]: 
...

```


At the end of the file, add the following line:


crontab
```
...
*/5 * * * * /home/sammy/node_project/ssl_renew.sh >> /var/log/cron.log 2>&1

```


This will set the job interval to every five minutes, so you can test whether your renewal request has worked as intended. You have also created a log file, cron.log, to record relevant output from the job.


After five minutes, check cron.log to confirm whether the renewal request has succeeded:


```
tail -f /var/log/cron.log


```


After a few moments, the following output signals a successful renewal:


```
Output- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/your_domain/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Killing webserver ... done

```


```
Output…
Congratulations, all simulated renewals succeeded: 
  /etc/letsencrypt/live/your_domain/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Killing webserver ... 
Killing webserver ... done
Deleted Containers:
00cad94050985261e5b377de43e314b30ad0a6a724189753a9a23ec76488fd78

Total reclaimed space: 824.5kB


```


Exit out by entering CTRL + C in your terminal.


You can now modify the crontab file to set a daily interval. To run the script every day at noon, for example, you could modify the last line of the file like the following:


crontab
```
...
0 12 * * * /home/sammy/node_project/ssl_renew.sh >> /var/log/cron.log 2>&1

```


You can also remove the --dry-run option from your ssl_renew.sh script:


~/node_project/ssl_renew.sh
```
#!/bin/bash

COMPOSE="/usr/local/bin/docker-compose --no-ansi"
DOCKER="/usr/bin/docker"

cd /home/sammy/node_project/
$COMPOSE run certbot renew && $COMPOSE kill -s SIGHUP webserver
$DOCKER system prune -af

```


Your cron job will ensure that your Let’s Encrypt certificates don’t lapse by renewing them when they are eligible. You can also set up log rotation with the Logrotate utility to rotate and compress your log files.


# Conclusion


You have used containers to set up and run a Node application with an Nginx reverse proxy. You have also secured SSL certificates for your application’s domain and set up a cron job to renew these certificates when necessary.


If you are interested in learning more about Let’s Encrypt plugins, please review our articles on using the Nginx plugin or the standalone plugin.


You can also learn more about Docker Compose with the following resources:


- How To Install Docker Compose on Ubuntu 18.04.
- How To Configure a Continuous Integration Testing Environment with Docker and Docker Compose on Ubuntu 16.04.
- How To Set Up Laravel, Nginx, and MySQL with Docker Compose.

The Compose documentation is also a great resource for learning more about multi-container applications.


