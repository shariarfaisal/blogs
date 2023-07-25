# How To Use Joi for Node API Schema Validation

```Node.js```

## Introduction


Imagine you are working on an API endpoint to create a new user. User data—such as firstname, lastname, age, and birthdate—will need to be included in the request. A user mistakenly entering their name in as the value for the age field when you are expecting a numeric value would not be desirable. A user typing out their birthday for the birthdate field when you are expecting a particular date format would also not be desirable. You do not want bad data making its way through your application. You can address this with Data Validation.


If you have ever used an ORM (object-relational mapping) when building your Node application—such as Sequelize, Knex, Mongoose (for MongoDB)—you will know that it is possible to set validation constraints for your model schemas. This makes it easier to handle and validate data at the application level before persisting it to the database. When building APIs, the data usually comes from HTTP requests to certain endpoints, and the need may soon arise to be able to validate data at the request level.


In this tutorial, you will learn how we can use the Joi validation module to validate data at the request level. You can learn more about how to use Joi and the supported schema types by checking out the API Reference.


At the end of this tutorial, you should be able to do the following:


- Create validation schema for the request data parameters
- Handle validation errors and give appropriate feedback
- Create a middleware to intercept and validate requests

# Prerequisites


To complete this tutorial, you’ll need:


- A local development environment for Node.js. Follow How to Install Node.js and Create a Local Development Environment.
- Downloading and installing a tool like Postman is recommended for testing API endpoints.

This tutorial was verified with Node v14.2.0, npm v6.14.5, and joi v13.0.2.


# Step 1 — Setting up the Project


For this tutorial, you will pretend that you are building a school portal and you want to create API endpoints:


- /people: add new students and teachers
- /auth/edit: set login credentials for teachers
- /fees/pay: make fee payments for students

You will create a REST API for this tutorial using Express to test your Joi schemas.


To begin, open your command line terminal and create a new project directory:


```
mkdir joi-schema-validation


```


Then navigate to that directory:


```
cd joi-schema-validation


```


Run the following command to set up a new project:


```
npm init -y


```


And install the required dependencies:


```
npm install body-parser@1.18.2<6> express@4.16.2 joi@13.0.2 lodash@4.17.4 morgan@1.9.0<^>


```


Create a new file named app.js in your project root directory to set up the Express app:


```
nano app.js


```


Here is a starter setup for the application.


First, require express, morgan, and body-parser:


app.js
```
// load app dependencies
const express = require('express');
const logger = require('morgan');
const bodyParser = require('body-parser');

```


Then, initialize the app:


app.js
```
// ...

const app = express();
const port = process.env.NODE_ENV || 3000;

// app configurations
app.set('port', port);

// establish http server connection
app.listen(port, () => { console.log(`App running on port ${port}`) });

```


Next, add morgan logging and body-parser middlewares to the request pipeline of your app:


app.js
```
// ...

const app = express();
const port = process.env.NODE_ENV || 3000;

// app configurations
app.set('port', port);

// load app middlewares
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

// establish http server connection
app.listen(port, () => { console.log(`App running on port ${port}`) });

```


These middlewares fetch and parse the body of the current HTTP request for application/json and application/x-www-form-urlencoded requests, and make them available in the req.body of the request’s route handling middleware.


Then, add Routes:


app.js
```
// ...

const Routes = require('./routes');

const app = express();
const port = process.env.NODE_ENV || 3000;

// app configurations
app.set('port', port);

// load app middlewares
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

// load our API routes
app.use('/', Routes);

// establish http server connection
app.listen(port, () => { console.log(`App running on port ${port}`) });

```


Your app.js file is complete for the moment.


## Handling the Endpoints


From your application setup, you specified that you are fetching your routes from a routes.js file.


Let’s create the file in your project root directory:


```
nano routes.js


```


Require express and handle requests with a response of "success" and the data in the request:


routes.js
```
const express = require('express');
const router = express.Router();

// generic route handler
const genericHandler = (req, res, next) => {
  res.json({
    status: 'success',
    data: req.body
  });
};

module.exports = router;

```


Next, establish endpoints for people, auth/edit, and fees/pay:


routes.js
```
// ...

// create a new teacher or student
router.post('/people', genericHandler);

// change auth credentials for teachers
router.post('/auth/edit', genericHandler);

// accept fee payments for students
router.post('/fees/pay', genericHandler);

module.exports = router;

```


Now, when a POST request hits either of these endpoints, your application will use the genericHandler and send a response.


Finally, add a start script to the scripts section of your package.json file:


```
nano package.json


```


It should look like this:


package.json
```
// ...
"scripts": {
  "start": "node app.js"
},
// ...

```


Run the app to see what you have so far and that everything is working properly:


```
npm start


```


You should see a message like: "App running on port 3000". Make a note of the port number that the service is running on. And leave your application running in the background.


## Testing the Endpoints


You can test the API endpoints using an application such as Postman.



Note: If this is your first time using Postman, here are some steps on how to use it for this tutorial:

Start with creating a new Request.
Set your request type to POST (by default, it may be set to GET).
Fill the Enter request URL field with the server location (in most cases, it should be: localhost:3000) and the endpoint (in this case: /people).
Select Body.
Set your encoding type to Raw (by default, it may be set to none").
Set the format to JSON (by default, it may be set to Text).
Enter your data.

Then click Send to view the response.

Let’s consider a scenario where an administrator is creating a new account for a teacher named “Glad Chinda”.


Provided this example request:


```
{
    "type": "TEACHER",
    "firstname": "Glad",
    "lastname": "Chinda"
}

```


You will receive this example response:


```
Output{
    "status": "success",
    "data": {
        "type": "TEACHER",
        "firstname": "Glad",
        "lastname": "Chinda"
    }
}

```


You will have received a "success" status, and the data you submitted is captured in the response. This verifies that your application is working as expected.


# Step 2 — Experimenting with Joi Validation Rules


A simplified example may help give you an idea of what you will achieve in later steps.


In this example, you will create validation rules using Joi to validate an email, phone number, and birthday for a request to create a new user. If the validation fails, you send back an error. Otherwise, you return the user data.


Let’s add a test endpoint to the app.js file:


```
nano app.js


```


Add the following code snippet:


app.js
```
// ...

app.use('/', Routes);

app.post('/test', (req, res, next) => {
  const Joi = require('joi');

  const data = req.body;

  const schema = Joi.object().keys({
    email: Joi.string().email().required(),
    phone: Joi.string().regex(/^\d{3}-\d{3}-\d{4}$/).required(),
    birthday: Joi.date().max('1-1-2004').iso()
  });
});

// establish http server connection
app.listen(port, () => { console.log(`App running on port ${port}`) });

```


This code adds a new /test endpoint. It defines data from the request body. And it defines schema with Joi rules for email, phone, and birthday.


The constraints for email include:


- it must be a valid email string
- it must be required

The constraints for phone include:


- it must be a string with digits in the format of XXX-XXX-XXXX
- it must be required

The constraints for birthday include:


- it must be a valid date in ISO 8601 format
- it cannot be after Jan 1, 2004
- it is not required

Next, handle passing and failing validation:


app.js
```
// ...

app.use('/', Routes);

app.post('/test', (req, res, next) => {
  // ...

  Joi.validate(data, schema, (err, value) => {
    const id = Math.ceil(Math.random() * 9999999);

    if (err) {
      res.status(422).json({
        status: 'error',
        message: 'Invalid request data',
        data: data
      });
    } else {
      res.json({
        status: 'success',
        message: 'User created successfully',
        data: Object.assign({id}, value)
      });
    }
  });
});

// establish http server connection
app.listen(port, () => { console.log(`App running on port ${port}`) });

```


This code takes the data and validates it against the schema.


If any of the rules for email, phone, or birthday fail, a 422 error is generated with a status of "error" and a message of "Invalid request data".


If all of the rules for email, phone, and birthday pass, a response is generated with a status of "success" and a message of "User created successfully".


Now, you can test the example route.


Start the app again by running the following command from your terminal:


```
npm start


```


You can use Postman to test the example route POST /test.


Configure your request:


```
POST localhost:3000/test
Body
Raw
JSON

```


Add your data to the JSON field:


```
{
    "email": "test@example.com",
    "phone": "555-555-5555",
    "birthday": "2004-01-01"
}

```


You should see something similar to the following response:


```
Output{
    "status": "success",
    "message": "User created successfully",
    "data": {
        "id": 1234567,
        "email": "test@example.com",
        "phone": "555-555-5555",
        "birthday": "2004-01-01T00:00:00.000Z"
    }
}

```


Here is a demo video accomplishing this:



    <a href="https://www.youtube.com/watch?v=necriO0nWXA" target="_blank">View YouTube video</a>

You can specify more validation constraints to the base schema to control the kind of values that are considered valid. Since each constraint returns a schema instance, it is possible to chain several constraints together via method chaining to define more specific validation rules.


It is recommended that you create object schemas using Joi.object() or Joi.object().keys(). When using any of these two methods, you can further control the keys that are allowed in the object using some additional constraints, which will not be possible to do using the object literal method.


Sometimes, you may want a value to be either a string or number or something else. This is where alternative schemas come into play. You can define alternative schemas using Joi.alternatives(). It inherits from the any() schema, so constraints like required() can be used with it.


Refer to the API Reference for detailed documentation of all the constraints available to you.


# Step 3 — Creating the API Schemas


After familiarizing yourself with constraints and schemas in Joi, you can now create the validation schemas for the API routes.


Create a new file named schemas.js in the project route directory:


```
nano schemas.js


```


Start by requiring Joi:


schemas.js
```
// load Joi module
const Joi = require('joi');

```


## people Endpoint and personDataSchema


The /people endpoint will use personDataSchema. In this scenario, an administrator is creating accounts for teachers and students. The API will want an id, type, name, and possibly an age if they are a student.


id: will be a string in UUID v4 format:


```
Joi.string().guid({version: 'uuidv4'})

```


type: will either be a string of STUDENT or TEACHER. Validation will accept any case, but will force uppercase():


```
Joi.string().valid('STUDENT', 'TEACHER').uppercase().required()

```


age: will either be an integer or a string with a value greater than 6. And the string can also contain shortened formats of “year” (like “y”, “yr”, and “yrs”):


```
Joi.alternatives().try([
  Joi.number().integer().greater(6).required(),
  Joi.string().replace(/^([7-9]|[1-9]\d+)(y|yr|yrs)?$/i, '$1').required()
]);

```


firstname, lastname, fullname: will be a string of alphabetic characters. Validation will accept any case, but will force uppercase():


A string of alphabetic characters for firstname and lastname:


```
Joi.string().regex(/^[A-Z]+$/).uppercase()

```


A space separated fullname:


```
Joi.string().regex(/^[A-Z]+ [A-Z]+$/i).uppercase()

```


If fullname is specified, then firstname and lastname must be ommitted. If firstname is specified, then lastname must also be specified. One of either fullname or firstname must be specified:


```
.xor('firstname', 'fullname')
.and('firstname', 'lastname')
.without('fullname', ['firstname', 'lastname'])

```


Putting it all together, peopleDataSchema will resemble this:


schemas.js
```
// ...

const personID = Joi.string().guid({version: 'uuidv4'});

const name = Joi.string().regex(/^[A-Z]+$/).uppercase();

const ageSchema = Joi.alternatives().try([
  Joi.number().integer().greater(6).required(),
  Joi.string().replace(/^([7-9]|[1-9]\d+)(y|yr|yrs)?$/i, '$1').required()
]);

const personDataSchema = Joi.object().keys({
  id: personID.required(),
  firstname: name,
  lastname: name,
  fullname: Joi.string().regex(/^[A-Z]+ [A-Z]+$/i).uppercase(),
  type: Joi.string().valid('STUDENT', 'TEACHER').uppercase().required(),

  age: Joi.when('type', {
    is: 'STUDENT',
    then: ageSchema.required(),
    otherwise: ageSchema
  })
})
.xor('firstname', 'fullname')
.and('firstname', 'lastname')
.without('fullname', ['firstname', 'lastname']);

```


## /auth/edit Endpoint and authDataSchema


The /auth/edit endpoint will use authDataSchema. In this scenario, a teacher is updating their account’s email and password. The API will want an id, email, password, and confirmPassword.


id: will use the validation defined earlier for personDataSchema.


email: will be a valid email address. Validation will accept any case, but will force lowercase().


```
Joi.string().email().lowercase().required()

```


password: will be a string of at least 7 characters:


```
Joi.string().min(7).required().strict()

```


confirmPassword: will be a string that references password to ensure the two match:


```
Joi.string().valid(Joi.ref('password')).required().strict()

```


Putting it all together, authDataSchema will resemble this:


schemas.js
```
// ...

const authDataSchema = Joi.object({
  teacherId: personID.required(),
  email: Joi.string().email().lowercase().required(),
  password: Joi.string().min(7).required().strict(),
  confirmPassword: Joi.string().valid(Joi.ref('password')).required().strict()
});

```


## /fees/pay Endpoint and feesDataSchema


The /fees/pay endpoint will use feesDataSchema. In this scenario, a student is submitting their credit card information to pay an amount of money, and the transaction timestamp is also recorded. The API will want an id, amount, cardNumber, and completedAt.


id: will use the validation defined earlier for personDataSchema.


amount: will either be an integer or a floating point number. The value must be a positive number greater than 1. If a floating point number is given, the precision is truncated to a maximum of 2:


```
Joi.number().positive().greater(1).precision(2).required()

```


cardNumber: will be a string that is a valid Luhn Algorithm compliant number:


```
Joi.string().creditCard().required()

```


completedAt: will be a date timestamp in JavaScript format:


```
Joi.date().timestamp().required()

```


Putting it all together, feesDataSchema will resemble this:


schemas.js
```
// ...

const feesDataSchema = Joi.object({
  studentId: personID.required(),
  amount: Joi.number().positive().greater(1).precision(2).required(),
  cardNumber: Joi.string().creditCard().required(),
  completedAt: Joi.date().timestamp().required()
});

```


Finally, export an object with the endpoints associates with schemas:


schemas.js
```
// ...

// export the schemas
module.exports = {
  '/people': personDataSchema,
  '/auth/edit': authDataSchema,
  '/fees/pay': feesDataSchema
};

```


Now, you have created schemas for the API endpoints and exported them in an object with the endpoints as keys.


# Step 4 — Creating the Schema Validation Middleware


Let’s create a middleware that will intercept every request to your API endpoints and validate the request data before handing control over to the route handler.


Create a new folder named middlewares in the project root directory:


```
mkdir middlewares


```


Then, create a new file named SchemaValidator.js inside it:


```
nano middlewares/SchemaValidator.js


```


The file should contain the following code for the schema validation middleware.


middlewares/SchemaValidator.js
```
const _ = require('lodash');
const Joi = require('joi');
const Schemas = require('../schemas');

module.exports = (useJoiError = false) => {
  // useJoiError determines if we should respond with the base Joi error
  // boolean: defaults to false
  const _useJoiError = _.isBoolean(useJoiError) && useJoiError;

  // enabled HTTP methods for request data validation
  const _supportedMethods = ['post', 'put'];

  // Joi validation options
  const _validationOptions = {
    abortEarly: false,  // abort after the last validation error
    allowUnknown: true, // allow unknown keys that will be ignored
    stripUnknown: true  // remove unknown keys from the validated data
  };

  // return the validation middleware
  return (req, res, next) => {
    const route = req.route.path;
    const method = req.method.toLowerCase();

    if (_.includes(_supportedMethods, method) && _.has(Schemas, route)) {
      // get schema for the current route
      const _schema = _.get(Schemas, route);

      if (_schema) {
        // Validate req.body using the schema and validation options
        return Joi.validate(req.body, _schema, _validationOptions, (err, data) => {
          if (err) {
            // Joi Error
            const JoiError = {
              status: 'failed',
              error: {
                original: err._object,
                // fetch only message and type from each error
                details: _.map(err.details, ({message, type}) => ({
                  message: message.replace(/['"]/g, ''),
                  type
                }))
              }
            };

            // Custom Error
            const CustomError = {
              status: 'failed',
              error: 'Invalid request data. Please review request and try again.'
            };

            // Send back the JSON error response
            res.status(422).json(_useJoiError ? JoiError : CustomError);
          } else {
            // Replace req.body with the data after Joi validation
            req.body = data;
            next();
          }
        });
      }
    }
    next();
  };
};

```


Here, you have loaded Lodash alongside Joi and the schemas into the middleware module. You are also exporting a factory function that accepts one argument and returns the schema validation middleware.


The argument to the factory function is a boolean value, which when true, indicates that Joi validation errors should be used; otherwise a custom generic error is used for errors in the middleware. It defaults to false if not specified or a non-boolean value is given.


You have also defined the middleware to only handle POST and PUT requests. Every other request methods will be skipped by the middleware. You can also configure it if you wish, to add other methods like DELETE that can take a request body.


The middleware uses the schema that matches the current route key from the Schemas object we defined earlier to validate the request data. The validation is done using the Joi.validate() method, with the following signature:


- data: the data to validate which in our case is req.body.
- schema: the schema with which to validate the data.
- options: an object that specifies the validation options. Here are the validation options we used:
- callback: a callback function that will be called after validation. It takes two arguments. The first is the Joi ValidationError object if there were validation errors or null if no errors. The second argument is the output data.

Finally, in the callback function of Joi.validate() you return the formatted error as a JSON response with the 422 HTTP status code if there are errors, or you simply overwrite req.body with the validation output data and then pass control over to the next middleware.


Now, you can use the middleware on your routes:


```
nano routes.js


```


Modify the routes.js file as follows:


routes.js
```
const express = require('express');
const router = express.Router();

const SchemaValidator = require('./middlewares/SchemaValidator');
const validateRequest = SchemaValidator(true);

// generic route handler
const genericHandler = (req, res, next) => {
  res.json({
    status: 'success',
    data: req.body
  });
};

// create a new teacher or student
router.post('/people', validateRequest, genericHandler);

// change auth credentials for teachers
router.post('/auth/edit', validateRequest, genericHandler);

// accept fee payments for students
router.post('/fees/pay', validateRequest, genericHandler);

module.exports = router;

```


Let’s run your app to test your application:


```
npm start


```


These are sample test data you can use to test the endpoints. You can edit them however you wish.



Note: For generating UUID v4 strings, you can use the Node UUID module or an online UUID Generator.

## /people Endpoint


In this scenario, an administrator is entering a new student named John Doe with an age of 12 into the system:


```
{
    "id": "a967f52a-6aa5-401d-b760-35eef7c68b32",
    "type": "Student",
    "firstname": "John",
    "lastname": "Doe",
    "age": "12yrs"
}

```


Example POST /people success response:


```
Output{
    "status": "success",
    "data": {
        "id": "a967f52a-6aa5-401d-b760-35eef7c68b32",
        "type": "STUDENT",
        "firstname": "JOHN",
        "lastname": "DOE",
        "age": "12"
    }
}

```


In this failed scenario, the administrator has not provided a value for the required age field:


```
Output{
    "status": "failed",
    "error": {
        "original": {
            "id": "a967f52a-6aa5-401d-b760-35eef7c68b32",
            "type": "Student",
            "fullname": "John Doe",
        },
        "details": [
            {
                "message": "age is required",
                "type": "any.required"
            }
        ]
    }
}

```


## /auth/edit Endpoint


In this scenario, a teacher is updating their email and password:


```
{
    "teacherId": "e3464323-22c1-4e31-9ac5-9bde207d61d2",
    "email": "teacher@example.com",
    "password": "password",
    "confirmPassword": "password"
}

```


Example POST /auth/edit success response:


```
Output{
    "status": "success",
    "data": {
        "teacherId": "e3464323-22c1-4e31-9ac5-9bde207d61d2",
        "email": "teacher@example.com",
        "password": "password",
        "confirmPassword": "password"
    }
}

```


In this failed scenario, the teacher has provided an invalid email address and incorrect confirmation password:


```
Output{
    "status": "failed",
    "error": {
        "original": {
            "teacherId": "e3464323-22c1-4e31-9ac5-9bde207d61d2",
            "email": "email_address",
            "password": "password",
            "confirmPassword": "Password"
        },
        "details": [
            {
                "message": "email must be a valid email",
                "type": "string.email"
            },
            {
                "message": "confirmPassword must be of [ref:password]",
                "type": "any.allowOnly"
            }
        ]
    }
}

```


## /fees/pay Endpoint


In this scenario, a student is paying a fee with a credit card and recording a timestamp for the transaction:



Note: For test purposes, use 4242424242424242 as a valid credit card number. This number has been designated for testing purposes by services like Stripe.

```
{
    "studentId": "c77b8a6e-9d26-428a-9df1-e852473f886f",
    "amount": 134.9875,
    "cardNumber": "4242424242424242",
    "completedAt": 1512064288409
}

```


Example POST /fees/pay success response:


```
Output{
    "status": "success",
    "data": {
        "studentId": "c77b8a6e-9d26-428a-9df1-e852473f886f",
        "amount": 134.99,
        "cardNumber": "4242424242424242",
        "completedAt": "2017-11-30T17:51:28.409Z"
    }
}

```


In this failed scenario, the student has provided an invalid credit card number:


```
Output{
    "status": "failed",
    "error": {
        "original": {
            "studentId": "c77b8a6e-9d26-428a-9df1-e852473f886f",
            "amount": 134.9875,
            "cardNumber": "5678901234567890",
            "completedAt": 1512064288409
        },
        "details": [
            {
                "message": "cardNumber must be a credit card",
                "type": "string.creditCard"
            }
        ]
    }
}

```


You can complete testing your application with different values to observe successful and failed validation.


# Conclusion


In this tutorial, you have created schemas for validating a collection of data using Joi and handled request data validation using a custom schema validation middleware on your HTTP request pipeline.


Having consistent data ensures that it will behave in a reliable and expected manner when you reference it in your application.


For a complete code sample of this tutorial, check out the joi-schema-validation-sourcecode repository on GitHub.


