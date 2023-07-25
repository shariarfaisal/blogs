# Using TypeScript with React

```React```

## Introduction


TypeScript is awesome. So is React. Let’s use them both together! Using TypeScript allows us to get the benefits of IntelliSense, as well as the ability to further reason about our code. As well as this, adopting TypeScript is low-friction, as files can be incrementally upgraded without causing issues throughout the rest of your project.


# Prerequisites


If you would like to follow along with this guide, you will need:


- One Ubuntu 20.04 server set up with a non-root user with sudo privileges and firewall enabled. You can do this by following the Ubuntu 20.04 initial server setup guide.
- Node.js and npm installed. You can do this with Apt using a NodeSource PPA. Follow Option 2 of our guide on How To Install Node.js on Ubuntu 20.04 to get started.

# Setting Up Your Project:


Assuming you’ve followed the prerequisite guide and installed Node.js and npm, begin by creating a new directory. Here we’ll call it react-typescript:


```
mkdir react-typescript


```


Change to this directory within the terminal:


```
cd react-typescript/


```


Then initialize a new npm project with defaults:


```
npm init -y


```


```
OutputWrote to /home/sammy/react-typescript/package.json:

{
  "name": "react-typescript",
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


After this is initialized, next, you’ll install the appropriate React dependencies.


# Installing React Dependencies


First, install the React dependencies react and react-dom:


```
sudo npm install react react-dom


```


Next, create a folder called src:


```
mkdir src


```


Change into that src directory:


```
cd src/


```


Then create the index.html file to insert into the src folder. You can do this with your preferred text editor. Here we’ll use nano:


```
nano index.html


```


Now that the index.html file is open, you can make some edits and add the following contents:


index.html
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>React + TypeScript</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <div id="main"></div>
  <script type="module" src="./App.tsx"></script>
</body>
</html>

```


When you’re done, save and close the file. If you’re using nano, you can do this by pressing CTRL + X, then Y and ENTER.


We’ll be using Parcel as our bundler, but you can elect to use webpack or another bundler if you wish. Or, check this section if you prefer using Create React App. Let’s first install Parcel to add to our project:


```
sudo npm install parcel


```


Next, install Typescript:


```
sudo npm install --location=global typescript


```


Then install types for React and ReactDOM:


```
sudo npm install @types/react @types/react-dom


```


After everything has been installed, return to the parent directory by using two periods:


```
cd ..


```


# Starting and Running the Development Server


Next, update package.json with a new task that will start your development server. You can do this by first creating and opening the file with your preferred text editor:


```
nano package.json


```


Once you’ve opened the file, replace the existing content with the following:


package.json
```
{
  "name": "react-typescript",
  "version": "1.0.0",
  "description": "An example of how to use React and TypeScript with Parcel",
  "scripts": {
    "dev": "parcel src/index.html"
  },
  "keywords": [],
  "author": "Paul Halliday",
  "license": "MIT"
}

```


When you’re done, save and close the file.


Now, change back into the src directory:


```
cd src/


```


Once you’re in the directory, create and open a Counter.tsx file that includes a basic counter:


```
nano Counter.tsx


```


Add the following contents to the file:


Counter.tsx
```
import * as React from 'react';

export default class Counter extends React.Component {
  state = {
    count: 0
  };

  increment = () => {
    this.setState({
      count: (this.state.count + 1)
    });
  };

  decrement = () => {
    this.setState({
      count: (this.state.count - 1)
    });
  };

  render () {
    return (
      <div>
        <h1>{this.state.count}</h1>
        <button onClick={this.increment}>Increment</button>
        <button onClick={this.decrement}>Decrement</button>
      </div>
    );
  }
}

```


When you’re done save and close the file.


Then create the App.tsx file with your preferred text editor:


```
nano App.tsx


```


Now that the file is open, add the following information to load the Counter:


App.tsx
```
import * as React from 'react';
import { render } from 'react-dom';

import Counter from './Counter';

render(<Counter />, document.getElementById('main'));

```


Once you’ve added the content, save and close the file.


Then update your firewall permissions to allow for port traffic for 1234:


```
sudo ufw allow 1234


```


Finally, run your project with the following command:


```
sudo npm run dev


```


```
Output
> react-typescript@1.0.0 dev
> parcel src/index.html

Server running at http://localhost:1234
✨ Built in 3.84s

```


Confirm everything is working by navigating to http://your_domain_or_IP_server:1234 in your browser. You should have the following increment and decrement functions on your page:


webpage with increment and decrement functions
Your sample project is now successfully up and running.


# Create React App and TypeScript


If you’d rather use Create React App to initiate your project, you’ll be pleased to know that CRA now supports TypeScript out of the box.


Use the --typescript flag when invoking the create-react-app command:


```

```


# Functional Components


Stateless or functional components can be defined in TypeScript as:


```
import * as React from 'react';

const Count: React.FunctionComponent<{
  count: number;
}> = (props) => {
  return <h1>{props.count}</h1>;
};

export default Count;

```


We’re using React.FunctionComponent and defining the object structure of our expected props. In this scenario, we’re expecting to be passed in a single prop named count and we’re defining it in-line. We can also define this in other ways, by creating an interface such as Props:


```
interface Props {
  count: number;
}

const Count: React.FunctionComponent<Props> = (props) => {
  return <h1>{props.count}</h1>;
};

```


# Class Components


Class components can similarly be defined in TypeScript as in the following:


```
import * as React from 'react';

import Count from './Count';

interface Props {}

interface State {
  count: number;
};

export default class Counter extends React.Component<Props, State> {
  state: State = {
    count: 0
  };

  increment = () => {
    this.setState({
      count: (this.state.count + 1)
    });
  };

  decrement = () => {
    this.setState({
      count: (this.state.count - 1)
    });
  };

  render () {
    return (
      <div>
        <Count count={this.state.count} />
        <button onClick={this.increment}>Increment</button>
        <button onClick={this.decrement}>Decrement</button>
      </div>
    );
  }
}

```


# Default Props


We can also define defaultProps in scenarios where we may want to set default props. We can update our Count example to reflect the following:


```
import * as React from 'react';

interface Props {
  count?: number;
}

export default class Count extends React.Component<Props> {
  static defaultProps: Props = {
    count: 10
  };

  render () {
    return <h1>{this.props.count}</h1>;
  }
}

```


We’ll need to stop passing this.state.count in to the Counter component too, as this will overwrite our default prop:


```
render () {
  return (
    <div>
      <Count />
      <button onClick={this.increment}>Increment</button>
      <button onClick={this.decrement}>Decrement</button>
    </div>
  )
}

```


You should now have a project that’s set up to use TypeScript and React, as well as the tools to create your own functional and class-based components!


