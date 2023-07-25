# How To Build a Lightweight Invoicing App with Node  Database and API

```Vue.js``` ```Node.js```

## Introduction


An invoice is a document of goods and services provided that a business can present to customers and clients.


A digital invoicing tool will need to track clients, record services and prices, update the status of paid invoices, and provide an interface for displaying invoices. This will require CRUD (Create, Read, Update, Delete), databases, and routing.



Note: This is Part 1 of a 3-part series. The second tutorial is How To Build a Lightweight Invoicing App with Node: User Interface. The third tutorial is How To Build a Lightweight Invoicing App with Vue and Node: JWT Authentication and Sending Invoices.

In this tutorial, you will build an invoicing application using Vue and NodeJS. This application will perform functions such as creating, sending, editing, and deleting an invoice.


# Prerequisites


To complete this tutorial, you will need:


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.
- SQLite installed locally, which you can do by following How To Install and Use SQLite.
- Downloading and installing a tool like Postman will be required for testing API endpoints.


Note: SQLite currently comes pre-installed on macOS and Mac OS X by default.

This tutorial was verified with Node v16.1.0, npm v7.12.1, and SQLite v3.32.3.


# Step 1 — Setting Up the Project


Now that we have the requirements all set, the next thing to do is to create the backend server for the application. The backend server will maintain the database connection.


Start by creating a directory for the new project:


```
mkdir invoicing-app


```


Navigate to the newly created project directory:


```
cd invoicing-app


```


Then initialize it as a Node project:


```
npm init -y


```


For the server to function appropriately, there are some Node packages that need to be installed. You can install them by running this command:


```
npm install bcrypt@5.0.1 bluebird@3.7.2 cors@2.8.5 express@4.17.1 lodash@4.17.21 multer@1.4.2 sqlite3@5.0.2^> umzug@2.3.0<^>


```


That command installs the following packages:


- bcrypt to hash user passwords
- bluebird to use Promises when writing migrations
- cors for cross-origin resource sharing
- express to power our web application
- lodash for utility methods
- multer to handle incoming form requests
- sqlite3 to create and maintain the database
- umzug as a task runner to run our database migrations


Note: Since the original publication, this tutorial was updated to include lodash for isEmpty(). The middleware library for handling multipart/form-data was changed from connect-multiparty to multer.

Create a server.js file that will house the application logic. In the server.js file, import the necessary modules and create an Express app:


server.js
```
const express = require('express');
const cors = require('cors');
const sqlite3 = require('sqlite3').verbose();
const PORT = process.env.PORT || 3128;

const app = express();

app.use(express.urlencoded({extended: false}));
app.use(express.json());

app.use(cors());

// ...

```


Create a / route to test that the server works:


server.js
```
// ...

app.get('/', function(req, res) {
  res.send('Welcome to Invoicing App.');
});

```


app.listen() tells the server the port to listen to for incoming routes:


server.js
```
// ...

app.listen(PORT, function() {
  console.log(`App running on localhost:${PORT}.`);
});

```


To start the server, run the following in your project directory:


```
node server


```


Your application will now begin to listen to incoming requests.


# Step 2 — Creating and Connecting to the Database Using SQLite


For an invoicing application, a database is needed to store the existing invoices. SQLite is going to be the database client of choice for this application.


Start by creating a database folder:


```
mkdir database


```


Run the sqlite3 client and create an InvoicingApp.db file for your database in this new directory:


```
sqlite3 database/InvoicingApp.db


```


Now that the database has been selected, next thing is to create the needed tables.


This application will use three tables:


- “Users” - This will contain the user data (id, name, email, company_name, password)
- “Invoices” - Store data for an invoice (id, name, paid, user_id)
- “Transactions” - Singular transactions that come together to make an invoice (name, price, invoice_id)

Since the necessary tables have been identified, the next step is to run the queries to create the tables.


Migrations are used to keep track of changes in a database as the application grows. To do this, create a migrations folder in the database directory.


```
mkdir database/migrations


```


This will be the location of all the migration files.


Now, create a 1.0.js file in the migrations folder. This naming convention is to keep track of the newest changes.


In the 1.0.js file, you first import the node modules:


database/migrations 1.0.js
```
"use strict";
const path = require('path');
const Promise = require('bluebird');
const sqlite3 = require('sqlite3');

// ...

```


Then, export an up function that will be executed when the migration file is run and a down function to reverse the changes to the database.


database/migrations/1.0.js
```
// ...

module.exports = {
  up: function() {
    return new Promise(function(resolve, reject) {
      let db = new sqlite3.Database('./database/InvoicingApp.db');

      db.run(`PRAGMA foreign_keys = ON`);

      // ...

```


In the up function, the connection is first made to the database. Then the foreign keys are enabled on the sqlite database. In SQLite, foreign keys are disabled by default to allow for backwards compatibility, so the foreign keys have to be enabled on every connection.


Next, specify the queries to create the tables:


database/migrations/1.0.js
```
// ...

      db.serialize(function() {
        db.run(`CREATE TABLE users (
          id INTEGER PRIMARY KEY,
          name TEXT,
          email TEXT,
          company_name TEXT,
          password TEXT
        )`);

        db.run(`CREATE TABLE invoices (
          id INTEGER PRIMARY KEY,
          name TEXT,
          user_id INTEGER,
          paid NUMERIC,
          FOREIGN KEY(user_id) REFERENCES users(id)
        )`);

        db.run(`CREATE TABLE transactions (
          id INTEGER PRIMARY KEY,
          name TEXT,
          price INTEGER,
          invoice_id INTEGER,
          FOREIGN KEY(invoice_id) REFERENCES invoices(id)
        )`);
      });

      db.close();
    });
  }

}

```


The serialize() function is used to specify that the queries will be run sequentially and not simultaneously.


Once the migration files have been created, the next step is running them to make the changes in the database. To do this, create a scripts folder from the root of your application:


```
mkdir scripts


```


Then create a file called migrate.js in this new directory. And add the following to the migrate.js file:


scripts/migrate.js
```
const path = require('path');
const Umzug = require('umzug');

let umzug = new Umzug({
  logging: function() {
    console.log.apply(null, arguments);
  },
  migrations: {
    path: './database/migrations',
    pattern: /\.js$/
  },
  upName: 'up'
});

// ...

```


First, the needed node modules are imported. Then a new umzug object is created with the configurations. The path and pattern of the migrations scripts are also specified. To learn more about the configurations, reference the umzug README.


To also give some verbose feedback, create a function to log events as shown below and then finally execute the up function to run the database queries specified in the migrations folder:


scripts/migrate.js
```
// ...

function logUmzugEvent(eventName) {
  return function(name, migration) {
    console.log(`${name} ${eventName}`);
  };
}

// using event listeners to log events
umzug.on('migrating', logUmzugEvent('migrating'));
umzug.on('migrated', logUmzugEvent('migrated'));
umzug.on('reverting', logUmzugEvent('reverting'));
umzug.on('reverted', logUmzugEvent('reverted'));

// this will run your migrations
umzug.up().then(console.log('all migrations done'));

```


Now, to execute the script, go to your terminal and in the root directory of your application, run:


```
node scripts/migrate.js


```


You will see output similar to the following:


```
Outputall migrations done
== 1.0: migrating =======
1.0 migrating

```


At this point, running the migrate.js script has applied the 1.0.js configuration to InvoicingApp.db.


# Step 3 — Creating Application Routes


Now that the database is adequately set up, the next thing is to go back to the server.js file and create the application routes. For this application, the following routes will be made available:





URL
METHOD
FUNCTION




/register
POST
To register a new user


/login
POST
To log in an existing user


/invoice
POST
To create a new invoice


/invoice/user/{user_id}
GET
To fetch all the invoices for a user


/invoice/user/{user_id}/{invoice_id}
GET
To fetch a certain invoice


/invoice/send
POST
To send invoice to client




## POST /register


To register a new user, a POST request will be made to the /register route of your server.


Revisit server.js and add the following lines of code:


server.js
```
// ...

const _ = require('lodash');

const multer  = require('multer');
const upload = multer();

const bcrypt = require('bcrypt');
const saltRounds = 10;

// POST /register - begin

app.post('/register', upload.none(), function(req, res) {
  // check to make sure none of the fields are empty
  if (
    _.isEmpty(req.body.name)
    || _.isEmpty(req.body.email)
    || _.isEmpty(req.body.company_name)
    || _.isEmpty(req.body.password)
  ) {
    return res.json({
      "status": false,
      "message": "All fields are required."
    });
  }

  // any other intended checks

// ...

```


A check is made to see if any of the fields are empty and if the data sent matches all the specifications. If an error occurs, an error message is sent to the user as a response. If not, the password is hashed and the data is then stored in the database and a response is sent to the user informing them that they are registered.


server.js
```
// ...

    bcrypt.hash(req.body.password, saltRounds, function(err, hash) {

    let db = new sqlite3.Database('./database/InvoicingApp.db');

    let sql = `INSERT INTO
                users(
                  name,
                  email,
                  company_name,
                  password
                )
                VALUES(
                  '${req.body.name}',
                  '${req.body.email}',
                  '${req.body.company_name}',
                  '${hash}'
                )`;

    db.run(sql, function(err) {
      if (err) {
        throw err;
      } else {
        return res.json({
          "status": true,
          "message": "User Created."
        });
      }
    });

    db.close();
  });

});

// POST /register - end

```


Now, if we use a tool like Postman to send a POST request to /register with name, email, company_name, and password, it will create a new user:





Key
Value




name
Test User


email
example@example.com


company_name
Test Company


password
password




We can use a query and display the Users table to verify the user creation:


```
select * from users;


```


The database now contains a newly created user:


```
Output1|Test User|example@example.com|Test Company|[hashed password]

```


Your /register route is now verified.


## POST /login


If an existing user tries to log in to the system using the /login route, they need to provide their email address and password. Once they do that, the route handles the request as follows:


server.js
```
// ...

// POST /login - begin

app.post('/login', upload.none(), function(req, res) {
  let db = new sqlite3.Database('./database/InvoicingApp.db');

  let sql = `SELECT * from users where email='${req.body.email}'`;

  db.all(sql, [], (err, rows) => {
    if (err) {
      throw err;
    }

    db.close();

    if (rows.length == 0) {
      return res.json({
        "status": false,
        "message": "Sorry, wrong email."
      });
    }

// ...

```


A query is made to the database to fetch the record of the user with a particular email. If the result returns an empty array, then it means that the user doesn’t exist and a response is sent informing the user of the error.


If the database query returns user data, a further check is made to see if the password entered matches that password in the database. If it does, then a response is sent with the user data.


server.js
```
// ...

    let user = rows[0];

    let authenticated = bcrypt.compareSync(req.body.password, user.password);

    delete user.password;

    if (authenticated) {
      return res.json({
        "status": true,
        "user": user
      });
    }

    return res.json({
      "status": false,
      "message": "Wrong password. Please retry."
    });
  });

});

// POST /login - end

// ...

```


When the route is tested, you will receive either a successful or failed result.


Now, if we use a tool like Postman to send a POST request to /login with email and password, it will send back a response.





Key
Value




email
example@example.com


password
password




Since this user exists in the database, we get the following response:


```
Output{
    "status": true,
    "user": {
        "id": 1,
        "name": "Test User",
        "email": "example@example.com",
        "company_name": "Test Company"
    }
}

```


Your /login route is now verified.


## POST /invoice


The /invoice route handles the creation of an invoice. Data passed to the route will include the user ID, name of the invoice, and invoice status. It will also include the singular transactions to make up the invoice.


The server handles the request as follows:


server.js
```
// ...

// POST /invoice - begin

app.post('/invoice', upload.none(), function(req, res) {
  // validate data
  if (_.isEmpty(req.body.name)) {
    return res.json({
      "status": false,
      "message": "Invoice needs a name."
    });
  }

  // perform other checks

// ...

```


First, the data sent to the server is validated. Then a connection is made to the database for the subsequent queries.


server.js
```
// ...

  // create invoice
  let db = new sqlite3.Database('./database/InvoicingApp.db');

  let sql = `INSERT INTO invoices(
                name,
                user_id,
                paid
              )
              VALUES(
                '${req.body.name}',
                '${req.body.user_id}',
                0
              )`;

// ...

```


The INSERT query needed to create the invoice is written and then executed. Afterward, the singular transactions are inserted into the transactions table with the invoice_id as a foreign key to reference them.


server.js
```
// ...

  db.serialize(function() {
    db.run(sql, function(err) {
      if (err) {
        throw err;
      }

      let invoice_id = this.lastID;

      for (let i = 0; i < req.body.txn_names.length; i++) {
        let query = `INSERT INTO
                      transactions(
                        name,
                        price,
                        invoice_id
                      ) VALUES(
                        '${req.body.txn_names[i]}',
                        '${req.body.txn_prices[i]}',
                        '${invoice_id}'
                      )`;

        db.run(query);
      }

      return res.json({
        "status": true,
        "message": "Invoice created."
      });
    });
  });

});
// POST /invoice - end

// ...

```


Now, if we use a tool like Postman to send a POST request to /invoice with name, user_id, txn_names, and txn_prices, it will create a new invoice and record the transactions:





Key
Value




name
Test Invoice


user_id
1


txn_names
iPhone


txn_prices
600


txt_names
MacBook


txn_prices
1700




Then, check the Invoices table:


```
select * from invoices;


```


Observe the following result:


```
Output1|Test Invoice|1|0

```


Run the following command:


```
select * from transactions;


```


Observe the following result:


```
Output1|iPhone|600|1
2|Macbook|1700|1

```


Your /invoice route is now verified.


## GET /invoice/user/{user_id}


Now, when a user wants to see all the created invoices, the client will make a GET request to the /invoice/user/:id route. The user_id is passed as a route parameter. The request is handled as follows:


index.js
```
// ...

// GET /invoice/user/:user_id - begin

app.get('/invoice/user/:user_id', upload.none(), function(req, res) {
  let db = new sqlite3.Database('./database/InvoicingApp.db');

  let sql = `SELECT * FROM invoices WHERE user_id='${req.params.user_id}' ORDER BY invoices.id`;

  db.all(sql, [], (err, rows) => {
    if (err) {
      throw err;
    }

    return res.json({
      "status": true,
      "invoices": rows
    });
  });
});

// GET /invoice/user/:user_id - end

// ...

```


A query is run to fetch all the invoices and the transactions related to the invoice belonging to a particular user.


Consider a request for all the invoices for a user:


```
localhost:3128/invoice/user/1

```


It will respond with the following data:


```
Output{"status":true,"invoices":[{"id":1,"name":"Test Invoice","user_id":1,"paid":0}]}

```


Your /invoice/user/:user_id route is now verified.


## GET /invoice/user/{user_id}/{invoice_id}


To fetch a specific invoice, a GET request is made with the user_id and invoice_id to the /invoice/user/{user_id}/{invoice_id} route. The request is handled as follows:


index.js
```
// ...

// GET /invoice/user/:user_id/:invoice_id - begin

app.get('/invoice/user/:user_id/:invoice_id', upload.none(), function(req, res) {
  let db = new sqlite3.Database('./database/InvoicingApp.db');

  let sql = `SELECT * FROM invoices LEFT JOIN transactions ON invoices.id=transactions.invoice_id WHERE user_id='${req.params.user_id}' AND invoice_id='${req.params.invoice_id}' ORDER BY transactions.id`;

  db.all(sql, [], (err, rows) => {
    if (err) {
      throw err;
    }

    return res.json({
      "status": true,
      "transactions": rows
    });
  });
});

// GET /invoice/user/:user_id/:invoice_id - end

// set application port
// ...

```


A query is run to fetch a single invoice and the transactions related to the invoice belonging to the user.


Consider a request for a specific invoice for a user:


```
localhost:3128/invoice/user/1/1

```


It will respond with the following data:


```
Output{"status":true,"transactions":[{"id":1,"name":"iPhone","user_id":1,"paid":0,"price":600,"invoice_id":1},{"id":2,"name":"Macbook","user_id":1,"paid":0,"price":1700,"invoice_id":1}]}

```


Your /invoice/user/:user_id/:invoice_id route is now verified.


# Conclusion


In this tutorial, you set up your server with all the needed routes for a lightweight invoicing application.


Continue your learning with How To Build a Lightweight Invoicing App with Node: User Interface.


