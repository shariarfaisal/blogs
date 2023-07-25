# Testing React   Redux Apps with Jest   Enzyme - Part 4  Testing Redux Reducers

```React```

This is part 4 of a 4-part series on testing React / Redux apps using both Jest and Enzyme for a robust testing solution. In this part we’ll cover some simple examples on how to test Redux reducers.


## The series


- Part 1: Installation & Setup
- Part 2: Testing React Components
- Part 3: Testing Redux Actions
- Part 4: Testing Redux Reducers (this post)

Continuing with testing Redux, we’ll finish out this series by showing you how to test your reducers. Reducers are what complete the render chain in React / Redux applications. They update our state when an action is taken, which causes React to re-render the UI.


Let’s do it!


# Setup the Test Suite


The first thing to do is to import the reducer we want to test and our action types. We’ll also go ahead and write the describe() block to encapsulate our test suite for this reducer.



The action types file exports constants that map to the string value of each action.

__tests__/reducers/select_reducer.test.js
```
// Reducer to be tested
import selectReducer from '../../reducers/select_reducer';
import {
  SELECT_AVATAR, // Only ones related to the reducer being tested
} from '../../actions/types';

describe('select_reducer', () => {
  // ...
});

```


Take a moment to look at the code above. You may notice a few omissions from the setup code we used for testing actions. We didn’t mock out our store. That means we don’t need to import the redux-mock-store package. We also don’t need to run clearActions() before each test. This is because we’ll test the result returned from our reducers after calling them directly. We won’t dispatch() them into our store like we did with the actions.


# Test Your Reducers


Let’s first test our initial state. We do this by passing a dummy action into our reducer. I have a switch statement in the reducer with a default case. If the action type is unrecognized by the reducer then it returns the current, unmodified state. Otherwise, it returns the modified state. So by passing in a dummy action we can test our initialState (since we run this test first).


__tests__/reducers/select_reducer.test.js
```

// ...

describe('INITIAL_STATE', () => {
  test('is correct', () => {
    const action = { type: 'dummy_action' };
    const initialState = { selectedAvatar: 0 };

    expect(selectReducer(undefined, action)).toEqual(initialState);
  });
});

// ...

```


Next let’s test that our reducer modifies our state as expected when passed a recognized action and payload.


__tests__/reducers/select_reducer.test.js
```

// ...

describe('SELECT_AVATAR', () => {
  test('returns the correct state', () => {
    const action = { type: SELECT_AVATAR, payload: 1 };
    const expectedState = { selectedAvatar: 1 };

    expect(selectReducer(undefined, action)).toEqual(expectedState);
  });
});

// ...

```


## Snapshots


Just like with actions and components, once you have your tests passing you can leverage Snapshots and cut out a few lines of code:


__tests__/reducers/select_reducer.test.js
```

// ...

describe('INITIAL_STATE', () => {
  test('is correct', () => {
    const action = { type: 'dummy_action' };

    expect(selectReducer(undefined, action)).toMatchSnapshot();
  });
});

describe('SELECT_AVATAR', () => {
  test('returns the correct state', () => {
    const action = { type: SELECT_AVATAR, payload: 1 };

    expect(selectReducer(undefined, action)).toMatchSnapshot();
  });
});

// ...

```


# Bonus Tip!


Group your reducers into similar functionality whenever possible. With our selectReducers file we worked with the SELECT_AVATAR action type, yet there could easily be additional ones like SELECT_USER, SELECT_THEME, and so on in the same reducer. By grouping them together we improve our application architecture. If their functionality is similar we can apply the DRY principle. We have more opportunities to reuse code and write test helpers that can cover all like reducers.


# Conclusion


In Part 1 of this series I showed you how to install and setup Jest and Enzyme. In Part 2 I covered testing React components. In Part 3 I covered testing Redux actions. And we finished up in this post with testing reducers.


You should now have a well rounded setup and accompanying knowledge to test your React / Redux applications. Testing doesn’t have to be difficult. Thank you for following along!


