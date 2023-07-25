# How To Set Up an OpenVPN Server on Debian 8

```Security``` ```VPN``` ```Firewall``` ```Debian```

## Introduction


OpenVPN is an open source VPN application that lets you create and join a private network securely over the public Internet. In short, this allows the end user to mask connections and more securely navigate an untrusted network.


With that said, this tutorial teaches you how to setup OpenVPN, an open source Secure Socket Layer (SSL) VPN solution, on Debian 8.



Note: If you plan to set up an OpenVPN server on a DigitalOcean Droplet, be aware that we, like many hosting providers, charge for bandwidth overages. For this reason, please be mindful of how much traffic your server is handling.
See this page for more info.

# Prerequisites


This tutorial assumes you have the following:


- One fresh Debian 8.1 Droplet
- A root user
- Optional: After completion of this tutorial, use a sudo-enabled, non-root account for general maintenance; you can set one up by following steps 2 and 3 of this tutorial

# Step 1 — Install OpenVPN


Before installing any packages, update the apt package index.


```
apt-get update


```


Now, we can install the OpenVPN server along with easy-RSA for encryption.


```
apt-get install openvpn easy-rsa


```


# Step 2 — Configure OpenVPN


The example VPN server configuration file needs to be extracted to /etc/openvpn so we can incorporate it into our setup. This can be done with one command:


```
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz > /etc/openvpn/server.conf


```


Once extracted, open the server configuration file using nano or your favorite text editor.


```
nano /etc/openvpn/server.conf


```


In this file, we will need to make four changes (each will be explained in detail):


1. Secure server with higher-level encryption
2. Forward web traffic to destination
3. Prevent DNS requests from leaking outside the VPN connection
4. Setup permissions

First, we’ll double the RSA key length used when generating server and client keys. After the main comment block and several more chunks, search for the line that reads:


/etc/openvpn/server.conf
```
# Diffie hellman parameters.
# Generate your own with:
#   openssl dhparam -out dh1024.pem 1024
# Substitute 2048 for 1024 if you are using
# 2048 bit keys.
dh dh1024.pem

```


Change dh1024.pem to dh2048.pem, so that the line now reads:


/etc/openvpn/server.conf
```
dh  dh2048.pem

```


Second, we’ll make sure to redirect all traffic to the proper location. Still in server.conf, scroll past more comment blocks, and look for the following section:


/etc/openvpn/server.conf
```
# If enabled, this directive will configure
# all clients to redirect their default
# network gateway through the VPN, causing
# all IP traffic such as web browsing and
# and DNS lookups to go through the VPN
# (The OpenVPN server machine may need to NAT
# or bridge the TUN/TAP interface to the internet
# in order for this to work properly).
;push "redirect-gateway def1 bypass-dhcp"

```


Uncomment push "redirect-gateway def1 bypass-dhcp" so the VPN server passes on clients’ web traffic to its destination. It should look like this when done:


/etc/openvpn/server.conf
```
push "redirect-gateway def1 bypass-dhcp"

```


Third, we will tell the server to use OpenDNS for DNS resolution where possible. This can help prevent DNS requests from leaking outside the VPN connection. Immediately after the previously modified block, edit the following:


/etc/openvpn/server.conf
```
# Certain Windows-specific network settings
# can be pushed to clients, such as DNS
# or WINS server addresses.  CAVEAT:
# http://openvpn.net/faq.html#dhcpcaveats
# The addresses below refer to the public
# DNS servers provided by opendns.com.
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"

```


Uncomment push "dhcp-option DNS 208.67.222.222" and push "dhcp-option DNS 208.67.220.220". It should look like this when done:


/etc/openvpn/server.conf
```
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"

```


Fourth, we will define permissions in server.conf:


/etc/openvpn/server.conf
```
# You can uncomment this out on
# non-Windows systems.
;user nobody
;group nogroup

```


Uncomment both user nobody and group nogroup. It should look like this when done:


/etc/openvpn/server.conf
```
user nobody
group nogroup

```


By default, OpenVPN runs as the root user and thus has full root access to the system. We’ll instead confine OpenVPN to the user nobody and group nogroup. This is an unprivileged user with no default login capabilities, often reserved for running untrusted applications like web-facing servers.


Now save your changes and exit.


# Step 3 — Enable Packet Forwarding


In this section, we will tell the server’s kernel to forward traffic from client services out to the Internet. Otherwise, the traffic will stop at the server.


Enable packet forwarding during runtime by entering this command:


```
echo 1 > /proc/sys/net/ipv4/ip_forward


```


Next, we’ll need to make this permanent so that this setting persists after a server reboot. Open the sysctl configuration file using nano or your favorite text editor.


```
nano /etc/sysctl.conf


```


Near the top of the sysctl file, you will see:


/etc/openvpn/server.conf
```
# Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1

```


Uncomment net.ipv4.ip_forward. It should look like this when done:


/etc/openvpn/server.conf
```
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

```


Save your changes and exit.


# Step 4 — Install and Configure ufw


UFW is a front-end for IPTables. We only need to make a few rules and configuration edits. Then we will switch the firewall on. As a reference for more uses for UFW, see How To Setup a Firewall with UFW on an Ubuntu and Debian Cloud Server.


First, install the ufw package.


```
apt-get install ufw


```


Second, set UFW to allow SSH:


```
ufw allow ssh


```


This tutorial will use OpenVPN over UDP, so UFW must also allow UDP traffic over port 1194.


```
ufw allow 1194/udp


```


The UFW forwarding policy needs to be set as well. We’ll do this in the primary configuration file.


```
nano /etc/default/ufw


```


Look for the following line:


/etc/default/ufw
```
DEFAULT_FORWARD_POLICY="DROP"

```


This must be changed from DROP to ACCEPT. It should look like this when done:


/etc/default/ufw
```
DEFAULT_FORWARD_POLICY="ACCEPT"

```


Save and exit.


Next we will add additional UFW rules for network address translation and IP masquerading of connected clients.


```
nano /etc/ufw/before.rules


```


Next, add the area in red for OPENVPN RULES:


/etc/ufw/before.rules
```
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#

# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES

# Don't delete these required lines, otherwise there will be errors
*filter

```


Save and exit.


With the changes made to UFW, we can now enable it. Enter into the command prompt:


```
ufw enable


```


Enabling UFW will return the following prompt:


```
Command may disrupt existing ssh connections. Proceed with operation (y|n)?

```


Answer y. The result will be this output:


```
Firewall is active and enabled on system startup

```


To check UFW’s primary firewall rules:


```
ufw status


```


The status command should return these entries:


```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
1194/udp                   ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
1194/udp (v6)              ALLOW       Anywhere (v6)

```


# Step 5 — Configure and Build the Certificate Authority


OpenVPN uses certificates to encrypt traffic.


In this section, we will setup our own Certificate Authority (CA) in two steps: (1) setup variables and (2) generate the CA.


OpenVPN supports bidirectional authentication based on certificates, meaning that the client must authenticate the server certificate and the server must authenticate the client certificate before mutual trust is established. We will use Easy RSA’s scripts to do this.


First copy over the Easy-RSA generation scripts.


```
cp -r /usr/share/easy-rsa/ /etc/openvpn


```


Then, create a directory to house the key.


```
mkdir /etc/openvpn/easy-rsa/keys


```


Next, we will set parameters for our certificate. Open the variables file using nano or your favorite text editor.


```
nano /etc/openvpn/easy-rsa/vars


```


The variables below marked in red should be changed according to your preference.


/etc/openvpn/easy-rsa/vars
```
export KEY_COUNTRY="US"
export KEY_PROVINCE="TX"
export KEY_CITY="Dallas"
export KEY_ORG="My Company Name"
export KEY_EMAIL="sammy@example.com"
export KEY_OU="MYOrganizationalUnit"

```


In the same vars file, also edit this one line shown below. For simplicity, we will use server as the key name. If you want to use a different name, you would also need to update the OpenVPN configuration files that reference server.key and server.crt.


Below, in the same file, we will specify the correct certificate. Look for the line, right after the previously modified block that reads


/etc/openvpn/easy-rsa/vars
```
# X509 Subject Field
export KEY_NAME="EasyRSA"

```


Change KEY_NAME’s default value of EasyRSA to your desired server name. This tutorial will use the name server.


/etc/openvpn/easy-rsa/vars
```
# X509 Subject Field
export KEY_NAME="server"

```


Save and exit.


Next, we will generate the Diffie-Helman parameters using a built-in OpenSSL tool called dhparam; this may take several minutes.


The -out flag specifies where to save the new parameters.


```
openssl dhparam -out /etc/openvpn/dh2048.pem 2048


```


Our certificate is now generated, and it’s time to generate a key.


First, we will switch into the easy-rsa directory.


```
cd /etc/openvpn/easy-rsa


```


Now, we can begin setting up the CA itself.  First, initialize the Public Key Infrastructure (PKI).


Pay attention to the dot (.) and space in front of ./vars command. That signifies the current working directory (source).


```
. ./vars


```



The following warning will be printed. Do not worry, as the directory specified in the warning is empty. NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/openvpn/easy-rsa/keys.

Next, we’ll clear all other keys that may interfere with our installation.


```
./clean-all


```


Finally, we will build the CA using an OpenSSL command. This command will prompt you for a confirmation of “Distinguished Name” variables that were entered earlier. Press ENTER to accept existing values.


```
./build-ca


```


Press ENTER to pass through each prompt since you just set their values in the vars file.


The Certificate Authority is now setup.


# Step 6 — Generate a Certificate and Key for the Server


In this section, we will setup and launch our OpenVPN server.


First, still working from /etc/openvpn/easy-rsa, build your key with the server name. This was specified earlier as KEY_NAME in your configuration file. The default for this tutorial is server.


```
./build-key-server server


```


Again, output will ask for confirmation of the Distinguished Name. Hit ENTER to accept defined, default values. This time, there will be two additional prompts.


```
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

```


Both should be left blank, so just press ENTER to pass through each one.


Two additional queries at the end require a positive (y) response:


```
Sign the certificate? [y/n]
1 out of 1 certificate requests certified, commit? [y/n]

```


You will then be prompted with the following, indicating success.


```
OutputWrite out database with 1 new entries
Data Base Updated

```


# Step 7 — Move the Server Certificates and Keys


We will now copy the certificate and key to /etc/openvpn, as OpenVPN will search in that directory for the server’s CA, certificate, and key.


```
cp /etc/openvpn/easy-rsa/keys/{server.crt,server.key,ca.crt} /etc/openvpn


```


You can verify the copy was successful with:


```
ls /etc/openvpn


```


You should see the certificate and key files for the server.


At this point, the OpenVPN server is ready to go. Start it and check the status.


```
service openvpn start
service openvpn status


```


The status command will return something to the following effect:


```
Output* openvpn.service - OpenVPN service
   Loaded: loaded (/lib/systemd/system/openvpn.service; enabled)
   Active: active (exited) since Thu 2015-06-25 02:20:18 EDT; 9s ago
  Process: 2505 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
 Main PID: 2505 (code=exited, status=0/SUCCESS)

```


Most importantly, from the output above, you should find Active: active (exited) since... instead of Active: inactive (dead) since....


Your OpenVPN server is now operational. If the status message says the VPN is not running, then take a look at the /var/log/syslog file for errors such as:


```
Options error: --key fails with 'server.key': No such file or directory

```


That error indicates server.key was not copied to /etc/openvpn correctly. Re-copy the file and try again.


# Step 8 — Generate Certificates and Keys for Clients


So far we’ve installed and configured the OpenVPN server, created a Certificate Authority, and created the server’s own certificate and key. In this step, we use the server’s CA to generate certificates and keys for each client device which will be connecting to the VPN.


## Key and Certificate Building


It’s ideal for each client connecting to the VPN to have its own unique certificate and key. This is preferable to generating one general certificate and key to use among all client devices.



Note: By default, OpenVPN does not allow simultaneous connections to the server from clients using the same certificate and key. (See duplicate-cn in /etc/openvpn/server.conf.)

To create separate authentication credentials for each device you intend to connect to the VPN, you should complete this step for each device, but change the name client1 below to something different such as client2 or iphone2. With separate credentials per device, they can later be deactivated at the server individually, if need be. The remaining examples in this tutorial will use client1 as our example client device’s name.


As we did with the server’s key, now we build one for our client1 example. You should still be working out of /etc/openvpn/easy-rsa.


```
./build-key client1


```


Once again, you’ll be asked to change or confirm the Distinguished Name variables and these two prompts which should be left blank. Press ENTER to accept the defaults.


```
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

```


As before, these two confirmations at the end of the build process require a (y) response:


```
Sign the certificate? [y/n]
1 out of 1 certificate requests certified, commit? [y/n]

```


You will then receive the following output, confirming successful key build.


```
Write out database with 1 new entries.
Data Base Updated

```


Then, we’ll copy the generated key to the Easy-RSA keys directory that we created earlier. Note that we change the extension from .conf to .ovpn. This is to match convention.


```
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/easy-rsa/keys/client.ovpn


```


You can repeat this section again for each client, replacing client1 with the appropriate client name throughout.


Note: The name of your duplicated client.ovpn doesn’t need to be related to the client device. The client-side OpenVPN application will use the filename as an identifier for the VPN connection itself. Instead, you should duplicate client.ovpn to whatever you want the VPN’s name tag to be in your operating system. For example: work.ovpn will be identified as work, school.ovpn as school, etc.


We need to modify each client file to include the IP address of the OpenVPN server so it knows what to connect to. Open client.ovpn using nano or your favorite text editor.


```
nano /etc/openvpn/easy-rsa/keys/client.ovpn


```


First, edit the line starting with remote. Change my-server-1 to your_server_ip.


/etc/openvpn/easy-rsa/keys/client.ovpn
```
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote your_server_ip 1194

```


Next, find the area shown below and uncomment user nobody and group nogroup, just like we did in server.conf in Step 1. Note: This doesn’t apply to Windows so you can skip it. It should look like this when done:


/etc/openvpn/easy-rsa/keys/client.ovpn
```
# Downgrade privileges after initialization (non-Windows only)
user nobody
group no group

```


## Transferring Certificates and Keys to Client Devices


Recall from the steps above that we created the client certificates and keys, and that they are stored on the OpenVPN server in the /etc/openvpn/easy-rsa/keys directory.


For each client we need to transfer the client certificate, key, and profile template files to a folder on our local computer or another client device.


In this example, our client1 device requires its certificate and key, located on the server in:


- /etc/openvpn/easy-rsa/keys/client1.crt
- /etc/openvpn/easy-rsa/keys/client1.key

The ca.crt and client.ovpn files are the same for all clients. Download these two files as well; note that the ca.crt file is in a different directory than the others.


- /etc/openvpn/easy-rsa/keys/client.ovpn
- /etc/openvpn/ca.crt

While the exact applications used to accomplish this transfer will depend on your choice and device’s operating system, you want the application to use SFTP (SSH file transfer protocol) or SCP (Secure Copy) on the backend. This will transport your client’s VPN authentication files over an encrypted connection.


Here is an example SCP command using our client1 example. It places the file client1.key into the Downloads directory on the local computer.


```
scp root@your-server-ip:/etc/openvpn/easy-rsa/keys/client1.key Downloads/


```


Here are several tools and tutorials for securely transferring files from the server to a local computer:


- WinSCP
- How To Use SFTP to Securely Transfer Files with a Remote Server
- How To Use Filezilla to Transfer and Manage Files Securely on your VPS

At the end of this section, make sure you have these four files on your client device:


- ``client1.crt
- ``client1.key
- client.ovpn
- ca.crt

# Step 9 — Creating a Unified OpenVPN Profile for Client Devices


There are several methods for managing the client files but the easiest uses a unified profile. This is created by modifying the client.ovpn template file to include the server’s Certificate Authority, and the client’s certificate and its key. Once merged, only the single client.ovpn profile needs to be imported into the client’s OpenVPN application.


The area given below needs the three lines shown to be commented out so we can instead include the certificate and key directly in the client.ovpn file. It should look like this when done:


/etc/openvpn/easy-rsa/keys/client.ovpn
```
# SSL/TLS parms.
# . . .
;ca ca.crt
;cert client.crt
;key client.key

```


Save the changes and exit. We will add the certificates by code.


First, add the Certificate Authority.


```
echo '<ca>' >> /etc/openvpn/easy-rsa/keys/client.ovpn
cat /etc/openvpn/ca.crt >> /etc/openvpn/easy-rsa/keys/client.ovpn
echo '</ca>' >> /etc/openvpn/easy-rsa/keys/client.ovpn


```


Second, add the certificate.


```
echo '<cert>' >> /etc/openvpn/easy-rsa/keys/client.ovpn
cat /etc/openvpn/easy-rsa/keys/client1.crt >> /etc/openvpn/easy-rsa/keys/client.ovpn
echo '</cert>' >> /etc/openvpn/easy-rsa/keys/client.ovpn


```


Third and finally, add the key.


```
echo '<key>' >> /etc/openvpn/easy-rsa/keys/client.ovpn
cat /etc/openvpn/easy-rsa/keys/client1.key >> /etc/openvpn/easy-rsa/keys/client.ovpn
echo '</key>' >> /etc/openvpn/easy-rsa/keys/client.ovpn


```


We now have a unified client profile. Using scp, you can then copy the client.ovpn file to your second system.


# Step 10 — Installing the Client Profile


Various platforms have more user-friendly applications to connect to this OpenVPN server. For platform-specific instructions, see Step 5 in this tutorial.


# Conclusion


Congratulations! You now have a working OpenVPN server and client file.


From your OpenVPN client, you can test the connection using Google to reveal your public IP. On the client, load it once before starting the OpenVPN connection and once after. The IP address should change.


