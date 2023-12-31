# How To Create Multistep Forms With React and Semantic UI

```JavaScript``` ```React```

## Introduction


Forms are used to collect user inputs on web applications. However, you may find yourself in situations where you need to collect large amounts of information from the user that may result in an overwhelming number of fields.


One solution is to break the form into multiple sections (or steps). Each section collecting a certain type of information at each point.


In this tutorial, you will build a multistep registration form with React and Semantic UI. You can achieve this by using React components for each section. You can then choose which components are rendered at each step by manipulating state.


# Prerequisites


To complete this tutorial, you will need:


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.
- It would be helpful to have a good understanding of React before starting this tutorial. You can learn more about React by following the How to Code in React.js series.

This tutorial was verified with Node v14.0.0, npm v6.14.4, react v16.13.1, semantic-ui-react v0.88.2, and semantic-ui-css v2.4.1.


# Step 1 — Initializing a new React Project with Semantic UI


First, you will use npx to use create-react-app to generate your project.


In your terminal, run the following:


```
npx create-react-app multistep-form


```


This will create a sample React application with the development environment fully configured.


Next, change into the newly created project directory:


```
cd multistep-form


```


For styling, you will use the UI framework, Semantic UI. To use Semantic UI in your React application, you will useSemantic UI React. Semantic UI React provides prebuilt components that you can use to speed up the development process.


It also supports responsive web design which makes it great for building cross-platform websites.


In your terminal, run the following:


```
npm install semantic-ui-react@0.88.2


```


Next, you will also include the default theme.


In your terminal, run the following:


```
npm install semantic-ui-css@2.4.1


```


To add the custom Semantic UI CSS, open the index.js file:


```
nano src/index.js


```


And add the following import:


src/index.js
```
import React from 'react';
import ReactDOM from 'react-dom';
import 'semantic-ui-css/semantic.min.css';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';

```


Now that the React project is initialized and you have added the necessary dependencies you can start creating your components.


# Step 2 — Creating the Components


In this tutorial step, you will be adding creating five components:


- MainForm.jsx
- UserDetails.jsx
- PersonalDetails.jsx
- Confirmation.jsx
- Success.jsx

First, let’s edit App.js to import the components and render them:


```
nano src/App.js


```


Replace all of the boilerplate code generated by create-react-app with the following lines of code:


src/App.js
```
import React, { Component } from 'react';
import { Grid } from 'semantic-ui-react';
import './App.css';
import MainForm from './components/MainForm';

class App extends Component {
  render() {
    return (
      <Grid verticalAlign='middle' style={{ height: '100vh' }}>
        <MainForm />
      </Grid>
    );
  }
}

export default App;

```



Note: If you start the development server now you will encounter errors until you have finished writing and importing all of the components for the form.

In this code, you have used the Grid component from Semantic UI React which makes it more presentable by centering the text and adding padding.


You have also imported and used the MainForm component. Let’s work on that now.


## Building the MainForm Component


The first component you will set up is the MainForm.jsx component which will be in charge of most of the functionality in your application.


Let’s create a components folder under the src directory:


```
mkdir src/components


```


And then create a Mainform.jsx file:


```
nano src/components/MainForm.jsx


```


Add the following lines of code:


src/components/MainForm.jsx
```
import React, { Component } from 'react';
import UserDetails from './UserDetails';
import PersonalDetails from './PersonalDetails';
import Confirmation from './Confirmation';
import Success from './Success';

```


This code will import the dependencies. It will also import the four sections of the form: UserDetails, PersonalDetails, Confirmation, and Success. You will be building these components next.


Then, underneath the imports, add state for the MainForm component:


src/components/MainForm.jsx
```
// ...

class MainForm extends Component {
  state = {
    step: 1,
    firstName: '',
    lastName: '',
    email: '',
    age: '',
    city: '',
    country: ''
  }
}

export default MainForm;

```


firstName, lastName, email, age, city, and country are the fields of information you are interested in end users providing.


step will be a number from 1 to 4. This is will allow you to track which step in the multistep process the user currently is.


Then, add the nextStep, prevStep, and handleChange functions:


src/components/MainForm.jsx
```
// ...

class MainForm extends Component {
  // ...

  nextStep = () => {
    const { step } = this.state
    this.setState({
      step : step + 1
    })
  }

  prevStep = () => {
    const { step } = this.state
    this.setState({
      step : step - 1
    })
  }

  handleChange = input => event => {
    this.setState({[input]: event.target.value})
  }
}

// ...

```


The component is initialized with the default value for step in state as 1, and the first section of your form is rendered. The user can then skip back and forth between steps using the prevStep and nextStep functions. These update the value of step in state so as to allow the user to switch between the rendered components.


The handleChange function updates the value of the details provided by the user inside state and like the prevStep and nextStep functions, it will be passed to the child components as props. This will allow you to pass the functionality implemented for use within the child components.


Then, add the render function:


src/components/MainForm.jsx
```
// ...

class MainForm extends Component {
  // ...

  render() {
    const { step } = this.state;
    const { firstName, lastName, email, age, city, country } = this.state;
    const values = { firstName, lastName, email, age, city, country };

    switch(step) {
      case 1:
        return <UserDetails
                  nextStep={this.nextStep} 
                  handleChange = {this.handleChange}
                  values={values}
                />
      case 2:
        return <PersonalDetails
                  nextStep={this.nextStep}
                  prevStep={this.prevStep}
                  handleChange = {this.handleChange}
                  values={values}
                />
      case 3:
        return <Confirmation
                  nextStep={this.nextStep}
                  prevStep={this.prevStep}
                  values={values}
                />
      case 4:
        return <Success />
    }
  }
}

// ...

```


The multistep form is using the switch statement, which reads the step from state and uses this to select which components are rendered at each step.


At Step 1, the UserDetails component will be rendered. At Step 2, the PersonalDetails component will be rendered. At Step 3, the Confirmation component will be rendered. And at Step 4, the Success component will be rendered.


Your MainForm component is now complete.


## Building the UserDetails Component


Now, let’s create the first section of the form.


Create a UserDetails.jsx file:


```
nano src/components/UserDetails.jsx


```


Add the following lines of code:


src/components/UserDetails.jsx
```
import React, { Component } from 'react';
import { Grid, Header, Segment, Form, Button } from 'semantic-ui-react';

class UserDetails extends Component {
  saveAndContinue = (e) => {
    e.preventDefault();
    this.props.nextStep();
  }

  render() {
    const { values } = this.props;

    return (
      <Grid.Column style={{ maxWidth: 450 }}>
        <Header textAlign='center'>
          <h1>Enter User Details</h1>
        </Header>

        <Form>
          <Segment>
            <Form.Field>
              <label>First Name</label>
              <input
                placeholder='First Name'
                onChange={this.props.handleChange('firstName')}
                defaultValue={values.firstName}
              />
            </Form.Field>

            <Form.Field>
              <label>Last Name</label>
              <input
                placeholder='Last Name'
                onChange={this.props.handleChange('lastName')}
                defaultValue={values.lastName}
              />
            </Form.Field>

            <Form.Field>
              <label>Email Address</label>
              <input
                type='email'
                placeholder='Email Address'
                onChange={this.props.handleChange('email')}
                defaultValue={values.email}
              />
            </Form.Field>
          </Segment>

          <Segment>
            <Button onClick={this.saveAndContinue}>Save And Continue</Button>
          </Segment>
        </Form>
      </Grid.Column>
    );
  }
}

export default UserDetails;

```


This creates a form that collects the user’s first name, last name, and email address.


The saveAndContinue function is then in charge of routing users to the next component once they are done filling in the details.


You will notice that you have called the nextStep function which you had provided to the component as props. Each time this function is called, it updates the state of the parent component (MainForm). You also called handleChange and provided the name of each field to be updated on each input element.


You may have also noticed that each input field is provided a defaultValue which it picks from the state of the MainForm component. This allows it to pick the updated value in state in cases where the user routes back to one step from another.


Your UserDetails component is now complete.


## Building the PersonalDetails Component


Now, let’s create your second section, which collects the user’s personal information.


Create a PersonalDetails.jsx file:


```
nano src/components/PersonalDetails.jsx


```


Add the following lines of code:


src/components/PersonalDetails.jsx
```
import React, { Component } from 'react';
import { Grid, Header, Segment, Form, Button } from 'semantic-ui-react';

class PersonalDetails extends Component {
  saveAndContinue = (e) => {
    e.preventDefault();
    this.props.nextStep();
  }

  back = (e) => {
    e.preventDefault();
    this.props.prevStep();
  }

  render() {
    const { values } = this.props;

    return (
      <Grid.Column style={{ maxWidth: 450 }}>
        <Header textAlign='center'>
          <h1>Enter Personal Details</h1>
        </Header>

        <Form>
          <Segment>
            <Form.Field>
              <label>Age</label>
              <input placeholder='Age'
                onChange={this.props.handleChange('age')}
                defaultValue={values.age}
              />
            </Form.Field>

            <Form.Field>
              <label>City</label>
              <input placeholder='City'
                onChange={this.props.handleChange('city')}
                defaultValue={values.city}
              />
            </Form.Field>

            <Form.Field>
              <label>Country</label>
              <input placeholder='Country'
                onChange={this.props.handleChange('country')}
                defaultValue={values.country}
              />
            </Form.Field>
          </Segment>

          <Segment textAlign='center'>
            <Button onClick={this.back}>Back</Button>

            <Button onClick={this.saveAndContinue}>Save And Continue</Button>
          </Segment>
        </Form>
      </Grid.Column>
    )
  }
}

export default PersonalDetails;

```


This section collects the user’s age and location. The overall functionality is similar to the user details section apart from the addition of the Back button which will take users back to the previous step by calling prevStep from props.


You have the back and saveAndContinue functions in each of your components take an event(e) as an argument and you call event.preventDefault() in these functions to stop the form from reloading each time users submit which is the default behavior in forms.


Your PersonalDetails component is now complete.


## Building the Confirmation Component


Now, let’s create the final section of your form where the user confirms the details they have fed the application are correct.


Create a Confirmation.jsx file:


```
nano src/components/Confirmation.jsx


```


Add the following lines of code:


src/components/Confirmation.jsx
```
import React, { Component } from 'react';
import { Grid, Header, Segment, Button, List } from 'semantic-ui-react';

class Confirmation extends Component {
  saveAndContinue = (e) => {
    e.preventDefault();
    this.props.nextStep();
  }

  back = (e) => {
    e.preventDefault();
    this.props.prevStep();
  }

  render() {
    const { values: { firstName, lastName, email, age, city, country } } = this.props;

    return (
      <Grid.Column style={{ maxWidth: 450 }}>
        <Header textAlign='center'>
          <h1>Confirm your Details</h1>

          <p>Click Confirm if the following details have been correctly entered</p>
        </Header>

        <Segment>
          <List divided relaxed>
            <List.Item>
              <List.Icon name='users' size='large' />
              <List.Content>First Name: {firstName}</List.Content>
            </List.Item>

            <List.Item>
              <List.Icon name='users' size='large' />
              <List.Content>Last Name: {lastName}</List.Content>
            </List.Item>

            <List.Item>
              <List.Icon name='mail' size='large' />
              <List.Content>Email: {email}</List.Content>
            </List.Item>

            <List.Item>
              <List.Icon name='calendar' size='large' />
              <List.Content>Age: {age}</List.Content>
            </List.Item>

            <List.Item>
              <List.Icon name='marker' size='large' />
              <List.Content>Location: {city}, {country}</List.Content>
            </List.Item>
          </List>
        </Segment>

        <Segment textAlign='center'>
          <Button onClick={this.back}>Back</Button>

          <Button onClick={this.saveAndContinue}>Confirm</Button>
        </Segment>
      </Grid.Column>
    )
  }
}

export default Confirmation;

```


This creates a section that displays all the details entered by the user and asks them to confirm the details before submitting them. This is typically where you would make the final API calls to the backend to submit and save the data.


Since this is a demo, the Confirm button implements the same saveAndContinue function you used in previous components. However, in a real-world scenario, you would implement a different submit function that would handle the final submission and save the data.


Your Confirmation component is now complete.


## Building the Success Component


The next and final component in your project is the Success component which would be rendered when the user successfully saves their information.


Create a Confirmation.jsx file:


```
nano src/components/Success.jsx


```


Add the following lines of code:


src/components/Success.jsx
```
import React, { Component } from 'react';
import { Grid, Header } from 'semantic-ui-react';

class Success extends Component {
  render() {
    return (
      <Grid.Column style={{ maxWidth: 450 }}>
        <Header textAlign='center'>
          <h1>Details Successfully Saved</h1>
        </Header>
      </Grid.Column>
    )
  }
}

export default Success;

```


This will display 'Details Successfully Saved' message when the user reaches the final step of the form.


You have completed all the components for your multistep form.


# Step 3 — Trying Out the Form


Now, view the form in your web browser.


Use a terminal window and ensure that you are in the project directory. Start the project with the following command:


```
npm start


```


Navigate to localhost:3000 in your web browser.


Your application will display Step 1 - Enter User Details:





After entering your information and clicking Save and Continue, you will be directed to Step 2 - Enter Personal Details:





After entering your information and clicking Save and Continue, you will be directed to Step 3 - Confirm Your Details:





After clicking Confirm, you will be directed to Step 4 - Success.


You now have a working multistep form in React.


# Conclusion


Multistep forms are great when you need to break down a form into sections. They can also allow you to carry out form validation at each step before carrying on to the next step.


The use of a switch statement for routing between steps eliminates the need for a router which means you can plug the form into other components without having to adjust routing.


It is also possible to set up API calls in between steps if necessary which opens up a number of new possibilities.


You can also handle the state better by adding state management tools such as Redux.


If you’d like to learn more about React, check out our React topic page for exercises and programming projects.


