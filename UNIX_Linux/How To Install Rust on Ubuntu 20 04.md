# How To Install Rust on Ubuntu 20 04

```Ubuntu``` ```UNIX/Linux```

## Introduction


The Rust programming language, also known as rust-lang, is a powerful general-purpose programming language. Rust is syntactically similar to C++ and is used for a wide range of software development projects, including browser components, game engines, and operating systems.


In this tutorial, you’ll install the latest version of Rust on Ubuntu 20.04, and then create, compile, and run a test program. The examples in this tutorial show the installation of Rust version 1.66.



Note: This tutorial also works for Ubuntu 22.04, however, you might be presented with interactive dialogs for various questions when you run apt upgrade. For example, you might be asked if you want to automatically restart services when required or if you want to replace a configuration file that you’ve modified. The answers to these questions depend on your software and preferences and are outside the scope of this tutorial.

# Prerequisites


To complete this tutorial, you’ll need an Ubuntu 20.04 server with a sudo-enabled non-root user and a firewall. You can set this up by following our Initial Server Setup with Ubuntu 20.04 tutorial.


# Step 1 — Installing Rust on Ubuntu Using the rustup Tool


Although there are several different ways to install Rust on Linux, the recommended method is to use the rustup command line tool.


Run the command to download the rustup tool and install the latest stable version of Rust:


```
curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh


```


You’re prompted to choose the type of installation:


```
Outputsammy@ubuntu:~$ curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
info: downloading installer

Welcome to Rust!

This will download and install the official compiler for the Rust
programming language, and its package manager, Cargo.

Rustup metadata and toolchains will be installed into the Rustup
home directory, located at:

  /home/sammy/.rustup

This can be modified with the RUSTUP_HOME environment variable.

The Cargo home directory is located at:

  /home/sammy/.cargo

This can be modified with the CARGO_HOME environment variable.

The cargo, rustc, rustup and other commands will be added to
Cargo's bin directory, located at:

  /home/sammy/.cargo/bin

This path will then be added to your PATH environment variable by
modifying the profile files located at:

  /home/sammy/.profile
  /home/sammy/.bashrc

You can uninstall at any time with rustup self uninstall and
these changes will be reverted.

Current installation options:


   default host triple: x86_64-unknown-linux-gnu
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
>

```


This tutorial uses the default option 1. However, if you’re familiar with the rustup installer and want to customize your installation, you can choose option 2. Type your selection and press Enter.


The output for option 1 is:


```
Outputinfo: profile set to 'default'
info: default host triple is x86_64-unknown-linux-gnu
info: syncing channel updates for 'stable-x86_64-unknown-linux-gnu'
info: latest update on 2023-01-10, rust version 1.66.1 (90743e729 2023-01-10)
info: downloading component 'cargo'
info: downloading component 'clippy'
info: downloading component 'rust-docs'
info: downloading component 'rust-std'
info: downloading component 'rustc'
 67.4 MiB /  67.4 MiB (100 %)  40.9 MiB/s in  1s ETA:  0s
info: downloading component 'rustfmt'
info: installing component 'cargo'
  6.6 MiB /   6.6 MiB (100 %)   5.5 MiB/s in  1s ETA:  0s
info: installing component 'clippy'
info: installing component 'rust-docs'
 19.1 MiB /  19.1 MiB (100 %)   2.4 MiB/s in  7s ETA:  0s
info: installing component 'rust-std'
 30.0 MiB /  30.0 MiB (100 %)   5.6 MiB/s in  5s ETA:  0s
info: installing component 'rustc'
 67.4 MiB /  67.4 MiB (100 %)   5.9 MiB/s in 11s ETA:  0s
info: installing component 'rustfmt'
info: default toolchain set to 'stable-x86_64-unknown-linux-gnu'

  stable-x86_64-unknown-linux-gnu installed - rustc 1.66.1 (90743e729 2023-01-10)


Rust is installed now. Great!

To get started you may need to restart your current shell.
This would reload your PATH environment variable to include
Cargo's bin directory ($HOME/.cargo/bin).

To configure your current shell, run:
source "$HOME/.cargo/env"
sammy@ubuntu:~$ 

```


Next, run the following command to add the Rust toolchain directory to the PATH environment variable:


```
source $HOME/.cargo/env


```


# Step 2 — Verifying the Installation


Verify the Rust installation by requesting the version:


```
rustc --version


```


The rustc --version command returns the version of the Rust programming language installed on your system. For example:


```
Outputsammy@ubuntu:~$ rustc --version
rustc 1.66.1 (90743e729 2023-01-10)
sammy@ubuntu:~$ 

```


# Step 3 — Installing a Compiler


Rust requires a linker program to join compiled outputs into one file. The GNU Compiler Collection (gcc) in the build-essential package includes a linker. If you don’t install gcc, then you might get the following error when you try to compile:


```
error: linker `cc` not found
  |
  = note: No such file or directory (os error 2)

error: aborting due to previous error

```


You’ll use apt to install the build-essential package.


First, update the Apt package index:


```
sudo apt update


```


Enter your password to continue if prompted. The apt update command outputs a list of packages that can be upgraded. For example:


```
Outputsammy@ubuntu:~$ sudo apt update
[sudo] password for sammy: 
Hit:1 http://mirrors.digitalocean.com/ubuntu focal InRelease
Get:2 http://mirrors.digitalocean.com/ubuntu focal-updates InRelease [114 kB]                                   
Hit:3 https://repos-droplet.digitalocean.com/apt/droplet-agent main InRelease                                   
Get:4 http://mirrors.digitalocean.com/ubuntu focal-backports InRelease [108 kB]                               
Get:5 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:6 http://mirrors.digitalocean.com/ubuntu focal-updates/main amd64 Packages [2336 kB]
Get:7 http://mirrors.digitalocean.com/ubuntu focal-updates/main Translation-en [403 kB]
Get:8 http://mirrors.digitalocean.com/ubuntu focal-updates/main amd64 c-n-f Metadata [16.2 kB]
Get:9 http://mirrors.digitalocean.com/ubuntu focal-updates/restricted amd64 Packages [1560 kB]
Get:10 http://mirrors.digitalocean.com/ubuntu focal-updates/restricted Translation-en [220 kB]
Get:11 http://mirrors.digitalocean.com/ubuntu focal-updates/restricted amd64 c-n-f Metadata [620 B]
Get:12 http://mirrors.digitalocean.com/ubuntu focal-updates/universe amd64 Packages [1017 kB]
Get:13 http://mirrors.digitalocean.com/ubuntu focal-updates/universe Translation-en [236 kB]
Get:14 http://mirrors.digitalocean.com/ubuntu focal-updates/universe amd64 c-n-f Metadata [23.2 kB]
Get:15 http://mirrors.digitalocean.com/ubuntu focal-updates/multiverse amd64 Packages [25.2 kB]
Get:16 http://mirrors.digitalocean.com/ubuntu focal-updates/multiverse Translation-en [7408 B]
Get:17 http://mirrors.digitalocean.com/ubuntu focal-updates/multiverse amd64 c-n-f Metadata [604 B]
Get:18 http://mirrors.digitalocean.com/ubuntu focal-backports/main amd64 Packages [45.7 kB]
Get:19 http://mirrors.digitalocean.com/ubuntu focal-backports/main Translation-en [16.3 kB]
Get:20 http://mirrors.digitalocean.com/ubuntu focal-backports/main amd64 c-n-f Metadata [1420 B]
Get:21 http://mirrors.digitalocean.com/ubuntu focal-backports/universe amd64 Packages [24.9 kB]
Get:22 http://mirrors.digitalocean.com/ubuntu focal-backports/universe Translation-en [16.3 kB]
Get:23 http://mirrors.digitalocean.com/ubuntu focal-backports/universe amd64 c-n-f Metadata [880 B]
Get:24 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [1960 kB]
Get:25 http://security.ubuntu.com/ubuntu focal-security/main Translation-en [320 kB]
Get:26 http://security.ubuntu.com/ubuntu focal-security/main amd64 c-n-f Metadata [11.7 kB]
Get:27 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [1463 kB]
Get:28 http://security.ubuntu.com/ubuntu focal-security/restricted Translation-en [207 kB]
Get:29 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 c-n-f Metadata [624 B]
Get:30 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [786 kB]
Get:31 http://security.ubuntu.com/ubuntu focal-security/universe Translation-en [152 kB]
Get:32 http://security.ubuntu.com/ubuntu focal-security/universe amd64 c-n-f Metadata [16.9 kB]
Get:33 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [22.2 kB]
Get:34 http://security.ubuntu.com/ubuntu focal-security/multiverse Translation-en [5464 B]
Get:35 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 c-n-f Metadata [516 B]
Fetched 11.2 MB in 5s (2131 kB/s)                          
Reading package lists... Done
Building dependency tree       
Reading state information... Done
103 packages can be upgraded. Run 'apt list --upgradable' to see them.
sammy@ubuntu:~$ 

```


Next, upgrade any out-of-date packages:


```
sudo apt upgrade


```


Enter Y if prompted to continue the upgrades.


When the upgrades are complete, install the build-essential package:


```
sudo apt install build-essential


```


Enter Y when prompted to continue the installation. The installation is complete when your terminal returns to the command prompt with no error messages.


# Step 4 — Creating, Compiling, and Running a Test Program


In this step, you’ll create a test program to try out Rust and verify that it’s working properly.


Start by creating some directories to store the test script:


```
mkdir ~/rustprojects
cd ~/rustprojects
mkdir testdir
cd testdir


```


Use nano, or your favorite text editor, to create a file in testdir to store your Rust code:


```
nano test.rs


```


You need to use the .rs extension for all your Rust programs.


Copy the following code into test.rs and save the file:


test.rs
```
fn main() {
    println!("Congratulations! Your Rust program works.");
}

```


Compile the code using the rustc command:


```
rustc test.rs


```


Run the resulting executable:


```
./test


```


The program prints to the terminal:


```
Outputsammy@ubuntu:~/rustprojects/testdir$ ./test
Congratulations! Your Rust program works.
sammy@ubuntu:~/rustprojects/testdir$

```


# Other Commonly-Used Rust Commands


It’s a good idea to update your installation of Rust on Ubuntu regularly.


Enter the following command to update Rust:


```
rustup update


```


You can also remove Rust from your system, along with its associated repositories.


Enter the following command to uninstall Rust:


```
rustup self uninstall


```


You’re prompted to enter Y to continue the uninstall process:


```
Outputammy@ubuntu:~/rustprojects/testdir$ rustup self uninstall


Thanks for hacking in Rust!

This will uninstall all Rust toolchains and data, and remove
$HOME/.cargo/bin from your PATH environment variable.

Continue? (y/N)

```


Enter Y to continue:


```
OutputContinue? (y/N) y

info: removing rustup home
info: removing cargo home
info: removing rustup binaries
info: rustup is uninstalled
sammy@ubuntu:~/rustprojects/testdir$ 

```


Rust is removed from your system.


# Conclusion


Now that you’ve installed and tested out Rust on Ubuntu, continue your learning with more Ubuntu tutorials.


