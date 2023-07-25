# Connecting Redux to React Using React Redux

```React```

Redux is a separate entity from React and can be used with any JavaScript front-end framework or with vanilla JavaScript. Still though, it’s undeniable that React and Redux are very commonly used together, and for this the React Redux library provides simple bindings that make it really easy to connect the two.


The API for the React Redux bindings is very simple: a Provider component that makes our store accessible throughout our app and a connect function that creates container components that can read the state from the store and dispatch actions.


# Getting Started


Let’s initialize a React project and make sure we have the necessary dependencies. Let’s use Create React App to create a sample bookmark manager app:


```
$ npx create-react-app fancy-bookmarks

```


Note here the use of npx to ensure we’re using the latest version of Create React App.


Now let’s cd into our app’s directory and add the redux and react-redux packages:


```
$ yarn add redux react-redux

# or, using npm:
$ npm install redux react-redux

```


# Redux Setup


Now let’s do the Redux setup for our app. I won’t explain much here because, but if you’re new to Redux in general, have a look at our intro to Redux.


First, some action types:


actions/types.js
```
export const ADD_BOOKMARK = 'ADD_BOOKMARK';
export const DELETE_BOOKMARK = 'DELETE_BOOKMARK';

```


And some action creators to go along with our action types:


actions/index.js
```
import uuidv4 from 'uuid/v4';
import { ADD_BOOKMARK, DELETE_BOOKMARK } from './types';

export const addBookmark = ({ title, url }) => ({
  type: ADD_BOOKMARK,
  payload: {
    id: uuidv4(),
    title,
    url
  }
});

export const deleteBookmark = id => ({
  type: DELETE_BOOKMARK,
  payload: {
    id
  }
});

```



Here you’ll note that I’m also using the uuid library to generate random IDs.


And here’s our the only reducer needed for our simple app:


reducer/index.js
```
import { ADD_BOOKMARK, DELETE_BOOKMARK } from '../actions/types';

export default function bookmarksReducer(state = [], action) {
  switch (action.type) {
    case ADD_BOOKMARK:
      return [...state, action.payload];
    case DELETE_BOOKMARK:
      return state.filter(bookmark => bookmark.id !== action.payload.id);
    default:
      return state;
  }
}

```


As you can see, so far we’re purely in Redux-land and haven’t done anything just yet to make our React app talk to our Redux store seamlessly. This is what we’ll tackle next.


# Provider Component


We’ll use the Provider component from React Redux to wrap our main App component and make the app’s Redux store accessible from any container component (connected component) down the React component tree:


index.js
```
import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import rootReducer from './reducers';

import App from './App';

const store = createStore(rootReducer);

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);

```


If you were using React Router, you’d wrap the Provider component around the BrowserRouter component.


# Container Components & the Connect Function


Now that the Redux store is accessible in our app, we still need to create some container components, also known as connected components that will have access to read from the store or dispatch actions. We’ll create two container components: AddBookmark and BookmarkList and our App component will use them and look like this:


App.js
```
import React, { Component } from 'react';
import AddBookmark from './containers/AddBookmark';
import BookmarksList from './containers/BookmarksList';

class App extends Component {
  render() {
    return (
      <div>
        <AddBookmark />
        <BookmarksList />
      </div>
    );
  }
}

export default App;

```


Now for our first container component, the BookmarksList component:


containers/BookmarksList.js
```
import React from 'react';
import { connect } from 'react-redux';
import Bookmark from '../components/Bookmark';
import { deleteBookmark } from '../actions';

function BookmarksList({ bookmarks, onDelete }) {
  return (
    <div>
      {bookmarks.map(bookmark => {
        return (
          <Bookmark bookmark={bookmark} onDelete={onDelete} key={bookmark.id} />
        );
      })}
    </div>
  );
}

const mapStateToProps = state => {
  return {
    bookmarks: state
  };
};

const mapDispatchToProps = dispatch => {
  return {
    onDelete: id => {
      dispatch(deleteBookmark(id));
    }
  };
};

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(BookmarksList);

```


We use the React Redux’s connect function and pass in a mapStateToProps function and a mapDispatchToProps function. We then call the returned function with the component that we want to connect.


mapStateToProps receives the current value of the state and should return an object that makes pieces of the state available as props to the connected component. Here our state only has bookmarks, but you could imagine a scenario where multiple pieces of state could be mapped different props. For example:


```
const mapStateToProps = state => {
  return {
    users: state.users,
    todos: state.todos,
    // ...
  };
};

```


mapStateToProps receives the store’s dispatch method and should return an object that makes some callbacks available as props that then dispatch the desired actions to the store.



Next, the AddBookmark component:


containers/AddBookmark.js
```
import { connect } from 'react-redux';
import { addBookmark } from '../actions';
import NewBookmark from '../components/NewBookmark';

const mapDispatchToProps = dispatch => {
  return {
    onAddBookmark: bookmark => {
      dispatch(addBookmark(bookmark));
    }
  };
};

export default connect(
  null,
  mapDispatchToProps
)(NewBookmark);

```


This component doesn’t need to read from the store, so we pass-in null as the first argument to the connect function.


You’ll also notice that this second container component component doesn’t render anything of its own, and instead all the UI rendering part is left to the NewBookmark presentational component.


# Presentational Components


Presentational components are much simpler and don’t have access to the store directly. Instead they receive props from container components with values from the state or callbacks that call our action creators. They don’t need to know anything about Redux, and are instead just a function of the props given to them. Presentational components are simple to write, easily reusable and easy to test.


For example, here’s our Bookmark presentational component, which renders one bookmark:


components/Bookmark.js
```
import React from 'react';

const styles = {
  borderBottom: '2px solid #eee',
  background: '#fafafa',
  margin: '.75rem auto',
  padding: '.6rem 1rem',
  maxWidth: '500px',
  borderRadius: '7px'
};

export default ({ bookmark: { title, url, id }, onDelete }) => {
  return (
    <div style={styles}>
      <h2>{title}</h2>
      <p>URL: {url}</p>
      <button type="button" onClick={() => onDelete(id)}>
        Remove
      </button>
    </div>
  );
};

```


As you can see, it receives the bookmark as well as the onDelete callback as props.



In the case of the NewBookmark, it’s not a purely presentational component, but more of an hybrid component because it holds some local state for the input values. Still though, this component is not aware of Redux at all:


components/NewBookmark.js
```
import React from 'react';

class NewBookmark extends React.Component {
  state = {
    title: '',
    url: ''
  };

  handleInputChange = e => {
    this.setState({
      [e.target.name]: e.target.value
    });
  };

  handleSubmit = e => {
    e.preventDefault();
    if (this.state.title.trim() && this.state.url.trim()) {
      this.props.onAddBookmark(this.state);
      this.handleReset();
    }
  };

  handleReset = () => {
    this.setState({
      title: '',
      url: ''
    });
  };

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <input
          type="text"
          placeholder="title"
          name="title"
          onChange={this.handleInputChange}
          value={this.state.title}
        />
        <input
          type="text"
          placeholder="URL"
          name="url"
          onChange={this.handleInputChange}
          value={this.state.url}
        />
        <hr />
        <button type="submit">Add bookmark</button>
        <button type="button" onClick={this.handleReset}>
          Reset
        </button>
      </form>
    );
  }
}

export default NewBookmark;

```


And that’s all there is to it! Our simple bookmark manager app is working, getting data from the store and dispatching actions.


🏇 With this, you should be off to the races! For a different take on the topic, you can also have a look at the official docs.


