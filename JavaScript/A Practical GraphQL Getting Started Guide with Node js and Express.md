# A Practical GraphQL Getting Started Guide with Node js and Express

```JavaScript``` ```NoSQL``` ```Node.js```

## Introduction


GraphQL is a query language created by Facebook with the purpose of building client applications based on intuitive and flexible syntax for describing their data requirements and interactions. A GraphQL service is created by defining types and fields on those types, then providing functions for each field on each type.


Once a GraphQL service is running (typically at a URL on a web service), it can receive GraphQL queries to validate and execute. A received query is first checked to ensure it only refers to the types and fields defined, then runs the provided functions to produce a result.


In this tutorial, we are going to implement a GraphQL server using Express and use it to learn important GraphQL features.





Some of GraphQL features include:


- 
Hierarchical - Queries look exactly like the data they return.

- 
Client-specified queries - The client has the liberty to dictate what to fetch from the server.

- 
Strongly typed - You can validate a query syntactically and within the GraphQL type system before execution. This also helps leverage powerful tools that improve the development experience, such as GraphiQL.

- 
Introspective - You can query the type system using the GraphQL syntax itself. This is great for parsing incoming data into strongly-typed interfaces, and not having to deal with parsing and manually transforming JSON into objects.


## Goals


One of the primary challenges with traditional REST calls is the inability of the client to request a customized (limited or expanded) set of data. In most cases, once the client requests information from the server, it either gets all or none of the fields.


Another difficulty is working and maintaining multiple endpoints. As a platform grows, consequently the number will increase. Therefore, clients often need to ask for data from different endpoints. GraphQL APIs are organized in terms of types and fields, not endpoints. You can access the full capabilities of your data from a single endpoint.


When building a GraphQL server, it is only necessary to have one URL for all data fetching and mutating. Thus, a client can request a set of data by sending a query string, describing what they want, to a server.


# Prerequisites


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.

# Step 1 — Setting Up GraphQL with Node


You’ll start by creating a basic file structure and a sample code snippet.


First, create a GraphQL directory:


```
mkdir GraphQL


```


Change into the new directory:


```
cd GraphQL


```


Initialize an npm project:


```
npm init -y


```


Then create the server.js file which will be the main file:


```
touch server.js


```


Your project should resemble the following:





Necessary packages will be discussed in this tutorial as they are implemented. Next, set up a server using Express and express-graphql, an HTTP server middleware:


```
npm install graphql express express-graphql


```


Open server.js in a text editor and add the following lines of code:


server.js
```
var express = require('express');
var graphqlHTTP = require('express-graphql');
var { buildSchema } = require('graphql');

// Initialize a GraphQL schema
var schema = buildSchema(`
  type Query {
    hello: String
  }
`);

// Root resolver
var root = { 
  hello: () => 'Hello world!'
};

// Create an express server and a GraphQL endpoint
var app = express();
app.use('/graphql', graphqlHTTP({
  schema: schema,  // Must be provided
  rootValue: root,
  graphiql: true,  // Enable GraphiQL when server endpoint is accessed in browser
}));
app.listen(4000, () => console.log('Now browse to localhost:4000/graphql'));

```



Note: This code was written with an earlier version of express-graphql. Prior to v0.10.0, you could use var graphqlHTTP = require('express-graphql');. After v0.10.0, you need to use var { graphqlHTTP } = require('express-graphql');.

This snippet accomplishes several things. It uses require to include the packages that were installed. It also initializes generic schema and root values. Furthermore, it creates an endpoint at /graphql which can be visited with a web browser.


Save and close the file after making these changes.


Start the Node server if it is not running:


```
node server.js


```



Note: Throughout this tutorial you will be making updates to server.js that will require restarting the node server to reflect the latest changes.

Visit localhost:4000/graphql in a web browser. You will see a Welcome to GraphiQL web interface.


There will be a pane on the left where you will be entering queries. There is an additional pane for entering query variables that you may need to drag and resize to view. The pane on the right will display the results of executing your queries. Furthermore, executing queries can be accomplished by pressing the button with the play icon.





So far we have explored some features and advantages of GraphQL. In this next section, we’ll delve into different terminologies and implementations of some technical features in GraphQL. We’ll be using an Express server to practice these features.


# Step 2 — Defining a Schema


In GraphQL, the Schema manages queries and mutations, defining what is allowed to be executed in the GraphQL server. A schema defines a GraphQL API’s type system. It describes the complete set of possible data (objects, fields, relationships, etc.) that a client can access. Calls from the client are validated and executed against the schema. A client can find information about the schema via introspection. A schema resides on the GraphQL API server.


GraphQL Interface Definition Language (IDL) or Schema Definition Language (SDL) are the most concise ways to specify a GraphQL Schema. The most basic components of a GraphQL schema are object types, which represent a kind of object we can fetch from our service, and what fields it has.


In the GraphQL schema language, you might represent a user with an id, name, and age like this example:


```
type User {
  id: ID!
  name: String!
  age: Int
}

```


In JavaScript, you would use the buildSchema function which builds a Schema object from GraphQL schema language. If you were to represent the same user above, it would look like this example:


```
var schema = buildSchema(`
  type User {
    id: Int
    name: String!
    age: Int
  }
`);

```


## Constructing Types


You can define different types inside buildSchema, which you might notice in most cases are type Query  {...} and type Mutation {...}. type Query {...} is an object holding the functions that will be mapped to GraphQL queries, used to fetch data (equivalent to GET in REST). type Mutation {...} holds functions that will be mapped to mutations, used to create, update, or delete data (equivalent to POST, UPDATE, and DELETE in REST).


You’ll make your schema a bit complex by adding some reasonable types. For instance, you want to return a user and an array of users of type Person, who have an id, name, age, and their favorite shark properties.


Replace the pre-existing lines of code for schema in server.js with this new Schema object:


server.js
```
// Initialize a GraphQL schema
var schema = buildSchema(`
  type Query {
    user(id: Int!): Person
    users(shark: String): [Person]
  },
  type Person {
    id: Int
    name: String
    age: Int
    shark: String
  }
`);

```


You may notice some interesting syntax above, [Person] means return an array of type Person while the exclamation in user(id: Int!) means that the id must be provided. users query takes an optional shark variable.


# Step 3 — Defining Resolvers


A resolver is responsible for mapping the operation to an actual function. Inside type Query, you have an operation called users. You map this operation to a function with the same name inside root.


You’ll also create some sample users for this functionality.


Add these new lines of code to server.js right after the buildSchema lines of code, but before the root lines of code:


server.js
```
...
// Sample users
var users = [
  {
    id: 1,
    name: 'Brian',
    age: '21',
    shark: 'Great White Shark'
  },
  {
    id: 2,
    name: 'Kim',
    age: '22',
    shark: 'Whale Shark'
  },
  {
    id: 3,
    name: 'Faith',
    age: '23',
    shark: 'Hammerhead Shark'
  },
  {
    id: 4,
    name: 'Joseph',
    age: '23',
    shark: 'Tiger Shark'
  },
  {
    id: 5,
    name: 'Joy',
    age: '25',
    shark: 'Hammerhead Shark'
  }
];

// Return a single user
var getUser = function(args) {
  // ...
}

// Return a list of users
var retrieveUsers = function(args) { 
  // ...
}
...

```


Replace the pre-existing lines of code for root in server.js with this new object:


server.js
```
// Root resolver
var root = { 
  user: getUser,  // Resolver function to return user with specific id
  users: retrieveUsers
};

```


To make the code more readable, create separate functions instead of piling everything in the root resolver. Both functions take an optional args parameter which carries variables from the client query. Let’s provide an implementation for the resolvers and test their functionality.


Replace the lines of code for getUser and retrieveUsers that you added earlier to server.js with the following:


server.js
```
// Return a single user (based on id)
var getUser = function(args) {
  var userID = args.id;
  return users.filter(user => user.id == userID)[0];
}

// Return a list of users (takes an optional shark parameter)
var retrieveUsers = function(args) {
  if (args.shark) {
    var shark = args.shark;
    return users.filter(user => user.shark === shark);
  } else {
    return users;
  }
}

```


In the web interface, enter the following query in the input pane:


```
query getSingleUser {
  user {
    name
    age
    shark
  }
}

```


You will receive the following output:


```
Output{
  "errors": [
    {
      "message": "Cannot query field \"user\" on type \"Query\".",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ]
    }
  ]
}

```


In the example above, we are using an operation named getSingleUser to get a single user with their name, age, and favorite shark. We could optionally specify that we need their name only if we did not need the age and shark.


According to the official documentation, it is easiest to identify queries in your codebase by name instead of by deciphering the contents.


This query does not provide the required id and GraphQL gives us a descriptive error message. We’ll now make a correct query. Take notice of the use of variables and arguments.


In the web interface, replace the content of the input pane with the following corrected query:


```
query getSingleUser($userID: Int!) {
  user(id: $userID) {
    name
    age
    shark
  }
}

```


While still in the web interface, replace the content of the variables pane with the following:


```
Query Variables{
  "userID": 1
}

```


You will receive the following output:


```
Output{
  "data": {
    "user": {
      "name": "Brian",
      "age": 21,
      "shark": "Great White Shark"
    }
  }
}

```


This returns a single user matching the id of 1, Brian. It also returns the requested name, age, and shark fields.


# Step 4 — Defining Aliases


In a situation where you need to retrieve two different users, you may be wondering how you would identify each user. In GraphQL, you can’t directly query for the same field with different arguments. Let’s demonstrate this.


In the web interface, replace the content of the input pane with the following:


```
query getUsersWithAliasesError($userAID: Int!, $userBID: Int!) {
  user(id: $userAID) {
    name
    age
    shark
  },
  user(id: $userBID) {
    name
    age
    shark
  }
}

```


While still in the web interface, replace the content of the variables pane with the following:


```
Query Variables{
  "userAID": 1,
  "userBID": 2
}

```


You will receive the following output:


```
Output{
  "errors": [
    {
      "message": "Fields \"user\" conflict because they have differing arguments. Use different aliases on the fields to fetch both if this was intentional.",
      "locations": [
        {
          "line": 2,
          "column": 3
        },
        {
          "line": 7,
          "column": 3
        }
      ]
    }
  ]
}

```


The error is descriptive and even suggests the use of aliases. Let’s correct the implementation.


In the web interface, replace the content of the input pane with the following corrected query:


```
query getUsersWithAliases($userAID: Int!, $userBID: Int!) {
  userA: user(id: $userAID) {
    name
    age
    shark
  },
  userB: user(id: $userBID) {
    name
    age
    shark
  }
}

```


While still in the web interface, ensure the variables pane contains the following:


```
Query Variables{
  "userAID": 1,
  "userBID": 2
}

```


You will receive the following output:


```
Output{
  "data": {
    "userA": {
      "name": "Brian",
      "age": 21,
      "shark": "Great White Shark"
    },
    "userB": {
      "name": "Kim",
      "age": 22,
      "shark": "Whale Shark"
    }
  }
}

```


Now we can correctly identify each user with their fields.


# Step 5 — Creating Fragments


The query above is not that bad, but it has one problem; we are repeating the same fields for both userA and userB. We could find something that will make our queries DRY. GraphQL includes reusable units called fragments that let you construct sets of fields, and then include them in queries where you need to.


In the web interface, replace the content of the variables pane with the following:


```
query getUsersWithFragments($userAID: Int!, $userBID: Int!) {
  userA: user(id: $userAID) {
    ...userFields
  },
  userB: user(id: $userBID) {
    ...userFields
  }
}

fragment userFields on Person {
  name
  age
  shark
}

```


While still in the web interface, ensure the variables pane contains the following:


```
Query Variables{ 
  "userAID": 1,
  "userBID": 2
}

```


You will receive the following output:


```
Output{
  "data": {
    "userA": {
      "name": "Brian",
      "age": 21,
      "shark": "Great White Shark"
    },
    "userB": {
      "name": "Kim",
      "age": 22,
      "shark": "Whale Shark"
    }
  }
}

```


You have created a fragment called userFields that can only be applied on type Person and then used it to retrieve users.


# Step 6 — Defining Directives


Directives enable us to dynamically change the structure and shape of our queries using variables. At some point, you might want to skip or include some fields without altering the schema. The two available directives are as follows:


- @include(if: Boolean)- Only include this field in the result if the argument is true.
- @skip(if: Boolean)- Skip this field if the argument is true.

Say you want to retrieve users that are fans of Hammerhead Shark, but include their id and skip their age fields. You can use variables to pass in the shark and use directives for the inclusion and skipping functionalities.


In the web interface, clear the input pane and add the following:


```
query getUsers($shark: String, $age: Boolean!, $id: Boolean!) {
  users(shark: $shark){
    ...userFields
  }
}

fragment userFields on Person {
  name
  age @skip(if: $age)
  id @include(if: $id)
}

```


While still in the web interface, clear the variables pane and add the following:


```
Query Variables{
  "shark": "Hammerhead Shark",
  "age": true,
  "id": true
}

```


You will receive the following output:


```
Output{
  "data": {
    "users": [
      {
        "name": "Faith",
        "id": 3
      },
      {
        "name": "Joy",
        "id": 5
      }
    ]
  }
}

```


This returns two users with shark values matching Hammerhead Shark–Faith and Joy.


# Step 7 — Defining Mutations


So far we have been dealing with queries, the operations to retrieve data. Mutations are the second main operation in GraphQL which deals with creating, deleting, and updating data.


Let’s focus on some examples of how to carry out mutations. For instance, we want to update a user with id == 1 and change their age, name, and then return the new user details.


Update your schema to include a mutation type in addition to the pre-existing lines of code:


server.js
```
// Initialize a GraphQL schema
var schema = buildSchema(`
  type Query {
    user(id: Int!): Person
    users(shark: String): [Person]
  },
  type Person {
    id: Int
    name: String
    age: Int
    shark: String
  }
  # newly added code
  type Mutation {
    updateUser(id: Int!, name: String!, age: String): Person
  }
`);

```


After getUser and retrieveUsers, add a new updateUser function to handle updating a user:


server.js
```
// Update a user and return new user details
var updateUser = function({id, name, age}) {
  users.map(user => {
    if (user.id === id) {
      user.name = name;
      user.age = age;
      return user;
    }
  });
  return users.filter(user => user.id === id)[0];
}

```


Also, update the root resolver with relevant resolver functions:


server.js
```
// Root resolver
var root = { 
  user: getUser,
  users: retrieveUsers,
  updateUser: updateUser  // Include mutation function in root resolver
};

```


Assuming these are the initial user details:


```
Output{
  "data": {
    "user": {
      "name": "Brian",
      "age": 21,
      "shark": "Great White Shark"
    }
  }
}

```


In the web interface, add the following query to the input pane:


```
mutation updateUser($id: Int!, $name: String!, $age: String) {
  updateUser(id: $id, name:$name, age: $age){
    ...userFields
  }
}

fragment userFields on Person {
  name
  age
  shark
}

```


While still in the web interface, clear the variables pane and add the following:


```
Query Variables{
  "id": 1,
  "name": "Keavin",
  "age": "27"
}

```


You will receive the following output:


```
Output{
  "data": {
    "updateUser": {
      "name": "Keavin",
      "age": 27,
      "shark": "Great White Shark"
    }
  }
}

```


After a mutation to update the user, you get the new user details.


The user with the id of 1 has been updated from Brian (age 21) to Keavin (age 27).


# Conclusion


In this guide, you have covered basic concepts of GraphQL to some fairly complex examples. Most of these examples reveal the differences between GraphQL and REST for users who have interacted with REST.


To learn more about GraphQL, check the official documentation.


