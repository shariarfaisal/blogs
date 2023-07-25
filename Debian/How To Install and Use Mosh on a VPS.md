# How To Install and Use Mosh on a VPS

```Ubuntu``` ```Miscellaneous``` ```Debian``` ```Fedora```

## Introduction


SSH is, without a doubt, the de facto method for remote server administration. However, its dominance doesn't mean it's without its own nuances under certain circumstances. If you have ever tried to maintain an SSH connection while on the move with a mobile connection, you'll appreciate this sentiment.


Mosh takes all the security benefits of SSH and builds upon it a greater tolerance to poor network conditions and roaming connections. It also increases responsiveness and lowers bandwidth usage by only communicating state changes to the currently visible screen region, rather than transmitting complete buffers.


Connection initiation and authentication with Mosh occurs through a regular SSH connection, meaning only a few extra configurations are required and any current key-based security mechanisms will work flawlessly. Once authenticated, a key is negotiated and Mosh switches to communicating through encrypted UDP datagrams, making the session more resilient to the changing client IPs and connection dropouts that can be common with mobile connections.


These benefits make Mosh a great option to have installed on your VPS for those situations when you need to perform a task while on the move.


# Installation


To get started, Mosh must first be installed on both the client and the server. Fortunately, Mosh packages exist on most popular distributions and below are the installation methods for some of the distributions offered on DigitalOcean.


On Ubuntu:


```
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:keithw/mosh
sudo apt-get update
sudo apt-get install mosh
```


On Debian:


```
sudo apt-get install mosh
```


On Arch Linux:


```
pacman -S mosh
```


On Fedora:


```
sudo yum install mosh
```


For any other OS, such as OSX or Windows, please consult the Mosh documentation to find the most relevant installation method.


# Firewall Configuration


If you have a firewall configured on your VPS (recommended), you will also need to open the extra ports Mosh requires.


If you are using iptables directly, the following command will open the ports that Mosh requires:


```
sudo iptables -I INPUT 1 -p udp --dport 60000:61000 -j ACCEPT
```


Remember that, by default, this firewall setting will not be retained after a system reboot. Solutions such as iptables-persistant exist to augment this behavior.


If you are using UFW, you can open the ports with the following:


```
sudo ufw allow 60000:61000/udp
```


If you are using any other program to manage your firewall, then you will need to manually ensure to open the UDP ports from 60000 to 61000. However, if you only expect to have a small number of concurrent connections, then a smaller range of ports can be opened provided it begins at port 60000 (e.g 60000:60020).


# Usage


In most use cases, Mosh is a drop-in replacement for SSH, meaning many SSH commands need only a simple alteration. For example:


```
ssh user@example.com

# Becomes:

mosh user@example.com
```


However, if you use any other arguments with SSH (such as -p), then a slightly different syntax is needed:


```
mosh --ssh="ssh -p 22000" user@example.com
```


Once executed, Mosh will connect you to a shell that appears to look much like any standard SSH connection. However, beneath the surface, Mosh is much more than a dumb pipe, with a number of unique features that make it perform better on poor connections.


While SSH will transmit the complete data output from any program running on the remote machine, Mosh runs a vt500 state machine on each end of the connection and will only communicate changes to the currently visible screen area. This allows it to radically reduce the bandwidth usage and maintain responsiveness, both of which can be bottlenecks on mobile connections.


In the event your connection goes down completely, Mosh will quickly let you know with a status bar the top of the window indicating the time since the last successful communication.


![](https://assets.digitalocean.com/articles/mosh/img1.png)
Once the connection is restored, Mosh will resynchronize automatically and you can continue where you left off with your session.


You may also notice that, even when your connection is slow or unresponsive, you can type new commands into your terminal and see your input appear immediately with an underline. Such underlined text indicates that Mosh has speculated what the remote terminal state will look like before seeing a response from the VPS. Once the underlining disappears, you can be sure that both ends of the connection are in sync.


However, this system does have one downside: due to Mosh's synchronization of only the current screen state, your local terminal will not maintain a scrollback buffer of previous program output. Hence, it is recommended that you use a terminal multiplexer such as screen or tmux on the VPS side to retain this output.


# Summary


This introduction to Mosh has highlighted some of its key benefits over SSH on mobile connections and, while it may not replace your daily use of SSH, in situations where you are forced to rely on a slow connection, it can make the difference between being frustrated and being productive.


