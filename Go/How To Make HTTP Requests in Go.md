# How To Make HTTP Requests in Go

```Go``` ```Development```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


When a program needs to communicate with another program, many developers will use HTTP. One of Go’s strengths is the breadth of its standard library, and HTTP is no exception. The Go net/http package not only supports creating HTTP servers, but it can also make HTTP requests as a client.


In this tutorial, you will create a program that makes several types of HTTP requests to an HTTP server. First, you will make a GET request using the default Go HTTP client. Then, you will enhance your program to make a POST request with a body. Finally, you will customize your POST request to include an HTTP header and add a timeout that will trigger if your request takes too long.


# Prerequisites


To follow this tutorial, you will need:


- Go version 1.16 or greater installed. To set this up, follow the How To Install Go tutorial for your operating system.
- Experience creating an HTTP server in Go, which can be found in the tutorial, How To Make an HTTP Server in Go.
- Familiarity with goroutines and reading channels. For more information, see the tutorial, How To Run Multiple Functions Concurrently in Go.
- An understanding of how an HTTP request is composed and sent is recommended.

# Making a GET Request


The Go net/http package has a few different ways to use it as a client. You can use a common, global HTTP client with functions such as http.Get to quickly make an HTTP GET request with only a URL and a body, or you can create an http.Request to begin customizing certain aspects of the individual request. In this section, you will create an initial program using http.Get to make an HTTP request, and then you will update it to use an http.Request with the default HTTP client.


## Using http.Get to Make a Request


In the first iteration of your program, you’ll use the http.Get function to make a request to the HTTP server you run in your program. The http.Get function is useful because you don’t need any additional setup in your program to make a request. If you need to make a single quick request, http.Get may be the best option.


To start creating your program, you’ll need a directory to keep the program’s directory in. In this tutorial, you’ll use a directory named projects.


First, make the projects directory and navigate to it:


```
mkdir projects
cd projects


```


Next, make the directory for your project and navigate to it. In this case, use the directory httpclient:


```
mkdir httpclient
cd httpclient


```


Inside the httpclient directory, use nano, or your favorite editor, to open the main.go file:


```
nano main.go


```


In the main.go file, begin by adding these lines:


main.go
```
package main

import (
	"errors"
	"fmt"
	"net/http"
	"os"
	"time"
)

const serverPort = 3333

```


You add the package name main so that your program is compiled as a program you can run, and then include an import statement with the various packages you’ll be using in this program. After that, you create a const called serverPort with the value 3333, which you’ll use as the port your HTTP server is listening on and the port your HTTP client will connect to.


Next, create a main function in the main.go file and set up a goroutine to start an HTTP server:


main.go
```
...
func main() {
	go func() {
		mux := http.NewServeMux()
		mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
			fmt.Printf("server: %s /\n", r.Method)
		})
		server := http.Server{
			Addr:    fmt.Sprintf(":%d", serverPort),
			Handler: mux,
		}
		if err := server.ListenAndServe(); err != nil {
			if !errors.Is(err, http.ErrServerClosed) {
				fmt.Printf("error running http server: %s\n", err)
			}
		}
	}()

	time.Sleep(100 * time.Millisecond)

```


Your HTTP server is set up to use fmt.Printf to print information about incoming requests whenever the root / path is requested. It’s also set to listen on serverPort. Finally, once you start up the server goroutine, your program uses time.Sleep for a short amount of time. This sleep time allows the HTTP server the time it needs to start up and start serving responses to the request you’ll be making next.


Now, also in the main function, set up the request URL using fmt.Sprintf to combine the http://localhost hostname with the serverPort value the server is listening on. Then, use http.Get to make a request to that URL, as shown below:


main.go
```
...
	requestURL := fmt.Sprintf("http://localhost:%d", serverPort)
	res, err := http.Get(requestURL)
	if err != nil {
		fmt.Printf("error making http request: %s\n", err)
		os.Exit(1)
	}

	fmt.Printf("client: got response!\n")
	fmt.Printf("client: status code: %d\n", res.StatusCode)
}

```


When the http.Get function is called, Go will make an HTTP request using the default HTTP client to the URL provided, then return either an http.Response or an error value if the request fails. If the request fails, it will print the error and then exit your program using os.Exit with an error code of 1. If the request succeeds, your program will print out that it got a response and the HTTP status code it received.


Save and close the file when you’re done.


To run your program, use the go run command and provide the main.go file to it:


```
go run main.go


```


You will see the following output:


```
Outputserver: GET /
client: got response!
client: status code: 200

```


On the first line of output, the server prints that it received a GET request from your client for the / path. Then, the following two lines say that the client got a response back from the server and that the response’s status code was 200.


The http.Get function is useful for quick HTTP requests like the one you made in this section. However, http.Request provides a broader range of options for customizing your request.


## Using http.Request to Make a Request


In contrast to http.Get , the http.Request function provides you with greater control over the request, other than just the HTTP method and the URL being requested. You won’t be using additional features yet, but by using an http.Request now, you’ll be able to add those customizations later in this tutorial.


In your code, the first update is to change the HTTP server handler to return a fake JSON data response using fmt.Fprintf. If this were a full HTTP server, this data would be generated using Go’s encoding/json package. If you’d like to learn more about using JSON in Go, our How To Use JSON in Go tutorial is available. In addition, you will also need to include io/ioutil as an import for use later in this update.


Now, open your main.go file again and update your program to start using an http.Request as shown below:


main.go
```
package main

import (
	...
	"io/ioutil"
	...
)

...

func main() {
	...
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Printf("server: %s /\n", r.Method)
		fmt.Fprintf(w, `{"message": "hello!"}`)
	})
	...

```


Now, update your HTTP request code so that instead of using http.Get to make a request to the server, you use http.NewRequest and http.DefaultClient’s Do method:


main.go
```
...
	requestURL := fmt.Sprintf("http://localhost:%d", serverPort)
	req, err := http.NewRequest(http.MethodGet, requestURL, nil)
	if err != nil {
		fmt.Printf("client: could not create request: %s\n", err)
		os.Exit(1)
	}

	res, err := http.DefaultClient.Do(req)
	if err != nil {
		fmt.Printf("client: error making http request: %s\n", err)
		os.Exit(1)
	}

	fmt.Printf("client: got response!\n")
	fmt.Printf("client: status code: %d\n", res.StatusCode)

	resBody, err := ioutil.ReadAll(res.Body)
	if err != nil {
		fmt.Printf("client: could not read response body: %s\n", err)
		os.Exit(1)
	}
	fmt.Printf("client: response body: %s\n", resBody)
}

```


In this update, you use the http.NewRequest function to generate an http.Request value, or handle the error if the value can’t be created. Unlike the http.Get function, though, the http.NewRequest function doesn’t send an HTTP request to the server right away. Since it doesn’t send the request right away, you can make any changes you’d like to the request before it’s sent.


Once the http.Request is created and configured, you use the Do method of http.DefaultClient to send the request to the server. The http.DefaultClient value is Go’s default HTTP client, the same you’ve been using with http.Get. This time, though, you’re using it directly to tell it to send your http.Request. The Do method of the HTTP client returns the same values you received from the http.Get function so that you can handle the response in the same way.


After you’ve printed the request results, you use the ioutil.ReadAll function to read the HTTP response’s Body. The Body is an io.ReadCloser value, a combination of io.Reader and io.Closer, which means you can read the body’s data using anything that can read from an io.Reader value. The ioutil.ReadAll function is useful because it will read from an io.Reader until it either gets to the end of the data or encounters an error. Then it will either return the data as a []byte value you can print using fmt.Printf, or the error value it encountered.


To run your updated program, save your changes and use the go run command:


```
go run main.go


```


This time, your output should look very similar to before, but with one addition:


```
Outputserver: GET /
client: got response!
client: status code: 200
client: response body: {"message": "hello!"}

```


In the first line, you see that the server is still receiving a GET request to the / path. The client also receives a 200 response from the server, but it’s also reading and printing the Body of the server’s response. In a more complex program, you could then take the {"message": "hello!"} value you received as the body from the server and process it as JSON using the encoding/json package.


In this section, you created a program with an HTTP server that you made HTTP requests to in various ways. First, you used the http.Get function to make a GET request to the server using only the server’s URL. Then, you updated your program to use http.NewRequest to create an http.Request value. Once that was created, you used the Do method of Go’s default HTTP client, http.DefaultClient, to make the request and print the http.Response Body to the output.


The HTTP protocol uses more than just GET requests to communicate between programs, though. A GET request is useful when you want to receive information from the other program, but another HTTP method, the POST method, can be used when you want to send information from your program to the server.


# Sending a POST Request


In a REST API, a GET request is only used for retrieving information from the server, so for your program to fully participate in a REST API, your program also needs to support sending POST requests. A POST request is almost the inverse of a GET request, where the client sends data to the server in the request’s body.


In this section, you will update your program to send your request as a POST request instead of a GET request. Your POST request will include a request body, and you will update your server to print out more information about the requests you’re making from the client.


To start making these updates, open your main.go file and add a few new packages that you’ll be using to your import statement:


main.go
```
...

import (
	"bytes"
	"errors"
	"fmt"
	"io/ioutil"
	"net/http"
	"os"
	"strings"
	"time"
)

...

```


Then, update your server handler function to print out various information about the request coming in, such as query string values, header values, and the request body:


main.go
```
...
  mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
	  fmt.Printf("server: %s /\n", r.Method)
	  fmt.Printf("server: query id: %s\n", r.URL.Query().Get("id"))
	  fmt.Printf("server: content-type: %s\n", r.Header.Get("content-type"))
	  fmt.Printf("server: headers:\n")
	  for headerName, headerValue := range r.Header {
		  fmt.Printf("\t%s = %s\n", headerName, strings.Join(headerValue, ", "))
	  }

	  reqBody, err := ioutil.ReadAll(r.Body)
	  if err != nil {
			 fmt.Printf("server: could not read request body: %s\n", err)
	  }
	  fmt.Printf("server: request body: %s\n", reqBody)

	  fmt.Fprintf(w, `{"message": "hello!"}`)
  })
...

```


In this update to the server’s HTTP request handler, you add a few more helpful fmt.Printf statements to see information about the request coming in. You use r.URL.Query().Get to get a query string value named id, and r.Header.Get to get the value of a header called content-type. You also use a for loop with r.Header to print the name and value of each HTTP header the server received. This information can be useful for troubleshooting issues if your client or server isn’t acting the way you expect. Finally, you also used the ioutil.ReadAll function to read the HTTP request’s body in r.Body.


After updating the server handler function, update the main function’s request code so that it’s sending a POST request with a request body:


main.go
```
...
 time.Sleep(100 * time.Millisecond)
	
 jsonBody := []byte(`{"client_message": "hello, server!"}`)
 bodyReader := bytes.NewReader(jsonBody)

 requestURL := fmt.Sprintf("http://localhost:%d?id=1234", serverPort)
 req, err := http.NewRequest(http.MethodPost, requestURL, bodyReader)
...

```


In your update to the main function’s request, one of the new values you’re defining is the jsonBody value. In this example, the value is represented as a []byte instead of the standard string because if you use the encoding/json package to encode JSON data, it will give you a []byte back instead of a string.


The next value, the bodyReader, is a bytes.Reader that wraps the jsonBody data. An http.Request body requires the value to be an io.Reader, and jsonBody’s []byte value doesn’t implement io.Reader, so you wouldn’t be able to use it as a request body on its own. The bytes.Reader value exists to provide that io.Reader interface, so you can use the jsonBody value as the request body.


The requestURL value is also updated to include an id=1234 query string value, primarily to show how a query string value can also be included in the request URL along with other standard URL components.


Finally, the http.NewRequest function call is updated to use a POST method with http.MethodPost, and the request body is included by updating the last parameter from a nil body to bodyReader, the JSON data io.Reader.


Once you’ve saved your changes, you can use go run to run your program:


```
go run main.go


```


The output will be longer than before because of your updates to the server to show additional information:


```
Outputserver: POST /
server: query id: 1234
server: content-type: 
server: headers:
        Accept-Encoding = gzip
        User-Agent = Go-http-client/1.1
        Content-Length = 36
server: request body: {"client_message": "hello, server!"}
client: got response!
client: status code: 200
client: response body: {"message": "hello!"}

```


The first line from the server shows that your request is now coming through as a POST request to the / path. The second line shows the 1234 value of the id query string value you added to the request’s URL. The third line shows the value of the Content-Type header the client sent, which happens to be empty in this request.


The fourth line may be slightly different from the output you see above. In Go, the order of a map value is not guaranteed when you iterate over them using range, so your headers from r.Headers may print out in a different order. Depending on the Go version you’re using, you may also see a different User-Agent version than the one above.


Finally, the last change in the output is that the server is showing the request body it received from the client. The server could then use the encoding/json package to parse the JSON data the client sent and formulate a response.


In this section, you updated your program to send an HTTP POST request instead of a GET request. You also updated your program to send a request body with []byte data being read by a bytes.Reader. Finally, you updated the server handler function to print out more information about the request your HTTP client is making.


Typically in an HTTP request, the client or the server will tell the other the type of content it’s sending in the body. As you saw in the last output, though, your HTTP request didn’t include a Content-Type header to tell the server how to interpret the body’s data. In the next section, you’ll make a few updates to customize your HTTP request, including setting a Content-Type header to let the server know the type of data you’re sending.


# Customizing an HTTP Request


Over time, HTTP requests and responses have been used to send a greater variety of data between clients and servers. At one point, HTTP clients could assume the data they’re receiving from an HTTP server is HTML and have a good chance of being correct. Now, though, it could be HTML, JSON, music, video, or any number of other data types. To provide more information about the data being sent over HTTP, the protocol includes HTTP headers, and one of those important headers is the Content-Type header. This header tells the server (or client, depending on the direction of the data) how to interpret the data it’s receiving.


In this section, you will update your program to set the Content-Type header on your HTTP request so the server knows it’s receiving JSON data. You will also update your program to use an HTTP client other than Go’s default http.DefaultClient so that you can customize how the request is sent.


To make these updates, open your main.go file again and update your main function like so:


main.go
```
...

  req, err := http.NewRequest(http.MethodPost, requestURL, bodyReader)
  if err != nil {
		 fmt.Printf("client: could not create request: %s\n", err)
		 os.Exit(1)
  }
  req.Header.Set("Content-Type", "application/json")

  client := http.Client{
	 Timeout: 30 * time.Second,
  }

  res, err := client.Do(req)
  if err != nil {
	  fmt.Printf("client: error making http request: %s\n", err)
	  os.Exit(1)
  }

...

```


In this update, you access the http.Request headers using req.Header, and then set the value of the Content-Type header on the request to application/json. The application/json media type is defined in the list of media types as the media type for JSON. This way, when the server receives your request, it knows to interpret the body as JSON and not, for example, XML.


The next update is to create your own http.Client instance in the client variable. In this client, you set the Timeout value to 30 seconds. This is important because it says that any requests made with the client will give up and stop trying to receive a response after 30 seconds. Go’s default http.DefaultClient doesn’t specify a timeout, so if you make a request using that client, it will wait until it receives a response, is disconnected by the server, or your program ends. If you have many requests hanging around like this waiting for a response, you could be using a large number of resources on your computer. Setting a Timeout value limits how long a request will wait by the time you define.


Finally, you updated your request to use the Do method of your client variable. You don’t need to make any other changes here because you’ve been calling Do on an http.Client value the whole time. Go’s default HTTP client, http.DefaultClient, is just an http.Client that’s created by default. So, when you called http.Get, the function was calling the Do method for you, and when you updated your request to use http.DefaultClient, you were using that http.Client directly. The only difference now is that you created the http.Client value you’re using this time.


Now, save your file and run your program using go run:


```
go run main.go


```


Your output should be very similar to the previous output but with more information about the content type:


```
Outputserver: POST /
server: query id: 1234
server: content-type: application/json
server: headers:
        Accept-Encoding = gzip
        User-Agent = Go-http-client/1.1
        Content-Length = 36
        Content-Type = application/json
server: request body: {"client_message": "hello, server!"}
client: got response!
client: status code: 200
client: response body: {"message": "hello!"}

```


You’ll see there’s a value from the server for content-type, and there’s a Content-Type header being sent by the client. This is how you could have the same HTTP request path serving both a JSON and an XML API at the same time. By specifying the request’s content type, the server and the client can interpret the data differently.


This example doesn’t trigger the client timeout you configured, though. To see what happens when a request takes too long and the timeout is triggered, open your main.go file and add a time.Sleep function call to your HTTP server handler function.  Then, make the time.Sleep last for longer than the timeout you specified. In this case, you’ll set it for 35 seconds:


main.go
```
...

func main() {
	go func() {
		mux := http.NewServeMux()
		mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
			...	
			fmt.Fprintf(w, `{"message": "hello!"}`)
			time.Sleep(35 * time.Second)
		})
		...
	}()
	...
}

```


Now, save your changes and run your program using go run:


```
go run main.go


```


When you run it this time, it will take longer to exit than before because it won’t exit until after the HTTP request is finished. Since you added the time.Sleep(35 * time.Second), the HTTP request won’t complete until the 30-second timeout is reached:


```
Outputserver: POST /
server: query id: 1234
server: content-type: application/json
server: headers:
        Content-Type = application/json
        Accept-Encoding = gzip
        User-Agent = Go-http-client/1.1
        Content-Length = 36
server: request body: {"client_message": "hello, server!"}
client: error making http request: Post "http://localhost:3333?id=1234": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
exit status 1

```


In this program output, you see the server received the request and processed it, but when it reached the end of the HTTP handler function where your time.Sleep function call is, it started sleeping for 35 seconds. At the same time, the timeout for your HTTP request is counting down and reaches the limit of 30 seconds before the HTTP request finishes. This results in the client.Do method call failing with a context deadline exceeded error because the request’s 30-second deadline passed. Then, your program exits with a failure status code of 1 using os.Exit(1).


In this section, you updated your program to customize an HTTP request by adding a Content-Type header to it. You also updated your program to create a new http.Client with a 30-second timeout, and then used that client to make an HTTP request. You also tested the 30-second timeout by adding a time.Sleep to your HTTP request handler. Finally, you also saw why it’s important to use your own http.Client values with timeouts set if you want to avoid many requests potentially idling forever.


# Conclusion


In this tutorial, you created a new program with an HTTP server and used Go’s net/http package to make HTTP requests to that server. First, you used the http.Get function to make a GET request to the server with Go’s default HTTP client. Then, you used http.NewRequest with http.DefaultClient’s Do method to make a GET request. Next, you updated your request to make it a POST request with a body using bytes.NewReader. Finally, you used the Set method on an http.Request’s Header field to set a request’s Content-Type header, and set a 30-second timeout on a request’s duration by creating your own HTTP client instead of using Go’s default client.


The net/http package includes more than just the functionality you used in this tutorial. It also includes an http.Post function that can be used to make a POST request, similar to the http.Get function. The package also supports saving and retrieving cookies, among other features.


This tutorial is also part of the DigitalOcean How to Code in Go series. The series covers a number of Go topics, from installing Go for the first time to how to use the language itself.


