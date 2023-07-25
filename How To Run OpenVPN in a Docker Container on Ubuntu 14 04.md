# How To Run OpenVPN in a Docker Container on Ubuntu 14 04

```Docker``` ```Ubuntu``` ```VPN```

# Introduction


This tutorial will explain how to set up and run an OpenVPN container with the help of Docker.


OpenVPN provides a way to create virtual private networks (VPNs) using TLS (evolution of SSL) encryption. OpenVPN protects the network traffic from eavesdropping and man-in-the-middle (MITM) attacks. The private network can be used to securely connect a device, such as a laptop or mobile phone running on an insecure WiFi network, to a remote server that then relays the traffic to the Internet. Private networks can also be used to securely connect devices to each other over the Internet.


Docker provides a way to encapsulate the OpenVPN server process and configuration data so that it is more easily managed. The Docker OpenVPN image is prebuilt and includes all of the necessary dependencies to run the server in a sane and stable environment. Scripts are included to significantly automate the standard use case, but still allow for full manual configuration if desired. A Docker volume container is used to hold the configuration and EasyRSA PKI certificate data as well.


Docker Registry is a central repository for both official and user developed Docker images. The image used in this tutorial is a user contributed image available at kylemanna/openvpn. The image is assembled on Docker Registry’s cloud build servers using the source from the GitHub project repository. The cloud server build linked to Github adds the ability to audit the Docker image so that users can review the source Dockerfile and related code, called a Trusted Build. When the code is updated in the GitHub repository, a new Docker image is built and published on the Docker Registry.


## Example Use Cases


- Securely route to the Internet when on an untrusted public (WiFi) networks
- Private network to connect a mobile laptop, office computer, home PC, or mobile phone
- Private network for secure services behind NAT routers that don’t have NAT traversal capabilities

## Goals


- Set up the Docker daemon on Ubuntu 14.04 LTS
- Set up a Docker volume container to hold the configuration data
- Generate a EasyRSA PKI certificate authority (CA)
- Extract auto-generated client configuration files
- Configure a select number of OpenVPN clients
- Handle starting the Docker container on boot
- Introduce advanced topics

## Prerequisites


- Linux shell knowledge. This guide largely assumes that the user is capable of setting up and running Linux daemons in the traditional sense
- Root access on a remote server

A DigitalOcean 1 CPU / 512 MB RAM Droplet running Ubuntu 14.04 is assumed for this tutorial. Docker makes running the image on any host Linux distribution easy
Any virtual host will work as long as the host is running QEMU/KVM or Xen virtualization technology; OpenVZ will not work
You will need root access on the server. This guide assumes the user is running as an unprivileged user with sudo enabled. Review the Digital Ocean tutorial about user management on Ubuntu 14.04 if needed


- A DigitalOcean 1 CPU / 512 MB RAM Droplet running Ubuntu 14.04 is assumed for this tutorial. Docker makes running the image on any host Linux distribution easy
- Any virtual host will work as long as the host is running QEMU/KVM or Xen virtualization technology; OpenVZ will not work
- You will need root access on the server. This guide assumes the user is running as an unprivileged user with sudo enabled. Review the Digital Ocean tutorial about user management on Ubuntu 14.04 if needed
- A local client device such as an Android phone, laptop, or PC. Almost all operating systems are supported via various OpenVPN clients

# Step 1 — Set Up and Test Docker


Docker is moving fast and Ubuntu’s long term support (LTS) policy doesn’t keep up. To work around this we’ll install a PPA that will get us the latest version of Docker.


Add the upstream Docker repository package signing key. The apt-key command uses elevated privileges via sudo, so a password prompt for the user’s password may appear:


```
curl -L https://get.docker.com/gpg | sudo apt-key add -

```


Note: Enter your sudo password at the blinking cursor if necessary.


Add the upstream Docker repository to the system list:


```
echo deb http://get.docker.io/ubuntu docker main | sudo tee /etc/apt/sources.list.d/docker.list

```


Update the package list and install the Docker package:


```
sudo apt-get update && sudo apt-get install -y lxc-docker

```


Add your user to the docker group to enable communication with the Docker daemon as a normal user, where sammy is your username. Exit and log in again for the new group to take effect:


```
sudo usermod -aG docker sammy

```


After re-logging in verify the group membership using the id command. The expected response should include docker like the following example:


```
uid=1001(test0) gid=1001(test0) groups=1001(test0),27(sudo),999(docker)

```


Optional: Run bash in a simple Debian Docker image (--rm to clean up container after exit and -it for interactive) to verify Docker operation on host:


```
docker run --rm -it debian:jessie bash -l

```


Expected response from docker as it pulls in the images and sets up the container:


```
Unable to find image 'debian:jessie' locally
debian:jessie: The image you are pulling has been verified
511136ea3c5a: Pull complete
36fd425d7d8a: Pull complete
aaabd2b41e22: Pull complete
Status: Downloaded newer image for debian:jessie
root@de8ffd8f82f6:/#

```


Once inside the container you’ll see the root@<container id>:/# prompt signifying that the current shell is in a Docker container. To confirm that it’s different from the host, check the version of Debian running in the container:


```
cat /etc/issue.net

```


Expected response for the OpenVPN container at the time of writing:


```
Debian GNU/Linux jessie/sid

```


If you see a different version of Debian, that’s fine.


Exit the container by typing logout, and the host’s prompt should appear again.


# Step 2 — Set Up the EasyRSA PKI Certificate Store


This step is usually a headache for those familiar with OpenVPN or any services utilizing PKI. Luckily, Docker and the scripts in the Docker image simplify this step by generating configuration files and all the necessary certificate files for us.


Create a volume container. This tutorial will use the $OVPN_DATA environmental variable to make it copy-paste friendly. Set this to anything you like. The default ovpn-data value is recommended for single OpenVPN Docker container servers. Setting the variable in the shell leverages string substitution to save the user from manually replacing it for each step in the tutorial:


```
OVPN_DATA="ovpn-data"

```


Create an empty Docker volume container using busybox as a minimal Docker image:


```
docker run --name $OVPN_DATA -v /etc/openvpn busybox

```


Initialize the $OVPN_DATA container that will hold the configuration files and certificates, and replace vpn.example.com with your FQDN. The vpn.example.com value should be the fully-qualified domain name you use to communicate with the server. This assumes the DNS settings are already configured. Alternatively, it’s possible to use just the IP address of the server, but this is not recommended.


```
docker run --volumes-from $OVPN_DATA --rm kylemanna/openvpn ovpn_genconfig -u udp://vpn.example.com:1194

```


Generate the EasyRSA PKI certificate authority. You will be prompted for a passphrase for the CA private key. Pick a good one and remember it; without the passphrase it will be impossible to issue and sign client certificates:


```
docker run --volumes-from $OVPN_DATA --rm -it kylemanna/openvpn ovpn_initpki

```


Note, the security of the $OVPN_DATA container is important.  It contains all the private keys to impersonate the server and all the client certificates. Keep this in mind and control access as appropriate. The default OpenVPN scripts use a passphrase for the CA key to increase security and prevent issuing bogus certificates.


See the Conclusion below for more details on how to back up the certificate store.


# Step 3 — Launch the OpenVPN Server


To autostart the Docker container that runs the OpenVPN server process (see Docker Host Integration for more) create an Upstart init file using nano or vim:


```
sudo vim /etc/init/docker-openvpn.conf

```


Contents to place in /etc/init/docker-openvpn.conf:


```
description "Docker container for OpenVPN server"
start on filesystem and started docker
stop on runlevel [!2345]
respawn
script
  exec docker run --volumes-from ovpn-data --rm -p 1194:1194/udp --cap-add=NET_ADMIN kylemanna/openvpn
end script

```


Start the process using the Upstart init mechanism:


```
sudo start docker-openvpn

```


Verify that the container started and didn’t immediately crash by looking at the STATUS column:


```
test0@tutorial0:~$ docker ps
CONTAINER ID        IMAGE                      COMMAND             CREATED             STATUS              PORTS                    NAMES
c3ca41324e1d        kylemanna/openvpn:latest   "ovpn_run"          2 seconds ago       Up 2 seconds        0.0.0.0:1194->1194/udp   focused_mestorf

```


# Step 4 — Generate Client Certificates and Config Files


In this section we’ll create a client certificate using the PKI CA we created in the last step.


Be sure to replace CLIENTNAME as appropriate (this doesn’t have to be a FQDN). The client name is used to identify the machine the OpenVPN client is running on (e.g., “home-laptop”, “work-laptop”, “nexus5”, etc.).


The easyrsa tool will prompt for the CA password. This is the password we set above during the ovpn_initpki command. Create the client certificate:


```
docker run --volumes-from $OVPN_DATA --rm -it kylemanna/openvpn easyrsa build-client-full CLIENTNAME nopass

```


After each client is created, the server is ready to accept connections.


The clients need the certificates and a configuration file to connect. The embedded scripts automate this task and enable the user to write out a configuration to a single file that can then be transfered to the client. Again, replace CLIENTNAME as appropriate:


```
docker run --volumes-from $OVPN_DATA --rm kylemanna/openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn

```


The resulting CLIENTNAME.ovpn file contains the private keys and certificates necessary to connect to the VPN. Keep these files secure and not lying around. You’ll need to securely transport the *.ovpn files to the clients that will use them. Avoid using public services like email or cloud storage if possible when transferring the files due to security concerns.


Recommend methods of transfer are ssh/scp, HTTPS, USB, and microSD cards where available.


# Step 5 — Set Up OpenVPN Clients


The following are commands or operations run on the clients that will connect to the OpenVPN server configured above.


## Ubuntu and Debian Distributions via Native OpenVPN


On Ubuntu 12.04/14.04 and Debian wheezy/jessie clients (and similar):


Install OpenVPN:


```
sudo apt-get install openvpn

```


Copy the client configuration file from the server and set secure permissions:


```
sudo install -o root -m 400 CLIENTNAME.ovpn /etc/openvpn/CLIENTNAME.conf

```


Configure the init scripts to autostart all configurations matching /etc/openvpn/*.conf:


```
echo AUTOSTART=all | sudo tee -a /etc/default/openvpn

```


Restart the OpenVPN client’s server process:


```
sudo /etc/init.d/openvpn restart

```


## Arch Linux via Native OpenVPN


Install OpenVPN:


```
pacman -Sy openvpn

```


Copy the client configuration file from the server and set secure permissions:


```
sudo install -o root -m 400 CLIENTNAME.ovpn /etc/openvpn/CLIENTNAME.conf

```


Start OpenVPN client’s server process:


```
systemctl start openvpn@CLIENTNAME

```


Optional: configure systemd to start /etc/openvpn/CLIENTNAME.conf at boot:


```
systemctl enable openvpn@CLIENTNAME

```


## MacOS X via TunnelBlick


Download and install TunnelBlick.


Copy CLIENTNAME.ovpn from the server to the Mac.


Import the configuration by double clicking the *.ovpn file copied earlier. TunnelBlick will be invoked and the import the configuration.


Open TunnelBlick, select the configuration, and then select connect.


## Android via OpenVPN Connect


Install the OpenVPN Connect App from the Google Play store.


Copy CLIENTNAME.ovpn from the server to the Android device in a secure manner. USB or microSD cards are safer. Place the file on your SD card to aid in opening it.


Import the configuration: Menu -> Import -> Import Profile from SD card


Select connect.


# Step 6 — Verify Operation


There are a few ways to verify that traffic is being routed through the VPN.


## Web Browser


Visit a website to determine the external IP address. The external IP address should be that of the OpenVPN server.


Try Google “what is my ip” or icanhazip.com.


## Command Line


From the command line, wget or curl come in handy. Example with curl:


```
curl icanhazip.com

```


Example with wget:


```
wget -qO - icanhazip.com

```


The expected response should be the IP address of the OpenVPN server.


Another option is to do a special DNS lookup to a specially configured DNS server just for this purpose using host or dig. Example using host:


```
host -t A myip.opendns.com resolver1.opendns.com

```


Example with dig:


```
dig +short myip.opendns.com @resolver1.opendns.com

```


The expected response should be the IP address of the OpenVPN server.


## Extra Things to Check


Review your network interface configuration. On Unix-based operating systems, this is as simple as running ifconfig in a terminal, and looking for OpenVPN’s tunX interface when it’s connected.


Review logs. On Unix systems check /var/log on old distributions or journalctl on systemd distributions.


# Conclusion


The Docker image built to run this is open source and capable of much more than described here.


The docker-openvpn source repository is available for review of the code as well as forking for modifications. Pull requests for general features or bug fixes are welcome.


Advanced topics such as backup and static client IPs are discussed under the docker-openvpn/docs folder.


Report bugs to the docker-openvpn issue tracker.


