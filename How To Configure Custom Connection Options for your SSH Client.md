# How To Configure Custom Connection Options for your SSH Client

```Linux Basics``` ```System Tools```

## Introduction


SSH, or Secure Shell, is the most common way of connecting to Linux servers for remote administration. Although connecting to a single server via the command line is relatively straightforward, there are many workflow optimizations for connecting to multiple remote systems.


OpenSSH, the most commonly used command-line SSH client on most systems, allows you to provide customized connection options. These can be saved to a configuration file that contains different options per server. This can help keep the different connection options you use for each host separated and organized, and avoids having to provide extensive options on the command line whenever you need to connect.


In this guide, we’ll cover the structure of the SSH client configuration file, and go over some common options.


# Prerequisites


To complete this guide, you will need a working knowledge of SSH and some of the options that you can provide when connecting. You should also configure SSH key-based authentication for some of your users or servers, at least for testing purposes.


# The SSH Config File Structure and Interpretation Algorithm


Each user on your system can maintain their own SSH configuration file within their home directory. These can contain any options that you would use on the command line to specify connection parameters. It is always possible to override the values defined in the configuration file at the time of the connection by adding additional flags to the ssh command.


## The Location of the SSH Client Config File


The client-side configuration file is located at ~/.ssh/config – the ~ is a universal shortcut to your home directory. Often, this file is not created by default, so you may need to create it yourself. The touch command will create it if it does not exist (and update the last modified timestamp if it does).


```
touch ~/.ssh/config


```


## Configuration File Structure


The config file is organized by hosts, i.e., by remote servers. Each host definition can define connection options for the specific matching host. Wildcards are also supported for options that should have a broader scope.


Each of the sections starts with a header defining the hosts that should match the configuration options that follow. The specific configuration items for that matching host are then defined below. Only items that differ from the default values need to be specified, as each entry will inherit the defaults for any undefined items. Each section spans from one Host header to the following Host header.


Typically, for organizational purposes and readability, the options being set for each host are indented. This is not a hard requirement, but a useful convention that allows for interpretation at a glance.


The general format will look something like this:


~/.ssh/config
```
Host firsthost
    Hostname your-server.com
    User username-to-connect-as
    IdentityFile /path/to/non/default/keys.pem

Host secondhost
    ANOTHER_OPTION custom_value

Host *host
    ANOTHER_OPTION custom_value

Host *
    CHANGE_DEFAULT custom_value

```


Here, we have four sections that will be applied on each connection attempt depending on whether the host in question matches.


## Interpretation Algorithm


It is important to understand the way that SSH will interpret the file to apply the configuration values. This has implications when using wildcards and the Host * generic host definition.


SSH will match the host name provided on the command line with each of the Host headers that define configuration sections.


For example, consider this definition:


~/.ssh/config
```
Host devel
    HostName devel.example.com
    User tom

```


This host allows us to connect as tom@devel.example.com by typing this on the command line:


```
ssh devel


```


SSH starts at the top of the config file and checks each Host definition to see if it matches the value given on the command line. When the first matching Host definition is found, each of the associated SSH options are applied to the upcoming connection.


SSH then moves down the file, checking to see if other Host definitions also match. If another definition is found that matches the current hostname given on the command line, it will consider the SSH options associated with the new section. It will then apply any SSH options defined for the new section that have not already been defined by previous sections.


This is an important point. SSH will interpret each of the Host sections that match the hostname given on the command line, in order. During this process, it will always use the first value given for each option. There is no way to override a value that has already been given by a previously matched section.


This means that your config file should follow the rule of having the most specific configurations at the top. More general definitions should come later on in order to apply options that were not defined by the previous matching sections.


Let’s look again at the example from the previous section:


~/.ssh/config
```
Host firsthost
    Hostname your-server.com
    User username-to-connect-as
    IdentityFile /path/to/non/default/keys.pem

Host secondhost
    ANOTHER_OPTION custom_value

Host *host
    ANOTHER_OPTION custom_value

Host *
    CHANGE_DEFAULT custom_value

```


Here, we can see that the first two sections are defined by literal hostnames (or aliases), meaning that they do not use any wildcards. If we connect using ssh firsthost, the very first section will be the first to be applied. This will set Hostname, User, and IdentityFile for this connection.


It will check the second section and find that it does not match and move on. It will then find the third section and find that it matches. It will check ANOTHER_OPTION to see if it already has a value for that from previous sections. Finding that it doesn’t, it will apply the value from this section. It will then match the last section since the Host * definition matches every connection. Since it doesn’t have a value for the mock CHANGE_DEFAULT option from other sections, it will take the value from this section. The connection is then made with the options collected from this process.


Let’s try this again, pretending to call ssh secondhost from the command line.


Again, it will start at the first section and check whether it matches. Since this matches only a connection to firsthost, it will skip this section. It will move on to the second section. Upon finding that this section matches the request, it will collect the value of ANOTHER_OPTION for this connection.


SSH then looks at the third definition and find that the wildcard matches the current connection. It will then check whether it already has a value for ANOTHER_OPTION. Since this option was defined in the second section, which was already matched, the value from the third section is dropped and has no effect.


SSH then checks the fourth section and applies the options within that have not been defined by previously matched sections. It then attempts the connection using the values it has gathered.


# Connection Options


Now that you have an idea about how to write your configuration file, let’s discuss some common options and the format to use to specify them on the command line.


The first ones we will cover are the minimum settings necessary to connect to a remote host. Namely, the hostname, username, and port that the SSH server is running on.


To connect as a user named apollo to a host called example.com that runs its SSH daemon on port 4567 from the command line, you could run ssh like this:


```
ssh -p 4567 apollo@example.com

```


However, you could also use the full option names with the -o flag, like this:


```
ssh -o "User=apollo" -o "Port=4567" -o "HostName=example.com"

```


You can find a full list of available options in the SSH manual page.


To set these in your config file, you have to choose a Host header name, like home:


```
Host home
    HostName example.com
    User apollo
    Port 4567

```


# Common SSH Configuration Options


So far, we have discussed some of the options necessary to establish a connection. We have covered these options:


- HostName: The actual hostname that should be used to establish the connection. This replaces any alias defined in the Host header. This option is not necessary if the Host definition specifies the actual valid address to connect to.
- User: The username to be used for the connection.
- Port: The port that the remote SSH daemon is running on. This option is only necessary if the remote SSH instance is not running on the default port 22.

There are many other useful options worth exploring. We will discuss some of the more common options, separated according to function.


## General Tweaks and Connection Items


- 
ServerAliveInterval: This option can be configured to let SSH know when to send a packet to test for a response from the server. This can be useful if your connection is unreliable and you want to know if it is still available.

- 
LogLevel: This configures the level of detail in which SSH will log on the client-side. This can be used for turning off logging in certain situations or increasing the verbosity when trying to debug. From least to most verbose, the levels are QUIET, FATAL, ERROR, INFO, VERBOSE, DEBUG1, DEBUG2, and DEBUG3.

- 
StrictHostKeyChecking: This option configures whether ssh SSH will ever automatically add hosts to the ~/.ssh/known_hosts file. By default, this will be set to “ask” meaning that it will warn you if the Host Key received from the remote server does not match the one found in the known_hosts file. If you are constantly connecting to a large number of ephemeral hosts (such as testing servers), you may want to turn this to “no”. SSH will then automatically add any hosts to the file. This can have security implications if your known hosts ever do change addresses when they shouldn’t, so think carefully before enabling it.

- 
UserKnownHostsFile: This option specifies the location where SSH will store the information about hosts it has connected to. Usually you do not have to worry about this setting, but you may wish to set this to /dev/null so they are discarded if you have turned off strict host checking above.

- 
VisualHostKey: This option can tell SSH to display an ASCII representation of the remote host’s key upon connection. Turning this on can be a useful way to get familiar with your host’s key, allowing you to recognize it if you have to connect from a different computer sometime in the future.

- 
Compression: Turning compression on can be helpful for very slow connections. Most users will not need this.


With the above configuration items in mind, we could make a number of useful configuration tweaks.


For instance, if we are creating and destroying hosts very quickly at a cloud provider, something like this may be useful:


```
Host home
    VisualHostKey yes

Host cloud*
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
    LogLevel QUIET

Host *
    StrictHostKeyChecking ask
    UserKnownHostsFile ~/.ssh/known_hosts
    LogLevel INFO
    ServerAliveInterval 120

```


This will turn on your visual host key for your home connection, allowing you to become familiar with it so you can recognize if it changes or when connecting from a different machine. We have also set up any host that begins with cloud* to not check hosts and not log failures. For other hosts, we have sane fallback values.


## Connection Forwarding


One common use of SSH is forwarding connections, either allowing a local connection to tunnel through the remote host, or allowing the remote machine access to tunnel through the local machine. This is sometimes necessary when you need to connect to a remote machine behind a firewall through a separate, designated “gateway” server. SSH can also do dynamic forwarding using protocols like SOCKS5 which include the forwarding information for the remote host.


The options that control this behavior are:


- 
LocalForward: This option is used to specify a connection that will forward a local port’s traffic to the remote machine, tunneling it out into the remote network. The first argument should be the local port you wish to direct traffic to and the second argument should be the address and port that you wish to direct that traffic to on the remote end.

- 
RemoteForward: This option is used to define a remote port where traffic can be directed to in order to tunnel out of the local machine. The first argument should be the remote port where traffic will be directed on the remote system. The second argument should be the address and port to point the traffic to when it arrives on the local system.

- 
DynamicForward: This is used to configure a local port that can be used with a dynamic forwarding protocol like SOCKS5. Traffic using the dynamic forwarding protocol can then be directed at this port on the local machine and on the remote end, it will be routed according to the included values.


These options can be used to forward ports in both directions, as you can see here:


```
# This will allow us to use port 8080 on the local machine
# in order to access example.com at port 80 from the remote machine
Host local_to_remote
    LocalForward 8080 example.com:80

# This will allow us to offer access to internal.com at port 443
# to the remote machine through port 7777 on the other side
Host remote_to_local
    RemoteForward 7777 internal.com:443

```


This is especially useful when you need to open a browser window to a private dashboard or another web application running on a server that is not directly accessible other than over SSH.


## Other Forwarding


Along with connection forwarding, SSH allows other types of forwarding as well.


You can forward any SSH keys stored in an agent on your local machine, allowing us to connect from the remote system using credentials stored on your local system. You can also start applications on a remote system and forward the graphical display to our local system using X11 forwarding. X11 is a Linux display server and is not very intuitive to use without a Linux desktop system, but can be very useful if you are using both a remote and a local Linux environment.


These are the directives that are associated with these capabilities:


- 
ForwardAgent: This option allows authentication keys stored on our local machine to be forwarded onto the system you are connecting to. This can allow you to hop from host-to-host using your home keys.

- 
ForwardX11: If you want to be able to forward a graphical screen of an application running on the remote system, you can turn this option on.


## Specifying Keys


If you have SSH keys configured for your hosts, these options can help you manage which keys to use for each host.


- 
IdentityFile: This option can be used to specify the location of the key to use for each host. SSH will use keys located in ~/.ssh by default, but if you have keys assigned per-server, this can be used to specify the exact path where they can be found.

- 
IdentitiesOnly: This option can be used to force SSH to only rely on the identities provided in the config file. This may be necessary if an SSH agent has alternative keys in memory that are not valid for the host in question.


These options are especially useful if you have to keep track of a large number of keys for different hosts and use one or more SSH agents to assist.


# Conclusion


As long as you keep in mind the way that SSH will interpret the values, you can establish rich sets of specific values with reasonable fall backs.


If you ever have to use SSH over a very poor or intermittent connection such as airplane Wi-Fi, you can also try using mosh, which is designed to make SSH work under adverse circumstances.


