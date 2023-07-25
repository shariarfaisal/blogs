# How To Use Go with MongoDB Using the MongoDB Go Driver

```Databases``` ```Go``` ```MongoDB```

The author selected the Free Software Foundation to receive a donation as part of the Write for DOnations program.


## Introduction


After relying on community developed solutions for many years, MongoDB announced that they were working on an official driver for Go. In March 2019, this new driver reached a production-ready status with the release of v1.0.0 and has been updated continually since then.


Like the other official MongoDB drivers, the Go driver is idiomatic to the Go programming language and provides an easy way to use MongoDB as the database solution for a Go program. It is fully integrated with the MongoDB API, and exposes all of the query, indexing, and aggregation features of the API, along with other advanced features. Unlike third-party libraries, it will be fully supported by MongoDB engineers so you can be assured of its continued development and maintenance.


In this tutorial you’ll get started with using the official MongoDB Go Driver. You’ll install the driver, connect to a MongoDB database, and perform several CRUD operations. In the process, you’ll create a task manager program for managing tasks through the command line.


# Prerequisites


For this tutorial, you’ll need the following:


- Go installed on your machine and a Go workspace configured following How To Install Go and Set Up a Local Programming Environment. In this tutorial, the project will be named tasker. You’ll need Go v1.11 or higher installed on your machine with Go Modules enabled.

- MongoDB installed for your operating system following How To Install MongoDB. MongoDB 2.6 or higher is the minimum version supported by the MongoDB Go driver.

If you’re using Go v1.11 or 1.12, ensure Go Modules is enabled by setting the GO111MODULE environment variable to on as shown following:


```
export GO111MODULE="on"


```


For more information on implementing environment variables, read this tutorial on How To Read and Set Environmental and Shell Variables.


The commands and code shown in this guide were tested with Go v1.14.1 and MongoDB v3.6.3.


# Step 1 — Installing the MongoDB Go Driver


In this step, you’ll install the Go Driver package for MongoDB and import it into your project. You’ll also connect to your MongoDB database and check the status of the connection.


Go ahead and create a new directory for this tutorial in your filesystem:


```
mkdir tasker


```


Once your project directory is set up, change into it with the following command:


```
cd tasker


```


Next, initialize the Go project with a go.mod file. This file defines project requirements and locks dependencies to their correct versions:


```
go mod init


```


If your project directory is outside the $GOPATH, you need to specify the import path for your module as follows:


```
go mod init github.com/<your_username>/tasker


```


At this point, your go.mod file will look like this:


go.mod
```
module github.com/<your_username>/tasker

go 1.14

```


Add the MongoDB Go Driver as a dependency for your project using following command:


```
go get go.mongodb.org/mongo-driver


```


You’ll see output like the following:


```
Outputgo: downloading go.mongodb.org/mongo-driver v1.3.2
go: go.mongodb.org/mongo-driver upgrade => v1.3.2

```


At this point, your go.mod file will look like this:


go.mod
```
module github.com/<your_username>/tasker

go 1.14

require go.mongodb.org/mongo-driver v1.3.1 // indirect

```


Next, create a main.go file in your project root and open it in your text editor:


```
nano main.go


```


To get started with the driver, import the following packages into your main.go file:


main.go
```
package main

import (
	"context"
	"log"

	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

```


Here you add the mongo and options packages, which the MongoDB Go driver provides.


Next, following your imports, create a new MongoDB client and connect to your running MongoDB server:


main.go
```
. . .
var collection *mongo.Collection
var ctx = context.TODO()

func init() {
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017/")
	client, err := mongo.Connect(ctx, clientOptions)
	if err != nil {
		log.Fatal(err)
	}
}

```


mongo.Connect() accepts a Context and a options.ClientOptions object, which is used to set the connection string and other driver settings. You can visit the options package documentation to see what configuration options are available.


Context is like a timeout or deadline that indicates when an operation should stop running and return. It helps to prevent performance degradation on production systems when specific operations are running slow. In this code, you’re passing context.TODO() to indicate that you’re not sure what context to use right now, but you plan to add one in the future.


Next, let’s ensure that your MongoDB server was found and connected to successfully using the Ping method. Add the following code inside the init function:


main.go
```
. . .
    log.Fatal(err)
  }

  err = client.Ping(ctx, nil)
  if err != nil {
    log.Fatal(err)
  }
}

```


If there are any errors while connecting to the database, the program should crash while you try to fix the problem as there’s no point keeping the program running without an active database connection.


Add the following code to create a database:


main.go
```
. . .
  err = client.Ping(ctx, nil)
  if err != nil {
    log.Fatal(err)
  }

  collection = client.Database("tasker").Collection("tasks")
}

```


You create a tasker database and a task collection to store the tasks you’ll be creating. You also set up collection as a package-level variable so you can reuse the database connection throughout the package.


Save and exit the file.


The full main.go at this point is as follows:


main.go
```
package main

import (
	"context"
	"log"

	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

var collection *mongo.Collection
var ctx = context.TODO()

func init() {
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017/")
	client, err := mongo.Connect(ctx, clientOptions)
	if err != nil {
		log.Fatal(err)
	}

	err = client.Ping(ctx, nil)
	if err != nil {
		log.Fatal(err)
	}

	collection = client.Database("tasker").Collection("tasks")
}

```


You’ve set up your program to connect to your MongoDB server using the Go driver. In the next step, you’ll proceed with the creation of your task manager program.


# Step 2 — Creating a CLI Program


In this step, you’ll install the well-known cli package to aid with the development of your task manager program. It provides an interface that you can take advantage of to rapidly create modern command line tools. For example, this package gives the ability to define subcommands for your program for a more git-like command line experience.


Run the following command to add the package as a dependency:


```
go get github.com/urfave/cli/v2


```


Next, open up your main.go file again:


```
nano main.go


```


Add the following highlighted code to your main.go file:


main.go
```
package main

import (
	"context"
	"log"
	"os"

	"github.com/urfave/cli/v2"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)
. . .

```


You import the cli package as mentioned. You also import the os package, which you’ll use to pass command line arguments to your program:


Add the following code after your init function to create your CLI program and cause your code to compile:


main.go
```
. . .
func main() {
	app := &cli.App{
		Name:     "tasker",
		Usage:    "A simple CLI program to manage your tasks",
		Commands: []*cli.Command{},
	}

	err := app.Run(os.Args)
	if err != nil {
		log.Fatal(err)
	}
}

```


This snippet creates a CLI program called tasker and adds a short usage description that will be printed out when you run the program. The Commands slice is where you’ll add commands for your program. The Run command parses the arguments slice to the appropriate command.


Save and exit your file.


Here’s the command you need to build and run the program:


```
go run main.go


```


You’ll see the following output:


```
OutputNAME:
   tasker - A simple CLI program to manage your tasks

USAGE:
   main [global options] command [command options] [arguments...]

COMMANDS:
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help (default: false)

```


The program runs and shows help text, which is handy for learning about what the program can do, and how to use it.


In the next steps, you’ll improve the utility of your program by adding subcommands to help manage your tasks in MongoDB.


# Step 3 — Creating a Task


In this step, you’ll add a subcommand to your CLI program using the cli package. At the end of this section, you’ll be able to add a new task to your MongoDB database by using a new add command in your CLI program.


Begin by opening up your main.go file:


```
nano main.go


```


Next, import the go.mongodb.org/mongo-driver/bson/primitive, time, and errors packages:


main.go
```
package main

import (
	"context"
	"errors"
	"log"
	"os"
	"time"

	"github.com/urfave/cli/v2"
	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)
. . .

```


Then create a new struct to represent a single task in the database and insert it immediately preceding the main function:


main.go
```
. . .
type Task struct {
	ID        primitive.ObjectID `bson:"_id"`
	CreatedAt time.Time          `bson:"created_at"`
	UpdatedAt time.Time          `bson:"updated_at"`
	Text      string             `bson:"text"`
	Completed bool               `bson:"completed"`
}
. . .

```


You use the primitive package to set the type of the ID of each task since MongoDB uses ObjectIDs for the _id field by default. Another default behavior of MongoDB is that the lowercase field name is used as the key for each exported field when it is being serialized, but this can be changed using bson struct tags.


Next, create a function that receives an instance of Task and saves it in the database. Add this snippet following the main function:


main.go
```
. . .
func createTask(task *Task) error {
	_, err := collection.InsertOne(ctx, task)
  return err
}
. . .

```


The collection.InsertOne() method inserts the provided task in the database collection and returns the ID of the document that was inserted. Since you don’t need this ID, you discard it by assigning to the underscore operator.


The next step is to add a new command to your task manager program for creating new tasks. Let’s call it add:


main.go
```
. . .
func main() {
	app := &cli.App{
		Name:  "tasker",
		Usage: "A simple CLI program to manage your tasks",
		Commands: []*cli.Command{
			{
				Name:    "add",
				Aliases: []string{"a"},
				Usage:   "add a task to the list",
				Action: func(c *cli.Context) error {
					str := c.Args().First()
					if str == "" {
						return errors.New("Cannot add an empty task")
					}

					task := &Task{
						ID:        primitive.NewObjectID(),
						CreatedAt: time.Now(),
						UpdatedAt: time.Now(),
						Text:      str,
						Completed: false,
					}

					return createTask(task)
				},
			},
		},
	}

	err := app.Run(os.Args)
	if err != nil {
		log.Fatal(err)
	}
}

```


Every new command that is added to your CLI program is placed inside the Commands slice. Each one consists of a name, usage description, and action. This is the code that will run upon command execution.


In this code, you collect the first argument to add and use it to set the Text property of a new Task instance while assigning the appropriate defaults for the other properties. The new task is subsequently passed on to createTask, which inserts the task into the database and returns nil if all goes well causing the command to exit.


Save and exit your file.


Test it out by adding a few tasks using the add command. If successful, you’ll see no errors printed to your screen:


```
go run main.go add "Learn Go"
go run main.go add "Read a book"


```


Now that you can add tasks successfully, let’s implement a way to display all the tasks that you’ve added to the database.


# Step 4 — Listing all Tasks


Listing the documents in a collection can be done using the collection.Find() method, which expects a filter as well as a pointer to a value into which the result can be decoded. Its return value is a Cursor, which provides a stream of documents that can be iterated over and decoded one at a time. The Cursor is then closed once it has been exhausted.


Open your main.go file:


```
nano main.go


```


Make sure to import the bson package:


main.go
```
package main

import (
	"context"
	"errors"
	"log"
	"os"
	"time"

	"github.com/urfave/cli/v2"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)
. . .

```


Then create the following functions immediately after createTask:


main.go
```
. . .
func getAll() ([]*Task, error) {
  // passing bson.D{{}} matches all documents in the collection
	filter := bson.D{{}}
	return filterTasks(filter)
}

func filterTasks(filter interface{}) ([]*Task, error) {
	// A slice of tasks for storing the decoded documents
	var tasks []*Task

	cur, err := collection.Find(ctx, filter)
	if err != nil {
		return tasks, err
	}

	for cur.Next(ctx) {
		var t Task
		err := cur.Decode(&t)
		if err != nil {
			return tasks, err
		}

		tasks = append(tasks, &t)
	}

	if err := cur.Err(); err != nil {
		return tasks, err
	}

  // once exhausted, close the cursor
	cur.Close(ctx)

	if len(tasks) == 0 {
		return tasks, mongo.ErrNoDocuments
	}

	return tasks, nil
}

```


BSON (Binary-encoded JSON) is how documents are represented in a MongoDB database and the bson package is what helps us work with BSON objects in Go. The bson.D type used in the getAll() function represents a BSON document and it’s used where the order of the properties matter. By passing bson.D{{}} as your filter to filterTasks(), you’re indicating that you want to match all the documents in the collection.


In the filterTasks() function, you iterate over the Cursor returned by the collection.Find() method and decode each document into an instance of Task. Each Task is then appended to the slice of tasks created at the start of the function. Once the Cursor is exhausted, it is closed and the tasks slice is returned.


Before you create a command for listing all tasks, let’s create a helper function that takes a slice of tasks and prints to the standard output. You’ll be using the color package to colorize the output.


Before you can use the this package, install it with:


```
go get gopkg.in/gookit/color.v1


```


You’ll see the following output:


```
Outputgo: downloading gopkg.in/gookit/color.v1 v1.1.6
go: gopkg.in/gookit/color.v1 upgrade => v1.1.6

```


And import it into your main.go file along with the fmt package:


main.go
```
package main

import (
	"context"
	"errors"
  "fmt"
	"log"
	"os"
	"time"

	"github.com/urfave/cli/v2"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"gopkg.in/gookit/color.v1"
)
. . .

```


Next, create a new printTasks function following your main function:


main.go
```
. . .
func printTasks(tasks []*Task) {
	for i, v := range tasks {
		if v.Completed {
			color.Green.Printf("%d: %s\n", i+1, v.Text)
		} else {
			color.Yellow.Printf("%d: %s\n", i+1, v.Text)
		}
	}
}
. . .

```


This printTasks function takes a slice of tasks, iterates over each one, and prints it out to the standard output using the green color to indicate completed tasks, and yellow for incomplete tasks.


Go ahead and add the following highlighted lines to create a new all command to the Commands slice. This command will print all added tasks to the standard output:


main.go
```
. . .
func main() {
	app := &cli.App{
		Name:  "tasker",
		Usage: "A simple CLI program to manage your tasks",
		Commands: []*cli.Command{
			{
				Name:    "add",
				Aliases: []string{"a"},
				Usage:   "add a task to the list",
				Action: func(c *cli.Context) error {
					str := c.Args().First()
					if str == "" {
						return errors.New("Cannot add an empty task")
					}

					task := &Task{
						ID:        primitive.NewObjectID(),
						CreatedAt: time.Now(),
						UpdatedAt: time.Now(),
						Text:      str,
						Completed: false,
					}

					return createTask(task)
				},
			},
			{
				Name:    "all",
				Aliases: []string{"l"},
				Usage:   "list all tasks",
				Action: func(c *cli.Context) error {
					tasks, err := getAll()
					if err != nil {
						if err == mongo.ErrNoDocuments {
							fmt.Print("Nothing to see here.\nRun `add 'task'` to add a task")
							return nil
						}

						return err
					}

					printTasks(tasks)
					return nil
				},
			},
		},
	}

	err := app.Run(os.Args)
	if err != nil {
		log.Fatal(err)
	}
}

. . .

```


The all command retrieves all the tasks present in the database and prints them to the standard output. If no tasks are present, a prompt to add a new task is printed instead.


Save and exit your file.


Build and run your program with the all command:


```
go run main.go all


```


It will list all the tasks that you’ve added so far:


```
Output1: Learn Go
2: Read a book

```


Now that you can view all the tasks in the database, let’s add the ability to mark a task as completed in the next step.


# Step 5 — Completing a Task


In this step, you’ll create a new subcommand called done that will allow you to mark an existing task in the database as completed. To mark a task as completed, you can use the collection.FindOneAndUpdate() method. It allows you to locate a document in a collection and update some or all of its properties. This method requires a filter to locate the document and an update document to describe the operation. Both of these are built using bson.D types.


Start by opening up your main.go file:


```
nano main.go


```


Insert the following snippet following your filterTasks function:


main.go
```
. . .
func completeTask(text string) error {
	filter := bson.D{primitive.E{Key: "text", Value: text}}

	update := bson.D{primitive.E{Key: "$set", Value: bson.D{
		primitive.E{Key: "completed", Value: true},
	}}}

	t := &Task{}
	return collection.FindOneAndUpdate(ctx, filter, update).Decode(t)
}
. . .

```


The function matches the first document where the text property is equal to the text parameter. The update document specifies that the completed property be set to true. If there’s an error in the FindOneAndUpdate() operation, it will be returned by completeTask(). Otherwise nil is returned.


Next, let’s add a new done command to your CLI program that marks a task as completed:


main.go
```
. . .
func main() {
	app := &cli.App{
		Name:  "tasker",
		Usage: "A simple CLI program to manage your tasks",
		Commands: []*cli.Command{
			{
				Name:    "add",
				Aliases: []string{"a"},
				Usage:   "add a task to the list",
				Action: func(c *cli.Context) error {
					str := c.Args().First()
					if str == "" {
						return errors.New("Cannot add an empty task")
					}

					task := &Task{
						ID:        primitive.NewObjectID(),
						CreatedAt: time.Now(),
						UpdatedAt: time.Now(),
						Text:      str,
						Completed: false,
					}

					return createTask(task)
				},
			},
			{
				Name:    "all",
				Aliases: []string{"l"},
				Usage:   "list all tasks",
				Action: func(c *cli.Context) error {
					tasks, err := getAll()
					if err != nil {
						if err == mongo.ErrNoDocuments {
							fmt.Print("Nothing to see here.\nRun `add 'task'` to add a task")
							return nil
						}

						return err
					}

					printTasks(tasks)
					return nil
				},
			},
			{
				Name:    "done",
				Aliases: []string{"d"},
				Usage:   "complete a task on the list",
				Action: func(c *cli.Context) error {
					text := c.Args().First()
					return completeTask(text)
				},
			},
		},
	}

	err := app.Run(os.Args)
	if err != nil {
		log.Fatal(err)
	}
}

. . .

```


You use the argument passed to the done command to find the first document whose text property matches. If found, the completed property on the document is set to true.


Save and exit your file.


Then run your program with the done command:


```
go run main.go done "Learn Go"


```


If you use the all command again, you will notice that the task that was marked as completed is now printed with green.


```
go run main.go all


```





Sometimes, you only want to view tasks that have not yet been done. We’ll add that feature next.


# Step 6 — Displaying Pending Tasks Only


In this step, you’ll incorporate code to retrieve pending tasks from the database using the MongoDB driver. Pending tasks are those whose completed property is set to false.


Let’s add a new function that retrieves tasks that have not been completed yet. Open your main.go file:


```
nano main.go


```


Then add this snippet following the completeTask function:


main.go
```
. . .
func getPending() ([]*Task, error) {
	filter := bson.D{
		primitive.E{Key: "completed", Value: false},
	}

	return filterTasks(filter)
}
. . .

```


You create a filter using the bson and primitive packages from the MongoDB driver, which will match documents whose completed property is set to false. The slice of pending tasks is then returned to the caller.


Instead of creating a new command to list pending tasks, let’s make it the default action when running the program without any commands. You can do this by adding an Action property to the program as follows:


main.go
```
. . .
func main() {
	app := &cli.App{
		Name:  "tasker",
		Usage: "A simple CLI program to manage your tasks",
		Action: func(c *cli.Context) error {
			tasks, err := getPending()
			if err != nil {
				if err == mongo.ErrNoDocuments {
					fmt.Print("Nothing to see here.\nRun `add 'task'` to add a task")
					return nil
				}

				return err
			}

			printTasks(tasks)
			return nil
		},
		Commands: []*cli.Command{
			{
				Name:    "add",
				Aliases: []string{"a"},
				Usage:   "add a task to the list",
				Action: func(c *cli.Context) error {
					str := c.Args().First()
					if str == "" {
						return errors.New("Cannot add an empty task")
					}

					task := &Task{
						ID:        primitive.NewObjectID(),
						CreatedAt: time.Now(),
						UpdatedAt: time.Now(),
						Text:      str,
						Completed: false,
					}

					return createTask(task)
				},
			},
. . .

```


The Action property performs a default action when the program is executed without any subcommands. This is where logic for listing pending tasks is placed. The getPending() function is called and the resulting tasks are printed to the standard output using printTasks(). If there are no pending tasks, a prompt is displayed instead, encouraging the user to add a new task using the add command.


Save and exit your file.


Running the program now without adding any commands will list all pending tasks in the database:


```
go run main.go


```


You’ll see the following output:


```
Output1: Read a book

```


Now that you can list incomplete tasks, let’s add another command that allows you to view completed tasks only.


# Step 7 — Displaying Finished Tasks


In this step, you’ll add a new finished subcommand that fetches completed tasks from the database and displays them on the screen. This involves filtering and returning tasks whose completed property is set to true.


Open your main.go file:


```
nano main.go


```


Then add in the following code at the end of your file:


main.go
```
. . .
func getFinished() ([]*Task, error) {
	filter := bson.D{
		primitive.E{Key: "completed", Value: true},
	}

	return filterTasks(filter)
}
. . .

```


Similar to the getPending() function, you’ve added a getFinished() function that returns a slice of completed tasks. In this case, the filter has the completed property set to true so only the documents that match this condition will be returned.


Next, create a finished command that prints all completed tasks:


main.go
```
. . .
func main() {
	app := &cli.App{
		Name:  "tasker",
		Usage: "A simple CLI program to manage your tasks",
		Action: func(c *cli.Context) error {
			tasks, err := getPending()
			if err != nil {
				if err == mongo.ErrNoDocuments {
					fmt.Print("Nothing to see here.\nRun `add 'task'` to add a task")
					return nil
				}

				return err
			}

			printTasks(tasks)
			return nil
		},
		Commands: []*cli.Command{
			{
				Name:    "add",
				Aliases: []string{"a"},
				Usage:   "add a task to the list",
				Action: func(c *cli.Context) error {
					str := c.Args().First()
					if str == "" {
						return errors.New("Cannot add an empty task")
					}

					task := &Task{
						ID:        primitive.NewObjectID(),
						CreatedAt: time.Now(),
						UpdatedAt: time.Now(),
						Text:      str,
						Completed: false,
					}

					return createTask(task)
				},
			},
			{
				Name:    "all",
				Aliases: []string{"l"},
				Usage:   "list all tasks",
				Action: func(c *cli.Context) error {
					tasks, err := getAll()
					if err != nil {
						if err == mongo.ErrNoDocuments {
							fmt.Print("Nothing to see here.\nRun `add 'task'` to add a task")
							return nil
						}

						return err
					}

					printTasks(tasks)
					return nil
				},
			},
			{
				Name:    "done",
				Aliases: []string{"d"},
				Usage:   "complete a task on the list",
				Action: func(c *cli.Context) error {
					text := c.Args().First()
					return completeTask(text)
				},
			},
			{
				Name:    "finished",
				Aliases: []string{"f"},
				Usage:   "list completed tasks",
				Action: func(c *cli.Context) error {
					tasks, err := getFinished()
					if err != nil {
						if err == mongo.ErrNoDocuments {
							fmt.Print("Nothing to see here.\nRun `done 'task'` to complete a task")
							return nil
						}

						return err
					}

					printTasks(tasks)
					return nil
				},
			},
		}
	}

	err := app.Run(os.Args)
	if err != nil {
		log.Fatal(err)
	}
}
. . .

```


The finished command retrieves tasks whose completed property is set to true via the getFinished() function created here. It then passes it to the printTasks function so that they are printed to the standard output.


Save and exit your file.


Run the following command:


```
go run main.go finished


```


You’ll see the following output:


```
Output1: Learn Go

```


In the final step, you’ll give users the option to delete tasks from the database.


# Step 8 — Deleting a Task


In this step, you’ll add a new delete subcommand to allow users to delete a task from the database. To delete a single task, you’ll use the collection.DeleteOne() method from the MongoDB driver. It also relies on a filter to match the document to delete.


Open your main.go file once more:


```
nano main.go


```


Add this deleteTask function to delete tasks from the database straight after your getFinished function:


main.go
```
. . .
func deleteTask(text string) error {
	filter := bson.D{primitive.E{Key: "text", Value: text}}

	res, err := collection.DeleteOne(ctx, filter)
	if err != nil {
		return err
	}

	if res.DeletedCount == 0 {
		return errors.New("No tasks were deleted")
	}

	return nil
}
. . .

```


This deleteTask method takes a string argument that represents the task item to be deleted. A filter is constructed to match the task item whose text property is set to the string argument. You pass the filter to the DeleteOne() method that matches the item in the collection and deletes it.


You can check the DeletedCount property on the result from the DeleteOne method to confirm if any documents were deleted. If the filter is unable to match a document to be deleted, the DeletedCount will be zero and you can return an error in that case.


Now add a new rm command as highlighted:


main.go
```
. . .
func main() {
	app := &cli.App{
		Name:  "tasker",
		Usage: "A simple CLI program to manage your tasks",
		Action: func(c *cli.Context) error {
			tasks, err := getPending()
			if err != nil {
				if err == mongo.ErrNoDocuments {
					fmt.Print("Nothing to see here.\nRun `add 'task'` to add a task")
					return nil
				}

				return err
			}

			printTasks(tasks)
			return nil
		},
		Commands: []*cli.Command{
			{
				Name:    "add",
				Aliases: []string{"a"},
				Usage:   "add a task to the list",
				Action: func(c *cli.Context) error {
					str := c.Args().First()
					if str == "" {
						return errors.New("Cannot add an empty task")
					}

					task := &Task{
						ID:        primitive.NewObjectID(),
						CreatedAt: time.Now(),
						UpdatedAt: time.Now(),
						Text:      str,
						Completed: false,
					}

					return createTask(task)
				},
			},
			{
				Name:    "all",
				Aliases: []string{"l"},
				Usage:   "list all tasks",
				Action: func(c *cli.Context) error {
					tasks, err := getAll()
					if err != nil {
						if err == mongo.ErrNoDocuments {
							fmt.Print("Nothing to see here.\nRun `add 'task'` to add a task")
							return nil
						}

						return err
					}

					printTasks(tasks)
					return nil
				},
			},
			{
				Name:    "done",
				Aliases: []string{"d"},
				Usage:   "complete a task on the list",
				Action: func(c *cli.Context) error {
					text := c.Args().First()
					return completeTask(text)
				},
			},
			{
				Name:    "finished",
				Aliases: []string{"f"},
				Usage:   "list completed tasks",
				Action: func(c *cli.Context) error {
					tasks, err := getFinished()
					if err != nil {
						if err == mongo.ErrNoDocuments {
							fmt.Print("Nothing to see here.\nRun `done 'task'` to complete a task")
							return nil
						}

						return err
					}

					printTasks(tasks)
					return nil
				},
			},
			{
				Name:  "rm",
				Usage: "deletes a task on the list",
				Action: func(c *cli.Context) error {
					text := c.Args().First()
					err := deleteTask(text)
					if err != nil {
						return err
					}

					return nil
				},
			},
		}
	}

	err := app.Run(os.Args)
	if err != nil {
		log.Fatal(err)
	}
}
. . .

```


Just as with all the other subcommands added previously, the rm command uses its first argument to match a task in the database and deletes it.


Save and exit your file.


You can list pending tasks by running your program without passing any subcommands:


```
go run main.go


```


```
Output1: Read a book

```


Running the rm subcommand on the "Read a book" task will delete it from the database:


```
go run main.go rm "Read a book"


```


If you list all pending tasks again, you’ll notice that the "Read a book" task does not appear anymore and a prompt to add a new task is shown instead:


```
go run main.go


```


```
OutputNothing to see here
Run `add 'task'` to add a task

```


In this step you added a function to delete tasks from the database.


# Conclusion


You’ve successfully created a task manager command line program and learned the fundamentals of using the MongoDB Go driver in the process.


Be sure to check out the full documentation for the MongoDB Go Driver at GoDoc to learn more about the features that using the driver provides. The documentation that describes using aggregations or transactions may be of particular interest to you.


The final code for this tutorial can be viewed in this GitHub repo.


