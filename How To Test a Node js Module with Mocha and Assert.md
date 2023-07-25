# How To Test a Node js Module with Mocha and Assert

```JavaScript``` ```Node.js``` ```Development```

The author selected the Open Internet/Free Speech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Testing is an integral part of software development. It’s common for programmers to run code that tests their application as they make changes in order to confirm it’s behaving as they’d like. With the right test setup, this process can even be automated, saving a lot of time. Running tests consistently after writing new code ensures that new changes don’t break pre-existing features. This gives the developer confidence in their code base, especially when it gets deployed to production so users can interact with it.


A test framework structures the way we create test cases. Mocha is a popular JavaScript test framework that organizes our test cases and runs them for us. However, Mocha does not verify our code’s behavior. To compare values in a test, we can use the Node.js assert module.


In this article, you’ll write tests for a Node.js TODO list module. You will set up and use the Mocha test framework to structure your tests. Then you’ll use the Node.js assert module to create the tests themselves. In this sense, you will be using Mocha as a plan builder, and assert to implement the plan.


# Prerequisites


- Node.js installed on your development machine. This tutorial uses Node.js version 10.16.0. To install this on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04.
- A basic knowledge of JavaScript, which you can find in our How To Code in JavaScript series.

# Step 1 — Writing a Node Module


Let’s begin this article by writing the Node.js module we’d like to test. This module will manage a list of TODO items. Using this module, we will be able to list all the TODOs that we are keeping track of, add new items, and mark some as complete. Additionally, we’ll be able to export a list of TODO items to a CSV file. If you’d like a refresher on writing Node.js modules, you can read our article on How To Create a Node.js Module.


First, we need to set up the coding environment. Create a folder with the name of your project in your terminal. This tutorial will use the name todos:


```
mkdir todos


```


Then enter that folder:


```
cd todos


```


Now initialize npm, since we’ll be using its CLI functionality to run the tests later:


```
npm init -y


```


We only have one dependency, Mocha, which we will use to organize and run our tests. To download and install Mocha, use the following:


```
npm i request --save-dev mocha


```


We install Mocha as a dev dependency, as it’s not required by the module in a production setting. If you would like to learn more about Node.js packages or npm, check out our guide on How To Use Node.js Modules with npm and package.json.


Finally, let’s create our file that will contain our module’s code:


```
touch index.js


```


With that, we’re ready to create our module. Open index.js in a text editor like nano:


```
nano index.js


```


Let’s begin by defining the Todos class. This class contains all the functions that we need to manage our TODO list. Add the following lines of code to index.js:


todos/index.js
```
class Todos {
    constructor() {
        this.todos = [];
    }
}

module.exports = Todos;

```


We begin the file by creating a Todos class. Its constructor() function takes no arguments, therefore we don’t need to provide any values to instantiate an object for this class. All we do when we initialize a Todos object is create a todos property that’s an empty array.


The modules line allows other Node.js modules to require our Todos class. Without explicitly exporting the class, the test file that we will create later would not be able to use it.


Let’s add a function to return the array of todos we have stored. Write in the following highlighted lines:


todos/index.js
```
class Todos {
    constructor() {
        this.todos = [];
    }
    
    list() {
        return [...this.todos];
    }
}

module.exports = Todos;

```


Our list() function returns a copy of the array that’s used by the class. It makes a copy of the array by using JavaScript’s destructuring syntax. We make a copy of the array so that changes the user makes to the array returned by list() does not affect the array used by the Todos object.



Note: JavaScript arrays are reference types. This means that for any variable assignment to an array or function invocation with an array as a parameter, JavaScript refers to the original array that was created. For example, if we have an array with three items called x, and create a new variable y such that y = x, y and x both refer to the same thing. Any changes we make to the array with y impacts variable x and vice versa.

Now let’s write the add() function, which adds a new TODO item:


todos/index.js
```
class Todos {
    constructor() {
        this.todos = [];
    }
    
    list() {
        return [...this.todos];
    }
    
    add(title) {
        let todo = {
            title: title,
            completed: false,
        }
        
        this.todos.push(todo);
    }
}

module.exports = Todos;

```


Our add() function takes a string, and places it in a new JavaScript object’s title property. The new object also has a completed property, which is set to false by default. We then add this new object to our array of TODOs.


Important functionality in a TODO manager is to mark items as completed. For this implementation, we will loop through our todos array to find the TODO item the user is searching for. If one is found, we’ll mark it as completed. If none is found, we’ll throw an error.


Add the complete() function like this:


todos/index.js
```
class Todos {
    constructor() {
        this.todos = [];
    }
    
    list() {
        return [...this.todos];
    }
    
    add(title) {
        let todo = {
            title: title,
            completed: false,
        }
        
        this.todos.push(todo);
    }
    
    complete(title) {
        let todoFound = false;
        this.todos.forEach((todo) => {
            if (todo.title === title) {
                todo.completed = true;
                todoFound = true;
                return;
            }
        });

        if (!todoFound) {
            throw new Error(`No TODO was found with the title: "${title}"`);
        }
    }
}

module.exports = Todos;

```


Save the file and exit from the text editor.


We now have a basic TODO manager that we can experiment with. Next, let’s manually test our code to see if the application is working.


# Step 2 — Manually Testing the Code


In this step, we will run our code’s functions and observe the output to ensure it matches our expectations. This is called manual testing. It’s likely the most common testing methodology programmers apply. Although we will automate our testing later with Mocha, we will first manually test our code to give a better sense of how manual testing differs from testing frameworks.


Let’s add two TODO items to our app and mark one as complete. Start the Node.js REPL in the same folder as the index.js file:


```
node


```


You will see the > prompt in the REPL that tells us we can enter JavaScript code. Type the following at the prompt:


```
const Todos = require('./index');


```


With require(), we load the TODOs module into a Todos variable. Recall that our module returns the Todos class by default.


Now, let’s instantiate an object for that class. In the REPL, add this line of code:


```
const todos = new Todos();


```


We can use the todos object to verify our implementation works. Let’s add our first TODO item:


```
todos.add("run code");


```


So far we have not seen any output in our terminal. Let’s verify that we’ve stored our "run code" TODO item by getting a list of all our TODOs:


```
todos.list();


```


You will see this output in your REPL:


```
Output[ { title: 'run code', completed: false } ]

```


This is the expected result: We have one TODO item in our array of TODOs, and it’s not completed by default.


Let’s add another TODO item:


```
todos.add("test everything");


```


Mark the first TODO item as completed:


```
todos.complete("run code");


```


Our todos object will now be managing two items: "run code" and "test everything". The "run code" TODO will be completed as well. Let’s confirm this by calling list() once again:


```
todos.list();


```


The REPL will output:


```
Output[
  { title: 'run code', completed: true },
  { title: 'test everything', completed: false }
]

```


Now, exit the REPL with the following:


```
.exit


```


We’ve confirmed that our module behaves as we expect it to. While we didn’t put our code in a test file or use a testing library, we did test our code manually. Unfortunately, this form of testing becomes time consuming to do every time we make a change. Next, let’s use automated testing in Node.js and see if we can solve this problem with the Mocha testing framework.


# Step 3 — Writing Your First Test with Mocha and Assert


In the last step, we manually tested our application. This will work for individual use cases, but as our module scales, this method becomes less viable. As we test new features, we must be certain that the added functionality has not created problems in the old functionality. We would like to test each feature over again for every change in the code, but doing this by hand would take a lot of effort and would be prone to error.


A more efficient practice would be to set up automated tests. These are scripted tests written like any other code block. We run our functions with defined inputs and inspect their effects to ensure they behave as we expect. As our codebase grows, so will our automated tests. When we write new tests alongside the features, we can verify the entire module still works—all without having to remember how to use each function every time.


In this tutorial, we’re using the Mocha testing framework with the Node.js assert module. Let’s get some hands-on experience to see how they work together.


To begin, create a new file to store our test code:


```
touch index.test.js


```


Now use your preferred text editor to open the test file. You can use nano like before:


```
nano index.test.js


```


In the first line of the text file, we will load the TODOs module like we did in the Node.js shell. We will then load the assert module for when we write our tests. Add the following lines:


todos/index.test.js
```
const Todos = require('./index');
const assert = require('assert').strict;

```


The strict property of the assert module will allow us to use special equality tests that are recommended by Node.js and are good for future-proofing, since they account for more use cases.


Before we go into writing tests, let’s discuss how Mocha organizes our code. Tests structured in Mocha usually follow this template:


```
describe([String with Test Group Name], function() {
    it([String with Test Name], function() {
        [Test Code]
    });
});

```


Notice two key functions: describe() and it(). The describe() function is used to group similar tests. It’s not required for Mocha to run tests, but grouping tests make our test code easier to maintain. It’s recommended that you group your tests in a way that’s easy for you to update similar ones together.


The it() contains our test code. This is where we would interact with our module’s functions and use the assert library. Many it() functions can be defined in a describe() function.


Our goal in this section is to use Mocha and assert to automate our manual test. We’ll do this step-by-step, beginning with our describe block. Add the following to your file after the module lines:


todos/index.test.js
```
...
describe("integration test", function() {
});

```


With this code block, we’ve created a grouping for our integrated tests. Unit tests would test one function at a time. Integration tests verify how well functions within or across modules work together. When Mocha runs our test, all the tests within that describe block will run under the "integration test" group.


Let’s add an it() function so we can begin testing our module’s code:


todos/index.test.js
```
...
describe("integration test", function() {
    it("should be able to add and complete TODOs", function() {
    });
});

```


Notice how descriptive we made the test’s name. If anyone runs our test, it will be immediately clear what’s passing or failing. A well-tested application is typically a well-documented application, and tests can sometimes be an effective kind of documentation.


For our first test, we will create a new Todos object and verify it has no items in it:


todos/index.test.js
```
...
describe("integration test", function() {
    it("should be able to add and complete TODOs", function() {
        let todos = new Todos();
        assert.notStrictEqual(todos.list().length, 1);
    });
});

```


The first new line of code instantiated a new Todos object as we would do in the Node.js REPL or another module. In the second new line, we use the assert module.


From the assert module we use the notStrictEqual() method. This function takes two parameters: the value that we want to test (called the actual value) and the value we expect to get (called the expected value). If both arguments are the same, notStrictEqual() throws an error to fail the test.


Save and exit from index.test.js.


The base case will be true as the length should be 0, which isn’t 1. Let’s confirm this by running Mocha. To do this, we need to modify our package.json file. Open your package.json file with your text editor:


```
nano package.json


```


Now, in your scripts property, change it so it looks like this:


todos/package.json
```
...
"scripts": {
    "test": "mocha index.test.js"
},
...

```


We have just changed the behavior of npm’s CLI test command. When we run npm test, npm will review the command we just entered in package.json. It will look for the Mocha library in our node_modules folder and run the mocha command with our test file.


Save and exit package.json.


Let’s see what happens when we run our test. In your terminal, enter:


```
npm test


```


The command will produce the following output:


```
Output> todos@1.0.0 test your_file_path/todos
> mocha index.test.js



integrated test
    ✓ should be able to add and complete TODOs


  1 passing (16ms)

```


This output first shows us which group of tests it is about to run. For every individual test within a group, the test case is indented. We see our test name as we described it in the it() function. The tick at the left side of the test case indicates that the test passed.


At the bottom, we get a summary of all our tests. In our case, our one test is passing and was completed in 16ms (the time varies from computer to computer).


Our testing has started with success. However, this current test case can allow for false-positives. A false-positive is a test case that passes when it should fail.


We currently check that the length of the array is not equal to 1. Let’s modify the test so that this condition holds true when it should not. Add the following lines to index.test.js:


todos/index.test.js
```
...
describe("integration test", function() {
    it("should be able to add and complete TODOs", function() {
        let todos = new Todos();
        todos.add("get up from bed");
        todos.add("make up bed");
        assert.notStrictEqual(todos.list().length, 1);
    });
});

```


Save and exit the file.


We added two TODO items. Let’s run the test to see what happens:


```
npm test


```


This will give the following:


```
Output...
integrated test
    ✓ should be able to add and complete TODOs


  1 passing (8ms)

```


This passes as expected, as the length is greater than 1. However, it defeats the original purpose of having that first test. The first test is meant to confirm that we start on a blank state. A better test will confirm that in all cases.


Let’s change the test so it only passes if we have absolutely no TODOs in store. Make the following changes to index.test.js:


todos/index.test.js
```
...
describe("integration test", function() {
    it("should be able to add and complete TODOs", function() {
        let todos = new Todos();
        todos.add("get up from bed");
        todos.add("make up bed");
        assert.strictEqual(todos.list().length, 0);
    });
});

```


You changed notStrictEqual() to strictEqual(), a function that checks for equality between its actual and expected argument. Strict equal will fail if our arguments are not exactly the same.


Save and exit, then run the test so we can see what happens:


```
npm test


```


This time, the output will show an error:


```
Output...
  integration test
    1) should be able to add and complete TODOs


  0 passing (16ms)
  1 failing

  1) integration test
       should be able to add and complete TODOs:

      AssertionError [ERR_ASSERTION]: Input A expected to strictly equal input B:
+ expected - actual

- 2
+ 0
      + expected - actual

      -2
      +0

      at Context.<anonymous> (index.test.js:9:10)



npm ERR! Test failed.  See above for more details.

```


This text will be useful for us to debug why the test failed. Notice that since the test failed there was no tick at the beginning of the test case.


Our test summary is no longer at the bottom of the output, but right after our list of test cases were displayed:


```
...
0 passing (29ms)
  1 failing
...

```


The remaining output provides us with data about our failing tests. First, we see what test case has failed:


```
...
1) integrated test
       should be able to add and complete TODOs:
...

```


Then, we see why our test failed:


```
...
      AssertionError [ERR_ASSERTION]: Input A expected to strictly equal input B:
+ expected - actual

- 2
+ 0
      + expected - actual

      -2
      +0

      at Context.<anonymous> (index.test.js:9:10)
...

```


An AssertionError is thrown when strictEqual() fails. We see that the expected value, 0, is different from the actual value, 2.


We then see the line in our test file where the code fails. In this case, it’s line 10.


Now, we’ve seen for ourselves that our test will fail if we expect incorrect values. Let’s change our test case back to its right value. First, open the file:


```
nano index.test.js 


```


Then take out the todos.add lines so that your code looks like the following:


todos/index.test.js
```
...
describe("integration test", function () {
    it("should be able to add and complete TODOs", function () {
        let todos = new Todos();
        assert.strictEqual(todos.list().length, 0);
    });
});

```


Save and exit the file.


Run it once more to confirm that it passes without any potential false-positives:


```
npm test


```


The output will be as follows:


```
Output...
integration test
    ✓ should be able to add and complete TODOs


  1 passing (15ms)

```


We’ve now improved our test’s resiliency quite a bit. Let’s move forward with our integration test. The next step is to add a new TODO item to index.test.js:


todos/index.test.js
```
...
describe("integration test", function() {
    it("should be able to add and complete TODOs", function() {
        let todos = new Todos();
        assert.strictEqual(todos.list().length, 0);
        
        todos.add("run code");
        assert.strictEqual(todos.list().length, 1);
        assert.deepStrictEqual(todos.list(), [{title: "run code", completed: false}]);
    });
});

```


After using the add() function, we confirm that we now have one TODO being managed by our todos object with strictEqual(). Our next test confirms the data in the todos with deepStrictEqual(). The deepStrictEqual() function recursively tests that our expected and actual objects have the same properties. In this case, it tests that the arrays we expect both have a JavaScript object within them. It then checks that their JavaScript objects have the same properties, that is, that both their title properties are "run code" and both their completed properties are false.


We then complete the remaining tests using these two equality checks as needed by adding the following highlighted lines:


todos/index.test.js
```
...
describe("integration test", function() {
    it("should be able to add and complete TODOs", function() {
        let todos = new Todos();
        assert.strictEqual(todos.list().length, 0);
        
        todos.add("run code");
        assert.strictEqual(todos.list().length, 1);
        assert.deepStrictEqual(todos.list(), [{title: "run code", completed: false}]);
    
        todos.add("test everything");
        assert.strictEqual(todos.list().length, 2);
        assert.deepStrictEqual(todos.list(),
            [
                { title: "run code", completed: false },
                { title: "test everything", completed: false }
            ]
        );
        
        todos.complete("run code");
        assert.deepStrictEqual(todos.list(),
            [
                { title: "run code", completed: true },
                { title: "test everything", completed: false }
            ]
    );
  });
});

```


Save and exit the file.


Our test now mimics our manual test. With these programmatic tests, we don’t need to check the output continuously if our tests pass when we run them. You typically want to test every aspect of use to make sure the code is tested properly.


Let’s run our test with npm test once more to get this familiar output:


```
Output...
integrated test
    ✓ should be able to add and complete TODOs


  1 passing (9ms)

```


You’ve now set up an integrated test with the Mocha framework and the assert library.


Let’s consider a situation where we’ve shared our module with some other developers and they’re now giving us feedback. A good portion of our users would like the complete() function to return an error if no TODOs were added as of yet. Let’s add this functionality in our complete() function.


Open index.js in your text editor:


```
nano index.js


```


Add the following to the function:


todos/index.js
```
...
complete(title) {
    if (this.todos.length === 0) {
        throw new Error("You have no TODOs stored. Why don't you add one first?");
    }

    let todoFound = false
    this.todos.forEach((todo) => {
        if (todo.title === title) {
            todo.completed = true;
            todoFound = true;
            return;
        }
    });

    if (!todoFound) {
        throw new Error(`No TODO was found with the title: "${title}"`);
    }
}
...

```


Save and exit the file.


Now let’s add a new test for this new feature. We want to verify that if we call complete on a Todos object that has no items, it will return our special error.


Go back into index.test.js:


```
nano index.test.js


```


At the end of the file, add the following code:


todos/index.test.js
```
...
describe("complete()", function() {
    it("should fail if there are no TODOs", function() {
        let todos = new Todos();
        const expectedError = new Error("You have no TODOs stored. Why don't you add one first?");
    
        assert.throws(() => {
            todos.complete("doesn't exist");
        }, expectedError);
    });
});

```


We use describe() and it() like before. Our test begins with creating a new todos object. We then define the error we are expecting to receive when we call the complete() function.


Next, we use the throws() function of the assert module. This function was created so we can verify the errors that are thrown in our code. Its first argument is a function that contains the code that throws the error. The second argument is the error we are expecting to receive.


In your terminal, run the tests with npm test once again and you will now see the following output:


```
Output...
integrated test
    ✓ should be able to add and complete TODOs

  complete()
    ✓ should fail if there are no TODOs


  2 passing (25ms)

```


This output highlights the benefit of why we do automated testing with Mocha and assert. Because our tests are scripted, every time we run npm test, we verify that all our tests are passing. We did not need to manually check if the other code is still working; we know that it is because the test we have still passed.


So far, our tests have verified the results of synchronous code. Let’s see how we would need to adapt our newfound testing habits to work with asynchronous code.


# Step 4 — Testing Asynchronous Code


One of the features we want in our TODO module is a CSV export feature. This will print all the TODOs we have in store along with the completed status to a file. This requires that we use the fs module—a built-in Node.js module for working with the file system.


Writing to a file is an asynchronous operation. There are many ways to write to a file in Node.js. We can use callbacks, Promises, or the async/await keywords. In this section, we’ll look at how we write tests for those different methods.


## Callbacks


A callback function is one used as an argument to an asynchronous function. It is called when the asynchronous operation is completed.


Let’s add a function to our Todos class called saveToFile(). This function will build a string by looping through all our TODO items and writing that string to a file.


Open your index.js file:


```
nano index.js


```


In this file, add the following highlighted code:


todos/index.js
```
const fs = require('fs');

class Todos {
    constructor() {
        this.todos = [];
    }

    list() {
        return [...this.todos];
    }
    
    add(title) {
        let todo = {
            title: title,
            completed: false,
        }
        this.todos.push(todo);
    }

    complete(title) {
        if (this.todos.length === 0) {
            throw new Error("You have no TODOs stored. Why don't you add one first?");
        }

        let todoFound = false
        this.todos.forEach((todo) => {
            if (todo.title === title) {
                todo.completed = true;
                todoFound = true;
                return;
            }
        });

        if (!todoFound) {
            throw new Error(`No TODO was found with the title: "${title}"`);
        }
    }

    saveToFile(callback) {
        let fileContents = 'Title,Completed\n';
        this.todos.forEach((todo) => {
            fileContents += `${todo.title},${todo.completed}\n`
        });

        fs.writeFile('todos.csv', fileContents, callback);
    }
}

module.exports = Todos;

```


We first have to import the fs module in our file. Then we added our new saveToFile() function. Our function takes a callback function that will be used once the file write operation is complete. In that function, we create a fileContents variable that stores the entire string we want to be saved as a file. It’s initialized with the CSV’s headers. We then loop through each TODO item with the internal array’s forEach() method. As we iterate, we add the title and completed properties of the individual todos objects.


Finally, we use the fs module to write the file with the writeFile() function. Our first argument is the file name: todos.csv. The second is the contents of the file, in this case, our fileContents variable. Our last argument is our callback function, which handles any file writing errors.


Save and exit the file.


Let’s now write a test for our saveToFile() function. Our test will do two things: confirm that the file exists in the first place, and then verify that it has the right contents.


Open the index.test.js file:


```
nano index.test.js


```


let’s begin by loading the fs module at the top of the file, as we’ll use it to help test our results:


todos/index.test.js
```
const Todos = require('./index');
const assert = require('assert').strict;
const fs = require('fs');
...

```


Now, at the end of the file let’s add our new test case:


todos/index.test.js
```
...
describe("saveToFile()", function() {
    it("should save a single TODO", function(done) {
        let todos = new Todos();
        todos.add("save a CSV");
        todos.saveToFile((err) => {
            assert.strictEqual(fs.existsSync('todos.csv'), true);
            let expectedFileContents = "Title,Completed\nsave a CSV,false\n";
            let content = fs.readFileSync("todos.csv").toString();
            assert.strictEqual(content, expectedFileContents);
            done(err);
        });
    });
});

```


Like before, we use describe() to group our test separately from the others as it involves new functionality. The it() function is slightly different from our other ones. Usually, the callback function we use has no arguments. This time, we have done as an argument. We need this argument when testing functions with callbacks. The done() callback function is used by Mocha to tell it when an asynchronous function is completed.


All callback functions being tested in Mocha must call the done() callback. If not, Mocha would never know when the function was complete and would be stuck waiting for a signal.


Continuing, we create our Todos instance and add a single item to it. We then call the saveToFile() function, with a callback that captures a file writing error. Note how our test for this function resides in the callback. If our test code was outside the callback, it would fail as long as the code was called before the file writing completed.


In our callback function, we first check that our file exists:


todos/index.test.js
```
...
assert.strictEqual(fs.existsSync('todos.csv'), true);
...

```


The fs.existsSync() function returns true if the file path in its argument exists, false otherwise.


<$>[note]
Note: The fs module’s functions are asynchronous by default. However, for key functions, they made synchronous counterparts. This test is simpler by using synchronous functions, as we don’t have to nest the asynchronous code to ensure it works. In the fs module, synchronous functions usually end with "Sync" at the end of their names.
<$>


We then create a variable to store our expected value:


todos/index.test.js
```
...
let expectedFileContents = "Title,Completed\nsave a CSV,false\n";
...

```


We use readFileSync() of the fs module to read the file synchronously:


todos/index.test.js
```
...
let content = fs.readFileSync("todos.csv").toString();
...

```


We now provide readFileSync() with the right path for the file: todos.csv. As readFileSync() returns a Buffer object, which stores binary data, we use its toString() method so we can compare its value with the string we expect to have saved.


Like before, we use the assert module’s strictEqual to do a comparison:


todos/index.test.js
```
...
assert.strictEqual(content, expectedFileContents);
...

```


We end our test by calling the done() callback, ensuring that Mocha knows to stop testing that case:


todos/index.test.js
```
...
done(err);
...

```


We provide the err object to done() so Mocha can fail the test in the case an error occurred.


Save and exit from index.test.js.


Let’s run this test with npm test like before. Your console will display this output:


```
Output...
integrated test
    ✓ should be able to add and complete TODOs

  complete()
    ✓ should fail if there are no TODOs

  saveToFile()
    ✓ should save a single TODO


  3 passing (15ms)

```


You’ve now tested your first asynchronous function with Mocha using callbacks. But at the time of writing this tutorial, Promises are more prevalent than callbacks in new Node.js code, as explained in our How To Write Asynchronous Code in Node.js article. Next, let’s learn how we can test them with Mocha as well.


## Promises


A Promise is a JavaScript object that will eventually return a value. When a Promise is successful, it is resolved. When it encounters an error, it is rejected.


Let’s modify the saveToFile() function so that it uses Promises instead of callbacks. Open up index.js:


```
nano index.js


```


First, we need to change how the fs module is loaded. In your index.js file, change the require() statement at the top of the file to look like this:


todos/index.js
```
...
const fs = require('fs').promises;
...

```


We just imported the fs module that uses Promises instead of callbacks. Now, we need to make some changes to saveToFile() so that it works with Promises instead.


In your text editor, make the following changes to the saveToFile() function to remove the callbacks:


todos/index.js
```
...
saveToFile() {
    let fileContents = 'Title,Completed\n';
    this.todos.forEach((todo) => {
        fileContents += `${todo.title},${todo.completed}\n`
    });

    return fs.writeFile('todos.csv', fileContents);
}
...

```


The first difference is that our function no longer accepts any arguments. With Promises we don’t need a callback function. The second change concerns how the file is written. We now return the result of the writeFile() promise.


Save and close out of index.js.


Let’s now adapt our test so that it works with Promises. Open up index.test.js:


```
nano index.test.js


```


Change the saveToFile() test to this:


todos/index.js
```
...
describe("saveToFile()", function() {
    it("should save a single TODO", function() {
        let todos = new Todos();
        todos.add("save a CSV");
        return todos.saveToFile().then(() => {
            assert.strictEqual(fs.existsSync('todos.csv'), true);
            let expectedFileContents = "Title,Completed\nsave a CSV,false\n";
            let content = fs.readFileSync("todos.csv").toString();
            assert.strictEqual(content, expectedFileContents);
        });
    });
});

```


The first change we need to make is to remove the done() callback from its arguments. If Mocha passes the done() argument, it needs to be called or it will throw an error like this:


```
1) saveToFile()
       should save a single TODO:
     Error: Timeout of 2000ms exceeded. For async tests and hooks, ensure "done()" is called; if returning a Promise, ensure it resolves. (/home/ubuntu/todos/index.test.js)
      at listOnTimeout (internal/timers.js:536:17)
      at processTimers (internal/timers.js:480:7)

```


When testing Promises, do not include the done() callback in it().


To test our promise, we need to put our assertion code in the then() function. Notice that we return this promise in the test, and we don’t have a catch() function to catch when the Promise is rejected.


We return the promise so that any errors that are thrown in the then() function are bubbled up to the it() function. If the errors are not bubbled up, Mocha will not fail the test case. When testing Promises, you need to use return on the Promise being tested. If not, you run the risk of getting a false-positive.


We also omit the catch() clause because Mocha can detect when a promise is rejected. If rejected, it automatically fails the test.


Now that we have our test in place, save and exit the file, then run Mocha with npm test and to confirm we get a successful result:


```
Output...
integrated test
    ✓ should be able to add and complete TODOs

  complete()
    ✓ should fail if there are no TODOs

  saveToFile()
    ✓ should save a single TODO


  3 passing (18ms)

```


We’ve changed our code and test to use Promises, and now we know for sure that it works. But the most recent asynchronous patterns use async/await keywords so we don’t have to create multiple then() functions to handle successful results. Let’s see how we can test with async/await.


## async/await


The async/await keywords make working with Promises less verbose. Once we define a function as asynchronous with the async keyword, we can get any future results in that function with the await keyword. This way we can use Promises without having to use the then() or catch() functions.


We can simplify our saveToFile() test that’s promise based with async/await. In your text editor, make these minor edits to the saveToFile() test in index.test.js:


todos/index.test.js
```
...
describe("saveToFile()", function() {
    it("should save a single TODO", async function() {
        let todos = new Todos();
        todos.add("save a CSV");
        await todos.saveToFile();

        assert.strictEqual(fs.existsSync('todos.csv'), true);
        let expectedFileContents = "Title,Completed\nsave a CSV,false\n";
        let content = fs.readFileSync("todos.csv").toString();
        assert.strictEqual(content, expectedFileContents);
    });
});

```


The first change is that the function used by the it() function now has the async keyword when it’s defined. This allows us to the use the await keyword inside its body.


The second change is found when we call saveToFile(). The await keyword is used before it is called. Now Node.js knows to wait until this function is resolved before continuing the test.


Our function code is easier to read now that we moved the code that was in the then() function to the it() function’s body. Running this code with npm test produces this output:


```
Output...
integrated test
    ✓ should be able to add and complete TODOs

  complete()
    ✓ should fail if there are no TODOs

  saveToFile()
    ✓ should save a single TODO


  3 passing (30ms)

```


We can now test asynchronous functions using any of three asynchronous paradigms appropriately.


We have covered a lot of ground with testing synchronous and asynchronous code with Mocha. Next, let’s dive in a bit deeper to some other functionality that Mocha offers to improve our testing experience, particularly how hooks can change test environments.


# Step 5 — Using Hooks to Improve Test Cases


Hooks are a useful feature of Mocha that allows us to configure the environment before and after a test. We typically add hooks within a describe() function block, as they contain setup and teardown logic specific to some test cases.


Mocha provides four hooks that we can use in our tests:


- before: This hook is run once before the first test begins.
- beforeEach: This hook is run before every test case.
- after: This hook is run once after the last test case is complete.
- afterEach: This hook is run after every test case.

When we test a function or feature multiple times, hooks come in handy as they allow us to separate the test’s setup code (like creating the todos object) from the test’s assertion code.


To see the value of hooks, let’s add more tests to our saveToFile() test block.


While we have confirmed that we can save our TODO items to a file, we only saved one item. Furthermore, the item was not marked as completed. Let’s add more tests to be sure that the various aspects of our module works.


First, let’s add a second test to confirm that our file is saved correctly when we have a completed a TODO item. Open your index.test.js file in your text editor:


```
nano index.test.js


```


Change the last test to the following:


todos/index.test.js
```
...
describe("saveToFile()", function () {
    it("should save a single TODO", async function () {
        let todos = new Todos();
        todos.add("save a CSV");
        await todos.saveToFile();

        assert.strictEqual(fs.existsSync('todos.csv'), true);
        let expectedFileContents = "Title,Completed\nsave a CSV,false\n";
        let content = fs.readFileSync("todos.csv").toString();
        assert.strictEqual(content, expectedFileContents);
    });

    it("should save a single TODO that's completed", async function () {
        let todos = new Todos();
        todos.add("save a CSV");
        todos.complete("save a CSV");
        await todos.saveToFile();

        assert.strictEqual(fs.existsSync('todos.csv'), true);
        let expectedFileContents = "Title,Completed\nsave a CSV,true\n";
        let content = fs.readFileSync("todos.csv").toString();
        assert.strictEqual(content, expectedFileContents);
    });
});

```


The test is similar to what we had before. The key differences are that we call the complete() function before we call saveToFile(), and that our expectedFileContents now have true instead of false for the completed column’s value.


Save and exit the file.


Let’s run our new test, and all the others, with npm test:


```
npm test


```


This will give the following:


```
Output...
integrated test
    ✓ should be able to add and complete TODOs

  complete()
    ✓ should fail if there are no TODOs

  saveToFile()
    ✓ should save a single TODO
    ✓ should save a single TODO that's completed


  4 passing (26ms)

```


It works as expected. There is, however, room for improvement. They both have to instantiate a Todos object at the beginning of the test. As we add more test cases, this quickly becomes repetitive and memory-wasteful. Also, each time we run the test, it creates a file. This can be mistaken for real output by someone less familiar with the module. It would be nice if we cleaned up our output files after testing.


Let’s make these improvements using test hooks. We’ll use the beforeEach() hook to set up our test fixture of TODO items. A test fixture is any consistent state used in a test. In our case, our test fixture is a new todos object that has one TODO item added to it already. We will then use afterEach() to remove the file created by the test.


In index.test.js, make the following changes to your last test for saveToFile():


todos/index.test.js
```
...
describe("saveToFile()", function () {
    beforeEach(function () {
        this.todos = new Todos();
        this.todos.add("save a CSV");
    });

    afterEach(function () {
        if (fs.existsSync("todos.csv")) {
            fs.unlinkSync("todos.csv");
        }
    });

    it("should save a single TODO without error", async function () {
        await this.todos.saveToFile();

        assert.strictEqual(fs.existsSync("todos.csv"), true);
        let expectedFileContents = "Title,Completed\nsave a CSV,false\n";
        let content = fs.readFileSync("todos.csv").toString();
        assert.strictEqual(content, expectedFileContents);
    });

    it("should save a single TODO that's completed", async function () {
        this.todos.complete("save a CSV");
        await this.todos.saveToFile();

        assert.strictEqual(fs.existsSync('todos.csv'), true);
        let expectedFileContents = "Title,Completed\nsave a CSV,true\n";
        let content = fs.readFileSync("todos.csv").toString();
        assert.strictEqual(content, expectedFileContents);
    });
});

```


Let’s break down all the changes we’ve made. We added a beforeEach() block to the test block:


todos/index.test.js
```
...
beforeEach(function () {
    this.todos = new Todos();
    this.todos.add("save a CSV");
});
...

```


These two lines of code create a new Todos object that will be available in each of our tests. With Mocha, the this object in beforeEach() refers to the same this object in it(). this is the same for every code block inside the describe() block. For more information on this, see our tutorial Understanding This, Bind, Call, and Apply in JavaScript.


This powerful context sharing is why we can quickly create test fixtures that work for both of our tests.


We then clean up our CSV file in the afterEach() function:


todos/index.test.js
```
...
afterEach(function () {
    if (fs.existsSync("todos.csv")) {
        fs.unlinkSync("todos.csv");
    }
});
...

```


If our test failed, then it may not have created a file. That’s why we check if the file exists before we use the unlinkSync() function to delete it.


The remaining changes switch the reference from todos, which were previously created in the it() function, to this.todos which is available in the Mocha context. We also deleted the lines that previously instantiated todos in the individual test cases.


Now, let’s run this file to confirm our tests still work. Enter npm test in your terminal to get:


```
Output...
integrated test
    ✓ should be able to add and complete TODOs

  complete()
    ✓ should fail if there are no TODOs

  saveToFile()
    ✓ should save a single TODO without error
    ✓ should save a single TODO that's completed


  4 passing (20ms)

```


The results are the same, and as a benefit, we have slightly reduced the setup time for new tests for the saveToFile() function and found a solution to the residual CSV file.


# Conclusion


In this tutorial, you wrote a Node.js module to manage TODO items and tested the code manually using the Node.js REPL. You then created a test file and used the Mocha framework to run automated tests. With the assert module, you were able to verify that your code works. You also tested synchronous and asynchronous functions with Mocha. Finally, you created hooks with Mocha that make writing multiple related test cases much more readable and maintainable.


Equipped with this understanding, challenge yourself to write tests for new Node.js modules that you are creating. Can you think about the inputs and outputs of your function and write your test before you write your code?


If you would like more information about the Mocha testing framework, check out the official Mocha documentation. If you’d like to continue learning Node.js, you can return to the How To Code in Node.js series page.


