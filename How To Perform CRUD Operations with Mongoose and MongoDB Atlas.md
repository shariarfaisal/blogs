# How To Perform CRUD Operations with Mongoose and MongoDB Atlas

```MongoDB``` ```Node.js```

## Introduction


Mongoose is one of the fundamental tools for manipulating data for a Node.js and MongoDB backend.


In this article, you will be looking into using Mongoose with the MongoDB Atlas remote database. The example in this tutorial will consist of a list of food and their caloric values. Users will be able to create new items, read items, update items, and delete items.


# Prerequisites


- Familiarity with using Express for a server. You can consult this article on Express and the official Express documentation.
- Familiarity with setting up REST API without any authentication. You can consult this article on routing with Express and this article on async and await.

Downloading and installing a tool like Postman is recommended for testing API endpoints.


This tutorial was verified with Node v15.3.0, npm v7.4.0, express v4.17.1, mongoose v5.11.12, and MongoDB v4.2.


## MongoDB Atlas Setup


This project also requires a MongoDB Atlas account.


After creating an account and signing in, follow these steps to deploy a free tier cluster.


Once you have set a cluster, a database user, and an IP address you will be prepared for later acquiring the connection string as you set up the rest of your project.


# Step 1 — Setting Up the Project


In this section, you will create a directory for your project and install dependencies.


Create a new directory for your project:


```
mkdir mongoose-mongodb-atlas-example


```


Navigate to the newly created directory:


```
cd mongoose-mongodb-atlas-example


```


At this point, you can initialize a new npm project:


```
npm init -y


```


Next, install express and mongoose:


```
npm install express@4.17.1 mongoose@5.11.12


```


At this point, you will have a new project with express and mongoose.


# Step 2 — Setting Up the Server


In this section, you will create a new file to run the Express server, connect to the MongoDB Atlas database, and import future routes.


Create a new server.js file and add the following lines of code:


server.js
```
const express = require("express");
const mongoose = require("mongoose");
const foodRouter = require("./routes/foodRoutes.js");

const app = express();

app.use(express.json());

mongoose.connect(
  "mongodb+srv://madmin:<password>@clustername.mongodb.net/<dbname>?retryWrites=true&w=majority",
  {
    useNewUrlParser: true,
    useFindAndModify: false,
    useUnifiedTopology: true
  }
);

app.use(foodRouter);

app.listen(3000, () => {
  console.log("Server is running...");
});

```


Pay attention to the connection string. This is the connection string that is provided by MongoDB Atlas. You will need to replace the administrator account (madmin), password, cluster name (clustername), and database name (dbname) with the values that are relavent to your cluster:


```
mongodb+srv://madmin:<password>@clustername.mongodb.net/<dbname>?retryWrites=true&w=majority

```


mongoose.connect() will take the connection string and an object of configuration options. For the purpose of this tutorial, the useNewUrlParser, useFindAndModify, and useUnifiedTopology configuration settings are necessary to avoid a deprecation warning.


At this point, you have the start of an Express server. You will next need to define the schema and handle routes.


# Step 3 — Building the Schema


First, you will need to have a pattern to structure your data on to and these patterns are referred to as schemas. Schemas allow you to decide exactly what data you want and what options you want the data to have as an object.


In this tutorial, you will use the mongoose.model method to make it usable with actual data and export it as a variable you can use in foodRoutes.js.


Create a new models directory:


```
mkdir models


```


Inside of this new directory, create a new food.js file and add the following lines of code:


./models/food.js
```
const mongoose = require("mongoose");

const FoodSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    lowercase: true,
  },
  calories: {
    type: Number,
    default: 0,
    validate(value) {
      if (value < 0) throw new Error("Negative calories aren't real.");
    },
  },
});

const Food = mongoose.model("Food", FoodSchema);

module.exports = Food;

```


This code defines your FoodSchema. It will consist of a name value that is of type String, it will be required, trim any whitespace, and set to lowercase characters. It will also consist of a calories value that is of type Number, it will have a default of 0, and validate to ensure no negative numbers are submitted.


# Step 4 — Building the Read Route


Once you have your data model set up, you can start setting up routes to use it. This will utilize various querying functions available through Mongoose.


You will start by reading all foods in the database. At this point, it will be an empty array.


Create a new routes directory:


```
mkdir routes


```


Inside of this new directory, create a new foodRoutes.js file and add the following lines of code:


./routes/foodRoutes.js
```
const express = require("express");
const foodModel = require("../models/food");
const app = express();

app.get("/foods", async (request, response) => {
  const foods = await foodModel.find({});

  try {
    response.send(foods);
  } catch (error) {
    response.status(500).send(error);
  }
});

module.exports = app;

```


This code establishes a /foods endpoint for GET requests (note the plural ‘s’). The Mongoose query function find() returns all objects with matching parameters. Since no parameters have been provided, it will return all of the items in the database.


Since Mongoose functions are asynchronous, you will be using async/await. Once you have the data this code uses a try/catch block to send it. This will be useful to verify the data with Postman.


Navigate to the root of your project directory and run your Express server with the following command in your terminal:


```
node server.js


```


In Postman, create a new Read All Food request. Ensure the request type is set to GET. Set the request URL to localhost:3000/foods. And click Send.



Note: If you need assistance navigating the Postman interface for requests, consult the official documentation.

The Postman results will display an empty array.


# Step 5 — Building the Create Route


Next, you will build the functionality to create a new food item and save it to the database.


Revisit the foodRoutes.js file and add the following lines of code between app.get and module.exports:


./routes/foodRoutes.js
```
// ...

app.post("/food", async (request, response) => {
  const food = new foodModel(request.body);

  try {
    await food.save();
    response.send(food);
  } catch (error) {
    response.status(500).send(error);
  }
});

// ...

```


This code establishes a /food endpoint for POST requests. The Mongoose query function .save() is used to save data passed to it to the database.


In Postman, create a new request called Create New Food. Ensure the request type is set to POST. Set the request URL to localhost:3000/food.


In the Body section, select raw and JSON. Then, add a new food item by constructing a JSON object with a name and calories:


```
{
  "name": "cotton candy",
  "calories": 100
}

```


After sending the Create New Food request, send the Read All Food request again. The Postman results will display the newly added object.


# Step 6 — Building the Update Route


Every object created with Mongoose is given its own _id and you can use this to target specific items. It will be a mix of alphabetical characters and letters. For example: 5d1f6c3e4b0b88fb1d257237.


Next, you will build the functionality to update an existing food item and save the changes to the database.


Revisit the foodRoutes.js file and add the following lines of code between app.post and module.exports:


./routes/foodRoutes.js
```
// ...

app.patch("/food/:id", async (request, response) => {
  try {
    await foodModel.findByIdAndUpdate(request.params.id, request.body);
    await foodModel.save();
    response.send(food);
  } catch (error) {
    response.status(500).send(error);
  }
});

// ...

```


This code establishes a /food/:id endpoint for PATCH requests. The Mongoose query function .findByIdAndUpdate() takes the target’s id and the request data you want to replace it with. Then, .save() is used to save the changes.


In Postman, create a new request called Update Food. Ensure the request type is set to PATCH. Set the request URL to localhost:3000/food/<id>, where id is the identifying string of the food you created previously.


In the Body section, select raw and JSON. Then, modify your food item by constructing a JSON object with a name and calories:


```
{
  "calories": "999"
}

```


After sending the Update Food request, send the Read All Food request again. The Postman results will display the object with modified calories.


# Step 7 — Building the Delete Route


Finally, you will build the functionality to remove an existing food item and save the changes to the database.


Revisit the foodRoutes.js file and add the following lines of code between app.patch and module.exports:


./routes/foodRoutes.js
```
// ...

app.delete("/food/:id", async (request, response) => {
  try {
    const food = await foodModel.findByIdAndDelete(request.params.id);

    if (!food) response.status(404).send("No item found");
    response.status(200).send();
  } catch (error) {
    response.status(500).send(error);
  }
});

// ...

```


This code establishes a /food/:id endpoint for DELETE requests. The Mongoose query function .findByIdAndDelete() takes the target’s id and removes it.


In Postman, create a new request called Delete Food. Ensure the request type is set to DELETE. Set the request URL to localhost:3000/food/<id>, where id is the identifying string of the food you created previously.


After sending the Delete Food request, send the Read All Food request again. The Postman results will display an array without the item that was deleted.



Note: Now that you have completed this tutorial, you may wish to Terminate any MongoDB Atlas clusters that you are no longer using.

At this point, you have an Express server using Mongoose methods to interact with a MongoDB Atlas cluster.


# Conclusion


In this article, you learned how to use Mongoose methods you can quickly make and manage your backend data.


If you’d like to learn more about Node.js, check out our Node.js topic page for exercises and programming projects.


If you’d like to learn more about MongoDB, check out our MongoDB topic page for exercises and programming projects.


