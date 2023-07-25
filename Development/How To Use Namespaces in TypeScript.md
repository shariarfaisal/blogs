# How To Use Namespaces in TypeScript

```JavaScript``` ```Development``` ```TypeScript```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


TypeScript is an extension of the JavaScript language that uses JavaScript’s runtime with a compile-time type checker. In TypeScript, you can use namespaces to organize your code. Previously known as internal modules, namespaces in TypeScript are based on an early draft of the ECMAScript modules. In the ECMAScript specification draft, internal modules were removed around September 2013, but TypeScript kept the idea under a different name.


Namespaces allow the developer to create separate organization units that can be used to hold multiple values, like properties, classes, types, and interfaces. In this tutorial, you will create and use namespaces to illustrate the syntax and what they can be used for. It will lead you through code samples of declaring and merging namespaces, how namespaces work as JavaScript code under the hood, and how they can be used to declare types for external libraries without typing.


# Prerequisites


To follow this tutorial, you will need:


- An environment in which you can execute TypeScript programs to follow along with the examples. To set this up on your local machine, you will need the following:

Both Node and npm (or yarn) installed in order to run a development environment that handles TypeScript-related packages. This tutorial was tested with Node.js version 14.3.0 and npm version 6.14.5. To install on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04. This also works if you are using the Windows Subsystem for Linux (WSL).
Additionally, you will need the TypeScript Compiler (tsc) installed on your machine. To do this, refer to the official TypeScript website.


- Both Node and npm (or yarn) installed in order to run a development environment that handles TypeScript-related packages. This tutorial was tested with Node.js version 14.3.0 and npm version 6.14.5. To install on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04. This also works if you are using the Windows Subsystem for Linux (WSL).
- Additionally, you will need the TypeScript Compiler (tsc) installed on your machine. To do this, refer to the official TypeScript website.
- If you do not wish to create a TypeScript environment on your local machine, you can use the official TypeScript Playground to follow along.
- You will need sufficient knowledge of JavaScript, especially ES6+ syntax, such as destructuring, rest operators, and imports/exports. If you need more information on these topics, reading our How To Code in JavaScript series is recommended.
- This tutorial will reference aspects of text editors that support TypeScript and show in-line errors. This is not necessary to use TypeScript but does take more advantage of TypeScript features. To gain the benefit of these, you can use a text editor like Visual Studio Code, which has full support for TypeScript out of the box. You can also try out these benefits in the TypeScript Playground.

All examples shown in this tutorial were created using TypeScript version 4.2.3.


# Creating Namespaces in TypeScript


In this section, you will create namespaces in TypeScript in order to illustrate the general syntax.


To create namespaces, you will use the namespace keyword followed by the name of the namespace and then a {} block. As an example, you will create a DatabaseEntity namespace to hold database entities, as if you were using an Object–relational mapping (ORM) library. Add the following code to a new TypeScript file:


```
namespace DatabaseEntity {
}

```


This declares the DatabaseEntity namespace, but doesn’t yet add code to that namespace. Next, add a User class inside the namespace to represent a User entity in the database:


```
namespace DatabaseEntity {
  class User {
    constructor(public name: string) {}
  }
}

```


You can use your User class normally inside your namespace. To illustrate this create a new User instance and store it in the newUser variable:


```
namespace DatabaseEntity {
  class User {
    constructor(public name: string) {}
  }

  const newUser = new User("Jon");
}

```


This is valid code. However, if you tried to use User outside of the namespace, the TypeScript Compiler would give you the error 2339:


```
OutputProperty 'User' does not exist on type 'typeof DatabaseEntity'. (2339)

```


If you wanted to use your class outside of your namespace, you would have to first export the User class to be available externally, as shown in the highlighted code below:


```
namespace DatabaseEntity {
  export class User {
    constructor(public name: string) {}
  }

  const newUser = new User("Jon");
}

```


You are now able to access the User class outside of the DatabaseEntity namespace by using its fully qualified name. In this case, the fully qualified name is DatabaseEntity.User:


```
namespace DatabaseEntity {
  export class User {
    constructor(public name: string) {}
  }

  const newUser = new User("Jon");
}

const newUserOutsideNamespace = new DatabaseEntity.User("Jane");

```


You can export anything from within a namespace, including variables, which then become properties in the namespace. In the following code, you are exporting the newUser variable:


```
namespace DatabaseEntity {
  export class User {
    constructor(public name: string) {}
  }

  export const newUser = new User("Jon");
}

console.log(DatabaseEntity.newUser.name);

```


Since the variable newUser was exported, you can access it as a property of the namespace. Running this code would print the following to the console:


```
OutputJon

```


Just like with interfaces, namespaces in TypeScript also allow for declaration merging. This means that multiple declarations of the same namespace will be merged into a single declaration. This can add flexibility to a namespace if you need to extend it later in your code.


Using the previous example, this means that if you declared your DatabaseEntity namespace again, you would be able to extend the namespace with more properties. Add a new class UserRole to your DatabaseEntity namespace by using another namespace declaration:


```
namespace DatabaseEntity {
  export class User {
    constructor(public name: string) {}
  }

  export const newUser = new User("Jon");
}

namespace DatabaseEntity {
  export class UserRole {
    constructor(public user: User, public role: string) {}
  }

  export const newUserRole = new UserRole(newUser, "admin");
}

```


Inside your new DatabaseEntity namespace declaration, you can use any member previously exported in the DatabaseEntity namespace, including from previous declarations, without having to use their fully qualified name. You are using the name as it was declared in the first namespace to set the type of the user parameter in the UserRole constructor to be of type User, and when creating a new UserRole instance by using the newUser value. This is only possible because you exported those in the previous namespace declaration.


Now that you’ve taken a look at the basic syntax of namespaces, you can move on to examining how namespaces are translated into JavaScript by the TypeScript Compiler.


# Examining the JavaScript Code Generated When Using Namespaces


Namespaces in TypeScript are not just a compile-time feature; they also change the resulting JavaScript code. To learn more about how namespaces work, you can analyze the JavaScript that powers this TypeScript feature. In this step, you’ll take the code snippets from the last section and examine their underlying JavaScript implementation.


Take the code you used in the first example:


```
namespace DatabaseEntity {
  export class User {
    constructor(public name: string) {}
  }

  export const newUser = new User("Jon");
}

console.log(DatabaseEntity.newUser.name);

```


The TypeScript Compiler would generate the following JavaScript code for this TypeScript snippet:


```
"use strict";
var DatabaseEntity;
(function (DatabaseEntity) {
    class User {
        constructor(name) {
            this.name = name;
        }
    }
    DatabaseEntity.User = User;
    DatabaseEntity.newUser = new User("Jon");
})(DatabaseEntity || (DatabaseEntity = {}));
console.log(DatabaseEntity.newUser.name);

```


To declare the DatabaseEntity namespace, the TypeScript Compiler creates an uninitialized variable called DatabaseEntity, and then creates an Immediately Invoked Function Expression (IIFE). This IIFE receives a single parameter DatabaseEntity || (DatabaseEntity = {}), which is the current value of the DatabaseEntity variable. If it is not set to a truthy value, it sets the value of the variable to an empty object.


Setting the value of your DatabaseEntity to an empty value when passing it to the IIFE works because the return value of an assignment operation is the value being assigned. In this case, this is the empty object.


Inside the IIFE, the User class is created and then assigned to the User property of the DatabaseEntity object. The same occurs for the newUser property, where you assign the property to the value of a new User instance.


Now take a look at the second code example, where you had multiple namespace declarations:


```
namespace DatabaseEntity {
  export class User {
    constructor(public name: string) {}
  }

  export const newUser = new User("Jon");
}

namespace DatabaseEntity {
  export class UserRole {
    constructor(public user: User, public role: string) {}
  }

  export const newUserRole = new UserRole(newUser, "admin");
}

```


The generated JavaScript code would look like this:


```
"use strict";
var DatabaseEntity;
(function (DatabaseEntity) {
    class User {
        constructor(name) {
            this.name = name;
        }
    }
    DatabaseEntity.User = User;
    DatabaseEntity.newUser = new User("Jon");
})(DatabaseEntity || (DatabaseEntity = {}));
(function (DatabaseEntity) {
    class UserRole {
        constructor(user, role) {
            this.user = user;
            this.role = role;
        }
    }
    DatabaseEntity.UserRole = UserRole;
    DatabaseEntity.newUserRole = new UserRole(DatabaseEntity.newUser, "admin");
})(DatabaseEntity || (DatabaseEntity = {}));

```


The beginning of the code looks the same as you had previously, with the uninitialized variable DatabaseEntity and then an IIFE with the actual code setting the properties of the DatabaseEntity object. This time though you also have another IIFE. This new IIFE matches the second declaration of your DatabaseEntity namespace.


Now, when the second IIFE is executed, DatabaseEntity is already bound to an object, so you are just extending upon the already available object by adding extra properties.


You’ve now taken a look at the syntax for TypeScript namespaces and how they work in the underlying JavaScript. With this context, you can now run through a common use case for namespaces: Defining types for external libraries without typing.


# Using Namespaces to Provide Typing for External Libraries


In this section, you will run through one of the scenarios where namespaces are useful: creating module declarations for external libraries. To do this, you will write a new file in your TypeScript project to declare the typing, then change your tsconfig.json file to make the TypeScript Compiler recognize the type.



Note: To follow the next steps, a TypeScript environment with access to the file system is necessary. If you are using the TypeScript Playground, you can export the existing code to a CodeSandbox project by clicking on Export in the top menu and then Open in CodeSandbox. This will allow you to create new files and edit the tsconfig.json file.

Not every package available in the npm registry bundles its own TypeScript module declaration. This means that when installing a package in your project, you may encounter a compilation error related to the package’s missing type declaration or have to work with a library that has all its types set to any. Depending on how strictly you are using TypeScript, this may be an undesired result.


Hopefully, this package will have a @types package created by the DefinetelyTyped community, allowing you to install the package and get working types for that library. However, this is not always the case, and sometimes you’ll have to deal with a library that does not bundle its own type module declaration. In this case, if you want to keep your code completely type-safe, you have to create the module declaration yourself.


As an example, imagine you are using a vector library called example-vector3 that exports a single class, Vector3, with a single method, add. This method is used to add two Vector3 vectors together.


The code in the library could look something like the following:


```
export class Vector3 {
  super(x, y, z) {
    this.x = x;
    this.y = y;
    this.z = z;
  }

  add(vec) {
    let x = this.x + vector.x;
    let y = this.y + vector.y;
    let z = this.z + vector.z;

    let newVector = new Vector3(x, y, z);

    return newVector
  }
}

```


This exports a class that creates vectors with properties x, y, and z, meant to represent the coordinate components of the vector.


Next, take a look at an example piece of code that uses the hypothetical library:


index.ts
```
import { Vector3 } from "example-vector3";

const v1 = new Vector3(1, 2, 3);
const v2 = new Vector3(1, 2, 3);

const v3 = v1.add(v2);

```


The example-vector3 library is not bundled with its own type declaration, so the TypeScript Compiler is going to give error 2307:


```
OutputCannot find module 'example-vector3' or its corresponding type declarations. ts(2307)

```


To fix this problem, you will now create a type declaration file for this package. First, create a new file called types/example-vector3/index.d.ts and open it in your favorite editor. Inside this file write the following code:


types/example-vector3/index.d.ts
```
declare module "example-vector3" {
  export = vector3;

  namespace vector3 {
  }
}

```


In this code, you are creating the type declaration for the example-vector3 module. The first part of the code is the declare module block itself. The TypeScript Compiler is going to parse this block and interpret everything inside as if it were the type representation of the module itself. This means that anything you declare here, TypeScript is going to use to infer the type of the module. Right now, you are saying that this module exports a single namespace called vector3, which is currently empty.


Save and exit from this file.


The TypeScript Compiler is currently not aware of your declaration file, so you must include it in your tsconfig.json. To do this, edit the project tsconfig.json by adding the types property to the compilerOptions option:


tsconfig.json
```
{
  "compilerOptions": {
    ...
    "types": ["./types/example-vector3/index.d.ts"]
  }
}

```


Now if you go back to your original code, you will see that the error has changed. The TypeScript Compiler is now giving error 2305:


```
OutputModule '"example-vector3"' has no exported member 'Vector3'. ts(2305)

```


While you created the module declaration for example-vector3, the export is currently set to an empty namespace. There is no Vector3 class being exported from within that namespace.


Re-open types/example-vector3/index.d.ts and write the following code:


types/example-vector3/index.d.ts
```
declare module "example-vector3" {
  export = vector3;

  namespace vector3 {
    export class Vector3 {
      constructor(x: number, y: number, z: number);
      add(vec: Vector3): Vector3;
    }
  }
}

```


In this code, notice how you are now exporting a class inside the vector3 namespace. The main goal of module declarations is to provide the type information of values that are exposed by a library. This way you can use it in a type-safe way.


In this case, you know that the example-vector3 library provides a class called Vector3 that accepts three numbers in the constructor, and that has an add method used to add two Vector3 instances together, returning a new instance as the result. You do not need to provide the implementation here, just the type information itself. Declarations that do not provide an implementation are known as ambient declarations in TypeScript, and it is common to create those inside .d.ts file.


This code will now compile correctly and have correct types for the Vector3 class.


Using namespaces, you can isolate what is exported by the library into a single type unit, which in this case is the vector3 namespace. This makes it easier to customize the module declaration down the road or to even make the type declaration available to all developers by submitting it to the DefinetelyTyped repository.


# Conclusion


In this tutorial, you ran through the basic syntax of namespaces in TypeScript and examined the JavaScript that the TypeScript Compiler changes it into. You also tried out a common use case of namespaces: To provide ambient typing for external libraries that are not yet typed.


While namespaces are not deprecated, using namespaces as the code organization mechanism in your code base is not always recommended. Modern code should use the ES Module syntax, as it has all the features provided by namespaces, and starting with ECMAScript 2015 it became part of the specification. However, when creating module declarations, the usage of namespaces is still recommended as it allows for more concise type declarations.


For more tutorials on TypeScript, check out our How To Code in TypeScript series page.


