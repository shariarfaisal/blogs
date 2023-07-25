# How To Set Up a GraphQL Server in Node js with Apollo Server and Sequelize

```API``` ```Node.js``` ```SQLite```

## Introdiction


GraphQL is a specification and therefore language agnostic. When it comes GraphQL development with Node.js, there are various options available, including graphql-js, express-graphql, and apollo-server. In this tutorial, you will set up a fully featured GraphQL server in Node.js with Apollo Server.


Since the launch of Apollo Server 2, creating a GraphQL server with Apollo Server has become more efficient, not to mention the other features that came with it.


For the purpose of this demonstration, you will build a GraphQL server for a recipe app.


# Prerequisites


To complete this tutorial, you’ll need:


- A local development environment for Node.js. Follow How to Install Node.js and Create a Local Development Environment.
- Basic knowledge of GraphQL.

This tutorial was verified with Node v14.4.0, npm v6.14.5, apollo-server v2.15.0, graphql v15.1.0, sequelize v5.21.13, and sqlite3 v4.2.0.


# What is GraphQL?


GraphQL is a declarative data fetching specification and query language for APIs. It was created by Facebook. GraphQL is an effective alternative to REST, as it was created to overcome some of the shortcomings of REST-like under and over fetching.


Unlike REST, GraphQL uses one endpoint. This means we make one request to the endpoint and we’ll get one response as JSON. This JSON response can contain as little or as much data as we want. Like REST, GraphQL can be operated over HTTP, though GraphQL is protocol agnostic.


A typical GraphQL server is comprised of schema and resolvers. A schema (or GraphQL schema) contains type definitions that would make up a GraphQL API. A type definition contains field(s), each with what it is expected to return. Each field is mapped to a function on the GraphQL server called a resolver. Resolvers contain the implementation logic and return data for a field. In other words, schemas contain type definitions, while resolvers contain the actual implementations.


# Step 1 — Setting Up the Database


We’ll start by setting up our database. We’ll be using SQLite for our database. Also, we’ll be using Sequelize, which is an ORM for Node.js, to interact with our database.


First, let’s create a new project:


```
mkdir graphql-recipe-server


```


Navigate to the new project directory:


```
cd graphql-recipe-server


```


Initialize a new project:


```
npm init -y


```


Next, let’s install Sequelize:


```
npm install sequelize sequelize-cli sqlite3


```


In addition to installing Sequelize, we are also installing the sqlite3 package for Node.js. To help us scaffold our project, we’ll be using the Sequelize CLI, which we are installing as well.


Let’s scaffold our project with the CLI:


```
node_modules/.bin/sequelize init


```


This will create the following folders:


- config: contains a config file, which tells Sequelize how to connect with our database.
- models: contains all models for our project, and also contains an index.js file which integrates all the models together.
- migrations: contains all migration files.
- seeders: contains all seed files.

For the purpose of this tutorial, we won’t be using any seeders. Open config/config.json and replace it with the following content:


config/config.json
```
{
  "development": {
    "dialect": "sqlite",
    "storage": "./database.sqlite"
  }
}

```


We set the dialect to sqlite and set the storage to point to a SQLite database file.


Next, we need to create the database file directly inside the project’s root directory:


```
touch database.sqlite


```


Now the dependencies for your project are installed to use SQLite.


# Step 2 — Creating Models and Migrations


With the database setup out of the way, we can start creating the models for our project. Our recipe app will have two models: User and Recipe. We’ll be using the Sequelize CLI for this:


```
node_modules/.bin/sequelize model:create --name User --attributes name:string,email:string,password:string


```


This is will create a user.js file inside the models directory and a corresponding migration file inside the migrations directory.


Since we don’t want any fields on the User model to be nullable, we need to explicitly define that. Open migrations/XXXXXXXXXXXXXX-create-user.js and update the fields definitions as follows:


migrations/XXXXXXXXXXXXXX-create-user.js
```
name: {
  allowNull: false,
  type: Sequelize.STRING
},
email: {
  allowNull: false,
  type: Sequelize.STRING
},
password: {
  allowNull: false,
  type: Sequelize.STRING
}

```


Then we’ll do the same in the User model:


models/user.js
```
name: {
  allowNull: false,
  type: DataTypes.STRING
},
email: {
  allowNull: false,
  type: DataTypes.STRING
},
password: {
  allowNull: false,
  type: DataTypes.STRING
}

```


Next, let’s create the Recipe model:


```
node_modules/.bin/sequelize model:create --name Recipe --attributes title:string,ingredients:string,direction:string


```


Just as we did with the User model, we’ll do the same for the Recipe model. Open migrations/XXXXXXXXXXXXXX-create-recipe.js and update the fields definitions as follows:


migrations/XXXXXXXXXXXXXX-create-recipe.js
```
userId: {
  allowNull: false,
  type: Sequelize.INTEGER.UNSIGNED
},
title: {
  allowNull: false,
  type: Sequelize.STRING
},
ingredients: {
  allowNull: false,
  type: Sequelize.STRING
},
direction: {
  allowNull: false,
  type: Sequelize.STRING
},

```


You’ll notice we have an additional field: userId, which would hold the ID of the user that created a recipe. More on this shortly.


Update the Recipe model as well:


models/recipe.js
```
title: {
  allowNull: false,
  type: DataTypes.STRING
},
ingredients: {
  allowNull: false,
  type: DataTypes.STRING
},
direction: {
  allowNull: false,
  type: DataTypes.STRING
}

```


Let’s define the one-to-many relationship between the user and recipe models.


Open models/user.js and update the User.associate function as below:


models/user.js
```
User.associate = function(models) {
  // associations can be defined here
  User.hasMany(models.Recipe)
};

```


We need to also define the inverse of the relationship on the Recipe model:


models/recipe.js
```
Recipe.associate = function(models) {
  // associations can be defined here
  Recipe.belongsTo(models.User, { foreignKey: 'userId' });
};

```


By default, Sequelize will use a camelcase name from the corresponding model name and its primary key as the foreign key. So in our case, it will expect the foreign key to be UserId. Since we named the column differently, we need to explicitly define the foreignKey on the association.


Now, we can run the migrations:


```
node_modules/.bin/sequelize db:migrate


```


Now the setup for your models and migrations is complete.


# Step 3 — Creating the GraphQL Server


As mentioned earlier, we’ll be using Apollo Server for building our GraphQL server. So, let’s install it:


```
npm install apollo-server graphql bcryptjs


```


Apollo Server requires graphql as a dependency, hence the need to install it as well. Also, we install bcryptjs, which we’ll use to hash user passwords later on.


With those installed, create a src directory, then within it, create an index.js file and add the following code to it:


src/index.js
```
const { ApolloServer } = require('apollo-server');
const typeDefs = require('./schema');
const resolvers = require('./resolvers');
const models = require('../models');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: { models },
});

server
  .listen()
  .then(({ url }) => console.log('Server is running on localhost:4000'));

```


Here, we create a new instance of Apollo Server, passing to it our schema and resolvers (both of which we’ll create shortly). We also pass the models as the context to the Apollo Server. This will allow us to have access to the models from our resolvers.


Finally, we start the server.


# Step 4 — Defining the GraphQL Schema


GraphQL schema is used to define the functionality a GraphQL API would have. A GraphQL schema is comprised of types. A type can be for defining the structure of our domain-specific entity. In addition to defining types for our domain-specific entities, we can also define types for GraphQL operations, which will in turn translates to the functionality a GraphQL API will have. These operations are queries, mutations, and subscriptions. Queries are used to perform read operations (fetching of data) on a GraphQL server. Mutations on the other hand are used to perform write operations (inserting, updating, or deleting data) on a GraphQL server. Subscriptions are completely different from these two, as they are used to add real-time functionality to a GraphQL server.


We’ll be focusing only on queries and mutations in this tutorial.


Now that we understand what a GraphQL schema is, let’s create the schema for our app. Within the src directory, create a schema.js file and add the following code into it:


src/schema.js
```
const { gql } = require('apollo-server');

const typeDefs = gql`
  type User {
    id: Int!
    name: String!
    email: String!
    recipes: [Recipe!]!
  }

  type Recipe {
    id: Int!
    title: String!
    ingredients: String!
    direction: String!
    user: User!
  }

  type Query {
    user(id: Int!): User
    allRecipes: [Recipe!]!
    recipe(id: Int!): Recipe
  }

  type Mutation {
    createUser(name: String!, email: String!, password: String!): User!
    createRecipe(
      userId: Int!
      title: String!
      ingredients: String!
      direction: String!
    ): Recipe!
  }
`;

module.exports = typeDefs;

```


First, we require the gql package from apollo-server. Then we use it to define our schema. Ideally, we’d want our GraphQL schema to mirror our database schema as much as possible. So we define two types, User and Recipe, which corresponds to our models. On the User type, in addition to defining the fields we have on the User model, we also define a recipes fields, which will be used to retrieve the user’s recipes. Same with the Recipe type; we define a user field, which will be used to get the user of a recipe.


Next, we define three queries: for fetching a single user, for fetching all recipes that have been created, and for fetching a single recipe respectively. Both the user and recipe queries can either return a user or recipe respectively or return null if no corresponding match was found for the ID. The allRecipes query will always return an array of recipes, which might be empty if no recipe as been created yet.



Note: The ! denotes a field is required, while [] denotes the field will return an array of items.

Lastly, we define mutations for creating a new user as well as creating a new recipe. Both mutations return back the created user and recipe respectively.


# Step 5 — Creating the Resolvers


Resolvers define how the fields in a schema are executed. In other words, our schema is useless without resolvers. Create a resolvers.js file inside the src directory and add the following code in it:


src/resolvers.js
```
const resolvers = {
  Query: {
    async user(root, { id }, { models }) {
      return models.User.findById(id);
    },
    async allRecipes(root, args, { models }) {
      return models.Recipe.findAll();
    },
    async recipe(root, { id }, { models }) {
      return models.Recipe.findById(id);
    },
  },
};

module.exports = resolvers;

```



Note: Modern versions of sequelize have deprecated findById and replaced it with findByPk. If you encounter errors like models.Recipe.findById is not a function or models.User.findById is not a function, you may need to update this snippet.

We start by creating the resolvers for our queries. Here, we are making use of the models to perform the necessary queries on the database and return the results.


Still inside src/resolvers.js, let’s import bcryptjs at the top of the file:


src/resolvers.js
```
const bcrypt = require('bcryptjs');

```


Then add the following code immediately after the Query object:


src/resolvers.js
```
Mutation: {
  async createUser(root, { name, email, password }, { models }) {
    return models.User.create({
      name,
      email,
      password: await bcrypt.hash(password, 10),
    });
  },
  async createRecipe(
    root,
    { userId, title, ingredients, direction },
    { models }
  ) {
    return models.Recipe.create({ userId, title, ingredients, direction });
  },
},

```


The createUser mutation accepts the name, email, and password of a user, and creates a new record in the database with the supplied details. We make sure to hash the password using the bcrypt package before persisting it to the database. It returns the newly created user. The createRecipe mutation accepts the ID of the user that’s creating the recipe as well as the details for the recipe itself, persists them to the database, and returns the newly created recipe.


To wrap up with the resolvers, let’s define how we want our custom fields (recipes on the User and user on Recipe) to be resolved. Add the following code inside src/resolvers.js just immediately after the Mutation object:


src/resolvers.js
```
User: {
  async recipes(user) {
    return user.getRecipes();
  },
},
Recipe: {
  async user(recipe) {
    return recipe.getUser();
  },
},

```


These use the methods, getRecipes() and getUser(), which are made available on our models by Sequelize due to the relationships we defined.


# Step 6 — Testing our GraphQL Server


It’s time to test our GraphQL server out. First, we need to start the server with:


```
node src/index.js


```


This will be running on localhost:4000, and we will see GraphQL Playground running if we access it.


Let’s try creating a new user:


```
# create a new user

mutation{
  createUser(
    name: "John Doe",
    email: "johndoe@example.com",
    password: "password"
  )
  {
    id,
    name,
    email
  }
}

```


This will produce the following result:


```
Output{
  "data": {
    "createUser": {
      "id": 1,
      "name": "John Doe",
      "email": "johndoe@example.com"
    }
  }
}

```


Let’s try creating a new recipe and associate it with the user that was created:


```
# create a new recipe

mutation {
  createRecipe(
    userId: 1
    title: "Salty and Peppery"
    ingredients: "Salt, Pepper"
    direction: "Add salt, Add pepper"
  ) {
    id
    title
    ingredients
    direction
    user {
      id
      name
      email
    }
  }
}

```


This will produce the following result:


```
Output{
  "data": {
    "createRecipe": {
      "id": 1,
      "title": "Salty and Peppery",
      "ingredients": "Salt, Pepper",
      "direction": "Add salt, Add pepper",
      "user": {
        "id": 1,
        "name": "John Doe",
        "email": "johndoe@example.com"
      }
    }
  }
}

```


Other queries you can perform here include: user(id: 1), recipe(id: 1), and allRecipes.


# Conclusion


In this tutorial, we looked at how to create a GraphQL server in Node.js with Apollo Server. We also saw how to integrate a database with a GraphQL server using Sequelize.


The code for this tutorial is available on GitHub.


