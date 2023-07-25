# How To Run an UnrealIRCd Chat Server on Debian 7

```Applications``` ```Debian```

## Introduction


In this article, we will learn how to build, install, and configure an IRC server using the Unreal IRC daemon (IRCd): a service that allows users to connect through IRC (Internet Relay Chat). IRC is a protocol that allows users to chat in real time via messages sent over a network.


In particular, we will be on a Debian 7 x64 Droplet for this tutorial.


# Step One — Creating an ircd User


We will assume you’ve just launched a fresh Debian 7 x64 Droplet and you’re logged in as root.  If you are not currently the root user, switch to root with:


```
su

```


As with any daemon or service outside users can see, it is best to not run UnrealIRCd as root, for both security and performance reasons.  Because of this, we’ll start off by creating a user specifically to run the daemon.


```
adduser ircd

```


Create a new password for the user at the prompt. I recommend a password created using the XKCD method of creating passwords — that is, combine 3-5 fairly common dictionary words into a memorable combination whose length will add enough entropy (strength) to the password to make it difficult for a computer to guess but easy for a human to remember.


If you are not prompted to create a password, simply type:


```
passwd ircd

```


You will then be prompted to create the password.


## Giving the User Privileges


We’ll now need to make sure the server comes with sudo, a popular utility for UNIX-based systems (such as Linux).  For those unfamiliar, the sudo command allows users to execute commands with root (su or superuser) privileges, hence the name — superuser do — which we’ll need when setting up the prerequisite libraries.


Debian 7 has sudo installed by default. If you need to install it, type:


```
apt-get install sudo

```


Now that we know sudo is installed, you’ll need to add the ircd user we created earlier to the sudo group to allow it access to the sudo command.


To do this, run this command:


```
adduser ircd sudo

```


That’s all for the user creation step! Now we can simply switch to the new user:


```
su ircd

```


Change to the ircd user’s home directory.


```
cd ~

```


# Step Two — Installing Prerequisites


Before we can install UnrealIRCd, there are a few libraries we’re missing.  These include libraries used for secure connection as well as the necessary compilers and libraries to build UnrealIRCd.


These packages are make, gcc, build-essential, openssl, libcurl4-openssl-dev, zlib1g, zlib1g-dev, zlibc, libgcrypt11, and libgcrypt11-dev.


We’ll be using APT to install these in Debian. Run the following command to get your package manager up to date:


```
sudo apt-get update

```


Run the following command to install the prerequisites. Enter yes when prompted. You may find that some of these packages are already installed — at the time of writing, zlib1g and openssl are already included with Debian — but they are listed here for completeness.


```
sudo apt-get install make gcc build-essential openssl libcurl4-openssl-dev zlib1g zlib1g-dev zlibc libgcrypt11 libgcrypt11-dev

```


Now would be a good time to grab a snack, because it may take a while for the package manager to download and install all of the packages.


Once this is done, we can download and install UnrealIRCd and get started on the build process.


# Step Three — Installing UnrealIRCd


UnrealIRCd isn’t available by default as a package on Debian or any of the major distros, so we will be manually downloading and building it from source.


We’ll be downloading the source package for UnrealIRCd using the wget utility built into Debian.  If you need to download wget, run this command:


```
sudo apt-get install wget

```


You’ll need to get the download link from UnrealIRCd’s website in order to proceed with the installation. Head over to the official UnrealIRCd download page and download the latest stable source branch.


When the download page is presented to you, you may see a link that states “The download should begin in 5 seconds, if it doesn’t click here.” This is the link to the file that contains the Unreal source code, and the URL you should use with wget.


At the time of writing, the URL is http://unrealircd.org/downloads/Unreal3.2.10.4.tar.gz.


While logged in as our ircd user, let’s make sure we’re in our home directory:


```
cd ~

```


Now, let’s download the source code. Warning: Using the --no-check-certificate flag with wget is a less secure way to get this file. You should use a different method if your goal is security over speed:


```
wget --no-check-certificate http://unrealircd.org/downloads/Unreal3.2.10.4.tar.gz

```


Extract the source code to its own folder:


```
tar xzvf Unreal3.2.10.4.tar.gz

```


Now we can switch to the new directory:


```
cd Unreal3.2.10.4

```


And finally, let’s begin the build process. Use the ./Config script built into the source package:


```
./Config

```


Read through the release notes and press Enter to continue scrolling through. Keep an eye on the percentage number at the bottom so you don’t miss the questions. Once the release notes end and the configure script starts, UnrealIRCd will start asking you questions.


```
What directory are all the server configuration files in?

```


You can leave this at the default. Press Enter to continue.


```
What is the path to the ircd binary including the name of the binary?

```


Leave this as the default as well.  Press Enter.


```
What should the default permissions for your configurations files be?  (Set this to 0 to disable)

```


Once more, leave it at the default. The next question is important:


```
Do you want to support SSL (Secure Sockets Layer) connections?

```


Answer:


```
Yes

```


By default UnrealIRCd’s configuration script will not support SSL connections, but entering Yes here will enable it. This is why we installed OpenSSL earlier.


You should then be prompted for the path to OpenSSL on the system. Because we let APT handle this for us earlier, we can leave it at the default.


```
If you know the path to OpenSSL on your system, enter it here. If not
leave this blank (in most cases it will be detected automatically).

```


Press Enter.


The next question is important for certain users using IPv6 on their Droplet.


```
Do you want to enable IPv6 support?

```


You should only answer Yes to this question if you enabled IPv6 when initially setting up your Droplet. If you do not have IPv6 enabled on your Droplet, you may leave this at the default (“No”) and press Enter.


The next question concerns users who will be operating the IRC daemon with multiple linked servers in an IRC network. Ziplinks is essentially a means of saving bandwidth when exchanging data between IRC servers by compressing the data before it is sent. You aren’t required to answer Yes, but it’s recommended that you do. If you encounter difficulties building UnrealIRCd not documented in this article, try running ./Config again and answer No to this question.


```
Do you want to enable ziplinks support?

```


Answer:


```
Yes

```


When asked about the path, press Enter.


```
If you know the path to zlib on your system, enter it here. If not
leave this blank

```


The next few questions can all be be answered with the default by pressing Enter:


```
Do you want to enable remote includes?
Do you want to enable prefixes for chanadmin and chanowner?
What listen() backlog value do you wish to use?
How far back do you want to keep the nickname history?
What is the maximum sendq length you wish to have?
How many buffer pools would you like?
How many file descriptors (or sockets) can the IRCd use?
Would you like to pass any custom parameters to configure?

```


Once you’ve finished answering questions, the configuration script will begin preparing the source directory for the build process. This may take a few moments, so feel free to take a short break.


You will eventually be prompted to generate an SSL certificate for IRCd.  Unless you already have an SSL certificate issued by a proper certification authority, you probably just want to leave this option at the default (which should be “Yes”) — it’s what we’ll be doing in this tutorial.


You’ll be asked for information about the server, such as the country, province, city, and organization. You can enter whatever you feel makes sense here. You should enter your server’s domain name when asked for the name.


Press Enter to continue. You’ll see a quick message about signed certificates. Press Enter again, and the configuration process is complete.


# Step Four — Building UnrealIRCd


This is the simplest step of them all! Move to the ~/Unreal[whatever version you downloaded] directory:


```
cd ~/Unreal3.2.10.4

```


Build it!


```
make

```


The build process should begin. This is the longest step and if you haven’t already gone to make tea or coffee during the last two breaks, now is definitely the time to do so — it’s going to take a few minutes to complete the build process.


# Step Five — Configuring UnrealIRCd


After the build process has completed, you are now ready to configure your new UnrealIRCd server. The configuration file is written with C-like syntax. It’s long, but fairly simple.


Make a copy of the example configuration file:


```
cp ~/Unreal3.2.10.4/doc/example.conf ~/Unreal3.2.10.4/unrealircd.conf

```


This will copy the English example configuration file to the root Unreal directory.  If your server will be operating in a language other than English, you are welcome to use the two-letter language code for whatever your target audience will be using; e.g. ./doc.example.fr.conf At the time of writing, German, Spanish, French, Hungarian, Dutch, Russian, and Turkish are supported alongside English, but in this tutorial we’ll be addressing the English example configuration.


Open the configuration file with your favorite text editor:


```
nano ~/Unreal3.2.10.4/unrealircd.conf

```


Now, let’s start editing the configuration file to our liking.  We will be stepping through the configuration file sequentially from top to bottom, so the ordering of the sections may be a bit odd. Going from top to bottom is the most straightforward method for our purposes.


Note that there are large commented out sections in this configuration file, and some settings that don’t affect most users, that we will not be showing in detail.


## Modules


UnrealIRCd is touted as being a modular IRCd, in that you can create functionality for the daemon without having to recompile the entire source code base and load/unload it on the fly.  Thus, some core functionality is stored in two stock modules of Unreal—the commands module, and the cloak module.  Look for these two lines:


```
// loadmodule “src/modules/commands.so”;
// loadmodule “src/modules/cloak.so”;

```


For those unfamiliar with C-like syntax, the double slash (//) tells Unreal to ignore whatever comes after the slashes on a line. Multiple lines can be commented out by adding /* and */ around the material.


Uncomment the two modules so they look like this:


```
/* FOR *NIX, uncomment the following 2lines: */
loadmodule "src/modules/commands.so";
loadmodule "src/modules/cloak.so";

```


## me {} — Server Name


Continue scrolling through the configuration file until you see a section labeled me {}. There will be a commented-out example section first. You want the uncommented one.


This section, referred to in the past as an M-line, defines specific details about the server for when users connect.


Set the name to the hostname of the server.


The info field should contain a formal name for your IRC server, such as “MyDomain IRC.”  The name and info fields must be enclosed in quote marks.


The numeric field should be left with the default number, unless you plan to link multiple IRC servers together. With multiple servers, you absolutely must ensure that all of the servers in the network have a different value in this field. This acts as their unique identifier to UnrealIRCd under the hood.


```
me
{
        name "unreal.example.com";
        info "Unreal Chat Server";
        numeric 1;
};

```


## admin {} — Administrator Contact Information


The next section is the admin {} block or a:line. Again, scroll until you reach the uncommented example.


This should contain your contact information as server administrator. You can put as much or as little here as you want, but you need to have at least one line.


Most users put their real names on the first line (“Bob Smith” in the example), a nickname on the second line (“bob” in the example), and an email address on the third line. The only requirement is that each of these lines be enclosed in quotation marks and ended with a semicolon (;).


```
admin {
        "Will Preston";
        "will";
        "will@example.com";
};

```


Now we can move on to more specific server-related configuration, beyond simple contact information.


## class {} — Client and Server Connection Settings


The next block, the class {} block, defines what the server will remember as “clients” connecting to the network (your users), and “servers” linking with the network (if you add your IRC server to a network of servers).


Generally, these settings can be left at the default for most small-network scenarios. However, if you are running a large network operation, you may want to read the documentation on what the values in these blocks do, as they have their usefulness but only in very specific situations.


The default settings are fine:


```
class           clients
{
        pingfreq 90;
        maxclients 500;
        sendq 100000;
        recvq 8000;
};      

class           servers
{
        pingfreq 90;
        maxclients 10;          /* Max servers we can have linked at a time */
        sendq 1000000;
        connfreq 100; /* How many seconds between each connection attempt */
};

```


## allow {} — Server Password, Exceptions


The allow {} blocks configure who is allowed to connect to the network.


Generally, for a large, open IRC server that anyone can join, you should leave this at the default.


For a server that you want kept private or password-protected, you should add the password line, as shown in the second example allow {} block, to the first global block. The first allow {} block has the global settings for client connections. You can see that the ip and hostname values are both set to *@*, meaning “everyone from everywhere.”


This allows you to prompt users for a password upon connection.


```
allow {
        ip             *@*;
        hostname       *@*;
        class           clients;
        maxperip 5;
        password "yourpasswordhere";    
};      

/* Passworded allow line */
allow {
        ip             *@255.255.255.255;
        hostname       *@*.passworded.people;
        class           clients;
        password "f00Ness";
        maxperip 1;
};

```


You can also comment out the second allow block if you don’t need to fine-tune your allow settings with different IPs or other connection parameters.


The next block, the allow channel {} block, defines what kinds of channels, or chatrooms, are exempt from restrictions set later in the configuration file.


A bit of explanation: In a later block, we will be able to define which channels or keywords in channel names are banned, and this block allows us to define exemptions from those bans. You can generally leave this alone unless you have a special case, and the example configuration shows us how this works:


```
    allow channel {
    	channel "#WarezSucks";
    	class "clients";
    };

```


This example makes the server allow all clients access to the #WarezSucks channel, despite the fact that channels matching the wildcard string *warez* later in the configuration file are restricted.


Unless you have a specific case where you want to ban a certain channel but not certain other channels, you can leave this alone without issue.


## oper {} - Operator Settings


The next block is by far one of the most important: the oper {} block, or o:line, which defines your credentials as the server owner, or what most users call the IRCop.


Tread carefully through this section, as you’ll need to make sure you get everything right.


The oper {} block defines what you and any other privileged users are allowed to do as an IRCop. This is where your administrators are defined, including yourself, and for this reason there are quite a few options at play. For now, however, we’ll get started with an example block.


The configuration file presently contains a sample oper {} block, with the username bobsmith. However, I recommend you simply remove this entirely and replace it with your own oper {} block. You’ll need to add a new block for each and every user you want to have as an operator on your server.


Below, I have an example o:line you can copy and paste into the configuration file.  The values denoted in red are to be changed by you.


```
oper username {
	class clients;
	from {
		userhost *@*;
	};
	password "yourpasswordhere";
	flags {
		global;
		netadmin;
		can_gkline;
		can_gzline;
		can_zline;
		can_kline;
		can_unkline;
		can_restart;
		can_die;
		can_rehash;
	};
	swhois "This will appear when users run a /WHOIS on you";
};

```


A bit of explanation for each of these items is as follows:


- class defines what kind of operator this user is, if you specified a class other than “clients” or “servers” previously.  You don’t need to change this.
- The from {} subblock contains which hostnames (specified by userhost) are allowed to use this o:line.  Generally, you can leave this at @, since most clients (including yourself) have a dynamic IP address, and therefore, an ever-changing hostname.
- password defines what password you will use to identify yourself to the server as an operator. The password must be enclosed in quotes, as with any string literal.
- The flags {} subblock contains which privileges the specified IRCop will have when identified as an operator.  The official documentation has a list of what each of these privilege indicators does, but we’ll go over them quickly to give you a basic idea of what to expect.
- global defines whether this IRCop will be considered an IRCop on all servers.  Alternatively, you can use local, where they will only have influence over users connected to the server their o:line is on, but for your own o:line, you should definitely use global.
- netadmin defines what level (or “rank”) the IRCop is when users request operator help.  netadmin is the highest, followed by servicesadmin, followed by admin, followed by coadmin, followed by no rank indicator at all.
- can_gkline, can_gzline, can_zline, can_kline, and can_unkline<^> all define privileges to be able to apply certain types of user bans in case you have to remove a problematic user.  In IRC lingo, a , can_gzline, can_zline, can_kline, and can_unkline all define privileges to be able to apply certain types of user bans in case you have to remove a problematic user.  In IRC lingo, a k-line refers to a ban from connecting to the server the line is applied to.  A z-line, also called a g-line on other types of non-Unreal servers, is like a k-line, except it applies to all servers on the network (while being stored locally on the server the ban is applied on).  A gz-line and gk-line are all the same, except their respective -lines are applied across all servers in the network–so if the server the ban is applied on delinks or disconnects from the network, the ban is still considered active. can_unkline allows you to remove these bans once set.
- can_restart specifies that you are allowed to send the /RESTART command to the server to restart the IRCd, disconnecting all users in the process.
- can_die specifies that you are allowed to send the /DIE command to the server to stop the IRCd from running, thereby disconnecting all users.
- can_rehash specifies that you are allowed to send the /REHASH command, which reloads the currently active configuration, useful for when you need to make changes without restarting the server
- Finally, the swhois option specifies any special messages you want users to see when they run a /WHOIS on you while you are identified as an operator.  This line can be omitted entirely, but I like to use this as an opportunity to put a funny message like “What are you looking at?”

You are welcome to duplicate the above o:line as necessary to accommodate for all of the users you want to be IRCops. Small networks generally only need 1-2 operators; however, you may feel it necessary to add more o:lines as your network grows, to help keep users in check.


## listen {} — IPs, Ports, and SSL


Now that your o:line is set up, we can move on to more just-as-critical parts of the configuration. In the listen {} block you can set up the ports users will be able to connect on.


The default settings should be fine for most servers.


```
listen		*:6697
{
	options
	{
		ssl;
		clientsonly;
	};
};

listen *:8067;
listen *:6667;

```


The * before the : in each listen declaration means “all.” This can be replaced with whatever IP address you’d like to bind UnrealIRCd to. If your server has more than one IP, you’ll need a separate listen declaration for each address.


For most purposes, however, you’ll simply use listen *:6667, and the * wildcard will attempt to have UnrealIRCd bind to any and all IP addresses on the port specified after the :, in this case, port 6667.


Note: In order to support IPv6 addresses in UnrealIRCd, you must have compiled it with IPv6 support in the build instructions above, and your Droplet must support IPv6 connections. Note, however, that when specifying IPv6 addresses, you must enclose the address itself in [brackets] so that the daemon can discern between what the IP is and what the port is:


```
listen [fd3a::a1c9:b311:985e]:6667

```


In the options {} subblock, the ssl flag defines whether the server will allow SSL connections.  The clientsonly flag (as opposed to serversonly) allows you to specify whether this port is for clients or servers only. Otherwise, by default, both clients and servers will be able to connect on this port. The clientsonly setting is fine.


NOTE: If you receive “address already in use” errors when first starting UnrealIRCd, you’ll need to replace the * wildcards with the IP address(es) of your Droplet, because Unreal may or may not be able to deduce the primary IP to bind to.


## link {} - Chat Server Networks


The next block, link {}, can be left with the default settings. This block is used when connecting two servers together to form an IRC network, which is not covered in this tutorial.


```
link            hub.mynet.com
{
        username        *;
        hostname        1.2.3.4;
        bind-ip         *;
        port            7029;
        hub             *;
        password-connect "LiNk";
        password-receive "LiNk";
        class           servers;
                options {
                        /* Note: You should not use autoconnect when linking services */
                        autoconnect;
                        ssl;
                        zip;
                };
};

```


## ulines {} — Services with Privileges


The ulines {} block defines which servers connecting to our network need to have elevated privileges.


Generally, when you connect services to an IRC network (services are a special set of programs and daemons that add a whole host of functionality to IRC not specified by the RFC standard, such as nickname registration), you use this to ensure the services’ privileges are elevated above those of a normal server.


You should not use this for a normal server. Replace the example servers with your own service servers:


```
ulines {
        services.roxnet.org;
        stats.roxnet.org;
};

```


## drpass {} — Passwords for /RESTART and /DIE


The next block, drpass {}, defines what passwords should be set in case an IRCop chooses to use the /DIE or /RESTART commands on the server.


Even if the IRCop has the privileges to do this, it’s good practice to set a password for this just in case their IRCop privileges get compromised. This way, a compromised operator account can’t bring the server down with simple IRCop access.


```
drpass {
	restart "your-restart-password-here";
	die "your-die-password-here";
};

```


## log {} — Log Settings


The next block, the log {} block, can be left with the default settings.


You can read up on the documentation for information on the log block if you wish, but generally it’s fine to leave this as it is. UnrealIRCd is very thorough in its log keeping.


## alias {} — Command Aliases


The next series of blocks, the alias {} blocks, contain definitions for command aliases in case you plan to install services alongside your new IRC server.


If you do not plan to launch services with your new server, you can just leave these blocks as they are without worry.


## files {} — Include Configuration Files


The next block we will be adjusting is the files {} block, wherein you specify certain special files to be included in the configuration.


Most of these are commented out, but you should at least look for, uncomment, and specify the following. WARNING: UnrealIRCd will not start if specify any file here that does not exist. When you’re finished editing the configuration, you’ll NEED to create these files. We’ll create the two files here later in the tutorial.


Locate and uncomment these two files:


```
motd ircd.motd;

...

rules ircd.rules

```


- The motd parameter specifies the server MOTD, or the message of the day, which gets sent to a user upon connection, or when they use the /MOTD command.
- The rules parameter specifies the file containing server rules provided by the /RULES command.  I recommend you put some thought into your server’s rules, as IRC has, in the past, been notorious for some not-so-legal file sharing and the like.

## tld {} — International Exceptions


The next block, the tld {} block, is functionally the same as the files block, except that it allows you to set special exceptions for users connecting from different countries, as indicated by their top-level domain (e.g. .au for Australia, .ru for Russia, etc.).


You may want to comment this section out, as files you create in this section are just as much required as the files found in the proper files {} block.  Enclose this section in C-style comments:


```
/*
tld {
	...
};
*/

```


This way, it no longer applies to the current configuration.


The next section is completely optional; however, because we’re covering the configuration sequentially, it may be best to at least skim over the instructions so you understand what each block does.


## ban {} — Ban Nicknames, IPs, and Names


These sections can all be left with default settings.


In this section, we’ll take a look at a variety of ways to secure your server from malicious users via the server configuration.


The next block, the ban {} block, allows you to ban a particular nickname from being used, or alternatively, a particular IP address from connecting to the server, as well as a whole host of other options.


Note that you should choose a nickname or an IP, not both, for an individual ban block.


```
ban nick/ip {
	mask ["nickname*goes*here"/12.223.98.1]; // Banned nick or IP
	reason "Put your reason for the ban here";
};

```


Or just comment it out:


```
/*
ban nick {
        mask "*C*h*a*n*S*e*r*v*";
        reason "Reserved for Services";
};
*/

```


Note that you can also use an actual IRC hostmask (user@host.name) in the mask field instead of a nick or IP in order to ban a particular user–just be sure to specify ban user instead of ban ip or ban nick.


Finally, you can also ban a particular realname from being used. When users first set up their clients, they are able to specify their “real name” in their user settings. This setting applies to those. For example, to ban all users named Jack:


```
ban realname {
	mask "Jack";
	reason "Go away!";
};

```


Note: The above ban is somewhat useless, considering users can just change their realname in the client settings, but it’s here for completeness.


You can also set up ban exceptions, such as the following:


```
except ban {
	// My username is sigtau--don't ban me!
	mask	*sigtau*@*
};

```


Now that we’ve had a look at ban {} blocks, which are used against users, let’s have a look at another type of block that can be used against other types of malicious activity.


## deny {} — Restricting Non-User Activity


The default deny {} block settings can be left as-is for most servers.


When IRC was a highly mainstream protocol in the 80’s, 90’s, and early 2000s, another additional protocol was in demand for users that wanted to communicate directly over IRC without a middleman centralized server.  The DCC (direct client-to-client) protocol was born, allowing users to do things that normal IRC was incapable of, such as sending chat messages directly to one another, as well as file sharing.


The file sharing aspect opened a slew of security holes, however, as DCC grew in popularity among users who were not savvy enough to discern files that might be harmful or malicious in nature. Thus, some IRC daemons started restricting certain kinds of DCC requests to prevent this kind of activity.


UnrealIRCd is among these IRC daemons.  The deny block is capable of doing a host of different restriction tasks, including blocking these types of file transfers.  For example, in the sample configuration, you’ll find this block:


```
deny dcc {
	filename "*sub7*"; // note that this supports wildcards
	reason "Possible Sub7 Virus";
};

```


This causes any DCC requests sent over the server containing the string “sub7” (based on the infamous Sub7 trojan from earlier IRC days) to be disallowed.


A deny block can also prevent users from joining or creating certain types of channels.  Remember earlier where we allowed users to join a channel called #WarezSucks?  This is the deny block that we were counteracting:


```
deny channel {
	channel "*warez*";
	reason "Warez is illegal";
	class "clients";
};

```


This prevents any user from creating a channel with the word “warez” in the name.


## vhost {} — Hiding User Hostnames


Most servers can use the default vhost {} block.


While some popular IRC service packages allow you to do this without editing the configuration file, the old fashioned way of hiding users’ hostnames from one another was automatic and did not require users to have a registered nickname.


This is useful for users who connect via a bouncer but use nicknames they do not wish to register, or if your server does not have services to begin with.


The optional vhost block sets a fake hostname for the specified user. We’ll take a look at the one preset in the sample configuration:


```
vhost {
	vhost		i.hate.microsefrs.com;
	from {
		userhost *@*.image.dk;
	};
	login		stskeeps;
	password	moocowsrulemyworld;
};

```


This allows users connecting from the hostname *@*.image.dk to set their hostname to *@i.hate.microsefrs.com. Users typically do this using the /CHGHOST command.


Use of this command has waned considerably in favor of using IRC services packages that perform the same functionality, so for most purposes, you do not need a thorough understanding of how this block works in order to set up a properly functional IRC network.


## set {} — Network Configuration


This section is absolutely required! The IRC daemon will not start unless you properly configure this section!


If you’ve been following along, the next block should one of the last, and ironically, one of the most important: the set {} block.


The set {} block contains a ton of configuration options the server will need to operate. Below is a copy of the example configuration as you will see it, with added in-line comments that explain what each option does. You don’t need to copy the comments into the configuration file unless you find them useful.


Most if not all of these options are required, so simply edit them to your liking instead of removing them wholesale. Note that if you plan to link together multiple servers, it’s best that you keep these consistent and identical across all servers.  If you do not plan to link servers, these options must still be configured.


Most of these options just involve replacing the example domain name with your own domain name.


Note that you will have to generate three random values for the three cloak-keys.


```
set {
	network-name "ROXnet"; // This is the name of the network reported to clients
			       // upon joining, and in many popular IRC clients, is the
			       // formal name used for tabbed view and logs.
	default-server "irc.roxnet.org"; // Replace this with your server's domain--the
					 // daemon reports this to the client as being
					 // the network's proper hostname.
	services-server "services.roxnet.org"; // If you plan to add services to your
					       // IRC server, use the same domain as
					       // above, e.g. services.mydomain.com.
					       // Even if you aren't going to run
					       // services, you should do this anyway.
	stats-server "stats.roxnet.org"; // Same as the above, except with stats
					 // stats servers.
 	help-channel "#ROXnet"; // Set this to a be your network's official "help
				// channel" so that users have a place to go in case
				// they need help with something that only an IRCop
				// or server administrator could do.
	hiddenhost-prefix "rox"; // For anonimity purposes, UnrealIRCd masks users'
				 // IP addresses using 'hidden hosts'--i.e. the host
				 // user331.newyork.someISP.com is converted to
				 // hiddenhost-3AB66E2-newyork.someISP.com.  This value
				 // can be customized--change "rox" to whatever you
				 // want (generally, alphanumeric lowercase) the hidden
				 // host mask to be.  This is sometimes referred to as
				 // "cloaking."
	cloak-keys {
		"brespumuste6ewR";
		"fUkAChekEswu6ed";
		"4aFachehetawedR";
	};
	
	// The above block are the random seeds used to generate unique cloaks for
	// each user without revealing their hostname to one another.  Generally, a
	// 10-20 character random alphanumeric mixed-case string will do.  Any online
	// password generator will work, or if you're in a pinch, just type whatever
	// alphanumeric string you want in all three string fields.
	//
	// THERE MUST BE EXACTLY THREE STRINGS.

	hosts {
		local		"locops.roxnet.org";
		global		"ircops.roxnet.org";
		coadmin		"admin.roxnet.org";
		admin		"admin.roxnet.org";
		servicesadmin	"csops.roxnet.org";
		netadmin	"netadmin.roxnet.org";
		host-on-oper-up "no";
	};
	
	// The above block can go mostly unchanged, apart from changing "roxnet.org"
	// to your domain of choice.  Setting host-on-oper-up automatically changes
	// the hostname of the IRCop user to be one of the above based on their rank
	// indicators in their o:line flags (`local`, `global`, `admin`, etc.).
};

```


The final block in the configuration should be yet another set {} block, but this time, the options are exclusive to this server instance instead of being consistent across the whole network.


You must set the kline-address email or the server will not start.


The rest of these settings can be left as default for most servers.


```
set {
	kline-address "contact@example.com"; // When k-lines are applied, users are sometimes
					// instructed to contact a particular e-mail for
					// more information.  This should be that e-mail.
	modes-on-connect "+ixw"; // Leave these at default; however if you need to tweak
				 // what modes users are automatically assigned upon
				 // connecting, you can view the full mode reference
				 // here: http://www.unrealircd.com/files/docs/unreal32docs.html#userchannelmodes
	modes-on-oper "+xwgs";   // These are modes that are applied when a user first
				 // authenticates as an IRCop.  These are generally left
				 // alone as well, but you can use the above link again
				 // if you, for some reason, need to tweak these.
	oper-auto-join "#opers"; // This is the channel that IRCops are set to
				 // be force-joined to as soon as they authenticate as
				 // an IRCop.  You can set multiple channels by
				 // delimiting them with commas: #chan1,#chan2,...
	maxchannelsperuser 10; // This is the maximum number of channels any one user
			       // session can be joined to.  You may raise or lower this
			       // as needed, however it must be greater than or equal
			       // to 1.
	options {
		hide-ulines; // This means any server specified in the ulines {} block
			     // will not appear when a connected user uses /list.
			     // IRCops automatically override this when they use /list.
		show-connect-info; // This allows a user to see messages such as
				   // "Looking up your hostname..." and "Requesting
				   // identd..." when they are connecting.  Generally a
				   // good idea to keep this.
	};
	
	anti-spam-quit-message-time 10s; // In the past, IRC servers have had issues
					 // with users joining and quitting with long
					 // quit messages in rapid succession to spam
					 // the server.  This helps combat that by
					 // specifying how long a user must be 
					 // connected before the server will allow them
					 // to specify a message when using /QUIT.
					 // You can leave this as it is or change it.
	oper-only-stats "okfGsMRUEelLCXzdD"; // You should probably leave this alone,
					     // it defines which server stats are
					     // restricted to be viewable by IRCops only.
					     // Since there's too many stat modes to list
					     // for an article, here's a hint: once your
					     // server is up and running, use the /stats
					     // command (while authenticated as an IRCop)
					     // to get a list of character->stat pairs,
					     // which can be used to modify this list.
	throttle { // When users disconnect, their clients will try to reconnect them.
		   // This prevents it from happening too quickly, potentially putting
		   // the server under high load.
		connections 3; // The number of connections allowed...
		period 60s; // ...per x seconds, as specified here.
	};

	anti-flood { // This configures anti-flood settings to prevent users from
		     // flooding the server with unnecessary commands.
		nick-flood 3:60; // Allows 3 nick changes per 60 seconds.

		// Some other useful options to read up on are away-flood for how
		// many times a user can use /AWAY, or unknown-flood-amount for
		// users that send unknown garbage data to the server.
	};
	
	spamfilter { // This controls settings for spam filtering and default ban info.
		ban-time 1d; // This is the default duration of a k-line/z-line ban when
			     // automatically set by a spam filter.
		ban-reason "Spam/Advertising" // The message used by automatic bans.
		virus-help-channel "#help"; // The channel users can turn to if they
					    // need help with viruses such as those
					    // sent over DCC.
	};
};

```


That’s it!


Save your changes to the configuration file, ensure that it is saved as unrealircd.conf. Press CTRL-x, then y, then Enter.


Congrats, you’re done editing the configuration file! Let’s complete the few final steps before we can start the server.


# Step Six — Creating ircd.rules and ircd.motd


Two files need to be created before UnrealIRCd will boot: ircd.rules and ircd.motd. At your shell prompt, type the following to create the file for your server rules:


```
nano ~/Unreal3.2.10.4/ircd.rules

```


Now, you can type your server rules as plain text. Users will be presented with these upon typing the /RULES command.  Do keep in mind that you should keep the lines in this file less than 80 characters long to prevent them from getting automatically wrapped by the IRCd (if you use indentation, this can be a pain).


Save your changes. Create the message of the day file:


```
nano ~/Unreal3.2.10.4/ircd.motd

```


This is where you can type your MOTD, which will be shown the moment users connect to the server. The same 80-character line length limit applies here as well. Useful information to put in this file would be featured channels, a quick breakdown of server rules, the current IRCops, and so on. You can put whatever you want here in plain text.


Save your changes.


If you listed any other files in your configuration, create them now, too.


# Step Seven — IRCd Log File


To ensure that the IRCd has a place to store its logs, let’s go ahead and create a blank file specifically for that.


```
touch ~/Unreal3.2.10.4/ircd.log

```


# Step Eight — Starting UnrealIRCd


Congratulations! You’ve successfully set up your own IRC daemon.


Move to the Unreal directory:


```
cd ~/Unreal3.2.10.4

```


Start the chat server:


```
./unreal start

```


You should see a message like this:


```
* Loading IRCd configuration ..
* Configuration loaded without any problems ..
* Initializing SSL.
* Dynamic configuration initialized .. booting IRCd.

```


If there’s a startup error, check the line number and setting mentioned in the error. You should be able to locate the problem in your configuration file.


Once the server starts, you will be able to connect to your server via your favorite IRC client. I recommend you do so on your local machine rather than on a Droplet, to ensure other users can connect.


# Step Nine — Connecting an IRC Client


In your favorite IRC client, type:


```
/server your Droplet IP address or domain here 6667

```


You’ll need your domain or IP address, and the port number (6667 in this example). If you set a global password for the server in the allow {} block, you’ll need to use that, too.


A few popular IRC clients for Windows are mIRC, HexChat, and Pidgin.  If you are on Linux, X-Chat is freely available.  For OS X, X-Chat Azure and a few paid applications on the App Store are available.


# Step Ten — Operator Authentication


Once connected, you will need to authenticate as an IRCop.  To authenticate as an operator, type:


```
/oper your-oline-username your-oline-password

```


These are the credentials you set in the oper {} block.


You should then be force-joined to the channels we previously specified in oper-auto-join, indicating that you are now an IRC operator.


# A Note about Connecting via SSL


Because the server instance was set up using a self-signed SSL certificate, some clients may not allow you to connect via SSL due to an “invalid” or “untrusted” certificate.  It is recommended that you seek a means of generating a valid SSL certificate with a free service such as CACert.org or a paid certificate authority (CA) of your choice.


Enjoy your new IRC server! From here, you can now set up services or start inviting users. Happy chatting!


