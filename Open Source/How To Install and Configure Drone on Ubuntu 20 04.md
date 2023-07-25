# How To Install and Configure Drone on Ubuntu 20 04

```Docker``` ```CI/CD``` ```Ubuntu 20.04``` ```Open Source``` ```Git```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Drone is an open-source container-native CI/CD platform written in Go. It works with configuration files written in YAML, JSON, JSONNet, or Starlark, which define multiple build pipelines consisting of a number of steps.


Drone integrates with multiple source code managers. Currently, three different SCMs are supported: GitHub (cloud/enterprise), BitBucket (cloud/server), and Gitea. In general, each provider supports all Drone functionality.


Drone also supports different runners for executing jobs. These runners are not interchangeable (except with the simplest pipelines), because their configuration formats, features, and execution environments differ. Here is a brief summary of your options:


- First, there are the two container-based runners:

The Docker Runner is the stablest and most commonly used runner. It executes each build step in a new container.
The Kubernetes runner is a newer (but still experimental) option. It has a similar syntax to the Docker runner and could integrate with DigitalOcean Kubernetes.


- The Docker Runner is the stablest and most commonly used runner. It executes each build step in a new container.
- The Kubernetes runner is a newer (but still experimental) option. It has a similar syntax to the Docker runner and could integrate with DigitalOcean Kubernetes.
- Second, there are two runners that run commands directly without isolation between builds or repositories:

The Exec runner executes commands directly on the host. This runner does not provide isolation between builds, and should only be used with trusted repositories that are not suitable for running within Docker (note that you may want to consider using privileged Docker containers before resorting to this option).
The SSH runner is similar to Exec, but it runs commands over SSH on a host separate from the one running the runner.


- The Exec runner executes commands directly on the host. This runner does not provide isolation between builds, and should only be used with trusted repositories that are not suitable for running within Docker (note that you may want to consider using privileged Docker containers before resorting to this option).
- The SSH runner is similar to Exec, but it runs commands over SSH on a host separate from the one running the runner.
- Third, there are two runners that execute build steps in a cloud-based virtual machine. These are suitable for workloads that need to run directly on a host, but are still isolated from each other:

The DigitalOcean runner creates a new Droplet for each build and runs commands within it. Note that if you only need the scaling features, you can still use the Docker runner but with the autoscaler enabled (more on this later).
The MacStadium runner allows you to execute builds on macOS machines in the cloud. This option is only useful for specialized workloads.


- The DigitalOcean runner creates a new Droplet for each build and runs commands within it. Note that if you only need the scaling features, you can still use the Docker runner but with the autoscaler enabled (more on this later).
- The MacStadium runner allows you to execute builds on macOS machines in the cloud. This option is only useful for specialized workloads.

In this tutorial, you will set up a Drone CI/CD server for source code on GitHub, add a Docker runner, use Let’s Encrypt to secure your instance, and then create a YAML pipeline. You will also encounter options to scale your runner using Drone Autoscaler and to store your logs on an S3-compatible server, such as DigitalOcean Spaces.


# Prerequisites


Before you start this tutorial, you will need:


- One Ubuntu 20.04 server with at least 1GB of RAM, 2GB of free disk space, and a non-root user with sudo priveliges. You can follow our initial server setup to configure your machine.


Note: This tutorial will optionally configure Drone to use its Autoscaler feature with DigitalOcean, which will automatically scale your number of Droplets on an as-needed basis. If you choose this route, ensure that your account will remain within your limit. On DigitalOcean, the default limit for most users is 5 Droplets, but you can contact support and request an increase. To do so, visit your account’s Cloud Dashboard and find ACCOUNT in the left-hand menu. A sub-menu will appear; click Settings. A page will open that displays your account username and your Member since date. Beneath this date you will see a line like Droplet Limit:5 Increase. Click Increase to submit a request for more Droplets.
Moreover, the Autoscaler path described in this tutorial will require a Personal Access Token from DigitalOcean. If you do choose to install Autoscaler, you can follow this tutorial to retrieve a token from your DigitalOcean Control Panel. Copy this key down before leaving the page; it will disappear once you leave or refresh the page and you will need to input it during Step 6.
Lastly, if you choose not to install Drone’s Autoscaler feature, you will need at least another 2GB of RAM and 10GB of free disk space to ensure that you can run pipelines.

- A domain (or subdomain) with an available A record pointed at your server’s IP. If you are managing your DNS on DigitalOcean then you can follow this guide to associate your IP with your domain. This tutorial will use drone.your_domain.
- Docker set up on your server. For instructions, you can follow this tutorial on how to install and use Docker on Ubuntu 20.04.
- You will encounter an option to store your logs on DigitalOcean Spaces. To follow this step, you will need to add a DigitalOcean Space to your account, and you will also need to generate a Spaces Access Key. Copy down your Access Key and Secret somewhere safe. Your Secret will disappear once you leave or refresh this page, and you will need to input both values wherever you see your_s3_access_key and your_s3_secret_key. Alternately, you can use a different S3-compatible service, or skip this step—Step 3—entirely. Note that skipping this step is only advisable if you are trying out Drone or if you know that your build volume will be quite low.
- A GitHub account.

# Step 1 — Creating the GitHub Application


To access code, authenticate users, and add webhooks to receive events, Drone requires an OAuth application for GitHub. For other providers, you can read Drone’s official documentation here.


To set up an OAuth application for GitHub, log in to your GitHub account and then click on your user menu in the top-right. Click Settings, then find the Developer Settings category in the menu on the left, and then click OAuth Applications. Alternatively, you can navigate directly to Github’s Developer Settings page.


Next, create a new application. Click on the New OAuth App button in the upper-right corner and a blank form will appear.





Use Drone for your application name. Replace drone.your_domain with your own domain, add a brief explanation of your app, and then add drone.your_domain/login for your Authorization callback URL.


Click Register application and you will see a dashboard containing information about your application. Included here are your app’s Client ID and Client Secret. Copy these two values somewhere safe; you will need to use them in the following steps wherever you see your_github_client_id and your_github_client_secret.


With your app now registered on GitHub, you are ready to configure Drone.


# Step 2 — Creating the Drone Configuration


Now start preparing your Docker configurations, which will build your Drone server. First, generate a shared secret to authenticate runners with the main Drone instance. Create one using the openssl command:


```
openssl rand -hex 16


```


openssl will generate a random 16-bit hexadecimal number. It will produce an output like this:


```
Output918...46c74b143a1719594d010ad24

```


Copy your own output to your clipboard. You will add it into the next command, where it will replace your_rpc_secret.


Now create your Drone configuration file. Rather than continually opening and closing this configuration file, we will leverage the tee command, which will split your command’s output to your console while also appending it to Drone’s configuration file. An explanation will follow every command block in this tutorial, but you can find a detailed description of all available Drone options in their official documentation.


Now begin building your Drone server’s configuration. Copy the following command to your terminal. Be sure to replace drone.your_domain with your domain. Also replace your_github_client_id and your_github_client_secret with your GitHub OAuth credentials, and then replace your_rpc_secret with the output from your openssl command. Lastly, replace sammy_the_shark with your GitHub username. This will grant you administrative privileges:


```
cat << 'EOF' | sudo tee /etc/drone
DRONE_SERVER_HOST=drone.your_domain
DRONE_SERVER_PROTO=https
DRONE_GITHUB_CLIENT_ID=your_github_client_id
DRONE_GITHUB_CLIENT_SECRET=your_github_client_secret
DRONE_RPC_SECRET=your_rpc_secret
DRONE_USER_CREATE=username:sammy_the_shark,admin:true
EOF


```


This command makes use of a heredoc. A heredoc uses the << redirection operator followed by an arbitrary word, where EOF is conventionally used to represent end-of-file. It allows the user to write a multiline input, ending with the EOF or whatever word the user has chosen. The quotes around the end-of-file affect how the text is parsed with regards to variable substitution, in a similar way to how they work around literals. It is a very useful tool, which in this case you are using to create a file and then add lines to it. Here you are adding your first Drone configurations and ending them with EOF. This input is then redirected to the cat command, and the output of the cat command is then piped to the tee command via the | pipe operator. Heredocs are a great way to quickly create or append text to a file.


Next, in order to prevent arbitrary users from logging in to your Drone server and having access to your runners, limit registration to specified usernames or organizations. If you need to add users at this time, run the following command, replacing users with a comma-separated list of GitHub usernames or organization names:


```
echo 'DRONE_USER_FILTER=users' | sudo tee -a /etc/drone


```


If you are not using an external load balancer or SSL proxy, you will also need to enable Let’s Encrypt for HTTPS:


```
echo 'DRONE_TLS_AUTOCERT=true' | sudo tee -a /etc/drone


```


You will note that your tee command now includes the -a switch, which instructs tee to append, and not overwrite, this output to your Drone configuration file. Let’s now set up your log storage system.


# Step 3 — Storing Build Logs Externally (Optional)


For heavily used installations, the volume of build logs can increase quite quickly to multiple gigabytes. By default, these logs are stored in the server’s database, but for performance, scalability, and stability, consider setting up external storage for your build logs. In this step, you’ll use DigitalOcean Spaces to do just that. You are welcome to modify these steps and use another S3-compatible storage service, or none at all if you are still prototyping your CI/CD workflow, or if you know that your build volume will be very low. In those cases you may continue to Step 4 now.


To store your logs on DigitalOcean Spaces, make sure that you have completed the necessary prerequisites and have set up a Spaces bucket and generated a matching Spaces Access Key and Secret. Copy that key to your clipboard and then update your configuration file with the following command:


```
cat << 'EOF' | sudo tee -a /etc/drone
DRONE_S3_ENDPOINT=your_s3_endpoint
DRONE_S3_BUCKET=your_s3_bucket_name
AWS_ACCESS_KEY_ID=your_s3_access_key
AWS_SECRET_ACCESS_KEY=your_s3_secret_key
EOF


```


Remember to replace your_s3_endpoint with the URL for your Space, your_s3_bucket_name with the name of the Space you created, your_s3_access_key with your access key, and your_s3_secret_key with your secret. You can find the first two values in your Control Panel by clicking the Manage menu button, then clicking Spaces, and then choosing your new Space. You can retrieve your Spaces Access Key by clicking the Account menu button, and then clicking the API button, and then scrolling down until you find the Spaces section. If you have misplaced your secret key, then you will need to generate a new access key/secret pair.


Your Drone configuration file is now complete. Run a cat command to view it:


```
cat /etc/drone


```


Your configuration file will look something like the following, depending on the options you chose:


```
OutputDRONE_SERVER_HOST=drone.your_domain
DRONE_SERVER_PROTO=https
DRONE_GITHUB_CLIENT_ID=your_github_client_id
DRONE_GITHUB_CLIENT_SECRET=your_github_client_secret
DRONE_RPC_SECRET=your_rpc_secret
DRONE_USER_CREATE=username:sammy_the_shark,admin:true
DRONE_USER_FILTER=the_shark_org
DRONE_TLS_AUTOCERT=true
DRONE_S3_ENDPOINT=your_s3_endpoint
DRONE_S3_BUCKET=your_s3_bucket
AWS_ACCESS_KEY_ID=your_s3_access_key
AWS_SECRET_ACCESS_KEY=your_s3_secret_key

```


Once you have confirmed that your configuration file is complete, you can start your Drone server.


# Step 4 — Installing and Starting Drone


With your proper configurations in place, your next step is to install and start Drone.


First, pull the Drone Server Docker image:


```
docker pull drone/drone:1


```


Next, create a volume to store the SQLite database:


```
docker volume create drone-data


```


Finally, start the server, set it to restart on boot, and forward port 80 and 443 to it:


```
docker run --name=drone --detach --restart=always --env-file=/etc/drone --volume=drone-data --publish=80:80 --publish=443:443 drone/drone:1


```


If you followed the DigitalOcean Initial Server Setup Guide, then you will have enabled ufw and only allowed OpenSSH through your firewall. You will now need to open ports 80 and 443:


```
sudo ufw allow 80
sudo ufw allow 443


```


Now reload ufw and check that your rules updated:


```
sudo ufw reload
sudo ufw status


```


You will see an output like this:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
80                         ALLOW       Anywhere
443                        ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
80 (v6)                    ALLOW       Anywhere (v6)
443 (v6)                   ALLOW       Anywhere (v6)

```


At this point, you will be able to access your server, log in, and manage your repositories. Go to https://drone.your_domain, enter your GitHub credentials, and authorize your new application when prompted.





Your Drone server is now live and nearly ready for use. Configuring your Drone runner and/or the DigitalOcean Autoscaler are all that remains.


# Step 5 — Option 1: Installing the Docker Runner


Before your server can execute jobs, you need to set up a runner.


If you want to automatically scale the runner with DigitalOcean Droplets, skip down to Option 2: Installing the Drone Autoscaler for DigitalOcean. If you want to use another runner, you can set that up instead and skip to Step 7. Otherwise, follow Option 1 and install the Docker runner.


## Option 1: Installing the Docker Runner


First, pull the Docker image for the runner:


```
docker pull drone/drone-runner-docker:1


```


Next, start the runner. Replace drone.your_domain and your_rpc_secret with your personal values. You can change the DRONE_RUNNER_CAPACITY to increase the number of pipelines that will be executed at once, but be mindful of your available system resources:


```
docker run --name drone-runner --detach --restart=always --volume=/var/run/docker.sock:/var/run/docker.sock -e DRONE_RPC_PROTO=https -e DRONE_RPC_HOST=drone.your_domain -e DRONE_RPC_SECRET=your_rpc_secret -e DRONE_RUNNER_CAPACITY=1 -e DRONE_RUNNER_NAME=${HOSTNAME} drone/drone-runner-docker:1


```


Finally, ensure that the runner started successfully:


```
docker logs drone-runner


```


You will see an output like this:


```
Outputtime="2020-06-13T17:58:33-04:00" level=info msg="starting the server" addr=":3000"
time="2020-06-13T17:58:33-04:00" level=info msg="successfully pinged the remote server"
time="2020-06-13T17:58:33-04:00" level=info msg="polling the remote server" arch=amd64 capacity=1 endpoint="https://drone.your_domain" kind=pipeline os=linux type=docker

```


If you ever need to change the runner configuration or secret, delete your container with docker rm drone-runner, and repeat this step. You can now proceed to Step 7 and create a basic pipeline.


## Option 2: Installing the Drone Autoscaler for DigitalOcean


The Drone Autoscaler for DigitalOcean can automatically create and destroy Droplets with the Docker runner as needed.


First, go to your Drone server, log in, and click on User settings in the user menu. Find and copy your Personal Token. This is your_drone_personal_token.





Next, generate a new character string with the following command:


```
openssl rand -hex 16


```


```
Outpute5cd27400...92b684526c622

```


Copy the output like you did in Step 2. This new output is your drone_user_token.


Now add a new machine user with these new credentials:


```
docker run --rm -it -e DRONE_SERVER=https://drone.your_domain -e DRONE_TOKEN=your_drone_personal_token --rm drone/cli:1 user add autoscaler --machine --admin --token=drone_user_token


```


Now, if you haven’t already, you’ll need to create a DigitalOcean API Token with read/write privileges. We will refer to this as your_do_token. If you did not complete this step in the prerequisites section then you can use this guide to create one now. Keep this token very safe; it grants full access to all resources on your account.


Finally, you can start the Drone Autoscaler. Make sure to replace all the highlighted variables with your own matching credentials:


```
docker volume create drone-autoscaler-data
docker run --name=drone-autoscaler --detach --restart=always --volume=drone-autoscaler-data -e DRONE_SERVER_PROTO=https -e DRONE_SERVER_HOST=drone.your_domain -e DRONE_SERVER_TOKEN=drone_user_token -e DRONE_AGENT_TOKEN=your_rpc_secret -e DRONE_POOL_MIN=0 -e DRONE_POOL_MAX=2 -e DRONE_DIGITALOCEAN_TOKEN=your_do_token -e DRONE_DIGITALOCEAN_REGION=nyc1 -e DRONE_DIGITALOCEAN_SIZE=s-2vcpu-4gb -e DRONE_DIGITALOCEAN_TAGS=drone-autoscaler,drone-agent drone/autoscaler


```


You can also configure the minimum/maximum number of Droplets to create and the type/region for the Droplet. For faster build start times, set the minimum to 1 or more. Also, note that by default, the autoscaler will determine if new Droplets need to be created or destroyed every minute, and Droplets will be left running for at least 1 hour after creation before being automatically destroyed when inactive.


Afterwards, verify that the autoscaler started correctly with:


```
docker logs drone-autoscaler


```


If you decide you no longer want to use the autoscaler, delete the container with docker rm drone-autoscaler, delete the leftover Droplets (if any) from your account, and revoke the DigitalOcean API Token. You are now prepared to test your new CI/CD workflow.


# Step 6 — Creating a YAML Pipeline


To test your new Drone installation, let’s create a YAML pipeline.


First, create a new repository on GitHub. From your GitHub profile page click on the Repositories menu, then click the green New button at the upper right. Give your repository a name on the following page and then click the green Create repository button. Now navigate to your Drone server, press SYNC, refresh the page, and your newly created repository should appear. Press the ACTIVATE button beside it.





Afterwards, create a new file in your repo named .drone.yml. You can do this using GitHub’s UI or from the command line using git. From the GitHub UI, click on the Repositories menu, then click on your new repository, and then click the Add file dropdown menu. Choose Create new file, name the file .drone.yaml, and add the following contents:


.drone.yml
```
name: drone-test
kind: pipeline
type: docker

steps:
- name: test
  image: alpine
  commands:
  - echo "It worked!"

```


If you are using the GitHub UI, press the green Commit new file button at the bottom of the page. If you are using the command line then commit and push your changes. In either case, now open and watch your Drone dashboard in a browser.


If the build remains pending and doesn’t start, ensure that your runners are set up correctly (and that a Droplet was created if using the autoscaler). You can view the logs for the runner with docker logs drone-runner and the logs for the autoscaler with docker logs drone-autoscaler.


If you are using the autoscaler, it may take up to a minute for the initial build to start (the last log message during that time would be starting the server).


After the build completes, you will see the text It worked! in the logs for the test stage of the drone-test pipeline. If the logs fail to load, ensure that your S3 credentials and bucket name are correct. You can use docker logs drone to view Drone’s logs for more information.





You’ve now set up and installed a Drone server to handle your CI/CD workflow.


# Conclusion


In this tutorial you set up the Drone CI/CD server for use with your GitHub projects and optionally set up external storage for build logs. You also set up a local runner or a service to automatically scale them using DigitalOcean Droplets.


You can continue adding teammates and other authorized users to Drone using the process outlined in Step 2. Should you ever want to prevent any new users from signing up, run the following commands:


```
echo 'DRONE_REGISTRATION_CLOSED=true' | sudo tee -a /etc/drone
docker restart drone


```


Drone is a very capable tool. From here, you might consider learning more about their pipeline syntax and Drone’s other functionalities, or reviewing the fundamentals of Docker.


