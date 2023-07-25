# How To Use Decorators in TypeScript

```JavaScript``` ```TypeScript``` ```Development```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


TypeScript is an extension of the JavaScript language that uses JavaScript’s runtime with a compile-time type checker. This combination allows developers to use the full JavaScript ecosystem and language features, while also adding optional static type-checking, enums, classes, and interfaces on top of it. One of those extra features is decorator support.


Decorators are a way to decorate members of a class, or a class itself, with extra functionality. When you apply a decorator to a class or a class member, you are actually calling a function that is going to receive details of what is being decorated, and the decorator implementation will then be able to transform the code dynamically, adding extra functionality, and reducing boilerplate code. They are a way to have metaprogramming in TypeScript, which is a programming technique that enables the programmer to create code that uses other code from the application itself as data.



Currently, there is a stage-2 proposal adding decorators to the ECMAScript standard. As it is not a JavaScript feature yet, TypeScript offers its own implementation of decorators, under an experimental flag.

This tutorial will show you how create your own decorators in TypeScript for classes and class members, and also how to use them. It will lead you through different code samples, which you can follow in your own TypeScript environment or the TypeScript Playground, an online environment that allows you to write TypeScript directly in the browser.


# Prerequisites


To follow this tutorial, you will need:


- An environment in which you can execute TypeScript programs to follow along with the examples. To set this up on your local machine, you will need the following.

Both Node and npm (or yarn) installed in order to run a development environment that handles TypeScript-related packages. This tutorial was tested with Node.js version 14.3.0 and npm version 6.14.5. To install on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04. This also works if you are using the Windows Subsystem for Linux (WSL).
Additionally, you will need the TypeScript Compiler (tsc) installed on your machine. To do this, refer to the official TypeScript website.


- Both Node and npm (or yarn) installed in order to run a development environment that handles TypeScript-related packages. This tutorial was tested with Node.js version 14.3.0 and npm version 6.14.5. To install on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04. This also works if you are using the Windows Subsystem for Linux (WSL).
- Additionally, you will need the TypeScript Compiler (tsc) installed on your machine. To do this, refer to the official TypeScript website.
- If you do not wish to create a TypeScript environment on your local machine, you can use the official TypeScript Playground to follow along.
- You will need sufficient knowledge of JavaScript, especially ES6+ syntax, such as destructuring, rest operators, and imports/exports. If you need more information on these topics, reading our How To Code in JavaScript series is recommended.
- This tutorial will reference aspects of text editors that support TypeScript and show in-line errors. This is not necessary to use TypeScript but does take more advantage of TypeScript features. To gain the benefit of these, you can use a text editor like Visual Studio Code, which has full support for TypeScript out of the box. You can also try out these benefits in the TypeScript Playground.

All examples shown in this tutorial were created using TypeScript version 4.2.2.


# Enabling Decorators Support in TypeScript


Currently, decorators are still an experimental feature in TypeScript, and as such, it must be enabled first. In this section, you will see how to enable decorators in TypeScript, depending on the way you are working with TypeScript.


## TypeScript Compiler CLI


To enable decorators support when using the TypeScript Compiler CLI (tsc) the only extra step needed is to pass an additional flag --experimentalDecorators:


```
tsc --experimentalDecorators

```


## tsconfig.json


When working in a project that has a tsconfig.json file, to enable experimental decorators you must add the experimentalDecorators property to the compilerOptions object:


```
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}

```


In the TypeScript Playground, decorators are enabled by default.


# Using Decorator Syntax


In this section, you will apply decorators in TypeScript classes.


In TypeScript, you can create decorators using the special syntax @expression, where expression is a function that will be called automatically during runtime with details about the target of the decorator.


The target of a decorator depends on where you add them. Currently, decorators can be added to the following components of a class:


- Class declaration itself
- Properties
- Accessors
- Methods
- Parameters

For example, let’s say you have a decorator called sealed which calls Object.seal in a class. To use your decorator you could write the following:


```
@sealed
class Person {}

```


Notice in the highlighted code that you added the decorator right before the target of your sealed decorator, in this case, the Person class declaration.


The same is valid for all other kinds of decorators:


```
@classDecorator
class Person {
  @propertyDecorator
  public name: string;

  @accessorDecorator
  get fullName() {
    // ...
  }

  @methodDecorator
  printName(@parameterDecorator prefix: string) {
    // ...
  }
}

```


To add multiple decorators, you add them together, one after the other:


```
@decoratorA
@decoratorB
class Person {}

```


# Creating Class Decorators in TypeScript


In this section, you will go through the steps to create class decorators in TypeScript.


For a decorator called @decoratorA, you tell TypeScript it should call the function decoratorA. The decoratorA function will be called with details about how you used the decorator in your code. For example, if you applied the decorator to a class declaration, the function will receive details about the class. This function must be in scope for your decorator to work.


To create your own decorator, you have to create a function with the same name as your decorator. That is, to create the sealed class decorator you saw in the previous section, you have to create a sealed function that receives a specific set of parameters. Let’s do exactly that:


```
@sealed
class Person {}

function sealed(target: Function) {
  Object.seal(target);
  Object.seal(target.prototype);
}

```


The parameter(s) passed to the decorator will depend on where the decorator will be used. The first parameter is commonly called target.


The sealed decorator will be used only on class declarations, so your function will receive a single parameter, the target, which will be of type Function. This will be the constructor of the class that the decorator was applied to.


In the sealed function you are then calling Object.seal on the target, which is the class constructor, and also on their prototype. When you do that no new properties can be added to the class constructor or their property, and the existing ones will be marked as non-configurable.


It is important to remember that it is currently not possible to extend the TypeScript type of the target when using decorators. This means, for example, you are not able to add a new field to a class using a decorator and make it type-safe.


If you returned a value in the sealed class decorator, this value will become the new constructor function for the class. This is useful if you want to completely overwrite the class constructor.


You have created your first decorator, and used it with a class. In the next section you will learn how to create decorator factories.


# Creating Decorator Factories


Sometimes you will need to pass additional options to the decorator when applying it, and for that, you have to use decorator factories. In this section, you will learn how to create those factories and use them.


Decorator factories are functions that return another function. They receive this name because they are not the decorator implementation itself. Instead, they return another function responsible for the implementation of the decorator and act as a wrapper function. They are useful in making decorators customizable, by allowing the client code to pass options to the decorators when using them.


Let’s imagine you have a class decorator called decoratorA and you want to add an option that can be set when calling the decorator, like a boolean flag. You can achieve this by writing a decorator factory similar to the following one:


```
const decoratorA = (someBooleanFlag: boolean) => {
  return (target: Function) => {
  }
}

```


Here, your decoratorA function returns another function with the implementation of the decorator. Notice how the decorator factory receives a boolean flag as its only parameter:


```
const decoratorA = (someBooleanFlag: boolean) => {
  return (target: Function) => {
  }
}

```


You are able to pass the value of this parameter when using the decorator. See the highlighted code in the following example:


```
const decoratorA = (someBooleanFlag: boolean) => {
  return (target: Function) => {
  }
}

@decoratorA(true)
class Person {}

```


Here, when you use the decoratorA decorator, the decorator factory is going to be called with the someBooleanFlag parameter set to true. Then the decorator implementation itself will run. This allows you to change the behavior of your decorator based on how it was used, making your decorators easy to customize and re-use through your application.


Notice that you are required to pass all the parameters expected by the decorator factory. If you simply applied the decorator without passing any parameters, like in the following example:


```
const decoratorA = (someBooleanFlag: boolean) => {
  return (target: Function) => {
  }
}

@decoratorA
class Person {}

```


The TypeScript Compiler will give you two errors, which may vary depending on the type of the decorator. For class decorators the errors are 1238 and 1240:


```
OutputUnable to resolve signature of class decorator when called as an expression.
  Type '(target: Function) => void' is not assignable to type 'typeof Person'.
    Type '(target: Function) => void' provides no match for the signature 'new (): Person'. (1238)
Argument of type 'typeof Person' is not assignable to parameter of type 'boolean'. (2345)

```


You just created a decorator factory that is able to receive parameters and change their behavior based on these parameters. In the next step you will learn how to create property decorators.


# Creating Property Decorators


Class properties are another place you can use decorators. In this section you will take a look at how to create them.


Any property decorator receives the following parameters:


- For static properties, the constructor function of the class. For all the other properties, the prototype of the class.
- The name of the member.

Currently, there is no way to obtain the property descriptor as a parameter. This is due to the way that property decorators are initialized in TypeScript.


Here is a decorator function that will print the name of the member to the console:


```
const printMemberName = (target: any, memberName: string) => {
  console.log(memberName);
};

class Person {
  @printMemberName
  name: string = "Jon";
}

```


When you run the above TypeScript code, you will see the following printed in the console:


```
Outputname

```


You can use property decorators to override the property being decorated. This can be done by using Object.defineProperty along with a new setter and getter for the property. Let’s see how you can create a decorator named allowlist, which only allows a property to be set to values present in a static allowlist:


```
const allowlist = ["Jon", "Jane"];

const allowlistOnly = (target: any, memberName: string) => {
  let currentValue: any = target[memberName];

  Object.defineProperty(target, memberName, {
    set: (newValue: any) => {
      if (!allowlist.includes(newValue)) {
        return;
      }
      currentValue = newValue;
    },
    get: () => currentValue
  });
};

```


First, you are creating a static allowlist at the top of the code:


```
const allowlist = ["Jon", "Jane"];

```


You are then creating the implementation of the property decorator:


```
const allowlistOnly = (target: any, memberName: string) => {
  let currentValue: any = target[memberName];

  Object.defineProperty(target, memberName, {
    set: (newValue: any) => {
      if (!allowlist.includes(newValue)) {
        return;
      }
      currentValue = newValue;
    },
    get: () => currentValue
  });
};

```


Notice how you are using any as the type of the target:


```
const allowlistOnly = (target: any, memberName: string) => {

```


For property decorators, the type of the target parameter can be either the constructor of the class or the prototype of the class, it is easier to use any in this situation.


In the first line of your decorator implementation you are storing the current value of the property being decorated to the currentValue variable:


```
  let currentValue: any = target[memberName];

```


For static properties, this will be set to their default value, if any. For non-static properties, this will always be undefined. This is because at runtime, in the compiled JavaScript code, the decorator runs before the instance property is set to its default value.


You are then overriding the property by using Object.defineProperty:


```
  Object.defineProperty(target, memberName, {
    set: (newValue: any) => {
      if (!allowlist.includes(newValue)) {
        return;
      }
      currentValue = newValue;
    },
    get: () => currentValue
  });

```


The Object.defineProperty call has a getter and a setter. The getter returns the value stored in the currentValue variable. The setter will set the value of currentVariable to newValue if it is within the allowlist.


Let’s use the decorator you just wrote. Create the following Person class:


```
class Person {
  @allowlistOnly
  name: string = "Jon";
}

```


You will now create a new instance of your class, and test setting and getting the name instance property:


```
const allowlist = ["Jon", "Jane"];

const allowlistOnly = (target: any, memberName: string) => {
  let currentValue: any = target[memberName];

  Object.defineProperty(target, memberName, {
    set: (newValue: any) => {
      if (!allowlist.includes(newValue)) {
        return;
      }
      currentValue = newValue;
    },
    get: () => currentValue
  });
};

class Person {
  @allowlistOnly
  name: string = "Jon";
}

const person = new Person();
console.log(person.name);

person.name = "Peter";
console.log(person.name);

person.name = "Jane";
console.log(person.name);

```


Running the code you should see the following output:


```
OutputJon
Jon
Jane

```


The value is never set to Peter, as Peter is not in the allowlist.


What if you wanted to make your code a bit more re-usable, allowing the allowlist to be set when applying the decorator? This is a great use case for decorator factories. Let’s do exactly that, by turning your allowlistOnly decorator into a decorator factory:


```
const allowlistOnly = (allowlist: string[]) => {
  return (target: any, memberName: string) => {
    let currentValue: any = target[memberName];

    Object.defineProperty(target, memberName, {
      set: (newValue: any) => {
        if (!allowlist.includes(newValue)) {
          return;
        }
        currentValue = newValue;
      },
      get: () => currentValue
    });
  };
}

```


Here you wrapped your previous implementation into another function, a decorator factory. The decorator factory receives a single parameter called allowlist, which is an array of strings.


Now to use your decorator, you must pass the allowlist, like in the following highlighted code:


```
class Person {
  @allowlistOnly(["Claire", "Oliver"])
  name: string = "Claire";
}

```


Try running a code similar to the previous one you wrote, but with the new changes:


```
const allowlistOnly = (allowlist: string[]) => {
  return (target: any, memberName: string) => {
    let currentValue: any = target[memberName];

    Object.defineProperty(target, memberName, {
      set: (newValue: any) => {
        if (!allowlist.includes(newValue)) {
          return;
        }
        currentValue = newValue;
      },
      get: () => currentValue
    });
  };
}

class Person {
  @allowlistOnly(["Claire", "Oliver"])
  name: string = "Claire";
}

const person = new Person();
console.log(person.name);
person.name = "Peter";
console.log(person.name);
person.name = "Oliver";
console.log(person.name);

```


The code should give you the following output:


```
OutputClaire
Claire
Oliver

```


Showing that it is working as expected, person.name is never set to Peter, as Peter is not in the given allowlist.


Now that you created your first property decorator using both a normal decorator function and a decorator factory, it is time to take a look at how to create decorators for class accessors.


# Creating Accessor Decorators


In this section, you will take a look at how to decorate class accessors.


Just like property decorators, decorators used in an accessor receives the following parameters:


1. For static properties, the constructor function of the class, for all other properties, the prototype of the class.
2. The name of the member.

But differently from the property decorator, it also receives a third parameter, with the Property Descriptor of the accessor member.


Given the fact that Property Descriptors contains both the setter and getter for a particular member, accessor decorators can only be applied to either the setter or the getter of a single member, not to both.


If you return a value from your accessor decorator, this value will become the new Property Descriptor of the accessor for both the getter and the setter members.


Here is an example of a decorator that can be used to change the enumerable flag of a getter/setter accessor:


```
const enumerable = (value: boolean) => {
  return (target: any, memberName: string, propertyDescriptor: PropertyDescriptor) => {
    propertyDescriptor.enumerable = value;
  }
}

```


Notice in the example how you are using a decorator factory. This allows you to specify the enumerable flag when calling your decorator. Here is how you could use your decorator:


```
class Person {
  firstName: string = "Jon"
  lastName: string = "Doe"

  @enumerable(true)
  get fullName () {
    return `${this.firstName} ${this.lastName}`;
  }
}

```


Accessor decorators are similar to property decorators. The only difference is that they receive a third parameter with the property descriptor. Now that you created your first accessor decorator, the next section will show you how to create method decorators.


# Creating Method Decorators


In this section, you will take a look at how to use method decorators.


The implementation of method decorators is very similar to the way you create accessor decorators. The parameters passed to the decorator implementation are identical to the ones passed to accessor decorators.


Let’s re-use that same enumerable decorator you created before, but this time in the getFullName method of the following Person class:


```
const enumerable = (value: boolean) => {
  return (target: any, memberName: string, propertyDescriptor: PropertyDescriptor) => {
    propertyDescriptor.enumerable = value;
  }
}

class Person {
  firstName: string = "Jon"
  lastName: string = "Doe"

  @enumerable(true)
  getFullName () {
    return `${this.firstName} ${this.lastName}`;
  }
}

```


If you returned a value from your method decorator, this value will become the new Property Descriptor of the method.


Let’s create a deprecated decorator which prints the passed message to the console when the method is used, logging a message saying that the method is deprecated:


```
const deprecated = (deprecationReason: string) => {
  return (target: any, memberName: string, propertyDescriptor: PropertyDescriptor) => {
    return {
      get() {
        const wrapperFn = (...args: any[]) => {
          console.warn(`Method ${memberName} is deprecated with reason: ${deprecationReason}`);
          propertyDescriptor.value.apply(this, args)
        }

        Object.defineProperty(this, memberName, {
            value: wrapperFn,
            configurable: true,
            writable: true
        });
        return wrapperFn;
      }
    }
  }
}

```


Here, you are creating a decorator using a decorator factory. This decorator factory receives a single argument of type string, which is the reason for the deprecation, as shown in the highlighted part below:


```
const deprecated = (deprecationReason: string) => {
  return (target: any, memberName: string, propertyDescriptor: PropertyDescriptor) => {
    // ...
  }
}

```


The deprecationReason will be used later when logging the deprecation message to the console. In the implementation of your deprecated decorator, you are returning a value. When you return a value from a method decorator, this value will overwrite this member’s Property Descriptor.


You are taking advantage of that to add a getter to your decorated class method. This way you can change the implementation of the method itself.


But why not just use Object.defineProperty instead of returning a new property decorator for the method? This is necessary as you need to access the value of this, which for non-static class methods, is bound to the class instance. If you used Object.defineProperty directly there would be no way for you to retrieve the value of this, and if the method used this in any way, the decorator would break your code when you run the wrapped method from within your decorator implementation.


In your case, the getter itself has the this value bound to the class instance for non-static methods and bound to the class constructor for static methods.


Inside your getter you are then creating a wrapper function locally, called wrapperFn, this function logs a message to the console using console.warn, passing the deprecationReason received from the decorator factory, you are then calling the original method, using propertyDescriptor.value.apply(this, args), this way the original method is called with their this value correctly bound to the class instance in case it was a non-static method.


You are then using defineProperty to overwrite the value of your method in the class. This works like a memoization mechanism, as multiple calls to the same method will not call your getter anymore, but the wrapperFn directly. You are now setting the member in the class to have your wrapperFn as their value by using Object.defineProperty.


Let’s use your deprecated decorator:


```
const deprecated = (deprecationReason: string) => {
  return (target: any, memberName: string, propertyDescriptor: PropertyDescriptor) => {
    return {
      get() {
        const wrapperFn = (...args: any[]) => {
          console.warn(`Method ${memberName} is deprecated with reason: ${deprecationReason}`);
          propertyDescriptor.value.apply(this, args)
        }

        Object.defineProperty(this, memberName, {
            value: wrapperFn,
            configurable: true,
            writable: true
        });
        return wrapperFn;
      }
    }
  }
}

class TestClass {
  static staticMember = true;

  instanceMember: string = "hello"

  @deprecated("Use another static method")
  static deprecatedMethodStatic() {
    console.log('inside deprecated static method - staticMember =', this.staticMember);
  }

  @deprecated("Use another instance method")
  deprecatedMethod () {
    console.log('inside deprecated instance method - instanceMember =', this.instanceMember);
  }
}

TestClass.deprecatedMethodStatic();

const instance = new TestClass();
instance.deprecatedMethod();

```


Here, you created a TestClass with two properties: one static and one non-static. You also created two methods: one static and one non-static.


You are then applying your deprecated decorator to both methods. When you run the code, the following will appear in the console:


```
Output(warning) Method deprecatedMethodStatic is deprecated with reason: Use another static method
inside deprecated static method - staticMember = true
(warning)) Method deprecatedMethod is deprecated with reason: Use another instance method
inside deprecated instance method - instanceMember = hello

```


This shows that both methods were correctly wrapped with your wrapper function, which logs a message to the console with the deprecation reason.


You have now created your first method decorator using TypeScript. The next section will show you how to create the last decorator type supported by TypeScript, a parameter decorator.


# Creating Parameter Decorators


Parameter decorators can be used in class method’s parameters. In this section, you will learn how to create one.


The decorator function used with parameters receives the following parameters:


1. For static properties, the constructor function of the class. For all other properties, the prototype of the class.
2. The name of the member.
3. The index of the parameter in the method’s parameter list.

It is not possible to change anything related to the parameter itself, so such decorators are only useful for observing the parameter usage itself (unless you use something more advanced like reflect-metadata).


Here is an example of a decorator that prints the index of the parameter that was decorated, along with the method name:


```
function print(target: Object, propertyKey: string, parameterIndex: number) {
  console.log(`Decorating param ${parameterIndex} from ${propertyKey}`);
}

```


Then you can use your parameter decorator like this:


```
class TestClass {
  testMethod(param0: any, @print param1: any) {}
}

```


Running the above code should display the following in the console:


```
OutputDecorating param 1 from testMethod

```


You’ve now created and executed a parameter decorator, and printed out the result that returns the index of the decorated parameter.


# Conclusion


In this tutorial, you have implemented all decorators supported by TypeScript, used them with classes and learned the differences between each one of them. You can now start writing your own decorators to reduce boilerplate code in your codebase, or use decorators with libraries, such as Mobx, with more confidence.


For more tutorials on TypeScript, check out our How To Code in TypeScript series page.


