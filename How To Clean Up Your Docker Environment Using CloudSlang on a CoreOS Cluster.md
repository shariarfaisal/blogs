# How To Clean Up Your Docker Environment Using CloudSlang on a CoreOS Cluster

```Docker``` ```Ubuntu``` ```CoreOS```

## Introduction


CoreOS is a Linux distribution focused on quickly spinning up clustered environments by utilizing Docker containers and service discovery. If you’re new to CoreOS, check out this Getting Started with CoreOS tutorial series.


However, Docker images can occupy quite a lot of disk space on the Docker host. A base image can be hundreds of MB in size and custom images can easily reach 1 GB. If you have many releases of new Docker images for your app, they can easily stockpile on the server storage; the server can run out of disk space if you don’t clear out old or unused images from time to time.


CloudSlang is an open source orchestration solution that makes it easy to automate processes using workflows, or flows for short. A flow contains a list of tasks and navigation logic. A task can call an operation, which contains an action that runs a Python script or Java method, or another flow. The CloudSlang language allows you to define flows in a textual, reusable manner, and you can either use the existing content (Docker, OpenStack, and utilities) to manage your deployed applications or create your own custom flows.


In this tutorial, we will clean up the Docker environment for each machine deployed in a CoreOS cluster using CloudSlang. We will use the already existing content, so you will not need to edit any CloudSlang files.


# Prerequisites


Before you begin, you will need:


- 
A Ubuntu 14.04 Droplet with a sudo non-root user, which will be your CloudSlang server.

- 
Java (version 7 or later) installed on the CloudSlang server. Note that you don’t need to install the JDK, only the JRE.

- 
A cluster of three CoreOS machines. If you don’t already have one, you can set one up by following this tutorial.


# Step 1 — Installing unzip


In this step, we will install unzip on the CloudSlang server.


First, make sure the package list is up to date.


```
sudo apt-get update


```


Then, install unzip.


```
sudo apt-get install unzip


```


# Step 2 — Downloading CloudSlang


In this section, we will download the CloudSlang CLI tool and the available content (predefined operations and flows). The CloudSlang CLI is a command line interface tool that can be used to run flows.


First, download the CloudSlang CLI archive.


```
wget https://github.com/CloudSlang/cloud-slang/releases/download/cloudslang-0.7.29/cslang-cli-with-content.zip


```


Unzip the archive.


```
unzip cslang-cli-with-content.zip


```


This will create a cslang directory. If you list the contents of that directory,


```
ls ~/cslang


```


You will notice three directories in it:


- 
python-lib, which is used for external Python libraries.

- 
cslang, which contains the CloudSlang CLI files. The cslang/bin folder contains a file named cslang which is used to start up the CLI. The cslang/lib contains the necessary dependencies for the application.

- 
content, which contains the ready-made CloudSlang content. The flow we are going to run is located at content/io/cloudslang/coreos and it is called cluster_docker_images_maintenance.sl. This flow iterates over all the machines in the cluster and deletes unused Docker images.


# Step 3 — Adding the Private Key


CloudSlang needs SSH key access to your CoreOS cluster. In this step, we will add this by creating a new key pair on the CloudSlang server and adding the public key to the CoreOS cluster.


First, create a key pair without a passphrase by following steps 1 and 2 of this tutorial. After you have a key pair, you’ll need to add your public key to each of the machines in your CoreOS cluster.


First, get the public key on your CloudSlang server.


```
cat ~/.ssh/id_rsa.pub

```


You’ll see a long output which begins with ssh-rsa and ends with username@hostname. Copy this to use in the next command.


SSH into one of your CoreOS servers (the default username is core), then run the following command to add your public key.


```
echo "your_public_key" >> ~/.ssh/authorized_keys

```


You’ll need to do this for each server in your CoreOS cluster.


# Step 4 — Running the Flow


In this section we will run the flow and verify its behavior.


In order to run the flow, on the CloudSlang server, first change to the /cslang/bin directory.


```
cd ~/cslang/cslang/bin/


```


Run the executable called cslang in order to start up the CLI.


```
./cslang


```


After a moment, you’ll see the CloudSlang welcome screen.


```
0.7.26-SNAPSHOT
Welcome to CloudSlang. For assistance type help.

```


Enter the following command in the CLI, replacing your_coreos_server_ip with the IP address of one of the CoreOS servers in your cluster.


```
custom_prefix(cslang>)
run --f  ../../content/io/cloudslang/coreos/cluster_docker_images_maintenance.sl --i coreos_host=your_coreos_server_ip,coreos_username=core,private_key_file=~/.ssh/id_rsa --cp ../../content/

```


The run command triggers the flow. --f specifies the path to the flow. --i specifies the flow inputs: a CoreOS host and username, and the associated private SSH key. --cp specifies the classpath when the flow depends upon other operations and flows. Because this flow has many different dependencies, we can specify the parent content folder; the scanning is recursive so subdirectories are also scanned.


The flow logic first retrieves the IP addresses of the machines from the cluster, then iterates over the machines and clear unused images. First, it gets all the images, leaving only unused ones by checking running/stopped containers. Next, it deletes the unused images. Finally, it does the same for dangling images.


While the flow is running, the CLI displays the task names that are executed. Once the flow finishes, the CLI outputs some useful information like the flow outputs and the flow result.


In our case, the flow result will either be SUCCESS (which means unused Docker images were cleared in the cluster) or FAILURE (which means a problem occurred).  This flow has one output: number_of_deleted_images_per_host, which is how many images were deleted on every host in the cluster.


If everything went well you should see an output similar to this:


```
...

Flow : cluster_docker_images_maintenance finished with result : SUCCESS
Execution id: 101600001, duration: 0:02:06.180

```


If you want more information on the execution, look in the execution.log file which is created by the CLI in the bin folder.


## Conclusion


Now all the unused Docker images are deleted in your CoreOS cluster!


In this tutorial, you have seen how to run CloudSlang on your Ubuntu machine and how to use the CloudSlang CLI to trigger flows. You also used a ready-made workflow to clean up your Docker environment.


Copyright June 9, 2015, Hewlett-Packard Development Company, L.P.  Reproduced with Permission.


