# How To Write a Bash Script To Restart Server Programs

```Linux Basics```

To ensure that the most imperative programs remain online as much as possible (even after a server crash or reboot), one can create a short bash script to check if the program is running, and if it is not, to launch it. By using cron to schedule the script to be executed on a regular basis, we can make sure that program relaunches whenever it goes down.


# Bash Script


The first step in this process is to create the script itself. There are a variety of programs such as upstart, supervisor, and monit, that have the capability to start and monitor applications on a virtual private server in a very nuanced way— this bash script will simply provide an on switch.


Below is a sample script that starts apache if it finds it off.


```
nano launch.sh
```


```
#!/bin/sh

ps auxw | grep apache2 | grep -v grep > /dev/null

if [ $? != 0 ]
then
        /etc/init.d/apache2 start > /dev/null
fi
```


Once you have saved the script, you must give it executable permissions in order to be able to run it:


```
chmod +x launch.sh
```


Apache can be replaced with any required application. Should you want to set up the script for a variety of applications, you can create a new script for each one, placing it on its own line in the cron file.


# Cron Setup


With the script in hand, we need to set up the schedule on which it will run. The cron utility allows us to schedule at what intervals the script should execute. Start by opening up the cron file:


```
crontab -e
```


Cron has a detailed explanation of how the timing system works at the beginning.


Once you know how often you want the script to run, you can write in the corresponding line.


The most often that the script can run in cron is every minute. Should you want to set up such a small increment, you can use this template:


```
* * * * * ~/launch.sh
```


Every five minutes would be set up like this:


```
*/5 * * * * ~/launch.sh
```


# See More


Setting up this simple script will keep the program starting up after it shuts down for any reason. This is convenient as it will ensure that the longest time that a program will be down is for the interval of time that you specified in the cron configuration.


Should you need a program that is even slightly more subtle, you can set up the details of your startup with one of the several server monitoring programs (Supervisor, Upstart, or Monit).


By Etel Sverdlov
