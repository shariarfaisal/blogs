# How To Set Up a Minecraft Server on Linux

```Miscellaneous``` ```CentOS``` ```Gaming```


Status: Deprecated
This article is deprecated and no longer maintained.
Reason
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.
See Instead
This article may still be useful as a reference, but may not follow best practices or work on this or other Ubuntu releases. We strongly recommend using a recent article written for the version of Ubuntu you are using.
If you are currently operating a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:

How to upgrade from Ubuntu 12.04 to Ubuntu 14.04.
How to upgrade from Ubuntu 14.04 to Ubuntu 16.04
How to migrate server data to a supported version


Setting up a Minecraft server on Linux (Ubuntu 12.04) is a fairly easy task on the command line.


When choosing your server, be sure that it has (at a minimum)1GB of RAM, preferably at least 2GB.


The first thing you need to do is to connect to your server through SSH. If you are on a mac, you can open up Terminal, or if you are on a PC, you can connect with PuTTY. Once the command line is opened, login by typing:


```
ssh username@ipaddress

```


Enter the password when prompted. Although you can set up the server on the root user, it is not as secure as setting it up under another username. You can check out this tutorial to see how to add users.


# Step One—Install the Requirements


Before going further, we should run a quick update on apt-get, the program through which we will download all of the server requirements.


```
sudo apt-get update

```


After that, we need to be sure that Java is installed on our server. You can check by typing this command:


```
 java -version

```


If you don’t have Java installed, you will get a message that says “java: command not found”. You can, then, download java through apt-get:


```
sudo apt-get install default-jdk

```


You also need to supply your server with Screen which will keep your server running if you drop the connection:


```
sudo apt-get install screen

```


There is a complete guide on how to install and use screen here.


# Install the Minecraft Server


Start off by creating a new directory where you will store the Minecraft files:


```
mkdir minecraft

```


Once the directory is created, switch into it:


```
cd minecraft

```


Within that directory, download the Minecraft server software:


```
wget -O minecraft_server.jar https://s3.amazonaws.com/Minecraft.Download/versions/1.7.4/minecraft_server.1.7.4.jar

```


Since we have installed screen, you can start it running (-S sets the sessions title):


```
screen -S "Minecraft server"

```


After the file downloads, you can run it with Java:


```
java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui

```


The launching text should look something like this:


```
2012-08-06 21:12:52 [INFO] Loading properties
2012-08-06 21:12:52 [WARNING] server.properties does not exist
2012-08-06 21:12:52 [INFO] Generating new properties file
2012-08-06 21:12:52 [INFO] Default game type: SURVIVAL
2012-08-06 21:12:52 [INFO] Generating keypair
2012-08-06 21:12:53 [INFO] Starting Minecraft server on *:25565
2012-08-06 21:12:53 [WARNING] Failed to load operators list: java.io.FileNotFoundException: ./ops.txt (No such file or directory)
2012-08-06 21:12:53 [WARNING] Failed to load white-list: java.io.FileNotFoundException: ./white-list.txt (No such file or directory)
2012-08-06 21:12:53 [INFO] Preparing level "world"
2012-08-06 21:12:53 [INFO] Preparing start region for level 0
2012-08-06 21:12:54 [INFO] Preparing spawn area: 4%
2012-08-06 21:12:55 [INFO] Preparing spawn area: 12%
2012-08-06 21:12:56 [INFO] Preparing spawn area: 20%
2012-08-06 21:12:57 [INFO] Preparing spawn area: 24%
2012-08-06 21:12:58 [INFO] Preparing spawn area: 32%
2012-08-06 21:12:59 [INFO] Preparing spawn area: 36%
2012-08-06 21:13:00 [INFO] Preparing spawn area: 44%
2012-08-06 21:13:01 [INFO] Preparing spawn area: 48%
2012-08-06 21:13:02 [INFO] Preparing spawn area: 52%
2012-08-06 21:13:03 [INFO] Preparing spawn area: 61%
2012-08-06 21:13:04 [INFO] Preparing spawn area: 69%
2012-08-06 21:13:05 [INFO] Preparing spawn area: 77%
2012-08-06 21:13:06 [INFO] Preparing spawn area: 85%
2012-08-06 21:13:07 [INFO] Preparing spawn area: 93%
2012-08-06 21:13:08 [INFO] Done (15.509s)! For help, type "help" or "?"

```


Your Minecraft server is now all set up. You can exit out of screen by pressing


```
ctl-a d

```


To reattach screen, type


```
screen -R

```


You can change the settings of your server by opening up the server properties file:


```
 nano ~/minecraft/server.properties

```


