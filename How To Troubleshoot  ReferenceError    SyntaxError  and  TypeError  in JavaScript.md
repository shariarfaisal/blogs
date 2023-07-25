# How To Troubleshoot  ReferenceError    SyntaxError  and  TypeError  in JavaScript

```Development``` ```JavaScript```

## Introduction


JavaScript is a programming language used in frontend and backend development. When working with JavaScript, deciphering error messages can feel a bit daunting. Understanding what an error message is referring to is important when you’re troubleshooting issues within an application. Fortunately, modern browsers come with built-in debugging tools that assist with this. Each browser handles error messages differently when it comes to visual representation. Nonetheless, the error messages clue you in to what is happening with your JavaScript code.


In this tutorial, you’ll learn about three common JavaScript error types that appear in a browser environment: ReferenceError, SyntaxError, and TypeError. To follow along, you should have an understanding of JavaScript and the developer console.


You can learn more about JavaScript from our How to Code in JavaScript tutorial and about the developer console with our How to Use the JavaScript Developer Console tutorial.


The following examples are not exhaustive and serve as a primer on common error messages you may encounter. Furthermore, the error messages in the examples are from the Chrome web browser. Error messages you receive from Firefox or Edge might contain slightly different messaging, but the error types are the same.


# Understanding JavaScript Error Types


Errors in JavaScript are based on the Error object. It is a built-in object filled with information about the type of error that has occurred, followed by a message detailing the possible cause. For example, you may encounter an error that states something like the following:


```
VM170:1 Uncaught ReferenceError: shark is not defined
    at <anonymous>:1:1

```


If you deconstruct this error message, you learn that ReferenceError is the type of error that was identified. Following the semicolon is the error message describing the error: shark is not defined. The last line in this message details where the error occurs in your code 1:1.


Any time an error is caught in the browser, the error type and message will indicate the issue. With this information, you can determine how to debug your code based on the error type and message you receive.


# Understanding the ReferenceError


A ReferenceError occurs when you try to access a variable that you haven’t created yet. It also occurs when you call a variable before initializing it.


## Encountering Undefined Variables


For instance, if you misspell a variable name while executing a console.log(), you’ll receive a ReferenceError:


```
let sammy = 'A Shark dreaming of the cloud.';

console.log(sammmy);

```


```
OutputUncaught ReferenceError: sammmy is not defined
    at <anonymous>:1:13

```


The variable sammmy, with three m's, does not exist and is not defined. To fix this error, you can update the variable to the correct spelling and run the command again. If it’s successful, you won’t receive an error message. Overall, reviewing your code for any misspelling can help prevent undefined variable errors.


## Accessing a Variable Before it’s Declared


Similarly, you may encounter the following error when you try to access a variable before it’s declared in your code:


```
function sharkName() {
    console.log(shark);

    let shark = 'sammy';
}

```


```
OutputVM223:2 Uncaught ReferenceError: Cannot access 'shark' before initialization
    at sharkName (<anonymous>:2:17)
    at <anonymous>:1:1

```


In this example, the console.log(shark) is executed before the shark variable is declared, which results in the error. In general, it is good practice to declare your variables first before trying to access it.



Note: Because of how let and const declarations operate, the declarations are hoisted, but not currently accessible in the previous example. In instances like this, let and const variables enter what’s referred to as the ‘Temporal Dead Zone’. To avoid this, declare your let and const variables at the beginning of a scope.

To fix the example code, declare the shark variable before executing the console.log() command:


```
function sharkName() {
    let shark = 'sammy';

    console.log(shark);
}

```


The ReferenceError is often tied to your variables and scope. Although it’s beyond the purview of this tutorial, you can learn more about scope, different variable types, and hoisting in our Understanding Variables, Scope, and Hoisting in JavaScript tutorial.


# Understanding the SyntaxError


When the JavaScript interpreter combs through your code, it may throw a SyntaxError when it comes across code that does not follow the language specifications. If this happens, your code will stop execution, and you’ll receive a message about your syntax.


## Missing Code Enclosure


For example, when you’re writing a function and forget a parenthesis, ), to enclose your code, you’ll receive a SyntaxError with a very specific message about what you’re missing:


```
function sammy(animal) {
    if(animal == 'shark'){
        return `I'm cool`;
    } else {
        return `You're cool`;
    }
}

sammy('shark';

```


```
OutputUncaught SyntaxError: missing ) after argument list

```


Fortunately, the error message specifies the missing element in the code. In this example, the sammy function call is missing the closing ) parenthesis:


```
. . .

sammy('shark');

```


Omitting an ending curly brace } at the end of a function, or a bracket ] in an array, will also throw this error. Make sure you’re closing your functions, arrays, and objects appropriately.


## Declaring the Same Variable Names


You may also encounter this error when using the same variable name as the function parameter and inside the function body. Take the following example:


```
function sammy(animal) {
    let animal = 'shark';
}

```


```
OutputVM132:2 Uncaught SyntaxError: Identifier 'animal' has already been declared

```


To fix the error, make sure to create unique and specific variable names within your function body. By declaring a new variable name, for instance, animalType, you remove the conflict between the function parameter and the let variable within the body:


```
function sammy(animal) {
    let animalType = 'shark';
}

```


If you intend to change or use the parameter within the function body, don’t use a variable declaration with the same variable name. For instance, you can remove the let declaration within the body:


```
function sammy(animal) {
    animal = 'shark';
}

```


Make sure, when working with variables inside and outside of the function body, that you name it uniquely. When accessing function parameters, you can use it within the function body without a variable declaration like let.


## Identifying Unexpected Tokens


Much like missing a bracket ] or a curly brace }, you may sometimes need a small, but crucial addition to your code. For example:


```
let sharkCount = 0;

function sammy() {
    sharkCount+;
    console.log(sharkCount);
}

```


```
OutputUncaught SyntaxError: Unexpected token ';'

```


The additional element in this example is the + plus sign after sharkCount inside the function body:


```
. . .
function sammy() {
    sharkCount++;
    console.log(sharkCount);
}

```


When you encounter a SyntaxError: Unexpected token, double check your code for missing or additional operators like the plus sign (+).


# Understanding the TypeError


TypeError occurs when the value of a function or a variable is of an unexpected type. You can learn more about the different JavaScript data types by reviewing our Understanding Data Types in JavaScript tutorial.


## Using Array Methods on Objects


A common mistake is using an array method to iterate over an object. For example, you cannot loop over an object with the .map() method because it is a method specific to an array:


```
const sharks = {
    shark1: 'sammy',
    shark2: 'shelly',
    shark3: 'sheldon'
}

sharks.map((shark) => `Hello there ${shark}!`);

```


```
OutputUncaught TypeError: sharks.map is not a function
    at <anonymous>:1:8

```


One option to fix the previous example is to use the for...in loop, which works for the object data type, on the sharks object to retrieve the values:


```
const sharks = {
    shark1: 'sammy',
    shark2: 'shelly',
    shark3: 'sheldon'
}

for (let key in sharks) {
    console.log(`Hello there ${sharks[key]}!`);
}

```


Alternatively, you can turn the sharks object, into an array to use the .map() method:


```
const sharks = ['sammy', 'shelly', 'sheldon'];

sharks.map((shark) => `Hello there ${shark}!`);

```


When you work different arrays and objects, it is easy to mistake the different methods. Double-check that your method is the appropriate one for the type of data you are working with.


## Using Correct Destructuring Methods


Likewise, trying to iterate over an object with array destructuring will throw a TypeError:


```
const shark = {
    name: 'sammy',
    age: 12,
    cloudPlatform: 'DigitalOcean'
}

const [name, age, cloudPlatform] = sharks;

```


```
OutputVM23:7 Uncaught TypeError: sharks is not iterable
    at <anonymous>:7:26

```


One way to fix this issue, is to use object destructuring to create new variables based on the object keys:


```
const shark = {
    name: 'sammy',
    age: 12,
    cloudPlatform: 'DigitalOcean'
}

const {name, age, cloudPlatform} = shark;

console.log(name);

```


```
Outputsammy

```


Depending on how you are structuring your data, whether with arrays or objects, make sure to use the appropriate methods to retrieve values.


# Conclusion


In this tutorial, you learned about three common JavaScript error types, some of their associated messages, and how to debug common issues when you encounter them. Though not exhaustive, you’ve gained insight into how JavaScript and the browser work together to indicate the problems in your code.


You can learn more about how to use JavaScript Object Methods and JavaScript Array Methods to better understand how to apply those methods.


You can also learn more about debugging with Chrome Dev tools with our How to Debug JavaScript with Google Chrome DevTools and Visual Studio Code tutorial.


