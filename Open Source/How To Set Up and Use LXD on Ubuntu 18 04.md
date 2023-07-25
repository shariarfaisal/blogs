# How To Set Up and Use LXD on Ubuntu 18 04

```Linux Basics``` ```Ubuntu``` ```Container``` ```Open Source``` ```Nginx```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


A Linux container is a set of processes that is separated from the rest of the system. To the end-user, a Linux container functions as a virtual machine, but it’s much more light-weight. You don’t have the overhead of running an additional Linux kernel, and the containers don’t require any CPU hardware virtualization support. This means you can create more containers than virtual machines on the same server.


Imagine that you have a server that should run multiple web sites for your customers. On the one hand, each web site could be a virtual host/server block of the same instance of the Apache or Nginx web server. On the other hand, when using virtual machines, you would create a separate nested virtual machine for each website. Linux containers sit somewhere between virtual hosts and virtual machines.


LXD lets you create and manage these containers. LXD provides a hypervisor service to manage the entire life cycle of containers. In this tutorial, you’ll configure LXD and use it to run Nginx in a container. You’ll then route traffic from the internet to the container to make a sample web page accessible.


# Prerequisites


To complete this tutorial, you’ll need the following:


- 
A server running Ubuntu 18.04. To set up a server, including a non-root sudo user and a firewall, you can create a DigitalOcean Droplet running Ubuntu 18.04 and then follow our Initial Server Setup Guide. Note your server’s public IP address. We will refer to it later as your_server_ip.

- 
At least 5GB of block storage. To set this up, you can follow DigitalOcean’s Block Storage Volumes Quickstart. In the configuration of the Block Storage, select Manually Format & Mount in order to allow LXD to prepare it as required. You will use this to store all data related to the containers.



Note: LXD is pre-installed in Ubuntu 18.04, and the installed LXD package is a deb package. But beginning with Ubuntu 20.04, newer versions of LXD are now only available as snap packages.
Therefore, Ubuntu 18.04 is the last Ubuntu version that has LXD as a deb package. This LXD deb package has standard support until 2023 and End of Life in 2028. See the table below to help you decide on the package format.



Feature
deb package
snap package




available LXD versions
3.0
2.0, 3.0, 4.0, 4.x


memory requirements
minimal
moderate, for snapd service


upgrade considerations
you can decide not to upgrade LXD
can defer LXD upgrade up to 60 days


ability to switch from the other package format
not supported
can switch from deb to snap



Follow the rest of this tutorial to use LXD from the deb package in Ubuntu 18.04. If, however, you want to use the LXD snap package in Ubuntu 18.04, see [TODO-TUTORIAL-FOR-LXD-IN-UBUNTU-20.04].

# Step 1 — Configuring LXD


LXD is available as a deb package in Ubuntu 18.04. It comes pre-installed, but you must configure it before you can use it. LXD is composed of the LXD service and the default client utility that helps you configure the service. This client utility is lxc. The client utility can access the LXD service if you either run it as root, or if your non-root account is a member of the lxd Unix group. In the following, we show how to add your non-root user account to the lxd Unix group and then continue with the configuration of the storage backend.


## Adding your non-root account to the lxd Unix group


When setting up your non-root account, add them to the lxd group using the following command. The adduser command takes as arguments the user account and the Unix group in order to add the user account into the existing Unix group:


```
sudo adduser sammy lxd


```


Now apply the new membership:


```
su sammy


```


Enter your password and press ENTER.


Finally, confirm that your user is now added to the lxd group:


```
id -nG


```


You will receive an output like this:


```
sammy sudo lxd


```


Now you are ready to continue configuring LXD.


## Preparing the storage backend


To begin, you will configure the storage backend.


The recommended storage backend for LXD when you run it on Ubuntu is the ZFS filesystem. ZFS also works very well with DigitalOcean Block Storage. To enable ZFS support in LXD, first update your package list and then install the zfsutils-linux auxiliary package:


```
sudo apt update
sudo apt install -y zfsutils-linux


```


We are almost ready to run the LXD initialization script.


Before you do, you must identify and take a note of the device name for your block storage.


To do so, use ls to check the /dev/disk/by-id/ directory:


```
ls -l /dev/disk/by-id/


```


In this specific example, the full path of the device name is /dev/disk/by-id/scsi-0DO_Volume_volume-fra1-0:


```
Outputtotal 0
lrwxrwxrwx 1 root root  9 Sep  16 20:30 scsi-0DO_Volume_volume-fra1-0 -> ../../sda


```


Note down the full file path for your storage device. You will use it in the following subsection.


You are now ready to initialize LXD. Initialize LXD using the sudo lxd init command:


```
sudo lxd init


```


A prompt will appear. The next two sections will walk you through the appropriate answers to each question.


## Configuring Storage Options for LXD


First, the program will ask if you want to enable LXD clustering. For the purposes of this tutorial, press ENTER to accept the default no, or type no and then press ENTER. LXD clustering is an advanced topic that enables high availability for your LXD setup and requires at least three LXD servers running in a cluster:


```
OutputWould you like to use LXD clustering? (yes/no) [default=no]: no

```


The next six prompts deal with the storage pool. Give the following responses:


- Press ENTER to configure a new storage pool.
- Press ENTER to accept the default storage pool name.
- Press ENTER to accept the default zfs storage backend.
- Press ENTER to create a new ZFS pool.
- Type yes to use an existing block device.
- Lastly, type the full path to the block storage device name (This is what you recorded earlier. It should be something like: /dev/disk/by-id/device_name).

Your answers will look like the following:


```
OutputDo you want to configure a new storage pool? (yes/no) [default=yes]: yes
Name of the new storage pool [default=default]: default
Name of the storage backend to use (btrfs, dir, lvm, zfs) [default=zfs]: zfs
Create a new ZFS pool? (yes/no) [default=yes]: yes
Would you like to use an existing block device? (yes/no) [default=no]: yes
Path to the existing block device: /dev/disk/by-id/scsi-0DO_Volume_volume-fra1-01

```


You have now configured the storage backend for LXD. Continuing with LXD’s init script, you will now configure some networking options.


## Configuring Networking Options for LXD


LXD now asks whether you want to connect to a MAAS (Metal As A Server) server. MAAS is software that makes a bare-metal server appear as, and be handled as if, a virtual machine.


We are running LXD in standalone mode, therefore accept the default and answer no:


```
OutputWould you like to connect to a MAAS server? (yes/no) [default=no]: no

```


You are then asked to configure a network bridge for LXD containers. This enables the following features:


- Each container automatically gets a private IP address.
- Each container can communicate with each other over the private network.
- Each container can initiate connections to the internet.
- Each container remains inaccessible from the internet by default; you cannot initiate a connection from the internet and reach a container unless you explicitly enable it. You’ll learn how to allow access to a specific container in the next step.

When asked to create a new local network bridge, choose yes:


```
OutputWould you like to create a new local network bridge? (yes/no) [default=yes]: yes

```


Then accept the default name, lxdbr0:


```
OutputWhat should the new bridge be called? [default=lxdbr0]: lxdbr0

```


Accept the automated selection of private IP address range for the bridge:


```
OutputWhat IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: auto
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: auto

```


Finally, LXD asks the following miscellaneous questions:


When asked if you want to manage LXD over the network, press ENTER or answer no:


```
OutputWould you like LXD to be available over the network? (yes/no) [default=no]: no

```


When asked if you want to update stale container images automatically, press ENTER or answer yes:


```
OutputWould you like stale cached images to be updated automatically? (yes/no) [default=yes] yes

```


When asked if you want to view and keep the YAML configuration you just created, answer yes if you do. Otherwise, you press ENTER or answer no:


```
OutputWould you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: no

```


You have now configured your network and storage options for LXD. Next you will create your first LXD container.


# Step 2 — Creating and Configuring an LXD Container


Now that you have successfully configured LXD, you are ready to create and manage your first container. In LXD, you manage containers using the lxc command followed by an action, such as list, launch, start, stop and delete.


Use lxc list to view the available installed containers:


```
lxc list


```


Since this is the first time that the lxc command communicates with the LXD hypervisor, it shows some information about how to launch a container. Finally, the command shows an empty list of containers. This is expected because we haven’t created any yet:


```
Output of the "lxd list" commandTo start your first container, try: lxc launch ubuntu:18.04
+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+

```


Now create a container that runs Nginx. To do so, first use the lxc launch command to create and start an Ubuntu 18.04 container named webserver.


Create the webserver container. The 18.04 in ubuntu:18.04 is a shortcut for Ubuntu 18.04. ubuntu: is the identifier for the preconfigured repository of LXD images. You could also use ubuntu:bionic for the image name:


```
lxc launch ubuntu:18.04 webserver


```



Note: You can find the full list of all available Ubuntu images by running lxc image list ubuntu: and other Linux distributions by running lxc image list images:. Both ubuntu: and images: are repositories of container images. For each container image, you can get more information with the command lxc image info ubuntu:18.04. While we launch a container with Ubuntu 18.04 for the purposes of this tutorial, you may select any available Ubuntu version for your own projects.

Because this is the first time you’ve created a container, this command downloads the container image from the internet and caches it. You’ll see this output once your new container finishes downloading:


```
OutputCreating webserver
Starting webserver

```


With the webserver container started, use the lxc list command to show information about it. We added --columns ns4 in order to show only the columns for name, state and IPv4 address. The default lxc list command shows three more columns: the IPv6 address, whether the container is persistent or ephemeral, and whether there are snapshots available for each container:


```
lxc list --columns ns4


```


The output shows a table with the name of each container, its current state, its IP address, and its type:


```
Output+-----------+---------+------------------------------------+
|   NAME    |  STATE  |        IPV4                        |
+-----------+---------+------------------------------------+
| webserver | RUNNING | your_webserver_container_ip (eth0) |
+-----------+---------+------------------------------------+

```


LXD’s DHCP server provides this IP address and in most cases it will remain the same even if the server is rebooted. However, in the following steps you will create iptables rules to forward connections from the internet to the container. Therefore, you should instruct LXD’s DHCP server to always give the same IP address to the container.


The following set of commands will configure the container to obtain a static IP assignment. First, you will override the network configuration for the eth0 device that is inherited from the default LXD profile. This allows you to set a static IP address, which ensures proper communication of web traffic into and out of the container.


Specifically, lxc config device is a command that performs the config action to configure a device. The first line has the sub-action override to override the device eth0 from the container webserver. The second line has the sub-action to set the ipv4.address field of the eth0 device of the webserver container to the IP address that was given by the DHCP server in the beginning.


Run the first config command:


```
lxc config device override webserver eth0


```


You will receive an output like this:


```
OutputDevice eth0 overridden for webserver

```


Now set the static IP:


```
lxc config device set webserver eth0 ipv4.address your_webserver_container_ip


```


If the command is successful you will receive no output.


Restart the container:


```
lxc restart webserver


```


Now check the status of the container:


```
lxc list


```


You should see that the container is RUNNING and the IPV4 address is your static address.


You are ready to install and configure Nginx inside the container.


# Step 3 — Configuring Nginx Inside an LXD Container


In this step you will connect to the webserver container and configure the web server.


Connect to the container with  lxc shell command, which takes the name of the container and starts a shell inside the container:


```
lxc shell webserver


```


Once inside the container, your shell prompt will look like the following:


```
[environment second]  


```


This shell, even if it is a root shell, is limited to the container. Anything that you run in this shell stays in the container and cannot escape to the host server.



Note: When getting a shell into a container, you may see a warning such as mesg: ttyname failed: No such device. This message is produced when the shell in the container tries to run the command mesg from the configuration file /root/.profile. You can safely ignore it. To avoid seeing it, you may remove the command mesg n || true from /root/.profile.

Once inside your container, update the package list and install Nginx:


```
apt update
apt install nginx


```


With Nginx installed, you will now edit the default Nginx web page. Specifically, you will add two lines of text so that it is clear that this site is hosted inside the webserver container.


Using nano or your preferred editor, open the file /var/www/html/index.nginx-debian.html:


```
nano /var/www/html/index.nginx-debian.html


```


Add the two highlighted phrases to the file:


```
/var/www/html/index.nginx-debian.html<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx on LXD container webserver!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx on LXD container webserver!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
...

```


You have edited the file in two places and specifically added the text on LXD container webserver. Save the file and exit your text editor.


Now log out of the container:


```
logout


```


Once the server’s default prompt returns, use curl to test that the web server in the container is working. To do this, you’ll need the IP address of the web container, which you found using the lxc list command earlier.


Use curl to test your web server:


```
curl http://your_webserver_container_ip


```


You will receive the Nginx default HTML welcome page as output. Note that it includes your edits:


```
Output<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx on LXD container webserver!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx on LXD container webserver!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
...

```


The web server is working but you can only access it while on the host using the private IP. In the next step, you will route external requests to this container so the world can access your web site through the internet.


# Step 4 — Forwarding Incoming Connections to the Nginx Container Using LXD


Now that you have configured Nginx, it’s time to connect the webserver container to the internet. To begin, you need to set up the server to forward any connections that it may receive on port 80 to the webserver container.  To do this, you’ll create an iptables rule to forward network connections. You can learn more about IPTables in our tutorials, How the IPtables Firewall Works and IPtables Essentials: Common Firewall Rules and Commands.


This iptables command requires two IP addresses: the public IP address of the server (your_server_ip) and the private IP address of the webserver container (your_webserver_container_ip), which you can obtain using the lxc list command.


Execute this command to create a new IPtables rule:


```
PORT=80 PUBLIC_IP=your_server_ip CONTAINER_IP=your_container_ip IFACE=eth0  sudo -E bash -c 'iptables -t nat -I PREROUTING -i $IFACE -p TCP -d $PUBLIC_IP --dport $PORT -j DNAT --to-destination $CONTAINER_IP:$PORT -m comment --comment "forward to the Nginx container"'


```


Let’s study that command:


- -t nat specifies that we’re using the nat table for address translation.
- -I PREROUTING specifies that we’re adding the rule to the PREROUTING chain.
- -i $IFACE specifies the interface eth0, which is the default public network interface on the host for Droplets.
- -p TCP says we’re using the TCP protocol.
- -d $PUBLIC_IP specifies the destination IP address for the rule.
- --dport $PORT: specifies the destination port (such as 80).
- -j DNAT says that we want to perform a jump to Destination NAT (DNAT).
- --to-destination $CONTAINER_IP:$PORT  says that we want the request to go to the IP address of the specific container and the destination port.


Note: You can reuse this command to set up forwarding rules. Reset the variables PORT, PUBLIC_IP, CONTAINER_IP and IFACE at the start of the line. Just change the highlighted values.

Now list your IPTables rules:


```
sudo iptables -t nat -L PREROUTING


```


You’ll see output like this:


```
[secondary_label Output] 
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DNAT       tcp  --  anywhere             your_server_ip       tcp dpt:http /* forward to this container */ to:your_container_ip:80
...

```


Now test that the webserver is accessible from the internet


Use the curl command from your local machine to test the connections:


```
curl --verbose  'http://your_server_ip'


```


You’ll see the headers followed by the contents of the web page you created in the container:


```
Output*   Trying your_server_ip...
* Connected to your_server_ip (your_server_ip) port 80 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.10.0 (Ubuntu)
...
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx on LXD container webserver!</title>
<style>
    body {
...

```


This confirms that the requests are going to the container.


Finally, you will save the firewall rule so that it reapplies after a reboot.


To do so, first install the iptables-persistent package:


```
sudo apt install iptables-persistent


```


When installing the package, the application will prompt you to save the current firewall rules. Accept and save all current rules.


When you reboot your machine, the firewall rule will load. In addition, the Nginx service in your LXD container will automatically restart.


You’ve successfully configured LXD. In the final step you will learn how to stop and destroy the service.


# Step 5 — Stopping and Removing Containers Using LXD


You may decide that you want to take down the container and delete it. In this step you will stop and remove your container.


First, stop the container:


```
lxc stop webserver


```


Use the lxc list command to verify the status:


```
lxc list


```


You will see that the container’s state reads STOPPED:


```
Output+-----------+---------+------+------+------------+-----------+
|   NAME    |  STATE  | IPV4 | IPV6 |    TYPE    | SNAPSHOTS |
+-----------+---------+------+------+------------+-----------+
| webserver | STOPPED |      |      | PERSISTENT | 0         |
+-----------+---------+------+------+------------+-----------+

```


To remove the container, use lxc delete:


```
lxc delete webserver


```


Running lxc list again shows that there’s no container running:


```
lxc list


```


The command will output the following:


```
+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+

```


Use the lxc help command to see additional options.


To remove the firewall rule that routes traffic to the container, first locate the rule in the list of rules with this command, which associates a line number with each rule:


```
sudo iptables -t nat -L PREROUTING --line-numbers


```


You’ll see your rule, prefixed with a line number, like this:


```
OutputChain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination
1    DNAT       tcp  --  anywhere             your_server_ip      tcp dpt:http /* forward to the Nginx container */ to:your_container_ip

```


Use that line number to remove the rule:


```
sudo iptables -t nat -D PREROUTING 1


```


List the rules again to ensure removal:


```
sudo iptables -t nat -L PREROUTING --line-numbers


```


The rule is removed:


```
OutputChain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination

```


Now save the changes so that the rule doesn’t come back when you restart your server:


```
sudo netfilter-persistent save


```


You can now bring up another container with your own settings and add a new firewall rule to forward traffic to it.


# Conclusion


In this tutorial, you installed and configured LXD. You then created a website using Nginx running inside an LXD container and made it publicly available us IPtables.


From here, you could configure more websites, each confined to its own container, and use a reverse proxy to direct traffic to the appropriate container. The tutorial How to Host Multiple Web Sites with Nginx and HAProxy Using LXD on Ubuntu 16.04 walks you through that setup.


See the LXD reference documentation for more information on how to use LXD.


To practice with LXD, you can try LXD online and follow the web-based tutorial.


To get user support on LXD, visit the LXD discussion forum.


