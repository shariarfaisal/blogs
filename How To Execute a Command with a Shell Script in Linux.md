# How To Execute a Command with a Shell Script in Linux

```UNIX/Linux```

## Introduction


Shell is a command-line interpreter that allows the user to interact with the system. It is responsible for taking inputs from the user and displaying the output.


Shell scripts are a series of commands written in order of execution. These scripts can contain functions, loops, commands, and variables. Scripts are useful for simplifying a complex series of commands and repetitive tasks.


In this article, you will learn how to create and execute shell scripts for the command line in Linux.


# Prerequisites


To complete this tutorial, you will need:


- Familiarity with using the terminal.
- Familiarity with a text editor.
- Familiarity with commands like chmod, mkdir, and cd.

# Getting Started


A shell script needs to be saved with the extension .sh.


The file needs to begin with the shebang line (#!) to let the Linux system know which interpreter to use for the shell script.


For environments that support bash, use:


```
#!/bin/bash 

```


For environments that support shell, use:


```
#!/bin/sh

```


This tutorial assumes that your environment supports bash.


Shell scripts can also have comments to increase readability. A good script always contains comments that help a reader understand exactly what the script is doing and the reasoning behind a design choice.


# Creating and Running a Basic Shell Script


You can create a shell script using the vi editor, a cat command, or a text editor.


For this tutorial, you will learn about creating a shell script with vi:


```
vi basic_script.sh


```


This starts the vi editor and creates a basic_script.sh file.


Then, press i on the keyboard to start INSERT MODE. Add the following lines:


basic_script.sh
```
#!/bin/bash
whoami
date

```


This script runs the commands whoami and date. whoami displays the active username. date displays the current system timestamp.


To save and exit the vi editor:


- Press ESC
- Type : (colon character)
- Type wq
- Press ENTER

Finally, you can run the script with the following command:


```
bash basic_script.sh


```


You may get output that resembles the following:


```
Outputroot
Fri Jun 19 16:59:48 UTC 2020

```


The first line of output corresponds to the whoami command. The second line of output corresponds to the date command.


You can also run a script without specifying bash:


```
./basic_script.sh 


```


Running the file this way might require the user to give permission first. Running it with bash doesn’t require this permission.


```
Output~bash: ./basic_script.sh: Permission denied

```


The command bash filename only requires the read permission from the file.


Whereas the command ./filename, runs the file as an executable and requires the execute permission.


To execute the script, you will need to update the permissions.


```
chmod +x basic_script.sh


```


This command applies chmod and gives x (executable) permissions to the current user.


# Using Variables in Shell Scripts


Scripts can include user-defined variables. In fact, as scripts get larger in size, it is essential to have variables that are clearly defined and that have self-descriptive names.


Add the following lines to the script:


basic_script.sh
```
#!/bin/bash
# This is a comment

# defining a variable
GREETINGS="Hello! How are you"
echo $GREETINGS

```


GREETINGS is the variable defined and later accessed using $ (dollar sign symbol. There should be no space in the line where variables are being assigned a value.


Run the script:


```
bash basic_script.sh


```


This prints out the value assigned to the variable:


```
OutputHello! How are you

```


When the script is run, GREETINGS is defined and accessed.


# Reading Input from the Command Line


Shell scripts can be made interactive with the ability to accept input from the command line. You can use the read command to store the command line input in a variable.


Add the following lines to the script:


basic_script.sh
```
#!/bin/bash
# This is a comment

# defining a variable
echo "What is your name?"
# reading input
read NAME
# defining a variable
GREETINGS="Hello! How are you"
echo $NAME $GREETINGS               

```


A variable NAME has been used to accept input from the command line. This script waits for the user to provide input for NAME. Then it prints NAME and GREETINGS.


```
OutputWhat is your name?
Sammy
Sammy Hello! How are you

```


In this example, the user has provided the prompt with the name: Sammy.


# Defining Functions


Users can define their own functions in a script. These functions can take multiple arguments.


Add the following lines to the script:


```
#!/bin/bash
#This is a comment

# defining a variable
echo "What is the name of the directory you want to create?"
# reading input 
read NAME

echo "Creating $NAME ..."
mkcd ()
{
  mkdir "$NAME" 
  cd "$NAME"
}

mkcd
echo "You are now in $NAME"

```


This script asks the user for a directory name. Then, it uses mkdir to create the directory and cd into it.


```
OutputWhat is the name of the directory you want to create?
test_dir
Creating test_dir ...
You are now in test_dir

```


In this example, the user has provided the prompt with the input: test_dir. Next, the script creates a new directory with that name. Finally, the script changes the user’s current working directory to test_dir.


# Conclusion


In this article, you learned how to create and execute shell scripts for the command line in Linux.


Consider some repetitive or time-consuming tasks that you frequently perform that could benefit from a script.


Continue your learning with if-else, arrays, and arguments in the command line.


