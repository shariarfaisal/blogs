# How To Use SQLite with Node js on Ubuntu 22 04

```Databases``` ```Development``` ```Node.js```

The author selected /dev/color to receive a donation as part of the Write for DOnations program.


## Introduction


SQLite is a popular open-source SQL database engine for storing data. It is serverless, meaning it does not need a server to run; instead it reads and writes data to a file that resides on the computer disk. Furthermore, SQLite doesn’t require any configurations; this makes it more portable and a popular choice for embedded systems, desktop/mobile apps, and prototyping, among other things.


To use SQLite with Node.js, you need a database client that connects to an SQLite database and sends SQL statements from your application to the database for execution. One of the popular choices is the node-sqlite3 package that provides asynchronous bindings for SQLite 3.


In this tutorial, you’ll use node-sqlite3 to create a connection with an SQLite database. Next, you’ll create a Node.js app that creates a table and inserts data into the database. Finally, you’ll modify the app to use node-sqlite3 to retrieve, update, and delete data from the database.


# Prerequisites


To follow this tutorial, you will need:


- 
A Node.js development environment set up on your system. If you are using Ubuntu 22.04, install the latest version of Node.js by following Option 3 of our tutorial How To Install Node.js on Ubuntu 22.04. For other systems, consult our tutorial series How to Install Node.js and Create a Local Development Environment.

- 
SQLite3 installed on your development environment. Follow Step 1 of our tutorial How To Install and Use SQLite on Ubuntu 20.04.

- 
Basic knowledge of how to create tables and write SQL queries to retrieve and modify data in a table. Follow Steps 2 through 6 of our tutorial How To Install and Use SQLite on Ubuntu 20.04.

- 
Familiarity with how to write a Node.js program, which you can find in our tutorial How To Write and Run Your First Program in Node.js.


# Step 1 — Setting Up the Project Directory


In this step, you’ll create the project directory and download node-sqlite3 as a dependency.


To begin, create a directory using the mkdir command. It is called sqlite_demo for the sake of this tutorial, but you can replace the name with one of your choosing:


```
mkdir sqlite_demo


```


Next, change into the newly created directory using the cd command:


```
cd sqlite_demo


```


Initialize the project directory as an npm package using the npm command:


```
npm init -y


```


The command creates a package.json file, which holds important metadata for your project. The -y option instructs npm to accept all defaults.


After running the command, the following output will display on your screen:


```
OutputWrote to /home/sammy/sqlite_demo/package.json:

{
  "name": "sqlite_demo",
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


The output indicates that the package.json file has been created, which contains properties that record important metadata for your project. Some of the important options are:


- name: the name of your project.
- version: your project version.
- main: the starting point for your project.

You can leave the default options as they are, but feel free to modify the property values to suit your preference. For more information about the properties, consult npm’s package.json documentation.


Next, install the node-sqlite3 package with npm install:


```
npm install sqlite3


```


Upon installing the package, the output will display as follows:


```
Outputadded 104 packages, and audited 105 packages in 9s

5 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities

```


Now that you’ve installed node-sqlite3, you’ll use it to connect to an SQLite database in the next section.


# Step 2 — Connecting to an SQLite Database


In this step, you will use node-sqlite3 to connect your Node.js program to an SQLite database that you will create, which contains different sharks and their attributes. To establish a database connection, the node-sqlite3 package provides a Database class. When instantiated, the class creates an SQLite database file on your computer disk and connects to it. Once the connection is established, you’ll create a table for your application, which in later sections you will use to insert, retrieve, or update data.


Using nano, or your favorite text editor, create and open the db.js file:


```
nano db.js


```


In your db.js file, add the following code to establish a connection with the SQLite database:


sqlite_demo/db.js
```
const sqlite3 = require("sqlite3").verbose();
const filepath = "./fish.db";

function createDbConnection() {
  const db = new sqlite3.Database(filepath, (error) => {
    if (error) {
      return console.error(error.message);
    }
  });
  console.log("Connection with SQLite has been established");
  return db;
}

```


In the first line, you import the node-sqlite3 module into your program file. In the second line, you set the variable filepath with the path you want your SQLite database to reside in and the name of the database file, which in this case is fish.db.


In the next line, you define the createDbConnection() function that establishes a connection with the SQLite database. Within the function, you instantiate the sqlite3.Database() class with the new keyword. The class takes two arguments: filepath and a callback.


The first argument, filepath, accepts the name and path to the SQLite database, which is the ./fish.db here. The second argument is a callback that runs once the database has been created and a connection to the database has been established. The callback takes an error parameter that is set to an error object if an error occurs when trying to establish a database connection. Within the callback, you use the if statement to check if there is an error. If the condition is true, you use the console.error() method to log the error message.


Now, when you create an instance using the sqlite3.Database() class, it creates an SQLite database file in your project directory and returns a database object that is stored in the db variable. The database object provides methods that you can use to pass SQL statements that create tables and insert, retrieve, or modify data.


Finally, you invoke console.log() to log a success message and return the database object in the db variable.


Next, add the highlighted code to create a function that creates a table:


sqlite_demo/db.js
```
...
function createDbConnection() {
    ...
}

function createTable(db) {
  db.exec(`
  CREATE TABLE sharks
  (
    ID INTEGER PRIMARY KEY AUTOINCREMENT,
    name   VARCHAR(50) NOT NULL,
    color   VARCHAR(50) NOT NULL,
    weight INTEGER NOT NULL
  );
`);
}

```


The createTable() function creates a table in the SQLite database. It takes a database object db as a parameter. Within the createTable() function, you invoke the exec() method of the db database object that sends the given SQL statement to the database to be executed. The exec() method is only used for queries that do not return result rows.


The CREATE TABLE sharks... SQL statement passed to the exec() method creates a table sharks with the following fields:


- ID: stores values of INTEGER datatype. The PRIMARY KEY constraint designates the column as the primary key and AUTOINCREMENT instructs SQLite to automatically increment the ID column values for each row in the table.
- name: details the name of the shark using the VARCHAR datatype that has a maximum of 50 characters. The NOT NULL constraint ensures that the field cannot store NULL values.
- color: represents the color of the shark using the VARCHAR datatype with a maximum of 50 characters. The NOT NULL constraint signifies that the field should not accept NULL values.
- weight: stores the weight of the shark in kilograms using the INTEGER datatype, and uses the NOT NULL constraint to ensure that NULL values are not allowed.

In the same db.js file, add the highlighted code to invoke the createTable() function:


sqlite_demo/db.js
```
function createDbConnection() {
  const db = new sqlite3.Database(filepath, (error) => {
    if (error) {
      return console.error(error.message);
    }
    createTable(db);
  });
  console.log("Connection with SQLite has been established");
  return db;
}


function createTable(db) {
    ...
}

```


When the callback runs, you call the createTable() function with the db database object as an argument.


Next, add the following line to call the createDbConnection() function:


```
...
function createDbConnection() {
    ...
}


function createTable(db) {
    ...
}

module.exports = createDbConnection();

```


In the preceding code, you call the createDbConnection() function, which establishes the connection to the database and returns a database object. You then use module.exports to export the database object so that you can reference it in other files.


Your file will now contain the following:


sqlite_demo/db.js
```
const sqlite3 = require("sqlite3").verbose();
const filepath = "./fish.db";

function createDbConnection() {
  const db = new sqlite3.Database(filepath, (error) => {
    if (error) {
      return console.error(error.message);
    }
    createTable(db);
  });
  console.log("Connection with SQLite has been established");
  return db;
}

function createTable(db) {
  db.exec(`
  CREATE TABLE sharks
  (
    ID INTEGER PRIMARY KEY AUTOINCREMENT,
    name   VARCHAR(50) NOT NULL,
    color   VARCHAR(50) NOT NULL,
    weight INTEGER NOT NULL
  );
`);
}

module.exports = createDbConnection();

```


Save and exit your file. If using nano, press CTRL+X to exit, press y to save the changes you made, and press ENTER to confirm the filename.


Run the db.js file using the node command:


```
node db.js


```


The output will reveal that the database connection has been established successfully:


```
OutputConnection with SQLite has been established

```


Next, check if the fish.db database file has been created using the ls command:


```
ls


```


```
Outputdb.js  fish.db  node_modules  package-lock.json  package.json

```


The appearance of the fish.db database file in the output confirms that the database was created successfully.


Now, each time you run the db.js file, it will call the createTable() function to create a table in the database. Attempting to create a table that already exists triggers SQLite to throw an error. To see this, rerun the db.js file with the node command:


```
node db.js


```


This time, you will get an error as revealed in the following output:


```
OutputConnection with SQLite has been established
undefined:0


[Error: SQLITE_ERROR: table sharks already exists
Emitted 'error' event on Database instance at:
] {
  errno: 1,
  code: 'SQLITE_ERROR'
}

Node.js v17.6.0

```


The error message indicates that the sharks table already exists. This is because when you run the node command for the first time, the fish database was created, as well as the sharks table. When you rerun the command, the createTable() function is run again for the second time, which triggers an error since the table already exists.


This error would also be triggered anytime you want to use the database object methods to manipulate the database in other files. For example, in the next step, you’ll create a file that inserts data into the database. To use the database object, you will import the db.js file and call the relevant method for inserting data into the database. When you run the file, it will in turn run the db.js, which will trigger the same error.


To remedy this, you will use the existsSync() method of the fs module to check for the existence of the database file fish.db in the project directory. If the database file exists, you will establish the connection to the database without calling the createTable() function. If it does not exist, you will establish the connection and call the createTable() function.


To do this, open the db.js in your editor once more:


```
nano db.js


```


In your db.js file, add the highlighted code to check for the existence of the database file:


sqlite_demo/db.js
```
const fs = require("fs");
const sqlite3 = require("sqlite3").verbose();
const filepath = "./fish.db";

function createDbConnection() {
  if (fs.existsSync(filepath)) {
    return new sqlite3.Database(filepath);
  } else {
    const db = new sqlite3.Database(filepath, (error) => {
      if (error) {
        return console.error(error.message);
      }
      createTable(db);
    });
    console.log("Connection with SQLite has been established");
    return db;
  }
}

```


First, you import the fs module used for interacting with the file system. Second, in the added if statement, you invoke the fs.existSync() method to check for the existence of the file in the given argument, which is the database file ./fish.db here. If the file exists, you call sqlite3.Database() with the database file path and omit the callback. However, if the file does not exist, you create the database instance and invoke the createTable() function in the callback to create the table in the database.


At this point, the complete file will now display as follows:


sqlite_demo/db.js
```
const fs = require("fs");
const sqlite3 = require("sqlite3").verbose();
const filepath = "./fish.db";

function createDbConnection() {
  if (fs.existsSync(filepath)) {
    return new sqlite3.Database(filepath);
  } else {
    const db = new sqlite3.Database(filepath, (error) => {
      if (error) {
        return console.error(error.message);
      }
      createTable(db);
    });
    console.log("Connection with SQLite has been established");
    return db;
  }
}

function createTable(db) {
  db.exec(`
  CREATE TABLE sharks
  (
    ID INTEGER PRIMARY KEY AUTOINCREMENT,
    name   VARCHAR(50) NOT NULL,
    color   VARCHAR(50) NOT NULL,
    weight INTEGER NOT NULL
  );
`);
}

module.exports = createDbConnection();

```


Save and close your file once you are done making the changes.


To make sure that the db.js file doesn’t throw an error when run multiple times, delete the fish.db file with the rm command to start afresh:


```
rm fish.db


```


Run the db.js file:


```
node db.js


```


```
OutputConnection with SQLite has been established

```


Now, confirm that db.js connects to the database and doesn’t attempt to create the table again for all the subsequent reruns of the db.js file by running the file again:


```
node db.js


```


You will now notice that you won’t get the error anymore.


Now that you established the connection to the SQLite database and created a table, you’ll insert data into the database.


# Step 3 — Inserting Data into the SQLite Database


In this step, you will create a function that inserts data in the SQLite database using the node-sqlite3 module. You’ll pass the data you want to insert as command-line arguments to the program.


Create and open the insertData.js file in your text editor:


```
nano insertData.js


```


In your insertData.js file, add the following code to get command-line arguments:


sqlite_demo/insertData.js
```
const db = require("./db");

function insertRow() {
  const [name, color, weight] = process.argv.slice(2);
}

```


In the first line, you import the database object that you exported in the db.js file in the previous step. In the second line, you define the function insertRow(), which you will soon use to insert data into the table. In the function, process.argv returns all the command-line arguments in an array. The first element on index 0 contains the path to Node. The second element on index 1 stores the JavaScript program’s filename. All the subsequent elements starting from index 2 contain the command-line arguments you passed to the file. To skip the first two arguments, you use JavaScript’s slice() method to make a shallow copy of the array and return elements from index 2 to the end of the array.


Next, add the highlighted code to insert data into the database:


sqlite_demo/insertData.js
```
const db = require("./db");

function insertRow() {
  const [name, color, weight] = process.argv.slice(2);
  db.run(
    `INSERT INTO sharks (name, color, weight) VALUES (?, ?, ?)`,
    [name, color, weight],
    function (error) {
      if (error) {
        console.error(error.message);
      }
      console.log(`Inserted a row with the ID: ${this.lastID}`);
    }
  );
}

```


In the preceding code, you call the db.run() method, which takes three arguments: the SQL statement, an array, and a callback. The first argument, INSERT INTO sharks..., is an SQL statement that inserts data into the database. The VALUES statement in the INSERT statement takes a comma-separated list of values that need to be inserted. Notice that you are passing the ? placeholders instead of passing the values directly. This is to avoid SQL injection attacks. During execution, SQLite will automatically substitute the placeholders with the values passed in the db.run() method second argument, which is an array containing command-line argument values.


Finally, the third argument for the db.run() method is a callback that runs when the data has been successfully inserted into the table. If there is an error, the error message is logged in the console. If the insertion was successful, you log a success message with the ID of the newly inserted row that this.lastID returned.


Now, add the highlighted line to call the insertRow() function:


sqlite_demo/insertData.js
```
const db = require("./db");

function insertRow() {
  const [name, color, weight] = process.argv.slice(2);
  db.run(
    `INSERT INTO sharks (name, color, weight) VALUES (?, ?, ?)`,
    [name, color, weight],
    function (error) {
      if (error) {
        console.error(error.message);
      }
      console.log(`Inserted a row with the ID: ${this.lastID}`);
    }
  );
}

insertRow();

```


Save and close your file, then run the file with the shark name, color, and weight arguments:


```
node insertData.js sammy blue 1900


```


The output indicates the row has been inserted into the table with the primary ID 1:


```
OutputInserted a row with the ID: 1

```


Run the command again with different arguments:


```
node insertData.js max white 2100


```


```
OutputInserted a row with the ID: 2

```


When you run the preceding commands, two rows will be created in the sharks table.


Now that you can insert data into the SQLite database, next you’ll retrieve the data from the database.


# Step 4 — Retrieving Data from the SQLite Database


In this step, you’ll use the node-sqlite3 module to retrieve all the data stored in the sharks table in the SQLite database and log them into the console.


First, open the listData.js file:


```
nano listData.js


```


In your listData.js file, add the following code to retrieve all rows:


sqlite_demo/listData.js
```
const db = require("./db");

function selectRows() {
  db.each(`SELECT * FROM sharks`, (error, row) => {
    if (error) {
      throw new Error(error.message);
    }
    console.log(row);
  });
}

```


First, you import the database object in the db.js file. Second, you define the selectRows() function that retrieves all the rows in the SQLite database. Within the function, you use the each() method of the database object db to retrieve rows from the database one by one. The each() method takes two arguments: an SQL statement and a callback.


The first argument SELECT returns all rows in the sharks table. The second argument is a callback that runs each time a row is retrieved from the database. In the callback, you check for an error. If there is an error, you use the throw statement to create a custom error. If no error occurred during retrieval, the data is logged in the console.


Now, add the highlighted code to call the selectRows() function:


sqlite_demo/listData.js
```
const db = require("./db");

function selectRows() {
  db.each(`SELECT * FROM sharks`, (error, row) => {
    if (error) {
      throw new Error(error.message);
    }
    console.log(row);
  });
}

selectRows();

```


Save and exit your file, then run the file:


```
node listData.js


```


Upon running the command, you’ll notice output that displays as follows:


```
Output{ ID: 1, name: 'sammy', color: 'blue', weight: 1900 }
{ ID: 2, name: 'max', color: 'white', weight: 2100 }

```


The output displays all the rows you inserted in the sharks table in the previous step. The node-sqlite3 module converts each SQL result into a JavaScript object.


Now that you can retrieve data from the SQLite database, you’ll update the data in the SQLite database.


# Step 5 — Modifying Data in the SQLite Database


In this step, you’ll use the node-sqlite3 module to update a row in the SQLite database. To do that, you’ll pass the program a command-line argument containing the primary ID of the row you want to modify, as well as the value you want to update the row to.


Create and open the updateData.js file in your text editor:


```
nano updateData.js


```


In your updateData.js file, add the following code to update a record:


sqlite_demo/updateData.js
```
const db = require("./db");

function updateRow() {
  const [id, name] = process.argv.slice(2);
  db.run(
    `UPDATE sharks SET name = ? WHERE id = ?`,
    [name, id],
    function (error) {
      if (error) {
        console.error(error.message);
      }
      console.log(`Row ${id} has been updated`);
    }
  );
}

updateRow();

```


First, you import the database object from the db.js file. Second, you define the updateRow function that updates a row in the database. Within the function, you unpack the command-line arguments into the id and name variables. The id variable contains the primary ID of the row you want to update, and name contains the value you want the name field to reflect.


Next, you invoke the db.run() function with the following arguments: an SQL statement and a callback. The UPDATE SQL statement changes the name column from the current value to the value passed in the name variable. The WHERE clause ensures that only the row with the ID in the id variable is updated. The db.run() method takes a second argument, which is a callback that runs once the value has been updated.


Finally, you call the updateRow() function.


Save and close your file when you are finished making the changes.


Run the updateData.js file with the id of the row you want to change and the new name:


```
node updateData.js 2 sonny


```


```
OutputRow 2 has been updated

```


Verify that the name has been changed:


```
node listData.js


```


When you run the command, your output will resemble the following:


```
Output{ ID: 1, name: 'sammy', color: 'blue', weight: 1900 }
{ ID: 2, name: 'sonny', color: 'white', weight: 2100 }

```


The output indicates that the row with the ID 2 now has sonny as the value for its name field.


With that, you can now update a row in the database. Next, you’ll delete data from the SQLite database.


# Step 6 — Deleting Data in the SQLite Database


In this section, you’ll use node-sqlite3 to select and delete a row from a table in the SQLite database.


Create and open the deleteData.js file in your text editor:


```
nano deleteData.js


```


In your deleteData.js file, add the following code to delete a row in the database:


sqlite_demo/deleteData.js
```
const db = require("./db");

async function deleteRow() {
  const [id] = process.argv.slice(2);
  db.run(`DELETE FROM sharks WHERE id = ?`, [id], function (error) {
    if (error) {
      return console.error(error.message);
    }
    console.log(`Row with the ID ${id} has been deleted`);
  });
}

deleteRow();

```


First, you import the database object in the db.js file. Second, you define the deleteRow() that deletes a row in the table sharks. Within the function, you unpack the primary key ID and store it in the id variable. Next, you invoke db.run(), which takes two arguments. The first argument is an SQL statement DELETE from sharks... that deletes a row in the table sharks. The WHERE clause ensures that only the row with the ID in the id variable is deleted. The second argument is a callback that runs once the row has been deleted. If successful, the function logs a success message; otherwise, it logs the error in the console.


Finally, you call the deleteRow() function.


Save and close your file, then run the following command:


```
node deleteData.js 2


```


```
OutputRow with the ID 2 has been deleted

```


Next, confirm that the row has been deleted:


```
node listData.js


```


When you run the command, your output will display similar to the following:


```
Output{ ID: 1, name: 'sammy', color: 'blue', weight: 1900 }

```


The row with the ID 2 is no longer in the results. This confirms that the row has been deleted.


With that, you can now delete rows in the SQLite database using the node-sqlite3 module.


# Conclusion


In this article, you created a Node.js app that uses the node-sqlite3 module to connect to and create a table on the SQLite database. Next, you modified the app to insert, retrieve, and update data in the database. Finally, you modified the app to delete data in the database.


For more insight into node-sqlite3 methods, visit the node-sqlite3 Wiki page. To learn about SQLite, consult the SQLite documentation. If you want to know how SQLite compares with other SQL databases, consult our tutorial SQLite vs MySQL vs PostgreSQL: A Comparison Of Relational Database Management Systems. To continue your Node.js journey, visit the  How To Code in Node.js series.


