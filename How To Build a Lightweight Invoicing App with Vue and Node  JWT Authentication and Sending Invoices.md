# How To Build a Lightweight Invoicing App with Vue and Node  JWT Authentication and Sending Invoices

```Vue.js``` ```Node.js```

## Introduction


In the previous parts of the series, we looked at how to create the User Interface of our Invoicing Application that allowed users to create and view existing invoices. In this final part of the series, you will set up persisting user sessions on the client and configure a single view for invoices.


# Prerequisites


To follow this article adequately, you need the following:


- Node installed on your machine.
- NPM installed on your machine.
- To have read the first and second parts of this series.

To confirm your installation, run the following command:


```
node --version
npm --version


```


If you get their version numbers as results then you’re good to go.


# Step 1 — Persisting User Sessions on Client Using JWTokens


To verify that our application is secure and only authorized users can make requests, we are going to make use of JWTokens. JWTokens, or JSON Web Tokens consist of a three-part string containing the header, payload, and signature of the request. The core idea of it is to create a token for each authenticated user to use when performing requests to the backend server.


To get started, change into the invoicing-app directory. After doing that, install the jsonwebtoken node module that will be used to create and verify our JSON Web Tokens:


```
cd invoicing-app 
npm install jsonwebtoken nodemon --save

```


nodemon is a node module that restarts your server once file changes occur.


Now, update the server.js file by adding the following:


server.js
```
    
    // import node modules
    [...]
    const jwt = require("jsonwebtoken");
    
    // create express app
    [...]
    app.set('appSecret', 'secretforinvoicingapp'); // this will be used later

```


Next thing to do is to tweak the /register and /login routes to create tokens and pass them back once a user has successfully registered or logged in. To do this, add the following to your server.js file:


server.js
```

    // edit the /register route
    app.post("/register", multipartMiddleware, function(req, res) {
      // check to make sure none of the fields are empty
      [...]
      bcrypt.hash(req.body.password, saltRounds, function(err, hash) {
        // create sql query 
        [...]
        db.run(sql, function(err) {
          if (err) {
            throw err;
          } else {
            let user_id = this.lastID;
            let query = `SELECT * FROM users WHERE id='${user_id}'`;
            db.all(query, [], (err, rows) => {
              if (err) {
                throw err;
              }
              let user = rows[0];
              delete user.password;
              //  create payload for JWT
              const payload = {
                user: user 
              }
              // create token
              let token = jwt.sign(payload, app.get("appSecret"), {
                expiresInMinutes: "24h" // expires in 24 hours
              });
              // send response back to client
              return res.json({
                status: true,
                token : token
              });
            });
          }
        });
        db.close();
      });
    });
    
    [...]

```


Do the same for the /login route:


server.js
```
    app.post("/login", multipartMiddleware, function(req, res) {
      //  connect to db 
      [...]
      db.all(sql, [], (err, rows) => {
        // attempt to authenticate the user
        [...]
        if (authenticated) {
          //  create payload for JWT
          const payload = { user: user };
          // create token
          let token = jwt.sign( payload, app.get("appSecret"),{
            expiresIn: "24h" // expires in 24 hours
          });
          return res.json({
            status: true,
            token: token
          });
        }
        
        return res.json({
          status: false,
          message: "Wrong Password, please retry"
        });
      });
    });

```


Now that this is done, the next thing to do is to test it. Run your server using the following command:


```
nodemon server.js


```


Your app will now create tokens on successful logins and registrations. The next step is to verify tokens for incoming requests. To do this, add the following middleware above the routes you want to protect:


server.js
```
    
    [...]
    // unprotected routes
    
    [...]
    // Create middleware for protecting routes
    app.use(function(req, res, next) {
      // check header or url parameters or post parameters for token
      let token =
        req.body.token || req.query.token || req.headers["x-access-token"];
      // decode token
      if (token) {
        // verifies secret and checks exp
        jwt.verify(token, app.get("appSecret"), function(err, decoded) {
          if (err) {
            return res.json({
              success: false,
              message: "Failed to authenticate token."
            });
          } else {
            // if everything is good, save to request for use in other routes
            req.decoded = decoded;
            next();
          }
        });
      } else {
        // if there is no token
        // return an error
        return res.status(403).send({
          success: false,
          message: "No token provided."
        });
      }
    });
    
    // protected routes 
    [...]

```


In the SignUp.vue file you need to store the token obtained from the server and the user data in the localStorage so that it can persist across different pages when the user is using your application. To do this, update the login and register methods of your frontend/src/components/SignUp.vue file to look like this:


frontend/src/components/SignUp.vue
```
    [...]
    export default {
      name: "SignUp",
      [...]
      methods:{
        register(){
          const formData = new FormData();
          let valid = this.validate();
          if(valid){
            // prepare formData
            [...]
            // Post to server
            axios.post("http://localhost:3128/register", formData)
            .then(res => {
              // Post a status message
              this.loading = "";
              if (res.data.status == true) {
                // store the user token and user data in localStorage
                localStorage.setItem('token', res.data.token);
                localStorage.setItem('user', JSON.stringify(res.data.user));
                // now send the user to the next route
                this.$router.push({
                  name: "Dashboard",
                });
              } else {
                this.status = res.data.message;
              }
            });
          }
          else{
            alert("Passwords do not match");
          }
        }
        [...]

```


Let’s also update the login method:


frontend/src/components/SignUp.vue
```
        login() {
          const formData = new FormData();
          formData.append("email", this.model.email);
          formData.append("password", this.model.password);
          this.loading = "Signing in";
          // Post to server
          axios.post("http://localhost:3128/login", formData).then(res => {
            // Post a status message
            console.log(res);
            this.loading = "";
            if (res.data.status == true) {
              // store the data in localStorage
              localStorage.setItem("token", res.data.token);
              localStorage.setItem("user", JSON.stringify(res.data.user));
              // now send the user to the next route
              this.$router.push({
                name: "Dashboard"
              });
            } else {
              this.status = res.data.message;
            }
          });

```


Previously, user data was passed using route parameters, but now the app gets the data from the local storage. Let’s see how this changes our components.


The Dashboard component previously looked like this:


frontend/src/components/Dashboard.vue
```
    
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


This meant that when a user signed in or registered that they were redirected to the Dashboard page, and then the user property of the Dashboard component was updated accordingly. If the user had decided to refresh the page, there would be no way to identify the user since this.$route.params.user no longer exists.


Edit your Dashboard component to now use the browser’s localStorage:


frontend/src/components/Dashboard.vue
```
    
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
          user : null,
        };
      },
      mounted(){
        this.user = JSON.parse(localStorage.getItem("user"));
      }
    };

```


Now the user data will persist after refreshing the page. When requests are being made, you also have to add the token to the requests.


Take a look at the ViewInvoices component. Here’s what the JavaScript for the component looks like:


frontend/src/components/ViewInvoices.vue
```
    <script>
    import axios from "axios";
    export default {
      name: "ViewInvoices",
      components: {},
      data() {
        return {
          invoices: [],
\          user: '',
        };
      },
      mounted() {
        this.user = JSON.parse(localStorage.getItem('user'));
        axios
          .get(`http://localhost:3128/invoice/user/${this.user.id}`)
          .then(res => {
            if (res.data.status == true) {
              console.log(res.data.invoices);
              this.invoices = res.data.invoices;
            }
          });
      }
    };
    </script>

```


If you currently attempt to view invoices for a logged in user, you will get an error when retrieving invoices due to an absence of tokens.


This is because the invoice/user/:user_id route of the application is now protected with the token middleware that you set up earlier. Add it to the request to fix this error:


frontend/src/components/ViewInvoices.vue
```
    <script>
    import axios from "axios";
    export default {
      name: "ViewInvoices",
      components: {},
      data() {
        return {
          invoices: [],
          user: '',
        };
      },
      mounted() {
        this.user = JSON.parse(localStorage.getItem('user'));
        axios
          .get(`http://localhost:3128/invoice/user/${this.user.id}`,
            {
              headers: {"x-access-token": localStorage.getItem("token")}
            }
          )
          .then(res => {
            if (res.data.status == true) {
              console.log(res.data.invoices);
              this.invoices = res.data.invoices;
            }
          });
      }
    };
    </script>

```


When you save this and go back to your browser, you can now fetch the invoices successfully:


# Step 2 — Creating a Single View for Invoices


When the TO INVOICE button is clicked nothing happens. To fix this, create a SingleInvoice.vue file and edit it as follows:


```
    <template>
      <div class="single-page">
        <Header v-bind:user="user"/>
        <!--  display invoice data -->
        <div class="invoice">
          <!-- display invoice name here -->
          <div class="container">
            <div class="row">
                <div class="col-md-12">
                  <h3>Invoice #{{ invoice.id }} by {{ user.company_name }}</h3>
                  <table class="table">
                    <thead>
                      <tr>
                        <th scope="col">#</th>
                        <th scope="col">Transaction Name</th>
                        <th scope="col">Price ($)</th>
                      </tr>
                    </thead>
                    <tbody>
                      <template v-for="txn in transactions">
                        <tr :key="txn.id">
                          <th>{{ txn.id }}</th>
                          <td>{{ txn.name }}</td>
                          <td>{{ txn.price }} </td>
                        </tr>
                      </template>
                    </tbody>
                    <tfoot>
                      <td></td>
                      <td style="text-align: right">Total :</td>
                      <td><strong>$ {{ total_price }}</strong></td>
                    </tfoot>
                  </table>
                </div>
              </div>
            </div>
          </div>
      </div>
    </template>

```


The v-for directive is used to allow you to loop through all the fetched transactions for the particular invoice.


The component structure can be seen below. You first import the necessary modules and components. When the component is mounted, a POST request using axios is made to the backend server to fetch the data. When the response is obtained, we assign them to the respective component properties.


```
    <script>
    import Header from "./Header";
    import axios from "axios";
    export default {
      name: "SingleInvoice",
      components: {
        Header
      },
      data() {
        return {
          invoice: {},
          transactions: [],
          user: "",
          total_price: 0
        };
      },
      methods: {
        send() {}
      },
      mounted() {
        // make request to fetch invoice data
        this.user = JSON.parse(localStorage.getItem("user"));
        let token = localStorage.getItem("token");
        let invoice_id = this.$route.params.invoice_id;
        axios
          .get(`http://localhost:3128/invoice/user/${this.user.id}/${invoice_id}`, {
            headers: {
              "x-access-token": token
            }
          })
          .then(res => {
            if (res.data.status == true) {
              this.transactions = res.data.transactions;
              this.invoice = res.data.invoice;
              let total = 0;
              this.transactions.forEach(element => {
                total += parseInt(element.price);
              });
              this.total_price = total;
            }
          });
      }
    };
    </script>

```



Note: There’s a send() method that is currently empty. As you move on through the article, you will get a better understanding as to why and how to add the necessary functionality.

The component has the following scoped styles:


frontend/src/components/SingleInvoice.vue
```
    <!-- Add "scoped" attribute to limit CSS to this component only -->
    <style scoped>
    h1,
    h2 {
      font-weight: normal;
    }
    ul {
      list-style-type: none;
      padding: 0;
    }
    li {
      display: inline-block;
      margin: 0 10px;
    }
    a {
      color: #426cb9;
    }
    .single-page {
      background-color: #ffffffe5;
    }
    .invoice{
      margin-top: 20px;
    }
    </style>

```


Now, if you back to the application and click the TO INVOICE button in the View Invoices tab, you will see the single invoice view.


# Step 3 — Sending Invoice via Email


This is the final step of the invoicing application is to allow your users to send invoices. In this step, you will use the nodemailer module to send emails to specified recipients on the backend server. To get started, first install the module:


```
npm install nodemailer


```


Now that the module is installed, update the server.js file as follows:


server.js
```
    // import node modules
    [...]
    let nodemailer = require('nodemailer')
    
    // create mail transporter
    let transporter = nodemailer.createTransport({
      service: 'gmail',
      auth: {
        user: 'COMPANYEMAIL@gmail.com',
        pass: 'userpass'
      }
    });
    
    // create express app
    [...]

```


This email will be set on the backend server and will be the account sending emails on behalf of the user. Also, you will need to temporarily allow non-secure sign-in for your Gmail account for testing purposes in your security settings.


server.js
```

    // configure app routes
    [...]
    app.post("/sendmail", multipartMiddleware, function(req, res) {
      // get name  and email of sender
      let sender = JSON.parse(req.body.user);
      let recipient = JSON.parse(req.body.recipient);
      let mailOptions = {
        from: "COMPANYEMAIL@gmail.com",
        to: recipient.email,
        subject: `Hi, ${recipient.name}. Here's an Invoice from ${
          sender.company_name
        }`,
        text: `You owe ${sender.company_name}`
      };
      transporter.sendMail(mailOptions, function(error, info) {
        if (error) {
          return res.json({
            status: 200,
            message: `Error sending main to ${recipient.name}`
          });
        } else {
          return res.json({
            status: 200,
            message: `Email sent to ${recipient.name}`
          });
        }
      });
    });

```


At this point, you have configured the emails to work when a POST request is made to the /sendmail route. You also need to allow the user to perform this action on the frontend and give them a form to enter in the recipient’s email address. To do this, update the SingleInvoice component by doing the following:


frontend/src/components/SingleInvoice.vue
```
    
    <template>
     <Header v-bind:user="user"/>
        <!--  display invoice data -->
        <div class="invoice">
          <!-- display invoice name here -->
          <div class="container">
            <div class="row">
              <div class="col-md-12">
                // display invoice
              </div>
            </div>
            <div class="row">
              <form @submit.prevent="send" class="col-md-12">
                <h3>Enter Recipient's Name and Email to Send Invoice</h3>
                <div class="form-group">
                  <label for="">Recipient Name</label>
                  <input type="text" required class="form-control" placeholder="eg Chris" v-model="recipient.name">
                </div>
                <div class="form-group">
                  <label for="">Recipient Email</label>
                  <input type="email" required placeholder="eg chris@invoiceapp.com" class="form-control" v-model="recipient.email">
                </div>
                <div class="form-group">
                    <button class="btn btn-primary" >Send Invoice</button>
                    {{ loading }}
                    {{ status }}
                </div>
              </form>
            </div>
          </div>
        </div> 
    </template>

```


Also, the component properties are updated as follows:


frontend/src/components/SingleInvoice.vue
```
    
    <script>
    import Header from "./Header";
    import axios from "axios";
    export default {
      name: "SingleInvoice",
      components: {
        Header
      },
      data() {
        return {
          invoice: {},
          transactions: [],
          user: '',
          total_price: 0,
          recipient : {
            name: '',
            email: ''
          },
          loading : '',
          status: '',
        };
      },
      methods: {
        send() {
          this.status = "";
          this.loading = "Sending Invoice, please wait....";
          const formData = new FormData();
          formData.append("user", JSON.stringify(this.user));
          formData.append("recipient", JSON.stringify(this.recipient));
          axios.post("http://localhost:3128/sendmail", formData, {
            headers: {"x-access-token": localStorage.getItem("token")}
          }).then(res => {
            this.loading = '';
            this.status = res.data.message
          }); 
        }
      },
      mounted() {
        // make request to fetch invoice data
      }
    };
    </script>

```


After making these changes, your users will be able to enter in a recipient email and receive a “Sent Invoice” notification from the app.


You can further edit the email by reviewing the nodemailer guide.


# Conclusion


In this part of the series, we looked at how to use JWTokens and the Browser’s Local Storage to keep users signed in. We also created the view for a single invoice.


