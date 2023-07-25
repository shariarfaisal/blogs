# How to Upgrade OSSEC 2 8 1 to OSSEC 2 8 2

```Security``` ```Ubuntu``` ```Debian```

## Introduction


OSSEC is an open-source, host-based intrusion detection system (HIDS) that performs log analysis, integrity checking, Windows registry monitoring, rootkit detection, time-based alerting, and active response. It can be installed to monitor a single server or thousands of servers.


This tutorial shows how to upgrade an installation of OSSEC 2.8.1 to the latest release, OSSEC 2.8.2, which addresses a recently-discovered bug.


# Prerequisites


- A Droplet already running OSSEC 2.8.1, set up following our tutorials for Ubuntu 14.04, Debian 8, or Fedora 21.

If you installed OSSEC 2.8.1 on FreeBSD 10.1 using this tutorial, you can easily perform the upgrade using that distribution’s package manager instead, and don’t need to follow this guide.


# Step 1 — Downloading and Verifying OSSEC 2.8.2


The first step to upgrading OSSEC is to download the tarball and its checksum file, which will be used to verify that the tarball has not been compromised.


First, download the new tarball.


```
wget -U ossec http://www.ossec.net/files/ossec-hids-2.8.2.tar.gz


```


Then download the checksum file.


```
wget -U ossec http://www.ossec.net/files/ossec-hids-2.8.2-checksum.txt


```


To verify that the tarball has not been compromised, first verify the MD5 checksum.


```
md5sum -c ossec-hids-2.8.2-checksum.txt


```


The output should be:


md5sum output
```
ossec-hids-2.8.2.tar.gz: OK
md5sum: WARNING: 1 line is improperly formatted

```


Then verify the SHA1 checksum.


```
sha1sum -c ossec-hids-2.8.2-checksum.txt


```


The expected output is:


sha1sum output
```
ossec-hids-2.8.2.tar.gz: OK
sha1sum: WARNING: 1 line is improperly formatted

```


# Step 2 — Fixing a Bug


Though OSSEC 2.8.2 fixed a security bug, it did not address a longstanding bug that caused OSSEC to overwrite the contents of the /etc/hosts.deny file. The fix for that has to be applied manually before initiating the upgrade. And the fix involves editing a file in the newly downloaded tarball.


That means we must first unpack the tarball.


```
tar xf ossec-hids-2.8.2.tar.gz


```


It should be unpacked into a directory whose name includes the version number of the program. Change (cd) into that directory.


```
cd ossec-hids-2.8.2


```


The file that we need to edit, host-deny.sh, is in the active-response directory. So open it using:


```
nano active-response/host-deny.sh


```


Towards the end of the file, look for two lines in the code that begin with TMP_FILE =, underneath the # Deleting from hosts.deny comment. Edit both lines to remove the spaces on either side of the = sign so the code block looks like this.


Modified host-deny.sh code block
```
# Deleting from hosts.deny
elif [ "x${ACTION}" = "xdelete" ]; then
   lock;
   TMP_FILE=`mktemp /var/ossec/ossec-hosts.XXXXXXXXXX`
   if [ "X${TMP_FILE}" = "X" ]; then
     # Cheap fake tmpfile, but should be harder then no random data
     TMP_FILE="/var/ossec/ossec-hosts.`cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -1 `" 
   fi

```


Save and close the file.


# Step 3 — Upgrading OSSEC 2.8.1


Now we can initiate the upgrade.


```
sudo ./install.sh


```


You’ll be prompted to select the language of installation. Press ENTER to accept the default, or type in the 2-letter code that represents your preferred language, then press ENTER. Following the on screen instructions, at some point, you’ll be asked two simple questions. For each, type y, then press ENTER.


OSSEC question prompts
```
- You already have OSSEC installed. Do you want to update it? (y/n): y
 - Do you want to update the rules? (y/n): y

```


The upgrade process should take about two minutes. The installer will stop then restart OSSEC at the end, and you should receive an email confirming that OSSEC has restarted.


You can double-check this by querying for OSSEC’s status.


```
sudo /var/ossec/bin/ossec-control status


```


The output should indicate that all the processes are running.


# Conclusion


By following these simple steps, you have just upgraded OSSEC 2.8.1 to OSSEC 2.8.2.


