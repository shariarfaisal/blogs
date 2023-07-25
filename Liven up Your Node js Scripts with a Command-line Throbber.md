# Liven up Your Node js Scripts with a Command-line Throbber

```Node.js```

A throbber is an animated element that indicates that the computer is working. Known by many names, such as “that hourglass” or the “spinning wheel of death”, this indicator is used to convey to a user that they need to wait a moment while the computer finishes up what it’s doing.


Not to be confused with a progress bar, a throbber is more of an icon and on the command-line can be as simple as just a blinking cursor, a series of ellipses or as complex as a spinning line represented by -, \, | and / characters.


On a website, you can get away with using an animated GIF or for the more advanced, construct an animation with CSS. The command-line implementation is a bit more complex as it involves writing over the same part of the screen over and over again.


If you were to just console.log() each character in the aforementioned animation, you’d end up with something like this:


```
- Loading...
\ Loading...
| Loading...
/ Loading...
- Loading...
\ Loading...

```


Between you and me, I wouldn’t even know how to write and rewrite over the same region of the terminal screen with Node.js. Fortunately, the infamous full-time open sourcerer Sindre Sorhus has things already sorted out and packaged up for us to use!


# Getting Started


The package in question is ora, the self proclaimed elegant terminal spinner. To get started with ora, simply add it to your package:


```
# via npm
$ npm install ora

# via yarn
$ yarn add ora

```


With the package installed, you will need to require() it from within your script:


```
const ora = require('ora');

```


# Basic Usage


Once included in your script, you can start and stop the throbber with no trouble at all:


```
const ora = require('ora');

const throbber = ora('Rounding up all the alligators').start();

// Simulating some asynchronous work for 10 seconds...
setTimeout(() => {
  throbber.stop();
}, 1000 * 10);

```


By default ora displays a snazzy spinner that is comprised of dots similar to a marching ants indicator (that dotted line around a selection inside of an image app):


```
⠏ Rounding up all the alligators

```


When the payload of setTimeout() fires, the throbber is stopped and erased from the screen.



Note: In place of the timeout, you could fire off an AJAX call or perform some other bit of work that will take some time, stopping the ora object when things are completed.

# Persisting Output


If you’d prefer, you can also keep your loading text on the screen and replace your throbber with another character. You can even change the loading text to say “done” or something similar:


```
const ora = require('ora');

const throbber = ora('Rounding up all the alligators').start();

// Simulating some asynchronous work for 10 seconds...
setTimeout(() => {
  throbber.stopAndPersist({
    symbol: '🐊',
    text: 'All done rounding up the alligators!',
  });
}, 1000 * 10);

```


Now instead of the throbber and text abruptly leaving the screen, you are left with a friendly message and an even friendlier emoji!


# Custom Throbbers


The default throbber output from ora is pretty great for an out of the box option, but if you want something more, ora does come with the cli-spinners project as a dependency.


This additional project, by the same author, comes with nearly 70 (at the time of this writing) different “spinner” animations, covering just about every use case you can think of.


Thought of something that’s not covered?


If that’s the case, you can easily define your own animation as some JSON with an interval (the time between frames) and an array of the frames you’d like to cycle through.


For instance, if you’d like to have your very own reptile themed throbber, you can write something similar to this:


```
const ora = require('ora');

const throbber = ora({
  text: 'Rounding up all the reptiles',
  spinner: {
    frames: ['🐊', '🐢', '🦎', '🐍'],
    interval: 300, // Optional
  },
}).start();

// Simulating some asynchronous work for 10 seconds...
setTimeout(() => {
  throbber.stop();
}, 1000 * 10);

```


Now you’re only limited by your imagination.


