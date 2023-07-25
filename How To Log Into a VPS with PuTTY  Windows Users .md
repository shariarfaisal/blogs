# How To Log Into a VPS with PuTTY  Windows Users 

```Linux Basics``` ```Getting Started```

## Introduction



You finally own a VPS (Virtual Private Server), but you are probably still wondering and saying: “How am I going to connect to my sever, since I have no cpanel to type my username and password into?”. Let me assure you, with Debian (what we’ll use in this tutorial) or any other Linux distribution, you could achieve and do anything a cpanel or any web panel can do and EVEN MORE.


So please, don’t get intimidated by the command line. It might look difficult at the beginning, but as soon as you read a couple of our excellent tutorials here at DigitalOcean, you will see for yourself how easy it can be.


# First Time Connecting to your VPS



Your VPS might be located thousands of miles away from you. However, with the help of a couple of programs, you can connect to it as if it were in front of you. And most importantly, all of these programs allow you to connect safely to your server through what is called ‘SSH’.


## What is SSH?



SSH (Secure Shell) is a network protocol used for secure data communication between a server and a client (You) to perform (for example: command-line login and authentication, remote command execution, and even data transfer). So in order to keep the communication between you and a server secure from the preying eyes of hackers, there are programs that implement SSH protocols mainly by using strong encryption methods to help you achieve that (Figure 1).



<p align=“center”><b>Figure 1: Basic concepts of SSH protocol</b></p>


## OpenSSH and PuTTY



To establish communication between a client and a server, you must have SSH program on each communicating end. Hence come OpenSSH and PuTTY, which are only two SSH programs from several others. OpenSSH is the most popular and most widely used SSH program that Debian comes shipped with. PuTTY is the most popular SSH program in Windows OS. In this tutorial, I will explain how to use them correctly in order to communicate securely with your server.


# Installing and Configuring PuTTY in Windows



Since OpenSSH is already installed on our server, you only need to install PuTTY before you can connect to your server. Go to PuTTY’s official download page from <a href=“http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html"target="_blank”>here</a> and download the installer file which looks like [putty-x.xx-installer.exe]. After downloading it, install as you would install any Windows program; just make sure to install it only for the current user, especially if your PC is used by many users. Once the install is finished, launch PuTTY and a configuration window will appear to you (Figure 2).



<p align=“center”><b>Figure 2: PuTTY configuration window</b></p><br>
Now, just follow the following steps in order to connect to your server for the first time:
<br><br/>1.  In the Host Name (or IP address) field, enter your droplet IP address, which you can get from either:


- The email that you have received once the droplet is created.(Figure 3)
- Your DigitalOcean control panel.(Figure 4)<br></br>


<p align=“center”><b>Figure 3: Email sample of newly created droplet</b></p>
<br><br/>



<p align=“center”><b>Figure 4: DigitalOcean control panel</b></p>
<br><br/>2.  Just make sure that the Port field shows number 22; since it’s the default port number for SSH protocol, and also the Connection type set to SSH as in (Figure 2).
<br><br/>3. In the Saved Sessions field, write a name for your session and then hit SAVE; this will save all the configurations we did earlier; so that the next time you launch PuTTY, you wouldn’t have to enter your server’s configuration and your PuTTY preferences all over again.
<br><br/>4. Finally you are ready to connect to your server, either by selecting the session name and then clicking the Open button at the bottom or simply double-clicking the session name that you have saved earlier.


# Server-Client authentication



## First: Server authentication



<br>

<p align=“center”><b>Figure 5: PuTTY Security Alert</b></p>
<br> After initiating the connection to your server as we discussed earlier, you’ll notice that the configuration window disappeared and instead a black terminal window has appeared but with a ‘Security Alert’ (Figure 5). Don’t freak out, this alert is expected to appear ONLY the first time you connect to a server that you’ve never connected to before. To make it simple for you to understand why this scary alert window appeared, I’m going to use a simple analogy: say you drove a fancy car to a 5-star hotel, then a valet asks for your car’s key in order to park it. The question is: Do you trust this guy who is asking for your fancy car’s keys? The answer to this question will determine your final decision. Your answer is going to be based (subconsciously) at least on:


- The guy’s clothing.
- The badge he is carrying and/or the hotel’s logo or name on his uniform.

Exactly the same happens when your client’s SSH program (PuTTY) connects for the first time to your server’s SSH program (OpenSSH). Now you, as a client, have a very sensitive information which is your login credentials, i.e. your Debian  server’s account username and password. So as you wouldn’t give your fancy car’s key to a total stranger, neither would PuTTY to an unknown sever.<br><br>



<p align=“center”><b>Figure 6: The 2-steps of Server-Client authentication</b></p>


<br><br/> Figure 6. above explains concisely the operations taking place in the background during sever authentication and establishment of a secure connection between the two ends:


1. PuTTY contacts the OpenSSH.
<br><br/>2. OpenSSH identifies itself to the PuTTY by sending it a Host key and some other parameters.
<br><br/>3. PuTTY, in turn, searches through its <u>known hosts database</u> to see whether the Host key that OpenSSH has sent in step 2 exits or not.
<br><br/>4. If NOT, then before terminating the current session, the security alert window (Figure 5) would appear; if YES, then proceed to step 5.


- So, a Host key to PuTTY is basically what a valet’s uniform and badge is to a car owner. It is simply a unique fingerprint to your server’s SSH program (OpenSSH) that helps PuTTY in future sessions to recognize it by.
- Back again to the ‘Security Alert’ window in (Figure 5), assuming you have entered your server’s IP address correctly, it would be safe now to click ‘Yes’. What that basically does is tell PuTTY to save your server’s Host key in its known hosts database in windows registry to be used for future authentications.

5. Finally, server authentication is completed and secure connection is established.<br><br/>


## Second: Client Authentication



Since you are finally connected securely to your sever, it is safe now to send your login credentials through the SSH encrypted connection (Figure 6). You can get your primary login credentials from the email that you have received once the droplet is created (Figure 3). The picture below shows a terminal command line where you are going to enter your username in the line that says login as:, then hit <code>Enter</code>, then a new line will appear asking for your password, type it and hit <code>Enter</code>.
<br></br> ATTENTION!



While entering your password, you will notice that the cursor in the command line is not moving or displaying any asterisk. Don’t worry– even though nothing seems to appear on the command line as you type, every key you press on your keyboard is actually being entered and sent to your server. And that’s the default behavior in all Linux distro regarding password submission.

<br><br>
<p align=“center”><b>Figure 7: The beginning of client authentication process</b></p>
<br>
You will know that the client authentication process was successful when the terminal shows you what appears in (Figure 7). This is basically brief information about the Linux distribution that is installed on your server, information about the last time you logged in, and the last line is where all the magic happens and it’s called the ‘Command or Prompt line’, and its structure is similar to the following:
<pre> username@hostname:~# </pre>


The first part indicates the username that you are currently logged in as then the host name, separated by the ‘@’ symbol, followed by your current directory (in this example ~, which refers to current user home directory) and the hash sign indicates the end of command line.


ATTENTION!



For every Linux distribution, the command line might differ slightly, but basically they all have the same structure.

## Conclusion



Learning is a slow and progressive process and I hope that you have grasped the basic concepts behind remote connection to a server and why security matters so much in the ever crowded insecure internet world and how SSH helps you maintain that. So remember that your server is like your home, you always have to protect it from unwanted visitors.


<div class=“author”>Submitted by: Saleh Salem</div>


