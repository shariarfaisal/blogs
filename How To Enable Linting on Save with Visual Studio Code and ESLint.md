# How To Enable Linting on Save with Visual Studio Code and ESLint

```JavaScript```

## Introduction


Style guides allow us to write consistent, reusable, and clean code. These rules can be automated and enforced with linters. This will spare you from manually checking for indentation or using single quotes or double-quotes.


Visual Studio code can support linting on every save. Your workflow may benefit from performing frequent lint checks to address small issues over time rather than addressing many large issues that may delay deploying code.


In this tutorial, you will install ESLint, construct rules, and enable codeActionsOnSave in Visual Studio Code.


# Prerequisites


If you would like to follow along with this article, you will need:


- Download and install the latest version of Visual Studio Code.
- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.

This tutorial was verified with Node v16.6.2, npm v7.21.0, eslint v7.32.0, and Visual Studio Code v1.59.1.


# Step 1 – Setting Up the Project


There are various linters for different languages and types of projects. For the needs of this tutorial, you will need to have ESLint installed and configured.


First, create a new project directory:


```
mkdir eslint-save-example


```


Then, navigate to the project directory:


```
cd eslint-save-example


```


Initialize a new project:


```
npm init -y


```


And install eslint:


```
npm install eslint@7.32.0


```


Then, initialize eslint:


```
npx eslint --init


```


Answer the prompt with the following choices:


```
? How would you like to use ESLint? To check syntax and find problems
? What type of modules does your project use? JavaScript modules (import/export)
? Which framework does your project use? None of these
? Does your project use TypeScript? No
? Where does your code run? Browser, Node
? What format do you want your config file to be in? JavaScript

```


At this point, you will have a new project with package.json and .eslintrc.js files.


In order to use ESLint in Visual Studio Code, you should install the ESLint extension available in Visual Studio Code’s marketplace.


# Step 2 – Creating an Example with Errors


Next, create a JavaScript file that intentionally breaks common rules, like inconsistent spacing and indentation, quotation marks, and semicolons:


index.js
```
const helloYou    = (name)=> {
  name = 'you' || name   ;
  console.log("hello" + name + "!" )
}

```


Open the file in Visual Studio Code and observe the indications for ESLint rule violations:





helloYou is underlined. Hovering over this line in Visual Studio Code displays the following tooltip message: 'helloYou' is assigned a value but never used. This is because the rule .eslint(no-unused-vars) is enabled by eslint:recommended.


We can address this issue by using the variable:


index.js
```
const helloYou    = (name)=> {
  name = 'you' || name   ;
  console.log("hello" + name + "!" )
}

console.log(helloYou)

```


To address the other issues, we will have to add rules.


# Step 3 – Adding Rules


eslint --init created a file called eslintrc.js (or .yml or .json if that’s the option you selected):


.eslintrc.js
```
module.exports = {
  'env': {
    'browser': true,
    'es2021': true,
    'node': true
  },
  'extends': 'eslint:recommended',
  'parserOptions': {
    'ecmaVersion': 12,
    'sourceType': 'module'
  },
  'rules': {
  }
};

```


Let’s add rules for consistent spacing and indentation. And enforce single quotes preferred over double-quotes. And also enforce mandatory semicolons at the end of lines.


.eslintrc.js
```
module.exports = {
  // ...
  'rules': {
    'quotes': ['error', 'single'],
    // we want to force semicolons
    'semi': ['error', 'always'],
    // we use 2 spaces to indent our code
    'indent': ['error', 2],
    // we want to avoid extraneous spaces
    'no-multi-spaces': ['error']
  }
};

```


Save the changes to your file.


In your code editor, open the JavaScript file you created earlier. All the broken rules will be indicated.


If you have the ESLint extension installed you can use CTRL+SHIFT+P to open the Command Palette. Then search for ESLint: Fix all auto-fixable Problems and press ENTER (or RETURN).


The auto-fixable problems will be corrected:


index.js
```
const helloYou = (name)=> {
  name = 'you' || name ;
  console.log('hello' + name + '!' );
};

console.log(helloYou());

```


You are free from counting indentation and checking for quotation marks and semicolons!


# Step 4 – Adding Code Actions on Save


Trying to manually run ESLint: Fix all auto-fixable Problems periodically is not very reliable. However, having lint rules run every time you save your work can be more reliable. You can set up ESLint to run auto-fix every time you press CTRL+S (or COMMAND+S).


For ESLint to work correctly on file same, you must change the Visual Studio Code preferences. Go to File > Preferences > Settings (or Code > Preferences > Settings).


For this tutorial, we will modify the Workspace settings - it is also possible to apply these settings for all projects. Click Workspace and search for Code Actions On Save:


```
Editor: Code Actions On Save
Code action kinds to be run on save.

Edit in settings.json

```


Then click settings.json.


.vscode/settings.json
```
{
  "editor.codeActionsOnSave": null
}

```


In settings.json paste the following code:


.vscode/settings.json
```
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.validate": ["javascript"]
 }

```


Now, undo the fixes you made to the JavaScript file you created earlier. Then save the file. The auto-fixable problems will be automatically addressed.


# Conclusion


In this tutorial, you installed ESLint, constructed rules, and enabled codeActionsOnSave in Visual Studio Code.


