# Export Command in Linux

```UNIX/Linux```

In this guide, we will look at the export command in Linux. Export is a built-in command of the Bash shell. It is used to mark variables and functions to be passed to child processes. Basically, a variable will be included in child process environments without affecting other environments. To get a clearer picture of what we are talking about, let’s dive in and have a look at the export command examples.


# Export command in Linux without any arguments


Without any arguments, the command will generate or display all exported variables. Below is an example of the expected output.


```
$ export

```


Sample output 


# Viewing all exported variables on current shell


If you wish to view all exported variables on the current shell, use the -p flag as shown in the example


```
$ export -p 

```


Sample output 


# Using export with functions


Suppose you have a function and you wish to export it, how do you go about it? In this case , the -f flag is used. In this example, we are exporting the function name (). First, call the function


```
$ name () { echo "Hello world"; }

```


Then export it using the -f flag


```
$ export -f name

```


Next, invoke bash shell


```
$ bash

```


Finally, call the function


```
$ name

```


Output


```
Hello World

```


 You can also assign a value before exporting a function as shown


```
$ export name[=value]

```


For example, you can define a variable before exporting it as shown


```
$ student=Divya

```


In the above example, the variable ‘student’ has been assigned the value ‘Divya’ To export the variable run


```
$ export students

```


You can use the printenv command to verify the contents of the variable as shown


```
$ printenv students

```


Check the output below of the commands we have just executed Output  The above can be achieved in 2 simple steps by declaring and exporting the variable in one line as shown


```
$ export student=Divya

```


To display the variable run


```
$ printenv student

```


Output  This concludes our tutorial about export command. Go ahead and give it a try and see the magic! Your feedback is most welcome.


