# How To Create Nagios Plugins With Ruby On CentOS 6

```Monitoring``` ```Ruby``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


# Introduction


Ruby is a popular programming language that allows you to quickly create scripts and install additional libraries (Gems).


We have previously covered how to install Nagios monitoring server on CentOS 6.


This time, we will expand on this idea and create Nagios plugins using Ruby.
These plugins will be running on client VPS, and be executed via NRPE.


# Step 1 - Install RPMForge Repository and NRPE on client VPS


```
rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
yum -y install ruby nagios-nrpe
useradd nrpe && chkconfig nrpe on

```


# Step 2 - Create your Ruby Script


It would be a good idea to keep your plugins in same directory as other Nagios plugins (/usr/lib64/nagios/plugins/ for example).


For our example, we will create a script that checks current disk usage by calling "df" from shell, and throw an alert if it is over 85% used:


```
#!/usr/bin/ruby
used_space=`df -h / | grep -v "Filesystem" | awk '{print $5}'`
 case used_space.chomp <=> "85%"
 when -1
   puts "OK - #{used_space.chomp!} of disk space used."
   exit 0
 when 0
   puts "WARNING - #{used_space.chomp!} of disk space used."
   exit 1
 when 1
   puts "CRITICAL - #{used_space.chomp!} of disk space used."
   exit 2
 else
   puts "UNKNOWN - #{used_space.chomp!} of disk space used."
   exit 3
end

```


![](https://assets.digitalocean.com/articles/community/usedspace.rb.png)
We will save this script in /usr/lib64/nagios/plugins/usedspace.rb and make it executable:


```
chmod +x /usr/lib64/nagios/plugins/usedspace.rb

```


The entire Nagios NRPE plugin boils down to using exit codes to trigger alerts.


You introduce your level of logic to the script, and if you want to trigger an alert (whether it is OK, WARNING, CRITICAL, or UNKNOWN) - you specify an exit code.


Refer to the following Nagios Exit Codes:


## Nagios Exit Codes





Exit Code
Status


0
OK


1
WARNING


2
CRITICAL


3
UNKNOWN



# Step 3 - Add Your Script to NRPE configuration on client host


Delete original /etc/nagios/nrpe.cfg and add the following lines to it:


```
log_facility=daemon
pid_file=/var/run/nrpe/nrpe.pid
server_port=5666
nrpe_user=nrpe
nrpe_group=nrpe
allowed_hosts=198.211.117.251
dont_blame_nrpe=1
debug=0
command_timeout=60
connection_timeout=300
include_dir=/etc/nrpe.d/

command[usedspace_ruby]=/usr/lib64/nagios/plugins/usedspace.rb

```


Where 198.211.117.251 is our monitoring server from previous articles.  Change these to your own values.


Make sure to restart Nagios NRPE service:


```
service nrpe restart

```


# Step 4 - Add Your New Command to Nagios Checks on Nagios Monitoring Server


Define new command in /etc/nagios/objects/commands.cfg


```
define command{
        command_name    usedspace_ruby
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c usedspace_ruby
        }

```


As you can see, it uses NRPE to make TCP connections to port 5666 and run command 'usedspace_ruby', which we defined in /etc/nagios/nrpe.cfg on that remote host.


Add this check to your Nagios configuration file for client VPS.


For our example, we will monitor a server called CentOSDroplet and edit /etc/nagios/servers/CentOSDroplet.cfg


```
define service {
        use                             generic-service
        host_name                       CentOSDroplet
        service_description             Custom Disk Checker In Ruby
        check_command                   usedspace_ruby
        }

```


![](https://assets.digitalocean.com/articles/community/CentOSDroplet.cfg-ruby.png)
Restart Nagios:


```
service nagios restart

```


Verify that the new check is working:


![](https://assets.digitalocean.com/articles/community/nagios-centos-ruby.png)
And you are all done!


