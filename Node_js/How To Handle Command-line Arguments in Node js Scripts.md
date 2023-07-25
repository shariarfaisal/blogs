# How To Handle Command-line Arguments in Node js Scripts

```Node.js```

## Introduction


Command-line arguments are a way to provide additional input for commands. You can use command-line arguments to add flexibility and customization to your Node.js scripts.


In this article, you will learn about argument vectors, detecting argument flags, handling multiple arguments and values, and using the commander package.


# Prerequisites


To follow through this tutorial, you’ll need:


- A local development environment for Node.js. Follow How to Install Node.js and Create a Local Development Environment.

This tutorial was verified with Node v16.10.0, npm v7.12.2, and commander v7.2.0.


# Using Argument Vectors


Node.js supports a list of passed arguments, known as an argument vector. The argument vector is an array available from process.argv in your Node.js script.


The array contains everything that’s passed to the script, including the Node.js executable and the path and filename of the script.


If you were to run the following command:


```
node example.js -a -b -c


```


Your argument vector would contain five items:


```
[
  '/usr/bin/node',
  '/path/to/example.js',
  '-a',
  '-b',
  '-c'
]

```


At the very least, a script that’s run without any arguments will still contain two items in the array, the node executable and the script file that is being run.


Typically the argument vector is paired with an argument count (argc) that tells you how many arguments have been passed in. Node.js lacks this particular variable but we can always grab the length of the argument vector array:


example.js
```
if (process.argv.length === 2) {
  console.error('Expected at least one argument!');
  process.exit(1);
}

```


This example code will check the length of argv. A length of 2 would indicate that only the node executable and the script file are present. If there are no arguments, it will print out the message: Expected at least one argument! and exit.


# Using Argument Flags


Let’s consider an example that displays a default message. However, when a specific flag is present, it will display a different message.


example.js
```
if (process.argv[2] && process.argv[2] === '-f') {
  console.log('Flag is present.');
} else {
  console.log('Flag is not present.');
}

```


This script checks if we have a third item in our argument vector. The index is 2 because arrays in JavaScript are zero-indexed. If a third item is present and is equal to -f it will alter the output.


Here is an example of running the script without arguments:


```
node example.js


```


And the generated output:


```
OutputFlag is not present.

```


Here is an example of running the script with arguments:


```
node example.js -f


```


And the generated output:


```
OutputFlag is present.

```


We don’t have to limit ourselves to modifying the conditional control structure, we can use the actual value that’s been passed to the script as well:


example.js
```
const custom = (process.argv[2] || 'Default');
console.log('Custom: ', custom);

```


Instead of a conditional based on the argument, this script takes the value that is passed in (defaulting to "Default" when the argument is missing) and injects it into the script output.


# Using Multiple Arguments with Values


We have written a script that accepts an argument and one that accepts a raw value, what about in scenarios where we want to use a value in conjunction with an argument?


To make things a bit more complex, let’s also accept multiple arguments:


example.js
```
// Check to see if the -f argument is present
const flag = (  
  process.argv.indexOf('-f') > -1 ? 'Flag is present.' : 'Flag is not present.'
);

// Checks for --custom and if it has a value
const customIndex = process.argv.indexOf('--custom');
let customValue;

if (customIndex > -1) {
  // Retrieve the value after --custom
  customValue = process.argv[customIndex + 1];
}

const custom = (customValue || 'Default');

console.log('Flag:', `${flag}`);
console.log('Custom:', `${custom}`);

```


By using indexOf instead of relying on specific index values, we are able to look for the arguments anywhere in the argument vector, regardless of the order!


Here is an example of running the script without arguments:


```
node example.js

```


And the generated output:


```
OutputFlag: Flag is not present.
Custom: Default

```


Here is an example of running the script with arguments:


```
node example.js -f --custom Override


```


And the generated output:


```
OutputFlag: Flag is present.
Custom: Override

```


Now, your command-line script can accept multiple arguments and values.


# Using commander


The aforementioned examples work when the argument input is quite specific. However, users may attempt to use arguments with and without equal signs (-nJaneDoe or --name=JohnDoe), quoted strings to pass-in values with spaces (-n "Jane Doe") and even have arguments aliased to provide short and longhand versions.


That’s where the commander library can help.


commander is a popular Node.js library that is inspired by the Ruby library of the same name.


First, in your project directory, initialize your project:


```
npm init


```


Then, install commander:


```
npm install commander@7.2.0


```


Let’s take our previous example and port it to use commander:


example-commander.js
```
const commander = require('commander');

commander
  .version('1.0.0', '-v, --version')
  .usage('[OPTIONS]...')
  .option('-f, --flag', 'Detects if the flag is present.')
  .option('-c, --custom <value>', 'Overwriting value.', 'Default')
  .parse(process.argv);

const options = commander.opts();

const flag = (options.flag ? 'Flag is present.' : 'Flag is not present.');

console.log('Flag:', `${flag}`);
console.log('Custom:', `${options.custom}`);

```


commander does all of the hard work by processing process.argv and adding the arguments and any associated values as properties in our commander object.


We can easily version our script and report the version number with -v or --version. We also get some friendly output that explains the script’s usage by passing the --help argument and if you happen to pass an argument that’s not defined or is missing a passed value, it will throw an error.


# Conclusion


In this article, you learned about argument vectors, detecting argument flags, handling multiple arguments and values, and using the commander package.


While you can quickly create scripts with your own command-line arguments, you may want to consider utilizing commander or Inquirer.js if you would like more robustness and maintainability.


