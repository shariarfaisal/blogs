# How To Use the res Object in Express

```Node.js```

## Introduction


In this article, you will learn about the res object in Express. Short for response, the res object is one half of the request and response cycle to send data from the server to the client-side through HTTP requests.


# Prerequisites


- 
An understanding of Node.js is helpful but not required. To learn more about Node.js, check out our How To Code in Node.js series.

- 
A general knowledge of HTTP requests. To learn more about HTTP Requests, check out our tutorial on How To Define Routes and HTTP Request Methods in Express.


# Examining the .status() and .append() Methods


The .send() method on the res object forwards any data passed as an argument to the client-side. The method can take a string, array, and an object as an argument.


In your index.js file, implement a GET request with the route '/home':


index.js
```
app.get('/home', (req, res) => {
  res.send('Hello World!'))
});

```


Notice that the GET request takes a callback argument with req and res as arguments. You can use the res object within the GET request to send the string Hello World! to the client-side.


The .send() method also defines its own built-in headers natively, depending on the Content-Type and Content-Length of the data.


The res object can also specify HTTP status codes with the .status() method. In your index.js file, integrate the .status() method on the res object and pass in an HTTP status code as an argument:


index.js
```
res.status(404).send('Not Found');

```


The .status() method on the res object will set a HTTP status code of 404. To send the status code to the client-side, you can method chain using the .send() method. The status code 404 tells the client side that the data requested is not found.


The .sendStatus() method is a shorthand syntax to adapt the functionality of both the .status() and .send() methods:


index.js
```
res.sendStatus(404);

```


Here, the .sendStatus() method will set the HTTP status code 404 and send it to the client-side in one call.


HTTP status codes summarizes your Express server’s response. Browsers rely on HTTP status codes to inform the client-side whether a specified data exists or if an internal server error occurs.


To define a header in your server response, apply the .append() method. In your index.js file, pass in a header as the first argument and a value as the second in your call to .append():


index.js
```
res.append('Content-Type', 'application/javascript; charset=UTF-8');
res.append('Connection', 'keep-alive')
res.append('Set-Cookie', 'divehours=fornightly')
res.append('Content-Length', '5089990');

```


In one line of code, the .append() method accepts standard and non-standard headers in your server response.


# Analyzing the .redirect(), .render(), and .end() Methods


The redirect() method on the res object will direct the client side to a different page. If a user inputs their login credentials on the client-side, the .redirect() method will facilitate the switch to their access page.


In your index.js file, set a .redirect() method on the res object:


index.js
```
res.redirect('/sharks/shark-facts')

```


Here, the .redirect() method will forward the client side to the route '/sharks/shark-facts'.


The .render() method accepts an HTML file as an argument and sends it to the client-side. The method also accepts an optional second argument, a locals object, with custom properties to define the file sent to the client-side.


In your index.js file, implement a GET request with the route '/shark-game':


```
[label index.js] 
app.get('/shark-game', (req, res) => {
  res.render('shark.html', {status: 'good'});
});

```


Using the .render() method on the res object will send the HTML file shark.html and the local object with the status property to the client-side.


The .end() method will terminate the response cycle. It is recommended to use the .end() method as the last call in your response to the client-side.


In your index.js file, set a .sentStatus() method chained with .end():


index.js
```
res.sendStatus(404).end();

```


The .end() method will complete the response once the HTTP status code 404 sets and sends it to the client-side.


The req object not only facilitates data transfer but also with files. Let’s look at other methods the req object contains for file management.


# Handling Files With the res Object


To send HTML, CSS, and JavaScript files to the client side, use the .sendFile() method on the res object. In your index.js file, set a GET request to the route '/gallery/:fileName':


```
// GET https://sharks.com/gallery/shark-image.jpg

app.get('/gallery/:fileName', function (req, res, next) {

  var options = {
    root: path.join(__dirname, 'public')
  };

  res.sendFile(req.params.fileName, options, function (err) {
    if (err) next(err);
    else console.log('Sent:', fileName);
  });
});

```


Within the GET request, the variable options takes an object and the public directory as the value in a call to path.join() to serve as an absolute path. Contents in the public directory include HTML, CSS, and JavaScript files. The call .sendFile method takes the options variable as a second argument and sets an error handler as the third. This will send the files stored in the public directory to the client-side.


You can also facilitate file handling with the .download() method on the res object. In your index.js file, implement a GET request to the route '/gallery/:fileName':


```
[label index.js] 
// GET https://sharkss.com/gallery/shark-image.jpg

app.get('/gallery/:fileName', function(req, res){
  const file = `${__dirname}/public/${req.params.fileName}`;
  res.download(file);
});

```


The .download() method sends and prompts the client-side to download a file and sets appropriate headers for the file type in one call.


To outline the value of a Content-Header on a file type, use the .type() method on the res object. In your index.js file, set a .type() method on the res object and pass a file type as an argument:


index.js
```
res.type('png')              // => 'image/png'
res.type('html')             // => 'text/html'
res.type('application/json') // =>'application/json'

```


The .type() method will output the file type with their associated values in a Content-Header.


# Conclusion


The res object holds methods to facilitate data and file transfer as part of your response cycle from your Express server to the client-side. To get comprehensive information about the res object, visit the Express.js official documentation website.


