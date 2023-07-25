# How To Set Up Multi-Factor Authentication for SSH on CentOS 8

```Security``` ```CentOS```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


SSH uses passwords for authentication by default, and most SSH hardening instructions recommend using an SSH key instead. However, an SSH key is still only a single factor, though a much more secure factor. The channel is the terminal on your computer sending the data via an encrypted tunnel to the remote machine. But like a hacker can guess a password, they can steal an SSH key, and then in either case, with that single piece of data, an attacker can gain access to your remote systems.


In this tutorial, we’ll set up multi-factor authentication to combat that. Multi-factor authentication (MFA) or Two-factor authentication (2FA) requires more than one factor to authenticate or log in. This means a bad actor would have to compromise multiple things, like your computer and your phone, to get in. There are several types of factors used in authentication:


1. Something you know, like a password or security question
2. Something you have, like an authenticator app or security token
3. Something you are, like your fingerprint or voice

One common factor is an OATH-TOTP app, like Google Authenticator. OATH-TOTP (Open Authentication Time-Based One-Time Password) is an open protocol that generates a one-time use password, commonly a six-digit number recycled every 30 seconds.


This article will go over how to enable SSH authentication using an OATH-TOTP app in addition to an SSH key. Logging into your server via SSH will require two factors across two channels, thereby making it more secure than a password or SSH key alone. Also, we’ll go over some additional use cases for MFA and some helpful tips and tricks.


And if you are seeking further guidance on securing SSH connections, check out these tutorials on Hardening OpenSSH and Hardening OpenSSH Client.


# Prerequisites


To follow this tutorial, you will need:


- One CentOS 8 server with a sudo non-root user and SSH key, which you can set up by following this Initial Server Setup tutorial.
- A smartphone or tablet with an OATH-TOTP app installed, like Google Authenticator (iOS, Android).
- Alternatively, you can also use a Linux command line app called ‘oathtool’ to generate an OATH-TOTP code. It’s available in various distribution repos
- If you really want secure your SSH connection there are several good steps outlined in this SSH Essentials article, such as whitelisting users, disabling root login, and changing which port SSH uses.

# Step 1 — Installing Google’s PAM


In this step, we’ll install and configure Google’s PAM.


PAM, which stands for Pluggable Authentication Module, is an authentication infrastructure used on Linux systems to authenticate a user. Because Google made an OATH-TOTP app, they also made a PAM that generates TOTPs and is fully compatible with any OATH-TOTP app, like Google Authenticator or Authy.


First, add the EPEL (Extra Packages for Enterprise Linux) repo:


```
sudo yum search epel


```


If your repositories have the EPEL install package, you will see the following output:


```
Output===== Name Matched: epel =====
epel-release.noarch : Extra Packages for Enterprise Linux repository configuration

```


Now install the epel-release package to enable the EPEL repository:


```
sudo yum install epel-release


```


However, if you don’t have the package epel-release, then you can install the repository information manually:


```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm


```


Next, install the PAM. You might be prompted to accept the EPEL key if this is the first time using the repo. Once accepted, you won’t be prompted for again to accept the key:


```
sudo yum install google-authenticator qrencode-libs


```


With the PAM installed, we’ll use a helper app that comes with the PAM to generate a TOTP key for the user you want to add a second factor to. This key is generated on a user-by-user basis, not system-wide. This means every user that wants to use a TOTP auth app will need to log in and run the helper app to get their own key; you can’t just run it once to enable it for everyone (but there are some tips at the end of this tutorial to set up or require MFA for many users).


Run these two commands to initialize the app:


```
google-authenticator -s ~/.ssh/google_authenticator


```


Normally, all you need to do is run the google-authenticator command with no arguments, but SELinux doesn’t allow the ssh daemon to write to files outside of the .ssh directory in your home folder. This prevents authentication.


SELinux is a powerful tool that protects your system from potential attacks, and it’s worth running in Enforcing mode. As such, turning off SELinux is not considered a best practice. Instead, we’ll move the default location of the google_authenticator file into your ~/.ssh directory.


After you run the command, you’ll be asked a few questions. The first one asks if authentication tokens should be time-based:


```
OutputDo you want authentication tokens to be time-based (y/n) y

```


This PAM allows for time-based or sequential-based tokens. Using sequential-based tokens mean the code starts at a certain point and then increments the code after every use. Using time-based tokens mean the code changes after a certain time frame. We’ll stick with time-based because that is what apps like Google Authenticator anticipate, so answer y for yes.


After answering this question, a lot of output will scroll past, including a large QR code. Use your authenticator app on your phone to scan the QR code or manually type in the secret key. If the QR code is too big to scan, you can use the URL above the QR code to get a smaller version. Once it’s added, you’ll see a six-digit code that changes every 30 seconds in your app.



Note: Make sure you record the secret key, verification code, and the emergency scratch codes in a safe place, like a password manager. The emergency scratch codes are the only way to regain access if you, for example, lose access to your TOTP app.

The remaining questions inform the PAM on how to function. We’ll go through them one by one:


```
OutputDo you want me to update your "~/.google_authenticator" file (y/n) y

```


This writes the key and options to the google_authenticator file. If you say no, the program quits and nothing is written, which means the authenticator won’t work:


```
OutputDo you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

```


By answering yes here, you are preventing a replay attack by making each code expire immediately after use. This prevents an attacker from capturing a code you just used and logging in with it:


```
OutputBy default, a new token is generated every 30 seconds by the mobile app.
In order to compensate for possible time-skew between the client and the server,
we allow an extra token before and after the current time. This allows for a
time skew of up to 30 seconds between authentication server and client. If you
experience problems with poor time synchronization, you can increase the window
from its default size of 3 permitted codes (one previous code, the current
code, the next code) to 17 permitted codes (the 8 previous codes, the current
code, and the 8 next codes). This will permit for a time skew of up to 4 minutes
between client and server.
Do you want to do so? (y/n) n

```


Answering yes here allows up to 17 valid codes in a moving four minute window. By answering no, you limit it to 3 valid codes in a 1:30 minute rolling window. Unless you find issues with the 1:30 minute window, answering no is the more secure choice. If you answer no and later realize you need more time, this setting can be adjusted in the .google_authenticator file stored at the root of your home directory:


```
OutputIf the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y

```


Rate limiting means a remote attacker can only attempt a certain number of guesses before being forced to wait some time before being able to try again. If you haven’t previously configured rate limiting directly into SSH, doing so now is a great hardening technique.


Once you finish this setup, if you want to back up your secret key, you can copy the ~/.ssh/google-authenticator file to a trusted location. From there, you can deploy it on additional systems or redeploy it after a clean install. Be aware that by using the same key on multiple machines you are introducing the possibility of a replay attack if an attacker is able to sniff the token for one server then use it, within the valid window, against a different server. You must evaluate the likelihood of that risk versus having every system use a different token and managing those different tokens.



Note: If you are using an encrypted home folder, which is beyond the scope of this article, you may need to store the ~./google-authenticator file in a directory outside of your home folder. The project README has details on how to do that.

Since we stored the config file in a non-standard location, we need to restore the SELinux context based on its new location.


Use the following command to do so:


```
restorecon -Rv ~/.ssh/


```


With those two changes done we now have the Google Authenticator PAM installed and configured, the next step is to configure SSH to use your TOTP key. We’ll need to tell SSH about the PAM and then configure SSH to use it.


# Step 2 — Configuring OpenSSH to Use MFA/2FA


Because we’ll be making SSH changes over SSH, it’s important to never close your initial SSH connection. Instead, open a second SSH session to do testing. This is to avoid locking yourself out of your server if there was a mistake in your SSH configuration. Once everything works, then you can safely close any sessions. Another safety precaution is to create a backup of the system files you will be editing so in case something goes wrong you can simply revert to the vanilla file and start over again with a clean configuration.


To begin, back up the sshd configuration file and then edit it. Here, we’re using nano, which isn’t installed on CentOS by default. You can install it with sudo yum install nano, or use your favorite alternative text editor.


Backup the file and then open it:


```
sudo cp /etc/pam.d/sshd /etc/pam.d/sshd.bak
sudo nano /etc/pam.d/sshd


```


Add the following line to the end of the file:


/etc/pam.d/sshd
```
auth       required     pam_google_authenticator.so secret=/home/${USER}/.ssh/google_authenticator nullok
auth       required     pam_permit.so

```


Since we had to put the google_authenticator config file in a non-standard location, we have to provide PAM with the path to the config file. The secret option tells PAM where the non-default location of the config file is stored.


The nullok word at the end of the line tells the PAM that this authentication method is optional. This allows users without a OATH-TOTP token to still log in just using their SSH key. Once all users have an OATH-TOTP token, you can remove nullok from this line to make MFA mandatory.


The second line with pam_permit.so is required to allow authentication if I user doesn’t use an MFA token to login. When logging in each method needs a SUCCESS to allow authentication. If a user doesn’t use the MFA auth tool by utilizing the nullok option that returns an IGNORE for the interactive keyboard authentication. So if that line is ignored then the next line triggers which pam_permit.so returns SUCCESS and allows authentication to proceed.


Save and close the file.


Next, we’ll configure SSH to support this kind of authentication.


Back up the the SSH configuration file then open it for editing:


```
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config


```


Look for lines beginning ChallengeResponseAuthentication. Comment out the no line and uncomment the yes line.


You file will look like this:


/etc/ssh/sshd_config
```
. . .
# Change to no to disable s/key passwords
ChallengeResponseAuthentication yes
#ChallengeResponseAuthentication no
. . .

```


Save and close the file and then restart SSH to reload the configuration files. Restarting the sshd service won’t close our current open connections, so you won’t risk locking yourself out with this command:


```
sudo systemctl restart sshd.service


```


To test that everything’s working so far, open ANOTHER terminal window and try logging in over SSH. It is very important that you keep your current SSH session open and test with an additional session or you will lock yourself out at some point and will need to use the web console to get yourself back in.



Note: If you’ve previously created an SSH key and are using it, you’ll notice you didn’t have to type in your user’s password or the MFA verification code. This is because an SSH key overrides all other authentication options by default. Otherwise, you should have gotten a password and verification code prompt.

Next, to enable an SSH key as one factor and the verification code as a second, we need to tell SSH which factors to use and prevent the SSH key from overriding all other types.


# Step 3 — Making SSH Aware of MFA


Reopen the sshd configuration file:


```
sudo nano /etc/ssh/sshd_config


```


Add the following line at the bottom of the file. This tells SSH which authentication methods are required. This line tells SSH we need a SSH key and either a password or a verification code (or all three):


/etc/ssh/sshd_config
```
. . .
AuthenticationMethods publickey,password publickey,keyboard-interactive

```


Save and close the file.


Next, open the PAM sshd configuration file again:


```
sudo nano /etc/pam.d/sshd


```


Find the line auth substack password-auth towards the top of the file. Comment it out by adding a # character as the first character on the line. This tells PAM not to prompt for a password:


/etc/pam.d/sshd
```
. . .
#auth       substack     password-auth
. . .

```


Save and close the file, then restart SSH:


```
sudo systemctl restart sshd.service


```


Now try logging into the server again with a different terminal session/window. Unlike last time, SSH should ask for your verification code. Upon entering it, you’ll be logged in. Even though you don’t see any indication that your SSH key was used, your login attempt used two factors. If you want to verify, you can add -v (for verbose) after the SSH command:


```
Example SSH Output. . .
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /Users/sammy/.ssh/id_rsa
debug1: Server accepts key: pkalg ssh-rsa blen 279
Authenticated with partial success.
debug1: Authentications that can continue: keyboard-interactive
debug1: Next authentication method: keyboard-interactive
Verification code:

```


Towards the end of the output, you’ll see where SSH uses your SSH key and then asks for the verification code. You can now log in over SSH with a SSH key and a one-time password. If you want to enforce all three authentication types, you can follow the next step.


You have now successfully added a second factor when logging in remotely to your server over SSH. If this is what you wanted — to use your SSH key and a TOTP token to enable MFA for SSH (for most people, this is the optimal configuration) — then you’re done.


What follows are some tips and tricks for recovery, automated usage, and more.


# Step 4 — Adding a Third Factor (Optional)


In Step 3, we listed the approved types of authentication in the sshd_config file:


1. publickey (SSH key)
2. password publickey (password)
3. keyboard-interactive (verification code)

Although we listed three different factors, with the options we’ve chosen so far, they only allow for an SSH key and the verification code. If you’d like to have all three factors (SSH key, password, and verification code), one quick change will enable all three.


Open the PAM sshd configuration file:


```
sudo nano /etc/pam.d/sshd


```


Locate the line you commented out previously, #auth substack password-auth, and uncomment the line by removing the # character. Save and close the file. Now once again, restart SSH:


```
sudo systemctl restart sshd.service


```


By enabling the option auth substack password-auth, PAM will now prompt for a password in addition the checking for an SSH key and asking for a verification code, which we had working previously. Now we can use something we know (password) and two different types of things we have (SSH key and verification code) over two different channels (your computer for the SSH key and your phone for the TOTP token).


# Step 5 — Recovering Access to Google MFA (optional)


As with any system that you harden and secure, you become responsible for managing that security. In this case, that means not losing your SSH key or your TOTP secret key and making sure you have access to your TOTP app. However, sometimes things happen, and you can lose control of the keys or apps you need to get in.


## Losing Access to the Secret Key


If you lose your TOTP secret key, recovery can be broken up into a couple of steps. The first is getting back in without knowing the verification code and the second is finding the secret key or regenerating it for normal MFA login. This often can happen if you get a new phone and don’t transfer over your secrets to a new authenticator app.


To get in after losing the TOTP secret key on a DigitalOcean Droplet, you can simply use the virtual console from your dashboard to log in using your username and password. This works because we only protected your user account with MFA for ssh connections. Non-ssh connections, such as a console login, doesn’t use the Google Authenticator PAM module.


If you’re on a non-Droplet system then you have two options for regaining access:


1. Console (local/non-ssh) access to the system (typically physical or via something like iDrac)
2. Have a different user that doesn’t have MFA enabled

The second option is the less secure option, since the point in using MFA is to harden all ssh connections, but it’s one fail safe if you lose access to your MFA authenticator app.


Once you’re logged in, there are two ways to get the TOTP secret:


1. Recover the existing key
2. Generate a new key

In each user’s home directory, the secret key and Google Authenticator settings are saved in the file  ~/.ssh/google-authenticator. The very first line of this file is a secret key. A quick way to get the key is to execute the following command, which displays the first line of the google-authenticator file (i.e. the secret key). Then, take that secret key and manually type it into a TOTP app:


```
head -n 1 /home/sammy/.ssh/google_authenticator


```


Once you’ve recovered your existing key, you can either manually type it into your authenticator app and then you should be back in business or you can fill in the relevant details in the URL below and have Google generate a QR code for you to scan. You’ll need to add your username, hostname, the secret key from the google-authenticator file, and then any name of your choosing for ‘entry-name-in-auth-app’ to easily identify this key versus a different TOTP token:


```
https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/username@hostname%3Fsecret%3D16-char-secret%26issuer%3Dentry-name-in-auth-app

```


If there is a reason not to use the existing key (for example, being unable to easily share the secret key with the impacted user securely), you can remove the ~/.ssh/google-authenticator file outright. This will allow the user to log in again using only a single factor, assuming you haven’t enforced MFA by removing the nullok option in the ‘/etc/pam.d/sshd’ file. They can then run google-authenticator to generate a new key.


## Losing Access to the TOTP App


If you need to log in to your server but don’t have access to your TOTP app to get your verification code, you can still log in using the recovery codes that were displayed when you first created your secret key. Note that these recovery codes are one-time use. Once one is used to log in, it cannot be used as a verification code again. Hopefully you saved them in a place that is accessible if you don’t have your TOTP app and yet still secure.


# Step 6 — Changing Authentication Settings (optional)


If you want to change your MFA settings after the initial configuration, instead of generating a new configuration with the updated settings, you can just edit the ~/.ssh/google-authenticator file. This file is laid out in the following manner:


.google-authenticator layout
```
<secret key>
<options>
<recovery codes>

```


Options that are set in this file have a line in the options section; if you answered “no” to a particular option during the initial setup, the corresponding line is excluded from the file. In other words, the file only contains enabled options. If an option isn’t present it is by default disabled.


Here are the changes you can make to this file:


- To enable sequential codes instead of time based codes, change the line " TOTP_AUTH to " HOTP_COUNTER 1.
- To allow multiple uses of a single code, remove the line " DISALLOW_REUSE.
- To extend the code expiration window to 4 minutes, add the line " WINDOW_SIZE 17.
- To disable multiple failed logins (rate limiting), remove the line " RATE_LIMIT 3 30.
- To change the threshold of rate limiting, find the line " RATE_LIMIT 3 30 and adjust the numbers. The 3 in the original indicates the number of attempts over a period of time, and the 30 indicates the period of time in seconds.
- To disable the use of recovery codes, remove the five 8 digit codes at bottom of the file.

# Step 7 — Avoiding MFA for Some Accounts (optional)


There may be a situation in which a single user or a few service accounts (i.e. accounts used by applications, not humans) need SSH access without MFA enabled. For example, some applications that use SSH, like some FTP clients, may not support MFA. If an application doesn’t have a way to request the verification code, the request may get stuck until the SSH connection times out.


As long as a couple of options in /etc/pam.d/sshd are set correctly, you can control which factors are used on a user-by-user basis.


To allow MFA for some accounts and SSH key only for others, make sure the following settings in /etc/pam.d/sshd are active:


/etc/pam.d/sshd
```
#%PAM-1.0
auth       required     pam_sepermit.so
#auth       substack     password-auth

. . .

# Used with polkit to reauthorize users in remote sessions
-session   optional     pam_reauthorize.so prepare
auth       required      pam_google_authenticator.so nullok

```


Here, auth substack password-auth is commented out because passwords need to be disabled. MFA cannot be forced if some accounts are meant to have MFA disabled, so leave the nullok option on the final line.


After setting this configuration, simply run google-authenticator as any users that need MFA, and don’t run it for users where only SSH keys will be used.


# Step 8 — Automating Setup with Configuration Management (optional)


Many system administrators use configuration management tools, like Puppet, Chef, or Ansible, to manage their systems. If you want to use a system like this to install set up a secret key when a new user’s account is created, there is a method to do that.


google-authenticator supports command line switches to set all the options in a single, non-interactive command. To see all the options, you can type google-authenticator --help. Below is the command that would set everything up as outlined in Step 1:


```
google-authenticator -t -d -f -r 3 -R 30 -w 3 -s ~/.ssh/google_authenticator


```


The options referenced above are as follows:


- -t => Time based counter
- -d => Disallow token reuse
- -f => Force writing the settings to file without prompting the user
- -r => How many attempts to enter the correct code
- -R => How long in seconds a user can attempt to enter the correct code
- -w => How many codes can are valid at a time (this references the 1:30 min - 4 min window of valid codes)
- -s => Path to where the authentication file should be stored

These answers all the questions we answered manually, saves it to a file, and then outputs the secret key, QR code, and recovery codes. (If you add the flag -q, then there won’t be any output.) If you do use this command in an automated fashion, make sure to capture the secret key and/or recovery codes and make them available to the user. Also remember to have your automation reset the SELinux context of the ‘.ssh’ directory (restorecon -Rv ~/.ssh/).


# Step 9 — Forcing MFA for All Users (optional)


If you want to force MFA for all users even on the first login, or if you would prefer not to rely on your users to generate their own keys, there’s an easy way to handle this. You can simply use the same google-authenticator file for each user, as there’s no user-specific data stored in the file.


To do this, after the configuration file is initially created, a privileged user needs to copy the file to the .ssh directory of every home directory and change its permissions to the appropriate user. You can also copy the file to /etc/skel/ so it’s automatically copied over to a new user’s home directory upon creation.



Warning: This can be a security risk because everyone is sharing the same second factor. This means that if it’s leaked, it’s as if every user had only one factor. Take this into consideration if you want to use this approach.

Another method to force the creation of a user’s secret key is to use a bash script that:


1. Creates a TOTP token,
2. Prompts them to download the Google Authenticator app and scan the QR code that will be displayed, and
3. Runs the google-authenticator application for them after checking if the google-authenticator file already exists.

To make sure the script runs when a user logs in, you can name it .bash_login and place it at the root of their home directory.


# Conclusion


In this tutorial, you added two factors (an SSH key + MFA token) across two channels (your computer + your phone) to your server. You’ve made it very difficult for an outside agent to brute force their way into your machine via SSH and greatly increased the security of your machine.


And remember, if you are seeking further guidance on securing SSH connections, check out these tutorials on Hardening OpenSSH and Hardening OpenSSH Client.


