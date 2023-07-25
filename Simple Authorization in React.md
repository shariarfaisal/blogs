# Simple Authorization in React

```React```

Most real world apps need authentication and authorization. While authentication identifies some entity as a valid user, authorization defines the actions that the user is allowed to perform, based on his/her roles and rights.


We don’t usually need any special module or library to handle authorization and in most cases, a few utility functions are enough. The solutions you provide for authentication or authorization in an app can vary: you may decide to keep the user state management on Redux, you may decide to build a dedicated module, etc.


Let’s see how to handle simple role-based authorization in React.



Keep in mind that the following is for client-side authorization, which can easily be bypassed. Authorization on the client-side has more to do with UX than with security. You’ll want to ensure that roles are secured on your backend.

# Simple Authorization


Say we have a user object, usually obtained by calling an endpoint like /me after authenticating, with the following structure:


```
const user = {
  name: 'Jackator',
  // ...
  roles: ['user'],
  rights: ['can_view_articles']
};

```


A user has several rights that can be grouped into roles. For your app, you may only need roles, or only rights, or both, it doesn’t matter. The REST API may give you the rights nested within the roles, it doesn’t matter either, just keep that in mind to adapt the solution to your needs. What’s important is that we have a user.


Then, we can create a auth.js file with some utility functions that we can use to check for user authorization:


auth.js
```
export const isAuthenticated = user => !!user;

export const isAllowed = (user, rights) =>
  rights.some(right => user.rights.includes(right));

export const hasRole = (user, roles) =>
  roles.some(role => user.roles.includes(role));

```


By using the some and includes methods from the Array prototype, we’re checking if that user has at least one of the rights or roles specified. That should be enough to perform some basic permission checking.


Since the user can be kept anywhere, in Redux for example, we allow to pass it as a parameter to the function.


Let’s finally create a basic React component that uses the functions defined in auth.js in order to conditionally show different pieces of UI:


App.js
```
import React from 'react';
import { render } from "react-dom";
import { hasRole, isAllowed } from './auth';

const user = {
  roles: ['user'],
  rights: ['can_view_articles']
};

const admin = {
  roles: ['user', 'admin'],
  rights: ['can_view_articles', 'can_view_users']
};

const App = ({ user }) => (
  <div>
    {hasRole(user, ['user']) && <p>Is User</p>}
    {hasRole(user, ['admin']) && <p>Is Admin</p>}
    {isAllowed(user, ['can_view_articles']) && <p>Can view Articles</p>}
    {isAllowed(user, ['can_view_users']) && <p>Can view Users</p>}
  </div>
);

render(
  <App user={user} />,
  document.getElementById('root')
);

```


I’m doing short circuit evaluation by using the logical && operator. In that way, only if the hasRole or isAllowed function returns true will the following content will be rendered.


Try to change the user to admin and you’ll see that the admin related UI is shown.


# Conditional Routes


With this solution, if you’re using React Router, you can conditionally render routes using the same method:


```
import React from 'react';
import { BrowserRouter, Switch, Route } from 'react-router-dom';

const App = ({ user }) => (
  <BrowserRouter>
    <Switch>
      {hasRole(user, ['user']) && <Route path='/user' component={User} />}
      {hasRole(user, ['admin']) && <Route path='/admin' component={Admin} />}
      <Route exact path='/' component={Home} />
    </Switch>
  </BrowserRouter>
);

```


React Router makes it easy to declare and compose routes using the Route component, and we can take advantage of it: the routes will be added and handled by the router only if the <Route> component is rendered by passing the hasRole evaluation.


# Wrapping Up


You’ve seen how to do a simple form of authorization yourself. The solution can be a bit different for you, depending on the application and requirements, but the core concepts should be the same.


You can check in this Codesandbox for the basic example seen in this article.


Stay cool 🦄


