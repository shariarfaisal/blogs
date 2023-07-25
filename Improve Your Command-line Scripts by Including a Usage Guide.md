# Improve Your Command-line Scripts by Including a Usage Guide

```Node.js```

If you’re already familiar with JavaScript in your day to day programming adventures, it wouldn’t be too shocking if you reached for it when you need to write a quick command-line script. Quick scripts have a tendency to be somewhat dirty, usually lacking usage documentation. The command-line-usage package makes it easy to add in professional-looking script usage output to your quick and dirty scripts.


# Getting Started


To get started with command-line-usage, you’ll need to add it to via your favorite package manager:


```
# npm
$ npm install command-line-usage --save

# Yarn
$ yarn add command-line-usage

```


Be sure to import command-line-usage:


```
const cliUsage = require('command-line-usage');

```


While this article is focused on the command-line-usage package, it’s worth noting that the package leverages the chalk template literal syntax and plays quite nicely with chalk methods like chalk.red().


If you would prefer to use the chalk methods instead of the template literal syntax, be sure to also import chalk, which is a dependency of command-line-usage:


```
const chalk = require('chalk');

```


For a more in-depth look at chalk you can check out our article Styling Output from Command-line Node.js Scripts with Chalk


# Basic Usage


With everything installed and properly imported, we can build out some simple usage information:


```
const sections = [
  {
    header: '🐊 Alligator.io CLI Script',
    content: 'From {bold your friends} at {underline Alligator.io}',
  },
  {
    header: 'Usage',
    content: [
      '% script {bold --file} {underline /path/to/file} ...',
    ],
  },
];

const usage = cliUsage(sections);
console.info(usage);

```


The sections array contains groupings of information you’d like displayed in your usage information. You can have as many sections as you’d like with different options within each section.


Running the sections array through command-line-usage will generate a string which we can then log to the console.


# List Arguments


To list out the arguments for a script, you can use the optionList property which takes a series of options:


```
const sections = [
  {
    header: 'Mandatory Options',
    optionList: [
      {
        name: 'file',
        alias: 'f',
        type: String,
        typeLabel: '{underline /path/to/file}',
        description: 'The file to do stuff with',
      },
    ],
  },
  {
    header: 'Optional Stuff',
    optionList: [
      {
        name: 'letters',
        alias: 'l',
        type: String,
        typeLabel: '{underline letter} ...',
        description: 'This option takes multiple values',
        multiple: true,
        defaultOption: 'a b c',
      },
      {
        name: 'help',
        alias: '?',
        type: Boolean,
        description: 'Print this usage guide.',
      }
    ],
  },
];

const usage = cliUsage(sections);
console.info(usage);

```


The command-line-usage package takes care of all of the hard work by spacing out the arguments in a table layout.


# Table Layout


Speaking of the table layout used when presenting the argument list, you can also use the table layout when passing the content property an array. Simply pass-in an array of objects instead of strings and command-line-usage will format the information like a table:


```
const sections = [
  {
    header: 'Table Layout Example',
    content: [
      {
        desc: 'More great Alligator.io articles:',
        url: '{underline https://alligator.io}',
      },
      {
        desc: '`command-line-usage` on GitHub:',
        url: '{underline https://git.io/fh9TQ}',
      },
    ],
  },
];

const usage = cliUsage(sections);
console.info(usage);

```


# Having Some Fun


Sometimes you just want to have some fun. As mentioned earlier, command-line-usage can leverage chalk which means you can spice up your usage output with a splash of color.


To get a little crazier, if you were to channel your inner Demoscene artist and put together some ASCII/ANSI art complete with escape codes, you’ll want to include the raw property to ensure things are displayed correctly.


```
const sections = [
  {
    header: 'Raw Like Sushi',
    raw: true,
    content: [
      '{red   ,iiiiiiiiii,}',
      '{red ,iiiiiiiiiiiiii,}',
      `{red iii'        'ii'}`,
      `{white   '.________.'}`,
    ],
  },
];

const usage = cliUsage(sections);
console.info(usage);

```


🍣


# Conclusion


As amazing as the command-line-usage package is at making it easy to generate script usage information, it’s still somewhat lacking. One thing to keep in mind is that the package doesn’t actually do any sort of command-line argument handling.


The actual handling of the command-line arguments will need to either be handled by rolling your own solution by parsing the argument vector process.argv yourself or by leveraging a package like commander.


For more information check out our article Handling Command-line Arguments in Node.js Scripts


