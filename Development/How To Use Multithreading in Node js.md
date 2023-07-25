# How To Use Multithreading in Node js

```Development``` ```JavaScript``` ```Node.js```

The author selected Open Sourcing Mental Illness to receive a donation as part of the Write for DOnations program.


## Introduction


Node.js runs JavaScript code in a single thread, which means that your code can only do one task at a time. However, Node.js itself is multithreaded and provides hidden threads through the libuv library, which handles I/O operations like reading files from a disk or network requests. Through the use of hidden threads, Node.js provides asynchronous methods that allow your code to make I/O requests without blocking the main thread.


Although Node.js has hidden threads, you cannot use them to offload CPU-intensive tasks, such as complex calculations, image resizing, or video compression. Since JavaScript is single-threaded when a CPU-intensive task runs, it blocks the main thread and no other code executes until the task completes. Without using other threads, the only way to speed up a CPU-bound task is to increase the processor speed.


However, in recent years, CPUs haven’t been getting faster. Instead, computers are shipping with extra cores, and it’s now more common for computers to have 8 or more cores. Despite this trend, your code will not take advantage of the extra cores on your computer to speed up CPU-bound tasks or avoid breaking the main thread because JavaScript is single-threaded.


To remedy this, Node.js introduced the worker-threads module, which allows you to create threads and execute multiple JavaScript tasks in parallel. Once a thread finishes a task, it sends a message to the main thread that contains the result of the operation so that it can be used with other parts of the code. The advantage of using worker threads is that CPU-bound tasks don’t block the main thread and you can divide and distribute a task to multiple workers to optimize it.


In this tutorial, you’ll create a Node.js app with a CPU-intensive task that blocks the main thread. Next, you will use the worker-threads module to offload the CPU-intensive task to another thread to avoid blocking the main thread. Finally, you will divide the CPU-bound task and have four threads work on it in parallel to speed up the task.


# Prerequisites


To complete this tutorial, you will need:


- 
A multi-core system with four or more cores. You can still follow the tutorial from Steps 1 through 6 on a dual-core system. However, Step 7 requires four cores to see the performance improvements.

- 
A Node.js development environment. If you’re on Ubuntu 22.04, install the recent version of Node.js by following step 3 of How To Install Node.js on Ubuntu 22.04. If you’re on another operating system, see How to Install Node.js and Create a Local Development Environment.

- 
A good understanding of the event loop, callbacks, and promises in JavaScript, which you can find in our tutorial, Understanding the Event Loop, Callbacks, Promises, and Async/Await in JavaScript.

- 
Basic knowledge of how to use the Express web framework. Check out our guide, How To Get Started with Node.js and Express.


# Setting up the Project and Installing Dependencies


In this step, you’ll create the project directory, initialize npm, and install all the necessary dependencies.


To begin, create and move into the project directory:


```
mkdir multi-threading_demo
cd multi-threading_demo


```


The mkdir command creates a directory and the cd command changes the working directory to the newly created one.


Following this, initialize the project directory with npm using the npm init command:


```
npm init -y


```


The -y option accepts all the default options.


When the command runs, your output will look similar to this:


```
Wrote to /home/sammy/multi-threading_demo/package.json:

{
  "name": "multi-threading_demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}


```


Next, install express, a Node.js web framework:


```
npm install express


```


You will use Express to create a server application that has blocking and non-blocking endpoints.


Node.js ships with the worker-threads module by default, so you don’t need to install it.


You’ve now installed the necessary packages. Next, you’ll learn more about processes and threads and how they execute on a computer.


# Understanding Processes and Threads


Before you start writing CPU-bound tasks and offloading them to separate threads, you first need to understand what processes and threads are, and the differences between them. Most importantly, you’ll review how the processes and threads execute on a single or multi-core computer system.


## Process


A process is a running program in the operating system. It has its own memory and cannot see nor access the memory of other running programs. It also has an instruction pointer, which indicates the instruction currently being executed in a program. Only one task can be executed at a time.


To understand this, you will create a Node.js program with an infinite loop so that it doesn’t exit when run.


Using nano, or your preferred text editor, create and open the process.js file:


```
nano process.js


```


In your process.js file, enter the following code:


multi-threading_demo/process.js
```
const process_name = process.argv.slice(2)[0];

count = 0;
while (true) {
  count++;
  if (count == 2000 || count == 4000) {
    console.log(`${process_name}: ${count}`);
  }
}


```


In the first line, the process.argv property returns an array containing the program command-line arguments. You then attach JavaScript’s slice() method with an argument of 2 to make a shallow copy of the array from index 2 onwards. Doing so skips the first two arguments, which are the Node.js path and the program filename. Next, you use the bracket notation syntax to retrieve the first argument from the sliced array and store it in the process_name variable.


After that, you define a while loop and pass it a true condition to make the loop run forever. Within the loop, the count variable is incremented by 1 during each iteration. Following this is an if statement that checks whether count is equal to 2000 or 4000. If the condition evaluates to true, console.log() method logs a message in the terminal.


Save and close your file using CTRL+X, then press Y to save the changes.


Run the program using the node command:


```
node process.js A &


```


A is a command-line argument that is passed to the program and stored in the process_name variable. The & at end the allows the Node program to run in the background, which lets you enter more commands in the shell.


When you run the program, you will see output similar to the following:


```
Output[1] 7754

A: 2000
A: 4000

```


The number 7754 is a process ID that the operating system assigned to it. A: 2000 and A: 4000 are the program’s output.


When you run a program using the node command, you create a process. The operating system allocates memory for the program, locates the program executable on your computer’s disk, and loads the program into memory. It then assigns it a process ID and begins executing the program. At that point, your program has now become a process.


When the process is running, its process ID is added to the process list of the operating system and can be seen with tools like htop, top, or ps. The tools provide more details about the processes, as well as options to stop or prioritize them.


To get a quick summary of a Node process, press ENTER in your terminal to get the prompt back. Next, run the ps command to see the Node processes:


```
ps |grep node


```


The ps command lists all processes associated with the current user on the system. The pipe operator | to pass all the ps output to the grep filters the processes to list only Node processes.


Running the command will yield output similar to the following:


```
Output7754 pts/0    00:21:49 node

```


You can create countless processes out of a single program. For example, use the following command to create three more processes with different arguments and put them in the background:


```
node process.js B & node process.js C & node process.js D &


```


In the command, you created three more instances of the process.js program. The & symbol puts each process in the background.


Upon running the command, the output will look similar to the following (although the order might differ):


```
Output[2] 7821
[3] 7822
[4] 7823

D: 2000
D: 4000
B: 2000
B: 4000
C: 2000
C: 4000

```


As you can see in the output, each process logged the process name into the terminal when the count reached 2000 and 4000. Each process is not aware of any other process running: process D isn’t aware of process C, and vice versa. Anything that happens in either process will not affect other Node.js processes.


If you examine the output closely, you will see that the order of the output isn’t the same order you had when you created the three processes. When running the command, the processes arguments were in order of B, C, and D. But now, the order is D, B, and C. The reason is that the OS has scheduling algorithms that decide which process to run on the CPU at a given time.


On a single core machine, the processes execute concurrently. That is, the operating system switches between the processes in regular intervals. For example, process D executes for a limited time, then its state is saved somewhere and the OS schedules process B to execute for a limited time, and so on. This happens back and forth until all the tasks have been finished. From the output, it might look like each process has run to completion, but in reality, the OS scheduler is constantly switching between them.


On a multi-core system—assuming you have four cores—the OS schedules each process to execute on each core at the same time. This is known as parallelism. However, if you create four more processes (bringing the total to eight), each core will execute two processes concurrently until they are finished.


## Threads


Threads are like processes: they have their own instruction pointer and can execute one JavaScript task at a time. Unlike processes, threads do not have their own memory. Instead, they reside within a process’s memory. When you create a process, it can have multiple threads created with the worker_threads module executing JavaScript code in parallel. Furthermore, threads can communicate with one another through message passing or sharing data in the process’s memory. This makes them lightweight in comparison to processes, since spawning a thread does not ask for more memory from the operating system.


When it comes to the execution of threads, they have similar behavior to that of processes. If you have multiple threads running on a single core system, the operating system will switch between them in regular intervals, giving each thread a chance to execute directly on the single CPU. On a multi-core system, the OS schedules the threads across all cores and executes the JavaScript code at the same time. If you end up creating more threads than there are cores available, each core will execute multiple threads concurrently.


With that, press ENTER, then stop all the currently running Node processes with the kill command:


```
sudo kill -9 `pgrep node`


```


pgrep returns the process ID’s of all the four Node processes to the kill command. The -9 option instructs kill to send a SIGKILL signal.


When you run the command, you will see output similar to the following:


```
Output[1]   Killed                  node process.js A
[2]   Killed                  node process.js B
[3]   Killed                  node process.js C
[4]   Killed                  node process.js D

```


Sometimes the output might be delayed and show up when you run another command later.


Now that you know the difference between a process and a thread, you’ll work with Node.js hidden threads in the next section.


# Understanding Hidden Threads in Node.js


Node.js does provide extra threads, which is why it’s considered to be multithreaded. In this section, you’ll examine hidden threads in Node.js, which help make I/O operations non-blocking.


As mentioned in the introduction, JavaScript is single-threaded and all the JavaScript code executes in a single thread. This includes your program source code and third-party libraries that you include in your program. When a program makes an I/O operation to read a file or a network request, this blocks the main thread.


However, Node.js implements the libuv library, which provides four extra threads to a Node.js process. With these threads, the I/O operations are handled separately and when they are finished, the event loop adds the callback associated with the I/O task in a microtask queue. When the call stack in the main thread is clear, the callback is pushed on the call stack and then it executes. To make this clear, the callback associated with the given I/O task does not execute in parallel; however, the task itself of reading a file or a network request happens in parallel with the help of the threads. Once the I/O task finishes, the callback runs in the main thread.


In addition to these four threads, the V8 engine, also provides two threads for handling things like automatic garbage collection. This brings the total number of threads in a process to seven: one main thread, four Node.js threads, and two V8 threads.


To confirm that every Node.js process has seven threads, run the process.js file again and put it in the background:


```
node process.js A &


```


The terminal will log the process ID, as well as output from the program:


```
Output[1] 9933

A: 2000
A: 4000

```


Note the process ID somewhere and press ENTER so that you can use the prompt again.


To see the threads, run the top command and pass it the process ID displayed in the output:


```
top -H -p 9933


```


-H instructs top to display threads in a process. The -p flag instructs top to monitor only the activity in the given process ID.


When you run the command, your output will look similar to the following:


```
Outputtop - 09:21:11 up 15:00,  1 user,  load average: 0.99, 0.60, 0.26
Threads:   7 total,   1 running,   6 sleeping,   0 stopped,   0 zombie
%Cpu(s): 24.8 us,  0.3 sy,  0.0 ni, 75.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7951.2 total,   6756.1 free,    248.4 used,    946.7 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   7457.4 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   9933 node-us+  20   0  597936  51864  33956 R  99.9   0.6   4:19.64 node
   9934 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.00 node
   9935 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.84 node
   9936 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.83 node
   9937 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.93 node
   9938 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.83 node
   9939 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.00 node

```


As you can see in the output, the Node.js process has seven threads in total: one main thread for executing JavaScript, four Node.js threads, and two V8 threads.


As discussed previously, the four Node.js threads are used for I/O operations to make them non-blocking. They work well for that task, and creating threads yourself for I/O operations may even worsen your application performance. The same cannot be said about CPU-bound tasks. A CPU-bound task does not make use of any extra threads available in the process and blocks the main thread.


Now press q to exit top and stop the Node process with the following command:


```
kill -9 9933


```


Now that you know about the threads in a Node.js process, you will write a CPU-bound task in the next section and observe how it affects the main thread.


# Creating a CPU-Bound Task Without Worker Threads


In this section, you will build an Express app that has a non-blocking route and a blocking route that runs a CPU-bound task.


First, open index.js in your preferred editor:


```
nano index.js


```


In your index.js file, add the following code to create a basic server:


multi-threading_demo/index.js
```
const express = require("express");

const app = express();
const port = process.env.PORT || 3000;

app.get("/non-blocking/", (req, res) => {
  res.status(200).send("This page is non-blocking");
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});


```


In the proceeding code block, you create an HTTP server using Express. In the first line, you import the express module. Next, you set the app variable to hold an instance of Express. After that, you define the port variable, which holds the port number the server should listen on.


Following this, you use app.get('/non-blocking') to define the route GET requests should be sent. Finally, you invoke the app.listen() method to instruct the server to start listening on port 3000.


Next, define another route, /blocking/, which will contain a CPU-intensive task:


multi-threading_demo/index.js
```
...
app.get("/blocking", async (req, res) => {
  let counter = 0;
  for (let i = 0; i < 20_000_000_000; i++) {
    counter++;
  }
  res.status(200).send(`result is ${counter}`);
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});

```


You define the /blocking route using app.get("/blocking"), which takes an asynchronous callback prefixed with the async keyword as a second argument that runs a CPU-intensive task. Within the callback, you create a for loop that iterates 20 billion times and during each iteration, it increments the counter variable by 1. This task runs on the CPU and will take a couple of seconds to complete.


At this point, your index.js file will now look like this:


multi-threading_demo/index.js
```
const express = require("express");

const app = express();
const port = process.env.PORT || 3000;

app.get("/non-blocking/", (req, res) => {
  res.status(200).send("This page is non-blocking");
});

app.get("/blocking", async (req, res) => {
  let counter = 0;
  for (let i = 0; i < 20_000_000_000; i++) {
    counter++;
  }
  res.status(200).send(`result is ${counter}`);
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});

```


Save and exit your file, then start the server with the following command:


```
node index.js


```


When you run the command, you will see output similar to the following:


```
OutputApp listening on port 3000

```


This shows that the server is running and ready to serve.


Now, visit http://localhost:3000/non-blocking in your preferred browser. You will see an instant response with the message This page is non-blocking.



Note: If you are following the tutorial on a remote server, you can use port forwarding to test the app in the browser.
While the Express server is still running, open another terminal on your local computer and enter the following command:
ssh -L 3000:localhost:3000  your-non-root-user@yourserver-ip


Upon connecting to the server, navigate to http://localhost:3000/non-blocking on your local machine’s web browser. Keep the second terminal open throughout the remainder of this tutorial.

Next, open a new tab and visit http://localhost:3000/blocking. As the page loads, quickly open two more tabs and visit http://localhost:3000/non-blocking again. You will see that you won’t get an instant response, and the pages will keep trying to load. It is only after the /blocking route finishes loading and returns a response result is 20000000000 that the rest of the routes will return a response.


The reason why all the /non-blocking routes don’t work as the /blocking route loads is because of the CPU-bound for loop, which blocks the main thread. When the main thread is blocked, Node.js cannot serve any requests until the CPU-bound task has finished. So if your application has thousands of simultaneous GET requests to the /non-blocking route, a single visit to the /blocking route is all it takes to make all the application routes non-responsive.


As you can see, blocking the main thread can harm the user’s experience with your app. To solve this issue, you will need to offload the CPU-bound task to another thread so that the main thread can continue handling other HTTP requests.


With that, stop the server by pressing CTRL+C. You will start the server again in the next section after making more changes to the index.js file. The reason why the server is stopped is that Node.js does not automatically refresh when new changes to the file are made.


Now that you understand the negative impact a CPU-intensive task can have on your application, you will now try to avoid blocking the main thread by using promises.


# Offloading a CPU-Bound Task Using Promises


Often when developers learn about the blocking effect from CPU-bound tasks, they turn to promises to make the code non-blocking. This instinct stems from the knowledge of using non-blocking promise-based I/O methods, such as readFile() and writeFile(). But as you have learned, the I/O operations make use of Node.js hidden threads, which CPU-bound tasks do not. Nevertheless, in this section, you will wrap the CPU-bound task in a promise as an attempt to make it non-blocking. It won’t work, but it will help you to see the value of using worker threads, which you will do in the next section.


Open the index.js file again in your editor:


```
nano index.js


```


In your index.js file, remove the highlighted code containing the CPU-intensive task:


multi-threading_demo/index.js
```
...
app.get("/blocking", async (req, res) => {
  let counter = 0;
  for (let i = 0; i < 20_000_000_000; i++) {
    counter++;
  }
  res.status(200).send(`result is ${counter}`);
});
...

```


Next, add the following highlighted code containing a function that returns a promise:


multi-threading_demo/index.js
```
...
function calculateCount() {
  return new Promise((resolve, reject) => {
    let counter = 0;
    for (let i = 0; i < 20_000_000_000; i++) {
      counter++;
    }
    resolve(counter);
  });
}

app.get("/blocking", async (req, res) => {
  res.status(200).send(`result is ${counter}`);
}

```


The calculateCount() function now contains the calculations you had in the /blocking handler function. The function returns a promise, which is initialized with the new Promise syntax. The promise takes a callback with resolve and reject parameters, which handle success or failure. When the for loop finishes running, the promise resolves with the value in the counter variable.


Next, call the calculateCount() function in the /blocking/ handler function in the index.js file:


multi-threading_demo/index.js
```
app.get("/blocking", async (req, res) => {
  const counter = await calculateCount();
  res.status(200).send(`result is ${counter}`);
});

```


Here you call the calculateCount() function with the await keyword prefixed to wait for the promise to resolve. Once the promise resolves, the counter variable is set to the resolved value.


Your complete code will now look like the following:


multi-threading_demo/index.js
```
const express = require("express");

const app = express();
const port = process.env.PORT || 3000;

app.get("/non-blocking/", (req, res) => {
  res.status(200).send("This page is non-blocking");
});

function calculateCount() {
  return new Promise((resolve, reject) => {
    let counter = 0;
    for (let i = 0; i < 20_000_000_000; i++) {
      counter++;
    }
    resolve(counter);
  });
}

app.get("/blocking", async (req, res) => {
  const counter = await calculateCount();
  res.status(200).send(`result is ${counter}`);
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});

```


Save and exit your file, then start the server again:


```
node index.js


```


In your web browser, visit http://localhost:3000/blocking and as it loads, quickly reload the http://localhost:3000/non-blocking tabs. As you will notice, the non-blocking routes are still affected and they will all wait for the /blocking route to finish loading. Because the routes are still affected, promises don’t make JavaScript code execute in parallel and cannot be used to make CPU-bound tasks non-blocking.


With that, stop the application server with CTRL+C.


Now that you know promises do not provide any mechanism to make CPU-bound tasks non-blocking, you will use the Node.js worker-threads module to offload a CPU-bound task into a separate thread.


# Offloading a CPU-Bound Task with the worker-threads Module


In this section, you will offload a CPU-intensive task to another thread using the worker-threads module to avoid blocking the main thread. To do this, you will create a worker.js file that will contain the CPU-intensive task. In the index.js file, you will use the worker-threads module to initialize the thread and start the task in the worker.js file to run in parallel to the main thread. Once the task completes, the worker thread will send a message containing the result back to the main thread.


To begin, verify that you have 2 or more cores using the nproc command:


```
nproc


```


```
Output4

```


If it shows two or more cores, you can proceed with this step.


Next, create and open the worker.js file in your text editor:


```
nano worker.js


```


In your worker.js file, add the following code to import the worker-threads module and do the CPU-intensive task:


multi-threading_demo/worker.js
```
const { parentPort } = require("worker_threads");

let counter = 0;
for (let i = 0; i < 20_000_000_000; i++) {
  counter++;
}

```


The first line loads the worker_threads module and extracts the parentPort class. The class provides methods you can use to send messages to the main thread. Next, you have the CPU-intensive task that is currenty in the calculateCount() function in the index.js file. Later in this step, you will delete this function from index.js.


Following this, add the highlighted code below:


multi-threading_demo/worker.js
```
const { parentPort } = require("worker_threads");

let counter = 0;
for (let i = 0; i < 20_000_000_000; i++) {
  counter++;
}

parentPort.postMessage(counter);

```


Here you invoke the postMessage() method of the parentPort class, which sends a message to the main thread containing the result of the CPU-bound task stored in the counter variable.


Save and exit your file. Open index.js in your text editor:


```
nano index.js


```


Since you already have the CPU-bound task in worker.js, remove the highlighted code from index.js:


multi-threading_demo/index.js
```
const express = require("express");

const app = express();
const port = process.env.PORT || 3000;

app.get("/non-blocking/", (req, res) => {
  res.status(200).send("This page is non-blocking");
});

function calculateCount() {
  return new Promise((resolve, reject) => {
    let counter = 0;
    for (let i = 0; i < 20_000_000_000; i++) {
      counter++;
    }
    resolve(counter);
  });
}

app.get("/blocking", async (req, res) => {
  const counter = await calculateCount();
  res.status(200).send(`result is ${counter}`);
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});

```


Next, in the app.get("/blocking") callback, add the following code to initialize the thread:


multi-threading_demo/index.js
```
const express = require("express");
const { Worker } = require("worker_threads");
...
app.get("/blocking", async (req, res) => {
  const worker = new Worker("./worker.js");
  worker.on("message", (data) => {
    res.status(200).send(`result is ${data}`);
  });
  worker.on("error", (msg) => {
    res.status(404).send(`An error occurred: ${msg}`);
  });
});
...

```


First, you import the worker_threads module and unpack the Worker class. Within the app.get("/blocking") callback, you create an instance of the Worker using the new keyword that is followed by a call to Worker with the worker.js file path as its argument. This creates a new thread and the code in the worker.js file starts running in the thread on another core.


Following this, you attach an event to the worker instance using the on("message") method to listen to the message event. When the message is received containing the result from the worker.js file, it is passed as a parameter to the method’s callback, which returns a response to the user containing the result of the CPU-bound task.


Next, you attach another event to the worker instance using the on("error") method to listen to the error event. If an error occurs, the callback returns a 404 response containing the error message back to the user.


Your complete file will now look like the following:


multi-threading_demo/index.js
```
const express = require("express");
const { Worker } = require("worker_threads");

const app = express();
const port = process.env.PORT || 3000;

app.get("/non-blocking/", (req, res) => {
  res.status(200).send("This page is non-blocking");
});

app.get("/blocking", async (req, res) => {
  const worker = new Worker("./worker.js");
  worker.on("message", (data) => {
    res.status(200).send(`result is ${data}`);
  });
  worker.on("error", (msg) => {
    res.status(404).send(`An error occurred: ${msg}`);
  });
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});

```


Save and exit your file, then run the server:


```
node index.js


```


Visit the http://localhost:3000/blocking tab again in your web browser. Before it finishes loading, refresh all http://localhost:3000/non-blocking tabs. You should now notice that they are loading instantly without waiting for the /blocking route to finish loading. This is because the CPU-bound task is offloaded to another thread, and the main thread handles all the incoming requests.


Now, stop your server using CTRL+C.


Now that you can make a CPU-intensive task non-blocking using a worker thread, you’ll use four worker threads to improve the performance of the CPU-intensive task.


# Optimizing a CPU-Intensive Task Using Four Worker Threads


In this section, you will divide the CPU-intensive task among four worker threads so that they can finish the task faster and shorten the load time of the /blocking route.


To have more worker threads work on the same task, you will need to split the tasks. Since the task involves looping 20 billion times, you will divide 20 billion with the number of threads you want to use. In this case, it is 4. Computing 20_000_000_000 / 4 will result in 5_000_000_000. So each thread will loop from 0 to 5_000_000_000 and increment counter by 1. When each thread finishes, it will send a message to the main thread containing the result. Once the main thread receives messages from all the four threads separately, you will combine the results and send a response to the user.


You can also use the same approach if you have a task that iterates over large arrays. For example, if you wanted to resize 800 images in a directory, you can create an array containing all the image file paths. Next, divide 800 by 4(the thread count) and have each thread work on a range. Thread one will resize images from the array index 0 to 199, thread two from index 200 to 399, and so on.


First, verify that you have four or more cores:


```
nproc


```


```
Output4

```


Make a copy of the worker.js file using the cp command:


```
cp worker.js four_workers.js


```


The current index.js and worker.js files will be left intact so that you can run them again to compare their performance with changes in this section later.


Next, open the four_workers.js file in your text editor:


```
nano four_workers.js


```


In your four_workers.js file, add the highlighted code to import the workerData object:


multi-threading_demo/four_workers.js
```
const { workerData, parentPort } = require("worker_threads");

let counter = 0;
for (let i = 0; i < 20_000_000_000 / workerData.thread_count; i++) {
  counter++;
}

parentPort.postMessage(counter);

```


First, you extract the WorkerData object, which will contain the data passed from the main thread when the thread is initialized (which you will do soon in the index.js file). The object has a thread_count property that contains the number of threads, which is 4. Next in the for loop, the value 20_000_000_000 is divided by 4, resulting in 5_000_000_000.


Save and close your file, then copy the index.js file:


```
cp index.js index_four_workers.js


```


Open the  index_four_workers.js file in your editor:


```
nano index_four_workers.js


```


In your index_four_workers.js file, add the highlighted code to create a thread instance:


multi-threading_demo/index_four_workers.js
```
...
const app = express();
const port = process.env.PORT || 3000;
const THREAD_COUNT = 4;
...
function createWorker() {
  return new Promise(function (resolve, reject) {
    const worker = new Worker("./four_workers.js", {
      workerData: { thread_count: THREAD_COUNT },
    });
  });
}

app.get("/blocking", async (req, res) => {
  ...
})
...

```


First, you define the THREAD_COUNT constant containing the number of threads you want to create. Later when you have more cores on your server, scaling will involve changing the value of the THREAD_COUNT to the number of threads you want to use.


Next, the createWorker() function creates and returns a promise. Within the promise callback, you initialize a new thread by passing the Worker class the file path to the four_workers.js file as the first argument. You then pass an object as the second argument. Next, you assign the object the workerData property that has another object as its value. Finally, you assign the object the thread_count property whose value is the number of threads in the THREAD_COUNT constant. The workerData object is the one you referenced in the workers.js file earlier.


To make sure the promise resolves or throws an error, add the following highlighted lines:


multi-threading_demo/index_four_workers.js
```
...
function createWorker() {
  return new Promise(function (resolve, reject) {
    const worker = new Worker("./four_workers.js", {
      workerData: { thread_count: THREAD_COUNT },
    });
    worker.on("message", (data) => {
      resolve(data);
    });
    worker.on("error", (msg) => {
      reject(`An error ocurred: ${msg}`);
    });
  });
}
...

```


When the worker thread sends a message to the main thread, the promise resolves with the data returned. However, if an error occurs, the promise returns an error message.


Now that you have defined the function that initializes a new thread and returns the data from the thread, you’ll use the function in app.get("/blocking") to spawn new threads.


But first, remove the following highlighted code, since you have already defined this functionality in the createWorker() function:


multi-threading_demo/index_four_workers.js
```
...
app.get("/blocking", async (req, res) => {
  const worker = new Worker("./worker.js");
  worker.on("message", (data) => {
    res.status(200).send(`result is ${data}`);
  });
  worker.on("error", (msg) => {
    res.status(404).send(`An error ocurred: ${msg}`);
  });
});
...

```


With the code deleted, add the following code to initialize four work threads:


multi-threading_demo/index_four_workers.js
```
...
app.get("/blocking", async (req, res) => {
  const workerPromises = [];
  for (let i = 0; i < THREAD_COUNT; i++) {
    workerPromises.push(createWorker());
  }
});
...

```


First, you create a workerPromises variable, which contains an empty array. Next, you iterate as many times as the value in THREAD_COUNT, which is 4. During each iteration, you call the createWorker() function to create a new thread. You then push the promise object that the function returns into the workerPromises array using JavaScript’s push method. When the loop finishes, the workerPromises will have four promise objects each returned from calling the createWorker() function four times.


Now, add the following highlighted code below to wait for the promises to resolve and return a response to the user:


multi-threading_demo/index_four_workers.js
```
app.get("/blocking", async (req, res) => {
  const workerPromises = [];
  for (let i = 0; i < THREAD_COUNT; i++) {
    workerPromises.push(createWorker());
  }

  const thread_results = await Promise.all(workerPromises);
  const total =
    thread_results[0] +
    thread_results[1] +
    thread_results[2] +
    thread_results[3];
  res.status(200).send(`result is ${total}`);
});

```


Since the workerPromises array contain promises returned calling createWorker(), you prefix the Promise.all() method with the await syntax and call the all() method with workerPromises as its argument. The Promise.all() method waits for all promises in the array to resolve. When that happens, the thread_results variable contains the values that the promises resolved. Since the calculations were split among four workers, you add them all together by getting each value from the thread_results using the bracket notation syntax. Once added, you return the total value to the page.


Your complete file should now look like this:


multi-threading_demo/index_four_workers.js
```
const express = require("express");
const { Worker } = require("worker_threads");

const app = express();
const port = process.env.PORT || 3000;
const THREAD_COUNT = 4;

app.get("/non-blocking/", (req, res) => {
  res.status(200).send("This page is non-blocking");
});

function createWorker() {
  return new Promise(function (resolve, reject) {
    const worker = new Worker("./four_workers.js", {
      workerData: { thread_count: THREAD_COUNT },
    });
    worker.on("message", (data) => {
      resolve(data);
    });
    worker.on("error", (msg) => {
      reject(`An error ocurred: ${msg}`);
    });
  });
}

app.get("/blocking", async (req, res) => {
  const workerPromises = [];
  for (let i = 0; i < THREAD_COUNT; i++) {
    workerPromises.push(createWorker());
  }
  const thread_results = await Promise.all(workerPromises);
  const total =
    thread_results[0] +
    thread_results[1] +
    thread_results[2] +
    thread_results[3];
  res.status(200).send(`result is ${total}`);
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});

```


Save and close your file. Before you run this file, first run index.js to measure its response time:


```
node index.js


```


Next, open a new terminal on your local computer and enter the following curl command, which measures how long it takes to get a response from the /blocking route:


```
time curl --get http://localhost:3000/blocking


```


The time command measures how long the curl command runs. The curl command sends an HTTP request to the given URL and the --get option instructs curl to make a GET request.


When the command runs, your output will look similar to this:


```
Outputreal	0m28.882s
user	0m0.018s
sys	0m0.000s

```


The highlighted output shows that it takes about 28 seconds to get a response, which might vary on your computer.


Next, stop the server with CTRL+C and run the index_four_workers.js file:


```
node index_four_workers.js


```


Visit the /blocking route again in your second terminal:


```
time curl --get http://localhost:3000/blocking


```


You will see output consistent with the following:


```
Outputreal	0m8.491s
user	0m0.011s
sys	0m0.005s

```


The output shows that it takes a about 8 seconds, which means you cut down the load time by roughly 70%.


You successfully optimized the CPU-bound task using four worker threads. If you have a machine with more than four cores, update the THREAD_COUNT to that number and you will cut the load time even further.


# Conclusion


In this article, you built a Node app with a CPU-bound task that blocks the main thread. You then tried to make the task non-blocking using promises, which was unsuccessful. After that, you used the worker_threads module to offload the CPU-bound task to another thread to make it non-blocking. Finally, you used the worker_threads module to create four threads to speed up the CPU-intensive task.


As a next step, see the Node.js Worker threads documentation to learn more about options. In addition, you can check out the piscina library, which allows you to create a worker pool for your CPU-intensive tasks. If you want to continue learning Node.js, see the tutorial series, How To Code in Node.js.


