# How to integrate user authentication in a SolidJS app using Supabase

```JavaScript``` ```PaaS``` ```Write for DO```

## Introduction


The vast majority of applications demand identity verification from users; because security for websites and applications is crucial. Verifying users’ identities through authentication is a procedure utilized by several applications to handle security; This is done mostly to prevent the public from learning about private information and to prevent users from acting on behalf of another.


Supabase is a free and open-source alternative to Firebase. It is a BaaS (Backend as a service) platform that gives you access to several tools and services to assist in creating and maintaining your app. One of the fastest-expanding BaaS platforms, it is supported by YC and Mozilla. Among the many offers that Supabase provides are Storage, Realtime Database, functions, and authentication. It saves developers the hassle of creating their backends and includes its unique security measures.


In this tutorial, you will integrate Supabase into a SolidJS application to handle user authentication.


# Prerequisites


To follow this tutorial, you will need the following:


- Node.js is installed on your machine, which you can set up by following the tutorial on How To Install Node.js.
- A web browser like Firefox or Chrome.
- A Supabase account. Visit Supabase website to create an account.
- (Optional) Basic knowledge of the SolidJS Library. You can review the SolidJS documentation.
- (Optional) A text editor that supports JavaScript syntax highlighting, such as Visual Studio Code or Atom. The nano command-line editor is used in this tutorial.

# Step 1 - Creating the SolidJS project


In this section, you will create the SolidJS project, install all necessary dependencies and create the required components.


You will utilize Vite Template, which provides the needed boilerplate code to create a SolidJS app in either JavaScript or TypeScript.


Open a fresh terminal window, Make a new directory that will be used for this tutorial, and move there:


```
    mkdir auth
    cd auth


```


The directory is called auth for this tutorial, but you are free to name it as you see fit.


Afterward, Execute the following command npx degit to clone the template from the solid/js/templates/js repository to your project directory:


```
npx degit solidjs/templates/js solid-auth


```


Any other name for your application can be used in place of solid-auth.



Note: TypeScript is JavaScript with a type-specific syntax. Building on JavaScript, TypeScript is a strongly typed programming language that offers better tooling at any size. It supports classes, interfaces, and optional static typing. Additionally, it provides improved code completion and IntelliSense for JSX. Execute these commands if you would rather use TypeScript:
npx degit solidjs/templates/ts solid-auth


You’ll receive the same result for the /ts template as for the /js template.
The /ts and /js templates will generate the same results for you.

You will get the following result:


```
Output> cloned solidjs/templates#HEAD to solid-auth

```


This result demonstrates that the template was successfully saved to your computer. When the template has been cloned to your project folder, go there and install the project’s required dependencies:


```
cd solid-auth
npm install


```


The dependencies needed for the project will be installed by NPM.
To configure user authentication in SolidJS, you will need three additional dependencies supabase-js, solid-supabase and solid-router.


Supabase-js is a Supabase client that is highly similar to JavaScript.
Solid-Supabase is a basic Supabase.js wrapper that provides you access to the client as a Solid hook.


Solid Router is a general-purpose router for SolidJS; it functions whether rendering is taking place on the client or the server. It draws on and integrates the React Router and Ember Router principles.


To install these dependencies, use the following command:


```
npm install @solidjs/router @supabase/supabase-js solid-supabase


```


Next, start the development server by running the following command:


```
npm run dev


```


The output will look something like this:


```
Output...
  VITE v4.0.4  ready in 848 ms

  ➜  Local:   http://localhost:3000/
  ➜  Network: use --host to expose
  ➜  press h to show help

```


Your application is now running on port 3000. To access the SolidJS startup page, launch a browser and input the following URL: http://localhost:3000/:


SolidJS startup page

Note: Port forwarding can be used to test the app in the browser if you are following the tutorial on a remote server.
Run the following command in a different terminal that is open on your local computer as the development server is still active:
ssh -L 3000:localhost:3000  your_non_root_user@your_server_ip


Open the local machine’s web browser and go to “http://localhost:3000” after connecting to the server. For the duration of this tutorial, leave the second terminal open.

Next, you create the components for the application along with the required styling which is the Login, Register, and Dashboard components.
Make a components folder in the src directory:


```
mkdir src/components


```


Subsequently, include a new “Login.jsx” file:


```
nano src/components/Login.jsx


```


Import createSignal and A into the new Login.jsx file:


src/components/Login.jsx
```
import { createSignal } from "solid-js";
import { A } from "@solidjs/router";

```


Then include the lines below to create a Login component:


src/components/Login.jsx
```
const Login = () => {
    const [email, setEmail] = createSignal(''); // email of the user
    const [password, setPassword] = createSignal(''); // password of the user

    return (
        <div class="account-section">
            <form>
                <h3>Login</h3>
                <label>Email</label>
                <input type="email"
                    onChange={(e) => setEmail(e.target.value)} />
                <label>Password</label>
                <input
                    type="password"
                    onChange={(e) => setPassword(e.target.value)}
                />
                <button type="submit">Login</button>
                <span>
                    Don't have an account? <A href="/register">Register here</A>
                </span>
            </form>
        </div>
    )
}

export default Login

```


Signals are the cornerstone of reactivity in Solid. When you alter the value of a Signal, all that depends on it no doubt updates because they comprise dynamic data.


The parameter used to generate the Signal is the starting value, and the return values are an array containing two functions (a getter and a setter).
A getter, a function that returns the current value, is the first returned value rather than the value itself. The library must monitor where that signal gets read to update accordingly.


An anchor element that directs you to a route can be created using the A component.


In the login function, The email and password are the getters with initial values of an empty string (“”) while setEmail and setPassword are the setters.
Now, you create a basic login form with a title, two inputs with labels, and a button. You utilize the onChange event and set the value of the input on the declared Signals. The A element to navigate to other pages once the route for your app gets configured. The Login component is then exported and accessible throughout the app.


After that, add a “Register.jsx” file:


```
nano src/components/Register.jsx


```


Import createSignal into the Register.jsx file:


src/components/Register.jsx
```
import { createSignal } from "solid-js";
import { A } from "@solidjs/router";

```


Then incorporate the following lines to create a Register component:


src/components/Register.jsx
```
const Register = () => {

    const [email, setEmail] = createSignal(''); // email of the user
    const [password, setPassword] = createSignal(''); // password of the user

    return (
        <div class="register-section">
            <form>
                <h3>Register</h3>
                <label>Email</label>
                <input type="email"
                    onChange={(e) => setEmail(e.target.value)} />
                <label>Password</label>
                <input
                    type="password"
                    onChange={(e) => setPassword(e.target.value)}
                />
                <button type="submit">Register</button>
                <span>
                    Already have an account? <A href="/login">Login here</A>
                </span>
            </form>
        </div>
    )
}

export default Register

```


The Signals used here are the same as those used in the src/components/Login.jsx.
At this point, you create a straightforward Register form with a title, two inputs with labels, and a button. You use the ‘onChange’ event and set the input’s value on the defined ‘Signals’. Finally, the Register component gets exported so it is available globally in the app.


Then include a “Dashboard.jsx” file:


```
nano src/components/Dashboard.jsx


```


Import createSignal into the Dashboard.jsx file:


src/components/Dashboard.jsx
```
import { createSignal } from "solid-js";

```


Next add the subsequent lines to create a “Dashboard” component:


```
const Dashboard = () => {
    const [user, setUser] = createSignal({}); // user details
    return (
       <div class="dashboard-section">
            <div class="user-detail">
                <h3>Dashboard</h3>
                <h4>Welcome, User</h4>
                <button type="button" class="logout">Log out</button>
            </div>
        </div>
    )
}

export default Dashboard

```


The Dashboard component contains the title and greetings in heading tags and a button. Then a Signal is created that will hold the details of the current user and initialize with an empty object({}).


Now in the src/index.css, add the basic styles that are utilized in the components:


src/index.css
```
* {
	box-sizing: border-box;
	margin: 0;
	padding: 0;
}

body {
	background-color: rgb(229, 231, 235);
}

h3 {
	font-size: 1.5rem;
	font-weight: 900;
	color: rgb(55, 65, 81);
    text-align: center;
}

form {
	padding: 48px;
	margin-top: 2rem;
	width: 370px;
	height: 400px;
	background-color: #fff;
	border-radius: 4px;
	box-shadow: 4px 4px 7px lightgray;
}


label {
	text-transform: capitalize;
	margin-top: 1.2rem;
    margin-bottom: 0.3rem;
	font-size: 0.8rem;
    letter-spacing: 0.5px;
	color: rgb(55, 65, 81);
}

input {
	width: 100%;
	line-height: 2rem;
	background-color: rgb(229, 231, 235);
	border: none;
    padding: 0.375rem 0.75rem;
	border-radius: 4px;
    margin: 0.2rem 0 1rem;
}

button {
	margin-top: 1.2rem;
    margin-bottom: 0.5rem;
	width: 100%;
	height: 3rem;
	font-size: 0.9rem;
	font-weight: 700;
	color: rgb(220, 229, 247);
	background-color: rgb(37, 99, 235);
	border-radius: 4px;
	border: none;
    cursor: pointer;
}

.account-section {
	display: flex;
	justify-content: center;
	width: 100%;
	margin-top: 1.4rem;
}


.dashboard-section {
    display: flex;
	justify-content: center;
	margin-top: 100px;
	width: 100%;
}

.user-detail {
    text-align: center;
    width: 370px;
}

.logout {
    background-color: #dc3545;
}

.error {
  color: #dc3545;
  margin: 0.5rem 0;
  display: block;
}


```


The CSS file is in the root directory, this means styles written there are available in all components, so there is no need to import it into each individual component.


Your login page now looks as below:

Similarly, the register appears as so:

The dashboard page is as shown:

You can now proceed to configure the routes for the application.
Open the src/index.jsx file:


```
nano src/index.jsx


```


Import the Router componenet:


src/index.jsx
```
import { Router } from "@solidjs/router";

```


Subsequently, wrap your root component inside the Router component:


src/index.jsx
```
render(() =>
        <Router>
            <App />
       </Router>
, document.getElementById('root'));

```


This provides a context that allows us to show the routes everywhere in the app.


Now, open the src/App.jsx file:


```
nano src/App.jsx


```


Import Routes,Route, then Login, Register  and Dashboard components respectively. Then configure the routes for your application:


src/App.jsx
```
import { Routes, Route } from "@solidjs/router"
import Register from './components/Register';
import Login from './components/Login';
import Dashboard from './components/Dashboard';

function App() {
  return (
    <>
      <Routes>
        <Route path="/" component={Dashboard}/>
        <Route path="/login" component={Login}/>
        <Route path="register" component={Register} />
      </Routes>
    </>
  );
}

export default App;

```


To determine where the routes should appear in your app, use the Routes component. Then, When adding a route, use the Route component to define a path, an element or component to render when the user navigates to that path, and the route directly.
Once done, you can proceed to set up your project on Supabase.


# Step 2 - Setting up Supabase


To get started, go to the Supabase website on your browser. Then, on the right of the navigation bar, click the “Start your project” button; you are directed to the sign-in page. To sign in or to sign up, enter your information as needed. You’ll be taken to the project page once you log in.


Create a project from here by clicking the “New project” button. You’ll need to enter the specifics of the project on the create new project page as shown below:


Supabase project creation page
Once done, You are taken to the dashboard of the project created.
Proceed to select authentication from the side menu, then select providers to enable the needed authentication method. In this article, you will be utilizing email authentication shown enabled below:



As a convention, the user must first validate their email address before logging in, but in this tutorial that function was disabled.
To integrate Supabase into your Solid app, you need to add the Supabase project URL and API key to your app. On your project dashboard side menu, navigate to settings then select API to see your project URL and keys as below:



Now, proceed to add Supabase to your SolidJS app.


# Step 3 - Integrating Supabase to SolidJS


To configure Supabase in SolidJS, copy the Supabase URL and key from your project setting on the Supabase console.
At the root of your app create an environment file .env file to store this URL and key as environment variables:


```
nano .env


```


Proceed to add the values in a variable as so:


.env
```
VITE_SUPABASE_URL=your supabase url
VITE_SUPABASE_KEY=your supabase key

```



Note: An environment variable is a value with a dynamic name that can modify how active processes behave on a computer. Any number of environment variables may be created and made accessible for use at one time; each environment variable consists of a name/value pair. Among the use cases for environment variables are Operation mode (e.g., production, development, staging, etc.), URLs/URIs for APIs, domain names, and public and private authentication keys.

The .env file’s environment variables are loaded by Vite using dotenv.


Then at the root of your app, set up the Supabase provider to wrap the entire app and pass the details from the .env as the parameters.


Open the src/index.jsx file and add the following codes:


src/index.js
```
import { render } from 'solid-js/web';
import { Router } from "@solidjs/router";
import { createClient } from '@supabase/supabase-js';
import { SupabaseProvider } from 'solid-supabase';

import './index.css';
import App from './App';

const supabase = createClient(import.meta.env.VITE_SUPABASE_URL, import.meta.env.VITE_SUPABASE_KEY);

render(() =>

     <SupabaseProvider client={supabase}>
        <Router>
            <App />
        </Router>
     </SupabaseProvider> 

, document.getElementById('root'));

```


First, you import createClient and SupabaseProvider from @supabase/supabase-js and solid-supabase. Then you create a constant supabase and utilize the createClient() method to initiate a new Supabase client. Also, proceed to pass the Supabase URL and key as parameters.


The Supabase client is the interface via which Supabase functionalities can be accessed, and it provides the simplest means of communication with all of the Supabase ecosystem’s components. The unique import.meta.env object provided by Vite reveals the environment variables.


Then take advantage of the SupabaseProvider as a wrapper and supply the supabase client with your credentials. The supabase client will then be accessible throughout the entire app as a result.


Proceed to the created components and hook the appropriate Supabase functions to set up the authentication.


In the src/components/Login.jsx file, first import createSupabase and useNavigate respectively:


src/components/Login.jsx
```
import { createSupabase } from 'solid-supabase';
import { useNavigate, A } from "@solidjs/router";

```


Then update the Login component with supabase authentication method:


src/components/Login.jsx
```
import { createSignal } from "solid-js";
import { createSupabase } from 'solid-supabase';
import { useNavigate, A } from "@solidjs/router";

const Login = () => {
 const supabase = createSupabase(); 
    const [email, setEmail] = createSignal(''); // email of the user
    const [password, setPassword] = createSignal(''); // password of the user
 const navigate = useNavigate(); 

  <^>  const loginUser = async (e) => {
        e.preventDefault();
        const { data, error } = await supabase.auth.signInWithPassword({
            email: email(),
            password: password(),
        })
        if (error) {
            alert(error.message);
            return;
        }

        if (data) {
            navigate("/");
        }
    } <^>

    return (
        <div class="account-section">
            <form onSubmit={(e) => loginUser(e)}>
                <h3>Login</h3>
                <label>Email</label>
                <input type="email"
                    onChange={(e) => setEmail(e.target.value)} />
                <label>Password</label>
                <input
                    type="password"
                    onChange={(e) => setPassword(e.target.value)}
                />
                <button type="submit">Login</button>
                <span>
                    Don't have an account? <A href="/register">Register here</A>
                </span>
            </form>
        </div>
    )
}

export default Login

```


The createClient() method gets utilized to initialize a new client and stored in a constant supabase. An instance of the useNavigate() method is created and passed to the constant navigate. useNavigate() method takes a route to move to an auxiliary object. The loginUser is an async function that accepts an event (e) parameter that gets called on submission of the login form. The preventDefault() method blocks the default action of forms, which involves refreshing the page after submission. The supabase.auth.signInWithPassword() method accepts an object of the entered email and password. It uses the email and password to log in as an existing user.


Then use destructuring assignment to get the value of data and error. If there is an error, the error message is alerted and the function is exited. If it is successful, data returns the user details and navigate to the dashboard. The loginUser function gets called when the onSubmit event of the login form is triggered.


Then in the src/components/Register.jsx file, first import createSupabase and useNavigate also:


src/components/Register.jsx
```
import { createSupabase } from 'solid-supabase';
import { useNavigate, A } from "@solidjs/router";

```


Update the “Register” component using the supabase authentication method after that:


src/components/Register.jsx
```
import { createSignal } from "solid-js";
import { createSupabase } from 'solid-supabase';
import { useNavigate, A } from "@solidjs/router";

const Register = () => {
    const supabase = createSupabase();
    const [email, setEmail] = createSignal(''); // email of the user
    const [password, setPassword] = createSignal(''); // password of the user
    const navigate = useNavigate();

    <^>const registerUser = async (e) => {
        e.preventDefault();
        const { data, error } = await supabase.auth.signUp({
            email: email(),
            password: password(),
        })
        if (error) {
            alert(error.message);
            return;
        }

        if (data) {
            navigate("/");
        }
    } <^>

    return (
        <div class="account-section">
            <form onSubmit={(e) => registerUser(e)}>
                <h3>Register</h3>
                <label>Email</label>
                <input
                    type="email"
                    value={email()}
                    onChange={(e) => setEmail(e.target.value)} />
                <label>Password</label>
                <input
                    type="password"
                    value={password()}
                    onChange={(e) => setPassword(e.target.value)}
                />
                <button type="submit">Register</button>
                <span>
                    Already have an account? <A href="/login">Login here</A>
                </span>
            </form>
        </div>
    )
}

export default Register

```


A new client gets created using the createClient() method, which is then saved in the supabase constant. The constant navigate receives a reference to a new instance of the useNavigate() method. When the registration form is submitted, the async function registerUser that accepts the event (e) parameter is called. The form’s default behavior, which involves refreshing the page after submission, is prevented with the preventDefault() method. The supabase.auth.signUp() method takes an object of email and password. The values of dataanderrorare then obtained using the destructuring assignment. If there is an error, the error message is displayed and the function is terminated. If it is successful,datagives you the user's information and directs you to the dashboard. When the registration form'sonSubmitevent fires, theregisterUser` function is called.


In src/components/Dashboard.jsx file, import createEffect, createSupabase and useNavigate:


```
[src/components/Dashboard.jsx]
   import { createEffect, createSignal } from "solid-js";
import { createSupabase } from 'solid-supabase';
import { useNavigate } from "@solidjs/router";

```


Effects are specifically for side effects that read but do not write to the reactive system. Effects are routine methods for causing code segments are sometimes known as “side effects” to run if dependencies change, such as when manually modifying the DOM, or making calls to API. CreateEffect generates a fresh calculation that executes the specified function in a tracking scope, continuously monitoring its dependents, and rerunning the function whenever the dependencies change.


Next, add the supabase authentication method to the Dashboard component:


```
[src/components/Dashboard.jsx]
import { createEffect, createSignal } from "solid-js";
import { createSupabase } from 'solid-supabase';
import { useNavigate } from "@solidjs/router";

const Dashboard = () => {
    const [user, setUser] = createSignal({});
const navigate = useNavigate();
const supabase = createSupabase();

<^> createEffect(() => {
        getLoggedUser();
    })

    const getLoggedUser = async () => {
        const { data: { user } } = await supabase.auth.getUser();
        setUser(user)
        if(!user) {
            navigate("/login", { replace: true });
        }
    }

    const logOut = async () => {
        let { error } = await supabase.auth.signOut();

        if (error) {
            alert(error.message);
            return
        }
        setUser({})
        navigate("/login", { replace: true });
    }
<^>
    return (
        <div class="dashboard-section">
            <div class="user-detail">
                <h3>Dashboard</h3>
                <h4>Welcome, {user() && user().email}</h4>

                <button type="button" class="logout"  onClick={logOut}>Log out</button>
            </div>
        </div>
    )
}

export default Dashboard

```


The createClient() method is used to create a new client, which is subsequently saved in the supabase constant. A new instance of the useNavigate() method is referenced by the constant navigate.


getLoggedUser is an async function that utilizes the supabase.auth.getUser() method to retrieve the current user. If a session is already in progress, it obtains the current user’s information. Instead of using a local session, it fetches the user object from the database. The resulting user object returned gets set by the setUser function. If there’s no current user, the navigate function is employed to go back to the login page. It uses an additional parameter that is replace set to true; This alters the history entry removing the dashboard route from the history stack. As soon as the Dashboard component is mounted, this method is invoked since it is called in a createEffect.
logOut  is also an async function that uses the supabase.auth.signOut() method to log out the logged-in user. It will end the currently logged-in user’s browser session and log them out, deleting all data from local storage. If an error occurs, a message is shown and the function is stopped. The user is set to an empty object and the navigate method redirects to the login page.
The logged-in user email is displayed in the h4 tag, while the logOut function gets triggered by the onClick event on the logout button.
Save all files and test to ensure all works fine.


# Conclusion


In this article, you built a SolidJS app with user authentication that utilizes Supabase as the backend. You used solid-firebase that provided a selection of helpful Solid hooks for Supabase and solidjs/router that handled navigation in the app.


Visit the SolidJS documentation to learn more about the features of SolidJS. Check out the Supabase documentation for additional information about Supabase and the numerous features it offers.


