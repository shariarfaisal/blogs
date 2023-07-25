# How To Install Drupal with Docker Compose

```Docker``` ```MySQL``` ```Drupal``` ```LEMP``` ```CMS``` ```Ubuntu 18.04``` ```Open Source``` ```Nginx```

The author selected United Nations Foundation to receive a donation as part of the Write for DOnations program.


The original WordPress version of this tutorial was written by Kathleen Juell.


## Introduction


Drupal is a content management system (CMS) written in PHP and distributed under the open-source GNU General Public License. People and organizations around the world use Drupal to power government sites, personal blogs, businesses, and more. What makes Drupal unique from other CMS frameworks is its growing community and a set of features that include secure processes, reliable performance, modularity, and flexibility to adapt.


Drupal requires installing the LAMP (Linux, Apache, MySQL, and PHP) or LEMP (Linux, Nginx, MySQL, and PHP) stack, but installing individual components is a time-consuming task. We can use tools like Docker and Docker Compose to simplify the process of installing Drupal. This tutorial will use Docker images for installing individual components within the Docker containers. By using Docker Compose, we can define and manage multiple containers for the database, application, and the networking/communication between them.


In this tutorial, we will install Drupal using Docker Compose so that we can take advantage of containerization and deploy our Drupal website on servers. We will be running containers for a MySQL database, Nginx webserver, and Drupal. We will also secure our installation by obtaining TLS/SSL certificates with Let’s Encrypt for the domain we want to associate with our site. Finally, we will set up a cron job to renew our certificates so that our domain remains secure.


# Prerequisites


To follow this tutorial, we will need:


- A server running Ubuntu 18.04, along with a non-root user with sudo privileges and an active firewall. For guidance on how to set these up, please see this Initial Server Setup guide.
- Docker installed on your server, following Steps 1 and 2 of How To Install and Use Docker on Ubuntu 18.04. This tutorial has been tested on version 19.03.8.
- Docker Compose installed on your server, following Step 1 of How To Install Docker Compose on Ubuntu 18.04. This tutorial has been tested on version 1.21.2.
- A registered domain name. This tutorial will use your_domain throughout. You can get one for free at Freenom, or use the domain registrar of your choice.
- Both of the following DNS records set up for your server. You can follow this introduction to DigitalOcean DNS for details on how to add them to a DigitalOcean account, if that’s what you’re using:

An A record with your_domain pointing to your server’s public IP address.
An A record with www.your_domain pointing to your server’s public IP address.


- An A record with your_domain pointing to your server’s public IP address.
- An A record with www.your_domain pointing to your server’s public IP address.

# Step 1 — Defining the Web Server Configuration


Before running any containers, we need to define the configuration for our Nginx web server. Our configuration file will include some Drupal-specific location blocks, along with a location block to direct Let’s Encrypt verification requests to the Certbot client for automated certificate renewals.


First, let’s create a project directory for our Drupal setup named drupal:


```
mkdir drupal 


```


Move into the newly created directory:


```
cd drupal


```


Now we can make a directory for our configuration file:


```
mkdir nginx-conf


```


Open the file with nano or your favorite text editor:


```
nano nginx-conf/nginx.conf


```


In this file, we will add a server block with directives for our server name and document root, and location blocks to direct the Certbot client’s request for certificates, PHP processing, and static asset requests.


Add the following code into the file. Be sure to replace your_domain with your own domain name:


~/drupal/nginx-conf/nginx.conf
```
server {
    listen 80;
    listen [::]:80;

    server_name your_domain www.your_domain;

    index index.php index.html index.htm;

    root /var/www/html;

    location ~ /.well-known/acme-challenge {
        allow all;
        root /var/www/html;
    }

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    rewrite ^/core/authorize.php/core/authorize.php(.*)$ /core/authorize.php$1;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass drupal:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ /\.ht {
        deny all;
    }

    location = /favicon.ico { 
        log_not_found off; access_log off; 
    }
    location = /robots.txt { 
        log_not_found off; access_log off; allow all; 
    }
    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
}

```


Our server block includes the following information:


Directives:


- 
listen: This tells Nginx to listen on port 80, which will allow us to use Certbot’s webroot plugin for our certificate requests. Note that we are not including port 443 yet—we will update our configuration to include SSL once we have successfully obtained our certificates.

- 
server_name: This defines our server name and the server block that should be used for requests to our server. Be sure to replace your_domain in this line with your own domain name.

- 
index: The index directive defines the files that will be used as indexes when processing requests to our server. We’ve modified the default order of priority here, moving index.php in front of index.html so that Nginx prioritizes files called index.php when possible.

- 
root: Our root directive names the root directory for requests to our server. This directory, /var/www/html, is created as a mount point at build time by instructions in our Drupal Dockerfile. These Dockerfile instructions also ensure that the files from the Drupal release are mounted to this volume.

- 
rewrite: If the specified regular expression (^/core/authorize.php/core/authorize.php(.*)$) matches a request URI, the URI is changed as specified in the replacement string (/core/authorize.php$1).


Location Blocks:


- 
location ~ /.well-known/acme-challenge: This location block will handle requests to the .well-known directory, where Certbot will place a temporary file to validate that the DNS for our domain resolves to our server. With this configuration in place, we will be able to use Certbot’s webroot plugin to obtain certificates for our domain.

- 
location /: In this location block, we’ll use a try_files directive to check for files that match individual URI requests. Instead of returning a 404 Not Found status as a default, however, we’ll pass control to Drupal’s index.php file with the request arguments.

- 
location ~ \.php$: This location block will handle PHP processing and proxy these requests to our drupal container. Because our Drupal Docker image will be based on the php:fpm image, we will also include configuration options that are specific to the FastCGI protocol in this block. Nginx requires an independent PHP processor for PHP requests: in our case, these requests will be handled by the php-fpm processor that’s included with the php:fpm image. Additionally, this location block includes FastCGI-specific directives, variables, and options that will proxy requests to the Drupal application running in our Drupal container, set the preferred index for the parsed request URI, and parse URI requests.

- 
location ~ /\.ht: This block will handle .htaccess files since Nginx won’t serve them. The deny_all directive ensures that .htaccess files will never be served to users.

- 
location = /favicon.ico, location = /robots.txt: These blocks ensure that requests to /favicon.ico and /robots.txt will not be logged.

- 
location ~* \.(css|gif|ico|jpeg|jpg|js|png)$: This block turns off logging for static asset requests and ensures that these assets are highly cacheable, as they are typically expensive to serve.


For more information about FastCGI proxying, see Understanding and Implementing FastCGI Proxying in Nginx. For information about server and location blocks, see Understanding Nginx Server and Location Block Selection Algorithms.


Save and close the file when you are finished editing.


With your Nginx configuration in place, you can move on to creating environment variables to pass to your application and database containers at runtime.


# Step 2 — Defining Environment Variables


Our Drupal application needs a database (MySQL, PostgresSQL, etc.) for saving information related to the site. The Drupal container will need access to certain environment variables at runtime in order to access the database (MySQL) container. These variables contain the sensitive information like the credentials of the database, so we can’t expose them directly in the Docker Compose file—the main file that contains information about how our containers will run.


It is always recommended to set the sensitive values in the .env file and restrict its circulation. This will prevent these values from copying over to our project repositories and being exposed publicly.


In the main project directory, ~/drupal, create and open a file called .env:


```
nano .env


```


Add the following variables to the .env file, replacing the highlighted sections with the credentials you want to use:


~/drupal/.env
```
MYSQL_ROOT_PASSWORD=root_password
MYSQL_DATABASE=drupal
MYSQL_USER=drupal_database_user
MYSQL_PASSWORD=drupal_database_password

```


We have now added the password for the MySQL root administrative account, as well as our preferred username and password for our application database.


Our .env file contains sensitive information so it is always recommended to include it in a project’s .gitignore and .dockerignore files so that it won’t be added in our Git repositories and Docker images.


If you plan to work with Git for version control, initialize your current working directory as a repository with git init:


```
git init


```


Open .gitignore file:


```
nano .gitignore


```


Add the following:


~/drupal/.gitignore
```
.env

```


Save and exit the file.


Similarly, open the .dockerignore file:


```
nano .dockerignore


```


Then add the following:


~/drupal/.dockerignore
```
.env
.git

```


Save and exit the file.


Now that we have taken measures to safeguard our credentials as environment variables, let’s move to our next step of defining our services in a docker-compose.yml file.


# Step 3 — Defining Services with Docker Compose


Docker Compose is a tool for defining and running multi-container Docker applications. We define a YAML file to configure our application’s services. A service in Docker Compose is a running container, and Compose allows us to link these services together with shared volumes and networks.


We will create different containers for our Drupal application, database, and web server. Along with these, we will also create a container to run Certbot in order to obtain certificates for our web server.


Create a docker-compose.yml file:


```
nano docker-compose.yml


```


Add the following code to define the Compose file version and mysql database service:


~/drupal/docker-compose.yml
```
version: "3"

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    env_file: .env
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - internal

```


Let’s go through these one-by-one with all the configuration options of the mysql service:


- 
image: This specifies the image that will be used/pulled for creating the container. It is always recommended to use the image with the proper version tag excluding the latest tag to avoid future conflicts. Read more on Dockerfile best practices from the Docker documentation.

- 
container_name: To define the name of the container.

- 
command: This is used to override the default command (CMD instruction) in the image. MySQL has supported different authentication plugins, but mysql_native_password is the traditional method to authenticate. Since PHP, and hence Drupal, won’t support the newer MySQL authentication, we need to set the --default-authentication-plugin=mysql_native_password as the default authentication mechanism.

- 
restart: This is used to define the container restart policy. The unless-stopped policy restarts a container unless it is stopped manually.

- 
env_file: This adds the environment variables from a file. In our case, it will read the environment variables from the .env file defined in the previous step.

- 
volumes: This mounts host paths or named volumes, specified as sub-options to a service. We are mounting a named volume called db-data to the /var/lib/mysql directory on the container, where MySQL by default will write its data files.

- 
networks: This defines the internal network that our application service will join. We will define the networks at the end of the file.


We have defined our mysql service definition, so now let’s add the definition of the drupal application service to the end of the file:


~/drupal/docker-compose.yml
```
...
  drupal:
    image: drupal:8.7.8-fpm-alpine
    container_name: drupal
    depends_on:
      - mysql
    restart: unless-stopped
    networks:
      - internal
      - external
    volumes:
      - drupal-data:/var/www/html

```


In this service definition, we are naming our container and defining a restart policy, as we did with the mysql service. We’re also adding some options specific to this container:


- 
image: Here, we are using the 8.7.8-fpm-alpine Drupal image. This image has the php-fpm processor that our Nginx web server requires to handle PHP processing. Moreover we are using the alpine image, derived from the Alpine Linux project, which will reduce the size of the overall image and is recommended in the Dockerfile best practices. Drupal has more versions of images, so check them out on Dockerhub.

- 
depends_on: This is used to express dependency between services. Defining the mysql service as the dependency to our drupal container will ensure that our drupal container will be created after the mysql container and enable our application to start smoothly.

- 
networks: Here, we have added this container to the external network along with the internal network. This will ensure that our mysql service is accessible only from the drupal container through the internal network while keeping this container accessible to other containers through the external network.

- 
volumes: We are mounting a named volume called drupal-data to the /var/www/html mountpoint created by the Drupal image. Using a named volume in this way will allow us to share our application code with other containers.


Next, let’s add the Nginx service definition after the drupal service definition:


~/drupal/docker-compose.yml
```
...
  webserver:
    image: nginx:1.17.4-alpine
    container_name: webserver
    depends_on:
      - drupal
    restart: unless-stopped
    ports:
      - 80:80
    volumes:
      - drupal-data:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
    networks:
      - external

```


Again, we’re naming our container and making it dependent on the Drupal container in order of starting. We’re also using an alpine image—the 1.17.4-alpine Nginx image.


This service definition also includes the following options:


- 
ports: This exposes port 80 to enable the configuration options we defined in our nginx.conf file in Step 1.

- 
volumes: Here, we are defining both the named volume and host path:


drupal-data:/var/www/html: This will mount our Drupal application code to the /var/www/html directory, which we set as the root in our Nginx server block.


./nginx-conf:/etc/nginx/conf.d: This will mount the Nginx configuration directory on the host to the relevant directory on the container, ensuring that any changes we make to files on the host will be reflected in the container.


certbot-etc:/etc/letsencrypt: This will mount the relevant Let’s Encrypt certificates and keys for our domain to the appropriate directory on the container.


networks: We have defined the external network only to let this container communicate with the drupal container and not with the mysql container.



- 
drupal-data:/var/www/html: This will mount our Drupal application code to the /var/www/html directory, which we set as the root in our Nginx server block.

- 
./nginx-conf:/etc/nginx/conf.d: This will mount the Nginx configuration directory on the host to the relevant directory on the container, ensuring that any changes we make to files on the host will be reflected in the container.

- 
certbot-etc:/etc/letsencrypt: This will mount the relevant Let’s Encrypt certificates and keys for our domain to the appropriate directory on the container.

- 
networks: We have defined the external network only to let this container communicate with the drupal container and not with the mysql container.


Finally, we will add our last service definition for the certbot service. Be sure to replace sammy@your_domain and your_domain with your own email and domain name:


~/drupal/docker-compose.yml
```
...
  certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - drupal-data:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain -d www.your_domain

```


This definition tells Compose to pull the certbot/certbot image from Docker Hub. It also uses named volumes to share resources with the Nginx container, including the domain certificates and key in certbot-etc and the application code in drupal-data.


We have also used depends_on to make sure that the certbot container will be started after the webserver service is running.


We haven’t specified any networks here because this container won’t communicate to any services over the network. It is only adding the domain certificates and key, which we have mounted using the named volumes.


We have also included the command option that specifies a subcommand to run with the container’s default certbot command. The Certbot client supports plugins for obtaining and installing certificates. We are using the webroot plugin to obtain a certificate by including certonly and --webroot on the command line. Read more about the plugin and additional commands from the official Certbot Documentation.


After the certbot service definition, add the network and volume definitions:


~/drupal/docker-compose.yml
```
...
networks:
  external:
    driver: bridge
  internal:
    driver: bridge

volumes:
  drupal-data:
  db-data:
  certbot-etc:

```


The top-level networks key lets us specify networks to be created. networks allows communication across the services/containers on all the ports since they are on the same Docker daemon host. We have defined two networks, internal and external, to secure the communication of the webserver, drupal, and mysql services.


The volumes key is used to define the named volumes drupal-data, db-data, and certbot-etc. When Docker creates volumes, the contents of the volume are stored in a directory on the host filesystem, /var/lib/docker/volumes/, that’s managed by Docker. The contents of each volume then get mounted from this directory to any container that uses the volume. In this way, it’s possible to share code and data between containers.


The finished docker-compose.yml file will look like this:


~/drupal/docker-compose.yml
```
version: "3"

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    env_file: .env
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - internal
  
  drupal:
    image: drupal:8.7.8-fpm-alpine
    container_name: drupal
    depends_on:
      - mysql
    restart: unless-stopped
    networks:
      - internal
      - external
    volumes:
      - drupal-data:/var/www/html
  
  webserver:
    image: nginx:1.17.4-alpine
    container_name: webserver
    depends_on:
      - drupal
    restart: unless-stopped
    ports:
      - 80:80
    volumes:
      - drupal-data:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
    networks:
      - external
  
  certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - drupal-data:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain -d www.your_domain

networks:
  external:
    driver: bridge
  internal:
    driver: bridge

volumes:
  drupal-data:
  db-data:
  certbot-etc:

```


We are done with defining our services. Next, let’s start the container and test our certificate requests.


# Step 4 — Obtaining SSL Certificates and Credentials


We can start our containers with the docker-compose up command, which will create and run our containers in the order we have specified. If our domain requests are successful, we will see the correct exit status in our output and the right certificates mounted in the /etc/letsencrypt/live folder on the web server container.


To run the containers in the background, use the docker-compose up command with the -d flag:


```
docker-compose up -d


```


You will see similar output confirming that your services have been created:


```
Output...
Creating mysql     ... done
Creating drupal    ... done
Creating webserver ... done
Creating certbot   ... done

```


Check the status of the services using the docker-compose ps command:


```
docker-compose ps


```


We will see the mysql, drupal, and webserver services with a State of Up, while certbot will be exited with a 0 status message:


```
Output  Name                 Command               State           Ports
--------------------------------------------------------------------------
certbot     certbot certonly --webroot ...   Exit 0
drupal      docker-php-entrypoint php-fpm    Up       9000/tcp
mysql       docker-entrypoint.sh --def ...   Up       3306/tcp, 33060/tcp
webserver   nginx -g daemon off;             Up       0.0.0.0:80->80/tcp

```


If you see anything other than Up in the State column for the mysql, drupal, or webserver services, or an exit status other than 0 for the certbot container, be sure to check the service logs with the docker-compose logs command:


```
docker-compose logs service_name


```


We can now check that our certificates mounted on the webserver container using the docker-compose exec command:


```
docker-compose exec webserver ls -la /etc/letsencrypt/live


```


This will give the following output:


```
Outputtotal 16
drwx------    3 root     root          4096 Oct  5 09:15 .
drwxr-xr-x    9 root     root          4096 Oct  5 09:15 ..
-rw-r--r--    1 root     root           740 Oct  5 09:15 README
drwxr-xr-x    2 root     root          4096 Oct  5 09:15 your_domain

```


Now that everything runs successfully, we can edit our certbot service definition to remove the --staging flag.


Open the docker-compose.yml file, go to the certbot service definition, and replace the --staging flag in the command option with the --force-renewal flag, which will tell Certbot that you want to request a new certificate with the same domains as an existing certificate. The updated certbot definition will look like this:


~/drupal/docker-compose.yml
```
...
  certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - drupal-data:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --force-renewal -d your_domain -d www.your_domain
...

```


We need to run docker-compose up again to recreate the certbot container. We will also include the --no-deps option to tell Compose that it can skip starting the webserver service, since it is already running:


```
docker-compose up --force-recreate --no-deps certbot


```


We will see output indicating that our certificate request was successful:


```
OutputRecreating certbot ... done
Attaching to certbot
certbot      | Saving debug log to /var/log/letsencrypt/letsencrypt.log
certbot      | Plugins selected: Authenticator webroot, Installer None
certbot      | Renewing an existing certificate
certbot      | Performing the following challenges:
certbot      | http-01 challenge for your_domain
certbot      | http-01 challenge for www.your_domain
certbot      | Using the webroot path /var/www/html for all unmatched domains.
certbot      | Waiting for verification...
certbot      | Cleaning up challenges
certbot      | IMPORTANT NOTES:
certbot      |  - Congratulations! Your certificate and chain have been saved at:
certbot      |    /etc/letsencrypt/live/your_domain/fullchain.pem
certbot      |    Your key file has been saved at:
certbot      |    /etc/letsencrypt/live/your_domain/privkey.pem
certbot      |    Your cert will expire on 2020-01-03. To obtain a new or tweaked
certbot      |    version of this certificate in the future, simply run certbot
certbot      |    again. To non-interactively renew *all* of your certificates, run
certbot      |    "certbot renew"
certbot      |  - Your account credentials have been saved in your Certbot
certbot      |    configuration directory at /etc/letsencrypt. You should make a
certbot      |    secure backup of this folder now. This configuration directory will
certbot      |    also contain certificates and private keys obtained by Certbot so
certbot      |    making regular backups of this folder is ideal.
certbot      |  - If you like Certbot, please consider supporting our work by:
certbot      |
certbot      |    Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
certbot      |    Donating to EFF:                    https://eff.org/donate-le
certbot      |
certbot exited with code 0

```


Now that we have successfully generated our certificates, we can update our Nginx Configuration to include SSL.


# Step 5 — Modifying the Web Server Configuration and Service Definition


After installing SSL certificates in Nginx, we will need to redirect all the HTTP requests to HTTPS. We will also have to specify our SSL certificate and key locations and add security parameters and headers.


Since you are going to recreate the webserver service to include these additions, you can stop it now:


```
docker-compose stop webserver


```


This will give the following output:


```
OutputStopping webserver ... done

```


Next, let’s remove the Nginx configuration file we created earlier:


```
rm nginx-conf/nginx.conf


```


Open another version of the file:


```
nano nginx-conf/nginx.conf


```


Add the following code to the file to redirect HTTP to HTTPS and to add SSL credentials, protocols, and security headers. Remember to replace your_domain with your own domain:


~/drupal/nginx-conf/nginx.conf
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
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name your_domain www.your_domain;

    index index.php index.html index.htm;

    root /var/www/html;

    server_tokens off;

    ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem;

    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src * data: 'unsafe-eval' 'unsafe-inline'" always;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    rewrite ^/core/authorize.php/core/authorize.php(.*)$ /core/authorize.php$1;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass drupal:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ /\.ht {
        deny all;
    }

    location = /favicon.ico {
        log_not_found off; access_log off;
    }
    location = /robots.txt {
        log_not_found off; access_log off; allow all;
    }
    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
}

```


The HTTP server block specifies the webroot plugin for Certbot renewal requests to the .well-known/acme-challenge directory. It also includes a rewrite directive that directs HTTP requests to the root directory to HTTPS.


The HTTPS server block enables ssl and http2. To read more about how HTTP/2 iterates on HTTP protocols and the benefits it can have for website performance, please see the introduction to How To Set Up Nginx with HTTP/2 Support on Ubuntu 18.04.


These blocks enable SSL, as we have included our SSL certificate and key locations along with the recommended headers. These headers will enable us to get an A rating on the SSL Labs and Security Headers server test sites.


Our root and index directives are also located in this block, as are the rest of the Drupal-specific location blocks discussed in Step 1.


Save and close the updated Nginx configuration file.


Before recreating the webserver container, we will need to add a 443 port mapping to our webserver service definition as we have enabled SSL certificates.


Open the docker-compose.yml file:


```
nano docker-compose.yml


```


Make the following changes in the webserver service definition:


~/drupal/docker-compose.yml
```
...
  webserver:
    image: nginx:1.17.4-alpine
    container_name: webserver
    depends_on:
      - drupal
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
    volumes:
      - drupal-data:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
    networks:
      - external
...

```


After enabling the SSL certificates, our docker-compose.yml will look like this:


~/drupal/docker-compose.yml
```
version: "3"

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    env_file: .env
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - internal
  
  drupal:
    image: drupal:8.7.8-fpm-alpine
    container_name: drupal
    depends_on:
      - mysql
    restart: unless-stopped
    networks:
      - internal
      - external
    volumes:
      - drupal-data:/var/www/html
  
  webserver:
    image: nginx:1.17.4-alpine
    container_name: webserver
    depends_on:
      - drupal
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
    volumes:
      - drupal-data:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
    networks:
      - external
  
  certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - drupal-data:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --force-renewal -d your_domain -d www.your_domain

networks:
  external:
    driver: bridge
  internal:
    driver: bridge

volumes:
  drupal-data:
  db-data:
  certbot-etc:

```


Save and close the file. Let’s recreate the webserver service with our updated configuration:


```
docker-compose up -d --force-recreate --no-deps webserver


```


This will give the following output:


```
OutputRecreating webserver ... done

```


Check the services with docker-compose ps:


```
docker-compose ps


```


We will see the mysql, drupal, and webserver services as Up while certbot will be exited with a 0 status message:


```
Output  Name                 Command               State           Ports
--------------------------------------------------------------------------
certbot     certbot certonly --webroot ...   Exit 0
drupal      docker-php-entrypoint php-fpm    Up       9000/tcp
mysql       docker-entrypoint.sh --def ...   Up       3306/tcp, 33060/tcp
webserver   nginx -g daemon off;             Up       0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp

```


Now, all our services are running and we are good to move forward with installing Drupal through the web interface.


# Step 6 — Completing the Installation Through the Web Interface


Let’s complete the installation through Drupal’s web interface.


In a web browser, navigate to the server’s domain. Remember to substitute your_domain here with your own domain name:


```
https://your_domain

```


Select the language to use:





Click Save and continue. We will land on the Installation profile page. Drupal has multiple profiles, so select the Standard profile and click on Save and continue.





After selecting the profile, we will move forward to Database configuration page. Select the Database type as MySQL, MariaDB, Percona Server, or equivalent and enter the values of Database name, username, and password from the values corresponding to MYSQL_DATABASE, MYSQL_USER, and MYSQL_PASSWORD respectively defined in the .env file in Step 2. Click on Advanced Options and set the value of Host to the name of the mysql service container. Click on Save and continue.





After configuring the database, it will start installing Drupal default modules and themes:





Once the site is installed, we will land on the Drupal site setup page for configuring the site name, email, username, password, and regional settings. Fill in the information and click on Save and continue:





After clicking Save and continue, we can see the Welcome to Drupal page, which shows that our Drupal site is up and running successfully.





Now that our Drupal installation is complete, we need to ensure that our SSL certificates will renew automatically.


# Step 7 — Renewing Certificates


Let’s Encrypt certificates are valid for 90 days, so we need to set up an automated renewal process to ensure that they do not lapse. One way to do this is to create a job with the cron scheduling utility. In this case, we will create a cron job to periodically run a script that will renew our certificates and reload our Nginx configuration.


Let’s create the ssl_renew.sh file to renew our certificates:


```
nano ssl_renew.sh


```


Add the following code. Remember to replace the directory name with your own non-root user:


~/drupal/ssl_renew.sh
```

#!/bin/bash

cd /home/sammy/drupal/
/usr/local/bin/docker-compose -f docker-compose.yml run certbot renew --dry-run && \
/usr/local/bin/docker-compose -f docker-compose.yml kill -s SIGHUP webserver

```


This script changes to the ~/drupal project directory and runs the following docker-compose commands.


- 
docker-compose run: This will start a certbot container and override the command provided in our certbot service definition. Instead of using the certonly subcommand, we’re using the renew subcommand here, which will renew certificates that are close to expiring. We’ve included the --dry-run option here to test our script.

- 
docker-compose kill: This will send a SIGHUP signal to the webserver container to reload the Nginx configuration.


Close the file and make it executable by running the following command:


```
sudo chmod +x ssl_renew.sh


```


Next, open the root crontab file to run the renewal script at a specified interval:


```
sudo crontab -e


```


If this is your first time editing this file, you will be asked to choose a text editor to open the file with:


```
Outputno crontab for root - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]:
...

```


At the end of the file, add the following line, replacing sammy with your username:


crontab
```
...
*/5 * * * * /home/sammy/drupal/ssl_renew.sh >> /var/log/cron.log 2>&1

```


This will set the job interval to every five minutes, so we can test whether or not our renewal request has worked as intended. We have also created a log file, cron.log, to record relevant output from the job.


After five minutes, use the tail command to check cron.log to see whether or not the renewal request has succeeded:


```
tail -f /var/log/cron.log


```


You will see output confirming a successful renewal:


```
Output** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/your_domain/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

```


Press CTRL+C to quit the tail process.


We can now modify the crontab file to run the script every 2nd day of the week at 2 AM. Change the final line of the crontab to the following:


crontab
```
...
* 2 * * 2 /home/sammy/drupal/ssl_renew.sh >> /var/log/cron.log 2>&1

```


Exit and save the file.


Now, let’s remove the --dry-run option from the ssl_renew.sh script. First, open it up:


```
nano ssl_renew.sh


```


Then change the contents to the following:


~/drupal/ssl_renew.sh
```
#!/bin/bash

cd /home/sammy/drupal/
/usr/local/bin/docker-compose -f docker-compose.yml run certbot renew && \
/usr/local/bin/docker-compose -f docker-compose.yml kill -s SIGHUP webserver

```


Our cron job will now take care of our SSL certificates expiry by renewing them when they are eligible.


# Conclusion


In this tutorial, we have used Docker Compose to create a Drupal installation with an Nginx web server. As part of this workflow, we obtained TLS/SSL certificates for the domain we wanted associated with our Drupal site and created a cron job to renew these certificates when necessary.


If you would like to learn more about Docker, check out our Docker topic page.


