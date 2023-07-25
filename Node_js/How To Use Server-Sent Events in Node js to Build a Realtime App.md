# How To Use Server-Sent Events in Node js to Build a Realtime App

```Node.js```

## Introduction


Server-Sent Events (SSE) is a technology based on HTTP. On the client-side, it provides an API called EventSource (part of the HTML5 standard) that allows us to connect to the server and receive updates from it.


Before making the decision to use server-sent events, we must take into account two very important aspects:


- It only allows data reception from the server (unidirectional)
- Events are limited to UTF-8 (no binary data)

If your project only receives something like stock prices or text information about something in progress it is a candidate for using Server-Sent Events instead of an alternative like WebSockets.


In this article, you will build a complete solution for both the backend and frontend to handle real-time information flowing from server to client. The server will be in charge of dispatching new updates to all connected clients and the web app will connect to the server, receive these updates and display them.


# Prerequisites


To follow through this tutorial, you’ll need:


- A local development environment for Node.js. Follow How to Install Node.js and Create a Local Development Environment.
- Familiarity with Express.
- Familiarity with React (and hooks).
- cURL is used to verify the endpoints. This may already be available in your environment or you may need to install it. Some familiarity with using command-line tools and options will also be helpful.

This tutorial was verified with cURL v7.64.1, Node v15.3.0, npm v7.4.0, express v4.17.1, body-parser v1.19.0, cors v2.8.5, and react v17.0.1.


# Step 1 – Building the SSE Express Backend


In this section, you will create a new project directory. Inside of the project directory will be a subdirectory for the server. Later, you will also create a subdirectory for the client.


First, open your terminal and create a new project directory:


```
mkdir node-sse-example


```


Navigate to the newly created project directory:


```
cd node-sse-example


```


Next, create a new server directory:


```
mkdir sse-server


```


Navigate to the newly created server directory:


```
cd sse-server


```


Initialize a new npm project:


```
npm init -y


```


Install express, body-parser, and cors:


```
npm install express@4.17.1 body-parser@1.19.0 cors@2.8.5 --save


```


This completes setting up dependencies for the backend.


In this section, you will develop the backend of the application. It will need to support these features:


- Keeping track of open connections and broadcast changes when new facts are added
- GET /events endpoint to register for updates
- POST /facts endpoint for new facts
- GET /status endpoint to know how many clients have connected
- cors middleware to allow connections from the frontend app

Use the first terminal session that is in the sse-server directory. Create a new server.js file:


Open the server.js file in your code editor. Require the needed modules and initialize Express app:


sse-server/server.js
```
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');

const app = express();

app.use(cors());
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: false}));

app.get('/status', (request, response) => response.json({clients: clients.length}));

const PORT = 3000;

let clients = [];
let facts = [];

app.listen(PORT, () => {
  console.log(`Facts Events service listening at http://localhost:${PORT}`)
})

```


Then, build the middleware for GET requests to the /events endpoint. Add the following lines of the code to server.js:


sse-server/server.js
```
// ...

function eventsHandler(request, response, next) {
  const headers = {
    'Content-Type': 'text/event-stream',
    'Connection': 'keep-alive',
    'Cache-Control': 'no-cache'
  };
  response.writeHead(200, headers);

  const data = `data: ${JSON.stringify(facts)}\n\n`;

  response.write(data);

  const clientId = Date.now();

  const newClient = {
    id: clientId,
    response
  };

  clients.push(newClient);

  request.on('close', () => {
    console.log(`${clientId} Connection closed`);
    clients = clients.filter(client => client.id !== clientId);
  });
}

app.get('/events', eventsHandler);

```


The eventsHandler middleware receives the request and response objects that Express provides.


Headers are required to keep the connection open. The Content-Type header is set to 'text/event-stream' and the Connection header is set to 'keep-alive'. The Cache-Control header is optional, set to 'no-cache'. Additionally, the HTTP Status is set to 200 - the status code for a successful request.


After a client opens a connection, the facts are turned into a string. Because this is a text-based transport you must stringify the array, also to fulfill the standard the message needs a specific format. This code declares a field called data and sets to it the stringified array. The last detail of note is the double trailing newline \n\n is mandatory to indicate the end of an event.


A clientId is generated based on the timestamp and the response Express object. These are saved to the clients array. When a client closes a connection, the array of clients is updated to filter out that client.


Then, build the middleware for POST requests to the /fact endpoint. Add the following lines of the code to server.js:


sse-server/server.js
```
// ...

function sendEventsToAll(newFact) {
  clients.forEach(client => client.response.write(`data: ${JSON.stringify(newFact)}\n\n`))
}

async function addFact(request, respsonse, next) {
  const newFact = request.body;
  facts.push(newFact);
  respsonse.json(newFact)
  return sendEventsToAll(newFact);
}

app.post('/fact', addFact);

```


The main goal of the server is to keep all clients connected and informed when new facts are added. The addNest middleware saves the fact, returns it to the client which made POST request, and invokes the sendEventsToAll function.


sendEventsToAll iterates the clients array and uses the write method of each Express response object to send the update.


# Step 2 – Testing the Backend


Before the web app implementation, you can test your server using cURL:


In a terminal window, navigate to the sse-server directory in your project directory. And run the following command:


```
node server.js


```


It will display the following message:


```
OutputFacts Events service listening at http://localhost:3001


```


In a second terminal window, open a connection waiting for updates with the following command:


```
curl -H Accept:text/event-stream http://localhost:3001/events


```


This will generate the following response:


```
Outputdata: []


```


An empty array.


In a third terminal window, create a post POST request to add a new fact with the following command:


```
curl -X POST \
 -H "Content-Type: application/json" \
 -d '{"info": "Shark teeth are embedded in the gums rather than directly affixed to the jaw, and are constantly replaced throughout life.", "source": "https://en.wikipedia.org/wiki/Shark"}'\
 -s http://localhost:3001/fact


```


After the POST request, the second terminal window should update with the new fact:


```
Outputdata: {"info": "Shark teeth are embedded in the gums rather than directly affixed to the jaw, and are constantly replaced throughout life.", "source": "https://en.wikipedia.org/wiki/Shark"}


```


Now the facts array is populated with one item if you close the communication on the second tab and open it again:


```
curl -H Accept:text/event-stream http://localhost:3001/events


```


Instead of an empty array, you should now receive a message with this new item:


```
Outputdata: [{"info": "Shark teeth are embedded in the gums rather than directly affixed to the jaw, and are constantly replaced throughout life.", "source": "https://en.wikipedia.org/wiki/Shark"}]


```


At this point, the backend is fully functional. It is now time to implement the EventSource API on the frontend.


# Step 3 – Building the React Web App Frontend


In this part of our project, you will write a React app that uses the EventSource API.


The web app will have the following set of features:


- Open and keep a connection to our previously developed server
- Render a table with the initial data
- Keep the table updated via SSE

Now, open a new terminal window and navigate to the project directory. Use create-react-app to generate a React App.


```
npx create-react-app sse-client


```


Navigate to the newly created client directory:


```
cd sse-client


```


Run the client application:


```
npm start


```


This should open a new browser window with your new React application. This completes setting up dependencies for the frontend.


For styling, open the App.css file in your code editor. And modify the contents with the following lines of code:


sse-client/src/App.css
```
body {
  color: #555;
  font-size: 25px;
  line-height: 1.5;
  margin: 0 auto;
  max-width: 50em;
  padding: 4em 1em;
}

.stats-table {
  border-collapse: collapse;
  text-align: center;
  width: 100%;
}

.stats-table tbody tr:hover {
  background-color: #f5f5f5;
}

```


Then, open the App.js file in your code editor. And modify the contents with the following lines of code:


sse-client/src/App.js
```
import React, { useState, useEffect } from 'react';
import './App.css';

function App() {
  const [ facts, setFacts ] = useState([]);
  const [ listening, setListening ] = useState(false);

  useEffect( () => {
    if (!listening) {
      const events = new EventSource('http://localhost:3001/events');

      events.onmessage = (event) => {
        const parsedData = JSON.parse(event.data);

        setFacts((facts) => facts.concat(parsedData));
      };

      setListening(true);
    }
  }, [listening, facts]);

  return (
    <table className="stats-table">
      <thead>
        <tr>
          <th>Fact</th>
          <th>Source</th>
        </tr>
      </thead>
      <tbody>
        {
          facts.map((fact, i) =>
            <tr key={i}>
              <td>{fact.info}</td>
              <td>{fact.source}</td>
            </tr>
          )
        }
      </tbody>
    </table>
  );
}

export default App;

```


The useEffect function argument contains the important parts: an EventSource object with the /events endpoint and an onmessage method where the data property of the event is parsed.


Unlike the cURL response, you now have the event as an object. You can now take the data property and parse it giving, as a result, a valid JSON object.


Finally, this code pushes the new fact to the list of facts, and the table gets re-rendered.


# Step 4 – Testing the Frontend


Now, try adding a new fact.


In a terminal window, run the following command:


```
curl -X POST \
 -H "Content-Type: application/json" \
 -d '{"info": "Shark teeth are embedded in the gums rather than directly affixed to the jaw, and are constantly replaced throughout life.", "source": "https://en.wikipedia.org/wiki/Shark"}'\
 -s http://localhost:3001/fact


```


The POST request added a new fact and all the connected clients should have received it. If you check the application in the browser you will have a new row with this information.


# Conclusion


This article served as an introduction to server-sent events. In this article, you built a complete solution for both the backend and frontend to handle real-time information flowing from server to client.


SSE were designed for text-based and unidirectional transport. Here’s the current support for EventSource in browsers.


Continue your learning by exploring all of the features available to EventSource like retry.


