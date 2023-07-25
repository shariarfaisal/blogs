# Testing React   Redux Apps with Jest   Enzyme - Part 3  Testing Redux Actions

```React```

This is part 3 of a 4-part series on testing React / Redux apps using both Jest and Enzyme for a robust testing solution. In this part we’ll cover some simple examples on how to test Redux actions.


## The series


- Part 1: Installation & Setup
- Part 2: Testing React Components
- Part 3: Testing Redux Actions (this post)
- Part 4: Testing Redux Reducers

We’ll now be moving into testing on the Redux side of things. This is where you begin to test business logic and application state. A benefit of using Redux in your applications is easy separation of business logic from rendering. You can write Redux tests that focus on the latter.


So let’s jump in and start writing some tests for our Redux actions.


# Mock out your Redux Store


Thanks to the redux-mock-store npm package that we installed we can mock our store in just two lines of code.


__tests__/actions/select_actions.test.js
```
import configureStore from 'redux-mock-store';

// Actions to be tested
import * as selectActions from '../../actions/select_actions';

const mockStore = configureStore();
const store = mockStore();

// ...

```


After mocking our store we gain access to some very nice test helpers such as dispatch(), getActions() and clearActions() which we’ll be putting to use next. For a full list of available methods you can check out the redux-mock-store API documentation.


- The dispatch() method dispatches an action through the mock store. The action will be stored in an array inside the instance and executed.
- The getActions() method returns the actions of the mock store.
- The clearActions() method clears the stored actions. Great for setup and teardown.

# Keep your Mock Store Sterile


With unit testing a lot of tests get ran in succession. To ensure we’re not getting results from a previous test we’ll write a small amount of setup code to clear out all actions from our mock store before running each test. We’ll leverage clearActions() for this.


__tests__/actions/select_actions.test.js
```
// ...

describe('select_actions', () => {
  beforeEach(() => { // Runs before each test in the suite
    store.clearActions();
  });

  // ...

});

```


# Test your Actions


In order for our Redux state to be updated properly we have to be sure that the correct actions and payloads are dispatched into our reducers. In our tests we’ll dispatch actions into our mock store using dispatch(). We’ll then check that the correct action type and payload are returned using getActions().


So let’s go inside of our select_actions test suite and add some action tests.


__tests__/actions/select_actions.test.js
```
// ...

describe('selectAvatar', () => {
  test('Dispatches the correct action and payload', () => {
    const expectedActions = [
      {
        'payload': 1,
        'type': 'select_avatar',
      },
    ];

    store.dispatch(selectActions.selectAvatar(1));
    expect(store.getActions()).toEqual(expectedActions);
  });
});

// ...

```


Once you have your test passing you can take advantage of Snapshot testing from Jest. This has the added benefit of letting you reduce your test code. Here’s an example:


__tests__/actions/select_actions.test.js
```
// ...

describe('selectAvatar', () => {
  test('Dispatches the correct action and payload', () => {
    store.dispatch(selectActions.selectAvatar(1));
    expect(store.getActions()).toMatchSnapshot();
  });
});

// ...

```


If you recall from the Testing Smart Components section of Part 2 I recommended testing your actions separately from your components. If you look at the expect() assertion in the code example in that section you’ll see some familiar code. This goes to show that you can keep things isolated without losing test coverage.


# Bonus Tip!


You can get into writing some very advanced action tests. I’d like to offer that with actions, less is more. If you find yourself writing a lot of complex tests you may need to rethink your Redux architecture. Things should feel light and easy when it comes to managing, thinking through and testing your Redux actions. Especially with this setup. Food for thought!


# What’s Next?


In Part 1 of this series I showed you how to install and setup Jest and Enzyme. In Part 2 I covered testing React components. We’ll finish up in Part 4 with testing Redux reducers. Stay tuned!


