# Using Derived State in React

```React```

React v16.3 introduces some interesting new features, such as the getDerivedStateFromProps method. In this post we’ll explore how to use it.


In React v16.3, the getDerivedStateFromProps static lifecycle method was introduced as a replacement for componentWillReceiveProps. It’s important to move your components to this new method, as componentWillReceiveProps will soon be deprecated in an upcoming version of React.


Just like componentWillReceiveProps, getDerivedStateFromProps is invoked whenever a component receives new props.


# componentWillReceiveProps


Here’s a sample of what the old method would look like:


```
// For example purposes only, this is now deprecated

class List extends React.Component {
  componentWillReceiveProps(nextProps) {
    if (nextProps.selected !== this.props.selected) {
      this.setState({ selected: nextProps.selected });
      this.selectNew();
    }
  }

  // ...
}

```


As you can see, componentWillReceiveProps is often used to update the component’s state. It can also have side effects, such as the call to this.selectNew().


# getDerivedStateFromProps


The new method works a bit differently:


```
class List extends React.Component {
  static getDerivedStateFromProps(props, state) {
    if (props.selected !== state.selected) {
      return {
        selected: props.selected,
      };
    }

    // Return null if the state hasn't changed
    return null;
  }

  // ...
}

```


Instead of calling setState like in the first example, getDerivedStateFromProps simply returns an object containing the updated state. Notice that the function has no side-effects; this is intentional.


getDerivedStateFromProps may be called multiple times for a single update, so it’s important to avoid any side-effects. Instead, you should use componentDidUpdate, which executes only once after the component updates.


Here’s our final code:


```
class List extends React.Component {
  static getDerivedStateFromProps(props, state) {
    if (props.selected !== state.selected) {
      return {
        selected: props.selected,
      };
    }

    // Return null if the state hasn't changed
    return null;
  }

  componentDidUpdate(prevProps, prevState) {
    if (this.props.selected !== prevProps.selected) {
      this.selectNew();
    }
  }

  // ...
}

```


# Wrapping Up


getDerivedStateFromProps improves on the older method by providing a function whose only purpose is to update the state based on prop changes, without any side effects. This makes the component as a whole much easier to reason about.


However, derived state adds some complexity to components, and it’s often completely unnecessary. Check out the post You Probably Don’t Need Derived State for alternatives.


