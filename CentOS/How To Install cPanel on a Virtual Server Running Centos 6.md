# How To Install cPanel on a Virtual Server Running Centos 6

```Control Panels``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


# About cPanel


cPanel is a convenient application that allows users to administer servers through a GUI interface instead of the traditional command line. Although the installation for cPanel is relatively simple, the script does take several hours to run.


## Notes


- Once cPanel is installed, it cannot be removed from the server without a complete server restore. cPanel does not offer an uninstaller
- Additionally, cPanel is subject to a licensing fee which may come out to be around $200 a year. DigitalOcean does not cover the cost of cPanel. You can find out more about cPanel pricing here

# Setup


Before installing cPanel on our droplet, we need to take two additional steps.


First we need to make sure that Perl is installed on the server


```
sudo yum install perl
```


After installing perl we need to take one more preliminary step. cPanel is very picky about making sure that server that it is installed on has a Fully Qualified Domain Name.  To that effect, we need to provide it with a valid hostname. Skipping this step will inevitably get you the following, very common, error.


```
2012-11-01 16:00:54  461 (ERROR): Your hostname () is not set properly. Please
2012-11-01 16:00:54  462 (ERROR): change your hostname to a fully qualified domain name,
2012-11-01 16:00:54  463 (ERROR): and re-run this installer.
```


Luckily this error has a very easy solution. If you have a FQDN, you can type it in with the command:


```
hostname your FQDN
```


Otherwise, if you want to proceed with the cPanel installation but do still lack the hostname, you can input a temporary one. Once cPanel is installed, you will be able to change the hostname to the correct one on one of the first setup pages.


```
hostname  host.example.com
```


# Install cPanel


Although the cPanel installation only has several steps, the installation does take a long time. Although using program "screen" is not necessary in order to install cPanel, it can be a very helpful addition to the installation process. It can be especially useful if you know that you may have issues with intermittent internet or that you will need to pause the lengthy install process.


To start off, go ahead and install screen and wget:


```
sudo yum install screen wget
```


Once screen is installed, start a new session running:


```
screen
```


After opening screen, you can proceed to install cPanel with WHM or a DNS only version of cPanel.


- Use this this command to install cPanel with WHM: 
wget -N http://httpupdate.cPanel.net/latest
- Use this command to install the DNS only version of cPanel:
wget -N http://httpupdate.cPanel.net/latest-dnsonly

With the requested package downloaded, we can go ahead and start the script running:


```
sh latest
```


Then close out of screen. The script, which may take one to two hours to complete will continue running while in the background—even if you close out the of server.


In order to detach screen type: Cntrl-a-d


To reattach to your screen you can use the command:


```
screen -r
```


Once cPanel finally installs, you can access the login by going to your ip address:2087 (eg. 12.34.45.678:2087l) or domain (example.com:2087)


Your login will be:


```
username: your_server_user
password: your_password
```


From there, you can create your cpanel user and finally login in at ipaddress/cpanel or domain/cpanel


By Etel Sverdlov
