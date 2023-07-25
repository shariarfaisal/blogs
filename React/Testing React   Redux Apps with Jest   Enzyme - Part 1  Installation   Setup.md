# Testing React   Redux Apps with Jest   Enzyme - Part 1  Installation   Setup

```React```

In this four-part series learn how to test your React / Redux applications using both Jest and Enzyme for a robust testing solution.



This series focuses on testing and assumes you have React / Redux knowledge.

## The series


- Part 1: Installation & Setup (this post)
- Part 2: Testing React Components
- Part 3: Testing Redux Actions
- Part 4: Testing Redux Reducers

# Recommended File Structure


- Leverage Default Behavior: By default Jest looks for tests in the __tests__ folder.
- Folder Hierarchy: Mirror your src folder hierarchy for easier test management.
- Separate By Type: Keep separate folders for components, actions and reducers.
- Append Filenames: For files containing tests use the .test.js extension to differentiate.


You can also use the .spec.js extension in place of .test.js. I prefer the latter.

```
.
├── __tests__
│   ├── actions
│   │   └── gator_actions.test.js
│   ├── components
│   │   └── Gator.test.js
│   ├── reducers
│   │   └── gator_reducer.test.js
│   └── setup
│       └── setupEnzyme.js
├── src
│   ├── actions
│   │   ├── gator_actions.js
│   │   ├── index.js
│   │   └── types.js
│   ├── components
│   │   └── Gator.js
│   ├── reducers
│   │   ├── gator_reducer.js
│   │   └── index.js
│   └── store
│       └── index.js
├── package.json
└── README.md


```


# Setup Jest & Enzyme



Jest is a zero-configuration JavaScript testing platform that keeps developer experience at the forefront.


Enzyme is a JavaScript testing utility for React that makes it easier to assert, manipulate, and traverse your React Components’ output.

## Install Jest


In your Terminal cd to the root of the React / Redux application you want to test. Then run one of the following commands depending on whether you use Yarn or npm to install Jest as a dev dependency.


Yarn:


```
$ yarn add -D jest

```


npm:


```
$ npm install -D jest

```


## Install Enzyme


Let’s install Enzyme which requires a couple of peer dependencies to easily integrate with Jest.



The following command assumes you’re using React 16. If you’re using a different version then make sure to install the Enzyme adapter for the version you’re using.

Yarn:


```
$ yarn add -D enzyme enzyme-adapter-react-16 enzyme-to-json

```


npm:


```
$ npm install -D enzyme enzyme-adapter-react-16 enzyme-to-json

```


## Configure Enzyme to work with Jest


There is a small amount of boilerplate code needed to integrate Enzyme with Jest and React.


__tests__/setup/setupEnzyme.js
```
import Enzyme from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';



```


Append the following to your package.json file:


package.json
```
"jest": {
  "setupTestFrameworkScriptFile": "<rootDir>__tests__/setup/setupEnzyme.js",
  "testPathIgnorePatterns": ["<rootDir>/__tests__/setup/"]
}

```


## Install Redux test helpers


Let’s go ahead and install some libraries to assist us with mocking our Redux store.


Yarn:


```
$ yarn add -D redux-mock-store

```


npm:


```
$ npm install -D redux-mock-store

```


## Add custom scripts for running tests


Let’s create some aliases for running our tests with either Yarn or npm:



Creating aliases via npm scripts allows you to keep a common interface for running tests among your projects which may have different flags an parameters being used.

package.json
```
...

"scripts": {
  "test": "jest",
  "test:watch": "jest --watch"
},

...

```


You can run tests like so:


Yarn:


```
// Single run
$ yarn test

// Watchmode
$ yarn test:watch

```


npm:


```
// Single run
$ npm run test

// Watchmode
$ npm run test:watch

```


# General Testing Tips


## Organize your tests into suites


Keeping your tests organized into suites is very easy with Jest and describe(). This makes it a bit easier to maintain your tests. You will also see the organization represented in the test results output.


Example:


__tests__/actions/gator_actions.js
```
...

describe('gator_actions', () => {
  beforeEach(() => {
    store.clearActions();
  });
  describe('eatFood', () => {
    test('Dispatches the correct action and payload', () => {
      store.dispatch(gatorActions.eatFood('fish'));
      expect(store.getActions()).toMatchSnapshot();
    });
  });


```


Test Results
```
PASS  __tests__/actions/gator_actions.test.js
  gator_actions
    eatFood
      ✓ Dispatches the correct action and payload (1ms)
    layEggs
      ✓ Dispatches the correct action and payload (1ms)


```


## Leverage Jest’s built-in coverage reports


With Jest you can generate coverage reports by adding the --coverage flag to your test running scripts. That’s all there is to it!





## When to use Snapshots


Jest offers a great feature called Snapshot Testing. If you’re not familiar with what it is, you can read this following post or ours: Introduction to Snapshot Testing With Jest.


Snapshots are ideal for testing things that you don’t expect to change or don’t want to change in the future. Snapshots provide a form of regression testing. We’ll use them to test components, actions and reducers and ensure they only change when we want them too.


## When not to use Snapshots


Don’t use Snapshots when you know that something will be different every time you run the test or when you first start writing a new component, action or reducer.


Example 1: You have a utility function that returns a random string. You cannot test this with Snapshots as it will be different each time. Instead, I recommend testing that the string is the proper length and made up of the correct types of characters based on the type of string being generated.


Example 2: You just started writing a new component and are writing tests as you go. I recommend avoiding Snapshots at first. Instead polish the component and get it close to what you know it will be. Then switch over to using Snapshots.


## Async tests


For some examples of writing asynchronous tests see the Asynchronous Tests in Jest post.


# What’s Next?


In Part 2 of this series I’ll be covering how to test React components. In Part 3 we’ll move on to testing Redux actions. We’ll finish up in Part 4 where I’ll show you how to test Redux reducers. Stay tuned!


