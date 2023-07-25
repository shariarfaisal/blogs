# How To Use Modules in TypeScript

```Development``` ```TypeScript``` ```JavaScript```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Modules are a way to organize your code into smaller, more manageable pieces, allowing programs to import code from different parts of the application. There have been a few strategies to implementing modularity into JavaScript code over the years. TypeScript evolved along with ECMAScript specifications to provide a standard module system for JavaScript programs that has the flexibility to work with these different formats. TypeScript offers support for creating and using modules with a unified syntax that is similar to the ES Module syntax, while allowing the developer to output code that targets different module loaders, like Node.js (CommonJS), require.js (AMD), UMD, SystemJS, or ECMAScript 2015 native modules (ES6).


In this tutorial, you will create and use modules in TypeScript. You will follow different code samples in your own TypeScript environment, showing how to use the import and export keyword, how to set default exports, and how to make files with overwritten exports objects compatible with your code.


# Prerequisites


To follow this tutorial, you will need:


- An environment in which you can execute TypeScript programs to follow along with the examples. To set this up on your local machine, you will need the following:

Both Node and npm (or yarn) installed in order to run a development environment that handles TypeScript-related packages. This tutorial was tested with Node.js version 14.3.0 and npm version 6.14.5. To install on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04. This also works if you are using the Windows Subsystem for Linux (WSL).


- Both Node and npm (or yarn) installed in order to run a development environment that handles TypeScript-related packages. This tutorial was tested with Node.js version 14.3.0 and npm version 6.14.5. To install on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04. This also works if you are using the Windows Subsystem for Linux (WSL).
- You will need sufficient knowledge of JavaScript, especially ES6+ syntax, such as destructuring, rest operators, and imports/exports. If you need more information on these topics, reading our How To Code in JavaScript series is recommended.
- This tutorial will reference aspects of text editors that support TypeScript and show in-line errors. This is not necessary to use TypeScript but does take more advantage of TypeScript features. To gain the benefit of these, you can use a text editor like Visual Studio Code, which has full support for TypeScript out of the box.

All examples shown in this tutorial were created using TypeScript version 4.2.2.


# Setting Up the Project


In this step, you will create a sample project that contains two small classes for handling vector operations: Vector2 and Vector3. A vector in this case refers to a mathematical measurement of magnitude and distance, often used in visual graphics programs.


The classes you build will have a single operation on each: vector addition. Later on, you will use these sample classes to test out the importing and exporting of code from one program to another.


First, make yourself a directory that will house your sample code:


```
mkdir vector_project


```


Once the directory is created, make it your working directory:


```
cd vector_project


```


Now that you are at the root of your project, create your Node.js app with npm:


```
npm init


```


This will create a package.json file for your project.


Next, add TypeScript as a development dependency:


```
npm install typescript@4.2.2 --save-dev


```


This will install TypeScript to your project, with the TypeScript Compiler set to its default settings. To make your own custom settings, you will need to create a specific configuration file.


Create and open a file named tsconfig.json in the root of your project. To have your project work with the exercises in this tutorial, add the following contents to the file:


tsconfig.json
```
{
  "compilerOptions": {
    "target": "ES6",
    "module": "CommonJS",
    "outDir": "./out",
    "rootDir": "./src",
    "strict": true
  }
}

```


In this code, you are setting multiple configurations for the TypeScript Compiler. "target": "ES6" determines the environment that your code will be compiled for, and "outDir": "./out" and "rootDir": "./src" specify which directories will hold the output and the input for your compiler, respectively. "strict": true sets a level of strong type checking. Finally, "module": "CommonJS" specifies the module system as CommonJS. You will use this to simulate working with a Node.js application.


With your project set up, you can now move on to creating modules with basic syntax.


# Creating Modules in TypeScript with export


In this section, you will create modules in TypeScript using the TypeScript module syntax.


By default, files in TypeScript are treated as global scripts. This means that any variable, class, function, or other construct declared in the file is available globally. As soon as you start using modules in a file, this file becomes module-scoped, and is no longer executed globally.


To show this in action, you will create your first class: Vector2. Make a new directory called src/ in the root of the project:


```
mkdir src


```


This is the directory that you set as the root directory (rootDir) in your tsconfig.json file.


Inside this folder, create a new file called vector2.ts. Open this file in your favorite text editor, then write your Vector2 class:


vector_project/src/vector2.ts
```
class Vector2 {
  constructor(public x: number, public y: number) {}

  add(otherVector2: Vector2) {
    return new Vector2(this.x + otherVector2.x, this.y + otherVector2.y);
  }
}

```


In this code, you are declaring a class named Vector2, which is created by passing two numbers as parameters, set as the properties x and y. This class has one method that adds a vector to itself by combining the respective x and y values.


As your file is currently not using modules, your Vector2 is globally scoped. To turn your file into a module, you just have to export your Vector2 class:


vector_project/src/vector2.ts
```
export class Vector2 {
  constructor(public x: number, public y: number) {}

  add(otherVector2: Vector2) {
    return new Vector2(this.x + otherVector2.x, this.y + otherVector2.y);
  }
}

```


The file src/vector2.ts is now a module that has a single export: the Vector2 class. Save and close your file.


Next, you can create your Vector3 class. Create the vector3.ts file inside the src/ directory, then open the file in your favorite text editor and write the following code:


vector_project/src/vector3.ts
```
export class Vector3 {
  constructor(public x: number, public y: number, public z: number) {}

  add(otherVector3: Vector3) {
    return new Vector3(
      this.x + otherVector3.x,
      this.y + otherVector3.y,
      this.z + otherVector3.z
    );
  }
}


```


This code creates a class similar to Vector2, but with an extra z property that stores the vector in three dimensions. Notice that the export keyword has already made this file its own module. Save and close this file.


Now you have two files, vector2.ts and vector3.ts, that both are modules. Each file has a single export, which is the class for the vector they represent. In the next section, you will bring these modules into other files with import statements.


# Using Modules in TypeScript with import


In the previous section, you saw how to create modules. In this section, you will import these modules to use elsewhere in your code.


A common scenario when working with modules in TypeScript is to have a single file that collects multiple modules and re-exports them as one module. To demonstrate this, create a file called vectors.ts inside your src/ directory, then open this file in your favorite editor and write the following:


vector_project/src/vectors.ts
```
import { Vector2 } from "./vector2";
import { Vector3 } from "./vector3";

```


To import another module available in your project, you use the relative path to the file in an import statement. In this case, you are importing both modules from ./vector2 and ./vector3, which are the relative paths from the current file to the files src/vector2.ts and src/vector3.ts.


Now that you’ve imported the vectors, you can re-export them in a single module with the following highlighted syntax:


vector_project/src/vectors.ts
```
import { Vector2 } from "./vector2";
import { Vector3 } from "./vector3";

export { Vector2, Vector3 };

```


The export {} syntax allows you to export multiple identifiers. In this case, you are exporting the Vector2 and Vector3 classes using a single export declaration.


You could also use two separate export statements, like this:


vector_project/src/vectors.ts
```
import { Vector2 } from "./vector2";
import { Vector3 } from "./vector3";

export { Vector2 };
export { Vector3 };

```


This has the same meaning as the previous code snippet.


Since your src/vectors.ts is only importing two classes to later re-export them, you can use an even briefer form of the syntax:


vector_project/src/vectors.ts
```
export { Vector2 } from "./vector2";
export { Vector3 } from "./vector3";

```


The import statement is implicit here, and the TypeScript Compiler will include it automatically. Then the files will be immediately exported with the same name. Save this file.


Now that you are exporting your two vector classes from the src/vectors.ts file, create a new file called src/index.ts, then open the file and write the following code:


vector_project/src/index.ts
```
import { Vector2, Vector3 } from "./vectors";

const vec2a = new Vector2(1, 2);
const vec2b = new Vector2(2, 1);

console.log(vec2a.add(vec2b));

const vec3a = new Vector3(1, 2, 3);
const vec3b = new Vector3(3, 2, 1);

console.log(vec3a.add(vec3b));

```


In this code, you are importing both vector classes from the src/vectors.ts file, which uses the relative path ./vectors. You are then creating a few vector instances, using the add method to add them together, then logging the results.


When importing named exports, you can also use a different alias, which can be helpful to avoid naming collisions inside a file. To try this out, make the following highlighted changes to your file:


vector_project/src/index.ts
```
import { Vector2 as Vec2, Vector3 as Vec3 } from "./vectors";

const vec2a = new Vec2(1, 2);
const vec2b = new Vec2(2, 1);

console.log(vec2a.add(vec2b));

const vec3a = new Vec3(1, 2, 3);
const vec3b = new Vec3(3, 2, 1);

console.log(vec3a.add(vec3b));

```


Here you are using the as keyword to set the aliases Vec2 and Vec3 for the imported classes.


Notice how you are importing everything available inside ./vectors. This file is only exporting those two classes, so you could use the following syntax to import everything into a single variable:


vector_project/src/index.ts
```
import * as vectors from "./vectors";

const vec2a = new vectors.Vector2(1, 2);
const vec2b = new vectors.Vector2(2, 1);

console.log(vec2a.add(vec2b));

const vec3a = new vectors.Vector3(1, 2, 3);
const vec3b = new vectors.Vector3(3, 2, 1);

console.log(vec3a.add(vec3b));

```


In the code highlighted above, you use the import * as syntax to import everything that is exported by a module into a single variable. You also had to change the way you were using the Vector2 and Vector3 classes, as they are now available inside the vectors object, which is created during the import.


If you save the file and compile the project using tsc:


```
npx tsc


```


The TypeScript Compiler is going to create the out/ directory (given the compileOptions.outDir option you set in the tsconfig.json file) then populate the directory with JavaScript files.


Open the compiled file available at out/index.js in your favorite text editor. It will look like this:


vector_project/out/index.js
```
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
const vectors = require("./vectors");
const vec2a = new vectors.Vector2(1, 2);
const vec2b = new vectors.Vector2(2, 1);
console.log(vec2a.add(vec2b));
const vec3a = new vectors.Vector3(1, 2, 3);
const vec3b = new vectors.Vector3(3, 2, 1);
console.log(vec3a.add(vec3b));

```


As the compilerOptions.module option in your tsconfig.json file is set to CommonJS, the TypeScript Compiler creates code that is compatible with the Node.js module system. This uses the require function to load other files as modules.


Next, take a look at the compiled src/vectors.ts file, which is available at out/vectors.js:


vector_project/out/vectors.js
```
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.Vector3 = exports.Vector2 = void 0;
var vector2_1 = require("./vector2");
Object.defineProperty(exports, "Vector2", { enumerable: true, get: function () { return vector2_1.Vector2; } });
var vector3_1 = require("./vector3");
Object.defineProperty(exports, "Vector3", { enumerable: true, get: function () { return vector3_1.Vector3; } });

```


Here the TypeScript Compiler created code compatible with the way that modules are exported when using CommonJS, which is assigning the exported values to the exports object.


Now that you have tried out the syntax for importing and exporting files then seen how they are compiled into JavaScript, you can move on to declaring default exports in your file.


# Using Default Exports


In this section, you will examine another way to export a value from a module called default export, which sets a specific export to be the assumed import from a module. This can simplify the code when you import the files.


Open the src/vector2.ts file again:


vector_project/src/vector2.ts
```
export class Vector2 {
  constructor(public x: number, public y: number) {}

  add(otherVector2: Vector2) {
    return new Vector2(this.x + otherVector2.x, this.y + otherVector2.y);
  }
}

```


Notice how you are exporting a single value from your file/module. Another way you could have written your export was by using a default export. Every file can have at most a single default export, so this could be handy here.


To change your export to a default export, add the following highlighted code:


vector_project/src/vector2.ts
```
export default class Vector2 {
  constructor(public x: number, public y: number) {}

  add(otherVector2: Vector2) {
    return new Vector2(this.x + otherVector2.x, this.y + otherVector2.y);
  }
}

```


Save the file, then do the same in the src/vector3.ts file:


vector_project/src/vector3.ts
```
export default class Vector3 {
  constructor(public x: number, public y: number, public z: number) {}

  add(otherVector3: Vector3) {
    return new Vector3(
      this.x + otherVector3.x,
      this.y + otherVector3.y,
      this.z + otherVector3.z
    );
  }
}

```


To import your default exports, save your files, then open the src/vectors.ts file and change its contents to the following:


vector_project/src/vectors.ts
```
import Vector2 from "./vector2";
import Vector3 from "./vector3";

export { Vector2, Vector3 };

```


Notice that for both imports you are just giving a name to your import, instead of having to use destructuring to import a specific value. This will automatically import the default export of each module.


Every module that has a default export also has a special export called default, which can be used to access the default exported value. To use the export ... from shorthand syntax you were using before, you can use that named export:


vector_project/src/vectors.ts
```
export { default as Vector2 } from "./vector2";
export { default as Vector3 } from "./vector3";

```


Now you are re-exporting the default export of each module under a specific name.


# Using export = and import = require() for Compatibility


Some module loaders, like AMD and CommonJS, have an object called exports that contains all the values exported by a module. When using any of those module loaders it is possible to overwrite the exported object by changing the value of the exports object. This is similar to default exports available in ES Modules, and thus also in TypeScript itself. However, these two syntaxes are incompatible. In this section, you will take a look at how TypeScript handles this behavior in a compatible way with default exports.


In TypeScript, when targetting a module loader that supports overwriting the exported object, you can change the value of the exported object by using the export =  syntax. To do this, you assign the value of the exported object to the export identifier. If you’ve used Node.js in the past, this is the same as using exports = .



Note: Make sure the option compilerOptions.module in your tsconfig.json file is set to CommonJS before making any of the following changes.

Imagine you want to change the exported object in each of your vector files to point to the vector class itself. Open the file src/vector2.ts. To change the value of the exported object itself, make the following highlighted change:


vector_project/src/vector2.ts
```
export = class Vector2 {
  constructor(public x: number, public y: number) {}

  add(otherVector2: Vector2) {
    return new Vector2(this.x + otherVector2.x, this.y + otherVector2.y);
  }
};

```


Save this, then do the same for the file src/vector3.ts:


vector_project/src/vector3.ts
```
export = class Vector3 {
  constructor(public x: number, public y: number, public z: number) {}

  add(otherVector3: Vector3) {
    return new Vector3(
      this.x + otherVector3.x,
      this.y + otherVector3.y,
      this.z + otherVector3.z
    );
  }
};

```


Finally, change your vectors.ts back to the following:


vector_project/src/vectors.ts
```
import Vector2 from "./vector2";
import Vector3 from "./vector3";

export { Vector2, Vector3 };

```


Save these files, then run the TypeScript Compiler:


```
npx tsc


```


The TypeScript Compiler will give you multiple errors, including the following:


```
Outputsrc/vectors.ts:1:8 - error TS1259: Module '"~/project/src/vector2"' can only be default-imported using the 'esModuleInterop' flag

1 import Vector2 from "./vector2";
         ~~~~~~~

  src/vector2.ts:1:1
      1 export = class Vector2 {
        ~~~~~~~~~~~~~~~~~~~~~~~~
      2   constructor(public x: number, public y: number) {}
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ... 
      6   }
        ~~~
      7 }
        ~
    This module is declared with using 'export =', and can only be used with a default import when using the 'esModuleInterop' flag.

src/vectors.ts:2:8 - error TS1259: Module '"~/project/src/vector3"' can only be default-imported using the 'esModuleInterop' flag

2 import Vector3 from "./vector3";
         ~~~~~~~

  src/vector3.ts:1:1
      1 export = class Vector3 {
        ~~~~~~~~~~~~~~~~~~~~~~~~
      2   constructor(public x: number, public y: number, public z: number) {}
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ... 
     10   }
        ~~~
     11 }
        ~
    This module is declared with using 'export =', and can only be used with a default import when using the 'esModuleInterop' flag.

```


This error is due to the way you are importing your vector files inside src/vectors.ts. This file is still using the syntax for importing the default export of a module, but you are now overwriting the exported object, so you do not have a default export anymore. Using both syntaxes together is incompatible.


There are two ways to solve this: Using import = require() and setting the esModuleInterop property to true in the TypeScript Compiler configuration file.


First, you will try out the correct syntax to import this kind of module by making the following highlighted change to your code in the src/vectors.ts file:


vector_project/src/vectors.ts
```
import Vector2 = require("./vector2");
import Vector3 = require("./vector3");

export { Vector2, Vector3 };

```


The import = require() syntax uses the exports object as the value for the import itself, allowing you to use each vector class. Compiling your code will now work as expected.


Another approach to solving the TypeScript error 1259 is to set the option compilerOptions.esModuleInterop to true in the tsconfig.json file. By default, this value is false. When this is set to true, the TypeScript Compiler will emit additional JavaScript that checks the exported object to detect if it is a default export or an exports object that was overwritten, then uses it accordingly.


This works as expected and allows you to keep your code inside src/vectors.ts as before. For more information on how esModuleInterop works and what changes are made to the emitted JavaScript code, check the TypeScript documentation for the esModuleInterop option.



Note: All the changes made in this section assume that you are targetting a module system that supports overwriting the exported object of a module. Currently, those modules are AMD and CommonJS. If you were targetting a different module system, the TypeScript Compiler would give you an error during compilation.
For example, if the compilerOptions.module configuration were set to ES6 to target the ES module system, the TypeScript Compiler would give us multiple errors, which boils down to just two repeating errors, 1202 and 1203:
"Output"src/vector2.ts:1:1 - error TS1203: Export assignment cannot be used when targeting ECMAScript modules. Consider using 'export default' or another module format instead.

  1 export = class Vector2 {
    ~~~~~~~~~~~~~~~~~~~~~~~~
  2   constructor(public x: number, public y: number) {}
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
...
  6   }
    ~~~
  7 };
    ~~

src/vectors.ts:1:1 - error TS1202: Import assignment cannot be used when targeting ECMAScript modules. Consider using 'import * as ns from "mod"', 'import {a} from "mod"', 'import d from "mod"', or another module format instead.

1 import Vector2 = require("./vector2");

To learn more about targeting the ES module system, take a look at the TypeScript Compiler documentation for the module property.

# Conclusion


TypeScript offers a fully-featured module system with syntax inspired by the ES module specification, while allowing the developer to target a variety of other module systems in the emitted JavaScript Code, like CommonJS, AMD, UMD, SystemJS, and ES6. By using the import and export options available in TypeScript, you can ensure that your code is modular and compatible with the greater JavaScript environment. Knowing how to use modules will allow you to organize your application code concisely and efficiently, as modules are a fundamental part of having a well-structured code-base that is easy to extend and maintain.


For more tutorials on TypeScript, check out our How To Code in TypeScript series page.


