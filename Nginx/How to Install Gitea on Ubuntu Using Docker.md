# How to Install Gitea on Ubuntu Using Docker

```Docker``` ```Ubuntu``` ```Git``` ```Let's Encrypt``` ```Nginx``` ```Ubuntu 20.04```

## Introduction


When working on software development, it’s important to be able to manage the source code in an efficient and traceable way. Source code management (SCM) systems are an excellent way to provide an efficient and flexible process for working on projects of any size with any number of developers. Many different pieces of SCM software have existed through the years, from CVS to SubVersion, Perforce to Mercurial, but the current industry leader is Git, which has seen major growth with the popularity of sites such as GitHub and GitLab.


However, with free accounts on these services geared towards public, open-source repositories, the ability to work on private or proprietary software incurs a cost to the developer. Additionally, one’s access to the repository is beholden to an external organization, and many would prefer to control their own software from start to finish.


To that end, several self-hosted solutions such as Gogs, Gitea, and GitLab have been developed over the last several years. This tutorial focuses on setting up one of the more popular solutions, Gitea, to allow you to host private repositories and manage your own projects throughout their entire life-cycle. Gitea is small, self-contained, and lightweight, making it a quick process to deploy without breaking the bank on hardware requirements. You will be using a Docker installation of Gitea, which ensures that the software will be kept up to date.


# Prerequisites


Before beginning this tutorial, you should have the following:


- An Ubuntu 20.04 server with a non-root user configured with sudo privileges as described in the initial server setup for Ubuntu 20.04.
- Docker installed on your server. Follow Steps 1 and 2 of How to Install Docker on Ubuntu 20.04 to install Docker.
- Docker Compose installed on your server. Follow Step 1 of our guide on How to Install and Use Docker Compose on Ubuntu 20.04 to set this up.
- A domain name pointed at your server. If you are using a DigitalOcean Droplet, you can accomplish this by following our Domains and DNS documentation. This tutorial will use your_domain in examples throughout.

# Step 1 — Creating the Git user


Gitea, like many source code repositories, uses SSH for accessing remote repositories. This allows users to control access to their code by managing their SSH keys within Gitea itself. In order for users to be able to access the host via SSH, however, you will need to create a git user on the host machine. This step is completed first so that you can access the user’s user and group ID.


First, create the user on the host who will be accepting these connections:


```
sudo adduser --system --shell /bin/bash --gecos 'Git Version Control' --group --disabled-password --home /home/git git


```


In this command, you create a system user that uses bash as its shell, but does not have a login password. This allows you to use sudo to run commands as that user but prevents logging in as it. You also set the user’s home directory to /home/git.


This command will output some information about the user it just created:


```
OutputAdding system user `git' (UID 112) ...
Adding new group `git' (GID 119) ...
Adding new user `git' (UID 112) with group `git' ...
Creating home directory `/home/git' …

```


Make note of the UID and GID values provided here (in this case, a UID of 112 and a GID of 119), as they will be used in a future step.


# Step 2 — Installing the Gitea Docker Image


Gitea has an image available in the global Docker repository, meaning that, using Docker Compose, you can install and run that image as a service with little extra work. The image itself runs the Gitea web and SSH services, allowing Git access both from the browser and the command line.


In order to spin up the Gitea container, you will be using Docker Compose, a declarative tool for setting up an environment.


To begin with, create a directory to host your service and enter it:


```
mkdir ~/gitea
cd ~/gitea


```


Once there, create a file called docker-compose.yml using your preferred text editor. The following example uses nano. This file will contain the descriptions of the containers that will run as part of your Gitea installation:


```
nano docker-compose.yml


```


Add the following into this new file:


~/gitea/docker-compose.yml
```
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: gitea/gitea:1.16.5
    container_name: gitea
    environment:
      - USER_UID=UID_from_step_1
      - USER_GID=GID_from_step_1
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
      - /home/git/.ssh/:/data/git/.ssh
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "127.0.0.1:3000:3000"
      - "127.0.0.1:2222:22"

```


Let’s walk through what this file does:


- version: "3": this lets Docker Compose what version of configuration file this is.
- networks: this section declares the networking setup of our collection of containers. In this case, a gitea network is created, but is not exposed externally.
- services

image: gitea/gitea:1.16.5: this specifies that we will be using Gitea version 1.16.5; however you can change the value after the colon to specify other versions, whether a specific release, a major version such as :1, or a tag such as :latest or :dev.
environment: the environment section specifies environment variables that will be available to the image during installation and running. In this case, we are specifying a user and group ID for the environment, using the UID and GID provided in the output of the adduser command in Step 1.
restart: always: this line instructs Docker to always restart the container if it goes down, whether because the container itself goes down or the host machine does; essentially, Gitea will start on boot.
networks: this specifies that the Gitea service will have access to and be accessible on the network named above.
./gitea:/data and /home/git/.ssh/:/data/git/.ssh: these are the locations where Gitea will store its repositories and related data. Currently, this is mapped to the folder named gitea in the current directory. Docker will create this folder when the container starts if it doesn’t exist. The .ssh folder will be described further later on in Step 6.
/etc/timezone and /etc/localtime: these two files contain information about the timezone and time on the host machine. By mapping these directly into the container as read-only files (specified with the final :ro part of the definitions), the container will have the same information as the host.
ports: Gitea listens for connections on two ports. It listens for HTTP connections on port 3000, where it serves the web interface for the source code repository, and it listens for SSH connections on port 22. In this case, you are keeping port 3000 for HTTP connections by mapping it to the same number, and you are mapping the port on Gitea’s container from the usual 22 to 2222 to avoid port-clashing. In Step 6, you will set up an SSH shim to direct traffic to Gitea when requested.


- image: gitea/gitea:1.16.5: this specifies that we will be using Gitea version 1.16.5; however you can change the value after the colon to specify other versions, whether a specific release, a major version such as :1, or a tag such as :latest or :dev.
- environment: the environment section specifies environment variables that will be available to the image during installation and running. In this case, we are specifying a user and group ID for the environment, using the UID and GID provided in the output of the adduser command in Step 1.
- restart: always: this line instructs Docker to always restart the container if it goes down, whether because the container itself goes down or the host machine does; essentially, Gitea will start on boot.
- networks: this specifies that the Gitea service will have access to and be accessible on the network named above.
- ./gitea:/data and /home/git/.ssh/:/data/git/.ssh: these are the locations where Gitea will store its repositories and related data. Currently, this is mapped to the folder named gitea in the current directory. Docker will create this folder when the container starts if it doesn’t exist. The .ssh folder will be described further later on in Step 6.
- /etc/timezone and /etc/localtime: these two files contain information about the timezone and time on the host machine. By mapping these directly into the container as read-only files (specified with the final :ro part of the definitions), the container will have the same information as the host.
- ports: Gitea listens for connections on two ports. It listens for HTTP connections on port 3000, where it serves the web interface for the source code repository, and it listens for SSH connections on port 22. In this case, you are keeping port 3000 for HTTP connections by mapping it to the same number, and you are mapping the port on Gitea’s container from the usual 22 to 2222 to avoid port-clashing. In Step 6, you will set up an SSH shim to direct traffic to Gitea when requested.


Note: This is a minimal example of a Docker Compose file for Gitea. There are several other options that one can include, such as using MySQL or PostGreSQL as the backing database or a named volume for storage. This minimal setup uses SQLite as the backing database and a volume using the directory named gitea for storage. You can read more about these options in Gitea’s documentation.

Save and close the file. If you used nano to edit the file, you can do so by pressing CTRL + X, Y, and then ENTER.


With this file in place you can then bring the containers up using Docker Compose:


```
docker-compose up


```


This command will pull down the images, start the Gitea container, and will return output like this:


```
Output[+] Running 9/9
 ⠿ server Pulled                                                                                                  8.2s
   ⠿ e1096b72685a Pull complete                                                                                   1.4s
   ⠿ ac9df86bb932 Pull complete                                                                                   3.3s
   ⠿ 6d34ed99b58a Pull complete                                                                                   3.4s
   ⠿ a8913d040fab Pull complete                                                                                   3.6s
   ⠿ a5d3a72a2366 Pull complete                                                                                   5.3s
   ⠿ 1f0dcaae29cc Pull complete                                                                                   5.6s
   ⠿ f284bcea5adb Pull complete                                                                                   7.3s
   ⠿ 0f09c34c97e3 Pull complete                                                                                   7.5s
[+] Running 2/2
 ⠿ Network gitea_gitea  Created                                                                                   0.2s
 ⠿ Container gitea      Created                                                                                   0.2s
Attaching to gitea
gitea  | Generating /data/ssh/ssh_host_ed25519_key...
gitea  | Generating /data/ssh/ssh_host_rsa_key...
gitea  | Generating /data/ssh/ssh_host_dsa_key...
gitea  | Generating /data/ssh/ssh_host_ecdsa_key...
gitea  | Server listening on :: port 22.
gitea  | Server listening on 0.0.0.0 port 22.
gitea  | 2022/03/31 17:26:21 cmd/web.go:102:runWeb() [I] Starting Gitea on PID: 14
gitea  | 2022/03/31 17:26:21 ...s/install/setting.go:21:PreloadSettings() [I] AppPath: /usr/local/bin/gitea
gitea  | 2022/03/31 17:26:21 ...s/install/setting.go:22:PreloadSettings() [I] AppWorkPath: /app/gitea
gitea  | 2022/03/31 17:26:21 ...s/install/setting.go:23:PreloadSettings() [I] Custom path: /data/gitea
gitea  | 2022/03/31 17:26:21 ...s/install/setting.go:24:PreloadSettings() [I] Log path: /data/gitea/log
gitea  | 2022/03/31 17:26:21 ...s/install/setting.go:25:PreloadSettings() [I] Configuration file: /data/gitea/conf/app.ini
gitea  | 2022/03/31 17:26:21 ...s/install/setting.go:26:PreloadSettings() [I] Prepare to run install page
gitea  | 2022/03/31 17:26:21 ...s/install/setting.go:29:PreloadSettings() [I] SQLite3 is supported
gitea  | 2022/03/31 17:26:21 cmd/web.go:208:listen() [I] Listen: http://0.0.0.0:3000
gitea  | 2022/03/31 17:26:21 cmd/web.go:212:listen() [I] AppURL(ROOT_URL): http://localhost:3000/

```


This will leave the container running in the foreground, however, and it will stop as soon as you exit the process with Ctrl + C or by losing your connection. In order to have the container run in the background as a separate process, you can append the -d flag to the Compose command:


```
docker-compose up -d


```


You will be notified when the container starts and then returned to your shell.


# Step 3 — Installing Nginx as a Reverse Proxy


Running a web service such as Gitea behind a reverse proxy is common practice, as modern server software such as Apache or Nginx can more easily handle multiple services on one machine, balance load across multiple servers, and handle SSL. Additionally, this will allow you to set up a domain name pointing to your Gitea instance running on standard HTTP(S) ports.


For the purposes of this tutorial, we’ll use Nginx. First, update the package lists on your host machine:


```
sudo apt update


```


Next, install Nginx using apt:


```
sudo apt install nginx


```


Now, as you are using the firewall ufw, you will need to allow access to these ports:


```
sudo ufw allow "Nginx Full"


```


Once this is installed, you should be able to access your server in  your browser by visiting http://your_domain. This will lead you to a very plain page welcoming you to Nginx.


At this point, you’ll need to create a reverse proxy entry to direct incoming traffic through Nginx to the Gitea instance running in Docker. Create a new file in the Nginx sites-available directory using your preferred text editor. The following example uses nano:


```
sudo nano /etc/nginx/sites-available/gitea


```


In this file, set up a new server block with requests to / proxied to your Gitea instance:


/etc/nginx/sites-available/gitea
```
server {
    # Listen for requests on your domain/IP address.
    server_name your_domain;

    root /var/www/html;

    location / {
        # Proxy all requests to Gitea running on port 3000
        proxy_pass http://localhost:3000;
        
        # Pass on information about the requests to the proxied service using headers
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

```


Once you are finished editing the file, save and close it.



Note: For more information on understanding what’s going on within these directives, see the Understanding Nginx HTTP Proxying, Load Balancing, Buffering, and Caching tutorial.

Nginx determines what sites it will actually serve based on whether or not those files are present in its sites-enabled directory. This is managed via symbolic links pointing to the files in the sites-available directory. You will need to create one of those symbolic links for Nginx to start serving Gitea:


```
sudo ln -s /etc/nginx/sites-available/gitea /etc/nginx/sites-enabled/gitea


```


Before you restart Nginx to make your changes live, you should have Nginx itself check that those changes are valid by testing its config.


```
sudo nginx -t


```


If everything’s okay, this command will return output like the following:


```
Outputnginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


If there are any issues, it will tell you what and where they are.


When you’re ready to move ahead with this change, restart the Nginx system service:


```
sudo systemctl restart nginx


```


Now, when you visit http://your_domain in your browser, you should find yourself on the initial setup page for Gitea ready for you to fill out.


# Step 4 — Installing Certbot and Setting Up TLS Certificates


Thanks to Certbot and the Let’s Encrypt free certificate authority, adding TLS encryption to your Gitea installation app will take only two commands.


First, install Certbot and its Nginx plugin:


```
sudo apt install certbot python3-certbot-nginx


```


Next, run certbot in --nginx mode, and specify the same domain that you used in the Nginx server_name configuration directive:


```
sudo certbot --nginx -d your_domain_here


```


You’ll be prompted to agree to the Let’s Encrypt terms of service, and to enter an email address.


Afterwards, you’ll be asked if you want to redirect all HTTP traffic to HTTPS. It’s up to you, but this is generally recommended and safe to do.


After that, Let’s Encrypt will confirm your request and Certbot will download your certificate:


```
OutputCongratulations! You have successfully enabled https://your_domain

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=your_domain
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your_domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your_domain/privkey.pem
   Your cert will expire on 2022-05-09. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```


Certbot will automatically reload Nginx with the new configuration and certificates. Reload your site in your browser and it should switch you over to HTTPS automatically if you chose the redirect option.


Your site is now secure and it’s safe to continue with the web-based setup steps.


You can find more information on securing domains with Let’s Encrypt on the How to Secure Nginx with Let’s Encrypt on Ubuntu 20.04 tutorial.


# Step 5 — Configuring Gitea and Adding a First User


Now you can move on to configuring Gitea itself and creating the first admin user. Visit your Gitea instance by opening https://your_domain in a browser. On the initial Gitea configuration screen, there will be several options for the service:





Some of these such as the site title are up to your particular use case, though for the purposes of this tutorial, you’ll need to change the following:


- Server domain: the server domain that you set up in Step 3
- Gitea Base URL: the full URL that you’ll use to access Gitea in the browser, including the protocol. For example, https://your_domain.

When you save your configuration changes, you will be directed to the Gitea login page.



Note: Once the configuration has been saved, the Gitea service will restart. As this may take a few seconds, you may encounter an Nginx error stating 502 Bad Gateway. If you do encounter this error, wait a few seconds and restart the page.

As you do not yet have a user, you will need to create one, first. Click the Need an account? Register now link below the login form to register a new user. As the first user on the system, this user will be created as an administrator. If you set up email settings on the configuration screen, you may need to verify your account first.


Once you are logged in as that user, clicking on your user icon in the upper right-hand corner of the page and then clicking Site Administration from the drop-down menu will take you to a page where you will be able to run maintenance jobs, manage user accounts and organizations, and further configure Gitea.


## Creating a Test Repository


In order to test Gitea, both on the web interface and using Git itself, create a test repository. You can always delete this repository later.


Click on the + sign in the upper right corner of the page, and then click on + New Repository from the drop-down menu. Here, you will be presented with a screen allowing you to name and customize your repository with information such as its description, settings such as whether or not it’s private, and any default contents such as a README or .gitignore file.


Once you hit Create Repository, you will have a new repository to play around with.


# Step 6 — Configuring an SSH Shim


The final step of the process is to prepare the host machine with an SSH shim. Because Gitea is running in a Docker container, it cannot accept SSH connections on the default port of 22, as this will clash with the host. In the docker-compose.yml file you created above, Docker was instructed to map a port on the host to port 22 on the container so that it accepts SSH connections to port 2222. Additionally, the SSH authorized_keys file will not be accessible to someone SSHing into the host by default.


In order to take this into account, you will need to create an SSH shim which will pass SSH connections to the git user on the host on to the container. In the compose file, you also specified that the USER in the container will have a user and group ID of 1000, and on the Gitea configure screen, you told the service to use the user named git.


## Creating the Git user and its SSH key


Next, you will need to create an SSH key for the user. This will only be used in a step below and not shared with anyone outside the host.


```
sudo -u git ssh-keygen -t rsa -b 4096 -C "Gitea Host Key"


```


This command uses sudo to create an SSH key as the user you created above. In this instance, the key will be a 4096-bit RSA key. You will be asked a series of questions such as what password you would like for the key and what to name the key file. Hit ENTER for each of them, leaving them blank to accept the default.



Warning: If you set a password on the key, you will not be able to use the shim.

You will need to ensure that the user within the Gitea container will accept this key. You can do this by adding it to the .ssh/authorized_keys file:


```
sudo -u git cat /home/git/.ssh/id_rsa.pub | sudo -u git tee -a /home/git/.ssh/authorized_keys
sudo -u git chmod 600 /home/git/.ssh/authorized_keys


```


These commands all work with the shim due to the fact that the directory /home/git/.ssh on the host is mounted as a volume on the container, meaning that the contents are shared between them. When a connection to the host via git over SSH is received, it will use the same authorized_keys file as the container.


## Creating the SSH Shim Script


The final step for the shim is to create a stub gitea command on the host. This is what allows git commands to work over SSH: when an SSH connection is made, a default command will be run. This gitea command on the host is what will proxy the SSH connection to the container.


For this script, use cat to write to the file /usr/local/bin/gitea:


```
cat <<"EOF" | sudo tee /usr/local/bin/gitea
#!/bin/sh
ssh -p 2222 -o StrictHostKeyChecking=no git@127.0.0.1 "SSH_ORIGINAL_COMMAND=\"$SSH_ORIGINAL_COMMAND\" $0 $@"
EOF


```


The command in this script SSHes to the Gitea Docker container, passing on the contents of the original command used by git.


Finally, make sure that the script is executable:


```
sudo chmod +x /usr/local/bin/gitea


```


## Testing Git SSH Connections


You can test out pulling from and pushing to Git repositories on your Gitea instance by adding your SSH key to your Gitea user.


You will need the contents of your SSH public key. This usually lives in a file named something like ~/.ssh/id_rsa.pub, depending on which algorithm you used when creating your key:


```
cat ~/.ssh/id_rsa.pub


```



Note: If you need to create an SSH key for the first time, you can learn how to do so with this How to Set Up SSH Keys on Ubuntu 20.04 tutorial.

Copy the output of this command.


In Gitea, click on your user icon in the upper right-hand corner and select Settings. On the settings page, there will be a series of tabs on the top. Click on SSH/GPG Keys, then the Add Key button beside Manage SSH Keys. Paste your key into the large text area in the form and then click the Add Key button beneath it.


Now, navigate to the test repository you created in Step 3 and copy the SSH URL provided. On your local machine, clone the repository:


```
git clone git@your_domain:username/test


```


This will use SSH to clone the repository. If you have a password set on your SSH key, you will be asked to provide that.


Move to that directory, create a new file:


```
cd test
touch just_testing


```


Next, add it to your staged changes:


```
git add just_testing


```


Finally, commit that file:


```
git commit -am "Just testing pushing over SSH!"


```


Now, you should be able to push your changes to the remote repository:


```
git push origin master


```


When you refresh the page in your browser, your new file will appear in the repository.


# Conclusion


You’ve set up a Gitea service using Docker in order to self-host your source-code repositories. From here, you will be able to work on both public and private repositories, using familiar workflows such as pull-request code reviews and projects organized by organization. Gitea also works well with various continuous integration and deployment (CI/CD) tools such as Drone, Jenkins, and GoCD. Additionally, using Docker volumes such as this allows you to extend your storage to fit Git LFS (large file storage) content on network or block storage.


