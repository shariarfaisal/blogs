# How To Create Interactive Command-line Prompts with Inquirer js

```Node.js```

## Introduction


Inquirer.js is a collection of common interactive command-line user interfaces. This includes typing answers to questions or selecting a choice from a list.


The inquirer package provides several default prompts and is highly configurable. It is also extensible by way of a plug-in interface. It even supports promises and async/await syntax.


In this article, you will install and explore some of the features of Inquirer.js.


# Prerequisites


If you would like to follow along with this article, you will need:


- A local development environment for Node.js. Follow How to Install Node.js and Create a Local Development Environment.

This tutorial was verified with Node v15.14.0, npm v7.10.0, and inquirer v8.0.0.


# Step 1 — Setting Up the Project


First, open your terminal window and create a new project directory:


```
mkdir inquirer-example

```


Then, navigate to this directory:


```
cd inquirer-example

```


To start adding prompts to your Node.js scripts, you will need to install the inquirer package:


```
npm install inquirer


```


At this point, you have a new project ready to use Inquirer.js.


# Step 2 — Creating Prompts


Now, create a new index.js file in your project directory and open it with your code editor.


Within your script, be sure to require inquirer:


index.js
```
const inquirer = require('inquirer');

```


Add a prompt asking the user for their favorite reptile:


index.js
```
const inquirer = require('inquirer');

inquirer
  .prompt([
    {
      name: 'faveReptile',
      message: 'What is your favorite reptile?'
    },
  ])
  .then(answers => {
    console.info('Answer:', answers.faveReptile);
  });

```


Revisit your terminal window and run the script:


```
node index.js


```


You will be presented with a prompt:


```
Output? What is your favorite reptile?

```


Providing an answer will display the response:


```
Output? What is your favorite reptile? Crocodiles
Answer: Crocodiles

```


You can provide a default value that allows the user to press ENTER without submitting any answer:


index.js
```
const inquirer = require('inquirer');

inquirer
  .prompt([
    {
      name: 'faveReptile',
      message: 'What is your favorite reptile?',
      default: 'Alligators'
    },
  ])
  .then(answers => {
    console.info('Answer:', answers.faveReptile);
  });

```


Run the script again, and you will be presented with a prompt:


```
Output? What is your favorite reptile? (Alligators)

```


Pressing ENTER without an answer will submit the default answer:


```
Output? What is your favorite reptile? Alligators
Answer: Alligators

```


Now, you can create prompts and set default values.


# Step 3 — Creating Multiple Prompts


You may have noticed the .prompt() method accepts an array or objects. That’s because you can string a series of prompt questions together and all of the answers will be available by name as part of the answers variable once all of the prompts have been resolved.


Revisit index.js in your code editor and add a prompt asking the user for their favorite color:


index.js
```
const inquirer = require('inquirer');

inquirer
  .prompt([
    {
      name: 'faveReptile',
      message: 'What is your favorite reptile?',
      default: 'Alligators'
    },
    {
      name: 'faveColor',
      message: 'What is your favorite color?',
      default: '#008f68'
    },
  ])
  .then(answers => {
    console.info('Answers:', answers);
  });

```


Run the script again, and you will be presented with two prompts:


```
Output? What is your favorite reptile? Alligators
? What is your favorite color? #008f68
Answers: { faveReptile: 'Alligators', faveColor: '#008f68' }

```


Now, you can create multiple prompts.


# Step 4 — Using Lists, Raw Lists, Expandable Lists, Checkboxes, Passwords, and Editors


inquirer does support more than prompting a user for text input. For the sake of example, the following types will be showcased by themselves but you very well could chain them together by passing them in the same array.


## List


The list type allows you to present the user with a fixed set of options to pick from, instead of a free form input as the input type provides:


list.js
```
const inquirer = require('inquirer');

inquirer
  .prompt([
    {
      type: 'list',
      name: 'reptile',
      message: 'Which is better?',
      choices: ['alligator', 'crocodile'],
    },
  ])
  .then(answers => {
    console.info('Answer:', answers.reptile);
  });

```


Revisit your terminal window and run the script:


```
node list.js


```


You will be presented with a list prompt:


```
Output? Which is better? (Use arrow keys)
❯ alligator
  crocodile

```


The user can ARROW UP and ARROW DOWN keys to navigate the list of choices. j and k keyboard navigation is also available.


## Raw List


The rawlist type is similar to list. It displays a list of choices and allows the user to enter the index of their choice (starting at 1):


rawlist.js
```
const inquirer = require('inquirer');

inquirer
  .prompt([
    {
      type: 'rawlist',
      name: 'reptile',
      message: 'Which is better?',
      choices: ['alligator', 'crocodile'],
    },
  ])
  .then(answers => {
    console.info('Answer:', answers.reptile);
  });

```


Revisit your terminal window and run the script:


```
node list.js


```


You will be presented with a rawlist prompt:


```
Output? Which is better?
  1) alligator
  2) crocodile
  Answer:

```


Submitting an invalid value will result in a "Please enter a valid index" error.


## Expandable List


The expand type is reminiscent of some command-line applications that present you with a list of characters that map to functionality that can be entered. expand prompts will initially present the user with a list of the available character values and give context to them when the key is pressed:


expand.js
```
const inquirer = require('inquirer');

inquirer
  .prompt([
    {
      type: 'expand',
      name: 'reptile',
      message: 'Which is better?',
      choices: [
        {
          key: 'a',
          value: 'alligator',
        },
        {
          key: 'c',
          value: 'crocodile',
        },
      ],
    },
  ])
  .then(answers => {
    console.info('Answer:', answers.reptile);
  });

```


Revisit your terminal window and run the script:


```
node expand.js


```


You will be presented with a expand prompt:


```
Output? Which is better? (acH)

```


By default, the H option is included which stands for "Help" and upon entering H and hitting ENTER will switch to a list of the options, indexed by their characters that can then be entered to make a selection.


```
Output? Which is better? (acH)
  a) alligator
  c) crocodile
  h) Help, list all options
  Answer:

```


Submitting an invalid value will result in a "Please enter a valid command" error.


## Checkbox


The checkbox type is also similar to list. Instead of a single selection, it allows you to select multiple choices.


checkbox.js
```
const inquirer = require('inquirer');

inquirer
  .prompt([
    {
      type: 'checkbox',
      name: 'reptiles',
      message: 'Which reptiles do you love?',
      choices: [
        'Alligators', 'Snakes', 'Turtles', 'Lizards',
      ],
    },
  ])
  .then(answers => {
    console.info('Answer:', answers.reptiles);
  });

```


Revisit your terminal window and run the script:


```
node checkbox.js


```


You will be presented with a checkbox prompt:


```
Output? Which reptiles do you love? (Press <space> to select, <a> to toggle all, <i> to invert selection)
❯◯ Alligators
 ◯ Snakes
 ◯ Turtles
 ◯ Lizards

```


Similar to the other list types, you can use the arrow keys to navigate. To make a selection, you hit SPACE and can also select all with a or invert your selection with i.


```
OutputAnswer: [ 'Alligators', 'Snakes', 'Turtles', 'Lizards' ]

```


Unlike the other prompt types, the answer for this prompt type will return an array instead of a string. It will always return an array, even if the user opted to not select any items.


## Password


The password type will hide input from the user. This allows users to provide sensitive information that should not be seen:


```
const inquirer = require('inquirer');

inquirer
  .prompt([
    {
      type: 'password',
      name: 'secret',
      message: 'Tell me a secret',
    },
  ])
  .then(answers => {
    // Displaying the password for debug purposes only.
    console.info('Answer:', answers.secret);
  });

```


Revisit your terminal window and run the script:


```
node password.js


```


You will be presented with a password prompt:


```
Output? Tell me a secret [hidden]

```


The input is hidden from the user.


## Editor


The editor type allows users to use their default text editor for larger text inputs.


editor.js
```
const inquirer = require('inquirer');

inquirer
  .prompt([
    {
      type: 'editor',
      name: 'story',
      message: 'Tell me a story, a really long one!',
    },
  ])
  .then(answers => {
    console.info('Answer:', answers.story);
  });

```


Revisit your terminal window and run the script:


```
node editor.js


```


You will be presented with an editor prompt:


```
Output? Tell me a story, a really long one! Press <enter> to launch your preferred editor.

```


inquirer will attempt to open a text editor on the user’s system based on the value of the $EDITOR and $VISUAL environment variables. If neither are present, vim (Linux) and notepad.exe (Windows) will be used instead.


# Conclusion


In this article, you installed and explored some of the features of Inquirer.js. This tool can be useful for retrieving information from users.


Continue your learning with some of the plugins. Like inquirer-autocomplete-prompt, inquirer-search-list, and inquirer-table-prompt.


