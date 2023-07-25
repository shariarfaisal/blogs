# How to Read Command Line Arguments in Shell Scripts 

```UNIX/Linux```

Reading user input is one part of the equation. In today’s article, we’ll learn to read command-line arguments in shell scripts. Shell scripts are an essential tool for any Linux user. They play a major role in automating daily tasks and creating your own commands or macros.


These shell scripts can receive input from the user in the form of arguments.


When we pass arguments to a shell script, we can use them for local sequencing of the script. We can use these arguments to obtain outputs and even modify outputs just like variables in shell scripts.


# What are Command-Line Arguments?


Command-line arguments are parameters that are passed to a script while executing them in the bash shell.


They are also known as positional parameters in Linux.


We use command-line arguments to denote the position in memory where the command and it’s associated parameters are stored. Understanding the command-line arguments is essential for people who are learning shell scripting.


In this article, we will go over the concept of command-line arguments along with their use in a shell script.


# How Shell Scripts Understand Command Line Arguments


Command-line arguments help make shell scripts interactive for the users. They help a script identify the data it needs to operate on. Hence, command-line arguments are an essential part of any practical shell scripting uses.


The bash shell has special variables reserved to point to the arguments which we pass through a shell script. Bash saves these variables numerically ($1, $2, $3, … $n)


Here, the first command-line argument in our shell script is $1, the second $2 and the third is $3. This goes on till the 9th argument. The variable $0 stores the name of the script or the command itself.


We also have some special characters which are positional parameters, but their function is closely tied to our command-line arguments.


The special character $# stores the total number of arguments. We also have $@ and $* as wildcard characters which are used to denote all the arguments. We use $$ to find the process ID of the current shell script, while $? can be used to print the exit code for our script.


# Read Command-line Arguments in Shell Scripts


Now we have developed an understanding of the command-line arguments in Linux. Now it’s time to use this knowledge for practical application of the netstat command.


For this tutorial, we will go over an example to learn how to use the command-line arguments in your shell script.


First, we will create a shell script to demonstrate the working of all the reserved variables which we discussed in the previous section. Use nano or any preferred editor of your choice and copy the following.


This is the shell script which we plan to use for this purpose.


```
#!/bin/sh
echo "Script Name: $0"
echo "First Parameter of the script is $1"
echo "The second Parameter is $2"
echo "The complete list of arguments is $@"
echo "Total Number of Parameters: $#"
echo "The process ID is $$"
echo "Exit code for the script: $?"

```


Once we are done, we will save the script as PositionalParameters.sh and exit our text editor.


Now, we will open the command line on our system and run the shell script with the following arguments.


```
./PositionalParameters.sh learning command line arguments

```


The script will run with our specified arguments and use positional parameters to generate an output. If you followed the step correctly, you should see the following screen.


Reading Arguments
Our output shows the correct output by substituting the reserved variables with the appropriate argument when we called it.


The process was run under the process ID 14974 and quit with the exit code 0.


# Wrapping up


Being able to read command-line arguments in shell scripts is an essential skill as it allows you to create scripts that can take input from the user and generate output based on a logical pathway.


With the help of command-line arguments, your scripts can vastly simplify the repetitive task which you may need to deal with on a daily basis, create your own commands while saving you both time and effort.


We hope this article was able to help you understand how to read the command line arguments in a shell script. If you have any comments, queries or suggestions, feel free to reach out to us in the comments below.


