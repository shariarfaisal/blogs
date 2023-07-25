# How To Set Up the Eclipse Theia Cloud IDE Platform on Ubuntu 18 04

```Docker``` ```Cloud Computing``` ```Nginx``` ```Let's Encrypt``` ```Ubuntu 18.04```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


With developer tools moving to the cloud, adoption of cloud IDE (Integrated Development Environment) platforms is growing. Cloud IDEs are accessible from every type of modern device through web browsers, and they offer numerous advantages for real-time collaboration scenarios. Working in a cloud IDE provides a unified development and testing environment for you and your team, while minimizing platform incompatibilities. Accessible through web browsers, cloud IDEs are available from every type of modern device.


Eclipse Theia is an extensible cloud IDE running on a remote server and accessible from a web browser. Visually, it’s designed to look and behave similarly to Microsoft Visual Studio Code, which means that it supports many programming languages, has a flexible layout, and has an integrated terminal. What separates Eclipse Theia from other cloud IDE software is its extensibility; it can be modified using custom extensions, which allow you to craft a cloud IDE suited to your needs.


In this tutorial, you’ll deploy Eclipse Theia to your Ubuntu 18.04 server using Docker Compose, a container orchestration tool. You’ll expose it at your domain using nginx-proxy, an automated system for Docker that simplifies the process of configuring Nginx to serve as a reverse proxy for a container. You’ll also secure it using a free Let’s Encrypt TLS certificate, which you’ll provision using its specialized add-on. In the end, you’ll have Eclipse Theia running on your Ubuntu 18.04 server available via HTTPS and requiring the user to log in.


# Prerequisites


- An Ubuntu 18.04 server with root privileges, and a secondary, non-root account. You can set this up by following our Initial Server Setup Guide for Ubuntu 18.04. For this tutorial the non-root user is sammy.
- Docker installed on your server. Follow Step 1 and Step 2 of How To Install Docker on Ubuntu 18.04. For an introduction to Docker, see The Docker Ecosystem: An Introduction to Common Components.
- Docker Compose installed on your server. Follow Step 1 of How To Install Docker Compose on Ubuntu 18.04.
- A fully registered domain name. This tutorial will use theia.your_domain throughout. You can purchase a domain name on Namecheap, get one for free on Freenom, or use the domain registrar of your choice.
- An A DNS record with theia.your_domain pointing to your server’s public IP address. You can follow this introduction to DigitalOcean DNS for details on how to add them.

# Step 1 — Deploying nginx-proxy with Let’s Encrypt


In this section, you’ll deploy nginx-proxy and its Let’s Encrypt add-on using Docker Compose. This will enable automatic TLS certificate provisioning and renewal, so that when you deploy Eclipse Theia it will be accessible at your domain via HTTPS.


For the purposes of this tutorial, you’ll store all files under ~/eclipse-theia. Create the directory by running the following command:


```
mkdir ~/eclipse-theia


```


Navigate to it:


```
cd ~/eclipse-theia


```


You’ll store the Docker Compose configuration for nginx-proxy in a file named nginx-proxy-compose.yaml. Create it using your text editor:


```
nano nginx-proxy-compose.yaml


```


Add the following lines:


~/eclipse-theia/nginx-proxy-compose.yaml
```
version: '2'

services:
  nginx-proxy:
    restart: always
    image: jwilder/nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/etc/nginx/htpasswd:/etc/nginx/htpasswd"
      - "/etc/nginx/vhost.d"
      - "/usr/share/nginx/html"
      - "/var/run/docker.sock:/tmp/docker.sock:ro"
      - "/etc/nginx/certs"

  letsencrypt-nginx-proxy-companion:
    restart: always
    image: jrcs/letsencrypt-nginx-proxy-companion
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    volumes_from:
      - "nginx-proxy"

```


Here you’re defining two services that Docker Compose will run, nginx-proxy and its Let’s Encrypt companion. For the proxy, you specify jwilder/nginx-proxy as the image, map HTTP and HTTPS ports, and define volumes that will be accessible to it during runtime.


Volumes are directories on your server that the defined service will have full access to, which you’ll later use to set up user authentication. To achieve that, you’ll make use of the first volume from the list, which maps the local /etc/nginx/htpasswd directory to the same one in the container. In that folder, nginx-proxy expects to find a file named exactly as the target domain, containing log-in credentials for user authentication in the htpasswd format (username:hashed_password).


For the add-on, you name the Docker image and allow access to Docker’s socket by defining a volume. Then, you specify that the add-on should inherit access to the volumes defined for nginx-proxy. Both services have restart set to always, which orders Docker to restart the containers in case of crash or system reboot.


Save and close the file.


Deploy the configuration by running:


```
docker-compose -f nginx-proxy-compose.yaml up -d


```


Here you pass in the nginx-proxy-compose.yaml filename to the -f parameter of the docker-compose command, which specifies the file to run. Then, you pass the up verb that instructs it to run the containers. The -d flag enables detached mode, which means that Docker Compose will run the containers in the background.


The final output will look like this:


```
OutputCreating network "eclipse-theia_default" with the default driver
Pulling nginx-proxy (jwilder/nginx-proxy:)...
latest: Pulling from jwilder/nginx-proxy
8d691f585fa8: Pull complete
5b07f4e08ad0: Pull complete
...
Digest: sha256:dfc0666b9747a6fc851f5fb9b03e65e957b34c95d9635b4b5d1d6b01104bde28
Status: Downloaded newer image for jwilder/nginx-proxy:latest
Pulling letsencrypt-nginx-proxy-companion (jrcs/letsencrypt-nginx-proxy-companion:)...
latest: Pulling from jrcs/letsencrypt-nginx-proxy-companion
89d9c30c1d48: Pull complete
668840c175f8: Pull complete
...
Digest: sha256:a8d369d84079a923fdec8ce2f85827917a15022b0dae9be73e6a0db03be95b5a
Status: Downloaded newer image for jrcs/letsencrypt-nginx-proxy-companion:latest
Creating eclipse-theia_nginx-proxy_1 ... done
Creating eclipse-theia_letsencrypt-nginx-proxy-companion_1 ... done

```


You’ve deployed nginx-proxy and its Let’s Encrypt companion using Docker Compose. Now you’ll move on to set up Eclipse Theia at your domain and secure it.


# Step 2 — Deploying Dockerized Eclipse Theia


In this section, you’ll create a file containing any allowed log-in combinations that a user will need to input. Then, you’ll deploy Eclipse Theia to your server using Docker Compose and expose it at your secured domain using nginx-proxy.


As explained in the previous step, nginx-proxy expects log-in combinations to be in a file named after the exposed domain, in the htpasswd format and stored under the /etc/nginx/htpasswd directory in the container. The local directory which maps to the virtual one does not need to be the same, as was specified in the nginx-proxy config.


To create log-in combinations, you’ll first need to install htpasswd by running the following command:


```
sudo apt install apache2-utils


```


The apache2-utils package contains the htpasswd utility.


Create the /etc/nginx/htpasswd directory:


```
sudo mkdir -p /etc/nginx/htpasswd


```


Create a file that will store the logins for your domain:


```
sudo touch /etc/nginx/htpasswd/theia.your_domain


```


Remember to replace theia.your_domain with your Eclipse Theia domain.


To add a username and password combination, run the following command:


```
sudo htpasswd /etc/nginx/htpasswd/theia.your_domain username


```


Replace username with the username you want to add. You’ll be asked for a password twice. After providing it, htpasswd will add the username and hashed password pair to the end of the file. You can repeat this command for as many logins as you wish to add.


Now, you’ll create configuration for deploying Eclipse Theia. You’ll store it in a file named eclipse-theia-compose.yaml. Create it using your text editor:


```
nano eclipse-theia-compose.yaml


```


Add the following lines:


~/eclipse-theia/eclipse-theia-compose.yaml
```
version: '2.2'

services:
  eclipse-theia:
    restart: always
    image: theiaide/theia:next
    init: true
    environment:
      - VIRTUAL_HOST=theia.your_domain
      - LETSENCRYPT_HOST=theia.your_domain

```


In this config, you define a single service called eclipse-theia with restart set to always and theiaide/theia:next as the container image. You also set init to true to instruct Docker to use init as the main process manager when running Eclipse Theia inside the container.


Then, you specify two environment variables in the environment section: VIRTUAL_HOST and LETSENCRYPT_HOST. The former is passed on to nginx-proxy and tells it at what domain the container should be exposed, while the latter is used by its Let’s Encrypt add-on and specifies for which domain to request TLS certificates. Unless you specify a wildcard as the value for VIRTUAL_HOST, they must be the same.


Remember to replace theia.your_domain with your desired domain, then save and close the file.


Now deploy Eclipse Theia by running:


```
docker-compose -f eclipse-theia-compose.yaml up -d


```


The final output will look like:


```
Output...
Pulling eclipse-theia (theiaide/theia:next)...
next: Pulling from theiaide/theia
63bc94deeb28: Pull complete
100db3e2539d: Pull complete
...
Digest: sha256:c36dff04e250f1ac52d13f6d6e15ab3e9b8cad9ad68aba0208312e0788ecb109
Status: Downloaded newer image for theiaide/theia:next
Creating eclipse-theia_eclipse-theia_1 ... done

```


Then, in your browser, navigate to the domain you’re using for Eclipse Theia. Your browser will show you a prompt asking you to log in. After providing the correct credentials, you’ll enter Eclipse Theia and immediately see its editor GUI. In your address bar you’ll see a padlock indicating that the connection is secure. If you don’t see this immediately, wait a few minutes for the TLS certificates to provision, then reload the page.





Now that you can securely access your cloud IDE, you’ll start using the editor in the next step.


# Step 3 — Using the Eclipse Theia Interface


In this section, you’ll explore some of the features of the Eclipse Theia interface.


On the left-hand side of the IDE, there is a vertical row of four buttons opening the most commonly used features in a side panel.





This bar is customizable so you can move these views to a different order or remove them from the bar. By default, the first view opens the Explorer panel that provides tree-like navigation of the project’s structure. You can manage your folders and files here—creating, deleting, moving, and renaming them as necessary.


After creating a new file through the File menu, you’ll see an empty file open in a new tab. Once saved, you can view the file’s name in the Explorer side panel. To create folders right click on the Explorer sidebar and click on New Folder. You can expand a folder by clicking on its name as well as dragging and dropping files and folders to upper parts of the hierarchy to move them to a new location.





The next two options provide access to search and replace functionality. Following it, the next one provides a view of source control systems that you may be using, such as Git.


The final view is the debugger option, which provides all the common actions for debugging in the panel. You can save debugging configurations in the launch.json file.





The central part of the GUI is your editor, which you can separate by tabs for your code editing. You can change your editing view to a grid system or to side-by-side files. Like all modern IDEs, Eclipse Theia supports syntax highlighting for your code.





You can gain access to a terminal by typing CTRL+SHIFT+` , or by clicking on Terminal in the upper menu, and selecting New Terminal. The terminal will open in a lower panel and its working directory will be set to the project’s workspace, which contains the files and folders shown in the Explorer side panel.





You’ve explored a high-level overview of the Eclipse Theia interface and reviewed some of the most commonly used features.


# Conclusion


You now have Eclipse Theia, a versatile cloud IDE, installed on your Ubuntu 18.04 server using Docker Compose and nginx-proxy. You’ve secured it with a free Let’s Encrypt TLS certificate and set up the instance to require log-in credentials from the user. You can work on your source code and documents with it individually or collaborate with your team. You can also try building your own version of Eclipse Theia if you need additional functionality. For further information on how to do that, visit the Theia docs.


