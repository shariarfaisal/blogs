# How To Build a GraphQL Server in Node js Using GraphQL-yoga and MongoDB

```JavaScript``` ```Node.js``` ```MongoDB```

Most applications today have the need to fetch data from a server where that data is stored in a database. GraphQL is a new API standard that provides a more efficient, powerful and flexible alternative to REST. It allows a client fetch only the data it needs from a server.
GraphQL is often confused with being a database technology, but his is a misconception, GraphQL is a query language for APIs , not databases. In that sense, it’s database agnostic and effectively can be used in any context where an API is used.


graphql-yoga is a fully-featured GraphQL Server with focus on ease-of-use, performance, and great developer experience. Additionally, it supports all GraphQL clients like Apollo.


Now that you understand what GraphQL is, you can learn to use it in your Nodejs applications in place of REST.


# Creating a New Project


Create a new folder called UsersAPI, open your terminal and navigate into the folder then run the following command:


```
npm init


```


Create a file named index.js inside your project folder:


```
touch index.js


```


That takes care of setting up your application.


# Installing Required Dependencies


You’ll need to install graphql-yoga to help set up your GraphQL server and then mongoose for help with connecting to your Mongo database.


Inside your terminal run the following:


```
npm install graphql-yoga mongoose


```


# Defining User Model


Models are responsible for creating and reading documents from the underlying MongoDB database. Here, we’ll create a User model for our GraphQL application.


Inside index.js type in the following lines of code:


```
const { GraphQLServer } = require('graphql-yoga')
const mongoose = require('mongoose');
mongoose.connect("mongodb://localhost:27017/UserApp");


const User= mongoose.model("User",{
    fullname: String,
    username: String,
    phone_number: String,
    city: String
});

```


We have successfully defined our User model and established a mongoose connection, it is time to get into GraphQL Schemas.


# Defining a GraphQL Schema


A GraphQL Schema describes the functionality available to the clients which connect to it. It is built using the Schema Definition Language.


Add these following lines of code to your index.js file:


```
const typeDefs = `type Query {
    getUser(id: ID!): User
    getUsers: [User]
  }
  type User {
      id: ID!
      fullname: String!
      username: String!
      phone_number: String!
      city: String!
  }
  type Mutation {
      addUser(fullname: String!, username: String!, phone_number: String!, city: String!): User!,
      deleteUser(id: ID!): String
  }`

```


We have three types inside our Schema, let’s break them down.


# Using the Query Type


A GraphQL query is for fetching data and compares to the GET verb in REST-based APIs.
In order to define what queries are possible on a server, the Query type is used within the Schema Definition Language. The Query type is a root-level type which defines functionality for clients and acts as an entrypoint to other more specific types within the schema.


```
type Query {
    getUser(id: ID): User
    getUsers: [User]
}

```


In this Query type, we define two types of queries which are available on this GraphQL server:
getUser: which returns a particular User object that matches the ID provided.
getUsers: which returns a list of User objects.


If you are familiar with REST-based APIs, you would normally find these located on separate end-points, but GraphQL allows them to be queried at the same time and returned at once.



Note: Square brackets signifies that you expect an iterable object or an array in the JSON response. You will only be returning a single User object for getUser and an array for getUsers.

# Using the User Type


User  is a GraphQL Object Type, meaning it’s a type with some fields. The object type is the most common type used in a schema and represents a group of fields.


```
type User {
    id: ID!
    fullname: String!
    username: String!
    phone_number: String!
    city: String!
}

```


id, fullname, username, phone_number and city are fields on the User type.


- String is one of the built-in scalar types. It specifies the data type of a field.
- String means that the field is non-nullable. In the Schema Definition Language, we’ll represent those with an exclamation mark.
- ID is a unique identifier, often used as the key for a cache.

# Using the Mutation Type


Mutations are sent to the server to create, update or delete data similar to the PUT, POST, PATCH and DELETE verbs on RESTful APIs.
Much like how the Query type defines the entry-points for data-fetching operations on a GraphQL server, the root-level Mutation type specifies the entry points for data-manipulation operations.


```
type Mutation {
    addUser(fullname: String!, username: String!, phone_number: String!, city: String!): User!,
    deleteUser(id: ID!): String
}

```


This implements two mutations:


- addUser: which accepts fullname, username, phone_number and city as arguments known as “input types” and the mutation will return the newly-created User object.
- deleteUser: which accepts a valid User ID and returns a String message to tell if the delete operation was successful or not.

Let’s move on to writing Resolvers for our defined Schema.


# Adding Resolvers


Resolvers are the actual functions to implement business logic on your data in a GraphQL API. Each query and mutation will have corresponding resolver functions to perform the logic.


Add the following lines of code to your index.js file:


```
const resolvers = {
    Query: {
      getUsers: ()=> User.find(),
      getUser: async (_,{id}) => {
        var result = await User.findById(id);
        return result;
    }
},
    Mutation: {
        addUser: async (_, { fullname, username, phone_number, city }) => {
            const user = new User({fullname, username, phone_number, city});
            await user.save();
            return user;
        },
        deleteUser: async (_, {id}) => {
            await User.findByIdAndRemove(id);
            return "User deleted";
        }
    }
  }

```


# Setting up your GraphQL Server


After building your Schema and Resolvers, it’s time to set up your server to handle requests. graphql-yoga is the easiest way to get a GraphQL server up and running, it is a full-featured GraphQL Server with focus on user-friendly configuration, performance & great developer experience.


Inside index.js, add the following lines of code to setup your server:


```
const server = new GraphQLServer({ typeDefs, resolvers })
mongoose.connection.once("open", function(){
    server.start(() => console.log('Server is running on localhost:4000'))
});

```


With that, you’ve built a functional GraphQL server from scratch, it’s time to make some requests. Open your terminal and run the following command to start the server:


```
node index.js

```


Open your web browser and navigate to http://localhost:4000/ a nice GraphQL playground will come up on your browser where you can run sample queries.


## Adding a New User


Run the following query inside the playground to add a new user:


```
mutation{
  addUser(fullname:"Ibe Ogele",username:"ibesoft",phone_number:"2348102331921",city:"Enugu"){
    id
    fullname
    username
    phone_number
    city
  }
}

```


## Fetching User information


Run the query below to fetch information for a particular user:


```
query{
  getUser(id:"user_id"){
    id
    fullname
    username
    phone_number
    city
  }
}

```


To get all Users run:


```
query{
  getUsers{
    id
    fullname
    username
    phone_number
    city
  }
}

```


## Deleting a User


You can delete a particular User by running the query below:


```
mutation{
  deleteUser(id: "user_id")
}

```


# Conclusion


With GraphQL you send a query to your API and get exactly what you need, nothing more and nothing less. GraphQL queries always return predictable results.


