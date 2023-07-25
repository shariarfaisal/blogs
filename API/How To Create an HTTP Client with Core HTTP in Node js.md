# How To Create an HTTP Client with Core HTTP in Node js

```JavaScript``` ```API``` ```Node.js``` ```Development```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


It’s common for a modern web application to communicate with other servers to accomplish a task. For example, a web app that allows you to purchase a book online may involve communication between a customer orders server, a book inventory server, and a payment server. In this design, the different services communicate via web APIs—standard formats that allow you to programmatically send and receive data. In a Node.js app, you can communicate with web APIs by making HTTP requests.


Node.js comes bundled with an http and an https module. These modules have functions to create an HTTP server so that a Node.js program can respond to HTTP requests. They can also make HTTP requests to other servers. This key functionality equips Node.js programmers to create modern, API-driven web applications with Node.js. As it’s a core module, you do not need to install any libraries to use it.


In this tutorial, you will use the https module to make HTTP requests to JSON Placeholder, a fake REST API for testing purposes. You will begin by making a GET request, the standard HTTP request to receive data. You will then look at ways to customize your request, such as by adding headers. Finally, you will make POST, PUT, and DELETE requests so that you can modify data in an external server.


# Prerequisites


- 
This tutorial requires that you have Node.js installed. Once installed, you will be able to access the https module that’s used throughout the tutorial. This tutorial uses Node.js version 10.19.0. To install Node.js on macOS or Ubuntu 18.04, follow the steps in How To Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04.

- 
The methods used to send HTTP requests have a Stream-based API. In Node.js, streams are instances of event emitters. The way in which you respond to data coming from a stream is the same as the way in which you respond to data from events. If you are curious, you can get more in-depth knowledge of event emitters by reading our Using Event Emitters in Node.js guide.


# Step 1 — Making a GET Request


When you interact with an API, you typically make GET requests to retrieve data from web servers. In this step, you’ll look at two functions to make GET requests in Node.js. Your code will retrieve a JSON array of user profiles from a publicly accessible API.


The https module has two functions to make GET requests—the get() function, which can only make GET requests, and the request() function, which makes other types of requests. You will begin by making a request with the get() function.


## Making Requests with get()


HTTP requests using the get() function have this format:


```
https.get(URL_String, Callback_Function) {
    Action
}

```


The first argument is a string with the endpoint you’re making the request to. The second argument is a callback function, which you use to handle the response.


First, set up your coding environment. In your terminal, create a folder to store all your Node.js modules for this guide:


```
mkdir requests


```


Enter that folder:


```
cd requests


```


Create and open a new file in a text editor. This tutorial will use nano as it’s available in the terminal:


```
nano getRequestWithGet.js


```


To make HTTP requests in Node.js, import the https module by adding the follow line:


requests/getRequestWithGet.js
```
const https = require('https');

```



Note:: Node.js has an http and an https module. They have the same functions and behave in a similar manner, but https makes the requests through the Transport Layer Security (TLS/SSL). As the web servers you are using are available via HTTPS, you will use the https module. If you are making requests to and from URLs that only have HTTP, then you would use the http module.

Now use the http object to make a GET request to the API to retrieve a list of users. You will use JSON Placeholder, a publicly available API for testing. This API does not keep a record of any changes you make in your requests. It simulates a real server, and returns mocked responses as long as you send a valid request.


Write the following highlighted code in your text editor:


requests/getRequestWithGet.js
```
const https = require('https');

let request = https.get('https://jsonplaceholder.typicode.com/users?_limit=2', (res) => { });

```


As mentioned in the function signature, the get() function takes two parameters. The first is the API URL you are making the request to in string format and the second is a callback to handle the HTTP response. To read the data from your response, you have to add some code in the callback.


HTTP responses come with a status code. A status code is a number that indicates how successful the response was. Status codes between 200 and 299 are positive responses, while codes between 400 and 599 are errors. You can learn more about status codes in our How To Troubleshoot Common HTTP Error Codes guide.


For this request, a successful response would have a 200 status code. The first thing you’ll do in your callback will be to verify that the status code is what you expect. Add the following code to the callback function:


requests/getRequestWithGet.js
```
const https = require('https');

let request = https.get('https://jsonplaceholder.typicode.com/users?_limit=2', (res) => {
  if (res.statusCode !== 200) {
    console.error(`Did not get an OK from the server. Code: ${res.statusCode}`);
    res.resume();
    return;
  }
});

```


The response object that’s available in the callback has a statusCode property that stores the status code. If the status code is not 200, you log an error to the console and exit.


Note the line that has res.resume(). You included that line to improve performance. When making HTTP requests, Node.js will consume all the data that’s sent with the request. The res.resume() method tells Node.js to ignore the stream’s data. In turn, Node.js would typically discard the data more quickly than if it left it for garbage collection—a periodic process that frees an application’s memory.


Now that you’ve captured error responses, add code to read the data. Node.js responses stream their data in chunks. The strategy for retrieving data will be to listen for when data comes from the response, collate all the chunks, and then parse the JSON so your application can use it.


Modify the request callback to include this code:


requests/getRequestWithGet.js
```
const https = require('https');

let request = https.get('https://jsonplaceholder.typicode.com/users?_limit=2', (res) => {
  if (res.statusCode !== 200) {
    console.error(`Did not get an OK from the server. Code: ${res.statusCode}`);
    res.resume();
    return;
  }

  let data = '';

  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('close', () => {
    console.log('Retrieved all data');
    console.log(JSON.parse(data));
  });
});

```


You begin by creating a new variable data that’s an empty string. You can store data as an array of numbers representing byte data or a string. This tutorial uses the latter as it’s easier to convert a JSON string to a JavaScript object.


After creating the data variable, you create an event listener. Node.js streams the data of an HTTP response in chunks. Therefore, when the response object emits a data event, you will take the data it received and add it to your data variable.


When all the data from the server is received, Node.js emits a close event. At this point, you parse the JSON string stored in data and log the result to the console.


Your Node.js module can now communicate with the JSON API and log the list of users, which will be a JSON array of three users. However, there’s one small improvement you can make first.


This script will throw an error if you are unable to make a request. You may not be able to make a request if you lose your internet connection, for example. Add the following code to capture errors when you’re unable to send an HTTP request:


requests/getRequestWithGet.js
```
...
  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('close', () => {
    console.log('Retrieved all data');
    console.log(JSON.parse(data));
  });

});

request.on('error', (err) => {
  console.error(`Encountered an error trying to make a request: ${err.message}`);
});

```


When a request is made but cannot be sent, the request object emits an error event. If an error event is emitted but not listened to, the Node.js program crashes. Therefore, to capture errors you add an event listener with the on() function and listen for error events. When you get an error, you log its message.


That’s all the code for this file. Save and exit nano by pressing CTRL+X.


Now execute this program with node:


```
node getRequestWithGet.js


```


Your console will display this response:


```
OutputRetrieved all data
[
  {
    id: 1,
    name: 'Leanne Graham',
    username: 'Bret',
    email: 'Sincere@april.biz',
    address: {
      street: 'Kulas Light',
      suite: 'Apt. 556',
      city: 'Gwenborough',
      zipcode: '92998-3874',
      geo: [Object]
    },
    phone: '1-770-736-8031 x56442',
    website: 'hildegard.org',
    company: {
      name: 'Romaguera-Crona',
      catchPhrase: 'Multi-layered client-server neural-net',
      bs: 'harness real-time e-markets'
    }
  },
  {
    id: 2,
    name: 'Ervin Howell',
    username: 'Antonette',
    email: 'Shanna@melissa.tv',
    address: {
      street: 'Victor Plains',
      suite: 'Suite 879',
      city: 'Wisokyburgh',
      zipcode: '90566-7771',
      geo: [Object]
    },
    phone: '010-692-6593 x09125',
    website: 'anastasia.net',
    company: {
      name: 'Deckow-Crist',
      catchPhrase: 'Proactive didactic contingency',
      bs: 'synergize scalable supply-chains'
    }
  }
]

```


This means you’ve successfully made a GET request with the core Node.js library.


The get() method you used is a convenient method Node.js provides because GET requests are a very common type of request. Node.js provides a request() method to make a request of any type. Next, this tutorial will examine how to make a GET request with request().


## Making Requests with request()


The request() method supports multiple function signatures. You’ll use this one for the subsequent example:


```
https.request(URL_String, Options_Object, Callback_Function) {
    Action
}

```


The first argument is a string with the API endpoint. The second argument is a JavaScript object containing all the options for the request. The last argument is a callback function to handle the response.


Create a new file for a new module called getRequestWithRequest.js:


```
nano getRequestWithRequest.js


```


The code you will write is similar to the getRequestWithGet.js module you wrote earlier. First, import the https module:


requests/getRequestWithRequest.js
```
const https = require('https');

```


Next, create a new JavaScript object that contains a method key:


requests/getRequestWithRequest.js
```
const https = require('https');

const options = {
  method: 'GET'
};

```


The method key in this object will tell the request() function what HTTP method the request is using.


Next, make the request in your code. The following codeblock highlights code that was different from the request made with the get() method. In your editor, enter all of the following lines:


requests/getRequestWithRequest.js
```
...

let request = https.request('https://jsonplaceholder.typicode.com/users?_limit=2', options, (res) => {
  if (res.statusCode !== 200) {
    console.error(`Did not get an OK from the server. Code: ${res.statusCode}`);
    res.resume();
    return;
  }

  let data = '';

  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('close', () => {
    console.log('Retrieved all data');
    console.log(JSON.parse(data));
  });
});

request.end();

request.on('error', (err) => {
  console.error(`Encountered an error trying to make a request: ${err.message}`);
});

```


To make a request using request(), you provide the URL in the first argument, an object with the HTTP options in the second argument, and a callback to handle the response in the third argument.


The options variable you created earlier is the second argument, telling Node.js that this is a GET request. The callback is unchanged from when you first wrote it.


You also call the end() method of the request variable. This is an important method that must be called when using the request() function. It completes the request, allowing it to be sent. If you don’t call it, the program will never complete, as Node.js will think you still have data to add to the request.


Save and exit nano with CTRL+X, or the equivalent with your text editor.


Run this program in your terminal:


```
node getRequestWithRequest.js


```


You will receive this output, which is the same as the first module:


```
OutputRetrieved all data
[
  {
    id: 1,
    name: 'Leanne Graham',
    username: 'Bret',
    email: 'Sincere@april.biz',
    address: {
      street: 'Kulas Light',
      suite: 'Apt. 556',
      city: 'Gwenborough',
      zipcode: '92998-3874',
      geo: [Object]
    },
    phone: '1-770-736-8031 x56442',
    website: 'hildegard.org',
    company: {
      name: 'Romaguera-Crona',
      catchPhrase: 'Multi-layered client-server neural-net',
      bs: 'harness real-time e-markets'
    }
  },
  {
    id: 2,
    name: 'Ervin Howell',
    username: 'Antonette',
    email: 'Shanna@melissa.tv',
    address: {
      street: 'Victor Plains',
      suite: 'Suite 879',
      city: 'Wisokyburgh',
      zipcode: '90566-7771',
      geo: [Object]
    },
    phone: '010-692-6593 x09125',
    website: 'anastasia.net',
    company: {
      name: 'Deckow-Crist',
      catchPhrase: 'Proactive didactic contingency',
      bs: 'synergize scalable supply-chains'
    }
  }
]

```


You have now used the request() method to make a GET request. It’s important to know this function as it allows you to customize your request in ways the get() method cannot, like making requests with other HTTP methods.


Next, you will configure and customize your requests with the request() function.


# Step 2 — Configuring HTTP request() Options


The request() function allows you to send HTTP requests without specifying the URL in the first argument. In this case, the URL would be contained with the options object, and the request() would have this function signature:


```
https.request(Options_Object, Callback_Function) {
    Action
}

```


In this step, you will use this functionality to configure your request() with the options object.


Node.js allows you to enter the URL in the options object you pass to the request. To try this out, reopen the getRequestWithRequest.js file:


```
nano getRequestWithRequest.js


```


Remove the URL from the request() call so that the only arguments are the options variable and the callback function:


requests/getRequestWithRequest.js
```
const https = require('https');

const options = {
  method: 'GET',
};

let request = https.request(options, (res) => {
...

```


Now add the following properties to the options object:


requests/getRequestWithRequest.js
```
const https = require('https');

const options = {
  host: 'jsonplaceholder.typicode.com',
  path: '/users?_limit=2',
  method: 'GET'
};

let request = https.request(options, (res) => {
...

```


Instead of one string URL, you have two properties—host and path. The host is the domain name or IP address of the server you’re accessing. The path is everything that comes after the domain name, including query parameters (values after the question mark).


The options object can hold other useful data that goes into a request. For example, you can provide request headers in the options. Headers typically send metadata about the request.


When developers create APIs, they may choose to support different data formats. One API endpoint may be able to return data in JSON, CSV, or XML. In those APIs, the server may look at the Accept header to determine the correct response type.


The Accept header specifies the type of data the user can handle. While the API being used in these examples only return JSON, you can add the Accept header to your request to explicitly state that you want JSON.


Add the following lines of code to append the Accept header:


requests/getRequestWithRequest.js
```
const https = require('https');

const options = {
  host: 'jsonplaceholder.typicode.com',
  path: '/users?_limit=2',
  method: 'GET',
  headers: {
    'Accept': 'application/json'
  }
};

```


By adding headers, you’ve covered the four most popular options that are sent in Node.js HTTP requests: host, path, method, and headers. Node.js supports many more options; you can read more at the official Node.js docs for more information.


Enter CTRL+X to save your file and exit nano.


Next, run your code once more to make the request by only using options:


```
node getRequestWithRequest.js


```


The results will be the same as your previous runs:


```
OutputRetrieved all data
[
  {
    id: 1,
    name: 'Leanne Graham',
    username: 'Bret',
    email: 'Sincere@april.biz',
    address: {
      street: 'Kulas Light',
      suite: 'Apt. 556',
      city: 'Gwenborough',
      zipcode: '92998-3874',
      geo: [Object]
    },
    phone: '1-770-736-8031 x56442',
    website: 'hildegard.org',
    company: {
      name: 'Romaguera-Crona',
      catchPhrase: 'Multi-layered client-server neural-net',
      bs: 'harness real-time e-markets'
    }
  },
  {
    id: 2,
    name: 'Ervin Howell',
    username: 'Antonette',
    email: 'Shanna@melissa.tv',
    address: {
      street: 'Victor Plains',
      suite: 'Suite 879',
      city: 'Wisokyburgh',
      zipcode: '90566-7771',
      geo: [Object]
    },
    phone: '010-692-6593 x09125',
    website: 'anastasia.net',
    company: {
      name: 'Deckow-Crist',
      catchPhrase: 'Proactive didactic contingency',
      bs: 'synergize scalable supply-chains'
    }
  }
]

```


As APIs can vary from provider to provider, being comfortable with the options object is key to adapting to their differing requirements, with the data types and headers being some of the most common variations.


So far, you have only done GET requests to retrieve data. Next, you will make a POST request with Node.js so you can upload data to a server.


# Step 3 — Making a POST Request


When you upload data to a server or want the server to create data for you, you typically send a POST request. In this section, you’ll create a POST request in Node.js. You will make a request to create a new user in the users API.


Despite being a different method from GET, you will be able to reuse code from the previous requests when writing your POST request. However, you will have to make the following adjustments:


- Change the method in the options object to POST
- Add a header to state you are uploading JSON
- Check the status code to confirm a user was created
- Upload the new user’s data

To make these changes, first create a new file called postRequest.js. Open this file in nano or an alternative text editor:


```
nano postRequest.js


```


Begin by importing the https module and creating an options object:


requests/postRequest.js
```
const https = require('https');

const options = {
  host: 'jsonplaceholder.typicode.com',
  path: '/users',
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json; charset=UTF-8'
  }
};

```


You change the path to match what’s required for POST requests. You also updated the method to POST. Lastly, you added a new header in your options Content-Type. This header tells the server what type of data you are uploading. In this case, you’ll be uploading JSON data with UTF-8 encoding.


Next, make the request with the request() function. This is similar to how you made GET requests, but now you look for a different status code than 200. Add the following lines to the end of your code:


requests/postRequest.js
```
...
const request = https.request(options, (res) => {
  if (res.statusCode !== 201) {
    console.error(`Did not get a Created from the server. Code: ${res.statusCode}`);
    res.resume();
    return;
  }

  let data = '';

  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('close', () => {
    console.log('Added new user');
    console.log(JSON.parse(data));
  });
});

```


The highlighted line of code checks if the status code is 201. The 201 status code is used to indicate that the server created a resource.


This POST request is meant to create a new user. For this API, you need to upload the user details. Create some user data and send that with your POST request:


requests/postRequest.js
```
...

const requestData = {
  name: 'New User',
  username: 'digitalocean',
  email: 'user@digitalocean.com',
  address: {
    street: 'North Pole',
    city: 'Murmansk',
    zipcode: '12345-6789',
  },
  phone: '555-1212',
  website: 'digitalocean.com',
  company: {
    name: 'DigitalOcean',
    catchPhrase: 'Welcome to the developer cloud',
    bs: 'cloud scale security'
  }
};

request.write(JSON.stringify(requestData));

```


You first created the requestData variable, which is a JavaScript object containing user data. Your request does not include an id field, as servers typically generate these while saving the new data.


You next use the request.write() function, which accepts a string or buffer object to send along with the request. As your requestData variable is an object, you used the JSON.stringify function to convert it to a string.


To complete this module, end the request and check for errors:


requests/postRequest.js
```
...

request.end();

request.on('error', (err) => {
  console.error(`Encountered an error trying to make a request: ${err.message}`);
});

```


It’s important that you write data before you use the end() function. The end() function tells Node.js that there’s no more data to be added to the request and sends it.


Save and exit nano by pressing CTRL+X.


Run this program to confirm that a new user was created:


```
node postRequest.js


```


The following output will be displayed:


```
OutputAdded new user
{
  name: 'New User',
  username: 'digitalocean',
  email: 'user@digitalocean.com',
  address: { street: 'North Pole', city: 'Murmansk', zipcode: '12345-6789' },
  phone: '555-1212',
  website: 'digitalocean.com',
  company: {
    name: 'DigitalOcean',
    catchPhrase: 'Welcome to the developer cloud',
    bs: 'cloud scale security'
  },
  id: 11
}

```


The output confirms that the request was successful. The API returned the user data that was uploaded, along with the ID that was assigned to it.


Now that you have learned how to make POST requests, you can upload data to servers in Node.js. Next you will try out PUT requests, a method used to update data in a server.


# Step 4 — Making a PUT Request


Developers make a PUT request to upload data to a server. While this may be similar to POST requests, PUT requests have a different function. PUT requests are idempotent—you can run a PUT request multiple times and it will have the same result.


In practice, the code you write is similar to that of a POST request. You set up your options, make your request, write the data you want to upload, and verify the response.


To try this out, you’re going to create a PUT request that updates the first user’s username.


As the code is similar to the POST request, you’ll use that module as a base for this one. Copy the postRequest.js into a new file, putRequest.js:


```
cp postRequest.js putRequest.js


```


Now open putRequest.js in a text editor:


```
nano putRequest.js


```


Make these highlighted changes so that you send a PUT request to https://jsonplaceholder.typicode.com/users/1:


requests/putRequest.js
```
const https = require('https');

const options = {
  host: 'jsonplaceholder.typicode.com',
  path: '/users/1',
  method: 'PUT',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json; charset=UTF-8'
  }
};

const request = https.request(options, (res) => {
  if (res.statusCode !== 200) {
    console.error(`Did not get an OK from the server. Code: ${res.statusCode}`);
    res.resume();
    return;
  }

  let data = '';

  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('close', () => {
    console.log('Updated data');
    console.log(JSON.parse(data));
  });
});

const requestData = {
  username: 'digitalocean'
};

request.write(JSON.stringify(requestData));

request.end();

request.on('error', (err) => {
  console.error(`Encountered an error trying to make a request: ${err.message}`);
});

```


You first change the path and method properties of the options object. path in this case identifies the user that you are going to update. When you make the request, you check if the response code was 200, meaning that the request was OK. The data you are uploading now only contains the property you are updating.


Save and exit nano with CTRL+X.


Now execute this Node.js program in your terminal:


```
node putRequest.js


```


You will receive this output:


```
OutputUpdated data
{ username: 'digitalocean', id: 1 }

```


You sent a PUT request to update a pre-existing user.


So far you have learned how to retrieve, add, and update data. To give us a full command of managing data via APIs, you’ll next make a DELETE request to remove data from a server.


# Step 5 — Making a DELETE Request


The DELETE request is used to remove data from a server. It can have a request body, but most APIs tend not to require them. This method is used to delete an entire object from the server. In this section, you are going to delete a user using the API.


The code you will write is similar to that of a GET request, so use that module as a base for this one. Copy the getRequestWithRequest.js file into a new deleteRequest.js file:


```
cp getRequestWithRequest.js deleteRequest.js


```


Open deleteRequest.js with nano:


```
nano deleteRequest.js


```


Now modify the code at the highlighted parts, so you can delete the first user in the API:


requests/putRequest.js
```
const https = require('https');

const options = {
  host: 'jsonplaceholder.typicode.com',
  path: '/users/1',
  method: 'DELETE',
  headers: {
    'Accept': 'application/json',
  }
};

const request = https.request(options, (res) => {
  if (res.statusCode !== 200) {
    console.error(`Did not get an OK from the server. Code: ${res.statusCode}`);
    res.resume();
    return;
  }

  let data = '';

  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('close', () => {
    console.log('Deleted user');
    console.log(JSON.parse(data));
  });
});

request.end();

request.on('error', (err) => {
  console.error(`Encountered an error trying to make a request: ${err.message}`);
});

```


For this module, you begin by changing the path property of the options object to the resource you want to delete—the first user. You then change the method to DELETE.


Save and exit this file by pressing CTRL+X.


Run this module to confirm it works. Enter the following command in your terminal:


```
node deleteRequest.js


```


The program will output this:


```
OutputDeleted user
{}

```


While the API does not return a response body, you still got a 200 response so the request was OK.


You’ve now learned how to make DELETE requests with Node.js core modules.


# Conclusion


In this tutorial, you made GET, POST, PUT, and DELETE requests in Node.js. No libraries were installed; these requests were made using the standard https module. While GET requests can be made with a get() function, all other HTTP methods are done via the request() method.


The code you wrote was written for a publicly available, test API. However, the way you write requests will work for all types of APIs. If you would like to learn more about APIs, check out our API topic page. For more on developing in Node.js, return to the How To Code in Node.js series.


