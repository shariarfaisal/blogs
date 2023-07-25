# How To Use docker exec to Run Commands in a Docker Container

```Docker``` ```Container```

## Introduction


Docker is a containerization tool that helps developers create and manage portable, consistent Linux containers.


When developing or deploying containers you’ll often need to look inside a running container to inspect its current state or debug a problem. To this end, Docker provides the docker exec command to run programs in containers that are already running.


In this tutorial we will learn about the docker exec command and how to use it to run commands and get an interactive shell in a running Docker container.


# Prerequisites


This tutorial assumes you already have Docker installed, and your user has permission to run docker. If you need to run docker as the root user, please remember to prepend sudo to the commands in this tutorial.


For more information on using Docker without sudo access, please see the Executing the Docker Command Without Sudo section of our How To Install Docker tutorial.


# Starting a Test Container


To use the docker exec command, you will need a running Docker container. If you don’t already have a container, start a test container with the following docker run command:


```
docker run -d --name container-name alpine watch "date >> /var/log/date.log"


```


This command creates a new Docker container from the official alpine image. This is a popular Linux container image that uses Alpine Linux, a lightweight, minimal Linux distribution.


We use the -d flag to detach the container from our terminal and run it in the background. --name container-name will name the container container-name. You could choose any name you like here, or leave this off entirely to have Docker automatically generate a unique name for the new container.


Next we have alpine, which specifies the image we want to use for the container.


And finally we have watch "date >> /var/log/date.log". This is the command we want to run in the container. watch will repeatedly run the command you give it, every two seconds by default. The command that watch will run in this case is date >> /var/log/date.log. date prints the current date and time, like this:


```
OutputFri Jul 23 14:57:05 UTC 2021


```


The >> /var/log/date.log portion of the command redirects the output from date and appends it to the file /var/log/date.log. Every two seconds a new line will be appended to the file, and after a few seconds it will look something like this:


```
OutputFri Jul 23 15:00:26 UTC 2021
Fri Jul 23 15:00:28 UTC 2021
Fri Jul 23 15:00:30 UTC 2021
Fri Jul 23 15:00:32 UTC 2021
Fri Jul 23 15:00:34 UTC 2021

```


In the next step we’ll learn how to find the names of Docker containers. This will be useful if you already have a container you’re targeting, but you’re not sure what its name is.


# Finding the Name of a Docker Container


We’ll need to provide docker exec with the name (or container ID) of the container we want to work with. We can find this information using the docker ps command:


```
docker ps


```


This command lists all of the Docker containers running on the server, and provides some high-level information about them:


```
OutputCONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
76aded7112d4   alpine    "watch 'date >> /var…"   11 seconds ago   Up 10 seconds             container-name

```


In this example, the container ID and name are highlighted. You may use either to tell docker exec which container to use.


If you’d like to rename your container, use the docker rename command:


```
docker rename container-name new-name


```


Next, we’ll run through several examples of using docker exec to execute commands in a running Docker container.


# Running an Interactive Shell in a Docker Container


If you need to start an interactive shell inside a Docker Container, perhaps to explore the filesystem or debug running processes, use docker exec with the -i and -t flags.


The -i flag keeps input open to the container, and the -t flag creates a pseudo-terminal that the shell can attach to. These flags can be combined like this:


```
docker exec -it container-name sh


```


This will run the sh shell in the specified container, giving you a basic shell prompt. To exit back out of the container, type exit then press ENTER:


```
exit


```


If your container image includes a more advanced shell such as bash, you could replace sh with bash above.


# Running a Non-interactive Command in a Docker Container


If you need to run a command inside a running Docker container, but don’t need any interactivity, use the docker exec command without any flags:


```
docker exec container-name tail /var/log/date.log


```


This command will run tail /var/log/date.log on the container-name container, and output the results. By default the tail command will print out the last ten lines of a file. If you’re running the demo container we set up in the first section, you will see something like this:


```
OutputMon Jul 26 14:39:33 UTC 2021
Mon Jul 26 14:39:35 UTC 2021
Mon Jul 26 14:39:37 UTC 2021
Mon Jul 26 14:39:39 UTC 2021
Mon Jul 26 14:39:41 UTC 2021
Mon Jul 26 14:39:43 UTC 2021
Mon Jul 26 14:39:45 UTC 2021
Mon Jul 26 14:39:47 UTC 2021
Mon Jul 26 14:39:49 UTC 2021
Mon Jul 26 14:39:51 UTC 2021

```


This is essentially the same as opening up an interactive shell for the Docker container (as done in the previous step with docker exec -it container-name sh) and then running the tail /var/log/date.log command. However, rather than opening up a shell, running the command, and then closing the shell, this command returns that same output in a single command and without opening up a pseudo-terminal.


# Running Commands in an Alternate Directory in a Docker Container


To run a command in a certain directory of your container, use the --workdir flag to specify the directory:


```
docker exec --workdir /tmp container-name pwd


```


This example command sets the /tmp directory as the working directory, then runs the pwd command, which prints out the present working directory:


```
Output/tmp

```


The pwd command has confirmed that the working directory is /tmp.


# Running Commands as a Different User in a Docker Container


To run a command as a different user inside your container, add the --user flag:


```
docker exec --user guest container-name whoami


```


This will use the guest user to run the whoami command in the container. The whoami command prints out the current user’s username:


```
Outputguest

```


The whoami command confirms that the container’s current user is guest.


# Passing Environment Variables into a Docker Container


Sometimes you need to pass environment variables into a container along with the command to run. The -e flag lets you specify an environment variable:


```
docker exec -e TEST=sammy container-name env


```


This command sets the TEST environment variable to equal sammy, then runs the env command inside the container. The env command then prints out all the environment variables:


```
OutputPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=76aded7112d4
TEST=sammy
HOME=/root

```


The TEST variable is set to sammy.


To set multiple variables, repeat the -e flag for each one:


```
docker exec -e TEST=sammy -e ENVIRONMENT=prod container-name env


```


If you’d like to pass in a file full of environment variables you can do that with the --env-file flag.


First, make the file with a text editor. We’ll open a new file with nano here, but you can use any editor you’re comfortable with:


```
nano .env


```


We’re using .env as the filename, as that’s a popular standard for using these sorts of files to manage information outside of version control.


Write your KEY=value variables into the file, one per line, like the following:


.env
```
TEST=sammy
ENVIRONMENT=prod

```


Save and close the file. To save the file and exit nano, press CTRL+O, then ENTER to save, then CTRL+X to exit.


Now run the docker exec command, specifying the correct filename after --env-file:


```
docker exec --env-file .env container-name env


```


```
OutputPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=76aded7112d4
TEST=sammy
ENVIRONMENT=prod
HOME=/root

```


The two variables in the file are set.


You may specify multiple files by using multiple --env-file flags. If the variables in the files overlap each other, whichever file was listed last in the command will override the previous files.


# Common Errors


When using the docker exec command, you may encounter a few common errors:


```
Error: No such container: container-name

```


The No such container error means the specified container does not exist, and may indicate a misspelled container name. Use docker ps to list out your running containers and double-check the name.


```
Error response from daemon: Container 2a94aae70ea5dc92a12e30b13d0613dd6ca5919174d73e62e29cb0f79db6e4ab is not running

```


This not running message means that the container exists, but it is stopped. You can start the container with docker start container-name


```
Error response from daemon: Container container-name is paused, unpause the container before exec

```


The Container is paused error explains the problem fairly well. You need to unpause the container with docker unpause container-name before proceeding.


# Conclusion


In this tutorial we learned how to execute commands in a running Docker container, along with some command line options available when doing so.


For more information on Docker in general, please see our Docker tag page, which has links to Docker tutorials, Docker-related Q&A pages, and more.


For help with installing Docker, take a look at How To Install and Use Docker on Ubuntu 20.04.


