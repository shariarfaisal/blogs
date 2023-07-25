# How To Build a GraphQL API with Prisma and Deploy to DigitalOcean s App Platform

```DigitalOcean App Platform``` ```API``` ```GraphQL``` ```Development``` ```Node.js``` ```Git```

The authors selected the COVID-19 Relief Fund and the Tech Education Fund to receive a donation as part of the Write for Donations program.


## Introduction


GraphQL is a query language for APIs that consists of a schema definition language and a query language, which allows API consumers to fetch only the data they need to support flexible querying. GraphQL enables developers to evolve the API while meeting the different needs of multiple clients, for example iOS, Android, and web variants of an app. Moreover, the GraphQL schema adds a degree of type safety to the API while also serving as a form of documentation for your API.


Prisma is an open-source database toolkit with three main tools:


- Prisma Client: Auto-generated and type-safe query builder for Node.js & TypeScript.
- Prisma Migrate: Declarative data modeling & migration system.
- Prisma Studio: GUI to view and edit data in your database.

Prisma facilitates working with databases for application developers who want to focus on implementing value-adding features instead of spending time on complex database workflows (such as schema migrations or writing complicated SQL queries).


In this tutorial, you will use GraphQL and Prisma in combination as their responsibilities complement each other. GraphQL provides a flexible interface to your data for use in clients, such as frontends and mobile apps—GraphQL isn’t tied to any specific database. This is where Prisma comes in to handle the interaction with the database where your data will be stored.


DigitalOcean’s App Platform provides a seamless way to deploy applications and provision databases in the cloud without worrying about infrastructure. This reduces the operational overhead of running an application in the cloud; especially with the ability to create a managed PostgreSQL database with daily backups and automated failover. App Platform has native Node.js support streamlining deployment.


You’ll build a GraphQL API for a blogging application in JavaScript using Node.js. You will first use Apollo Server to build the GraphQL API backed by in-memory data structures. You will then deploy the API to the DigitalOcean App Platform. Finally you will use Prisma to replace the in-memory storage and persist the data in a PostgreSQL database and deploy the application again.


At the end of the tutorial, you will have a Node.js GraphQL API deployed to DigitalOcean, which handles GraphQL requests sent over HTTP and performs CRUD operations against the PostgreSQL database.


You can find the code for this project in the DigitalOcean Community respository.


# Prerequisites


Before you begin this guide you’ll need the following:


- A GitHub account.
- A DigitalOcean account.
- Git installed on your computer. You can follow the tutorial Contributing to Open Source: Getting Started with Git to install and set up Git on your computer.
- Node.js version 14 or higher installed on your computer. You can follow the tutorial How to Install Node.js and Create a Local Development Environment to install and set up Node.js on your computer.
- Docker installed on your computer (to run the PostgreSQL database locally).

Basic familiarity with JavaScript, Node.js, GraphQL, and PostgreSQL is helpful, but not strictly required for this tutorial.


# Step 1 — Creating the Node.js Project


In this step, you will set up a Node.js project with npm and install the dependencies apollo-server and graphql. This project will be the foundation for the GraphQL API that you’ll build and deploy throughout this tutorial.


First, create a new directory for your project:


```
mkdir prisma-graphql


```


Next, navigate into the directory and initialize an empty npm project:


```
cd prisma-graphql
npm init --yes


```


This command creates a minimal package.json file that is used as the configuration file for your npm project.


You will receive the following output:


```
OutputWrote to /Users/your_username/workspace/prisma-graphql/package.json:
{
  "name": "prisma-graphql",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```


You’re now ready to configure TypeScript in your project.


Install the necessary dependencies:


```
npm install apollo-server graphql --save


```


This command installs two packages as dependencies in your project:


- apollo-server is the HTTP library that you use to define how GraphQL requests are resolved and how to fetch data.
- graphql is the library you’ll use to build the GraphQL schema.

You’ve created your project and installed the dependencies. In the next step, you will define the GraphQL schema.


# Step 2 — Defining the GraphQL Schema and Resolvers


In this step, you will define the GraphQL schema and corresponding resolvers. The schema will define the operations that the API can handle. The resolvers will define the logic for handling those requests using in-memory data structures, which you will replace with database queries in the next step.


First, create a new directory called src that will contain your source files:


```
mkdir src


```


Then run the following command to create the file for the schema:


```
nano src/schema.js


```


Add the following code to the file:


prisma-graphql/src/schema.js
```
const { gql } = require('apollo-server')

const typeDefs = gql`
  type Post {
    content: String
    id: ID!
    published: Boolean!
    title: String!
  }

  type Query {
    feed: [Post!]!
    post(id: ID!): Post
  }

  type Mutation {
    createDraft(content: String, title: String!): Post!
    publish(id: ID!): Post
  }
`

```


You define the GraphQL schema using the gql tagged template. A schema is a collection of type definitions (hence typeDefs) that together define the shape of queries that can be executed against your API. This will convert the GraphQL schema string into the format that Apollo expects.


The schema introduces three types:


- Post defines the type for a post in your blogging app and contains four fields where each field is followed by its type: for example, String.
- Query defines the feed query which returns multiple posts as denoted by the square brackets and the post query which accepts a single argument and returns a single Post.
- Mutation defines the createDraft mutation for creating a draft Post and the publish mutation which accepts an id and returns a Post.

Every GraphQL API has a query type and may or may not have a mutation type. These types are the same as a regular object type, but they are special because they define the entry point of every GraphQL query.


Next, add the posts array to the src/schema.js file, below the typeDefs variable:


prisma-graphql/src/schema.js
```
...
const posts = [
  {
    id: 1,
    title: 'Subscribe to GraphQL Weekly for community news ',
    content: 'https://graphqlweekly.com/',
    published: true,
  },
  {
    id: 2,
    title: 'Follow DigitalOcean on Twitter',
    content: 'https://twitter.com/digitalocean',
    published: true,
  },
  {
    id: 3,
    title: 'What is GraphQL?',
    content: 'GraphQL is a query language for APIs',
    published: false,
  },
]

```


You define the posts array with three pre-defined posts. The structure of each post object matches the Post type you defined in the schema. This array holds the posts that will be served by the API. In a subsequent step, you will replace the array once the database and Prisma Client are introduced.


Next, define the resolvers object by adding the following code below the posts array you just defined:


prisma-graphql/src/schema.js
```
...
const resolvers = {
  Query: {
    feed: (parent, args) => {
      return posts.filter((post) => post.published)
    },
    post: (parent, args) => {
      return posts.find((post) => post.id === Number(args.id))
    },
  },
  Mutation: {
    createDraft: (parent, args) => {
      posts.push({
        id: posts.length + 1,
        title: args.title,
        content: args.content,
        published: false,
      })
      return posts[posts.length - 1]
    },
    publish: (parent, args) => {
      const postToPublish = posts.find((post) => post.id === Number(args.id))
      postToPublish.published = true
      return postToPublish
    },
  },
  Post: {
    content: (parent) => parent.content,
    id: (parent) => parent.id,
    published: (parent) => parent.published,
    title: (parent) => parent.title,
  },
}

module.exports = {
  resolvers,
  typeDefs,
}

```


You define the resolvers following the same structure as the GraphQL schema. Every field in the schema’s types has a corresponding resolver function whose responsibility is to return the data for that field in your schema. For example, the Query.feed() resolver will return the published posts by filtering the posts array.


Resolver functions receive four arguments:


- parent is the return value of the previous resolver in the resolver chain. For top-level resolvers, the parent is undefined, because no previous resolver is called. For example, when making a feed query, the query.feed() resolver will be called with parent’s value undefined and then the resolvers of Post will be called where parent is the object returned from the feed resolver.
- args carries the parameters for the query. For example, the post query, will receive the id of the post to be fetched.
- context is an object that gets passed through the resolver chain that each resolver can write to and read from, which allows the resolvers to share information.
- info is an AST representation of the query or mutation. You can read more about the details in this Prisma series on GraphQL Basics.

Since context and info are not necessary in these resolvers, only parent and args are defined.


Save and exit the file once you’re done.



Note: When a resolver returns the same field as the resolver’s name, like the four resolvers for Post, Apollo Server will automatically resolve those. This means you don’t have to explicitly define those resolvers.
-  Post: {
-    content: (parent) => parent.content,
-    id: (parent) => parent.id,
-    published: (parent) => parent.published,
-    title: (parent) => parent.title,
-  },


You export the schema and resolvers so that you can use them in the next step to instantiate the server with Apollo Server.


# Step 3 — Creating the GraphQL Server


In this step, you will create the GraphQL server with Apollo Server and bind it to a port so that the server can accept connections.


First, run the following command to create the file for the server:


```
nano src/server.js


```


Add the following code to the file:


prisma-graphql/src/server.js
```
const { ApolloServer } = require('apollo-server')
const { resolvers, typeDefs } = require('./schema')

const port = process.env.PORT || 8080

new ApolloServer({ resolvers, typeDefs }).listen({ port }, () =>
  console.log(`Server ready at: http://localhost:${port}`),
)

```


Here, you instantiate the server and pass the schema and resolvers from the previous step.


The port the server will bind to is set from the PORT environment variable. If not set, it will default to 8080. The PORT environment variable will be automatically set by App Platform and will ensure your server can accept connections once deployed.


Save and exit the file.


Your GraphQL API is ready to run. Start the server with the following command:


```
node src/server.js


```


You will receive the following output:


```
OutputServer ready at: http://localhost:8080

```


It’s considered good practice to add a start script to your package.json file so that the entry point to your server is clear. Doing so will allow App Platform to start the server once deployed.


First, stop the server by pressing CTRL+C. Then, to add a start script, open the package.json file:


```
nano package.json


```


Add the highlighted text to the "scripts" object in package.json:


package.json
```
{
  "name": "prisma-graphql",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node ./src/server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "apollo-server": "^3.11.1",
    "graphql": "^16.6.0"
  }
}

```


Save and exit the file.


Now you can start the server with the following command:


```
npm start


```


You will receive the following output:


```
Output> prisma-graphql@1.0.0 start
> node ./src/server.js

Server ready at: http://localhost:8080

```


To test the GraphQL API, open the URL from the output, which will lead you to the Apollo GraphQL Studio. Click the Query Your Server button on the home page to interact with the IDE.


Apollo GraphQL Studio
The Apollo GraphQL Studio is an IDE where you can test the API by sending queries and mutations.


For example, to test the feed query, which only returns published posts, enter the following query to the left side of the IDE and send the query by pressing the Run or play button:


```
query {
  feed {
    id
    title
    content
    published
  }
}

```


The response will display a title of Subscribe to GraphQL Weekly with its URL and Follow DigitalOcean on Twitter with its URL.


GraphQL Feed Query
Click on the + button on the bar above your previous query to create a new tab. Then, to test the createDraft mutation, enter the following mutation:


```
mutation {
  createDraft(title: "Deploying a GraphQL API to DigitalOcean") {
    id
    title
    content
    published
  }
}

```


After you submit the mutation using the play button, you will receive a response with Deploying a GraphQL API to DigitalOcean within the title field as part of the response.


GraphQL Create Draft Mutation

Note: You can choose which fields to return from the mutation by adding or removing fields within the curly braces ({}) following createDraft. For example, if you wanted to only return the id and title you could send the following mutation:
mutation {
  createDraft(title: "Deploying a GraphQL API to DigitalOcean") {
    id
    title
  }
}


You have successfully created and tested the GraphQL server. In the next step, you will create a GitHub repository for the project.


# Step 4 — Creating the GitHub Repository


In this step, you will create a GitHub repository for your project and push your changes so that the GraphQL API can be automatically deployed from GitHub to App Platform.


First, stop the development server by pressing CTRL+C. Then initialize a repository from the prisma-graphql folder using the following command:


```
git init


```


Next, use the following two commands to commit the code to the repository:


```
git add src package-lock.json package.json
git commit -m 'Initial commit'


```


Now that the changes have been committed to your local repository, you will create a repository in GitHub and push your changes.


Go to GitHub to create a new repository. For consistency, name the repository prisma-graphql and then click Create repository.


After the repository is created, push the changes with the following commands, which includes renaming the default local branch to main:


```
git remote add origin git@github.com:your_github_username/prisma-graphql.git
git branch -M main
git push --set-upstream origin main


```


You have successfully committed and pushed the changes to GitHub. Next, you will connect the repository to App Platform and deploy the GraphQL API.


# Step 5 — Deploying to App Platform


In this step, you will connect the GitHub repository you just created to DigitalOcean and then configure App Platform so that the GraphQL API can be automatically deployed when you push changes to GitHub.


First, visit the App Platform page in the DigitalOcean Cloud Console and click on the Create App button.


You will see service provider options with GitHub as the default.


If you have not configured DigitalOcean to your GitHub account, click on the Manage Access button to be redirected to GitHub.


Manage GitHub Access
You can select all repositories or specific repositories. Click Install & Authorize, then you will be redirected back to the DigitalOcean App Platform creation.


Choose the repository your_github_username/prisma-graphql and click Next. Autodeploy is selected by default, and you can leave it selected for consistency in redeploys.


Choose Repository
On the Resources page, click the Edit Plan button to choose a suitable plan. Select the Basic plan with the plan size you need (this tutorial will use the $5.00/mo - Basic plan).


Edit Payment Plan
Then press the Back to return to the creation page.


If you press the pen icon next to your project name, you can customize the configuration for the app. The Application Settings page will open:


Application Settings
Ensure that the Run Command is set as npm start. By default, App Platform will set the HTTP port to 8080, which is the same port that you’ve configured your GraphQL server to bind to.


When you have finished customizing the configuration, press the Back button to return to the setup page. Then, press the Next button to move to the Environment Variables page.


Your environment variables will not need further configuration at the moment. Click the Next button.


Environment Variables
On the Info page, you can adjust App Details and Location. Edit your app information to choose the region you want to deploy your app to. Confirm your app details by pressing the Save button. Then, click the Next button.


Location of Application
You will be able to review all of your selected options on the Review page. Then click Create Resources. You will be redirected to the app page, where you will see the progress of the initial deployment.


Once the build finishes, you will get a notification indicating that your app is deployed.


Deployment Progress Bar
You can now visit your deployed GraphQL API at the URL below the app’s name in your DigitalOcean Console. It will be linked via the ondigitalocean.app subdomain. When you open the URL, the GraphQL Playground will open the same way as it did in Step 3 of this tutorial.


You have successfully connected your repository to App Platform and deployed your GraphQL API. Next you will evolve your app and replace the in-memory data of the GraphQL API with a database.


# Step 6 — Setting Up Prisma with PostgreSQL


So far, you have built a GraphQL API using the in-memory posts array to store data. If your server restarts, all changes to the data will be lost. To ensure that your data is safely persisted, you will replace the posts array with a PostgreSQL database and use Prisma to access the data.


In this step, you will install the Prisma CLI, create your initial Prisma schema (the main configuration file for your Prisma setup, containing your database schema), set up PostgreSQL locally with Docker, and connect Prisma to it.


Begin by installing the Prisma CLI with the following command:


```
npm install --save-dev prisma


```


The Prisma CLI will help with database workflows such as running database migrations and generating Prisma Client.


Next, you’ll set up your PostgreSQL database using Docker. Create a new Docker Compose file with the following command:


```
nano docker-compose.yml


```


Add the following code to the newly created file:


prisma-graphql/docker-compose.yml
```
version: '3.8'
services:
  postgres:
    image: postgres:14
    restart: always
    environment:
      - POSTGRES_USER=test-user
      - POSTGRES_PASSWORD=test-password
    volumes:
      - postgres:/var/lib/postgresql/data
    ports:
      - '5432:5432'
volumes:
  postgres:

```


This Docker Compose configuration file is responsible for starting the official PostgreSQL Docker image on your machine. The POSTGRES_USER and POSTGRES_PASSWORD environment variables set the credentials for the superuser (a user with admin privileges). You will also use these credentials to connect Prisma to the database. Replace the test-user and test-password with your user credentials.


Finally, you define a volume where PostgreSQL will store its data and bind the 5432 port on your machine to the same port in the Docker container.


Save and exit the file.


With this setup in place, you can launch the PostgreSQL database server with the following command:


```
docker-compose up -d


```


It may take a few minutes to load.


You can verify that the database server is running with the following command:


```
docker ps


```


This command will output something similar to:


```
OutputCONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
198f9431bf73        postgres:10.3       "docker-entrypoint.s…"   45 seconds ago      Up 11 seconds       0.0.0.0:5432->5432/tcp   prisma-graphql_postgres_1

```


With the PostgreSQL container running, you can now create your Prisma setup. Run the following command from the Prisma CLI:


```
npx prisma init


```


As a best practice, all invocations of the Prisma CLI should be prefixed with npx to ensure it uses your local installation.


An output like this will print:


```
Output✔ Your Prisma schema was created at prisma/schema.prisma
  You can now open it in your favorite editor.

Next steps:
1. Set the DATABASE_URL in the .env file to point to your existing database. If your database has no tables yet, read https://pris.ly/d/getting-started
2. Set the provider of the datasource block in schema.prisma to match your database: postgresql, mysql, sqlite, sqlserver, mongodb or cockroachdb.
3. Run prisma db pull to turn your database schema into a Prisma schema.
4. Run prisma generate to generate the Prisma Client. You can then start querying your database.

More information in our documentation:
https://pris.ly/d/getting-started

```


After running the command, the Prisma CLI generates a dotenv file named .env in the project folder to define your database connection URL, as well as a new nested folder called prisma that contains the schema.prisma file. This is the main configuration file for your Prisma project (in which you will include your data model).


To make sure Prisma knows about the location of your database, open the .env file:


```
nano .env


```


Adjust the DATABASE_URL environment variable with your user credentials:


prisma-graphql/.env
```
DATABASE_URL="postgresql://test-user:test-password@localhost:5432/my-blog?schema=public"

```


You use the database credentials test-user and test-password, which are specified in the Docker Compose file. If you modified the credentials in your Docker Compose file, be sure to update this line to match the credentials in that file. To learn more about the format of the connection URL, visit the Prisma docs.


You have successfully started PostgreSQL and configured Prisma using the Prisma schema. In the next step, you will define your data model for the blog and use Prisma Migrate to create the database schema.


# Step 7 — Defining the Data Model with Prisma Migrate


Now you will define your data model in the Prisma schema file you’ve just created. This data model will then be mapped to the database with Prisma Migrate, which will generate and send the SQL statements for creating the tables that correspond to your data model.


Since you’re building a blog, the main entities of the application will be users and posts. In this step, you will define a Post model with a similar structure to the Post type in the GraphQL schema. In a later step, you will evolve the app and add a User model.



Note: The GraphQL API can be seen as an abstraction layer for your database. When building a GraphQL API, it’s common for the GraphQL schema to closely resemble your database schema. However, as an abstraction, the two schemas won’t necessarily have the same structure, thereby allowing you to control which data you want to expose over the API as some data might be considered sensitive or irrelevant for the API layer.

Prisma uses its own data modeling language to define the shape of your application data.


Open your schema.prisma file from the project’s folder where package.json is located:


```
nano prisma/schema.prisma


```



Note: You can verify from the terminal in which folder you are with the pwd command, which will output the current working directory. Additionally, listing the files with the ls command will help you navigate your file system.

Add the following model definitions to it:


prisma-graphql/prisma/schema.prisma
```
...
model Post {
  id        Int     @default(autoincrement()) @id
  title     String
  content   String?
  published Boolean @default(false)
}

```


You define a model called Post with a number of fields. The model will be mapped to a database table; the fields represent the individual columns.


The id fields have the following field attributes:


- @default(autoincrement()) sets an auto-incrementing default value for the column.
- @id sets the column as the primary key for the table.

Save and exit the file.


With the model in place, you can now create the corresponding table in the database using Prisma Migrate with the migrate dev command to create and run the migration files.


In your terminal, run the following command:


```
npx prisma migrate dev --name init --skip-generate


```


This command creates a new migration on your file system and runs it against the database to create the database schema. The --name init flag specifies the name of the migration (will be used to name the migration folder that’s created on your file system). The --skip-generate flag skips generating Prisma Client (this will be done in the next step).


This command will output something similar to:


```
OutputEnvironment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db": PostgreSQL database "my-blog", schema "public" at "localhost:5432"

PostgreSQL database my-blog created at localhost:5432

Applying migration `20201201110111_init`

The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20201201110111_init/
    └─ migration.sql

Your database is now in sync with your schema.

```


Your prisma/migrations directory is now populated with the SQL migration file. This approach allows you to track changes to the database schema and create the same database schema in production.



Note: If you’ve already used Prisma Migrate with the my-blog database and there is an inconsistency between the migrations in the prisma/migration folder and the database schema, you will be prompted to reset the database with the following output:
Output? We need to reset the PostgreSQL database "my-blog" at "localhost:5432". All data will be lost.
Do you want to continue? › (y/N)

You can resolve this by entering y which will reset the database. Beware that this will cause all data in the database to be lost.

You’ve now created your database schema. In the next step, you will install Prisma Client and use it in your GraphQL resolvers.


# Step 8 — Using Prisma Client in the GraphQL Resolvers


Prisma Client is an auto-generated and type-safe Object Relational Mapper (ORM) that you can use to programmatically read and write data in a database from a Node.js application. In this step, you’ll install Prisma Client in your project.


In your terminal, install the Prisma Client npm package:


```
npm install @prisma/client


```



Note: Prisma Client provides rich auto-completion by generating code based on your Prisma schema to the node_modules folder. To generate the code, you use the npx prisma generate command. This is typically done after you create and run a new migration. On the first install, however, this is not necessary as it will automatically be generated for you in a postinstall hook.

After creating the database and GraphQL schema and installing Prisma Client, you will now use Prisma Client in the GraphQL resolvers to read and write data in the database. You’ll do this by replacing the posts array, which you’ve used so far to hold your data.


Create and open the following file:


```
nano src/db.js


```


Add the following lines to the new file:


prisma-graphql/src/db.js
```
const { PrismaClient } = require('@prisma/client')

module.exports = {
  prisma: new PrismaClient(),
}

```


This code imports Prisma Client, creates an instance of it, and exports the instance that you’ll use in your resolvers.


Now save and close the src/db.js file.


Next, you will import the prisma instance into src/schema.js. To do so, open src/schema.js:


```
nano src/schema.js


```


Add this line to import prisma from ./db at the top of the file:


prisma-graphql/src/schema.js
```
const { prisma } = require('./db')
...

```


Then remove the posts array by deleting the lines that are marked with the hyphen symbol (-):


prisma-graphql/src/schema.js
```
...
-const posts = [
-  {
-    id: 1,
-    title: 'Subscribe to GraphQL Weekly for community news ',
-    content: 'https://graphqlweekly.com/',
-    published: true,
-  },
-  {
-    id: 2,
-    title: 'Follow DigitalOcean on Twitter',
-    content: 'https://twitter.com/digitalocean',
-    published: true,
-  },
-  {
-    id: 3,
-    title: 'What is GraphQL?',
-    content: 'GraphQL is a query language for APIs',
-    published: false,
-  },
-]
...

```


You will next update the Query resolvers to fetch published posts from the database. First, delete the existing lines in the resolvers.Query, then update the object by adding the highlighted lines:


prisma-graphql/src/schema.js
```
...
const resolvers = {
  Query: {
    feed: (parent, args) => {
      return prisma.post.findMany({
        where: { published: true },
      })
    },
    post: (parent, args) => {
      return prisma.post.findUnique({
        where: { id: Number(args.id) },
      })
    },
  },
...

```


Here, you use two Prisma Client queries:


- findMany fetches posts whose publish field is false.
- findUnique fetches a single post whose id field equals the id GraphQL argument.

Per the GraphQL specification, the ID type is serialized the same way as a String. Therefore you convert to a Number because the id in the Prisma schema is an int.


Next, you will update the Mutation resolver to save and update posts in the database. First, delete the code in the resolvers.Mutation object and the Number(args.id) lines, then add the highlighted lines:


prisma-graphql/src/schema.js
```
const resolvers = {
  ...
  Mutation: {
    createDraft: (parent, args) => {
      return prisma.post.create({
        data: {
          title: args.title,
          content: args.content,
        },
      })
    },
    publish: (parent, args) => {
      return prisma.post.update({
        where: {
          id: Number(args.id),
        },
        data: {
          published: true,
        },
      })
    },
  },
}

```


You’re using two Prisma Client queries:


- create to create a Post record.
- update to update the published field of the Post record whose id matches the one in the query argument.

Finally, remove the resolvers.Post object:


prisma-graphql/src/schema.js
```
...
-Post: {
-  content: (parent) => parent.content,
-  id: (parent) => parent.id,
-  published: (parent) => parent.published,
-  title: (parent) => parent.title,
-},
...

```


Your schema.js should now read as follows:


prisma-graphql/src/schema.js
```
const { gql } = require('apollo-server')
const { prisma } = require('./db')

const typeDefs = gql`
  type Post {
    content: String
    id: ID!
    published: Boolean!
    title: String!
  }

  type Query {
    feed: [Post!]!
    post(id: ID!): Post
  }

  type Mutation {
    createDraft(content: String, title: String!): Post!
    publish(id: ID!): Post
  }
`

const resolvers = {
  Query: {
    feed: (parent, args) => {
      return prisma.post.findMany({
        where: { published: true },
      })
    },
    post: (parent, args) => {
      return prisma.post.findUnique({
        where: { id: Number(args.id) },
      })
    },
  },
  Mutation: {
    createDraft: (parent, args) => {
      return prisma.post.create({
        data: {
          title: args.title,
          content: args.content,
        },
      })
    },
    publish: (parent, args) => {
      return prisma.post.update({
        where: {
          id: Number(args.id),
        },
        data: {
          published: true,
        },
      })
    },
  },
}

module.exports = {
  resolvers,
  typeDefs,
}

```


Save and close the file.


Now that you’ve updated the resolvers to use Prisma Client, start the server to test the flow of data between the GraphQL API and the database with the following command:


```
npm start


```


Once again, you will receive the following output:


```
OutputServer ready at: http://localhost:8080

```


Open the Apollo GraphQL Studio at the address from the output and test the GraphQL API using the same queries from Step 3.


Now you will commit your changes so that the changes can be deployed to App Platform. Stop the Apollo server with CTRL+C.


To avoid committing the node_modules folder and the .env file, check the .gitignore file in your project folder:


```
cat .gitignore


```


Confirm that your .gitignore file contains these lines:


prisma-graphql/.gitignore
```
node_modules
.env

```


If it doesn’t, update the file to match.


Save and exit the file.


Then run the following two commands to commit the changes:


```
git add .
git commit -m 'Add Prisma'


```


You will receive an output response like this:


```
Outputgit commit -m 'Add Prisma'
[main 1646d07] Add Prisma
 9 files changed, 157 insertions(+), 39 deletions(-)
 create mode 100644 .gitignore
 create mode 100644 docker-compose.yml
 create mode 100644 prisma/migrations/20201201110111_init/migration.sql
 create mode 100644 prisma/migrations/migration_lock.toml
 create mode 100644 prisma/schema.prisma
 create mode 100644 src/db.js

```


You have updated your GraphQL resolvers to use the Prisma Client to make queries and mutations to your database, then committed all the changes to your remote repository. Next you’ll add a PostgreSQL database to your app in App Platform.


# Step 9 — Creating and Migrating the PostgreSQL Database in App Platform


In this step, you will add a PostgreSQL database to your app in App Platform. Then you will use Prisma Migrate to run the migration against it so that the deployed database schema matches your local database.


First, visit the App Platform console and select the prisma-graphql project you created in Step 5.


Next, click the Create button and select Create/Attach Database from the dropdown menu, which will lead you to a page to configure your database.


Create/Attach Database
Choose Dev Database, select a name, and click Create and Attach.


Database Config
You will be redirected back to the Project view, where there will be a progress bar for creating the database.


Progress Bar
After the database has been created, you will run the database migration against the production database on DigitalOcean from your local machine. To run the migration, you will need the connection string of the hosted database.


To get it, click on the db icon in the Components section of the Settings tab.


Database Component Settings
Under Connection Details, press View and then select Connection String in the dropdown menu. Copy the database URL, which will have the following structure:


```
postgresql://db:some_password@unique_identifier.db.ondigitalocean.com:25060/db?sslmode=require

```


Then, run the following command in your terminal, ensuring that you set your_db_connection_string to the URL you just copied:


```
DATABASE_URL="your_db_connection_string" npx prisma migrate deploy


```


This command will run the migrations against the live database with Prisma Migrate.


If the migration succeeds, you will receive the following output:


```
OutputPostgreSQL database db created at unique_identifier.db.ondigitalocean.com:25060

Prisma Migrate applied the following migration(s):

migrations/
  └─ 20201201110111_init/
    └─ migration.sql

```


You have successfully migrated the production database on DigitalOcean, which now matches the Prisma schema.



Note: If you receive the following error message:
OutputError: P1001: Can't reach database server at `unique_identifier.db.ondigitalocean.com`:`25060`

Navigate to the database dashboard to confirm that your database has been provisioned. You may need to update or disable the Trusted Sources for the database.

Now you can deploy your app by pushing your Git changes with the following command:


```
git push


```



Note: App Platform will make the DATABASE_URL environment variable available to your application at run-time. Prisma Client will use that environment variable with the env("DATABASE_URL") in the datasource block of your Prisma schema.

This will automatically trigger a build. If you open the App Platform console, you will have a deployment progress bar.


Deployment Progress Bar
Once the deployment succeeds, you will receive a Deployment went live message.


You’ve now backed up your deployed GraphQL API with a database. Open the Live App, which will lead you to the Apollo GraphQL Studio. Test the GraphQL API using the same queries from Step 3.


In the final step you will evolve the GraphQL API by adding the User model.


# Step 10 — Adding the User Model


Your GraphQL API for blogging has a single entity named Post. In this step, you’ll evolve the API by defining a new model in the Prisma schema and adapting the GraphQL schema to make use of the new model. You will introduce a User model with a one-to-many relation to the Post model, which will allow you to represent the author of posts and associate multiple posts to each user. Then you will evolve the GraphQL schema to allow the creation of users and association of posts with users through the API.


First, open the Prisma schema:


```
nano prisma/schema.prisma


```


Add the highlighted lines to add the authorId field to the Post model and to define the User model:


prisma-graphql/prisma/schema.prisma
```
...
model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User?   @relation(fields: [authorId], references: [id])
  authorId  Int?
}

model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
  posts Post[]
}

```


You’ve added the following items to the Prisma schema:


- Two relation fields: author and posts. Relation fields define connections between models at the Prisma level and do not exist in the database. These fields are used to generate the Prisma Client and to access relations with Prisma Client.
- The authorId field, which is referenced by the @relation attribute. Prisma will create a foreign key in the database to connect Post and User.
- The User model to represent users.

The author field in the Post model is optional but allows you to create posts that are not associated with a user.


Save and exit the file once you’re done.


Next, create and apply the migration locally with the following command:


```
npx prisma migrate dev --name "add-user"


```


When the migration succeeds, you will receive the following message:


```
OutputEnvironment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db": PostgreSQL database "my-blog", schema "public" at "localhost:5432"

Applying migration `20201201123056_add_user`

The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20201201123056_add_user/
    └─ migration.sql

Your database is now in sync with your schema.

✔ Generated Prisma Client (4.6.1 | library) to ./node_modules/@prisma/client in 53ms

```


The command also generates Prisma Client so that you can make use of the new table and fields.


You will now run the migration against the production database on App Platform so that the database schema is the same as your local database. Run the following command in your terminal and set DATABASE_URL to the connection URL from App Platform:


```
DATABASE_URL="your_db_connection_string" npx prisma migrate deploy


```


You will receive the following output:


```
OutputEnvironment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db": PostgreSQL database "db", schema "public" at "unique_identifier.db.ondigitalocean.com:25060"

2 migrations found in prisma/migrations

Applying migration `20201201123056_add_user`

The following migration have been applied:

migrations/
  └─ 20201201123056_add_user/
    └─ migration.sql
      
All migrations have been successfully applied.

```


You will now update the GraphQL schema and resolvers to make use of the updated database schema.


Open the src/schema.js file:


```
nano src/schema.js


```


Update typeDefs with the highlighted lines as follows:


prisma-graphql/src/schema.js
```
...
const typeDefs = gql`
  type User {
    email: String!
    id: ID!
    name: String
    posts: [Post!]!
  }

  type Post {
    content: String
    id: ID!
    published: Boolean!
    title: String!
    author: User
  }

  type Query {
    feed: [Post!]!
    post(id: ID!): Post
  }

  type Mutation {
    createUser(data: UserCreateInput!): User!
    createDraft(authorEmail: String, content: String, title: String!): Post!
    publish(id: ID!): Post
  }

  input UserCreateInput {
    email: String!
    name: String
    posts: [PostCreateWithoutAuthorInput!]
  }

  input PostCreateWithoutAuthorInput {
    content: String
    published: Boolean
    title: String!
  }
`
...

```


In this updated code, you add the following changes to the GraphQL schema:


- The User type, which returns an array of Post.
- The author field to the Post type.
- The createUser mutation, which expects the UserCreateInput as its input type.
- The PostCreateWithoutAuthorInput input type used in the UserCreateInput input for creating posts as part of the createUser mutation.
- The authorEmail optional argument to the createDraft mutation.

With the schema updated, you will now update the resolvers to match the schema.


Update the resolvers object with the highlighted lines as follows:


prisma-graphql/src/schema.js
```
...
const resolvers = {
  Query: {
    feed: (parent, args) => {
      return prisma.post.findMany({
        where: { published: true },
      })
    },
    post: (parent, args) => {
      return prisma.post.findUnique({
        where: { id: Number(args.id) },
      })
    },
  },
  Mutation: {
    createDraft: (parent, args) => {
      return prisma.post.create({
        data: {
          title: args.title,
          content: args.content,
          published: false,
          author: args.authorEmail && {
            connect: { email: args.authorEmail },
          },
        },
      })
    },
    publish: (parent, args) => {
      return prisma.post.update({
        where: { id: Number(args.id) },
        data: {
          published: true,
        },
      })
    },
    createUser: (parent, args) => {
      return prisma.user.create({
        data: {
          email: args.data.email,
          name: args.data.name,
          posts: {
            create: args.data.posts,
          },
        },
      })
    },
  },
  User: {
    posts: (parent, args) => {
      return prisma.user
        .findUnique({
          where: { id: parent.id },
        })
        .posts()
    },
  },
  Post: {
    author: (parent, args) => {
      return prisma.post
        .findUnique({
          where: { id: parent.id },
        })
        .author()
    },
  },
}
...

```


The createDraft mutation resolver now uses the authorEmail argument (if passed) to create a relation between the created draft and an existing user.


The new createUser mutation resolver creates a user and related posts using nested writes.


The User.posts and Post.author resolvers define how to resolve the posts and author fields when the User or Post are queried. These use Prisma’s Fluent API to fetch the relations.


Save and exit the file.


Start the server to test the GraphQL API:


```
npm start


```


Begin by testing the createUser resolver with the following GraphQL mutation:


```
mutation {
  createUser(data: { email: "natalia@prisma.io", name: "Natalia" }) {
    email
    id
  }
}

```


This mutation will create a user.


Next, test the createDraft resolver with the following mutation:


```
mutation {
  createDraft(
    authorEmail: "natalia@prisma.io"
    title: "Deploying a GraphQL API to App Platform"
  ) {
    id
    title
    content
    published
    author {
      id
      name
    }
  }
}

```


You can fetch the author whenever the return value of a query is Post. In this example, the Post.author resolver will be called.


Close the server when finished testing.


Then commit your changes and push to deploy the API:


```
git add .
git commit -m "add user model"
git push


```


It may take a few minutes for your updates to deploy.


You have successfully evolved your database schema with Prisma Migrate and exposed the new model in your GraphQL API.


# Conclusion


In this article, you built a GraphQL API with Prisma and deployed it to DigitalOcean’s App Platform. You defined a GraphQL schema and resolvers with Apollo Server. You then used Prisma Client in your GraphQL resolvers to persist and query data in the PostgreSQL database. As a next step, you can extend the GraphQL API with a query to fetch individual users and a mutation to connect an existing draft to a user.


If you’re interested in exploring the data in the database, check out Prisma Studio. You can also visit the Prisma documentation to learn about different aspects of Prisma and explore some ready-to-run example projects in the prisma-examples repository.


You can find the code for this project in the DigitalOcean Community repository.


