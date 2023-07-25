# How To Install the Deno JavaScript Runtime on Ubuntu 20 04

```JavaScript``` ```Ubuntu``` ```Development``` ```Ubuntu 20.04```

## Introduction


Deno is a new JavaScript runtime being developed by the creator of Node.js, with a focus on security, developer experience, and compatibility with standard browser APIs.


Deno uses the same V8 JavaScript engine as Node.js and the Chrome web browser, but ships with secure sandboxing, built-in TypeScript support, and a curated set of standard modules.


In this tutorial we will download and install Deno on Ubuntu 20.04, and run a hello world statement to test out our installation.


# Prerequisites


This tutorial assumes you are running Ubuntu 20.04 and are logged in as a non-root, sudo-enabled user. For help setting this up, please refer to our Initial Server Setup with Ubuntu 20.04 tutorial.


# Step 1 — Downloading Deno


Deno ships as a single executable file, making it possible to download and install it manually. First navigate to a directory where you can download the roughly 30mb file. We’ll use the /tmp directory here:


```
cd /tmp


```


Next, use curl to download the latest release of Deno from GitHub:


```
curl -Lo "deno.zip" "https://github.com/denoland/deno/releases/latest/download/deno-x86_64-unknown-linux-gnu.zip"


```


This will display a progress bar:


```
Output  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   158  100   158    0     0   3361      0 --:--:-- --:--:-- --:--:--  3361
100   641  100   641    0     0   8902      0 --:--:-- --:--:-- --:--:--  8902
100 31.3M  100 31.3M    0     0   132M      0 --:--:-- --:--:-- --:--:--  132M

```


When the download is complete, you’ll have a deno.zip file in your current directory. In the next step, you’ll decompress this file and install the deno executable.


# Step 2 — Installing Deno


Now that you’ve downloaded the Deno zip file, it’s time to install it. First you’ll need to make sure you have the unzip command installed to decompress the file. Update your system’s package index and then install unzip with apt. You may be prompted to provide your sudo user’s password if this is the first time you’re using sudo in this session:


```
sudo apt update
sudo apt install unzip


```


When installed, use unzip to decompress the deno executable into the /usr/local/bin directory:


```
sudo unzip -d /usr/local/bin /tmp/deno.zip


```


The -d flag tells unzip to place the resulting file in /usr/local/bin. Note that because you’re unzipping into a protected system directory, you’ll need to use sudo above.


The installation should now be complete. Use ls to list the new /usr/local/bin/deno file and make sure it has the correct owner and permissions:


```
ls -al /usr/local/bin/deno


```


```
Output-rwxr-xr-x 1 root root 87007232 Aug 23 21:06 /usr/local/bin/deno

```


The above permissions are typical. Only root should have write (w) permissions, and everybody should have execute (x) permissions. For more information on Linux permissions, please see our Introduction to Linux Permissions tutorial.


Next, run the deno command with a --version flag to make sure it executes properly:


```
deno --version


```


deno will print out some version information:


```
Outputdeno 1.13.2 (release, x86_64-unknown-linux-gnu)
v8 9.3.345.11
typescript 4.3.5

```


You’ve now successfully downloaded and installed Deno. Next we’ll use it to run a hello world statement.


# Step 3 — Using the Deno REPL


If you run the bare deno command with no subcommands, it’ll put you into the Deno REPL. REPL is short for “read-eval-print loop”, an interactive prompt where you can input statements and have them evaluated, with the results printed immediately.


REPLs can be a good way to experiment with a new programming language.


Open the Deno REPL now:


```
deno


```


deno will print out its version, some help text, and a > prompt:


```
OutputDeno 1.13.2
exit using ctrl+d or close()
>

```


Type in the following JavaScript hello world example and hit ENTER to have Deno evaluate it and print the results:


```
['hello', 'world'].join(' ')


```


This statement creates a JavaScript array (['hello', 'world']), then uses the array’s join() method to join the two words together with a space character:


```
Output"hello world"

```


It worked! To exit the Deno REPL, press CTRL+D or type close() and press ENTER.


# Conclusion


In this tutorial you downloaded and installed Deno, then ran a hello world statement in its REPL. For more information on Deno, please see the official Deno Manual and the Deno API documentation.


For general JavaScript information, see our How To Code in JavaScript tutorial series, or our JavaScript tag page which will have links to more tutorials and community Q&A.


