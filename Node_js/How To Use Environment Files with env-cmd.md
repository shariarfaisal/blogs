# How To Use Environment Files with env-cmd

```Node.js```

## Introduction


Environment variables allow you to switch between your local development, testing, staging, user acceptance testing (UAT), production, and any other environments that are part of your project’s workflow.


Instead of passing variables into your scripts individually, env-cmd lets you group variables into an environment file (.env) and pass them to your script.


In this article, you will install and use env-cmd in an example project.


# Prerequisites


To complete this tutorial, you will need:


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.
- There is an optional section on adding files to .gitignore. This may require installing and configuring git if you wish to follow along.
- Familiarity with the terminal window and a code editor will also be beneficial.


Note: This tutorial has been updated to use the commands for env-cmd after version 9.0.0.

This tutorial was verified with Node v15.14.0, npm v7.10.0, and env-cmd v10.0.1.


# Step 1 – Setting Up the Project


This tutorial assumes you have a new project. Create a new directory:


```
mkdir env-cmd-example


```


Then navigate to the directory:


```
cd env-cmd-example


```


It is generally considered a bad practice to commit your environment files to your version control system. If the repository is forked or shared, the credentials will be available to others as they will be forever recorded in the project history.


It is recommended to add the file to your .gitignore.



Note: This will not be required for the scope of this tutorial, but is presented here for educational purposes.
Initialize a new git project:
git init


Create a .gitignore file and add the patterns to exclude your environment file:
.gitignore
.env
.env.js
.env.json
.env-cmdrc

For this tutorial, you can exclude .env, .env.js, .env.json, .env-cmdrc.

Then, create an .env file for the project.


Open the file in your code editor and add the following line of code:


.env
```
creature=shark
green=#008f68
yellow=#fae042

```


This defines creature to shark, green to #008f68, yellow to #fae042.


Then, create a new log.js file:


log.js
```
console.log('NODE_ENV:', process.env.NODE_ENV);
console.log('Creature:', process.env.creature);
console.log('Green:', process.env.green);
console.log('Yellow:', process.env.yellow);

```


This will log the previously defined variables to the console. And it will also print out the NODE_ENV value.


Now, your example is prepared for using environment files with env-cmd.


# Step 2 – Using env-cmd


Modern npm and yarn can run env-cmd without making it a dependency.


Use either npx:


```
npx env-cmd node log.js


```


Or yarn run:


```
yarn run env-cmd node log.js


```


Otherwise, you can install the package as a dependency or devDependency:


```
npm install env-cmd@10.0.1


```


The env-cmd package installs an executable script named env-cmd which can be called before your scripts to easily load environment variables from an external file.


Depending on your setup, you can reference env-cmd in a few different ways.


Perhaps the most compatible across package managers is to add a custom script to your package.json file:


package.json
```
{
  "scripts": {
    "print-log": "env-cmd node log.js"
  }
}

```


For example, with npm, you will be able to run this custom script with the following command:


```
npm run print-log


```


If you would prefer to use env-cmd directly from the command line, you can call it directly from node_modules:


```
./node_modules/.bin/env-cmd node log.js


```


Going forward, this tutorial will use the npx approach, but all approaches are designed to work similarly.


Now, use one of the approaches in your terminal.


Regardless of how you choose to run the script, env-cmd will load the .env file, and the logging script will report back the variables.


```
OutputNODE_ENV: undefined
Creature: shark
Green: #008f68
Yellow: #fae042

```


You may have noticed that the NODE_ENV value is undefined. That’s because NODE_ENV was not defined in the .env file.


It is possible to pass in NODE_ENV before calling env-cmd.


For example, here is the command for npx:


```
NODE_ENV=development npx env-cmd node log.js


```


Run the command again with the NODE_ENV defined:


```
OutputNODE_ENV: development
Creature: shark
Green: #008f68
Yellow: #fae042

```


At this point, you have learned to use env-cmd with an .env file.


# Step 3 – Using Different File Formats


env-cmd by default expects an .env file in the project root directory. However, you can change the file type and path with the --file (-f) option.


Regardless of how you reference it, you have a wide variety of file formats available to store your environment variables.


## JSON File


Here is an example of an .env.json file:


.env.json
```
{
  "creature": "shark",
  "green": "#008f68",
  "yellow": "#fae042"
}

```


And here is an example of using this file with env-cmd:


```
NODE_ENV=development npx env-cmd --file .env.json node log.js


```


Now you have learned how to use a JSON environment file.


## JavaScript


Here is an example of an .env.js file:


.env.js
```
module.exports = {
  creature: 'shark',
  green: '#008f68',
  yellow: '#fae042'
};

```


And here is an example of using this file with env-cmd:


```
NODE_ENV=development npx env-cmd --file .env.js node log.js


```


Now you have learned how to use a JavaScript environment file.


## RC File


The rc file format is special because it allows you to define multiple environments in a single JSON file and reference the environment by name instead of by file.


The “runcom” file is also special in that it must be named .env-cmdrc and be present at the root of your project.


Here is an example of an .env-cmdrc file with environments defined for development, staging, and production:


.env-cmdrc
```
{
  "development": {
    "NODE_ENV": "development",
    "creature": "shark",
    "green": "#008f68",
    "yellow": "#fae042",
    "otherVar1": 1
  },
  "staging": {
    "NODE_ENV": "staging",
    "creature": "whale",
    "green": "#6db65b",
    "yellow": "#efbb35",
    "otherVar2": 2
  },
  "production": {
    "NODE_ENV": "production",
    "creature": "octopus",
    "green": "#4aae9b",
    "yellow": "#dfa612",
    "otherVar3": 3
  }
}

```


Using the .env-cmdrc values will require an --environments (-e) option.


Then you can reference a single environment:


```
npx env-cmd --environments development node log.js


```


You can even reference multiple environments, which will merge together each of the environment’s variables, with the last environment taking precedence if there are overlapping variables:


```
npx env-cmd --environments development,staging,production node log.js


```


By specifying all three of our environments, each of the otherVar values will be set, with the rest of the variables being sourced from the final environment listed, production.


# Step 4 – Using Graceful Fallbacks with --fallback


In situations where a custom environment file is not present:


```
npx env-cmd -f .env.missing node log.js

```


env-cmd will throw an error:


```
OutputError: Failed to find .env file at path: .env.missing

```


In situations where there is an unexpected problem with the custom env file path, env-cmd can attempt to load an .env file from the root of your project. To do so, pass in the --fallback flag:


```
npx env-cmd --file .env.missing --fallback node log.js


```


Now, if there is a valid .env file to fall back to, this command will not display any errors.


# Step 5 – Using Existing Environment Values with --no-override


There are situations where you may want to keep all or some of the variables already set in the environment.


To respect the existing environment variables instead of using the values in your .env file, pass env-cmd the --no-override flag:


```
NODE_ENV=development creature=squid npx env-cmd --no-override node log.js


```


This will result in the following output:


```
OutputNODE_ENV: development
Creature: squid
Green: #008f68
Yellow: #fae042

```


Notice that the creature value has been set to squid instead of shark which was defined in the .env file.


# Conclusion


In this article, you installed and used env-cmd in an example project.


Using environment files can help you switch between “development” and “production” environments.


