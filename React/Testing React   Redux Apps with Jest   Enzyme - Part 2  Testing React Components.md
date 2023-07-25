# Testing React   Redux Apps with Jest   Enzyme - Part 2  Testing React Components

```React```

This is part 2 of a 4-part series on testing React / Redux apps using both Jest and Enzyme for a robust testing solution. In this part we’ll cover some simple examples on how to test React components.


## The series


- Part 1: Installation & Setup
- Part 2: Testing React Components (this post)
- Part 3: Testing Redux Actions
- Part 4: Testing Redux Reducers

When it comes to testing React components, things get a bit more involved than testing regular JavaScript functions. The good news is that you’re still just testing the public interface of your components. You still have an input (props) and output (what it renders).


If you read and followed Part 1 of this series there is even better news. You already have Jest and Enzyme installed and setup. These two libraries combined abstract away a lot of the pain of testing React components.


# What to Import


__tests__/components/GatorMenu.test.js
```
import React from 'react';
import { shallow } from 'enzyme';
import toJson from 'enzyme-to-json';
import configureStore from 'redux-mock-store'; // Smart components

// Component to be tested
import GatorMenu from '../../components/GatorMenu';

...

```



The shallow() method from Enzyme renders the current node and returns a shallow wrapper around it. This allows you to stay true to unit testing, only testing the component, and not asserting on children.


The toJson() method from enzyme-to-json converts Enzyme wrappers to a format compatible with Jest snapshot testing.


The configureStore() method from redux-mock-store is used to help mock out interactions with your Redux store. Only required for smart components that are connected to the store.

# Test that Your Component Renders


__tests__/components/GatorMenu.test.js
```
...

describe('<GatorMenu />', () => {
  describe('render()', () => {
    test('renders the component', () => {
      const wrapper = shallow(<GatorMenu />);
      const component = wrapper.dive();

      expect(toJson(component)).toMatchSnapshot();
    });
  });
});

...

```



The dive()method returns the rendered non-DOM child of the current wrapper. That becomes useful if your component wraps another component in something like a div element, and what you’re interested in testing is that inner component.

# Simulating Events


Here’s an example simulating a click event on a rendered component. We use the shallow-rendered component’s find method to find an element and then call the simulate method on the returned element with the name of the event passed-in as a string:


__tests__/components/GatorButton.test.js
```
...

describe('<GatorButton />', () => {
  describe('onClick()', () => {
    test('successfully calls the onClick handler', () => {
      const mockOnClick = jest.fn();
      const wrapper = shallow(
        <GatorButton onClick={mockOnClick} label="Eat Food" />
      );
      const component = wrapper.dive();

      component.find('button').simulate('click');

      expect(mockOnClick.mock.calls.length).toEqual(1);
    });
  });
});

...

```



The jest.fn() method allows you to easily mock and spy on functions.

# Testing Smart Components


I recommended testing your interactions with the Redux store through tests covering your actions and reducers. Try and keep your component tests focused on what is being rendered and the client-side behavior. With that being said, you can test interactions with your store within your smart components:


__tests__/components/GatorAvatar.test.js
```
import { Avatar } from 'react-native-elements';

...

const mockStore = configureStore();
const initialState = {
  selectReducer: {
    selectedAvatar: 0,
  },
  avatars: [
    {
      name: 'Green Gator',
      image: 'https://cdn.alligator.io/images/avatars/green-gator.jpg',
    },
    {
      name: 'Yellow Gator',
      image: 'https://cdn.alligator.io/images/avatars/yellow-gator.jpg',
    },
    {
      name: 'Blue Gator',
      image: 'https://cdn.alligator.io/images/avatars/blue-gator.jpg',
    },
  ],
};
const store = mockStore(initialState);

describe('<GatorAvatar />', () => {
  test('dispatches event to show the avatar selection list', () => {
    const wrapper = shallow(<GatorAvatar store={store} />);
    const component = wrapper.dive();

    component.find(Avatar).props().onPress();

    expect(store.getActions()).toMatchSnapshot();
  });
});

...

```


As you can see, we can get access to a component’s props by calling props() on an element.


# Bonus Tip!


Prior to Jest a lot of unit tests were written with Mocha or Jasmine. You can easily port those tests over to Jest, especially if you’re using the expect() flavor of assertions. You can even keep it() in place of test() and they should run. Or if you want to update things to test() then Find and Replace is your best friend.


# What’s Next?


In Part 1 of this series I showed you how to install and setup Jest and Enzyme. Now in Part 3 we’ll move on to testing Redux actions. We’ll finish up in Part 4 where I’ll show you how to test Redux reducers. Stay tuned!


