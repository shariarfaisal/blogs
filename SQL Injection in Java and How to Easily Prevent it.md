# SQL Injection in Java and How to Easily Prevent it

```Database``` ```Java```

# What is SQL Injection?


SQL Injection is one of the top 10 web application vulnerabilities. In simple words, SQL Injection means injecting/inserting SQL code in a query via user-inputted data. It can occur in any applications using relational databases like Oracle, MySQL, PostgreSQL and SQL Server.


To perform SQL Injection, a malicious user first tries to find a place in the application where he can embed SQL code along with data. It can be the login page of any web application or any other place. So when data embedded with SQL code is received by the application, SQL code will be executed along with the application query.


# Impact of SQL Injection


- A malicious user can obtain unauthorized access to your application and steal data.
- They can alter, delete data in your database and take your application down.
- A hacker can also get control of the system on which database server is running by executing database specific system commands.

# How Does SQL Injection Works?


Suppose we have a database table named tbluser which stores data of application users. The userId is the primary column of the table. We have functionality in the application, which lets you get information via userId. The value of userId is received from the user request.


Let’s have a look at the below example code.


```
String userId = {get data from end user}; 
String sqlQuery = "select * from tbluser where userId = " + userId;

```


## 1. Valid User Input


When the above query is executed with valid data i.e. userId value 132, it will look like below.


Input Data: 132


Executed Query: select * from tbluser where userId=132


Result: Query will return data of user having userId 132. No SQL Injection is happening in this case.


## 2. Hacker User Input


A hacker can alter user requests using tools like Postman, cURL, etc. to send SQL code as data and this way bypassing any UI side validations.


Input Data: 2 or 1=1


Executed Query: select * from tbluser where userId=2 or 1=1


Result: Now the above query is having two conditions with SQL OR expression.


- userId=2: This part will match table rows having userId value as ‘2’.
- 1=1: This part will be always evaluate as true. So Query will return all the rows of the table.

# Types of SQL Injection


Let’s look at the four types of SQL injections.


## 1. Boolean Based SQL Injection


The above example is a case of Boolean Based SQL Injection. It uses a boolean expression that evaluates to true or false. It can be used to get additional information from the database. For example;


Input Data: 2 or 1=1


SQL Query:  select first_name, last_name from tbl_employee where empId=2 or 1=1


## 2. Union Based SQL Injection


SQL union operator combines data from two different queries with the same number of columns. In this case, the union operator is used to get data from other tables.


Input Data: 2 union select username, password from tbluser


Query:  Select first_name, last_name from tbl_employee where empId=2 union select username, password from tbluser


By using Union Based SQL Injection,  an attacker can obtain user credentials.


## 3. Time-Based SQL Injection


In  Time Based SQL Injection, special functions are injected in the query which can pause execution for a specified amount of time. This attack slows down the database server. It can bring down your application by affecting the database server performance. For example, In MySQL:


Input Data: 2 + SLEEP(5)


Query:  select emp_id, first_name, last_name from tbl_employee where empId=2 + SLEEP(5)


In the above example, query execution will pause for 5 seconds.


## 4. Error Based SQL Injection


In this variation, the attacker tries to get information like an error code and a message from the database. The attacker injects SQL which are syntactically incorrect so database server will return error code and messages which can be used to get database and system information.


# Java SQL Injection Example


We will use a simple Java Web application to demonstrate SQL Injection. We have Login.html, which is a basic login page that takes username and password from the user and submit them to LoginServlet.


The LoginServlet gets username and password from request and validates them against database values. If authentication is successful then Servlet redirects the user to the home page otherwise it will return an error.


Login.html Code:


```
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Sql Injection Demo</title>
    </head>
    <body>
    <form name="frmLogin" method="POST" action="https://localhost:8080/Web1/LoginServlet">
        <table>
            <tr>
                <td>Username</td>
                <td><input type="text" name="username"></td>
            </tr>
            <tr>
                <td>Password</td>
                <td><input type="password" name="password"></td>
            </tr>
            <tr>
                <td colspan="2"><button type="submit">Login</button></td>
            </tr>
        </table>
    </form>
    </body>
</html>

```


LoginServlet.java Code:


```
package com.journaldev.examples;
import java.io.IOException;
import java.sql.*;
import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;

@WebServlet("/LoginServlet")
public class LoginServlet extends HttpServlet {
    static {
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (Exception e) {}
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        boolean success = false;
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        // Unsafe query which uses string concatenation
        String query = "select * from tbluser where username='" + username + "' and password = '" + password + "'";
        Connection conn = null;
        Statement stmt = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/user", "root", "root");
            stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery(query);
            if (rs.next()) {
                // Login Successful if match is found
                success = true;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                stmt.close();
                conn.close();
            } catch (Exception e) {}
        }
        if (success) {
            response.sendRedirect("home.html");
        } else {
            response.sendRedirect("login.html?error=1");
        }
    }
}

```


Database Queries [MySQL]:


```
create database user;

create table tbluser(username varchar(32) primary key, password varchar(32));

insert into tbluser (username,password) values ('john','secret');
insert into tbluser (username,password) values ('mike','pass10');

```


## 1. When Valid Username and Password is entered from the login page


Input username: john


Input username: secret


Query: select * from tbluser where username=‘john’ and password = ‘secret’


Result: Username and Password exists in the database so authentication is successful. The user will be redirected to the home page.


## 2. Getting Unauthorized access to the system using SQL Injection


Input username: dummy


Input password: ’ or ‘1’='1


Query: select * from tbluser where username=‘dummy’ and password = ‘’ or ‘1’=‘1’


Result: Inputted Username and Password doesn’t exist in the database but authentication is successful.  Why?


It is due to SQL Injection as we have entered ’ or ‘1’='1 as password. There are 3 conditions in the query.


1. username=‘dummy’: It will be evaluated to false as there is no user with username dummy in the table.
2. password = ‘’: It will be evaluated to false as there is no empty password in the table.
3. ‘1’=‘1’: It will be evaluated to true as this is static string comparison.

Now combining all 3 conditions i.e false and false or true => Final result will be true.


In the above scenario, we have used the boolean expression to perform SQL Injection. There are some other ways to do SQL Injection. In the next section, we will see ways to prevent SQL injection in our Java application.


# Preventing SQL Injection in Java Code


The simplest solution is to use PreparedStatement instead of Statement to execute the query.


Instead of concatenating username and password into the query, we provide them to query via PreparedStatement’s setter methods.


Now, the value of username and password received from the request is treated as only data so no SQL Injection will happen.


Let’s look at the modified servlet code.


```
String query = "select * from tbluser where username=? and password = ?";
Connection conn = null;
PreparedStatement stmt = null;
try {
    conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/user", "root", "root");
    stmt = conn.prepareStatement(query);
    stmt.setString(1, username);
    stmt.setString(2, password);
    ResultSet rs = stmt.executeQuery();
    if (rs.next()) {
        // Login Successful if match is found
        success = true;
    }
    rs.close();
} catch (Exception e) {
    e.printStackTrace();
} finally {
    try {
        stmt.close();
        conn.close();
    } catch (Exception e) {
    }
}

```


Let’s understand what’s happening in this case.


Query: select * from tbluser where username = ? and password = ?


The question mark (?) in the above query is called a positional parameter.  There are 2 positional parameters in the above query. We don’t concatenate username and password to query. We use methods available in the PreparedStatement to provide user Input.


We have set the first Parameter by using stmt.setString(1, username)  and the second parameter by using stmt.setString(2, password). The underlying JDBC API takes care of sanitizing the values to avoid SQL injection.


# Best Practices to avoid SQL Injection


1. Validate data before using them in the query.
2. Do not use common words as your table name or column name. For example, many applications use tbluser or tblaccount to store user data. Email, firstname, lastname are common column names.
3. Do not directly concatenate data ( received as user input) to create SQL queries.
4. Use frameworks like Hibernate and Spring Data JPA for the data layer of an application.
5. Use positional parameters in the query. If you are using plain JDBC, then use PreparedStatement to execute the query.
6. Limit the application’s access to the database via permissions & grants.
7. Do not return sensitive error code and message to the end-user.
8. Do proper code review so that no developer accidentally write unsafe SQL code.
9. Use tools like SQLMap to find and fix SQL Injection vulnerabilities in your application.

That’s all for Java SQL Injection, I hope nothing important got missed here.


You can download the sample java web application project from the below link.


SQL Injection Java Project


