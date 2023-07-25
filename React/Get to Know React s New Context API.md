# Get to Know React s New Context API

```JavaScript``` ```React```

## Introduction


In a world where there are lots of different front-end frameworks, it’s always hard to know which one to pick. Do I want to use the ever popular Angular? Or would diving into VueJS be beneficial to my scope of knowledge?


Then we have ReactJS, a framework created by Facebook that seems to be taking the front-end framework world by storm. Using components, a virtual DOM, and JSX (that’s for a different day!), React seems to cover it all, making it a powerful framework.


The new Context API was recently introduced in React 16.3 as:



A way to pass data through the component tree without having to pass props down manually at every level

Sounds great! Let’s dig in.





# Props and State


In React, you have props and state. Two very important things to understand.


Props, short for properties, is the data that is getting passed to the component from a parent component.


State is data that is being managed within the component. So if each component has it’s own state, how do we share that information to another component? You could use props, but props can only be passed between parent and child components.



So what are we to do if we have many layers of components to pass just one bit of information? Also known as, prop-drilling?

# Prop Drilling (What the Context API Solves)


Let’s take a look at an example of prop-drilling so we can understand what the Context API is solving. In this example, we will see how we can pass information from one component, to the child component, then to that component’s child.


```
const Lowest = (props) => (
  <div className="lowest">{props.name}</div>
)

const Middle = (props) => (
  <div className="middle">
    <Lowest name={props.name} />
  </div>
)
 
class Highest extends Component {
  state = {
    name: "Context API"
  }
  
  render() {
    return  <div className="highest">
      {this.state.name}
      <Middle name={this.state.name} />
    </div>
  }
}

```


I know the naming isn’t the most real-world, but it helps to demonstrate context’s ability to pass down to nested components. A more real-world scenario is one that may happen within a card-based user interface: CardGrid -> CardContent -> CardFooter -> LikeButton.


Back to our example, this is how those Highest -> Middle -> Lowest components would get nested:


```
<Highest>

	<Middle>
  
		<Lowest>
    
			{/* we want our content here but dont want to prop pass ALLLL the way down ! */}
      
		</Lowest>
    
	</Middle>
  
</Highest>

```


Notice how in order for the Highest and the Lowest to talk, they need the Middle to be the messenger?


Well lo and behold, we have React Context that can take care of all the work for us.


# React’s Context API


React Context allows us to have a state that can be seen globally to the entire application.


We have to start with the context provider (<Provider />) to define the data you want to be sending around and you need the context consumer (<Consumer />) that grabs that data and uses it where called.



With Context, you now have the ability to declare the state once, and then use that data, via the context consumer, in every part of the application.

Sounds incredible, right? Well let’s look at how we could set that up in a simple React application.


Let’s build!


# Building a Name Transfer with Context API


Today we are going to be setting up a basic React app. Let’s do an app that passes a name from one component to another component that just so happens to not be the child component! Great! We will have three different levels, one will be the highest component that has the name stored in state, we will have the middle component, and then we’ll have the lowest component.


Our application will send the name in state from the highest to the lowest without having to talk to the middle. Open up whichever code editor you like to use and let’s start coding!


First, we will need the react dependency for our app. Go ahead and add that to your dependencies or if you are working in a text editor, do the following steps to install it:


1. Install npm globally on your machine if you do not already have it.
2. npm install —save react
3. Check your package.json for the react dependency.

In our main .js file, this is where the magic happens. Whenever we are building a React app, you always need to import your dependencies, otherwise that file won’t know to use it. So at the top of the index.js file, let’s import what we need:


```
import React, { Component } from 'react';

```


We have our import, now let’s move on to the component. We will want to declare our context in a variable for readability. Under our import let’s do:


```
const AppContext = React.createContext()

```


# Our Layers of Components


Our Highest component will have the state. Our state will have a name that we will want to pass to the Lowest component without having to talk to the Middle component:


```
class Highest extends Component {
	state = {
		name : “React Context API”,
	}

	render() {
		return ( 
		<AppContext.Provider value={this.state}>
		  {this.props.children}
		</AppContext.Provider>
		)
	}
}

```


We will build our child component to that calling it the Middle component:


```
const Middle = () => (
  <div>
    <p>I’m the middle component</p>
    <Lowest />
  </div>
)

```


And the child component to that will be called Lowest:


```
const Lowest = () => (
  <div>
     <AppContext.Consumer>
        {(context) => context.name}
      </AppContext.Consumer>
  </div>
)

```


Let’s go over this. You will see that we have a state in Highest that we will want to pass to Lowest. We have our static property that will allow us to declare what we want our context to be. In our case, the name “React Context API”.


The Provider is holding on to that data so that when it’s consumed by another component, it knows what to give it.  In our Lowest component you will see that we have the Consumer wanting that data without having to first pass it to the Middle component. That component instead just hangs out, declaring that Lowest is it’s child.


# When to Not Use Context


For a simple prop drilling solution that can scale decently, give context a go! For larger scale apps that have multiple (and more complex) states, reducers, etc, Redux may be a better fit.


There is no need to use context all over your application making things a little too messy. Be resourceful with your code, do not just use context to skip extra typing.


# Conclusion


The React Context API is pretty awesome. But don’t use it unless you know it will be beneficial to you and your code. Redux might be just fine. Stay away from prop drilling and know that something like context can help you avoid that. It’s a great alternative!


If you want to check out all the code, you can get it all on codesandbox.


