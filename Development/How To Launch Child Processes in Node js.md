# How To Launch Child Processes in Node js

```JavaScript``` ```Node.js``` ```Development```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


When a user executes a single Node.js program, it runs as a single operating system (OS) process that represents the instance of the program running. Within that process, Node.js executes programs on a single thread. As mentioned earlier in this series with the How To Write Asynchronous Code in Node.js tutorial, because only one thread can run on one process, operations that take a long time to execute in JavaScript can block the Node.js thread and delay the execution of other code. A key strategy to work around this problem is to launch a child process, or a process created by another process, when faced with long-running tasks. When a new process is launched, the operating system can employ multiprocessing techniques to ensure that the main Node.js process and the additional child process run concurrently, or at the same time.


Node.js includes the child_process module, which has functions to create new processes. Aside from dealing with long-running tasks, this module can also interface with the OS and run shell commands. System administrators can use Node.js to run shell commands to structure and maintain their operations as a Node.js module instead of shell scripts.


In this tutorial, you will create child processes while executing a series of sample Node.js applications. You’ll create processes with the child_process module by retrieving the results of a child process via a buffer or string with the exec() function, and then from a data stream with the spawn() function. You’ll finish by using fork() to create a child process of another Node.js program that you can communicate with as it’s running. To illustrate these concepts, you will write a program to list the contents of a directory, a program to find files, and a web server with multiple endpoints.


# Prerequisites


- 
You must have Node.js installed to run through these examples. This tutorial uses version 10.22.0. To install this on macOS or Ubuntu 18.04, follow the steps in How To Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04.

- 
This article uses an example that creates a web server to explain how the fork() function works. To get familiar with creating web servers, you can read our guide on How To Create a Web Server in Node.js with the HTTP Module.


# Step 1 — Creating a Child Process with exec()


Developers commonly create child processes to execute commands on their operating system when they need to manipulate the output of their Node.js programs with a shell, such as using shell piping or redirection. The exec() function in Node.js creates a new shell process and executes a command in that shell. The output of the command is kept in a buffer in memory, which you can accept via a callback function passed into exec().


Let’s begin creating our first child processes in Node.js. First, we need to set up our coding environment to store the scripts we’ll create throughout this tutorial. In the terminal, create a folder called child-processes:


```
mkdir child-processes


```


Enter that folder in the terminal with the cd command:


```
cd child-processes


```


Create a new file called listFiles.js and open the file in a text editor. In this tutorial we will use nano, a terminal text editor:


```
nano listFiles.js


```


We’ll be writing a Node.js module that uses the exec() function to run the ls command. The ls command list the files and folders in a directory. This program takes the output from the ls command and displays it to the user.


In the text editor, add the following code:


~/child-processes/listFiles.js
```
const { exec } = require('child_process');

exec('ls -lh', (error, stdout, stderr) => {
  if (error) {
    console.error(`error: ${error.message}`);
    return;
  }

  if (stderr) {
    console.error(`stderr: ${stderr}`);
    return;
  }

  console.log(`stdout:\n${stdout}`);
});

```


We first import the exec() command from the child_process module using JavaScript destructuring. Once imported, we use the exec() function. The first argument is the command we would like to run. In this case, it’s ls -lh, which lists all the files and folders in the current directory in long format, with a total file size in human-readable units at the top of the output.


The second argument is a callback function with three parameters: error, stdout, and stderr. If the command failed to run, error will capture the reason why it failed. This can happen if the shell cannot find the command you’re trying to execute. If the command is executed successfully, any data it writes to the standard output stream is captured in stdout, and any data it writes to the standard error stream is captured in stderr.



Note: It’s important to keep the difference between error and stderr in mind. If the command itself fails to run, error will capture the error. If the command runs but returns output to the error stream, stderr will capture it. The most resilient Node.js programs will handle all possible outputs for a child process.

In our callback function, we first check if we received an error. If we did, we display the error’s message (a property of the Error object) with console.error() and end the function with return. We then check if the command printed an error message and return if so. If the command successfully executes, we log its output to the console with console.log().


Let’s run this file to see it in action. First, save and exit nano by pressing CTRL+X.


Back in your terminal, run your application with the node command:


```
node listFiles.js


```


Your terminal will display the following output:


```
Outputstdout:
total 4.0K
-rw-rw-r-- 1 sammy sammy 280 Jul 27 16:35 listFiles.js

```


This lists the contents of the child-processes directory in long format, along with the size of the contents at the top. Your results will have your own user and group in place of sammy. This shows that the listFiles.js program successfully ran the shell command ls -lh.


Now let’s look at another way to execute concurrent processes. Node.js’s child_process module can also run executable files with the execFile() function. The key difference between the execFile() and exec() functions is that the first argument of execFile() is now a path to an executable file instead of a command. The output of the executable file is stored in a buffer like exec(), which we access via a callback function with error, stdout, and stderr parameters.



Note: Scripts in Windows such as .bat and .cmd files cannot be run with execFile() because the function does not create a shell when running the file. On Unix, Linux, and macOS, executable scripts do not always need a shell to run. However, a Windows machines needs a shell to execute scripts. To execute script files on Windows, use exec(), since it creates a new shell. Alternatively, you can use spawn(), which you’ll use later in this Step.
However, note that you can execute .exe files in Windows successfully using execFile(). This limitation only applies to script files that require a shell to execute.

Let’s begin by adding an executable script for execFile() to run. We’ll write a bash script that downloads the Node.js logo from the Node.js website and Base64 encodes it to convert its data to a string of ASCII characters.


Create a new shell script file called processNodejsImage.sh:


```
nano processNodejsImage.sh


```


Now write a script to download the image and base64 convert it:


~/child-processes/processNodejsImage.sh
```
#!/bin/bash
curl -s https://nodejs.org/static/images/logos/nodejs-new-pantone-black.svg > nodejs-logo.svg
base64 nodejs-logo.svg

```


The first statement is a shebang statement. It’s used in Unix, Linux, and macOS when we want to specify a shell to execute our script. The second statement is a curl command. The cURL utility, whose command is curl, is a command-line tool that can transfer data to and from a server. We use cURL to download the Node.js logo from the website, and then we use redirection to save the downloaded data to a new file nodejs-logo.svg. The last statement uses the base64 utility to encode the nodejs-logo.svg file we downloaded with cURL. The script then outputs the encoded string to the console.


Save and exit before continuing.


In order for our Node program to run the bash script, we have to make it executable. To do this, run the following:


```
chmod u+x processNodejsImage.sh


```


This will give your current user the permission to execute the file.


With our script in place, we can write a new Node.js module to execute it. This script will use execFile() to run the script in a child process, catching any errors and displaying all output to console.


In your terminal, make a new JavaScript file called getNodejsImage.js:


```
nano getNodejsImage.js


```


Type the following code in the text editor:


~/child-processes/getNodejsImage.js
```
const { execFile } = require('child_process');

execFile(__dirname + '/processNodejsImage.sh', (error, stdout, stderr) => {
  if (error) {
    console.error(`error: ${error.message}`);
    return;
  }

  if (stderr) {
    console.error(`stderr: ${stderr}`);
    return;
  }

  console.log(`stdout:\n${stdout}`);
});

```


We use JavaScript destructuring to import the execFile() function from the child_process module. We then use that function, passing the file path as the first name. __dirname contains the directory path of the module in which it is written. Node.js provides the __dirname variable to a module when the module runs. By using __dirname, our script will always find the processNodejsImage.sh file across different operating systems, no matter where we run getNodejsImage.js. Note that for our current project setup, getNodejsImage.js and processNodejsImage.sh must be in the same folder.


The second argument is a callback with the error, stdout, and stderr parameters. Like with our previous example that used exec(), we check for each possible output of the script file and log them to the console.


In your text editor, save this file and exit from the editor.


In your terminal, use node to execute the module:


```
node getNodejsImage.js


```


Running this script will produce output like this:


```
Outputstdout:
PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIiB2aWV3Qm94PSIwIDAgNDQyLjQgMjcwLjkiPjxkZWZzPjxsaW5lYXJHcmFkaWVudCBpZD0iYiIgeDE9IjE4MC43IiB5MT0iODAuNyIge
...

```


Note that we truncated the output in this article because of its large size.


Before base64 encoding the image, processNodejsImage.sh first downloads it. You can also verify that you downloaded the image by inspecting the current directory.


Execute listFiles.js to find the updated list of files in our directory:


```
node listFiles.js


```


The script will display content similar to the following on the terminal:


```
Outputstdout:
total 20K
-rw-rw-r-- 1 sammy sammy  316 Jul 27 17:56 getNodejsImage.js
-rw-rw-r-- 1 sammy sammy  280 Jul 27 16:35 listFiles.js
-rw-rw-r-- 1 sammy sammy 5.4K Jul 27 18:01 nodejs-logo.svg
-rwxrw-r-- 1 sammy sammy  129 Jul 27 17:56 processNodejsImage.sh

```


We’ve now successfully executed processNodejsImage.sh as a child process in Node.js using the execFile() function.


The exec() and execFile() functions can run commands on the operating system’s shell in a Node.js child process. Node.js also provides another method with similar functionality, spawn(). The difference is that instead of getting the output of the shell commands all at once, we get them in chunks via a stream. In the next section we’ll use the spawn() command to create a child process.


# Step 2 — Creating a Child Process with spawn()


The spawn() function runs a command in a process. This function returns data via the stream API. Therefore, to get the output of the child process, we need to listen for stream events.


Streams in Node.js are instances of event emitters. If you would like to learn more about listening for events and the foundations of interacting with streams, you can read our guide on Using Event Emitters in Node.js.


It’s often a good idea to choose spawn() over exec() or execFile() when the command you want to run can output a large amount of data. With a buffer, as used by exec() and execFile(), all the processed data is stored in the computer’s memory. For large amounts of data, this can degrade system performance. With a stream, the data is processed and transferred in small chunks. Therefore, you can process a large amount of data without using too much memory at any one time.


Let’s see how we can use spawn() to make a child process. We will write a new Node.js module that creates a child process to run the find command. We will use the find command to list all the files in the current directory.


Create a new file called findFiles.js:


```
nano findFiles.js


```


In your text editor, begin by calling the spawn() command:


~/child-processes/findFiles.js
```
const { spawn } = require('child_process');

const child = spawn('find', ['.']);

```


We first imported the spawn() function from the child_process module. We then called the spawn() function to create a child process that executes the find command. We hold the reference to the process in the child variable, which we will use to listen to its streamed events.


The first argument in spawn() is the command to run, in this case find. The second argument is an array that contains the arguments for the executed command. In this case, we are telling Node.js to execute the find command with the argument ., thereby making the command find all the files in the current directory. The equivalent command in the terminal is find ..


With the exec() and execFile() functions, we wrote the arguments along with the command in one string. However, with spawn(), all arguments to commands must be entered in the array. That’s because spawn(), unlike exec() and execFile(), does not create a new shell before running a process. To have commands with their arguments in one string, you need Node.js to create a new shell as well.


Let’s continue our module by adding listeners for the command’s output. Add the following highlighted lines:


~/child-processes/findFiles.js
```
const { spawn } = require('child_process');

const child = spawn('find', ['.']);

child.stdout.on('data', data => {
  console.log(`stdout:\n${data}`);
});

child.stderr.on('data', data => {
  console.error(`stderr: ${data}`);
});

```


Commands can return data in either the stdout stream or the stderr stream, so you added listeners for both. You can add listeners by calling the on() method of each streams’ objects. The data event from the streams gives us the command’s output to that stream. Whenever we get data on either stream, we log it to the console.


We then listen to two other events: the error event if the command fails to execute or is interrupted, and the close event for when the command has finished execution, thus closing the stream.


In the text editor, complete the Node.js module by writing the following highlighted lines:


~/child-processes/findFiles.js
```
const { spawn } = require('child_process');

const child = spawn('find', ['.']);

child.stdout.on('data', (data) => {
  console.log(`stdout:\n${data}`);
});

child.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

child.on('error', (error) => {
  console.error(`error: ${error.message}`);
});

child.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});

```


For the error and close events, you set up a listener directly on the child variable. When listening for error events, if one occurs Node.js provides an Error object. In this case, you log the error’s message property.


When listening to the close event, Node.js provides the exit code of the command. An exit code denotes if the command ran successfully or not. When a command runs without errors, it returns the lowest possible value for an exit code: 0. When executed with an error, it returns a non-zero code.


The module is complete. Save and exit nano with CTRL+X.


Now, run the code with the node command:


```
node findFiles.js


```


Once complete, you will find the following output:


```
Outputstdout:
.
./findFiles.js
./listFiles.js
./nodejs-logo.svg
./processNodejsImage.sh
./getNodejsImage.js

child process exited with code 0

```


We find a list of all files in our current directory and the exit code of the command, which is 0 as it ran successfully. While our current directory has a small number of files, if we ran this code in our home directory, our program would list every single file in every accessible folder for our user. Because it has such a potentially large output, using the spawn() function is most ideal as its streams do not require as much memory as a large buffer.


So far we’ve used functions to create child processes to execute external commands in our operating system. Node.js also provides a way to create a child process that executes other Node.js programs. Let’s use the fork() function to create a child process for a Node.js module in the next section.


# Step 3 — Creating a Child Process with fork()


Node.js provides the fork() function, a variation of spawn(), to create a child process that’s also a Node.js process. The main benefit of using fork() to create a Node.js process over spawn() or exec() is that fork() enables communication between the parent and the child process.


With fork(), in addition to retrieving data from the child process, a parent process can send messages to the running child process. Likewise, the child process can send messages to the parent process.


Let’s see an example where using fork() to create a new Node.js child process can improve the performance of our application. Node.js programs run on a single process. Therefore, CPU intensive tasks like iterating over large loops or parsing large JSON files stop other JavaScript code from running. For certain applications, this is not a viable option. If a web server is blocked, then it cannot process any new incoming requests until the blocking code has completed its execution.


Let’s see this in practice by creating a web server with two endpoints. One endpoint will do a slow computation that blocks the Node.js process. The other endpoint will return a JSON object saying hello.


First, create a new file called httpServer.js, which will have the code for our HTTP server:


```
nano httpServer.js


```


We’ll begin by setting up the HTTP server. This involves importing the http module, creating a request listener function, creating a server object, and listening for requests on the server object. If you would like to dive deeper into creating HTTP servers in Node.js or would like a refresher, you can read our guide on How To Create a Web Server in Node.js with the HTTP Module.


Enter the following code in your text editor to set up an HTTP server:


~/child-processes/httpServer.js
```
const http = require('http');

const host = 'localhost';
const port = 8000;

const requestListener = function (req, res) {};

const server = http.createServer(requestListener);
server.listen(port, host, () => {
  console.log(`Server is running on http://${host}:${port}`);
});

```


This code sets up an HTTP server that will run at http://localhost:8000. It uses template literals to dynamically generate that URL.


Next, we will write an intentionally slow function that counts in a loop 5 billion times. Before the requestListener() function, add the following code:


~/child-processes/httpServer.js
```
...
const port = 8000;

const slowFunction = () => {
  let counter = 0;
  while (counter < 5000000000) {
    counter++;
  }

  return counter;
}

const requestListener = function (req, res) {};
...

```


This uses the arrow function syntax to create a while loop that counts to 5000000000.


To complete this module, we need to add code to the requestListener() function. Our function will call the slowFunction() on subpath, and return a small JSON message for the other. Add the following code to the module:


~/child-processes/httpServer.js
```
...
const requestListener = function (req, res) {
  if (req.url === '/total') {
    let slowResult = slowFunction();
    let message = `{"totalCount":${slowResult}}`;

    console.log('Returning /total results');
    res.setHeader('Content-Type', 'application/json');
    res.writeHead(200);
    res.end(message);
  } else if (req.url === '/hello') {
    console.log('Returning /hello results');
    res.setHeader('Content-Type', 'application/json');
    res.writeHead(200);
    res.end(`{"message":"hello"}`);
  }
};
...

```


If the user reaches the server at the /total subpath, then we run slowFunction(). If we are hit at the /hello subpath, we return this JSON message: {"message":"hello"}.


Save and exit the file by pressing CTRL+X.


To test, run this server module with node:


```
node httpServer.js


```


When our server starts, the console will display the following:


```
OutputServer is running on http://localhost:8000

```


Now, to test the performance of our module, open two additional terminals. In the first terminal, use the curl command to make a request to the /total endpoint, which we expect to be slow:


```
curl http://localhost:8000/total


```


In the other terminal, use curl to make a request to the /hello endpoint like this:


```
curl http://localhost:8000/hello


```


The first request will return the following JSON:


```
Output{"totalCount":5000000000}

```


Whereas the second request will return this JSON:


```
Output{"message":"hello"}

```


The request to /hello completed only after the request to /total. The slowFunction() blocked all other code from executing while it was still in its loop. You can verify this by looking at the Node.js server output that was logged in your original terminal:


```
OutputReturning /total results
Returning /hello results

```


To process the blocking code while still accepting incoming requests, we can move the blocking code to a child process with fork(). We will move the blocking code into its own module. The Node.js server will then create a child process when someone accesses the /total endpoint and listen for results from this child process.


Refactor the server by first creating a new module called getCount.js that will contain slowFunction():


```
nano getCount.js


```


Now enter the code for slowFunction() once again:


~/child-processes/getCount.js
```
const slowFunction = () => {
  let counter = 0;
  while (counter < 5000000000) {
    counter++;
  }

  return counter;
}

```


Since this module will be a child process created with fork(), we can also add code to communicate with the parent process when slowFunction() has completed processing. Add the following block of code that sends a message to the parent process with the JSON to return to the user:


~/child-processes/getCount.js
```
const slowFunction = () => {
  let counter = 0;
  while (counter < 5000000000) {
    counter++;
  }

  return counter;
}

process.on('message', (message) => {
  if (message == 'START') {
    console.log('Child process received START message');
    let slowResult = slowFunction();
    let message = `{"totalCount":${slowResult}}`;
    process.send(message);
  }
});

```


Let’s break down this block of code. The messages between a parent and child process created by fork() are accessible via the Node.js global process object. We add a listener to the process variable to look for message events. Once we receive a message event, we check if it’s the START event. Our server code will send the START event when someone accesses the /total endpoint. Upon receiving that event, we run slowFunction() and create a JSON string with the result of the function. We use process.send() to send a message to the parent process.


Save and exit getCount.js by entering CTRL+X in nano.


Now, let’s modify the httpServer.js file so that instead of calling slowFunction(), it creates a child process that executes getCount.js.


Re-open httpServer.js with nano:


```
nano httpServer.js


```


First, import the fork() function from the child_process module:


~/child-processes/httpServer.js
```
const http = require('http');
const { fork } = require('child_process');
...

```


Next, we are going to remove the slowFunction() from this module and modify the requestListener() function to create a child process. Change the code in your file so it looks like this:


~/child-processes/httpServer.js
```
...
const port = 8000;

const requestListener = function (req, res) {
  if (req.url === '/total') {
    const child = fork(__dirname + '/getCount');

    child.on('message', (message) => {
      console.log('Returning /total results');
      res.setHeader('Content-Type', 'application/json');
      res.writeHead(200);
      res.end(message);
    });

    child.send('START');
  } else if (req.url === '/hello') {
    console.log('Returning /hello results');
    res.setHeader('Content-Type', 'application/json');
    res.writeHead(200);
    res.end(`{"message":"hello"}`);
  }
};
...

```


When someone goes to the /total endpoint, we now create a new child process with fork(). The argument of fork() is the path to the Node.js module. In this case, it is the getCount.js file in our current directory, which we receive from __dirname. The reference to this child process is stored in a variable child.


We then add a listener to the child object. This listener captures any messages that the child process gives us. In this case, getCount.js will return a JSON string with the total number counted by the while loop. When we receive that message, we send the JSON to the user.


We use the send() function of the child variable to give it a message. This program sends the message START, which begins the execution of slowFunction() in the child process.


Save and exit nano by entering CTRL+X.


To test the improvement using fork() made on HTTP server, begin by executing the httpServer.js file with node:


```
node httpServer.js


```


Like before, it will output the following message when it launches:


```
OutputServer is running on http://localhost:8000

```


To test the server, we will need an additional two terminals as we did the first time. You can re-use them if they are still open.


In the first terminal, use the curl command to make a request to the /total endpoint, which takes a while to compute:


```
curl http://localhost:8000/total


```


In the other terminal, use curl to make a request to the /hello endpoint, which responds in a short time:


```
curl http://localhost:8000/hello


```


The first request will return the following JSON:


```
Output{"totalCount":5000000000}

```


Whereas the second request will return this JSON:


```
Output{"message":"hello"}

```


Unlike the first time we tried this, the second request to /hello runs immediately. You can confirm by reviewing the logs, which will look like this:


```
OutputChild process received START message
Returning /hello results
Returning /total results

```


These logs show that the request for the /hello endpoint ran after the child process was created but before the child process had finished its task.


Since we moved the blocking code in a child process using fork(), the server was still able to respond to other requests and execute other JavaScript code. Because of the fork() function’s message passing ability, we can control when a child process begins an activity and we can return data from a child process to a parent process.


# Conclusion


In this article, you used various functions to create a child process in Node.js. You first created child processes with exec() to run shell commands from Node.js code. You then ran an executable file with the execFile() function. You looked at the spawn() function, which can also run commands but returns data via a stream and does not start a shell like exec() and execFile(). Finally, you used the fork() function to allow for two-way communication between the parent and child process.


To learn more about the child_process module, you can read the Node.js documentation. If you’d like to continue learning Node.js, you can return to the How To Code in Node.js series, or browse programming projects and setups on our Node topic page.


