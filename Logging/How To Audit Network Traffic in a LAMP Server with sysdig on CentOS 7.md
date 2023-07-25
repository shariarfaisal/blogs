# How To Audit Network Traffic in a LAMP Server with sysdig on CentOS 7

```Linux Basics``` ```Logging``` ```System Tools``` ```CentOS```

# Introduction


sysdig is a brand new system-level exploration and troubleshooting tool that combines the benefits of well-known utilities such as strace, tcpdump, and lsof into one single application. And, as if this were not enough, sysdig also provides the capabilities to save system activity to trace files for later analysis.


In addition, a rich library of scripts (called chisels) is provided along with the installation in order to help you solve common problems or meet monitoring needs, from displaying failed disk I/O operations to finding the files where a given process has spent most time, and everything in between. You can also write your own scripts to enhance sysdig even further according to your needs.


In this article we will first introduce basic sysdig usage, and then explore network analysis with sysdig, including an example of auditing network traffic on a CentOS 7 LAMP server. Please note that the VPS used in the examples has not been placed under significant load, but it is enough for showing the basics of the present auditing tasks.


## Prerequisites


Before you begin, please make sure you have these prerequisites.


- CentOS 7 Droplet
- Set up a LAMP server on your CentOS 7 VPS. Please refer to this article for instructions
- In addition, you should have a non-root user account with sudo access that will be used to run sysdig

## Installing sysdig


Log in to your server and follow these steps:


# Step 1 — Trust the Draios GPG key


Draios is the firm behind sysdig.


Before proceeding with the installation itself, yum will use this key to verify the authenticity of the package you’re about to download.


To manually add the Draios key to your RPM keyring, use the rpm tool with the --import flag:


```
sudo rpm --import https://s3.amazonaws.com/download.draios.com/DRAIOS-GPG-KEY.public  

```


Then, download the Draios repository and configure yum to use it:


```
sudo curl -s -o /etc/yum.repos.d/draios.repo http://download.draios.com/stable/rpm/draios.repo

```


# Step 2 — Enable the EPEL Repository


Extra Packages for Enterprise Linux (EPEL) is a repository of high-quality free and open-source software maintained by the Fedora project and is 100% compatible with its spinoffs, such as Red Hat Enterprise Linux and CentOS. This repository is needed in order to download the Dynamic Kernel Module Support (DKMS) package, which is needed by sysdig, and to download other dependencies.


```
sudo yum -y install epel-release

```


# Step 3 — Install Kernel Headers


This is needed because sysdig will need to build a customized kernel module (named sysdig-probe) and use it to operate.


```
sudo yum -y install kernel-devel-$(uname -r)

```


# Step 4 — Install the sysdig package


Now we can install sysdig.


```
sudo yum -y install sysdig

```


# Step 5 — Run sysdig as a Non-root User


For security, it’s best to have a non-root user to run sysdig. Create a custom group for sysdig:


```
sudo groupadd sysdig

```


Add one or more users to the group. In our example, we’ll add the user sammy.


```
sudo usermod -aG sysdig sammy

```


Locate the binary file for sysdig:


```
which sysdig  

```


You might receive a response like


```
/usr/bin/sysdig

```


Give all members of the sysdig group privileges to run the sysdig executable (and that binary only). Edit /etc/sudoers with:


```
sudo visudo

```


Add the following lines for the sysdig group in the groups section. Adding the new lines after the %wheel section is fine. Replace the path with sysdig’s location on your system:


```
## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

## sysdig
%sysdig ALL= /usr/bin/sysdig 

```


If you need further clarifications on editing the /etc/sudoers file, it is recommended that you take a look at this article.


# Running sysdig


You can run sysdig in two modes.


You can view the the real-time stream of server activity live, or you can save records of system operations to a file for later offline analysis.


Since you will most likely want to use the second option, that is what we will cover here. Note that when saving system activity to a file, sysdig takes a full snapshot of the operating system, so that everything that happens on your server during that interval of time will be available for offline analysis.



Note: When you run sysdig commands, please make sure that each option is preceded by a single short dash. Copying and pasting may cause an issue where a single dash is pasted as a long dash and therefore not recognized by the program.

Let’s run a basic sysdig command to capture 1000 lines of server activity.


To capture system activity to a file named act1.scap, and limit the output to 1000 events, run the following command (omit the -n 1000 part if you want to run sysdig for an unspecified period of time). The -z switch is used to enable compression of the trace file.


```
sudo sysdig -w act1.scap.gz -n 1000 -z

```



Note: If you omitted the -n switch in the last step, you can interrupt the execution of sysdig by pressing the CTRL + C key combination.

# Chisels — An Overview of sysdig Scripts


Chisels are sysdig scripts. To display a list of the available chisels, we need to run the following command:


```
sudo sysdig -cl

```


In order to audit the network traffic on our CentOS 7 LAMP server using the trace file created by sysdig, we will use the chisels available under the Net category:


```
Category: Net
-------------
iobytes_net         Show total network I/O bytes
spy_ip              Show the data exchanged with the given IP address
spy_port            Show the data exchanged using the given IP port number
topconns            top network connections by total bytes
topports_server     Top TCP/UDP server ports by R+W bytes
topprocs_net        Top processes by network I/O

```


Further description of a specific chisel, along with instructions for its use, can be viewed with:


```
sudo sysdig -i chisel name

```


For example:


```
sudo sysdig -i spy_ip

```


This outputs:


```
Category: Net
-------------
spy_ip              Show the data exchanged with the given IP address
shows the network payloads exchanged with an IP endpoint. You can combine this chisel with the -x, -X or -A sysdig command line switches to customize the screen output
Args:
[ipv4] host_ip - the remote host IP address
[string] disable_color - Set to 'disable_colors' if you want to disable color output

```


The Args section indicates whether you need to pass an argument to the chisel or not. In the case of spy_ip, you need to pass an IP address as an argument to the chisel.


# Auditing Network Traffic (Practical Example)


Let’s walk through a practical example of how to use sysdig to analyze bandwidth use and see detailed information about network traffic.


To get the best results from this test, you will need to set up a dummy web form on your server so appropriate traffic is generated. If this is a server with a fresh LAMP installation, you can make this form at /var/www/html/index.php.


```
<body>
<form id="loginForm" name="loginForm" method="post" action="login.php">
  <table width="300" border="0" align="center" cellpadding="2" cellspacing="0">
    <tr>
      <td width="112"><b>Username:</b></td>
      <td width="188"><input name="login" type="text" class="textfield" id="login" /></td>
    </tr>
    <tr>
      <td><b>Password:</b></td>
      <td><input name="pass" type="password" class="textfield" id="pass" /></td>
    </tr>
    <tr>
      <td>&nbsp;</td>
      <td><br />
      <input type="submit" name="Submit" value="Login" /></td></tr>
  </table>
</form>
</body>

```


This isn’t required, but to make everything tidy, you can also create the /var/www/html/login.php page:


```
<body>
	<p>Form submitted.</p>
</body>

```



Warning: Please delete this form when you are done testing!

# Step 1 — Saving Live Data for Offline Analysis


We will starting capturing our log collection of data by issuing the following command:


```
sudo sysdig -w act1.scap.gz -z -s 4096

```


Leave sysdig running for a reasonable amount of time. Your command prompt will hang while sysdig runs.


Now, visit your server’s domain or IP address in your web browser. You can visit both existing and non-existing pages to generate some traffic. If you want this specific example to work, you should visit the home page, fill out the login information with anything you like, and submit the login form a few times. In addition, feel free to run queries to your MySQL/MariaDB database as well.


Once you’ve generated some traffic, press CTRL + C to stop sysdig. Then you will be ready to run the analysis queries that we will discuss later in this tutorial.


In a production environment, you could start the sysdig data collection during a busy time on your server.


## Understanding Filters: Classes and Fields


Before we get into sorting the sysdig data, let’s explain some basic sysdig command elements.


Sysdig provides classes and fields as filters. You can think of classes as objects and fields as properties, following an analogy based on object-oriented programming theory.


You can display the complete list of classes and fields with:


```
sudo sysdig -l

```


We will use classes and fields to filter output when analyzing a trace file.


# Step 2 — Performing Offline Analysis Using Trace Files


Since we want to audit the network traffic to and from our LAMP server, we will load the trace file act1.scap.gz and perform the following tests with sysdig:


## Displaying the list of top processes using network bandwidth


```
sudo sysdig -r act1.scap.gz -c topprocs_net

```


You should see output somewhat like this:


```
Bytes     Process
------------------------------
331.68KB  httpd
24.14KB   sshd
4.48KB    mysqld

```


Here you can see that Apache is using the most bandwidth (the httpd process).


Based on this output, you can make an informed and supported judgment call to decide whether you need to increase your available bandwidth in order to serve your current and future estimated requests. Otherwise, you may want to place appropriate restrictions on the maximum rate of already available bandwidth that can be used by a process.


## Displaying network usage by process


We may also want to know which IPs are using the network bandwidth consumed by httpd, as shown in the previous example.


To that purpose, we will use the topconns chisel (which shows the top network connections by total bytes) and add a filter formed with the class proc and field name to filter results to show only http connections. In other words, the following command:


```
sudo sysdig -r act1.scap.gz -c topconns proc.name=httpd

```


This will return the top network connections to your server, including the source, where the process serving the request is httpd.


```
Bytes     Proto     Conn 	 
------------------------------
56.24KB   tcp   	111.111.111.111:12574->your_server_ip:80
51.94KB   tcp   	111.111.111.111:15249->your_server_ip:80
51.57KB   tcp   	111.111.111.111:27832->your_server_ip:80
51.26KB   tcp   	111.111.222.222:42487->your_server_ip:80
48.20KB   tcp   	111.111.222.222:42483->your_server_ip:80
48.20KB   tcp   	111.111.222.222:42493->your_server_ip:80
4.17KB	  tcp   	111.111.111.111:13879->your_server_ip:80
3.14KB	  tcp   	111.111.111.111:27873->your_server_ip:80
3.06KB	  tcp   	111.111.222.222:42484->your_server_ip:80
3.06KB	  tcp   	111.111.222.222:42494->your_server_ip:80

```


Note that the original source and destination IP addresses have been obscured for privacy reasons.


This type of query can help you find top bandwidth users that are sending traffic to your server.


After looking at the output above you may be thinking that the numbers after the source IP addresses represent ports. However, that is not the case. Those numbers indicate the event numbers as recorded by sysdig.


# Step 3 — Analyzing Data Exchanged Between a Specific IP and Apache


Now we’ll examine the connections between a specific IP address and Apache in more detail.


The echo_fds chisel allows us to display the data that was read and written by processes. When combined with a specific process name and a client IP (such as proc.name=httpd and fd.cip=111.111.111.111 in this case), this chisel will show the data that was exchanged between our LAMP server and that client IP address.


In addition, using the following switches helps us to show results in a more friendly and accurate way:


- 
-s 4096: For each event, read up to 4096 bytes from its buffer (this flag can also be used to specify how many bytes of each data buffer should be saved to disk when saving live data to a trace file for offline analysis)

- 
-A:  Print only the text portion of data buffers, and echo end-of-lines (we want to only display human-readable data)


Here’s the command. Be sure to replace 111.111.111.111 with a client IP address from the previous output.


```
sudo sysdig -r act1.scap.gz -s 4096 -A -c echo_fds fd.cip=111.111.111.111 and proc.name=httpd

```


You should see quite a bit of output, depending on the number of connections made by that IP address. Here’s an example showing a 404 error:


```
GET /hi HTTP/1.1
Host: your_server_ip
Connection: keep-alive
Cache-Control: m

------ Write 426B to 111.111.111.111:39003->your_server_ip:80

HTTP/1.1 404 Not Found
Date: Tue, 02 Dec 2014 19:38:16 GMT
Server: Apache/2.4.6 (CentOS) PHP/5.4.16
Content-Length: 200
Keep-Alive: timeout=5, max=99
Connection: Keep-Alive
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC \"-//IETF//DTD HTML 2.0//EN\">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL /hi was not found on this server.</p>
</body></html>

```


This type of query can help you figure out exactly what kinds of connections were made by a top bandwidth-using IP address. For example, if you found that the IP address was reaching a certain page very frequently, you could make that page’s assets as small as possible, to reduce bandwidth use. Or, if the traffic doesn’t seem to be legitimate, you could create a new firewall rule to block bandwidth-hogging IPs.


# Step 4 — Examining Data Exchanged with an IP Address by Keyword


Depending on the server activity during the capture interval, the trace file may contain quite a lot of events and information. Thus, going through the results of the command in the previous section by hand could take an impractical amount of time. For that reason, we can look for specific words in event buffers.


Suppose we have a set of web applications running on our web server, and we want to make sure that login credentials are not being passed as plain text through forms.


Let’s add a few flags to the command used in the previous example:


```
sudo sysdig -r act1.scap.gz -A -c echo_fds fd.ip=111.111.111.111 and proc.name=httpd and evt.is_io_read=true and evt.buffer contains form

```


Here the class evt, along with field is_io_read, allow us to examine only read events (from the server’s point of view). In addition, evt.buffer allows us to search for a specific word inside the event buffer (the word is form in this case). You can change the search keyword to one that make sense for your own applications.


The following output shows that a username and password are being passed from the client to the server in plain text (thus becoming readable to anyone with enough expertise):


```
------ Read 551B from 111.111.111.111:41135->your_server_ip:80

POST /login.php HTTP/1.1
Host: your_server_ip
Connection: keep-alive
Content-Length: 35
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Origin: http://104.236.40.111
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.122 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Referer: http://104.236.40.111/
Accept-Encoding: gzip,deflate
Accept-Language: en-US,en;q=0.8

login=sammy&pass=password&Submit=Login

```


Should you find a similar security hole, notify your developer team immediately.


## Conclusion


What you can accomplish with sysdig in auditing network traffic on a LAMP server is mostly limited by one’s imagination and application requests. We’ve seen how to find top bandwidth users, examine traffic from specific IPs, and sort connections by keywords based on requests from your applications.


Should you have any further questions about the present article, or would like suggestions on how to work with sysdig in your current LAMP environment, feel free to submit your comment using the form below.


