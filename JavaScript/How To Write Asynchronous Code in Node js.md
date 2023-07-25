# How To Write Asynchronous Code in Node js

```JavaScript``` ```Node.js``` ```Development```

The author selected the Open Internet/Free Speech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


For many programs in JavaScript, code is executed as the developer writes it—line by line. This is called synchronous execution, because the lines are executed one after the other, in the order they were written. However, not every instruction you give to the computer needs to be attended to immediately. For example, if you send a network request, the process executing your code will have to wait for the data to return before it can work on it. In this case, time would be wasted if it did not execute other code while waiting for the network request to be completed. To solve this problem, developers use asynchronous programming, in which lines of code are executed in a different order than the one in which they were written. With asynchronous programming, we can execute other code while we wait for long activities like network requests to finish.


JavaScript code is executed on a single thread within a computer process. Its code is processed synchronously on this thread, with only one instruction run at a time. Therefore, if we were to do a long-running task on this thread, all of the remaining code is blocked until the task is complete. By leveraging JavaScript’s asynchronous programming features, we can offload long-running tasks to a background thread to avoid this problem. When the task is complete, the code we need to process the task’s data is put back on the main single thread.


In this tutorial, you will learn how JavaScript manages asynchronous tasks with help from the Event Loop, which is a JavaScript construct that completes a new task while waiting for another. You will then create a program that uses asynchronous programming to request a list of movies from a Studio Ghibli API and save the data to a CSV file. The asynchronous code will be written in three ways: callbacks, promises, and with the async/await keywords.



Note: As of this writing, asynchronous programming is no longer done using only callbacks, but learning this obsolete method can provide great context as to why the JavaScript community now uses promises. The async/await keywords enable us to use promises in a less verbose way, and are thus the standard way to do asynchronous programming in JavaScript at the time of writing this article.

# Prerequisites


- Node.js installed on your development machine. This tutorial uses version 10.17.0. To install this on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04.
- You will also need to be familiar with installing packages in your project. Get up to speed by reading our guide on How To Use Node.js Modules with npm and package.json.
- It is important that you’re comfortable creating and executing functions in JavaScript before learning how to use them asynchronously. If you need an introduction or refresher, you can read our guide on How To Define Functions in JavaScript

# The Event Loop


Let’s begin by studying the internal workings of JavaScript function execution. Understanding how this behaves will allow you to write asynchronous code more deliberately, and will help you with troubleshooting code in the future.


As the JavaScript interpreter executes the code, every function that is called is added to JavaScript’s call stack. The call stack is a stack—a list-like data structure where items can only be added to the top, and removed from the top. Stacks follow the “Last in, first out” or LIFO principle. If you add two items on the stack, the most recently added item is removed first.


Let’s illustrate with an example using the call stack. If JavaScript encounters a function functionA() being called, it is added to the call stack. If that function functionA() calls another function functionB(), then functionB() is added to the top of the call stack. As JavaScript completes the execution of a function, it is removed from the call stack. Therefore, JavaScript will execute functionB() first, remove it from the stack when complete, and then finish the execution of functionA() and remove it from the call stack. This is why inner functions are always executed before their outer functions.


When JavaScript encounters an asynchronous operation, like writing to a file, it adds it to a table in its memory. This table stores the operation, the condition for it to be completed, and the function to be called when it’s completed. As the operation completes, JavaScript adds the associated function to the message queue. A queue is another list-like data structure where items can only be added to the bottom but removed from the top. In the message queue, if two or more asynchronous operations are ready for their functions to be executed, the asynchronous operation that was completed first will have its function marked for execution first.


Functions in the message queue are waiting to be added to the call stack. The event loop is a perpetual process that checks if the call stack is empty. If it is, then the first item in the message queue is moved to the call stack. JavaScript prioritizes functions in the message queue over function calls it interprets in the code. The combined effect of the call stack, message queue, and event loop allows JavaScript code to be processed while managing asynchronous activities.


Now that you have a high-level understanding of the event loop, you know how the asynchronous code you write will be executed. With this knowledge, you can now create asynchronous code with three different approaches: callbacks, promises, and async/await.


# Asynchronous Programming with Callbacks


A callback function is one that is passed as an argument to another function, and then executed when the other function is finished. We use callbacks to ensure that code is executed only after an asynchronous operation is completed.


For a long time, callbacks were the most common mechanism for writing asynchronous code, but now they have largely become obsolete because they can make code confusing to read. In this step, you’ll write an example of asynchronous code using callbacks so that you can use it as a baseline to see the increased efficiency of other strategies.


There are many ways to use callback functions in another function. Generally, they take this structure:


```
function asynchronousFunction([ Function Arguments ], [ Callback Function ]) {
    [ Action ]
}

```


While it is not syntactically required by JavaScript or Node.js to have the callback function as the last argument of the outer function, it is a common practice that makes callbacks easier to identify. It’s also common for JavaScript developers to use an anonymous function as a callback. Anonymous functions are those created without a name. It’s usually much more readable when a function is defined at the end of the argument list.


To demonstrate callbacks, let’s create a Node.js module that writes a list of Studio Ghibli movies to a file. First, create a folder that will store our JavaScript file and its output:


```
mkdir ghibliMovies


```


Then enter that folder:


```
cd ghibliMovies


```


We will start by making an HTTP request to the Studio Ghibli API, which our callback function will log the results of. To do this, we will install a library that allows us to access the data of an HTTP response in a callback.


In your terminal, initialize npm so we can have a reference for our packages later:


```
npm init -y


```


Then, install the request library:


```
npm i request --save


```


Now open a new file called callbackMovies.js in a text editor like nano:


```
nano callbackMovies.js


```


In your text editor, enter the following code. Let’s begin by sending an HTTP request with the request module:


callbackMovies.js
```
const request = require('request');

request('https://ghibliapi.herokuapp.com/films');

```


In the first line, we load the request module that was installed via npm. The module returns a function that can make HTTP requests; we then save that function in the request constant.


We then make the HTTP request using the request() function. Let’s now print the data from the HTTP request to the console by adding the highlighted changes:


callbackMovies.js
```
const request = require('request');

request('https://ghibliapi.herokuapp.com/films', (error, response, body) => {
    if (error) {
        console.error(`Could not send request to API: ${error.message}`);
        return;
    }

    if (response.statusCode != 200) {
        console.error(`Expected status code 200 but received ${response.statusCode}.`);
        return;
    }

    console.log('Processing our list of movies');
    movies = JSON.parse(body);
    movies.forEach(movie => {
        console.log(`${movie['title']}, ${movie['release_date']}`);
    });
});

```


When we use the request() function, we give it two parameters:


- The URL of the website we are trying to request
- A callback function that handles any errors or successful responses after the request is complete

Our callback function has three arguments: error, response, and body. When the HTTP request is complete, the arguments are automatically given values depending on the outcome. If the request failed to send, then error would contain an object, but response and body would be null. If it made the request successfully, then the HTTP response is stored in response. If our HTTP response returns data (in this example we get JSON) then the data is set in body.


Our callback function first checks to see if we received an error. It’s best practice to check for errors in a callback first so the execution of the callback won’t continue with missing data. In this case, we log the error and the function’s execution. We then check the status code of the response. Our server may not always be available, and APIs can change causing once sensible requests to become incorrect. By checking that the status code is 200, which means the request was “OK”, we can have confidence that our response is what we expect it to be.


Finally, we parse the response body to an Array and loop through each movie to log its name and release year.


After saving and quitting the file, run this script with:


```
node callbackMovies.js


```


You will get the following output:


```
OutputCastle in the Sky, 1986
Grave of the Fireflies, 1988
My Neighbor Totoro, 1988
Kiki's Delivery Service, 1989
Only Yesterday, 1991
Porco Rosso, 1992
Pom Poko, 1994
Whisper of the Heart, 1995
Princess Mononoke, 1997
My Neighbors the Yamadas, 1999
Spirited Away, 2001
The Cat Returns, 2002
Howl's Moving Castle, 2004
Tales from Earthsea, 2006
Ponyo, 2008
Arrietty, 2010
From Up on Poppy Hill, 2011
The Wind Rises, 2013
The Tale of the Princess Kaguya, 2013
When Marnie Was There, 2014

```


We successfully received a list of Studio Ghibli movies with the year they were released. Now let’s complete this program by writing the movie list we are currently logging into a file.


Update the callbackMovies.js file in your text editor to include the following highlighted code, which creates a CSV file with our movie data:


callbackMovies.js
```
const request = require('request');
const fs = require('fs');

request('https://ghibliapi.herokuapp.com/films', (error, response, body) => {
    if (error) {
        console.error(`Could not send request to API: ${error.message}`);
        return;
    }

    if (response.statusCode != 200) {
        console.error(`Expected status code 200 but received ${response.statusCode}.`);
        return;
    }

    console.log('Processing our list of movies');
    movies = JSON.parse(body);
    let movieList = '';
    movies.forEach(movie => {
        movieList += `${movie['title']}, ${movie['release_date']}\n`;
    });

    fs.writeFile('callbackMovies.csv', movieList, (error) => {
        if (error) {
            console.error(`Could not save the Ghibli movies to a file: ${error}`);
            return;
        }

        console.log('Saved our list of movies to callbackMovies.csv');;
    });
});

```


Noting the highlighted changes, we see that we import the fs module. This module is standard in all Node.js installations, and it contains a writeFile() method that can asynchronously write to a file.


Instead of logging the data to the console, we now add it to a string variable movieList. We then use writeFile() to save the contents of movieList to a new file—callbackMovies.csv. Finally, we provide a callback to the writeFile() function, which has one argument: error. This allows us to handle cases where we are not able to write to a file, for example when the user we are running the node process on does not have those permissions.


Save the file and run this Node.js program once again with:


```
node callbackMovies.js


```


In your ghibliMovies folder, you will see callbackMovies.csv, which has the following content:


callbackMovies.csv
```
Castle in the Sky, 1986
Grave of the Fireflies, 1988
My Neighbor Totoro, 1988
Kiki's Delivery Service, 1989
Only Yesterday, 1991
Porco Rosso, 1992
Pom Poko, 1994
Whisper of the Heart, 1995
Princess Mononoke, 1997
My Neighbors the Yamadas, 1999
Spirited Away, 2001
The Cat Returns, 2002
Howl's Moving Castle, 2004
Tales from Earthsea, 2006
Ponyo, 2008
Arrietty, 2010
From Up on Poppy Hill, 2011
The Wind Rises, 2013
The Tale of the Princess Kaguya, 2013
When Marnie Was There, 2014

```


It’s important to note that we write to our CSV file in the callback of the HTTP request. Once the code is in the callback function, it will only write to the file after the HTTP request was completed. If we wanted to communicate to a database after we wrote our CSV file, we would make another asynchronous function that would be called in the callback of writeFile(). The more asynchronous code we have, the more callback functions have to be nested.


Let’s imagine that we want to execute five asynchronous operations, each one only able to run when another is complete. If we were to code this, we would have something like this:


```
doSomething1(() => {
    doSomething2(() => {
        doSomething3(() => {
            doSomething4(() => {
                doSomething5(() => {
                    // final action
                });
            });
        }); 
    });
});

```


When nested callbacks have many lines of code to execute, they become substantially more complex and unreadable. As your JavaScript project grows in size and complexity, this effect will become more pronounced, until it is eventually unmanageable. Because of this, developers no longer use callbacks to handle asynchronous operations. To improve the syntax of our asynchronous code, we can use promises instead.


# Using Promises for Concise Asynchronous Programming


A promise is a JavaScript object that will return a value at some point in the future. Asynchronous functions can return promise objects instead of concrete values. If we get a value in the future, we say that the promise was fulfilled. If we get an error in the future, we say that the promise was rejected. Otherwise, the promise is still being worked on in a pending state.


Promises generally take the following form:


```
promiseFunction()
    .then([ Callback Function for Fulfilled Promise ])
    .catch([ Callback Function for Rejected Promise ])

```


As shown in this template, promises also use callback functions. We have a callback function for the then() method, which is executed when a promise is fulfilled. We also have a callback function for the catch() method to handle any errors that come up while the promise is being executed.


Let’s get firsthand experience with promises by rewriting our Studio Ghibli program to use promises instead.


Axios is a promise-based HTTP client for JavaScript, so let’s go ahead and install it:


```
npm i axios --save


```


Now, with your text editor of choice, create a new file promiseMovies.js:


```
nano promiseMovies.js


```


Our program will make an HTTP request with axios and then use a special promised-based version of fs to save to a new CSV file.


Type this code in promiseMovies.js so we can load Axios and send an HTTP request to the movie API:


promiseMovies.js
```
const axios = require('axios');

axios.get('https://ghibliapi.herokuapp.com/films');

```


In the first line we load the axios module, storing the returned function in a constant called axios. We then use the axios.get() method to send an HTTP request to the API.


The axios.get() method returns a promise. Let’s chain that promise so we can print the list of Ghibli movies to the console:


promiseMovies.js
```
const axios = require('axios');
const fs = require('fs').promises;


axios.get('https://ghibliapi.herokuapp.com/films')
    .then((response) => {
        console.log('Successfully retrieved our list of movies');
        response.data.forEach(movie => {
            console.log(`${movie['title']}, ${movie['release_date']}`);
        });
    })

```


Let’s break down what’s happening. After making an HTTP GET request with axios.get(), we use the then() function, which is only executed when the promise is fulfilled. In this case, we print the movies to the screen like we did in the callbacks example.


To improve this program, add the highlighted code to write the HTTP data to a file:


promiseMovies.js
```
const axios = require('axios');
const fs = require('fs').promises;


axios.get('https://ghibliapi.herokuapp.com/films')
    .then((response) => {
        console.log('Successfully retrieved our list of movies');
        let movieList = '';
        response.data.forEach(movie => {
            movieList += `${movie['title']}, ${movie['release_date']}\n`;
        });

        return fs.writeFile('promiseMovies.csv', movieList);
    })
    .then(() => {
        console.log('Saved our list of movies to promiseMovies.csv');
    })

```


We additionally import the fs module once again. Note how after the fs import we have .promises. Node.js includes a promised-based version of the callback-based fs library, so backward compatibility is not broken in legacy projects.


The first then() function that processes the HTTP request now calls fs.writeFile() instead of printing to the console. Since we imported the promise-based version of fs, our writeFile() function returns another promise. As such, we append another then() function for when the writeFile() promise is fulfilled.


A promise can return a new promise, allowing us to execute promises one after the other. This paves the way for us to perform multiple asynchronous operations. This is called promise chaining, and it is analogous to nesting callbacks. The second then() is only called after we successfully write to the file.



Note: In this example, we did not check for the HTTP status code like we did in the callback example. By default, axios does not fulfil its promise if it gets a status code indicating an error. As such, we no longer need to validate it.

To complete this program, chain the promise with a catch() function as it is highlighted in the following:


promiseMovies.js
```
const axios = require('axios');
const fs = require('fs').promises;


axios.get('https://ghibliapi.herokuapp.com/films')
    .then((response) => {
        console.log('Successfully retrieved our list of movies');
        let movieList = '';
        response.data.forEach(movie => {
            movieList += `${movie['title']}, ${movie['release_date']}\n`;
        });

        return fs.writeFile('promiseMovies.csv', movieList);
    })
    .then(() => {
        console.log('Saved our list of movies to promiseMovies.csv');
    })
    .catch((error) => {
        console.error(`Could not save the Ghibli movies to a file: ${error}`);
    });

```


If any promise is not fulfilled in the chain of promises, JavaScript automatically goes to the catch() function if it was defined. That’s why we only have one catch() clause even though we have two asynchronous operations.


Let’s confirm that our program produces the same output by running:


```
node promiseMovies.js


```


In your ghibliMovies folder, you will see the promiseMovies.csv file containing:


promiseMovies.csv
```
Castle in the Sky, 1986
Grave of the Fireflies, 1988
My Neighbor Totoro, 1988
Kiki's Delivery Service, 1989
Only Yesterday, 1991
Porco Rosso, 1992
Pom Poko, 1994
Whisper of the Heart, 1995
Princess Mononoke, 1997
My Neighbors the Yamadas, 1999
Spirited Away, 2001
The Cat Returns, 2002
Howl's Moving Castle, 2004
Tales from Earthsea, 2006
Ponyo, 2008
Arrietty, 2010
From Up on Poppy Hill, 2011
The Wind Rises, 2013
The Tale of the Princess Kaguya, 2013
When Marnie Was There, 2014

```


With promises, we can write much more concise code than using only callbacks. The promise chain of callbacks is a cleaner option than nesting callbacks. However, as we make more asynchronous calls, our promise chain becomes longer and harder to maintain.


The verbosity of callbacks and promises come from the need to create functions when we have the result of an asynchronous task. A better experience would be to wait for an asynchronous result and put it in a variable outside the function. That way, we can use the results in the variables without having to make a function. We can achieve this with the async and await keywords.


# Writing JavaScript with async/await


The async/await keywords provide an alternative syntax when working with promises. Instead of having the result of a promise available in the then() method, the result is returned as a value like in any other function. We define a function with the async keyword to tell JavaScript that it’s an asynchronous function that returns a promise. We use the await keyword to tell JavaScript to return the results of the promise instead of returning the promise itself when it’s fulfilled.


In general, async/await usage looks like this:


```
async function() {
    await [Asynchronous Action]
}

```


Let’s see how using async/await can improve our Studio Ghibli program. Use your text editor to create and open a new file asyncAwaitMovies.js:


```
nano asyncAwaitMovies.js


```


In your newly opened JavaScript file, let’s start by importing the same modules we used in our promise example:


asyncAwaitMovies.js
```
const axios = require('axios');
const fs = require('fs').promises;

```


The imports are the same as promiseMovies.js because async/await uses promises.


Now we use the async keyword to create a function with our asynchronous code:


asyncAwaitMovies.js
```
const axios = require('axios');
const fs = require('fs').promises;

async function saveMovies() {}

```


We create a new function called saveMovies() but we include async at the beginning of its definition. This is important as we can only use the await keyword in an asynchronous function.


Use the await keyword to make an HTTP request that gets the list of movies from the Ghibli API:


asyncAwaitMovies.js
```
const axios = require('axios');
const fs = require('fs').promises;

async function saveMovies() {
    let response = await axios.get('https://ghibliapi.herokuapp.com/films');
    let movieList = '';
    response.data.forEach(movie => {
        movieList += `${movie['title']}, ${movie['release_date']}\n`;
    });
}

```


In our saveMovies() function, we make an HTTP request with axios.get() like before. This time, we don’t chain it with a then() function. Instead, we add await before it is called. When JavaScript sees await, it will only execute the remaining code of the function after axios.get() finishes execution and sets the response variable. The other code saves the movie data so we can write to a file.


Let’s write the movie data to a file:


asyncAwaitMovies.js
```
const axios = require('axios');
const fs = require('fs').promises;

async function saveMovies() {
    let response = await axios.get('https://ghibliapi.herokuapp.com/films');
    let movieList = '';
    response.data.forEach(movie => {
        movieList += `${movie['title']}, ${movie['release_date']}\n`;
    });
    await fs.writeFile('asyncAwaitMovies.csv', movieList);
}

```


We also use the await keyword when we write to the file with fs.writeFile().


To complete this function, we need to catch errors our promises can throw. Let’s do this by encapsulating our code in a try/catch block:


asyncAwaitMovies.js
```
const axios = require('axios');
const fs = require('fs').promises;

async function saveMovies() {
    try {
        let response = await axios.get('https://ghibliapi.herokuapp.com/films');
        let movieList = '';
        response.data.forEach(movie => {
            movieList += `${movie['title']}, ${movie['release_date']}\n`;
        });
        await fs.writeFile('asyncAwaitMovies.csv', movieList);
    } catch (error) {
        console.error(`Could not save the Ghibli movies to a file: ${error}`);
    }
}


```


Since promises can fail, we encase our asynchronous code with a try/catch clause. This will capture any errors that are thrown when either the HTTP request or file writing operations fail.


Finally, let’s call our asynchronous function saveMovies() so it will be executed when we run the program with node


asyncAwaitMovies.js
```
const axios = require('axios');
const fs = require('fs').promises;

async function saveMovies() {
    try {
        let response = await axios.get('https://ghibliapi.herokuapp.com/films');
        let movieList = '';
        response.data.forEach(movie => {
            movieList += `${movie['title']}, ${movie['release_date']}\n`;
        });
        await fs.writeFile('asyncAwaitMovies.csv', movieList);
    } catch (error) {
        console.error(`Could not save the Ghibli movies to a file: ${error}`);
    }
}

saveMovies();

```


At a glance, this looks like a typical synchronous JavaScript code block. It has fewer functions being passed around, which looks a bit neater. These small tweaks make asynchronous code with async/await easier to maintain.


Test this iteration of our program by entering this in your terminal:


```
node asyncAwaitMovies.js


```


In your ghibliMovies folder, a new asyncAwaitMovies.csv file will be created with the following contents:


asyncAwaitMovies.csv
```
Castle in the Sky, 1986
Grave of the Fireflies, 1988
My Neighbor Totoro, 1988
Kiki's Delivery Service, 1989
Only Yesterday, 1991
Porco Rosso, 1992
Pom Poko, 1994
Whisper of the Heart, 1995
Princess Mononoke, 1997
My Neighbors the Yamadas, 1999
Spirited Away, 2001
The Cat Returns, 2002
Howl's Moving Castle, 2004
Tales from Earthsea, 2006
Ponyo, 2008
Arrietty, 2010
From Up on Poppy Hill, 2011
The Wind Rises, 2013
The Tale of the Princess Kaguya, 2013
When Marnie Was There, 2014

```


You have now used the JavaScript features async/await to manage asynchronous code.


# Conclusion


In this tutorial, you learned how JavaScript handles executing functions and managing asynchronous operations with the event loop. You then wrote programs that created a CSV file after making an HTTP request for movie data using various asynchronous programming techniques. First, you used the obsolete callback-based approach. You then used promises, and finally async/await to make the promise syntax more succinct.


With your understanding of asynchronous code with Node.js, you can now develop programs that benefit from asynchronous programming, like those that rely on API calls. Have a look at this list of public APIs. To use them, you will have to make asynchronous HTTP requests like we did in this tutorial. For further study, try building an app that uses these APIs to practice the techniques you learned here.


