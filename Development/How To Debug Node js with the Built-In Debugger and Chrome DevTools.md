# How To Debug Node js with the Built-In Debugger and Chrome DevTools

```JavaScript``` ```Node.js``` ```Development```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


In Node.js development, tracing a coding error back to its source can save a lot of time over the course of a project. But as a program grows in complexity, it becomes harder and harder to do this efficiently. To solve this problem, developers use tools like a debugger, a program that allows developers to inspect their program as it runs. By replaying the code line-by-line and observing how it changes the program’s state, debuggers can provide insight into how a program is running, making it easier to find bugs.


A common practice programmers use to track bugs in their code is to print statements as the program runs. In Node.js, that involves adding extra console.log() or console.debug() statements in their modules. While this technique can be used quickly, it is also manual, making it less scalable and more prone to errors. Using this method, it is possible to mistakenly log sensitive information to the console, which could provide malicious agents with private information about customers or your application. On the other hand, debuggers provide a systematic way to observe what’s happening in a program, without exposing your program to security threats.


The key features of debuggers are watching objects and adding breakpoints. By watching objects, a debugger can help track the changes of a variable as the programmer steps through a program. Breakpoints are markers that a programmer can place in their code to stop the code from continuing beyond points that the developer is investigating.


In this article, you will use a debugger to debug some sample Node.js applications. You will first debug code using the built-in Node.js debugger tool, setting up watchers and breakpoints so you can find the root cause of a bug. You will then use Google Chrome DevTools as a Graphical User Interface (GUI) alternative to the command line Node.js debugger.


# Prerequisites


- You will need Node.js installed in your development environment. This tutorial uses version 10.19.0. To install this on macOS or Ubuntu 18.04, follow the steps in How To Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04.
- For this article we expect the user to be comfortable with basic JavaScript, especially creating and using functions. You can learn those fundamentals and more by reading our How To Code in JavaScript series.
- To use the Chrome DevTools debugger, you will need to download and install the Google Chrome web browser or the open-source Chromium web browser.

# Step 1 — Using Watchers with the Node.js Debugger


Debuggers are primarily useful for two features: their ability to watch variables and observe how they change when a program is run and their ability to stop and start code execution at different locations called breakpoints. In this step, we will run through how to watch variables to identify errors in code.


Watching variables as we step through code gives us insight into how the values of variables change as the program runs. Let’s practice watching variables to help us find and fix logical errors in our code with an example.


We begin by setting up our coding environment. In your terminal, create a new folder called debugging:


```
mkdir debugging


```


Now enter that folder:


```
cd debugging


```


Open a new file called badLoop.js. We will use nano as it’s available in the terminal:


```
nano badLoop.js


```


Our code will iterate over an array and add numbers into a total sum, which in our example will be used to add up the number of daily orders over the course of a week at a store. The program will return the sum of all the numbers in the array. In the editor, enter the following code:


debugging/badLoop.js
```
let orders = [341, 454, 198, 264, 307];

let totalOrders = 0;

for (let i = 0; i <= orders.length; i++) {
  totalOrders += orders[i];
}

console.log(totalOrders);

```


We start by creating the orders array, which stores five numbers. We then initialize totalOrders to 0, as it will store the total of the five numbers. In the for loop, we iteratively add each value in orders to totalOrders. Finally, we print the total amount of orders at the end of the program.


Save and exit from the editor. Now run this program with node:


```
node badLoop.js


```


The terminal will show this output:


```
OutputNaN

```


NaN in JavaScript means Not a Number. Given that all the input are valid numbers, this is unexpected behavior. To find the error, let’s use the Node.js debugger to see what happens to the two variables that are changed in the for loop: totalOrders and i.


When we want to use the built-in Node.js debugger on a program, we include inspect before the file name. In your terminal, run the node command with this debugger option as follows:


```
node inspect badLoop.js


```


When you start the debugger, you will find output like this:


```
Output< Debugger listening on ws://127.0.0.1:9229/e1ebba25-04b8-410b-811e-8a0c0902717a
< For help, see: https://nodejs.org/en/docs/inspector
< Debugger attached.
Break on start in badLoop.js:1
> 1 let orders = [341, 454, 198, 264, 307];
  2 
  3 let totalOrders = 0;

```


The first line shows us the URL of our debug server. That’s used when we want to debug with external clients, like a web browser as we’ll see later on. Note that this server listens on port :9229 of the localhost (127.0.0.1) by default. For security reasons, it is recommended to avoid exposing this port to the public.


After the debugger is attached, the debugger outputs Break on start in badLoop.js:1.


Breakpoints are places in our code where we’d like execution to stop. By default, Node.js’s debugger stops execution at the beginning of the file.


The debugger then shows us a snippet of code, followed by a special debug prompt:


```
Output...
> 1 let orders = [341, 454, 198, 264, 307];
  2 
  3 let totalOrders = 0;
debug>

```


The > next to 1 indicates which line we’ve reached in our execution, and the prompt is where we will type in our commends to the debugger. When this output appears, the debugger is ready to accept commands.


When using a debugger, we step through code by telling the debugger to go to the next line that the program will execute. Node.js allows the following commands to use a debugger:


- c or cont: Continue execution to the next breakpoint or to the end of the program.
- n or next: Move to the next line of code.
- s or step: Step into a function. By default, we only step through code in the block or scope we’re debugging. By stepping into a function, we can inspect the code of the function our code calls and observe how it reacts to our data.
- o: Step out of a function. After stepping into a function, the debugger goes back to the main file when the function returns. We can use this command to go back to the original function we were debugging before the function has finished execution.
- pause: Pause the running code.

We’ll be stepping through this code line-by-line. Press n to go to the next line:


```
n


```


Our debugger will now be stuck on the third line of code:


```
Outputbreak in badLoop.js:3
  1 let orders = [341, 454, 198, 264, 307];
  2 
> 3 let totalOrders = 0;
  4 
  5 for (let i = 0; i <= orders.length; i++) {

```


Empty lines are skipped for convenience. If we press n once more in the debug console, our debugger will be situated on the fifth line of code:


```
Outputbreak in badLoop.js:5
  3 let totalOrders = 0;
  4 
> 5 for (let i = 0; i <= orders.length; i++) {
  6   totalOrders += orders[i];
  7 }

```


We are now beginning our loop. If the terminal supports color, the 0 in let i = 0 will be highlighted. The debugger highlights the part of the code the program is about to execute, and in a for loop, the counter initialization is executed first. From here, we can watch to see why totalOrders is returning NaN instead of a number. In this loop, two variables are changed every iteration—totalOrders and i. Let’s set up watchers for both of those variables.


We’ll first add a watcher for the totalOrders variable. In the interactive shell, enter this:


```
watch('totalOrders')


```


To watch a variable, we use the built-in watch() function with a string argument that contains the variable name. As we press ENTER on the watch() function, the prompt will move to the next line without providing feedback, but the watch word will be visible when we move the debugger to the next line.


Now let’s add a watcher for the variable i:


```
watch('i')


```


Now we can see our watchers in action. Press n to go to the next step. The debug console will show this:


```
Outputbreak in badLoop.js:5
Watchers:
  0: totalOrders = 0
  1: i = 0

  3 let totalOrders = 0;
  4 
> 5 for (let i = 0; i <= orders.length; i++) {
  6   totalOrders += orders[i];
  7 }

```


The debugger now displays the values of totalOrders and i before showing the line of code, as shown in the output. These values are updated every time a line of code changes them.


At this point, the debugger is highlighting length in orders.length. This means the program is about to check the condition before it executes the code within its block. After the code is executed, the final expression i++ will be executed. You can read more about for loops and their execution in our How To Construct For Loops in JavaScript guide.


Enter n in the console to enter the for loop’s body:


```
Outputbreak in badLoop.js:6
Watchers:
  0: totalOrders = 0
  1: i = 0

  4 
  5 for (let i = 0; i <= orders.length; i++) {
> 6   totalOrders += orders[i];
  7 }
  8 

```


This step updates the totalOrders variable. Therefore, after this step is complete our variable and watcher will be updated.


Press n to confirm. You will see this:


```
OutputWatchers:
  0: totalOrders = 341
  1: i = 0

  3 let totalOrders = 0;
  4 
> 5 for (let i = 0; i <= orders.length; i++) {
  6   totalOrders += orders[i];
  7 }

```


As highlighted, totalOrders now has the value of the first order: 341.


Our debugger is just about to process the final condition of the loop. Enter n so we execute this line and update i:


```
Outputbreak in badLoop.js:5
Watchers:
  0: totalOrders = 341
  1: i = 1

  3 let totalOrders = 0;
  4 
> 5 for (let i = 0; i <= orders.length; i++) {
  6   totalOrders += orders[i];
  7 }

```


After initialization, we had to step through the code four times to see the variables updated. Stepping through the code like this can be tedious; this problem will be addressed with breakpoints in Step 2. But for now, by setting up our watchers, we are ready to observe their values and find our problem.


Step through the program by entering n twelve more times, observing the output. Your console will display this:


```
Outputbreak in badLoop.js:5
Watchers:
  0: totalOrders = 1564
  1: i = 5

  3 let totalOrders = 0;
  4 
> 5 for (let i = 0; i <= orders.length; i++) {
  6   totalOrders += orders[i];
  7 }

```


Recall that our orders array has five items, and i is now at position 5. But since i is used as the index of an array, there is no value at orders[5]; the last value of the orders array is at index 4. This means that orders[5] will have a value of undefined.


Type n in the console and you’ll observe that the code in the loop is executed:


```
Outputbreak in badLoop.js:6
Watchers:
  0: totalOrders = 1564
  1: i = 5

  4 
  5 for (let i = 0; i <= orders.length; i++) {
> 6   totalOrders += orders[i];
  7 }
  8 

```


Typing n once more shows the value of totalOrders after that iteration:


```
Outputbreak in badLoop.js:5
Watchers:
  0: totalOrders = NaN
  1: i = 5

  3 let totalOrders = 0;
  4 
> 5 for (let i = 0; i <= orders.length; i++) {
  6   totalOrders += orders[i];
  7 }

```


Through debugging and watching totalOrders and i, we can see that our loop is iterating six times instead of five. When i is 5, orders[5] is added to totalOrders. Since orders[5] is undefined, adding this to a number will yield NaN. The problem with our code therefore lies within our for loop’s condition. Instead of checking if i is less than or equal to the length of the orders array, we should only check that it’s less than the length.


Let’s exit our debugger, make the changes and run the code again. In the debug prompt, type the exit command and press ENTER:


```
.exit


```


Now that you’ve exited the debugger, open badLoop.js in your text editor:


```
nano badLoop.js


```


Change the for loop’s condition:


debugger/badLoop.js
```
...
for (let i = 0; i < orders.length; i++) {
...

```


Save and exit nano. Now let’s execute our script like this:


```
node badLoop.js


```


When it’s complete, the correct result will be printed:


```
Output1564

```


In this section, we used the debugger’s watch command to find a bug in our code, fixed it, and watched it work as expected.


Now that we have some experience with the basic use of the debugger to watch variables, let’s look at how we can use breakpoints so that we can debug without stepping through all the lines of code from the start of the program.


# Step 2 — Using Breakpoints With the Node.js Debugger


It’s common for Node.js projects to consist of many interconnected modules. Debugging each module line-by-line would be time consuming, especially as an app scales in complexity. To solve this problem, breakpoints allow us to jump to a line of code where we’d like to pause execution and inspect the program.


When debugging in Node.js, we add a breakpoint by adding the debugger keyword directly to our code. We can then go from one breakpoint to the next by pressing c in the debugger console instead of n. At each breakpoint, we can set up watchers for expressions of interest.


Let’s see this with an example. In this step, we’ll set up a program that reads a list of sentences and determines the most common word used throughout all the text. Our sample code will return the first word with the highest number of occurrences.


For this exercise, we will create three files. The first file, sentences.txt, will contain the raw data that our program will process. We’ll add the beginning text from Encyclopaedia Britannica’s article on the Whale Shark as sample data, with the punctuation removed.


Open the file in your text editor:


```
nano sentences.txt


```


Next, enter the following code:


debugger/sentences.txt
```
Whale shark Rhincodon typus gigantic but harmless shark family Rhincodontidae that is the largest living fish
Whale sharks are found in marine environments worldwide but mainly in tropical oceans
They make up the only species of the genus Rhincodon and are classified within the order Orectolobiformes a group containing the carpet sharks
The whale shark is enormous and reportedly capable of reaching a maximum length of about 18 metres 59 feet
Most specimens that have been studied however weighed about 15 tons about 14 metric tons and averaged about 12 metres 39 feet in length
The body coloration is distinctive
Light vertical and horizontal stripes form a checkerboard pattern on a dark background and light spots mark the fins and dark areas of the body

```


Save and exit the file.


Now let’s add our code to textHelper.js. This module will contain some handy functions we’ll use to process the text file, making it easier to determine the most popular word. Open textHelper.js in your text editor:


```
nano textHelper.js


```


We’ll create three functions to process the data in sentences.txt. The first will be to read the file. Type the following into textHelper.js:


debugger/textHelper.js
```
const fs = require('fs');

const readFile = () => {
  let data = fs.readFileSync('sentences.txt');
  let sentences = data.toString();
  return sentences;
};

```


First, we import the fs Node.js library so we can read files. We then create the readFile() function that uses readFileSync() to load the data from sentences.txt as a Buffer object and the toString() method to return it as a string.


The next function we’ll add processes a string of text and flattens it to an array with its words. Add the following code into the editor:


textHelper.js
```
...

const getWords = (text) => {
  let allSentences = text.split('\n');
  let flatSentence = allSentences.join(' ');
  let words = flatSentence.split(' ');
  words = words.map((word) => word.trim().toLowerCase());
  return words;
};

```


In this code, we are using the methods split(), join(), and map() to manipulate the string into an array of individual words. The function also lowercases each word to make counting easier.


The last function needed returns the counts of different words in a string array. Add the last function like this:


debugger/textHelper.js
```
...

const countWords = (words) => {
  let map = {};
  words.forEach((word) => {
    if (word in map) {
      map[word] = 1;
    } else {
      map[word] += 1;
    }
  });

  return map;
};

```


Here we create a JavaScript object called map that has the words as its keys and their counts as the values. We loop through the array, adding one to a count of each word when it’s the current element of the loop. Let’s complete this module by exporting these functions, making them available to other modules:


debugger/textHelper.js
```
...

module.exports = { readFile, getWords, countWords };

```


Save and exit.


Our third and final file we’ll use for this exercise will use the textHelper.js module to find the most popular word in our text. Open index.js with your text editor:


```
nano index.js


```


We begin our code by importing the textHelpers.js module:


debugger/index.js
```
const textHelper = require('./textHelper');

```


Continue by creating a new array containing stop words:


debugger/index.js
```
...

const stopwords = ['i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 'your', 'yours', 'yourself', 'yourselves', 'he', 'him', 'his', 'himself', 'she', 'her', 'hers', 'herself', 'it', 'its', 'itself', 'they', 'them', 'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 'this', 'that', 'these', 'those', 'am', 'is', 'are', 'was', 'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 'about', 'against', 'between', 'into', 'through', 'during', 'before', 'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 'only', 'own', 'same', 'so', 'than', 'too', 'very', 's', 't', 'can', 'will', 'just', 'don', 'should', 'now', ''];

```


Stop words are commonly used words in a language that we filter out before processing a text. We can use this to find more meaningful data than the result that the most popular word in English text is the or a.


Continue by using the textHelper.js module functions to get a JavaScript object with words and their counts:


debugger/index.js
```
...

let sentences = textHelper.readFile();
let words = textHelper.getWords(sentences);
let wordCounts = textHelper.countWords(words);

```


We can then complete this module by determining the words with the highest frequency. To do this, we’ll loop through each key of the object with the word counts and compare its count to the previously stored maximum. If the word’s count is higher, it becomes the new maximum.


Add the following lines of code to compute the most popular word:


debugger/index.js
```
...

let max = -Infinity;
let mostPopular = '';

Object.entries(wordCounts).forEach(([word, count]) => {
  if (stopwords.indexOf(word) === -1) {
    if (count > max) {
      max = count;
      mostPopular = word;
    }
  }
});

console.log(`The most popular word in the text is "${mostPopular}" with ${max} occurrences`);

```


In this code, we are using Object.entries() to transform the key-value pairs in the wordCounts object into individual arrays, all of which are nested within a larger array. We then use the forEach() method and some conditional statements to test the count of each word and store the highest number.


Save and exit the file.


Let’s now run this file to see it in action. In your terminal enter this command:


```
node index.js


```


You will see the following output:


```
OutputThe most popular word in the text is "whale" with 1 occurrences

```


From reading the text, we can see that the answer is incorrect. A quick search in sentences.txt would highlight that the word whale appears more than once.


We have quite a few functions that can cause this error: We may not be reading the entire file, or we may not be processing the text into the array and JavaScript object correctly. Our algorithm for finding the maximum word could also be incorrect. The best way to figure out what’s wrong is to use the debugger.


Even without a large codebase, we don’t want to spend time stepping through each line of code to observe when things change. Instead, we can use breakpoints to go to those key moments before the function returns and observe the output.


Let’s add breakpoints in each function in the textHelper.js module. To do so, we need to add the keyword debugger into our code.


Open the textHelper.js file in the text editor. We’ll be using nano once again:


```
nano textHelper.js


```


First, we’ll add the breakpoint to the readFile() function like this:


debugger/textHelper.js
```
...

const readFile = () => {
  let data = fs.readFileSync('sentences.txt');
  let sentences = data.toString();
  debugger;
  return sentences;
};

...

```


Next, we’ll add another breakpoint to the getWords() function:


debugger/textHelper.js
```
...

const getWords = (text) => {
  let allSentences = text.split('\n');
  let flatSentence = allSentences.join(' ');
  let words = flatSentence.split(' ');
  words = words.map((word) => word.trim().toLowerCase());
  debugger;
  return words;
};

...

```


Finally, we’ll add a breakpoint to the countWords() function:


debugger/textHelper.js
```
...

const countWords = (words) => {
  let map = {};
  words.forEach((word) => {
    if (word in map) {
      map[word] = 1;
    } else {
      map[word] += 1;
    }
  });

  debugger;
  return map;
};

...

```


Save and exit textHelper.js.


Let’s begin the debugging process. Although the breakpoints are in textHelpers.js, we are debugging the main point of entry of our application: index.js. Start a debugging session by entering the following command in your shell:


```
node inspect index.js


```


After entering the command, we’ll be greeted with this output:


```
Output< Debugger listening on ws://127.0.0.1:9229/b2d3ce0e-3a64-4836-bdbf-84b6083d6d30
< For help, see: https://nodejs.org/en/docs/inspector
< Debugger attached.
Break on start in index.js:1
> 1 const textHelper = require('./textHelper');
  2 
  3 const stopwords = ['i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 'your', 'yours', 'yourself', 'yourselves', 'he', 'him', 'his', 'himself', 'she', 'her', 'hers', 'herself', 'it', 'its', 'itself', 'they', 'them', 'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 'this', 'that', 'these', 'those', 'am', 'is', 'are', 'was', 'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 'about', 'against', 'between', 'into', 'through', 'during', 'before', 'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 'only', 'own', 'same', 'so', 'than', 'too', 'very', 's', 't', 'can', 'will', 'just', 'don', 'should', 'now', ''];

```


This time, enter c into the interactive debugger. As a reminder, c is short for continue. This jumps the debugger to the next breakpoint in the code. After pressing c and typing ENTER, you will see this in your console:


```
Outputbreak in textHelper.js:6
  4   let data = fs.readFileSync('sentences.txt');
  5   let sentences = data.toString();
> 6   debugger;
  7   return sentences;
  8 };

```


We’ve now saved some debugging time by going directly to our breakpoint.


In this function, we want to be sure that all the text in the file is being returned. Add a watcher for the sentences variable so we can see what’s being returned:


```
watch('sentences')


```


Press n to move to the next line of code so we can observe what’s in sentences. You will see the following output:


```
Outputbreak in textHelper.js:7
Watchers:
  0: sentences =
    'Whale shark Rhincodon typus gigantic but harmless shark family Rhincodontidae that is the largest living fish\n' +
      'Whale sharks are found in marine environments worldwide but mainly in tropical oceans\n' +
      'They make up the only species of the genus Rhincodon and are classified within the order Orectolobiformes a group containing the carpet sharks\n' +
      'The whale shark is enormous and reportedly capable of reaching a maximum length of about 18 metres 59 feet\n' +
      'Most specimens that have been studied however weighed about 15 tons about 14 metric tons and averaged about 12 metres 39 feet in length\n' +
      'The body coloration is distinctive\n' +
      'Light vertical and horizontal stripes form a checkerboard pattern on a dark background and light spots mark the fins and dark areas of the body\n'

  5   let sentences = data.toString();
  6   debugger;
> 7   return sentences;
  8 };
  9

```


It seems that we aren’t having any problems reading the file; the problem must lie elsewhere in our code. Let’s move to the next breakpoint by pressing c once again. When you do, you’ll see this output:


```
Outputbreak in textHelper.js:15
Watchers:
  0: sentences =
    ReferenceError: sentences is not defined
        at eval (eval at getWords (your_file_path/debugger/textHelper.js:15:3), <anonymous>:1:1)
        at Object.getWords (your_file_path/debugger/textHelper.js:15:3)
        at Object.<anonymous> (your_file_path/debugger/index.js:7:24)
        at Module._compile (internal/modules/cjs/loader.js:1125:14)
        at Object.Module._extensions..js (internal/modules/cjs/loader.js:1167:10)
        at Module.load (internal/modules/cjs/loader.js:983:32)
        at Function.Module._load (internal/modules/cjs/loader.js:891:14)
        at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:71:12)
        at internal/main/run_main_module.js:17:47

 13   let words = flatSentence.split(' ');
 14   words = words.map((word) => word.trim().toLowerCase());
>15   debugger;
 16   return words;
 17 };

```


We get this error message because we set up a watcher for the sentences variable, but that variable does not exist in our current function scope. A watcher lasts for the entire debugging session, so as long as we keep watching sentences where it’s not defined, we’ll continue to see this error.


We can stop watching variables with the unwatch() command. Let’s unwatch sentences so we no longer have to see this error message every time the debugger prints its output. In the interactive prompt, enter this command:


```
unwatch('sentences')


```


The debugger does not output anything when you unwatch a variable.


Back in the getWords() function, we want to be sure that we are returning a list of words that are taken from the text we loaded earlier. Let’s watch the value of the words variable:


```
watch('words')


```


Then enter n to go to the next line of the debugger, so we can see what’s being stored in words. The debugger will show the following:


```
Outputbreak in textHelper.js:16
Watchers:
  0: words =
    [ 'whale',
      'shark',
      'rhincodon',
      'typus',
      'gigantic',
      'but',
      'harmless',
      ...
      'metres',
      '39',
      'feet',
      'in',
      'length',
      '',
      'the',
      'body',
      'coloration',
      ... ]

 14   words = words.map((word) => word.trim().toLowerCase());
 15   debugger;
>16   return words;
 17 };
 18

```


The debugger does not print out the entire array as it’s quite long and would make the output harder to read. However, the output meets our expectations of what should be stored: the text from sentences split into lowercase strings. It seems that getWords() is functioning correctly.


Let’s move on to observe the countWords() function. First, unwatch the words array so we don’t cause any debugger errors when we are at the next breakpoint. In the command prompt, enter this:


```
unwatch('words')


```


Next, enter c in the prompt. At our last breakpoint, we will see this in the shell:


```
Outputbreak in textHelper.js:29
 27   });
 28 
>29   debugger;
 30   return map;
 31 };

```


In this function, we want to be sure that the map variable correctly contains the count of each word from our sentences. First, let’s tell the debugger to watch the map variable:


```
watch('map')


```


Press n to move to the next line. The debugger will then display this:


```
Outputbreak in textHelper.js:30
Watchers:
  0: map =
    { 12: NaN,
      14: NaN,
      15: NaN,
      18: NaN,
      39: NaN,
      59: NaN,
      whale: 1,
      shark: 1,
      rhincodon: 1,
      typus: NaN,
      gigantic: NaN,
      ... }

 28
 29   debugger;
>30   return map;
 31 };
 32

```


That does not look correct. It seems as though the method for counting words is producing erroneous results. We don’t know why those values are being entered, so our next step is to debug what’s happening in the loop used on the words array. To do this, we need to make some changes to where we place our breakpoint.


First, exit the debug console:


```
.exit


```


Open textHelper.js in your text editor so we can edit the breakpoints:


```
nano textHelper.js


```


First, knowing that readFile() and getWords() are working, we will remove their breakpoints. We then want to remove the breakpoint in countWords() from the end of the function, and add two new breakpoints to the beginning and end of the forEach() block.


Edit textHelper.js so it looks like this:


debugger/textHelper.js
```
...

const readFile = () => {
  let data = fs.readFileSync('sentences.txt');
  let sentences = data.toString();
  return sentences;
};

const getWords = (text) => {
  let allSentences = text.split('\n');
  let flatSentence = allSentences.join(' ');
  let words = flatSentence.split(' ');
  words = words.map((word) => word.trim().toLowerCase());
  return words;
};

const countWords = (words) => {
  let map = {};
  words.forEach((word) => {
    debugger;
    if (word in map) {
      map[word] = 1;
    } else {
      map[word] += 1;
    }
    debugger;
  });

  return map;
};

...

```


Save and exit nano with CTRL+X.


Let’s start the debugger again with this command:


```
node inspect index.js


```


To get insight into what’s happening, we want to debug a few things in the loop. First, let’s set up a watcher for word, the argument used in the forEach() loop containing the string that the loop is currently looking at. In the debug prompt, enter this:


```
watch('word')


```


So far, we have only watched variables. But watches are not limited to variables. We can watch any valid JavaScript expression that’s used in our code.


In practical terms, we can add a watcher for the condition word in map, which determines how we count numbers. In the debug prompt, create this watcher:


```
watch('word in map')


```


Let’s also add a watcher for the value that’s being modified in the map variable:


```
watch('map[word]')


```


Watchers can even be expressions that aren’t used in our code but could be evaluated with the code we have. Let’s see how this works by adding a watcher for the length of the word variable:


```
watch('word.length')


```


Now that we’ve set up all our watchers, let’s enter c into the debugger prompt so we can see how the first element in the loop of countWords() is evaluated. The debugger will print this output:


```
Outputbreak in textHelper.js:20
Watchers:
  0: word = 'whale'
  1: word in map = false
  2: map[word] = undefined
  3: word.length = 5

 18   let map = {};
 19   words.forEach((word) => {
>20     debugger;
 21     if (word in map) {
 22       map[word] = 1;

```


The first word in the loop is whale. At this point, the map object has no key with whale as its empty. Following from that, when looking up whale in map, we get undefined. Lastly, the length of whale is 5. That does not help us debug the problem, but it does validate that we can watch any expression that could be evaluated with the code while debugging.


Press c once more to see what’s changed by the end of the loop. The debugger will show this:


```
Outputbreak in textHelper.js:26
Watchers:
  0: word = 'whale'
  1: word in map = true
  2: map[word] = NaN
  3: word.length = 5

 24       map[word] += 1;
 25     }
>26     debugger;
 27   });
 28

```


At the end of the loop, word in map is now true as the map variable contains a whale key. The value of map for the whale key is NaN, which highlights our problem. The if statement in countWords() is meant to set a word’s count to one if it’s new, and add one if it existed already.


The culprit is the if statement’s condition. We should set map[word] to 1 if the word is not found in map. Right now, we are adding one if word is found. At the beginning of the loop, map["whale"] is undefined. In JavaScript, undefined + 1 evaluates to NaN—not a number.


The fix for this would be to change the condition of the if statement from (word in map) to (!(word in map)), using the ! operator to test if word is not in map. Let’s make that change in the countWords() function to see what happens.


First, exit the debugger:


```
.exit


```


Now open the textHelper.js file with your text editor:


```
nano textHelper.js


```


Modify the countWords() function as follows:


debugger/textHelper.js
```
...

const countWords = (words) => {
  let map = {};
  words.forEach((word) => {
    if (!(word in map)) {
      map[word] = 1;
    } else {
      map[word] += 1;
    }
  });

  return map;
};

...

```


Save and close the editor.


Now let’s run this file without a debugger. In the terminal, enter this:


```
node index.js


```


The script will output the following sentence:


```
OutputThe most popular word in the text is "whale" with 3 occurrences

```


This output seems a lot more likely than what we received before. With the debugger, we figured out which function caused the problem and which functions did not.


We’ve debugged two different Node.js programs with the built-in CLI debugger. We are now able to set up breakpoints with the debugger keyword and create various watchers to observe changes in internal state. But sometimes, code can be more effectively debugged from a GUI application.


In the next section, we’ll use the debugger in Google Chrome’s DevTools. We’ll start the debugger in Node.js, navigate to a dedicated debugging page in Google Chrome, and set up breakpoints and watchers using the GUI.


# Step 3 — Debugging Node.js with Chrome DevTools


Chrome DevTools is a popular choice for debugging Node.js in a web browser. As Node.js uses the same V8 JavaScript engine that Chrome uses, the debugging experience is more integrated than with other debuggers.


For this exercise, we’ll create a new Node.js application that runs an HTTP server and returns a JSON response. We’ll then use the debugger to set up breakpoints and gain deeper insight into what response is being generated for the request.


Let’s create a new file called server.js that will store our server code. Open the file in the text editor:


```
nano server.js


```


This application will return a JSON with a Hello World greeting. It will have an array of messages in different languages. When a request is received, it will randomly pick a greeting and return it in a JSON body.


This application will run on our localhost server on port :8000. If you’d like to learn more about creating HTTP servers with Node.js, read our guide on How To Create a Web Server in Node.js with the HTTP Module.


Type the following code into the text editor:


debugger/server.js
```
const http = require("http");

const host = 'localhost';
const port = 8000;

const greetings = ["Hello world", "Hola mundo", "Bonjour le monde", "Hallo Welt", "Salve mundi"];

const getGreeting = function () {
  let greeting = greetings[Math.floor(Math.random() * greetings.length)];
  return greeting
}

```


We begin by importing the http module, which is needed to create an HTTP server. We then set up the host and port variables that we will use later to run the server. The greetings array contains all the possible greetings our server can return. The getGreeting() function randomly selects a greeting and returns it.


Let’s add the request listener that processes HTTP requests and add code to run our server. Continue editing the Node.js module by typing the following:


debugger/server.js
```
...

const requestListener = function (req, res) {
  let message = getGreeting();
  res.setHeader("Content-Type", "application/json");
  res.writeHead(200);
  res.end(`{"message": "${message}"}`);
};

const server = http.createServer(requestListener);
server.listen(port, host, () => {
  console.log(`Server is running on http://${host}:${port}`);
});

```


Our server is now ready for use, so let’s set up the Chrome debugger.


We can start the Chrome debugger with the following command:


```
node --inspect server.js


```



Note: Keep in mind the difference between the CLI debugger and the Chrome debugger commands. When using the CLI you use inspect. When using Chrome you use --inspect.

After starting the debugger, you’ll find the following output:


```
OutputDebugger listening on ws://127.0.0.1:9229/996cfbaf-78ca-4ebd-9fd5-893888efe8b3
For help, see: https://nodejs.org/en/docs/inspector
Server is running on http://localhost:8000

```


Now open Google Chrome or Chromium and enter chrome://inspect in the address bar. Microsoft Edge also uses the V8 JavaScript engine, and can thus use the same debugger. If you are using Microsoft Edge, navigate to edge://inspect.


After navigating to the URL, you will see the following page:





Under the Devices header, click the Open dedicated DevTools for Node button. A new window will pop up:





We’re now able to debug our Node.js code with Chrome. Navigate to the Sources tab if not already there. On the left-hand side, expand the file tree and select server.js:





Let’s add a breakpoint to our code. We want to stop when the server has selected a greeting and is about to return it. Click on the line number 10 in the debug console. A red dot will appear next to the number and the right-hand panel will indicate a new breakpoint was added:





Now let’s add a watch expression. On the right panel, click the arrow next to the Watch header to open the watch words list, then click +. Enter greeting and press ENTER so that we can observe its value when processing a request.


Next, let’s debug our code. Open a new browser window and navigate to http://localhost:8000—the address the Node.js server is running on. When pressing ENTER, we will not immediately get a response. Instead, the debug window will pop up once again. If it does not immediately come into focus, navigate to the debug window to see this:





The debugger pauses the server’s response where we set our breakpoint. The variables that we watch are updated in the right panel and also in the line of code that created it.


Let’s complete the response’s execution by pressing the continue button at the right panel, right above Paused on breakpoint. When the response is complete, you will see a successful JSON response in the browser window used to speak with the Node.js server:


```
{"message": "Hello world"}

```


In this way, Chrome DevTools does not require changes to the code to add breakpoints. If you prefer to use graphical applications over the command line to debug, the Chrome DevTools are more suitable for you.


# Conclusion


In this article, we debugged sample Node.js applications by setting up watchers to observe the state of our application, and then by adding breakpoints to allow us to pause execution at various points in our program’s execution. We accomplished this using both the built-in CLI debugger and Google Chrome’s DevTools.


Many Node.js developers log to the console to debug their code. While this is useful, it’s not as flexible as being able to pause execution and watch various state changes. Because of this, using debugging tools is often more efficient, and will save time over the course of developing a project.


To learn more about these debugging tools, you can read the Node.js documentation or the Chrome DevTools documentation. If you’d like to continue learning Node.js, you can return to the How To Code in Node.js series, or browse programming projects and setups on our Node topic page.


