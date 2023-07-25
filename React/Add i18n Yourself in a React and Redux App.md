# Add i18n Yourself in a React and Redux App

```React```

One thing that surprised me at first with React is how everything is a component. That may sound simple on the surface, but for the functional stuff (say form management, i18n (internationalization), routing, state…) it just doesn’t feel very natural to me and adds component layers for non-visual functional-only components.


Usually other UI libs/frameworks have an API-based way to deal with functional logic: a Router API, a Form API, etc. They provide a way to extend their own API or instance for those cases.


When looking into i18n solutions for React, I found that most solutions use translation components. They’re great, especially for large projects when you need more functionality and features, but my use case was quite simple, so I decided to do it myself. In this post I’ll describe how I did it.



If you’re interested in trying out one of the i18n libraries for React, check out this post covering i18next and react-i18next.

# The Literals Reducer


The solution I’ll show you uses Redux since it’s already a great state container, but you could build a container for the literals yourself if you’d like, the mechanics are similar.


First, let’s create a store/literals.js as the Redux piece of state to store the literals:


store/literals.js
```
const defaultState = {};

const LOAD_LITERALS = "LOAD_LITERALS";

export default (state = defaultState, { type, payload }) => {
  switch (type) {
    case LOAD_LITERALS:
      return payload;
    default:
      return state;
  }
};

export const loadLiterals = literals => ({
  type: LOAD_LITERALS,
  payload: literals,
});

```


Nothing special here if you’re already familiar with Redux. We just have a reducer that replaces the whole state of literals and a loadLiterals action creator to set them.



In a React/Redux app, the setup usually starts with the index.js file using React Redux’s Provider component:


index.js
```
import React from "react";
import ReactDOM from "react-dom";
import { Provider } from "react-redux";

import App from "./App";
import store from "./store";

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);

```


Here, store comes from a file where you create the Redux store, passing the root reducers to a function like combineReducers:


store/index.js
```
import { createStore, combineReducers } from "redux";
import literals from "./literals.js";

const rootReducer = combineReducers({
  literals,
  // other reducers...
});

export createStore(rootReducer);

```


Nothing special yet, just the usual steps to create a Redux store.


# Loading the Literals


Let’s create a folder with the following structure to organize your i18n logic:


```
+ i18n
  - index.js
  - en.json
  - es.json
  ....

```


The JSON files just have key-value data with the language literals:


i18n/en.json
```
{
  "app_greet": "Hey Joe!"
}

```


As for the index.js file, we can expose a function that returns the literals for a given language:


i18n/index.js
```
import en from "./en.json";
import es from "./es.json";

const langs = {
  en,
  es
};

export default function (lang = "en") {
  return langs[lang];
};

```


The idea is to load the literals early in your app. You probably have to initialize and configure other stuff as well at the same time. I sometimes create a init.js file for that, but just do that as you want.


However you must be sure that the store is already created. Let’s just do it in index.js, right after creating the store:


index.js
```
import React from "react";
import ReactDOM from "react-dom";
import { Provider } from "react-redux";

import App from "./App";
import { loadLiterals } from "store/literals";
import store from "./store";
import loadLang from "./i18n";

const lang = loadLang();
store.dispatch(loadLiterals(lang))

ReactDOM.render(
  
    <App />
  ,
  document.getElementById("root")
);

```


As you can see, we’re loading them by calling the loadLiterals function. We can call Redux actions from outside the components by using the store.dispatch instance method.


That should be enough to have your literals loaded. Then, in any component, you could just get your piece of the store using the connect function. Here’s a basic example:


index.js
```
import React from "react";
import { connect } from "react-redux";

const App = ({ literals }) => (
  <div>
    {literals.app_greet}
  </div>
);

const mapStateToProps = ({ literals }) => ({
  literals
});

export default connect(mapStateToProps)(App);

```


# Lazy Loading Literals


If we want to go a step further, we could change the default export in i18n/index.js to lazy load the literals by using the JavaScript dynamic import feature:


i18n/index.js
```
export default function (lang = "en") {
  return import(`./${lang}.json`);
};

```


Not only the function becomes simpler, but also the literals will be lazy loaded on demand, making the bundle size smaller, meaning an app that loads faster.


Since the dynamic import returns a promise, now we need to update how we load the literals in the store as follows:


index.js
```
import React from "react";
import ReactDOM from "react-dom";
import { Provider } from "react-redux";

import App from "./App";
import { loadLiterals } from "store/literals";
import store from "./store";
import loadLang from "./i18n";

loadLang().then(lang => store.dispatch(loadLiterals(lang)));

ReactDOM.render(
  
    <App />
  ,
  document.getElementById("root")
);

```


# Wrapping Up


We’ve seen how you can add some simple i18n functionality from scratch to your React/Redux apps. You don’t need to do things that way, and there surely are different ways to accomplish the same thing, but I hope you’ve seen that it can be easy and fun to do it yourself, and that this might be enough for simple use cases.


