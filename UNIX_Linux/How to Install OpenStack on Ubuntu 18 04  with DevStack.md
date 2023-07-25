# How to Install OpenStack on Ubuntu 18 04  with DevStack

```Ubuntu``` ```UNIX/Linux```

Openstack is a free and opensource IaaS cloud platform that handles cloud compute, storage and network resources. It comes with an intuitive dashboard that enables systems administrators to provide and monitor these resources. You can seamlessly install OpenStack locally on your Ubuntu 18.04 instance for learning and testing purposes using Devstack. Devstack is a set of extensible scripts that facilitate OpenStack deployment. In this guide, you will learn how to deploy OpenStack on Ubuntu 18.04 with devstack.


# Minimum Requirements


Before we begin, ensure you have the following minimum prerequisites


1. A fresh Ubuntu 18.04 installation
2. User with sudo privileges
3. 4 GB RAM
4. 2 vCPUs
5. Hard disk capacity of 10 GB
6. Internet connection

With the minimum requirements satisfied, we can now proceed.


# Step 1: Update and Upgrade the System


To start off, log into your Ubuntu 18.04 system using SSH protocol and update & upgrade system repositories using the following command.


```
apt update -y && apt upgrade -y

```


Sample Output  Next reboot the system using the command.


```
sudo reboot

```


OR


```
init 6

```


# Step 2: Create Stack user and assign sudo priviledge


Best practice demands that devstack should be run as a regular user with sudo privileges. With that in mind, we are going to add a new user called “stack” and assign sudo privileges. To create stack user execute


```
sudo adduser -s /bin/bash -d /opt/stack -m stack

```


Next, run the command below to assign sudo privileges to the user


```
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack

```


Sample Output 


# Step 3: Install git and download DevStack


Once you have successfully created the user ‘stack’ and assigned sudo privileges, switch to the user using the command.


```
su - stack

```


In most Ubuntu 18.04 systems, git comes already installed. If by any chance git is missing, install it by running the following command.


```
sudo apt install git -y

```


Sample output  Using git, clone devstack’s git repository as shown.


```
git clone https://git.openstack.org/openstack-dev/devstack

```


Sample output 


# Step 4: Create devstack configuration file


In this step, navigate to the devstack directory.


```
cd devstack

```


Then create a local.conf configuration file.


```
vim local.conf

```


Paste the following content


```
[[local|localrc]]

# Password for KeyStone, Database, RabbitMQ and Service
ADMIN_PASSWORD=StrongAdminSecret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

# Host IP - get your Server/VM IP address from ip addr command
HOST_IP=10.208.0.10


```


Save and exit the text editor. NOTE:


1. The ADMIN_PASSWORD is the password that you will use to log in to the OpenStack login page. The default username is admin.
2. The HOST_IP is your system’s IP address that is obtained by running ifconfig or ip addr commands.

# Step 5: Install OpenStack with Devstack


To commence the installation of OpenStack on Ubuntu 18.04, run the script below contained in devstack directory.


```
./stack.sh

```


The following features will be installed:


- Horizon – OpenStack Dashboard
- Nova – Compute Service
- Glance – Image Service
- Neutron – Network Service
- Keystone – Identity Service
- Cinder – Block Storage Service
- Placement – Placement API

The deployment takes about 10 to 15 minutes depending on the speed of your system and internet connection. In our case, it took roughly 12 minutes. At the very end, you should see output similar to what we have below.  This confirms that all went well and that we can proceed to access OpenStack via a web browser.


# Step 6: Accessing OpenStack on a web browser


To access OpenStack via a web browser browse your Ubuntu’s IP address as shown. https://server-ip/dashboard This directs you to a login page as shown.  Enter the credentials and hit “Sign In” You should be able to see the Management console dashboard as shown below.  For more on Devstack’s customization, check out their system configuration guide. Additionally, check out the Openstack documentation for administration guide.


