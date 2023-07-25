# How To Run TypeScript Scripts with ts-node

```TypeScript```

## Introduction


TypeScript has been gaining in popularity. With TypeScript being a superset of JavaScript, using it means compiling your TypeScript files down to pure JavaScript before the V8 engine can understand them. You could watch for file changes and automate the compiling. But sometimes, you just want to run your script and get results. This is where ts-node comes in. With ts-node, you can skip the fuss and execute your TypeScript scripts with ease.


# Prerequisites


To successfully complete this tutorial, you will need the following:


- The latest version of Node installed on your machine. You can accomplish this by following the How to Install Node.js and Create a Local Development Environment tutorial
- Familiarity with npm. To learn more about working with npm, read this How To Use Node.js Modules with npm and package.json tutorial
- Familiarity with TypeScript. This How To Set Up a New TypeScript Project article is a great place to start.

# Step 1 — Getting Started


To get things started, you need to install typescript and ts-node:


```
npm install typescript ts-node


```


Since ts-node is an executable you can run, there’s nothing to import or require in your scripts.


If you don’t already have a TypeScript project to work with, you can just grab use this script to test ts-node with:


Script: reptile.ts
```
class Reptile {
  private reptiles: Array<string> = [
    'Alligator',
    'Crocodile',
    'Chameleon',
    'Komodo Dragon',
    'Iguana',
    'Salamander',
    'Snake',
    'Lizard',
    'Python',
    'Tortoise',
    'Turtle',
  ];

  shuffle(): void {
    for (let i = this.reptiles.length - 1; i > 0; i--) {
      let j: number = Math.floor(Math.random() * (i + 1));
      let temp: string = this.reptiles[i];
      this.reptiles[i] = this.reptiles[j];
      this.reptiles[j] = temp;
    }
  }

  random(count: number = 1, allowDupes?: boolean): Array<string> {
    let selected: Array<string> = [];

    if (!allowDupes && count > this.reptiles.length) {
      throw new Error(`Can't ensure no dupes for that count`);
    }

    for (let i: number = 0; i < count; i++) {
      if (allowDupes) {
        // Dupes are cool, so let's just pull random reptiles
        selected.push(this.reptiles[
          Math.floor(Math.random() * this.reptiles.length)
        ]);
      } else {
        // Dupes are no go, shuffle the array and grab a few
        this.shuffle();
        selected = this.reptiles.slice(0, count);
      }
    }

    return selected;
  }
}

const reptile = new Reptile();
console.log(`With Dupes: ${reptile.random(10, true)}`);
console.log(`And Without: ${reptile.random(10)}`);

```


The above script pulls random values from an array and returns two lists of different types of reptiles: one with duplicates and one without. Again, you can use your own script if you prefer.


Ensure that your script returns a value and prints something to the console. You will want to see the results of your code. With your TypeScript script in place, you can now move on to running your script.


# Step 2 — Running Scripts


Before you usets-node, it’s good practice to know what happens when you run a TypeScript script with Node.


You will run the reptile.ts script (or your own TypeScript script) using the node command:


```
node reptile.ts


```


After running this command, you will see an error message:


```
OutputSyntaxError: Unexpected identifier

```


The SyntaxError: Unexpected identifier message specifically points out the private class variable on the second line of reptile.ts.


Using Node to run TypeScript scripts returns an error. Now that you know what not to do and what to expect when you do it. This is where ts-node comes in.


Use ts-node to run the reptile.ts script:


```
npx ts-node reptile.ts


```


If you would like to know more about the npx command, it’s a tool that now comes with npm that allows you to run binaries that are local to the project from the command line.


You will see this output from running this script:


```
OutputWith Dupes: Komodo Dragon,Python,Tortoise,Iguana,Turtle,Salamander,Python,Python,Salamander,Snake
And Without: Alligator,Iguana,Lizard,Snake,Tortoise,Chameleon,Crocodile,Komodo Dragon,Turtle,Salamander

```


Running reptile.ts with ts-node will return two lists of types of reptiles, one potentially having duplicates and one without. The ts-node command efficiently runs TypeScript scripts. But there is a way to make it even faster.


# Step 3 — Speeding Things Up


Under the hood, ts-node takes your script, does some semantic checking to ensure your code is error-free, and then compiles your TypeScript into JavaScript.


This is the safest option. But if you’re not worried about TypeScript errors, you can pass in the -T or --transpileOnly flag. This flag tells ts-node to transpile down to JavaScript without checking for any TypeScript errors.


While it’s not always advisable to use this flag, there are scenarios where it makes sense. If you’re trying to run someone else’s script or if you’re confident that your editor and linter are catching everything, using the -T or --transpileOnly flag is appropriate.


```
npx ts-node -T reptile.ts


```


Running this command will give you the same output as npx ts-node reptile.ts. There is still more ts-node can do. This command also offers a TypeScript REPL.


# Step 4 — Using the TypeScript REPL


Another added bonus to ts-node is being able to use a TypeScript REPL (read-evaluate-print loop) similar to running node without any options.


This TypeScript REPL allows you to write TypeScript right on the command-line and is super handy for quickly testing something out.


To access the TypeScript REPL, run ts-node without any arguments:


```
npx ts-node


```


And now you can enjoy all of the strictness that TypeScript has to offer, right in your favorite terminal!


# Conclusion


In this article, you used ts-node to run TypeScript scripts. You also used the TypeScript REPL with ts-node.


If you’re interested in moving further with TypeScript, you may find this How To Work With TypeScript in Visual Studio Code article useful.


