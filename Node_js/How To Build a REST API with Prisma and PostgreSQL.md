# How To Build a REST API with Prisma and PostgreSQL

```Databases``` ```Docker``` ```PostgreSQL``` ```API``` ```TypeScript``` ```Node.js```

The author selected the Diversity in Tech Fund and the Tech Education Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Prisma is an open-source ORM for Node.js and TypeScript. It consists of three main tools:


- Prisma Client: An auto-generated and type-safe query builder.
- Prisma Migrate: A powerful data modeling and migration system.
- Prisma Studio: A GUI to view and edit data in your database.

These tools aim to increase an application developer’s productivity in their database workflows. One of the top benefits of Prisma is the level of abstraction it provides: Instead of figuring out complex SQL queries or schema migrations, application developers can reason about their data in a more intuitive way when using Prisma.


In this tutorial, you will build a REST API for a small blogging application in TypeScript using Prisma and a PostgreSQL database. You will set up your PostgreSQL database locally with Docker and implement the REST API routes using Express. At the end of the tutorial, you will have a web server running locally on your machine that can respond to various HTTP requests and read and write data in the database.


# Prerequisites


This tutorial assumes the following:


- Node.js version 14 or higher installed on your machine. You can use one of the How To Install Node.js and Create a Local Development Environment guides for your OS to set this up.
- Docker installed on your machine (to run the PostgreSQL database). You can install on macOS and Windows via the Docker website, or follow How To Install and User Docker for Linux distributions.

Basic familiarity with TypeScript and REST APIs is helpful but not required for this tutorial.


# Step 1 — Creating Your TypeScript Project


In this step, you will set up a plain TypeScript project using npm. This project will be the foundation for the REST API you’re going to build in this tutorial.


First, create a new directory for your project:


```
mkdir my-blog


```


Next, navigate into the directory and initialize an empty npm project. Note that the -y option here means that you’re skipping the interactive prompts of the command. To run through the prompts, remove -y from the command:


```
cd my-blog
npm init -y


```


For more details on these prompts, you can follow Step 1 in How To Use Node.js Modules with npm and package.json.


You’ll receive output similar to the following with the default responses in place:


```
OutputWrote to /.../my-blog/package.json:

{
  "name": "my-blog",
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


This command creates a minimal package.json file that you use as the configuration file for your npm project. You’re now ready to configure TypeScript in your project.


Execute the following command for a plain TypeScript setup:


```
npm install typescript ts-node @types/node --save-dev


```


This installs three packages as development dependencies in your project:


- typescript: The TypeScript toolchain.
- ts-node: A package to run TypeScript applications without prior compilation to JavaScript.
- @types/node: The TypeScript type definitions for Node.js.

The last thing to do is to add a tsconfig.json file to ensure TypeScript is properly configured for the application you’re going to build.


First, run the following command to create the file:


```
nano tsconfig.json


```


Add the following JSON code into the file:


my-blog/tsconfig.json
```
{
  "compilerOptions": {
    "sourceMap": true,
    "outDir": "dist",
    "strict": true,
    "lib": ["esnext"],
    "esModuleInterop": true
  }
}

```


Save and exit the file.


This setup is a standard and minimal configuration for a TypeScript project. If you want to learn about the individual properties of the configuration file, you can review the TypeScript documentation.


You’ve set up your plain TypeScript project using npm. Next you’ll set up your PostgreSQL database with Docker and connect Prisma to it.


# Step 2 — Setting Up Prisma with PostgreSQL


In this step, you will install the Prisma CLI, create your initial Prisma schema file, and set up PostgreSQL with Docker and connect Prisma to it. The Prisma schema is the main configuration file for your Prisma setup and contains your database schema.


Start by installing the Prisma CLI with the following command:


```
npm install prisma --save-dev


```


As a best practice, it is recommended to install the Prisma CLI locally in your project (rather than as a global installation). This practice helps avoid version conflicts in case you have more than one Prisma project on your machine.


Next, you’ll set up your PostgreSQL database using Docker. Create a new Docker Compose file with the following command:


```
nano docker-compose.yml


```


Now add the following code to the newly created file:


my-blog/docker-compose.yml
```
version: '3.8'
services:
  postgres:
    image: postgres:10.3
    restart: always
    environment:
      - POSTGRES_USER=sammy
      - POSTGRES_PASSWORD=your_password
    volumes:
      - postgres:/var/lib/postgresql/data
    ports:
      - '5432:5432'
volumes:
  postgres:

```


This Docker Compose file configures a PostgreSQL database that can be accessed via port 5432 of the Docker container. The database credentials are currently set as sammy (user) and your_password (password). Feel free to adjust these credentials to your preferred user and password. Save and exit the file.


With this setup in place, launch the PostgreSQL database server with the following command:


```
docker-compose up -d


```


The output of this command will be similar to this:


```
OutputPulling postgres (postgres:10.3)...
10.3: Pulling from library/postgres
f2aa67a397c4: Pull complete
6de83ca23e55: Pull complete
. . .
Status: Downloaded newer image for postgres:10.3
Creating my-blog_postgres_1 ... done

```


You can verify that the database server is running with the following command:


```
docker ps


```


This command will output something similar to this:


```
OutputCONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
8547f8e007ba        postgres:10.3       "docker-entrypoint.s…"   3 seconds ago       Up 2 seconds        0.0.0.0:5432->5432/tcp   my-blog_postgres_1

```


With the database server running, you can now create your Prisma setup. Run the following command from the Prisma CLI:


```
npx prisma init


```


This command will print the following output:


```
Output✔ Your Prisma schema was created at prisma/schema.prisma.
  You can now open it in your favorite editor.

```


As a best practice, you should prefix all invocations of the Prisma CLI with npx to ensure your local installation is being used.


After you run the command, the Prisma CLI creates a new folder called prisma in your project. Inside it, you will find a schema.prisma file, which is the main configuration file for your Prisma project (including your data model). This command also adds a .env dotenv file to your root folder, which is where you will define your database connection URL.


To ensure Prisma knows about the location of your database, open the .env file and adjust the DATABASE_URL environment variable.


First open the .env file:


```
nano .env


```


Now you can update the environment variable as follows:


my-blog/.env
```
DATABASE_URL="postgresql://sammy:your_password@localhost:5432/my-blog?schema=public"

```


Make sure to change the database credentials to the ones you specified in the Docker Compose file. To learn more about the format of the connection URL, visit the Prisma docs.


Once you’re done, save and exit the file.


In this step, you set up your PostgreSQL database with Docker, installed the Prisma CLI, and connected Prisma to the database via an environment variable. In the next section, you’ll define your data model and create your database tables.


# Step 3 — Defining Your Data Model and Creating Database Tables


In this step, you will define your data model in the Prisma schema file. This data model will then be mapped to the database with Prisma Migrate, which will generate and send the SQL statements for creating the tables that correspond to your data model. Since you’re building a blogging application, the main entities of the application will be users and posts.


Prisma uses its own data modeling language to define the shape of your application data.


First, open your schema.prisma file with the following command:


```
nano prisma/schema.prisma


```


Now, add the following model definitions to it. You can place the models at the bottom of the file, right after the generator client block:


my-blog/prisma/schema.prisma
```
. . .
model User {
  id    Int     @default(autoincrement()) @id
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @default(autoincrement()) @id
  title     String
  content   String?
  published Boolean @default(false)
  author    User?   @relation(fields: [authorId], references: [id])
  authorId  Int?
}

```


You are defining two models: User and Post. Each of these has a number of fields that represent the properties of the model. The models will be mapped to database tables; the fields represent the individual columns.


There is a one-to-many relation between the two models, specified by the posts and author relation fields on User and Post. This means that one user can be associated with many posts.


Save and exit the file.


With these models in place, you can now create the corresponding tables in the database using Prisma Migrate. In your terminal, run the following command:


```
npx prisma migrate dev --name init


```


This command creates a new SQL migration on your filesystem and sends it to the database. The --name init option provided to the command specifies the name of the migration and will be used to name the migration folder created on your filesystem.


The output of this command will be similar to this:


```
OutputEnvironment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db": PostgreSQL database "my-blog", schema "public" at "localhost:5432"

PostgreSQL database my-blog created at localhost:5432

The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20201209084626_init/
    └─ migration.sql

Running generate... (Use --skip-generate to skip the generators)

✔ Generated Prisma Client (2.13.0) to ./node_modules/@prisma/client in 75ms

```


The SQL migration file in the prisma/migrations/20201209084626_init/migration.sql directory has the following statements that were executed against the database (the highlighted portion of the filename may differ in your setup):


prisma/migrations/20201209084626_init/migration.sql
```
-- CreateTable
CREATE TABLE "User" (
"id" SERIAL,
    "email" TEXT NOT NULL,
    "name" TEXT,

    PRIMARY KEY ("id")
);

-- CreateTable
CREATE TABLE "Post" (
"id" SERIAL,
    "title" TEXT NOT NULL,
    "content" TEXT,
    "published" BOOLEAN NOT NULL DEFAULT false,
    "authorId" INTEGER,

    PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "User.email_unique" ON "User"("email");

-- AddForeignKey
ALTER TABLE "Post" ADD FOREIGN KEY("authorId")REFERENCES "User"("id") ON DELETE SET NULL ON UPDATE CASCADE;

```


You can also customize the generated SQL migration file if you add the --create-only option to the prisma migrate dev command; for example, you could set up a trigger or use other features of the underlying database.


In this step, you defined your data model in your Prisma schema and created the respective databases tables with Prisma Migrate. In the next step, you’ll install Prisma Client in your project so that you can query the database.


# Step 4 — Exploring Prisma Client Queries in a Plain Script


Prisma Client is an auto-generated and type-safe query builder that you can use to programmatically read and write data in a database from a Node.js or TypeScript application. You will use it for database access within your REST API routes, replacing traditional ORMs, plain SQL queries, custom data access layers, or any other method of talking to a database.


In this step, you will install Prisma Client and become familiar with the queries you can send with it. Before implementing the routes for your REST API in the next steps, you will first explore some of the Prisma Client queries in a plain, executable script.


First, install Prisma Client in your project folder with the Prisma Client npm package:


```
npm install @prisma/client


```


Next, create a new directory called src that will contain your source files:


```
mkdir src


```


Now create a TypeScript file inside of the new directory:


```
nano src/index.ts


```


All of the Prisma Client queries return promises that you can await in your code. This requires you to send the queries inside of an async function.


In the src/index.ts file, add the following boilerplate with an async function that’s executed in your script:


my-blog/src/index.ts
```
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  // ... your Prisma Client queries will go here
}

main()
  .catch((e) => console.error(e))
  .finally(async () => await prisma.$disconnect())

```


Here’s a quick breakdown of the boilerplate:


1. You import the PrismaClient constructor from the previously installed @prisma/client npm package.
2. You instantiate PrismaClient by calling the constructor and obtaining an instance called prisma.
3. You define an async function called main where you’ll add your Prisma Client queries.
4. You call the main function, catching any potential exceptions and ensuring Prisma Client closes any open database connections with prisma.$disconnect().

With the main function in place, you can start adding Prisma Client queries to the script. Adjust index.ts to include the highlighted lines in the async function:


my-blog/src/index.ts
```
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const newUser = await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@prisma.io',
      posts: {
        create: {
          title: 'Hello World',
        },
      },
    },
  })
  console.log('Created new user: ', newUser)

  const allUsers = await prisma.user.findMany({
    include: { posts: true },
  })
  console.log('All users: ')
  console.dir(allUsers, { depth: null })
}

main()
  .catch((e) => console.error(e))
  .finally(async () => await prisma.$disconnect())

```


In this code, you’re using two Prisma Client queries:


- create: Creates a new User record. You use a nested write query to create both a User and Post record in the same query.
- findMany: Reads all existing User records from the database. You provide the include option that additionally loads the related Post records for each User record.

Save and close the file.


Now run the script with the following command:


```
npx ts-node src/index.ts


```


You will receive the following output in your terminal:


```
OutputCreated new user:  { id: 1, email: 'alice@prisma.io', name: 'Alice' }
[
  {
    id: 1,
    email: 'alice@prisma.io',
    name: 'Alice',
    posts: [
      {
        id: 1,
        title: 'Hello World',
        content: null,
        published: false,
        authorId: 1
      }
    ]
  }

```



Note: If you are using a database GUI you can validate that the data was created by reviewing the User and Post tables. Alternatively, you can explore the data in Prisma Studio by running npx prisma studio.

You’ve now used Prisma Client to read and write data in your database. In the remaining steps, you’ll implement the routes for a sample REST API.


# Step 5 — Implementing Your First REST API Route


In this step, you will install Express in your application. Express is a popular web framework for Node.js that you will use to implement your REST API routes in this project. The first route you will implement will allow you to fetch all users from the API using a GET request. The user data will be retrieved from the database using Prisma Client.


Install Express with the following command:


```
npm install express


```


Since you’re using TypeScript, you’ll also want to install the respective types as development dependencies. Run the following command to do so:


```
npm install @types/express --save-dev


```


With the dependencies in place, you can set up your Express application.


Open your main source file again:


```
nano src/index.ts


```


Now delete all the code in index.ts and replace it with the following to start your REST API:


my-blog/src/index.ts
```
import { PrismaClient } from '@prisma/client'
import express from 'express'

const prisma = new PrismaClient()
const app = express()

app.use(express.json())

// ... your REST API routes will go here

app.listen(3000, () =>
  console.log('REST API server ready at: http://localhost:3000'),
)

```


Here’s a quick breakdown of the code:


1. You import PrismaClient and express from the respective npm packages.
2. You instantiate PrismaClient by calling the constructor and obtaining an instance called prisma.
3. You create your Express app by calling express().
4. You add the express.json() middleware to ensure JSON data can be processed properly by Express.
5. You start the server on port 3000.

Now you can implement your first route. Between the calls to app.use and app.listen, add the highlighted lines to create an app.get call:


my-blog/src/index.ts
```
. . .
app.use(express.json())

app.get('/users', async (req, res) => {
  const users = await prisma.user.findMany()
  res.json(users)
})

app.listen(3000, () =>
console.log('REST API server ready at: http://localhost:3000'),
)

```


Once added, save and exit your file. Then start your local web server using the following command:


```
npx ts-node src/index.ts


```


You will receive the following output:


```
OutputREST API server ready at: http://localhost:3000

```


To access the /users route you can point your browser to http://localhost:3000/users or any other HTTP client.


In this tutorial, you will test all REST API routes using curl, a terminal-based HTTP client.



Note: If you prefer to use a GUI-based HTTP client, you can use alternatives like Hoppscotch or Postman.

To test your route, open up a new terminal window or tab (so that your local web server can keep running) and execute the following command:


```
curl http://localhost:3000/users


```


You will receive the User data that you created in the previous step:


```
Output[{"id":1,"email":"alice@prisma.io","name":"Alice"}]

```


The posts array is not included this time because you’re not passing the include option to the findMany call in the implementation of the /users route.


You’ve implemented your first REST API route at /users. In the next step you will implement the remaining REST API routes to add more functionality to your API.


# Step 6 — Implementing the Remaining REST API Routes


In this step, you will implement the remaining REST API routes for your blogging application. At the end, your web server will serve various GET, POST, PUT, and DELETE requests.


The routes you will implement include the following options:





HTTP Method
Route
Description




GET
/feed
Fetches all published posts.


GET
/post/:id
Fetches a specific post by its ID.


POST
/user
Creates a new user.


POST
/post
Creates a new post (as a draft).


PUT
/post/publish/:id
Sets the published field of a post to true.


DELETE
post/:id
Deletes a post by its ID.




You will implement the two remaining GET routes first.


You can stop the server by pressing CTRL+C on your keyboard. Then, you can update your index.ts file by first opening the file for editing:


```
nano src/index.ts


```


Next, add the highlighted lines following the implementation of the /app.get users route:


my-blog/src/index.ts
```
. . .

app.get('/feed', async (req, res) => {
  const posts = await prisma.post.findMany({
    where: { published: true },
    include: { author: true }
  })
  res.json(posts)
})

app.get(`/post/:id`, async (req, res) => {
  const { id } = req.params
  const post = await prisma.post.findUnique({
    where: { id: Number(id) },
  })
  res.json(post)
})

app.listen(3000, () =>
  console.log('REST API server ready at: http://localhost:3000'),
)

```


This code implements the API routes for two GET requests:


- /feed: Returns a list of published posts.
- /post/:id: Returns a specific post by its ID.

Prisma Client is used in both implementations. In the /feed route implementation, the query you send with Prisma Client filters for all Post records where the published column contains the value true. Additionally, the Prisma Client query uses include to also fetch the related author information for each returned post. In the /post/:id route implementation, you pass the ID that is retrieved from the URL’s path in order to read a specific Post record from the database.


Save and exit your file. Then, restart the server using:


```
npx ts-node src/index.ts


```


To test the /feed route, you can use the following curl command in your second terminal session:


```
curl http://localhost:3000/feed


```


Since no posts have been published yet, the response is an empty array:


```
Output[]

```


To test the /post/:id route, you can use the following curl command:


```
curl http://localhost:3000/post/1


```


This command will return the post you initially created:


```
Output{"id":1,"title":"Hello World","content":null,"published":false,"authorId":1}

```


Next, you will implement the two POST routes. In your original terminal session, stop the server with CTRL+C, then open index.ts for editing:


```
nano src/index.ts


```


Add the highlighted lines to index.ts following the implementations of the three GET routes:


my-blog/src/index.ts
```
. . .

app.post(`/user`, async (req, res) => {
  const result = await prisma.user.create({
    data: { ...req.body },
  })
  res.json(result)
})

app.post(`/post`, async (req, res) => {
  const { title, content, authorEmail } = req.body
  const result = await prisma.post.create({
    data: {
      title,
      content,
      published: false,
      author: { connect: { email: authorEmail } },
    },
  })
  res.json(result)
})

app.listen(3000, () =>
  console.log('REST API server ready at: http://localhost:3000'),
)

```


This code implements the API routes for two POST requests:


- /user: Creates a new user in the database.
- /post: Creates a new post in the database.

Like before, Prisma Client is used in both implementations. In the /user route implementation, you pass in the values from the body of the HTTP request to the Prisma Client create query.


The /post route is a more involved. You can’t directly pass in the values from the body of the HTTP request; instead you first need to  extract them manually to pass them to the Prisma Client query. Because the structure of the JSON in the request body does not match the structure that’s expected by Prisma Client, you need to create the expected structure manually.


Once you’re done, save and exit your file.


Restart the server using:


```
npx ts-node src/index.ts


```


To create a new user via the /user route, you can send the following POST request with curl:


```
curl -X POST -H "Content-Type: application/json" -d '{"name":"Bob", "email":"bob@prisma.io"}' http://localhost:3000/user


```


This will create a new user in the database, printing the following output:


```
Output{"id":2,"email":"bob@prisma.io","name":"Bob"}

```


To create a new post via the /post route, you can send the following POST request with curl:


```
curl -X POST -H "Content-Type: application/json" -d '{"title":"I am Bob", "authorEmail":"bob@prisma.io"}' http://localhost:3000/post


```


This will create a new post in the database and connect it to the user with the email bob@prisma.io. It prints the following output:


```
Output{"id":2,"title":"I am Bob","content":null,"published":false,"authorId":2}

```


Finally, you will implement the PUT and DELETE routes. Stop the development server, then open up index.ts with the following command:


```
nano src/index.ts


```


Next, following the implementation of the two POST routes, add the highlighted code:


my-blog/src/index.ts
```
. . .

app.put('/post/publish/:id', async (req, res) => {
  const { id } = req.params
  const post = await prisma.post.update({
    where: { id: Number(id) },
    data: { published: true },
  })
  res.json(post)
})

app.delete(`/post/:id`, async (req, res) => {
  const { id } = req.params
  const post = await prisma.post.delete({
    where: { id: Number(id) },
  })
  res.json(post)
})

app.listen(3000, () =>
  console.log('REST API server ready at: http://localhost:3000'),
)

```


This code implements the API routes for one PUT and one DELETE request:


- /post/publish/:id (PUT): Publishes a post by its ID.
- /post/:id (DELETE): Deletes a post by its ID.

Again, Prisma Client is used in both implementations. In the /post/publish/:id route implementation, the ID of the post to be published is retrieved from the URL and passed to the update query of Prisma Client. The implementation of the /post/:id route to delete a post in the database also retrieves the post ID from the URL and passes it to the delete query of Prisma Client.


Save and exit your file.


Restart the server using:


```
npx ts-node src/index.ts


```


You can test the PUT route with the following curl command:


```
curl -X PUT http://localhost:3000/post/publish/2


```


This command will publish the post with an ID value of 2. If you resend the /feed request, this post will now be included in the response.


Finally, you can test the DELETE route with the following curl command:


```
curl -X DELETE http://localhost:3000/post/1


```


This command will delete the post with an ID value of 1. To validate that the post with this ID has been deleted, you can resend a GET request to the /post/1 route with the following curl command:


```
curl http://localhost:3000/post/1


```


In this step, you implemented the remaining REST API routes for your blogging application. The API now responds to various GET, POST, PUT, and DELETE requests and implements functionality to read and write data in the database.


# Conclusion


In this article, you created a REST API server with a number of different routes to create, read, update, and delete user and post data for a sample blogging application. Inside the API routes, you use the Prisma Client to send the respective queries to your database.


As next steps, you can implement additional API routes or extend your database schema using Prisma Migrate. Visit the Prisma documentation to learn about different aspects of Prisma and explore some ready-to-run example projects using tools such as GraphQL or grPC APIs in the prisma-examples repository.


