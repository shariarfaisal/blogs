# How To Build a Lightweight Invoicing App with Node  User Interface

```Node.js``` ```Vue.js``` ```Applications```

## Introduction


In the previous tutorial, you built the backend server for an invoicing application. In this tutorial, you will build the part of the application that users will interact with, known as the user interface.



Note: This is Part 2 of a 3-part series. The first tutorial is How To Build a Lightweight Invoicing App with Node: Database and API. The third tutorial is How To Build a Lightweight Invoicing App with Vue and Node: JWT Authentication and Sending Invoices.

The user interface in this tutorial will be built with Vue and allow users to log in to view and create invoices.


# Prerequisites


To complete this tutorial, you will need:


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.

This tutorial was verified with Node v16.1.0, npm v7.12.1, Vue v2.6.11, Vue Router v3.2.0, axios v0.21.1, and Bootstrap v5.0.1.


# Step 1 — Setting Up the Project


You can use @vue/cli to create a new Vue.js project.



Note: You should be able to place this new project directory next to invoicing-app directory you created in the previous tutorial. This introduces a common practice of separating server and client.

In your terminal window, use the following command:


```
npx @vue/cli create --inlinePreset='{ "useConfigFiles": false, "plugins": { "@vue/cli-plugin-babel": {}, "@vue/cli-plugin-eslint": { "config": "base", "lintOn": ["save"] } }, "router": true, "routerHistoryMode": true }' invoicing-app-frontend


```


This will use the inline preset configuration for creating a Vue.js Project with Vue Router.


Navigate to the newly created project directory:


```
cd invoicing-app-frontend


```


Start the project to verify that there are no errors.


```
npm run serve


```


If you visit the local app (typically at localhost:8080) in your web browser, you will see a "Welcome to Your Vue.js App" message.


This creates a sample Vue project that we’ll build upon in this article.


For the frontend of this invoicing application, a lot of requests are going to be made to the backend server.


To achieve this, we’ll make use of axios. To install axios, run the command in your project directory:


```
npm install axios@0.21.1


```


To allow some default styling in the application, you will make use of Bootstrap.


First, open the public/index.html file in your code editor.


Add the CDN-hosted CSS file for Bootstrap to the head of the document:


public/index.html
```
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-+0n0xVW2eSR5OomGNYDnhzAbDsOXxcvSN1TPprVMTNDbiYZCxYbOOl7+AMvyTG2x" crossorigin="anonymous">

```


Add the CDN-hosted JavaScript files for Popper and Bootstrap to the head of the document:


public/index.html
```
<script src="https://cdn.jsdelivr.net/npm/@popperjs/core@2.9.2/dist/umd/popper.min.js" integrity="sha384-IQsoLXl5PILFhosVNubq5LC7Qb9DXgDA9i+tQ8Zj3iwWAwPtgFTxbJ8NT4GN1R8p" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/js/bootstrap.min.js" integrity="sha384-Atwg2Pkwv9vp0ygtn1JAojH0nYbwNJLPhwyoVbhoPwBhjQPR5VtM2+xf0Uwh9KtT" crossorigin="anonymous"></script>

```


You can replace the contents of App.vue with the following lines of code:


src/App.vue
```
<template>
  <div id="app">
    <router-view/>
  </div>
</template>

```


And you can ignore or delete the src/views/Home.vue, src/views/About.vue, and src/components/HelloWorld.vue files that were automatically generated.


At this point, you have a new Vue project with Axios and Bootstrap.


# Step 2 — Configuring Vue Router


For this application, you are going to have two major routes:


- / to render the login page
- /dashboard to render the user dashboard

To configure these routes, open the src/router/index.js and update it with the following lines of code:


src/router/index.js
```
import Vue from 'vue'
import VueRouter from 'vue-router'

import SignUp from '@/components/SignUp'
import Dashboard from '@/components/Dashboard'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'SignUp',
    component: SignUp
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: Dashboard
  }
]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router

```


This specifies the components that should be displayed to the user when they visit your application.


# Step 3 — Creating Components


Components allow the frontend of your application to be more modular and reusable. This application will have the following components:


- Header
- Navigation
- Sign Up (and Sign In)
- Dashboard
- Create Invoice
- View Invoices

## Creating the Header Component


The Header component displays the name of the application and Navigation if a user is signed in.


Create a Header.vue file in the src/components directory. The component file has the following lines of code:


src/components/Header.vue
```
<template>
  <nav class="navbar navbar-light bg-light">
    <div class="navbar-brand m-0 p-3 h1 align-self-start">{{title}}</div>
    <template v-if="user != null">
      <Navigation v-bind:name="user.name" v-bind:company="user.company_name"/>
    </template>
  </nav>
</template>

<script>
import Navigation from './Navigation'

export default {
  name: "Header",
  props : ["user"],
  components: {
    Navigation
  },
  data() {
    return {
      title: "Invoicing App",
    };
  }
};
</script>

```


The Header component has a single prop called user. This prop will be passed by any component that will use the header component. In the template for the header, the Navigation component is imported and conditional rendering is used to determine if the Navigation should be displayed or not.


## Creating the Navigation Component


The Navigation component is the sidebar that will house the links of different actions.


Create a new Navigation.vue component in the /src/components directory. The component has the following template:


src/components/Navigation.vue
```
<template>
  <div class="flex-grow-1">
    <div class="navbar navbar-expand-lg">
      <ul class="navbar-nav flex-grow-1 flex-row">
        <li class="nav-item">
          <a class="nav-link" v-on:click="setActive('create')">Create Invoice</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" v-on:click="setActive('view')">View Invoices</a>
        </li>
      </ul>
    </div>
    <div class="navbar-text"><em>Company: {{ company }}</em></div>
    <div class="navbar-text h3">Welcome, {{ name }}</div>
  </div>
</template>

...

```


Next, open the Navigation.vue file in your code editor and add the following lines of code:


src/components/Navigation.vue
```
...

<script>
export default {
  name: "Navigation",
  props: ["name", "company"],
  methods: {
   setActive(option) {
      this.$parent.$parent.isactive = option;
    },
  }
};
</script>

```


The component is created with two props: the name of the user and the name of the company. The setActive method will update the component calling the parent of the Navigation component, in this case, Dashboard, when a user clicks on a navigation link.


## Creating the Sign Up Component


The SignUp component houses the sign up and sign in form. Create a new file in /src/components directory.


First, create the component:


src/components/SignUp.vue
```
<template>
  <div class="container">
    <Header/>

    <ul class="nav nav-tabs" role="tablist">
      <li class="nav-item" role="presentation">
        <button class="nav-link active" id="login-tab" data-bs-toggle="tab" data-bs-target="#login" type="button" role="tab" aria-controls="login" aria-selected="true">Login</button>
      </li>
      <li class="nav-item" role="presentation">
        <button class="nav-link" id="register-tab" data-bs-toggle="tab" data-bs-target="#register" type="button" role="tab" aria-controls="register" aria-selected="false">Register</button>
      </li>
    </ul>

    <div class="tab-content p-3">
      ...
    </div>
  </div>
</template>

<script>
import axios from "axios"

import Header from "./Header"

export default {
  name: "SignUp",
  components: {
    Header
  },
  data() {
    return {
      model: {
        name: "",
        email: "",
        password: "",
        c_password: "",
        company_name: ""
      },
      loading: "",
      status: ""
    };
  },
  methods: {
    ...
  }
}
</script>

```


The Header component is imported and the data properties of the components are also specified.


Next, create the methods to handle what happens when data is submitted:


src/components/SignUp.vue
```
...

  methods: {
    validate() {
      // checks to ensure passwords match
      if (this.model.password != this.model.c_password) {
        return false;
      }
      return true;
    },

    ...
  }

...

```


The validate() method performs checks to make sure the data sent by the user meets our requirements.


src/components/SignUp.vue
```
  ...

  methods: {
    ...

    register() {
      const formData = new FormData();
      let valid = this.validate();

      if (valid) {
        formData.append("name", this.model.name);
        formData.append("email", this.model.email);
        formData.append("company_name", this.model.company_name);
        formData.append("password", this.model.password);

        this.loading = "Registering you, please wait";

        // Post to server
        axios.post("http://localhost:3128/register", formData).then(res => {
          // Post a status message
          this.loading = "";

          if (res.data.status == true) {
            // now send the user to the next route
            this.$router.push({
              name: "Dashboard",
              params: { user: res.data.user }
            });
          } else {
            this.status = res.data.message;
          }
        });
      } else {
        alert("Passwords do not match");
      }
    },

    ...
  }

...

```


The register method of the component handles the action when a user tries to register a new account. First, the data is validated using the validate method. Then if all criteria are met, the data is prepared for submission using the formData.


We’ve also defined the loading property of the component to let the user know when their form is being processed. Finally, a POST request is sent to the backend server using axios. When a response is received from the server with a status of true, the user is directed to the dashboard. Otherwise, an error message is displayed to the user.


src/components/SignUp.vue
```
  ...

  methods: {
    ...

    login() {
      const formData = new FormData();

      formData.append("email", this.model.email);
      formData.append("password", this.model.password);
      this.loading = "Logging In";

      // Post to server
      axios.post("http://localhost:3128/login", formData).then(res => {
        // Post a status message
        this.loading = "";
    
        if (res.data.status == true) {
          // now send the user to the next route
          this.$router.push({
            name: "Dashboard",
            params: { user: res.data.user }
          });
        } else {
          this.status = res.data.message;
        }
      });
    }
  }

...

```


The login method is similar to the register method. The data is prepared and sent over to the backend server to authenticate the user. If the user exists and the details match, the user is directed to their dashboard.


Now, take a look at the template for registration:


src/components/SignUp.vue
```
<template>
  <div class="container">
    ...

    <div class="tab-content p-3">
      <div id="login" class="tab-pane fade show active" role="tabpanel" aria-labelledby="login-tab">
        <div class="row">
          <div class="col-md-12">
            <form @submit.prevent="login">
              <div class="form-group mb-3">
                <label for="login-email" class="label-form">Email:</label>
                <input id="login-email" type="email" required class="form-control" placeholder="example@example.com" v-model="model.email">
              </div>

              <div class="form-group mb-3">
                <label for="login-password" class="label-form">Password:</label>
                <input id="login-password" type="password" required class="form-control" placeholder="Password" v-model="model.password">
              </div>

              <div class="form-group">
                <button class="btn btn-primary">Log In</button>
                {{ loading }}
                {{ status }}
              </div>
            </form>
          </div>
        </div>
      </div>

      ...
    </div>
  </div>
</template>

```


The login in form is shown above and the input fields are linked to the respective data properties specified when the components were created. When the submit button of the form is clicked, the login method of the component is called.


Usually, when the submit button of a form is clicked, the form is submitted via a GET or POST request. Instead of using that, we added <form @submit.prevent="login"> when creating the form to override the default behavior and specify that the login function should be called.


The registration form also looks like this:


src/components/SignUp.vue
```
<template>
  <div class="container">
    ...

    <div class="tab-content p-3">
      ...

      <div id="register" class="tab-pane fade" role="tabpanel" aria-labelledby="register-tab">
        <div class="row">
          <div class="col-md-12">
            <form @submit.prevent="register">
              <div class="form-group mb-3">
                <label for="register-name" class="label-form">Name:</label>
                <input id="register-name" type="text" required class="form-control" placeholder="Full Name" v-model="model.name">
              </div>

              <div class="form-group mb-3">
                <label for="register-email" class="label-form">Email:</label>
                <input id="register-email" type="email" required class="form-control" placeholder="example@example.com" v-model="model.email">
              </div>

              <div class="form-group mb-3">
                <label for="register-company" class="label-form">Company Name:</label>
                <input id="register-company" type="text" required class="form-control" placeholder="Company Name" v-model="model.company_name">
              </div>

              <div class="form-group mb-3">
                <label for="register-password" class="label-form">Password:</label>
                <input id="register-password" type="password" required class="form-control" placeholder="Password" v-model="model.password">
              </div>

              <div class="form-group mb-3">
                <label for="register-confirm" class="label-form">Confirm Password:</label>
                <input id="register-confirm" type="password" required class="form-control" placeholder="Confirm Password" v-model="model.c_password">
              </div>

              <div class="form-group mb-3">
                <button class="btn btn-primary">Register</button>
                {{ loading }}
                {{ status }}
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

```


The @submit.prevent is also used here to call the register method when the submit button is clicked.


Now, run your development server using this command:


```
npm run serve


```


Visit localhost:8080 in your browser to observe the newly created login and registration page.



Note: When experimenting with the user interface, you will need to have the invoicing-app server running. Furthermore, you may encounter a CORS (cross-origin resource sharing) error that you may need to address by setting Access-Control-Allow-Origin headers.

Experiment with logging in and registering new users.


## Creating the Dashboard Component


The Dashboard component will be displayed when the user gets routed to the /dashboard route. It displays the Header and the CreateInvoice component by default.


Create the Dashboard.vue file in the src/components directory. The component has the following lines of code:


src/component/Dashboard.vue
```
<template>
  <div class="container">
    <Header v-bind:user="user"/>
    <template v-if="this.isactive == 'create'">
      <CreateInvoice />
    </template>
    <template v-else>
      <ViewInvoices />
    </template>
  </div>
</template>

...

```


Below the template, add the following lines of code:


src/component/Dashboard.vue
```
...

<script>
import Header from "./Header";
import CreateInvoice from "./CreateInvoice";
import ViewInvoices from "./ViewInvoices";

export default {
  name: "Dashboard",
  components: {
    Header,
    CreateInvoice,
    ViewInvoices,
  },
  data() {
    return {
      isactive: 'create',
      title: "Invoicing App",
      user : (this.$route.params.user) ? this.$route.params.user : null
    };
  }
};
</script>

```


## Creating the CreateInvoice Component


The CreateInvoice component contains the form needed to create a new invoice. Create a new file in the src/components directory:


Edit the CreateInvoice component to look like this:


src/components/CreateInvoice.vue
```
<template>
  <div class="container">
    <div class="tab-pane p-3 fade show active">
      <div class="row">
        <div class="col-md-12">
          <h3>Enter details below to create invoice</h3>
          <form @submit.prevent="onSubmit">
            <div class="form-group mb-3">
              <label for="create-invoice-name" class="form-label">Invoice Name:</label>
              <input id="create-invoice-name" type="text" required class="form-control" placeholder="Invoice Name" v-model="invoice.name">
            </div>

            <div class="form-group mb-3">
              Invoice Price: <span>${{ invoice.total_price }}</span>
            </div>

            ...
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

```


This creates a form that accepts the name of the invoice and displays the total price of the invoice. The total price is obtained by summing up the prices of individual transactions for the invoice.


Let’s take a look at how transactions are added to the invoice:


src/components/CreateInvoice.vue
```
        ...

          <form @submit.prevent="onSubmit">
            ...

            <hr />
            <h3>Transactions </h3>
            <div class="form-group">
              <button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#transactionModal">Add Transaction</button>

              <!-- Modal -->
              <div class="modal fade" id="transactionModal" tabindex="-1" aria-labelledby="transactionModalLabel" aria-hidden="true">
                <div class="modal-dialog" role="document">
                  <div class="modal-content">
                    <div class="modal-header">
                      <h5 class="modal-title" id="exampleModalLabel">Add Transaction</h5>
                      <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                      <div class="form-group mb-3">
                        <label for="txn_name_modal" class="form-label">Transaction name:</label>
                        <input id="txn_name_modal" type="text" class="form-control">
                      </div>
                      <div class="form-group mb-3">
                        <label for="txn_price_modal" class="form-label">Price ($):</label>
                        <input id="txn_price_modal" type="numeric" class="form-control">
                      </div>
                    </div>
                    <div class="modal-footer">
                      <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Discard Transaction</button>
                      <button type="button" class="btn btn-primary" data-bs-dismiss="modal" v-on:click="saveTransaction()">Save Transaction</button>
                    </div>
                  </div>
                </div>
              </div>

            </div>

            ...
          </form>

        ...

```


A button is displayed for the user to add a new transaction. When the Add Transaction button is clicked, a modal is shown to the user to enter the details of the transaction. When the Save Transaction button is clicked, a method adds it to the existing transactions.


src/components/CreateInvoice.vue
```
        ...

          <form @submit.prevent="onSubmit">
            ...

            <div class="col-md-12">
              <table class="table">
                <thead>
                  <tr>
                    <th scope="col">#</th>
                    <th scope="col">Transaction Name</th>
                    <th scope="col">Price ($)</th>
                    <th scope="col"></th>
                  </tr>
                </thead>
                <tbody>
                  <template v-for="txn in transactions">
                    <tr :key="txn.id">
                      <th>{{ txn.id }}</th>
                      <td>{{ txn.name }}</td>
                      <td>{{ txn.price }} </td>
                      <td><button type="button" class="btn btn-danger" v-on:click="deleteTransaction(txn.id)">Delete</button></td>
                    </tr>
                  </template>
                </tbody>
              </table>
            </div>

            <div class="form-group">
              <button class="btn btn-primary">Create Invoice</button>
              {{ loading }}
              {{ status }}
            </div>
          </form>

        ...

```


The existing transactions are displayed in a tabular format. When the Delete button is clicked, the transaction in question is deleted from the transaction list and the Invoice Price is recalculated. Finally, the Create Invoice button triggers a function that then prepares the data and sends it to the backend server for the creation of the invoice.


Let’s also take a look at the component structure of the Create Invoice component:


src/components/CreateInvoice.vue
```
...

<script>
import axios from "axios";

export default {
  name: "CreateInvoice",
  data() {
    return {
      invoice: {
        name: "",
        total_price: 0
      },
      transactions: [],
      nextTxnId: 1,
      loading: "",
      status: ""
    };
  },
  methods: {
    ...
  }
};
</script>

```


First, you defined the data properties for the component. The component will have an invoice object containing the invoice name and total_price. It’ll also have an array of transactions with the nextTxnId index. This will keep track of the transactions and variables to send status updates to the user.


src/components/CreateInvoice.vue
```
  ...

  methods: {
    saveTransaction() {
      // append data to the arrays
      let name = document.getElementById("txn_name_modal").value;
      let price = document.getElementById("txn_price_modal").value;

      if (name.length != 0 && price > 0) {
        this.transactions.push({
          id: this.nextTxnId,
          name: name,
          price: price
        });

        this.nextTxnId++;
        this.calcTotal();

        // clear their values
        document.getElementById("txn_name_modal").value = "";
        document.getElementById("txn_price_modal").value = "";
      }
    },

    ...
  }

...

```


The methods for the CreateInvoice component are also defined here. The saveTransaction() method takes the values in the transaction form modal and then adds them to the transaction list. The deleteTransaction() method deletes an existing transaction object from the list of transactions while the calcTotal() method recalculates the total invoice price when a new transaction is added or deleted.


src/components/CreateInvoice.vue
```
  ...

  methods: {
    ...

    deleteTransaction(id) {
      let newList = this.transactions.filter(function(el) {
        return el.id !== id;
      });
  
      this.nextTxnId--;
      this.transactions = newList;
      this.calcTotal();
    },

    calcTotal() {
      let total = 0;

      this.transactions.forEach(element => {
        total += parseInt(element.price, 10);
      });
      this.invoice.total_price = total;
    },

    ...
  }

...

```


Finally, the onSubmit() method will submit the form to the backend server. In the method, formData and axios are used to send the requests. The transaction array containing the transaction objects is split into two different arrays. One array holds the transaction names and the other holds the transaction prices. The server then attempts to process the request and send back a response to the user.


src/components/CreateInvoice.vue
```
  ...

  methods: {
    ...

    onSubmit() {
      const formData = new FormData();

      this.transactions.forEach(element => {
        formData.append("txn_names[]", element.name);
        formData.append("txn_prices[]", element.price)
      });

      formData.append("name", this.invoice.name);
      formData.append("user_id", this.$route.params.user.id);
      this.loading = "Creating Invoice, please wait ...";

      // Post to server
      axios.post("http://localhost:3128/invoice", formData).then(res => {
        // Post a status message
        this.loading = "";

        if (res.data.status == true) {
          this.status = res.data.message;
        } else {
          this.status = res.data.message;
        }
      });
    }
  }

...

```


When you go back to the application on localhost:8080 and sign in, you will get redirected to a dashboard.


## Creating the ViewInvoice Component


Now that you can create invoices, the next step is to create a visual picture of invoices and their statuses. To do this, create a ViewInvoices.vue file in the src/components directory of the application.


Edit the file to look like this:


src/components/ViewInvoices.vue
```
<template>
  <div>
    <div class="tab-pane p-3 fade show active">
      <div class="row">
        <div class="col-md-12">
          <h3>Here is a list of your invoices</h3>
          <table class="table">
            <thead>
              <tr>
                <th scope="col">Invoice #</th>
                <th scope="col">Invoice Name</th>
                <th scope="col">Status</th>
                <th scope="col"></th>
              </tr>
            </thead>
            <tbody>
              <template v-for="invoice in invoices">
                <tr :key="invoice.id">
                  <th scope="row">{{ invoice.id }}</th>
                  <td>{{ invoice.name }}</td>
                  <td v-if="invoice.paid == 0">Unpaid</td>
                  <td v-else>Paid</td>
                  <td><a href="#" class="btn btn-success">To Invoice</a></td>
                </tr>
              </template>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

...

```


The template above contains a table displaying the invoices a user has created. It also has a button that takes the user to a single invoice page when an invoice is clicked.


src/components/ViewInvoice.vue
```
...

<script>
import axios from "axios";

export default {
  name: "ViewInvoices",
  data() {
    return {
      invoices: [],
      user: this.$route.params.user
    };
  },
  mounted() {
    axios
      .get(`http://localhost:3128/invoice/user/${this.user.id}`)
      .then(res => {
        if (res.data.status == true) {
          this.invoices = res.data.invoices;
        }
      });
  }
};
</script>

```


The ViewInvoices component has its data properties as an array of invoices and the user details. The user details are obtained from the route parameters. When the component is mounted, a GET request is made to the backend server to fetch the list of invoices created by the user which are then displayed using the template that was shown earlier.


When you go to the /dashboard, click the View Invoices option on the Navigation to see a listing of invoices and payment status.


# Conclusion


In this part of the series, you configured the user interface of the invoicing application using concepts from Vue.


Continue your learning with How To Build a Lightweight Invoicing App with Vue and Node: JWT Authentication and Sending Invoices.


