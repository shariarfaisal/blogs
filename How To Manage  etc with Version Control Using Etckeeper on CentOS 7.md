# How To Manage  etc with Version Control Using Etckeeper on CentOS 7

```Miscellaneous``` ```Linux Commands``` ```CentOS```

## Introduction


In the Linux ecosystem, installing, maintaining and upgrading software on a periodic basis is a must. However, tracking changes made to local configuration files is still necessary. As opposed to the old standby of making copies of configuration files before making changes, etckeeper lets you keep track of modifications using a Git, Mercurial, Bazaar, or Darcs repository, just like you would do with a software development project.


In addition, etckeeper integrates seamlessly with yum to automatically commit changes made to the contents of the /etc directory when packages are upgraded. This will allow you to revert to previous versions of your configuration files should you want — or need — to do so.


# Prerequisites


To follow this tutorial, you will need:


- One CentOS 7 Droplet with a sudo non-root user, which you can set up by following steps 1 through 4 in this tutorial.

etckeeper only tracks file permissions, metadata, and changes. It does not provide out-of-the-box tools to perform the restoration of files, so an understanding of the fundamentals of a revision control system is necessary.


In this article we will use Git, which is the default VCS that etckeeper uses. If you want to refresh your memory about Git and version control, you can check the this tutorial series. Although you will not use Git directly in this guide, you can run Git-specific commands through etckeeper.


# Step 1 — Installing etckeeper


In this step, we will install etckeeper.


First, you will need to enable EPEL (Extra Packages for Enterprise Linux) on your CentOS 7 server, because that is the repository which contains etckeeper.


```
sudo yum update && sudo yum install epel-release

```


Then install etckeeper.


```
sudo yum update && sudo yum install etckeeper

```


Git comes with CentOS 7 by default, so we don’t need to install it.


# Step 2 — Customizing etckeeper’s Configuration


Once you have installed etckeeper, the next step is updating the /etc/etckeeper/etckeeper.conf configuration file.


First, open the configuration file with Nano or your favorite text editor.


```
sudo nano /etc/etckeeper/etckeeper.conf

```


The following are the essential variables that you need to configure in order for etckeeper to work properly. Feel free to leave the rest of the settings commented out.


First, under the comment # The VCS to use., make sure the VCS="git" is uncommented (i.e. there is no # at the beginning of the line). By default, this option will already be uncommented as git is the default VCS used by etckeeper installations on CentOS 7.


If you want to prevent etckeeper from committing changes automatically once per day, make sure the AVOID_DAILY_AUTOCOMMITS=1 is uncommented. In order to decide if you want to set this, you should consider whether your system configuration files undergo frequent changes (e.g. testing environments often change every day). If so, you should comment out that line; otherwise, you can leave it commented.


If you want yum install to abort when there are uncommitted changes in /etc, make sure to uncomment AVOID_COMMIT_BEFORE_INSTALL=1. This will require a manual commit before using yum to install packages. Otherwise, leave it commented out and yum will automatically commit the updated files before running an installation. This choice is entirely up to you; it depends largely on your environment, and the quantity of changes. This is much like the previous example, except that this time it will depend on the frequency with which you install packages.


When you’re done updating the options, save and close the file.


# Step 3 — Initializing the Git Repository


In this step, we will initialize the Git repository in /etc.


First, change to the /etc directory.


```
cd /etc

```


Next, initialize the repository by running the following command.


```
sudo etckeeper init

```


You should get the following message:


```
Initialized empty Git repository in /etc/.git/

```


You should now see the .git directory and the .gitignore file inside /etc. For example, if you run the following command:


```
ls -la /etc | grep git

```


You should see these lines included in the output:


```
drwx------.  7 root     root       4096 Apr  2 21:42 .git
-rw-r--r--.  1 root     root        874 Apr  2 21:42 .gitignore

```


Note: .git must be protected in the local system (hence the read, write, and execute permissions for the superuser only); because version control systems don’t keep track of file permissions by themselves, etckeeper provides this feature instead.


The .git directory contains several configuration and description files and other subdirectories intended for the use of Git itself. The .gitignore file, which specifies explicitly untracked files that git should ignore, is intended to be managed by etckeeper in its entirety. It is not advisable to edit it by hand, with one exception.


If there are certain files that you don’t want to track using version control, you can add them to the .gitignore file manually. To stop tracking a file, first open .gitignore for editing.


```
sudo nano .gitignore

```


The last line of the file will read # end section managed by etckeeper. Add the filenames of the files you want to ignore, one per line, above this one.


```
file_to_ignore
another_file_to_ignore
# end section managed by etckeeper

```


Then save and close the file.


You additionally will need to remove those files from the cache that is currently being managed by git since you initialized the local repository earlier.


```
etckeeper vcs rm --cached file_to_ignore

```


Repeat the above command for as many files as you previously added to .gitignore.


# Step 4 — Committing /etc to the Git Repository


In this step, we will commit our initial /etc.


Adding your first commit is easy; simply enter the following command. Although not strictly necessary, you should add a description to each commit to be able to easily identify them later.


```
sudo etckeeper commit "First commit of my /etc directory"

```


You should then see the outputted list of files being committed to your repository, like the below (truncated):


```
create mode 100644 selinux/targeted/modules/active/modules/dnsmasq.pp
create mode 100644 selinux/targeted/modules/active/modules/dnssec.pp
create mode 100644 selinux/targeted/modules/active/modules/docker.pp
create mode 100644 selinux/targeted/modules/active/modules/dovecot.pp

```


# Step 5 — Making Changes


In this step, we will make some changes to a file in /etc and commit them. In the next step, we will revert these changes.


First, make a change to the contents of a file of your choice. For example, you can add a new host to your local name resolution by adding a line consisting of an IP address and its associated hostname at the end of /etc/hosts.


First, open the file.


```
sudo nano /etc/hosts

```


Then, add the following line to the end of the file.


```
192.168.0.2    node01

```


Save and close the file. Now, let’s commit this change.


```
sudo etckeeper commit "Added a line to the hosts file"

```


Finally, modify the file’s permissions.


```
sudo chmod 640 /etc/hosts

```


And modify its ownership (making sure to replace sammy with your own username).


```
sudo chown sammy:sammy /etc/hosts

```


You can check the current permissions and ownership of /etc/hosts.


```
ls -l /etc/hosts

```


The output should look like this:


```
-rw-r----- 1 sammy sammy 675 Apr 17 15:01 /etc/hosts

```


# Step 6 — Undoing Changes


Now let’s test the restoring capabilities of etckeeper — not just of the file and its contents, but also its permissions and ownership.


First, list the commits you’ve made so far.


```
sudo git log --pretty=oneline

```


The first column of the output is a SHA-1 hash that uniquely identifies the commit; the second is the description that you used when you submit the changes earlier.


Your output should look similar to this, with different hashes.


```
d0958fbe4d947a6a3ad98141f9fe89d1fd1a95c4 Added a line to the hosts file
76c193da740a3e137fa000773a79de8bb5c898b7 First commit of my /etc directory

```


Take note of the hash of each commit. You will use them to roll back to that previous state.


Let’s roll back /etc/hosts to how it looked before we started this tutorial. Replace the characters in red with the SHA-1 hash that corresponds to your first commit. Note that you don’t need to specify the whole SHA-1 hash string; a few characters that uniquely identify it will do.


```
sudo etckeeper vcs checkout 76c19 /etc/hosts

```


Now we can check the contents, permissions, and ownership of /etc/hosts to see that they’ve been changed back.


Look at the last few lines of the file.


```
tail /etc/hosts

```


Should give you an output like this, which is missing the 192.168.0.2    node01 line we added, as expected.


```
...

# The following lines are desirable for IPv6 capable hosts
::1 test-etckeeper test-etckeeper
::1 localhost.localdomain localhost
::1 localhost6.localdomain6 localhost6

```


Check the current permissions and ownership of /etc/hosts again.


```
ls -l /etc/hosts

```


You’ll see an output that looks like this.


```
-rw-r--r-- 1 root root 704 Apr 17 15:01 /etc/hosts

```


The contents of the file were restored correctly, as well as the permissions and ownership.


## Conclusion


In this tutorial we have explained how to use etckeeper, a great tool to store your /etc directory in a Git repository. You can also use to a Bazaar, Mercurial, or Darcs repository.


Regardless of the VCS of your choice, etckeeper will help you keep on top of your configuration files and ensure that you can always roll back to a previous state if you need to undo changes or modify functionality.


