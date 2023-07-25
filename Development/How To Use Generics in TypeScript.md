# How To Use Generics in TypeScript

```JavaScript``` ```Development``` ```TypeScript```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Generics are a fundamental feature of statically-typed languages, allowing developers to pass types as parameters to another type, function, or other structure. When a developer makes their component a generic component, they give that component the ability to accept and enforce typing that is passed in when the component is used, which improves code flexibility, makes components reusable, and removes duplication.


TypeScript fully supports generics as a way to introduce type-safety into components that accept arguments and return values whose type will be indeterminate until they are consumed later in your code. In this tutorial, you will try out real-world examples of TypeScript generics and explore how they are used in functions, types, classes, and interfaces. You will also use generics to create mapped types and conditional types, which will help you create TypeScript components that have the flexibility to apply to all necessary situations in your code.


# Prerequisites


To follow this tutorial, you will need:


- An environment in which you can execute TypeScript programs to follow along with the examples. To set this up on your local machine, you will need the following:

Both Node and npm (or yarn) installed in order to run a development environment that handles TypeScript-related packages. To install on macOS or Ubuntu 20.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Node.js with Apt Using a NodeSource PPA section of How To Install Node.js on Ubuntu 20.04. This also works if you are using the Windows Subsystem for Linux (WSL).
Additionally, you will need the TypeScript Compiler (tsc) installed on your machine. To do this, refer to the official TypeScript website.


- Both Node and npm (or yarn) installed in order to run a development environment that handles TypeScript-related packages. To install on macOS or Ubuntu 20.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Node.js with Apt Using a NodeSource PPA section of How To Install Node.js on Ubuntu 20.04. This also works if you are using the Windows Subsystem for Linux (WSL).
- Additionally, you will need the TypeScript Compiler (tsc) installed on your machine. To do this, refer to the official TypeScript website.
- If you do not wish to create a TypeScript environment on your local machine, you can use the official TypeScript Playground to follow along.
- You will need sufficient knowledge of JavaScript, especially ES6+ syntax, such as destructuring, rest operators, and imports/exports. If you need more information on these topics, reading our How To Code in JavaScript series is recommended.
- This tutorial will reference aspects of text editors that support TypeScript and show in-line errors. This is not necessary to use TypeScript, but does take more advantage of TypeScript features. To gain the benefit of these, you can use a text editor like Visual Studio Code, which has full support for TypeScript out of the box. You can also try out these benefits in the TypeScript Playground.

All examples shown in this tutorial were created using TypeScript version 4.2.3.


# Generics Syntax


Before getting into the application of generics, this tutorial will first go through syntax for TypeScript generics, followed by an example to illustrate their general purpose.


Generics appear in TypeScript code inside angle brackets, in the format <T>, where T represents a passed-in type. <T> can be read as a generic of type T. In this case, T will operate in the same way that parameters work in functions, as placeholders for a type that will be declared when an instance of the structure is created. The generic types specified inside angle brackets are therefore also known as generic type parameters or just type parameters. Multiple generic types can also appear in a single definition, like <T, K, A>.



Note: By convention, programmers usually use a single letter to name a generic type. This is not a syntax rule, and you can name generics like any other type in TypeScript, but this convention helps to immediately convey to those reading your code that a generic type does not require a specific type.

Generics can appear in functions, types, classes, and interfaces. Each of these structures will be covered later in this tutorial, but for now a function will be used as an example to illustrate the basic syntax of generics.


To see how useful generics are, imagine that you have a JavaScript function that takes two parameters: an object and an array of keys. The function will return a new object based on the original one, but with only the keys you want:


```
function pickObjectKeys(obj, keys) {
  let result = {}
  for (const key of keys) {
    if (key in obj) {
      result[key] = obj[key]
    }
  }
  return result
}

```


This snippet shows the pickObjectKeys() function, which iterates over the keys array and creates a new object with the keys specified in the array.


Here is an example showing how to use the function:


```
const language = {
  name: "TypeScript",
  age: 8,
  extensions: ['ts', 'tsx']
}

const ageAndExtensions = pickObjectKeys(language, ['age', 'extensions'])

```


This declares an object language, then isolates the age and extensions property with the pickObjectKeys() function. The value of ageAndExtensions would be as follows:


```
{
  age: 8,
  extensions: ['ts', 'tsx']
}

```


If you were to migrate this code to TypeScript to make it type-safe, you would have to use generics. You could refactor the code by adding the following highlighted lines:


```
function pickObjectKeys<T, K extends keyof T>(obj: T, keys: K[]) {
  let result = {} as Pick<T, K>
  for (const key of keys) {
    if (key in obj) {
      result[key] = obj[key]
    }
  }
  return result
}

const language = {
  name: "TypeScript",
  age: 8,
  extensions: ['ts', 'tsx']
}

const ageAndExtensions = pickObjectKeys(language, ['age', 'extensions'])

```


<T, K extends keyof T> declares two parameter types for the function, where K is assigned a type that is the union of the keys in T. The obj function parameter is then set to whatever type T represents, and keys to an array of whatever type K represents. Since T in the case of the language object sets age as a number and extensions as an array of strings, the variable ageAndExtensions will now be assigned the type of an object with properties age: number and extensions: string[].


This enforces a return type based on the arguments supplied to pickObjectKeys, allowing the function the flexibility to enforce a typing structure before it knows the specific type it needs to enforce. This also adds a better developer experience when using the function in an IDE like Visual Studio Code, which will create suggestions for the keys parameter based on the object you provided. This is shown in the following screenshot:





With an idea of how generics are created in TypeScript, you can now move on to exploring using generics in specific situations. This tutorial will first cover how generics can be used in functions.


# Using Generics with Functions


One of the most common scenarios for using generics with functions is when you have some code that is not easily typed for all use cases. In order to make the function apply to more situations, you can include generic typing. In this step, you will run through an example of an identity function to illustrate this. You will also explore an asynchronous example of when to pass type parameters into your generic directly, and how to create constraints and default values for your generic type parameters.


## Assigning Generic Parameters


Take a look at the following function, which returns what was passed in as the first argument:


```
function identity(value) {
  return value;
}

```


You could add the following code to make the function type-safe in TypeScript:


```
function identity<T>(value: T): T {
  return value;
}

```


You turned your function into a generic function that accepts the generic type parameter T, which is the type of the first argument, then set the return type to be the same with : T.


Next, add the following code to try out the function:


```
function identity<T>(value: T): T {
  return value;
}

const result = identity(123);

```


result has the type 123, which is the exact number that you passed in. TypeScript here is inferring the generic type from the calling code itself. This way the calling code does not need to pass any type parameters. You can also be explicit and set the generic type parameters to the type you want:


```
function identity<T>(value: T): T {
  return value;
}

const result = identity<number>(123);

```


In this code, result has the type number. By passing in the type with the <number> code, you are explicitly letting TypeScript know that you want the generic type parameter T of the identity function to be of type number. This will enforce the number type as the argument and the return value.


## Passing Type Parameters Directly


Passing type parameters directly is also useful when using custom types. For example, take a look at the following code:


```
type ProgrammingLanguage = {
  name: string;
};

function identity<T>(value: T): T {
  return value;
}

const result = identity<ProgrammingLanguage>({ name: "TypeScript" });

```


In this code, result has the custom type ProgrammingLanguage because it is passed in directly to the identity function. If you did not include the type parameter explicitly, result would have the type { name: string } instead.


Another example that is common when working with JavaScript is using a wrapper function to retrieve data from an API:


```
async function fetchApi(path: string) {
  const response = await fetch(`https://example.com/api${path}`)
  return response.json();
}

```


This asynchronous function takes a URL path as an argument, uses the fetch API to make a request to the URL, then returns a JSON response value. In this case, the return type of the fetchApi function is going to be Promise<any>, which is the return type of the json() call on the fetch’s response object.


Having any as a return type is not very helpful. any means any JavaScript value, and by using it you are losing static type-checking, one of the main benefits of TypeScript. If you know that the API is going to return an object in a given shape, you can make this function type-safe by using generics:


```
async function fetchApi<ResultType>(path: string): Promise<ResultType> {
  const response = await fetch(`https://example.com/api${path}`);
  return response.json();
}

```


The highlighted code turns your function into a generic function that accepts the ResultType generic type parameter. This generic type is used in the return type of your function: Promise<ResultType>.



Note: As your function is async, you must return a Promise object. The TypeScript Promise type is itself a generic type that accepts the type of the value the promise resolves to.

If you take a closer look at your function, you will see that the generic is not being used in the argument list or any other place that TypeScript would be able to infer its value. This means that the calling code must explicitly pass a type for this generic when calling your function.


Here is a possible implementation of the fetchApi generic function to retrieve user data:


```
type User = {
  name: string;
}

async function fetchApi<ResultType>(path: string): Promise<ResultType> {
  const response = await fetch(`https://example.com/api${path}`);
  return response.json();
}

const data = await fetchApi<User[]>('/users')

export {}

```


In this code, you are creating a new type called User and using an array of that type (User[]) as the type for the ResultType generic parameter. The data variable now has the type User[] instead of any.



Note: As you are using await to asynchronously process the result of your function, the return type is going to be the type of T in Promise<T>, which in this case is the generic type ResultType.

## Default Type Parameters


Creating your generic fetchApi function like you are doing, the calling code always has to provide the type parameter. If the calling code does not include the generic type, ResultType would be bound to unknown. Take for example the following implementation:


```
async function fetchApi<ResultType>(path: string): Promise<ResultType> {
  const response = await fetch(`https://example.com/api${path}`);
  return 
response.json();
}

const data = await fetchApi('/users')

console.log(data.a)

export {}

```


This code tries to access a theoretical a property of data. But since the type of data is unknown, this code will not be able to access a property of the object.


If you are not planning to add a specific type to every single call of your generic function, you can add a default type to the generic type parameter. This can be done by adding = DefaultType right after the generic type, like this:


```
async function fetchApi<ResultType = Record<string, any>>(path: string): Promise<ResultType> {
  const response = await fetch(`https://example.com/api${path}`);
  return response.json();
}

const data = await fetchApi('/users')

console.log(data.a)

export {}

```


With this code, there is no longer a need for you to pass a type to the ResultType generic parameter when calling the fetchApi function, as it has a default type of Record<string, any>. This means TypeScript will recognize data as an object with keys of type string and values of type any, allowing you to access its properties.


## Type Parameters Constraints


In some situations, a generic type parameter needs to allow only certain shapes to be passed into the generic. To create this additional layer of specificity to your generic, you can put constraints on your parameter.


Imagine you have a storage constraint where you are only allowed to store objects that have string values for all their properties. For that, you can create a function that takes any object and returns another object with the same keys as the original one, but with all their values transformed to strings. This function will be called stringifyObjectKeyValues.


This function is going to be a generic function. This way, you are able to make the resulting object have the same shape as the original object. The function will look like this:


```
function stringifyObjectKeyValues<T extends Record<string, any>>(obj: T) {
  return Object.keys(obj).reduce((acc, key) =>  ({
    ...acc,
    [key]: JSON.stringify(obj[key])
  }), {} as { [K in keyof T]: string })
}

```


In this code, stringifyObjectKeyValues uses the reduce array method to iterate over an array of the original keys, stringifying the values and adding them to a new array.


To make sure the calling code is always going to pass an object to your function, you are using a type constraint on the generic type T, as shown in the following highlighted code:


```
function stringifyObjectKeyValues<T extends Record<string, any>>(obj: T) {
  // ...
}

```


extends Record<string, any> is known as generic type constraint, and it allows you to specify that your generic type must be assignable to the type that comes after the extends keyword. In this case, Record<string, any> indicates an object with keys of type string and values of type any. You can make your type parameter extend any valid TypeScript type.


When calling reduce, the return type of the reducer function is based on the initial value of the accumulator. The {} as { [K in keyof T]: string } code sets the type of the initial value of the accumulator to { [K in keyof T]: string } by using a type cast on an empty object, {}. The type { [K in keyof T]: string } creates a new type with the same keys as T, but with all the values set to have the string type. This is known as a mapped type, which this tutorial will explore further in a later section.


The following code shows the implementation of your stringifyObjectKeyValues function:


```
function stringifyObjectKeyValues<T extends Record<string, any>>(obj: T) {
  return Object.keys(obj).reduce((acc, key) =>  ({
    ...acc,
    [key]: JSON.stringify(obj[key])
  }), {} as { [K in keyof T]: string })
}

const stringifiedValues = stringifyObjectKeyValues({ a: "1", b: 2, c: true, d: [1, 2, 3]})

```


The variable stringifiedValues will have the following type:


```
{
  a: string;
  b: string;
  c: string;
  d: string;
}

```


This will ensure that the return value is consistent with the purpose of the function.


This section covered multiple ways to use generics with functions, including directly assigning type parameters and making defaults and constraints to the parameter shape. Next, you’ll run through some examples of how generics can make interfaces and classes apply to more situations.


# Using Generics with Interfaces, Classes, and Types


When creating interfaces and classes in TypeScript, it can be useful to use generic type parameters to set the shape of the resulting objects. For example, a class could have properties of different types depending on what is passed in to the constructor. In this section, you will see the syntax for declaring generic type parameters in classes and interfaces and examine a common use case in HTTP applications.


## Generic Interfaces and Classes


To create a generic interface, you can add the type parameters list right after the interface name:


```
interface MyInterface<T> {
  field: T
}

```


This declares an interface that has a property field whose type is determined by the type passed in to T.


For classes, it’s almost the same syntax:


```
class MyClass<T> {
  field: T
  constructor(field: T) {
    this.field = field
  }
}

```


One common use case of generic interfaces/classes is for when you have a field whose type depends on how the client code is using the interface/class. Say you have an HttpApplication class that is used to handle HTTP requests to your API, and that some context value is going to be passed around to every request handler. One such way to do this would be:


```
class HttpApplication<Context> {
  context: Context
	constructor(context: Context) {
    this.context = context;
  }

  // ... implementation

  get(url: string, handler: (context: Context) => Promise<void>): this {
    // ... implementation
    return this;
  }
}

```


This class stores a context whose type is passed in as the type of the argument for the handler function in the get method. During usage, the parameter type passed to the get handler would correctly be inferred from what is passed to the class constructor.


```
...
const context = { someValue: true };
const app = new HttpApplication(context);

app.get('/api', async () => {
  console.log(context.someValue)
});

```


In this implementation, TypeScript will infer the type of context.someValue as boolean.


## Generic Types


Having now gone through some examples of generics in classes and interfaces, you can now move on to making generic custom types. The syntax for applying generics to types is similar to how they are applied to interfaces and classes. Take a look at the following code:


```
type MyIdentityType<T> = T

```


This generic type returns the type that is passed as the type parameter. Imagine you implemented this type with the following code:


```
...
type B = MyIdentityType<number>

```


In this case, the type B would be of type number.


Generic types are commonly used to create helper types, especially when using mapped types. TypeScript provides many pre-built helper types. One such example is the Partial type, which takes a type T and returns another type with the same shape as T, but with all their fields set to optional. The implementation of Partial looks like this:


```
type Partial<T> = {
  [P in keyof T]?: T[P];
};

```


The type Partial here takes in a type, iterates over its property types, then returns them as optional in a new type.



Note: Since Partial is already built in to TypeScript, compiling this code into your TypeScript environment would re-declare Partial and throw an error. The implementation of Partial cited here is only for illustrative purposes.

To see how powerful generic types are, imagine that you have an object literal that stores the shipping costs from a store to all other stores in your business distribution network. Each store will be identified by a three-character code, like this:


```
{
  ABC: {
    ABC: null,
    DEF: 12,
    GHI: 13,
  },
  DEF: {
    ABC: 12,
    DEF: null,
    GHI: 17,
  },
  GHI: {
    ABC: 13,
    DEF: 17,
    GHI: null,
  },
}

```


This object is a collection of objects that represent the store location. Within each store location, there are properties that represent the cost to ship to other stores. For example, the cost to ship from ABC to DEF is 12. The shipping cost from one store to itself is null, as there will be no shipping at all.


To ensure that locations for other stores have a consistent value and that a store shipping to itself is always null, you can create a generic helper type:


```
type IfSameKeyThanParentTOtherwiseOtherType<Keys extends string, T, OtherType> = {
  [K in Keys]: {
    [SameThanK in K]: T;
  } &
    { [OtherThanK in Exclude<Keys, K>]: OtherType };
};

```


The type IfSameKeyThanParentTOtherwiseOtherType receives three generic types. The first one, Keys, are all the keys you want to make sure your object has. In this case, it is a union of all the stores’ codes. T is the type for when the nested object field has the same key as the key on the parent object, which in this case represents a store location shipping to itself. Finally, OtherType is the type for when the key is different, representing a store shipping to another store.


You can use it like this:


```
...
type Code = 'ABC' | 'DEF' | 'GHI'

const shippingCosts: IfSameKeyThanParentTOtherwiseOtherType<Code, null, number> = {
  ABC: {
    ABC: null,
    DEF: 12,
    GHI: 13,
  },
  DEF: {
    ABC: 12,
    DEF: null,
    GHI: 17,
  },
  GHI: {
    ABC: 13,
    DEF: 17,
    GHI: null,
  },
}

```


This code is now enforcing the type shape. If you set any of the keys to an invalid value, TypeScript will give us an error:


```
...
const shippingCosts: IfSameKeyThanParentTOtherwiseOtherType<Code, null, number> = {
  ABC: {
    ABC: 12,
    DEF: 12,
    GHI: 13,
  },
  DEF: {
    ABC: 12,
    DEF: null,
    GHI: 17,
  },
  GHI: {
    ABC: 13,
    DEF: 17,
    GHI: null,
  },
}

```


Since the shipping cost between ABC and itself is no longer null, TypeScript will throw the following error:


```
OutputType 'number' is not assignable to type 'null'.(2322)

```


You have now tried out using generics in interfaces, classes, and custom helper types. Next, you will explore further a topic that has already come up a few times in this tutorial: creating mapped types with generics.


# Creating Mapped Types with Generics


When working with TypeScript, there are times when you will need to create a type that should have the same shape as another type. This means that it should have the same properties, but with the type of the properties set to something different. For this situation, using mapped types can reuse the initial type shape and reduce repeated code in your application.


In TypeScript, this structure is known as a mapped type and relies on generics. In this section, you will see how to create a mapped type.


Imagine you want to create a type that, given another type, returns a new type where all the properties are set to have a boolean value. You could create this type with the following code:


```
type BooleanFields<T> = {
  [K in keyof T]: boolean;
}

```


In this type, you are using the syntax [K in keyof T] to specify the properties that the new type will have. The keyof T operator is used to return a union with the name of all the properties available in T. You are then using the K in syntax to designate that the properties of the new type are all the properties currently available in the union type returned by keyof T.


This creates a new type called K, which is bound to the name of the current property. This can be used to access the type of this property in the original type using the syntax T[K]. In this case, you are setting the type of the properties to be a boolean.


One usage scenario for this BooleanFields type is creating an options object. Imagine you have a database model, like a User. When getting a record for this model from the database, you will also allow passing an object that specifies which fields to return. This object would have the same properties as the model, but with the type set to a boolean. Passing true in a field means you want it to be returned and false that you want it to be omitted.


You could use your BooleanFields generic on the existing model type to return a new type with the same shape as the model, but with all the fields set to have a boolean type, like in the following highlighted code:


```
type BooleanFields<T> = {
  [K in keyof T]: boolean;
};

type User = {
  email: string;
  name: string;
}

type UserFetchOptions = BooleanFields<User>;

```


In this example, UserFetchOptions would be the same as creating it like this:


```
type UserFetchOptions = {
  email: boolean;
  name: boolean;
}

```


When creating mapped types, you can also provide modifiers for the fields. One such example is the existing generic type available in TypeScript called Readonly<T>. The Readonly<T> type returns a new type where all the properties of the passed type are set to be readonly properties. The implementation of this type looks like this:


```
type Readonly<T> = {
  readonly [K in keyof T]: T[K]
}

```



Note: Since Readonly is already built in to TypeScript, compiling this code into your TypeScript environment would re-declare Readonly and throw an error. The implementation of Readonly cited here is only for illustrative purposes.

Notice the modifier readonly that is added as a prefix to the [K in keyof T] part in this code. Currently, the two available modifiers that can be used in mapped types are the readonly modifier, which must be added as a prefix to the property, and the ? modifier, which can be added as a suffix to the property. The ? modifier marks the field as optional. Both modifiers can receive a special prefix to specify if the modifier should be removed (-) or added (+). If only the modifier is provided, + is assumed.


Now that you can use mapped types to create new types based on type shapes that you’ve already created, you can move on to the final use case for generics: conditional typing.


# Creating Conditional Types with Generics


In this section, you will try out another helpful feature of generics in TypeScript: creating conditional types. First, you will run through the basic structure of conditional typing. Then you will explore an advanced use case by creating a conditional type that omits nested fields of an object type based on dot notation.


## Basic Structure of Conditional Typing


Conditional types are generic types that have a different resulting type depending on some condition. For example, take a look at the following generic type IsStringType<T>:


```
type IsStringType<T> = T extends string ? true : false;

```


In this code, you are creating a new generic type called IsStringType that receives a single type parameter, T. Inside the definition of your type, you are using a syntax that looks like a conditional expression using the ternary operator in JavaScript: T extends string ? true : false. This conditional expression is checking if the type T extends the type string. If it does, the resulting type will be the exact type true; otherwise, it will be set to the type false.



Note: This conditional expression is evaluated during compilation. TypeScript only works with types, so make sure to always read the identifiers within a type declaration as types, not as values. In this code, you are using the exact type of each boolean value, true and false.

To try out this conditional type, pass some types as its type parameter:


```
type IsStringType<T> = T extends string ? true : false;

type A = "abc";
type B = {
  name: string;
};

type ResultA = IsStringType<A>;
type ResultB = IsStringType<B>;

```


In this code, you are creating two types, A and B. The type A is the type of the string literal "abc", while the type B is the type of an object that has a property called name of type string. You are then using both types with your IsStringType conditional type and storing the resulting type into two new types, ResultA and ResultB.


If you check the resulting type of ResultA and ResultB, you will notice that the ResultA type is set to the exact type true and that the ResultB type is set to false. This is correct, as A does extend the string type and B does not extend the string type, as it is set to the type of an object with a single name property of type string.


One useful feature of conditional types is that it allows you to infer type information inside the extends clause using the special keyword infer. This new type can then be used in the true branch of the condition. One possible usage for this feature is retrieving the return type of any function type.


Write the following GetReturnType type to illustrate this:


```
type GetReturnType<T> = T extends (...args: any[]) => infer U ? U : never;

```


In this code, you are creating a new generic type, which is a conditional type called GetReturnType. This generic type accepts a single type parameter, T. Inside the type declaration itself, you are checking if the type T extends a type matching a function signature that accepts a variadic number of arguments (including zero), and you are then inferring the return type of that function creating a new type U, which is available to be used inside the true branch of the condition. The type of U is going to be bound to the type of the return value of the passed function. If the passed type T is not a function, then the code will return the type never.


Use your type with the following code:


```
type GetReturnType<T> = T extends (...args: any[]) => infer U ? U : never;

function someFunction() {
  return true;
}

type ReturnTypeOfSomeFunction = GetReturnType<typeof someFunction>;

```


In this code, you are creating a function called someFunction, which returns true. You are then using the typeof operator to pass in the type of this function to the GetReturnType generic and storing the resulting type in the ReturnTypeOfSomeFunction type.


As the type of your someFunction variable is a function, the conditional type would evaluate the true branch of the condition. This will return the type U as the result. The type U was inferred from the return type of the function, which in this case is a boolean. If you check the type of ReturnTypeOfSomeFunction, you will find that it is correctly set to have the boolean type.


## Advanced Conditional Type Use Case


Conditional types are one of the most flexible features available in TypeScript and allow for the creation of some advanced utility types. In this section, you will explore one of these use cases by creating a conditional type called NestedOmit<T, KeysToOmit>. This utility type will be able to omit fields from an object, just like the existing Omit<T, KeysToOmit> utility type, but will also allow omitting nested fields by using dot notation.


Using your new NestedOmit<T, KeysToOmit> generic, you will be able to use the type as shown in the following example:


```
type SomeType = {
  a: {
    b: string,
    c: {
      d: number;
      e: string[]
    },
    f: number
  }
  g: number | string,
  h: {
    i: string,
    j: number,
  },
  k: {
    l: number,<F3>
  }
}

type Result = NestedOmit<SomeType, "a.b" | "a.c.e" | "h.i" | "k">;

```


This code declares a type named SomeType that has a multi-level structure of nested properties. Using your NestedOmit generic, you pass in the type, then list the keys of the properties you’d like to omit. Notice how you can use dot notation in the second type parameter to identify the keys to omit. The resulting type is then stored in Result.


Constructing this conditional type will use many features available in TypeScript, like template literal types, generics, conditional types, and mapped types.


To try out this generic, start by creating a generic type called NestedOmit that accepts two type parameters:


```
type NestedOmit<T extends Record<string, any>, KeysToOmit extends string>

```


The first type parameter is called T, which must be a type that is assignable to the Record<string, any> type. This will be the type of the object you want to omit properties from. The second type parameter is called KeysToOmit, which must be of type string. You will use this to specify the keys you want to omit from your type T.


Next, check if KeysToOmit is assignable to the type ${infer KeyPart1}.${infer KeyPart2} by adding the following highlighted code:


```
type NestedOmit<T extends Record<string, any>, KeysToOmit extends string> =
  KeysToOmit extends `${infer KeyPart1}.${infer KeyPart2}`

```


Here, you are using a template literal string type while taking advantage of conditional types to infer two other types inside the template literal itself. By inferring two parts of the template literal string type, you are splitting the string into two other strings. The first part will be assigned to the type KeyPart1 and will contain everything before the first dot. The second part will be assigned to the type KeyPart2 and will contain everything after the first dot. If you passed "a.b.c" as the KeysToOmit, initially KeyPart1 would be set to the exact string type "a", and KeyPart2 would be set to "b.c".


Next you will add the ternary operator to define the first true branch of the condition:


```
type NestedOmit<T extends Record<string, any>, KeysToOmit extends string> =
  KeysToOmit extends `${infer KeyPart1}.${infer KeyPart2}`
    ?
      KeyPart1 extends keyof T

```


This uses KeyPart1 extends keyof T to check if KeyPart1 is a valid property of the given type T. In case you do have a valid key, add the following code to make the condition evaluate to an intersection between the two types:


```
type NestedOmit<T extends Record<string, any>, KeysToOmit extends string> =
  KeysToOmit extends `${infer KeyPart1}.${infer KeyPart2}`
    ?
      KeyPart1 extends keyof T
      ?
        Omit<T, KeyPart1>
        & {
          [NewKeys in KeyPart1]: NestedOmit<T[NewKeys], KeyPart2>
        }

```


Omit<T, KeyPart1> is a type built by using the Omit helper shipped by default with TypeScript. At this point, KeyPart1 is not in dot notation: It is going to contain the exact name of a field that contains nested fields that you want to omit from the original type. Because of this, you can safely use the existing utility type.


You are using Omit to remove some nested fields that are inside T[KeyPart1], and to do that, you have to rebuild the type of T[KeyPart1]. To avoid rebuilding the whole T type, you use Omit to remove just KeyPart1 from T, preserving other fields. Then you are rebuilding T[KeyPart1] in the type in the next part.


[NewKeys in KeyPart1]: NestedOmit<T[NewKeys], KeyPart2> is a mapped type where the properties are the ones assignable to KeyPart1, which means the part you just extracted from KeysToOmit. This is the parent of the fields you want to remove. If you passed a.b.c, during the first evaluation of your condition it would be NewKeys in "a". You are then setting the type of this property to be the result of recursively calling your NestedOmit utility type, but now passing as the first type parameter the type of this property inside T by using T[NewKeys], and passing as the second type parameter the remaining keys in dot notation, available in KeyPart2.


In the false branch of the internal condition, you return the current type bound to T, as if KeyPart1 is not a valid key of T:


```
type NestedOmit<T extends Record<string, any>, KeysToOmit extends string> =
  KeysToOmit extends `${infer KeyPart1}.${infer KeyPart2}`
    ?
      KeyPart1 extends keyof T
      ?
        Omit<T, KeyPart1>
        & {
          [NewKeys in KeyPart1]: NestedOmit<T[NewKeys], KeyPart2>
        }
      : T

```


This branch of the conditional means you are trying to omit a field that does not exist in T. In this case, there is no need to go any further.


Finally, in the false branch of the outer condition, use the existing Omit utility type to omit KeysToOmit from Type:


```
type NestedOmit<T extends Record<string, any>, KeysToOmit extends string> =
  KeysToOmit extends `${infer KeyPart1}.${infer KeyPart2}`
    ?
      KeyPart1 extends keyof T
      ?
        Omit<T, KeyPart1>
        & {
          [NewKeys in KeyPart1]: NestedOmit<T[NewKeys], KeyPart2>
        }
      : T
    : Omit<T, KeysToOmit>;

```


If the condition KeysToOmit extends `${infer KeyPart1}.${infer KeyPart2}` is false, it means KeysToOmit is not using dot notation, and thus you can use the existing Omit utility type.


Now, to use your new NestedOmit conditional type, create a new type called NestedObject:


```
type NestedObject = {
  a: {
    b: {
      c: number;
      d: number;
    };
    e: number;
  };
  f: number;
};

```


Then call NestedOmit on it to omit the nested field available at a.b.c:


```
type Result = NestedOmit<NestedObject, "a.b.c">;

```


On the first evaluation of the conditional type, the outer condition would be true, as the string literal type "a.b.c" is assignable to the template literal type `${infer KeyPart1}.${infer KeyPart2}`. In this case, KeyPart1 would be inferred as the string literal type "a" and KeyPart2 would be inferred as the remaining of the string, in this case "b.c".


The inner condition is now going to be evaluated. This will evaluate to true, as KeyPart1 at this point is a key of T. KeyPart1 now is "a", and T does have a property "a":


```
type NestedObject = {
  a: {
    b: {
      c: number;
      d: number;
    };
    e: number;
  };
  f: number;
};

```


Moving forward with the evaluation of the condition, you are now inside the inner true branch. This builds a new type that is an intersection of two other types. The first type is the result of using the Omit utility type on T to omit the fields that are assignable to KeyPart1, in this case the a field. The second type is a new type you are building by calling NestedOmit recursively.


If you go through the next evaluation of NestedOmit, for the first recursive call, the intersection type is now building a type to use as the type of the a field. This recreates the a field without the nested fields you need to omit.


In the final evaluation of NestedOmit, the first condition would return false, as the passed string type is just "c" now. When this happens, you omit the field from the object with the built-in helper. This would return the type for the b field, which is the original type with c omitted. The evaluation now ends and TypeScript returns the new type you want to use, with the nested field omitted.


# Conclusion


In this tutorial, you explore generics as they apply to functions, interfaces, classes, and custom types. You also used generics to create mapped and conditional types. Each of these makes generics a powerful tool you have at your disposal when using TypeScript. Using them correctly will save you from repeating code over and over again, and will make the types you have written more flexible. This is especially true if you are a library author and are planning to make your code legible for a wide audience.


For more tutorials on TypeScript, check out our How To Code in TypeScript series page.


