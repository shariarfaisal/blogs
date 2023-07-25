# How To Set Up SSH Keys on Ubuntu 12 04

```Linux Basics``` ```Security``` ```DigitalOcean```

## Introduction


The Secure Shell Protocol (or SSH) is a cryptographic network protocol that allows users to securely access a remote computer over an unsecured network.


Though SSH supports password-based authentication, it is generally recommended that you use SSH keys instead. SSH keys are a more secure method of logging into an SSH server, because they are not vulnerable to common brute-force password hacking attacks.


Generating an SSH key pair creates two long strings of characters: a public and a private key. You can place the public key on any server, and then connect to the server using an SSH client that has access to the private key.


When the public and private keys match up, the SSH server grants access without the need for a password. You can increase the security of your key pair even more by protecting the private key with an optional (but highly encouraged) passphrase.



Note: If you are looking for information about setting up SSH keys in your DigitalOcean account, please refer to our DigitalOcean product documentation on SSH Keys

# Step 1 — Creating the Key Pair


The first step is to create a key pair on the client machine. This will likely be your local computer. Type the following command into your local command line:


```
ssh-keygen -t ed25519


```


```
OutputGenerating public/private ed25519 key pair.

```


You will see a confirmation that the key generation process has begun, and you will be prompted for some information, which we will discuss in the next step.



Note: if you are on an older system that does not support creating ed25519 key pairs, or the server you’re connecting to does not support them, you should create a strong rsa keypair instead:
ssh-keygen -t rsa -b 4096


This changes the -t “type” flag to rsa, and adds the -b 4096 “bits” flag to create a 4096 bit key.

# Step 2 — Specifying Where to Save the Keys


The first prompt from the ssh-keygen command will ask you where to save the keys:


```
OutputEnter file in which to save the key (/home/sammy/.ssh/id_ed25519):

```


You can press ENTER here to save the files to the default location in the .ssh directory of your home directory.


Alternately, you can choose another file name or location by typing it after the prompt and hitting ENTER.


# Step 3 — Creating a Passphrase


The second and final prompt from ssh-keygen will ask you to enter a passphrase:


```
OutputEnter passphrase (empty for no passphrase):

```


It’s up to you whether you want to use a passphrase, but it is strongly encouraged: the security of a key pair, no matter the encryption scheme, still depends on the fact that it is not accessible to anyone else.


Should a private key with no passphrase fall into an unauthorized user’s possession, they will be able to log in to any server you’ve configured with the associated public key.


The main downside to having a passphrase — typing it in — can be mitigated by using an ssh-agent service, which will temporarily store your unlocked key and make it accessible to the SSH client. Many of these agents are integrated with your operating system’s native keychain, making the unlocking process even more seamless.


To recap, the entire key generation process looks like this:


```
ssh-keygen -t ed25519


```


```
OutputGenerating public/private ed25519 key pair.
Enter file in which to save the key (/home/sammy/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/sammy/.ssh/id_ed25519
Your public key has been saved in /home/sammy/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:EGx5HEXz7EqKigIxHHWKpCZItSj1Dy9Dqc5cYae+1zc sammy@hostname
The key's randomart image is:
+--[ED25519 256]--+
| o+o o.o.++      |
|=oo.+.+.o  +     |
|*+.oB.o.    o    |
|*. + B .   .     |
| o. = o S . .    |
|.+ o o . o .     |
|. + . ... .      |
|.  . o. . E      |
| .. o.   . .     |
+----[SHA256]-----+

```


The public key is now located in /home/sammy/.ssh/id_ed25519.pub. The private key is now located in /home/sammy/.ssh/id_ed25519.


# Step 4 — Copying the Public Key to Your Server


Once the key pair is generated, it’s time to place the public key on the server that you want to connect to.


You can copy the public key into the server’s authorized_keys file with the ssh-copy-id command. Make sure to replace the example username and address:


```
ssh-copy-id sammy@your_server_address


```


Once the command completes, you will be able to log into the server via SSH without being prompted for a password. However, if you set a passphrase when creating your SSH key, you will be asked to enter the passphrase at that time. This is your local ssh client asking you to decrypt the private key, it is not the remote server asking for a password.


# Step 5 — Disabling Password-based SSH Authentication (Optional)


Once you have copied your SSH keys onto the server, you may want to completely prohibit password logins by configuring the SSH server to disable password-based authentication.



Warning: before you disable password-based authentication, be certain you can successfully log onto the server with your SSH key, and that there are no other users on the server using passwords to log in.

In order to disable password-based SSH authentication, open up the SSH configuration file. It is typically found at the following location:


```
sudo nano /etc/ssh/sshd_config


```


This command will open up the file within the nano text editor. Find the line in the file that includes PasswordAuthentication (or create the line if it doesn’t exist), make sure it is not commented out with a # at the beginning of the line, and change it to no:


/etc/ssh/sshd_config
```
PasswordAuthentication no

```


Save and close the file when you are finished. In nano, use CTRL+O to save, hit ENTER to confirm the filename, then CTRL+X to exit.


Reload the sshd service to put these changes into effect:


```
sudo systemctl reload sshd


```


Before exiting your current SSH session, make a test connection in another terminal to verify you can still connect.


# Conclusion


In this tutorial we created an SSH key pair, copied our public key to a server, and (optionally) disabled password-based authentication completely.


For more information about SSH and the SSH service, including how to set up multifactor authentication, please read our related tutorials:


- How To Use SSH to Connect to a Remote Server
- SSH Essentials: Working with SSH Servers, Clients, and Keys
- How To Set Up Multi-Factor Authentication for SSH on Ubuntu 20.04

