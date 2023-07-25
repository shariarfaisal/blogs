# How to Enable Server-Side Rendering for a React App

```React```


Warning: This tutorial is intended to be a brief introduction to ReactDOM.hydrate() and ReactDOMServer.rendertoString(). It is not intended for production use.
Alternatively, Next.js offers a modern approach to creating static and server-rendered applications built with React.

## Introduction


Server-side rendering (SSR) is a popular technique for rendering a client-side single page application (SPA) on the server and then sending a fully rendered page to the client. This allows for dynamic components to be served as static HTML markup.


This approach can be useful for search engine optimization (SEO) when indexing does not handle JavaScript properly. It may also be beneficial in situations where downloading a large JavaScript bundle is impaired by a slow network.


In this tutorial, you will initialize a React app using Create React App and then modify the project to enable server-side rendering.


At the end of this tutorial, you will have a working project with a client-side React application and a server-side Express application.


# Prerequisites


To complete this tutorial, you will need:


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.

This tutorial was verified with Node v16.13.1, npm v8.1.2, react v17.0.2, @babel/core v7.16.0, webpack v4.44.2, express v4.17.1, nodemon v2.0.15, and npm-run-all v4.1.5.


# Step 1 — Creating the React App and Modifying the App Component


First, use npx to start up a new React app using the latest version of Create React App.


Let’s call the app, react-ssr-example:


```
npx create-react-app react-ssr-example


```


Then, cd into the new directory:


```
cd react-ssr-example

```


Finally, start the new client-side app in order to verify the installation:


```
npm start


```


You will observe an example React app displayed in your browser window.


Now, in the src directory, let’s create a new <Home> component:


```
nano src/Home.js


```


Next, add the following code to the Home.js file:


src/Home.js
```
function Home(props) {
  return <h1>Hello {props.name}!</h1>;
};

export default Home;

```


This will create a <h1> heading with a "Hello" message directed to a name.


Next, let’s render the <Home> in the <App> component. Open the App.js file in the src directory:


```
nano src/App.js


```


Then, replace the existing lines of code with these new lines of code:


src/App.js
```
import Home from './Home';

function App() {
  return <Home name="Sammy"/>;
}

export default App;

```


This passes along a name to <Home> component so the message that you expect to display is:


```
Output"Hello Sammy!"

```


In the app’s index.js file, you will use ReactDOM’s hydrate method instead of render to indicate to the DOM renderer that you intend to rehydrate the app after a server-side render.


Let’s open the index.js file in the src directory:


```
nano src/index.js


```


Then, replace the contents of the index.js file with the following code:


src/index.js
```
import React from 'react';
import ReactDOM from 'react-dom';

import App from './App';

ReactDOM.hydrate(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

```


That concludes setting up the client-side, you can move on to setting up the server-side.


# Step 2 — Creating an Express Server and Rendering the App Component


Now that you have the app in place, let’s set up a server that will send along a rendered version. You will use Express for the server.



Note: Create React App’s react-scripts installs a version of express as requirement for webpack-dev-server. To avoid dependency tree conflicts, this tutorial no longer includes this installation step.

Next, create a new server directory in the project’s root directory:


```
mkdir server


```


Then, inside of the server directory, create a new index.js file that will contain the Express server code:


```
nano server/index.js


```


Add the imports that will need and define some constants:


server/index.js
```
import path from 'path';
import fs from 'fs';

import React from 'react';
import ReactDOMServer from 'react-dom/server';
import express from 'express';

import App from '../src/App';

const PORT = process.env.PORT || 3006;
const app = express();

```


Next, add the server code with some error handling:


server/index.js
```
// ...

app.get('/', (req, res) => {
  const app = ReactDOMServer.renderToString(<App />);
  const indexFile = path.resolve('./build/index.html');

  fs.readFile(indexFile, 'utf8', (err, data) => {
    if (err) {
      console.error('Something went wrong:', err);
      return res.status(500).send('Oops, better luck next time!');
    }

    return res.send(
      data.replace('<div id="root"></div>', `<div id="root">${app}</div>`)
    );
  });
});

app.use(express.static('./build'));

app.listen(PORT, () => {
  console.log(`Server is listening on port ${PORT}`);
});

```


It is possible to import the <App> component from the client app directly from the server.


Three important things are taking place here:


- Express is used to serve contents from the build directory as static files.
- ReactDOMServer’s renderToString is used to render the app to a static HTML string.
- The static index.html file from the built client app is read. The app’s static content is injected into the <div> with an id of "root". This is sent as a response to the request.

# Step 3 — Configuring webpack, Babel, and npm Scripts


For the server code to work, you will need to bundle and transpile it, using webpack and Babel. To accomplish this.



Note: An earlier version of this tutorial installed babel-core, babel-preset-env, and babel-preset-react-app. These packages have since been archived, and the mono repo versions were utilized instead.
Create React App’s react-scripts handles the installation of the following packages: webpack, webpack-cli, webpack-node-externals, @babel/core, babel-loader, @babel/preset-env, @babel/preset-react. To avoid dependency tree conflicts, this tutorial no longer includes this installation step.

Next, create a new Babel configuration file in the project’s root directory:


```
nano .babelrc.json


```


Then, add the env and react-app presets:


.babelrc.json
```
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react"
  ]
}

```



Note: An earlier version of this tutorial used a .babelrc file (without the .json file extension). This was a configuration file for Babel 6, but this is no longer the case for Babel 7.

Now, create a webpack config for the server that uses Babel Loader to transpile the code. Start by creating the webpack.server.js file in the project’s root directory:


```
nano webpack.server.js


```


Then, add the following configurations to the webpack.server.js file:


webpack.server.js
```
const path = require('path');
const nodeExternals = require('webpack-node-externals');

module.exports = {
  entry: './server/index.js',
  target: 'node',
  externals: [nodeExternals()],
  output: {
    path: path.resolve('server-build'),
    filename: 'index.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader'
      }
    ]
  }
};

```


With this configuration, the transpiled server bundle will be output to the server-build folder in a file called index.js.


Note the use of target: 'node' and externals: [nodeExternals()] from webpack-node-externals, which will omit the files from node_modules in the bundle; the server can access these files directly.


This completes the dependency installation and webpack and Babel configuration.


Now, revisit package.json and add helper npm scripts:


```
nano package.json


```


Add dev:build-server, dev:start, and dev scripts to the package.json file to build and serve the SSR application:


package.json
```
"scripts": {
  "dev:build-server": "NODE_ENV=development webpack --config webpack.server.js --mode=development -w",
  "dev:start": "nodemon ./server-build/index.js",
  "dev": "npm-run-all --parallel build dev:*",
  // ...
},

```


The dev:build-server script sets the environment to "development" and invokes webpack with the configuration file you created earlier. The dev:start script invokes nodemon to serve the built output.


The dev script invokes npm-run-all to run in parallel the build script and all scripts that start with dev:* - including dev:build-server and dev:start.



Note: You do not need to modify the existing "start", "build", "test", and "eject" scripts in the package.json file.

nodemon is used to restart the server when changes are made. npm-run-all is used to run multiple commands in parallel.


Let’s install those packages now by entering the following commands in your terminal window:


```
npm install nodemon@2.0.15 --save-dev
npm install npm-run-all@4.1.5 --save-dev


```


With this in place, you can run the following to build the client-side app, bundle and transpile the server code, and start up the server on :3006:


```
npm run dev


```


The server webpack config will now watch for changes and the server will restart on changes. For the client app, however, will require to be manually rebuilt each time a change is made.



Note: There’s an open issue for that here.

Now, open http://localhost:3006/ in your web browser and observe your server-side rendered app.


Previously, viewing the source code revealed:


```
Output<div id="root"></div>

```


But now, with the changes you made, the source code reveals:


```
Output<div id="root"><h1 data-reactroot="">Hello <!-- -->Sammy<!-- -->!</h1></div>

```


The server-side rendering successfully converted the <App> component into HTML.


# Conclusion


In this tutorial, you initialized a React application and enabled server-side rendering.


With this post, we just scratched the surface at what’s possible. Things tend to get a bit more complicated once routing, data fetching, or Redux also need to be part of a server-side rendered app.


One major benefit of using SSR is having an app that can be crawled for its content, even for crawlers that don’t execute JavaScript code. This can help with search engine optimization (SEO) and providing metadata to social media channels.


SSR can also often help with performance because a fully loaded app is sent down from the server on the first request. For non-trivial apps, your mileage may vary because SSR requires a setup that can get a bit complicated, and it creates a bigger load on the server. Whether to use server-side rendering for your React app depends on your specific needs and on which trade-offs make the most sense for your use case.


If you’d like to learn more about React, take a look at our How To Code in React.js series, or check out our React topic page for exercises and programming projects.


