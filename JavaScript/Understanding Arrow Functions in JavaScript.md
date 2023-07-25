# Understanding Arrow Functions in JavaScript

```JavaScript``` ```Development```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


The 2015 edition of the ECMAScript specification (ES6) added arrow function expressions to the JavaScript language. Arrow functions are a new way to write anonymous function expressions, and are similar to lambda functions in some other programming languages, such as Python.


Arrow functions differ from traditional functions in a number of ways, including the way their scope is determined and how their syntax is expressed. Because of this, arrow functions are particularly useful when passing a function as a parameter to a higher-order function, such as when you are looping over an array with built-in iterator methods. Their syntactic abbreviation can also allow you to improve the readability of your code.


In this article, you will review function declarations and expressions, learn about the differences between traditional function expressions and arrow function expressions, learn about lexical scope as it pertains to arrow functions, and explore some of the syntactic shorthand permitted with arrow functions.


# Defining Functions


Before delving into the specifics of arrow function expressions, this tutorial will briefly review traditional JavaScript functions in order to better show the unique aspects of arrow functions later on.


The How To Define Functions in JavaScript tutorial earlier in this series introduced the concept of function declarations and function expressions. A function declaration is a named function written with the function keyword. Function declarations load into the execution context before any code runs. This is known as hoisting, meaning you can use the function before you declare it.


Here is an example of a sum function that returns the sum of two parameters:


```
function sum(a, b) {
  return a + b
}

```


You can execute the sum function before declaring the function due to hoisting:


```
sum(1, 2)

function sum(a, b) {
  return a + b
}

```


Running this code would give the following output:


```
Output3

```


You can find the name of the function by logging the function itself:


```
console.log(sum)

```


This will return the function, along with its name:


```
Outputƒ sum(a, b) {
  return a + b
}

```


A function expression is a function that is not pre-loaded into the execution context, and only runs when the code encounters it. Function expressions are usually assigned to a variable, and can be anonymous, meaning the function has no name.


In this example, write the same sum function as an anonymous function expression:


```
const sum = function (a, b) {
  return a + b
}

```


You’ve now assigned the anonymous function to the sum constant. Attempting to execute the function before it is declared will result in an error:


```
sum(1, 2)

const sum = function (a, b) {
  return a + b
}

```


Running this will give:


```
OutputUncaught ReferenceError: Cannot access 'sum' before initialization

```


Also, note that the function does not have a named identifier. To illustrate this, write the same anonymous function assigned to sum, then log sum to the console:


```
const sum = function (a, b) {
  return a + b
}

console.log(sum)

```


This will show you the following:


```
Outputƒ (a, b) {
  return a + b
}

```


The value of sum is an anonymous function, not a named function.


You can name function expressions written with the function keyword, but this is not popular in practice. One reason you might want to name a function expression is to make error stack traces easier to debug.


Consider the following function, which uses an if statement to throw an error if the function parameters are missing:


```
const sum = function namedSumFunction(a, b) {
  if (!a || !b) throw new Error('Parameters are required.')

  return a + b
}

sum();

```


The highlighted section gives the function a name, and then the function uses the or || operator to throw an error object if either of the parameters is missing.


Running this code will give you the following:


```
OutputUncaught Error: Parameters are required.
    at namedSumFunction (<anonymous>:3:23)
    at <anonymous>:1:1

```


In this case, naming the function gives you a quick idea of where the error is.


An arrow function expression is an anonymous function expression written with the “fat arrow” syntax (=>).


Rewrite the sum function with arrow function syntax:


```
const sum = (a, b) => {
  return a + b
}

```


Like traditional function expressions, arrow functions are not hoisted, and so you cannot call them before you declare them. They are also always anonymous—there is no way to name an arrow function. In the next section, you will explore more of the syntactical and practical differences between arrow functions and traditional functions.


# Arrow Function Behavior and Syntax


Arrow functions have a few important distinctions in how they work that distinguish them from traditional functions, as well as a few syntactic enhancements. The biggest functional differences are that arrow functions do not have their own this binding or prototype and cannot be used as a constructor. Arrow functions can also be written as a more compact alternative to traditional functions, as they grant the ability to omit parentheses around parameters and add the concept of a concise function body with implicit return.


In this section, you will go through examples that illustrate each of these cases.


## Lexical this


The keyword this is often considered a tricky topic in JavaScript. The article Understanding This, Bind, Call, and Apply in JavaScript explains how this works, and how this can be implicitly inferred based on whether the program uses it in the global context, as a method within an object, as a constructor on a function or class, or as a DOM event handler.


Arrow functions have lexical this, meaning the value of this is determined by the surrounding scope (the lexical environment).


The next example will demonstrate the difference between how traditional and arrow functions handle this. In the following printNumbers object, there are two properties: phrase and numbers. There is also a method on the object, loop, which should print the phrase string and the current value in numbers:


```
const printNumbers = {
  phrase: 'The current value is:',
  numbers: [1, 2, 3, 4],

  loop() {
    this.numbers.forEach(function (number) {
      console.log(this.phrase, number)
    })
  },
}

```


One might expect the loop function to print the string and current number in the loop on each iteraton. However, in the result of running the function the phrase is actually undefined:


```
printNumbers.loop()

```


This will give the following:


```
Outputundefined 1
undefined 2
undefined 3
undefined 4

```


As this shows, this.phrase is undefined, indicating that this within the anonymous function passed into the forEach method does not refer to the printNumbers object. This is because a traditional function will not determine its this value from the scope of the environment, which is the printNumbers object.


In older versions of JavaScript, you would have had to use the bind method, which explicitly sets this. This pattern can be found often in some earlier versions of frameworks, like React, before the advent of ES6.


Use bind to fix the function:


```
const printNumbers = {
  phrase: 'The current value is:',
  numbers: [1, 2, 3, 4],

  loop() {
    // Bind the `this` from printNumbers to the inner forEach function
    this.numbers.forEach(
      function (number) {
        console.log(this.phrase, number)
      }.bind(this),
    )
  },
}

printNumbers.loop()

```


This will give the expected result:


```
OutputThe current value is: 1
The current value is: 2
The current value is: 3
The current value is: 4

```


Arrow functions provide a more direct way of dealing with this. Since their this value is determined based on the lexical scope, the inner function called in forEach can now access the properties of the outer printNumbers object, as demonstrated:


```
const printNumbers = {
  phrase: 'The current value is:',
  numbers: [1, 2, 3, 4],

  loop() {
    this.numbers.forEach((number) => {
      console.log(this.phrase, number)
    })
  },
}

printNumbers.loop()

```


This will give the expected result:


```
OutputThe current value is: 1
The current value is: 2
The current value is: 3
The current value is: 4

```


These examples establish that using arrow functions in built-in array methods like forEach, map, filter, and reduce can be more intuitive and easier to read, making this strategy more likely to fulfill expectations.


## Arrow Functions as Object Methods


While arrow functions are excellent as parameter functions passed into array methods, they are not effective as object methods because of the way they use lexical scoping for this. Using the same example as before, take the loop method and turn it into an arrow function to discover how it will execute:


```
const printNumbers = {
  phrase: 'The current value is:',
  numbers: [1, 2, 3, 4],

  loop: () => {
    this.numbers.forEach((number) => {
      console.log(this.phrase, number)
    })
  },
}

```


In this case of an object method, this should refer to properties and methods of the printNumbers object. However, since an object does not create a new lexical scope, an arrow function will look beyond the object for the value of this.


Call the loop() method:


```
printNumbers.loop()

```


This will give the following:


```
OutputUncaught TypeError: Cannot read property 'forEach' of undefined

```


Since the object does not create a lexical scope, the arrow function method looks for this in the outer scope–Window in this example. Since the numbers property does not exist on the Window object, it throws an error. As a general rule, it is safer to use traditional functions as object methods by default.


## Arrow Functions Have No constructor or prototype


The Understanding Prototypes and Inheritance in JavaScript tutorial earlier in this series explained that functions and classes have a prototype property, which is what JavaScript uses as a blueprint for cloning and inheritance.


To illustrate this, create a function and log the automatically assigned prototype property:


```
function myFunction() {
  this.value = 5
}

// Log the prototype property of myFunction
console.log(myFunction.prototype)

```


This will print the following to the console:


```
Output{constructor: ƒ}

```


This shows that in the prototype property there is an object with a constructor. This allows you to use the new keyword to create an instance of the function:


```
const instance = new myFunction()

console.log(instance.value)

```


This will yield the value of the value property that you defined when you first declared the function:


```
Output5

```


In contrast, arrow functions do not have a prototype property. Create a new arrow function and try to log its prototype:


```
const myArrowFunction = () => {}

// Attempt to log the prototype property of myArrowFunction
console.log(myArrowFunction.prototype)

```


This will give the following:


```
Outputundefined

```


As a result of the missing prototype property, the new keyword is not available and you cannot construct an instance from the arrow function:


```
const arrowInstance = new myArrowFunction()

console.log(arrowInstance)

```


This will give the following error:


```
OutputUncaught TypeError: myArrowFunction is not a constructor

```


This is consistent with our earlier example: Since arrow functions do not have their own this value, it follows that you would be unable to use an arrow function as a constructor.


As shown here, arrow functions have a lot of subtle changes that make them operate differently from traditional functions in ES5 and earlier. There have also been a few optional syntactical changes that make writing arrow functions quicker and less verbose. The next section will show examples of these syntax changes.


## Implicit Return


The body of a traditional function is contained within a block using curly brackets {} and ends when the code encounters a return keyword. The following is what this implementation looks like as an arrow function:


```
const sum = (a, b) => {
  return a + b
}

```


Arrow functions introduce concise body syntax, or implicit return. This allows the omission of the curly brackets and the return keyword.


```
const sum = (a, b) => a + b

```


Implicit return is useful for creating succinct one-line operations in map, filter, and other common array methods. Note that both the brackets and the return keyword must be omitted. If you cannot write the body as a one-line return statement, then you will have to use the normal block body syntax.


In the case of returning an object, syntax requires that you wrap the object literal in parentheses. Otherwise, the brackets will be treated as a function body and will not compute a return value.


To illustrate this, find the following example:


```
const sum = (a, b) => ({result: a + b})

sum(1, 2)

```


This will give the following output:


```
Output{result: 3}

```


## Omitting Parentheses Around a Single Parameter


Another useful syntactic enhancement is the ability to remove parentheses from around a single parameter in a function. In the following example, the square function only operates on one parameter, x:


```
const square = (x) => x * x

```


As a result, you can omit the parentheses around the parameter, and it will work just the same:


```
const square = x => x * x

square(10)

```


This will give the following:


```
Output100

```


Note that if a function takes no parameters, parentheses will be required:


```
const greet = () => 'Hello!'

greet()

```


Calling greet() will work as follows:


```
Output'Hello!'

```


Some codebases choose to omit parentheses wherever possible, and others choose to always keep parentheses around parameters no matter what, particularly in codebases that use TypeScript and require more information about each variable and parameter. When deciding how to write your arrow functions, check the style guide of the project to which you are contributing.


# Conclusion


In this article, you reviewed traditional functions and the difference between function declarations and function expressions. You learned that arrow functions are always anonymous, do not have a prototype or constructor, cannot be used with the new keyword, and determine the value of this through lexical scope. Finally, you explored the new syntactic enhancements available to arrow functions, such as implicit return and parentheses omission for single parameter functions.


For a review of basic functions, read How To Define Functions in JavaScript. To read more about the concept of scope and hoisting in JavaScript, read Understanding Variables, Scope, and Hoisting in JavaScript.


