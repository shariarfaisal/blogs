# How To Build Command Line Applications with Node js

```Node.js```

As a developer, chances are you spend most of your time in your terminal, typing in commands to help you get around some tasks.


Some of these commands come built into your Operating System, while some of them you install through some third party helper such as npm, or brew, or even downloading a binary and adding it to your $PATH.


A good example of commonly used applications include npm, eslint, typescript, and project generators, such as Angular CLI, Vue CLI or Create React App.


In this tutorial you’ll build two small CLI applications in Node.js:


1. Quote Of The Day tool that retrieves quotes of the day from https://quotes.rest/qod.
2. A To-Do List app that uses JSON to save data.

# Prerequisites


To complete this tutorial, you will need:


- A local development environment for Node.js. Follow How to Install Node.js and Create a Local Development Environment

# Step 1 – Understanding the Shebang


Whenever you look at any scripting file, you’ll see characters like these in the beginning of the file:


```
file.sh#!/usr/bin/env sh

```


Or this:


```
file.py#!/usr/bin/env python -c

```


They serve as a way for your operating system program loader to locate and use toe  parse the correct interpreter for your executable file. This only works in Unix Systems though.


From Wikipedia:



In computing, a shebang is the character sequence consisting of the characters number sign and exclamation mark (#!) at the beginning of a script.

NodeJS has its own supported shebang characters.


Create a new file in your editor called logger.js:


```
nano logger.js


```


Add the following code:


```
#!/usr/bin/env node

console.log("I am a logger")

```


The first line tells the program loader to parse this file with NodeJS. The second line prints text to the screen.


You can try and run the file by typing this in your terminal. You’ll get a permission denied for execution.


```
./logger


```


```
Outputzsh: permission denied: ./logger

```


You need to give the file execution permissions. You can do that with.


```
chmod +x logger
./logger


```


This time you’ll see the output.


```
OutputI am a logger

```


You could have run this program with node logger, but adding the shebang and making the program executable own command lets you avoid typing  node to run it.


# Creating the Quote of the Day App


Let’s create a directory and call it qod. And inside, instantiate a NodeJs app.


```
mkdir qod
cd qod
npm init -y


```


Next, we know we need to make requests to the quotes server, so we could use existing libraries to do just this. We’ll use axios


```
npm install --save axios

```


We’ll also add a chalk, a library to help us print color in the terminal.


```
npm install --save chalk

```


We then write the logic needed to retrieve these quotes.


Create a new file called qod:


```
nano qod


```


Add the following code to the qod file to specify the shebang, load the libraries, and store the API URL:


qod
```
#!/usr/bin/env node

const axios = require('axios');
const chalk = require('chalk');

const url = "https://quotes.rest/qod";

```


Next, add this code to make a GET request to the API:


```
[label qod]// make a get request to the url
axios({
  method: 'get',
  url: url,
  headers: { 'Accept': 'application/json' }, // this api needs this header set for the request
}).then(res => {
  const quote = res.data.contents.quotes[0].quote
  const author = res.data.contents.quotes[0].author
  const log = chalk.green(`${quote} - ${author}`) // we use chalk to set the color green on successful response
  console.log(log)
}).catch(err => {
  const log = chalk.red(err) // we set the color red here for errors.
  console.log(log)
})

```


Save the file.


Change the file permissions so the file is executable:


```
chmod +x qod


```


Then run the application:


```
./qod

```


You’ll see a quote:


```
OutputThe best way to not feel hopeless is to get up and do something. Don’t wait for good things to happen to you. If you go out and make some good things happen, you will fill the world with hope, you will fill yourself with hope. - Barack Obama

```


This example shows you can use external libraries in your CLI applications.


Now let’s create a CLI program that saves data.


# Creating a To-Do List


This will be a bit more complex, as it will involve data storage, and retrieval. Here’s what we’re trying to achieve.


1. We need to have a command called todo
2. The command will take in four arguments. new,  get, complete, and help.

So the available commands will be


```
./todo new // create a new todo
./todo get // get a list of all your todos
./todo complete // complete a todo item.
./todo help // print the help text

```


Create a directory called todo, and instantiate a Node.js app:


```
mkdir todo
cd todo
npm install -y


```


Next, install chalk again, so that you can log with colors.


```
npm install --save chalk

```


The first thing we’re going to do is make sure we have these commands available. To get the commands working, we’ll use NodeJs’ process/argv which returns a string array of command line arguments The process.argv property returns an array containing the command line arguments passed when the Node.js process was launched.


Create the file todo:


```
nano todo


```


Add this to the todo file.


todo
```
#!/usr/bin/env node

console.log(process.argv)

```


Give the file executable permissions, and then run it with a new command.


```
chmod +x ./todo
./todo new


```


You’re going to get this output:


```
Output[ '/Users/sammy/.nvm/versions/node/v8.11.2/bin/node',
  '/Users/sammy/Dev/scotch/todo/todo',
  'new' ]

```


Notice that the first two strings in the array are the interpreter and the file full path to the program. The rest of the array contains the arguments passed; in this case it’s new.


To be safe, let’s restrict these, so that we can only accept the correct number of arguments, which is one, and they can only be new, get and complete.


Modify the todo file so it looks like the following:


todo
```
#!/usr/bin/env node

const chalk = require('chalk')
const args = process.argv

// usage represents the help guide
const usage = function() {
  const usageText = `
  todo helps you manage you todo tasks.

  usage:
    todo <command>

    commands can be:

    new:      used to create a new todo
    get:      used to retrieve your todos
    complete: used to mark a todo as complete
    help:     used to print the usage guide
  `

  console.log(usageText)
}

// used to log errors to the console in red color
function errorLog(error) {
  const eLog = chalk.red(error)
  console.log(eLog)
}

// we make sure the length of the arguments is exactly three
if (args.length > 3) {
  errorLog(`only one argument can be accepted`)
  usage()
}

```


We’ve first assigned the command line arguments to a variable, and then we check at the bottom that the length is not greater than three.


We’ve also added a usage string, that will print what the command line app expects. Run the app with wrong parameters like below.


```
./todo new app


```


```
Outputonly one argument can be accepted

todo helps you manage you todo tasks.

usage:
  todo <command>

  commands can be:

  new:      used to create a new todo
  get:      used to retrieve your todos
  complete: used to mark a todo as complete
  help:     used to print the usage guide

```


If you run it with one parameter, it will not print anything, which means the code passes.


Next, we need to make sure only the four commands are expected, and everything else will be printed as invalid.


Add a list of the commands at the top of the file:


todo
```
const commands = ['new', 'get', 'complete', 'help']

```


And then check with the passed in command after we’ve checked the length:


todo
```
...
if (commands.indexOf(args[2]) == -1) {
  errorLog('invalid command passed')
  usage()
}


```


Now, if we run the app with an invalid command, we get this.


```
./todo ne


```


```
Outputinvalid command passed

  todo helps you manage you todo tasks.

  usage:
    todo <command>

    commands can be:

    new:      used to create a new todo
    get:      used to retrieve your todos
    complete: used to mark a todo as complete
    help:     used to print the usage guide

```


Now let’s implement the help command by calling the usage function. Let’s add this to the todo file:


todo
```

//...
switch(args[2]) {
  case 'help':
    usage()
    break
  case 'new':
    break
  case 'get':
    break
  case 'complete':
    break
  default:
    errorLog('invalid command passed')
    usage()
}
//...

```


We have a switch statement which will call functions based on what command has been called. If you look closely, you’ll notice the help case just calls the usage function.


The new command will create a new todo item and save it in a json file. The library we will use is lowdb. We could easily write functions to read and write to a json file, if we wanted to.


Install lowdb


```
npm install --save lowdb


```


Let’s add [readline](https://nodejs.org/api/readline.html) and lowdb dependencies, to help us with storing data. The lowdb code is standard from their github page.


todo
```

//...
const rl = require('readline');

const low = require('lowdb')
const FileSync = require('lowdb/adapters/FileSync')

const adapter = new FileSync('db.json')
const db = low(adapter)

// Set some defaults (required if your JSON file is empty)
db.defaults({ todos: []}).write()
//...

```


Next, we’ll add a function to prompt the user to input data.


todo
```

//...
function prompt(question) {
  const r = rl.createInterface({
    input: process.stdin,
    output: process.stdout,
    terminal: false
  });
  return new Promise((resolve, error) => {
    r.question(question, answer => {
      r.close()
      resolve(answer)
    });
  })
}
//...

```


Here we are using the readline library to create an interface that will help us prompt a user to and then read the output.


Next, we need to add a function that will be called when a user types in the new command:


todo
```

//...
function newTodo() {
  const q = chalk.blue('Type in your todo\n')
  prompt(q).then(todo => {
    console.log(todo)
  })
}
//...

```


We’re using chalk to get the blue color for the prompt. And then we will log the result.


Lastly, call the function in the new case.


todo
```

// ...
switch(args[2]) {
  //...
  case 'new':
    newTodo()
    break
	// ...
}
// ...

```


When you run the app now with the new command, you will be prompted to add in a todo. Type and press enter.


```
./todo new


```


```
OutputType in your todo
This my todo  aaaaaaw yeah
This my todo  aaaaaaw yeah

```


You should see something similar to this.





Notice also, that a db.json file has been created in your file system, and it has a todos property.


Next, let’s add in the logic for adding a todo. Modify the newTodo function.


todo
```

//...
function newTodo() {
  const q = chalk.blue('Type in your todo\n')
  prompt(q).then(todo => {
    // add todo
    db.get('todos')
      .push({
	  title: todo,
	  complete: false
	  })
      .write()
  })
}
//...

```


Run the code again.


```
./todo new


```


```
OutputType in your todo
Take a Scotch course

```


If you look at your db.json, you’ll see the todo added. Add two more, so that we can retrieve them in the next get command. Here’s what the db.json file looks like with more records:


db.json
```

{
  "todos": [
    {
      "title": "Take a Scotch course",
      "complete": false
    },
    {
      "title": "Travel the world",
      "complete": false
    },
    {
      "title": "Rewatch Avengers",
      "complete": false
    }
  ]
}

```


After creating the new command, you should already have an idea of how to implement the  get command.


Create a function that will retrieve the todos.


todo
```

//...
function getTodos() {
  const todos = db.get('todos').value()
  let index = 1;
  todos.forEach(todo => {
    const todoText = `${index++}. ${todo.title}`
    console.log(todoText)
  })
}
//...

// switch statements
switch(args[2]) {
	//...
	case 'get':
		getTodos()
		break
	//...
}
//....

```


Run the command again:


```
./todo get


```


Running the app now will give this output:


```
Output1. Take a Scotch course
2. Travel the world
3. Rewatch Avengers

```


You can make the color green by using chalk.green.


Next, add the complete command, which is a little bit complicated.


You can do it in two ways.


1. Whenever a user types in ./todo complete, we could list all the todos, and the ask them to type in the number/key for the todo to mark as complete.
2. We can add in another parameter, so that a user can type in ./todo get, and then choose the task to mark as complete with a parameter, such as ./todo complete 1.

Since you learned how to do the first method when you implemented the new command, we’ll look at option 2.


With this option, the command ./todo complete 1, will fail our validity check for the number of commands given. We therefore first need to handle this. Change the function that checks the length of the arguments to this:


todo
```

//...
// we make sure the length of the arguments is exactly three
if (args.length > 3 && args[2] != 'complete') {
  errorLog('only one argument can be accepted')
  usage()
  return
}
///...

```


This approach uses truth tables, where TRUE && FALSE will equal FALSE, and the code will be skipped when complete is passed.


We’ll then grab the value of the new argument and make the value of todo as completed:


todo
```

//...
function completeTodo() {
  // check that length
  if (args.length != 4) {
    errorLog("invalid number of arguments passed for complete command")
    return
  }

  let n = Number(args[3])
  // check if the value is a number
  if (isNaN(n)) {
    errorLog("please provide a valid number for complete command")
    return
  }

  // check if correct length of values has been passed
  let todosLength = db.get('todos').value().length
  if (n > todosLength) {
    errorLog("invalid number passed for complete command.")
    return
  }

  // update the todo item marked as complete
  db.set(`todos[${n-1}].complete`, true).write()
}
//...

```


Also, update the switch statement to include the complete command:


todo
```

//...
case 'complete':
    completeTodo()
    break
//...

```


When you run this with ./todo complete 2, you’ll notice your db.json has changed to this, marking the second task as complete:


db.json
```

{
  "todos": [
    {
      "title": "Take a Scotch course",
      "complete": false
    },
    {
      "title": "Travel the world",
      "complete": true
    },
    {
      "title": "Rewatch Avengers",
      "complete": false
    }
  ]
}

```


The last thing we need to do is change ./todo get to only show tasks that are done. We’ll use emojis for this. Modify getTodos with this code:


todo
```

//...
function getTodos() {
  const todos = db.get('todos').value()
  let index = 1;
  todos.forEach(todo => {
    let todoText = `${index++}. ${todo.title}`
    if (todo.complete) {
      todoText += ' ✔ ️' // add a check mark
    }
    console.log(chalk.strikethrough(todoText))
  })
  return
}
//...

```


When you now type in the ./todo get you’ll see this.


```
Output1. Take a Scotch course
2. Travel the world ✔ ️
3. Rewatch Avengers

```


# Conclusion


You’ve written two CLI applications in Node.js.


Once your app is working, put the file into a bin folder.  This way, npm knows how to work with the executable when you distribute it. Also, regardless of where you place the executable, you should update package.json bin property.


The focus for this article was to look at how CLI applications are built with vanilla nodejs, but when working in the real world, it would be more productive to use libraries.


Here’s a list of helpful libraries to help you write awesome CLI applications, which you can publish to npm.


1. vopral - fully featured interactive CLI framework
2. meow - CLI helper library
3. commanderjs - CLI library
4. minimist - for arguments parsing
5. yargs - arguments parsing

And not to mention libraries like chalk that helped us with colors.


As an additional exercise, try to add a Delete command to the CLI.


