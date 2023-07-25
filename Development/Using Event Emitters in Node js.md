# Using Event Emitters in Node js

```JavaScript``` ```Node.js``` ```Development```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Event emitters are objects in Node.js that trigger an event by sending a message to signal that an action was completed. JavaScript developers can write code that listens to events from an event emitter, allowing them to execute functions every time those events are triggered. In this context, events are composed of an identifying string and any data that needs to be passed to the listeners.


Typically in Node.js, when we want to have an action occur upon completion of another action, we use asynchronous programming techniques like nesting callbacks or chaining promises. However, these techniques tightly couple the triggering action and the resulting action, making it difficult to modify the resulting action in the future. Event emitters provide a different way to structure this relationship: the publish-subscribe pattern. In this software architecture pattern, a publisher (the event emitter) sends a message (an event), and a subscriber receives the event and performs an action. The power of this pattern is that the publisher does not need to know about the subscribers. A publisher publishes a message, and it’s up to the subscribers to react to it in their respective ways. If we wanted to change the behavior of our application, we could adjust how the subscribers react to the events without having to change the publisher.


In this article, we will create an event listener for a TicketManager JavaScript class that allows a user to buy tickets. We will set up listeners for the buy event, which will trigger every time a ticket is bought. This process will also show how to manage erroneous events from the emitter and how to manage event subscribers.


# Prerequisites


- 
Node.js installed on your development machine. This tutorial uses version 10.20.1. To install this on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04.

- 
The main example in this article makes use of JavaScript classes as they were introduced in ES2015 (commonly referred to as ES6). If you’d like to learn about classes in JavaScript, read our Understanding Classes in JavaScript tutorial.


# Step 1 — Emitting Events


In this step, we’ll explore the two most common ways to create an event emitter in Node.js. The first is to use an event emitter object directly, and the second is to create an object that extends the event emitter object.


Deciding which one to use depends on how coupled your events are to the actions of your objects. If the events you want to emit are an effect of an object’s actions, you would likely extend from the event emitter object to have access to its functions for convenience. If the events you want to emit are independent of your business objects or are a result of actions from many business objects, you would instead create an independent event emitter object that’s referenced by your objects.


Let’s begin by creating a standalone, event-emitting object. We’ll begin by creating a folder to store all of our code. In your terminal, make a new folder called event-emitters:


```
mkdir event-emitters


```


Then enter that folder:


```
cd event-emitters


```


Open the first event emitter, firstEventEmitter.js, in a text editor. We will use nano as it’s available in the terminal:


```
nano firstEventEmitter.js


```


In Node.js, we emit events via the EventEmitter class. This class is part of the events module. Let’s begin by first loading the events module in our file by adding the following line:


event-emitters/firstEventEmitter.js
```
const { EventEmitter } = require("events");

```


With the class imported, we can use it to create a new object instance from it:


event-emitters/firstEventEmitter.js
```
const { EventEmitter } = require("events");

const firstEmitter = new EventEmitter();

```


Let’s emit an event by adding the following highlighted line at the end of firstEventEmitter.js:


event-emitters/firstEventEmitter.js
```
const { EventEmitter } = require("events");

const firstEmitter = new EventEmitter();


firstEmitter.emit("My first event");

```


The emit() function is used to fire events. We need to pass the name of the event to it as a string. We can add any number of arguments after the event name. Events with just a name are fairly limited; the other arguments allow us to send data to our listeners. When we set up our ticket manager, our events will pass data about the purchase when it happens. Keep the name of the event in mind, because event listeners will identify it by this name.



Note: While we don’t capture it in this example, the emit() function returns true if there are listeners for the event. If there are no listeners for an event, it returns false.

Let’s run this file to see what happens. Save and exit nano, then execute the file with the node command:


```
node firstEventEmitter.js


```


When the script finishes its execution, you will see no output in the terminal. That’s because we do not log any messages in firstEventEmitter.js and there’s nothing that listens to the event that was sent. The event is emitted, but nothing acts on these events.


Let’s work toward seeing a more complete example of publishing, listening to, and acting upon events. We’ll do this by creating a ticket manager example application. The ticket manager will expose a function to buy tickets. When a ticket is bought, an event will be sent with details of the purchaser. Later, we’ll create another Node.js module to simulate an email being sent to the purchaser’s email confirming the purchase.


Let’s begin by creating our ticket manager. It will extend the EventEmitter class so that we don’t have to create a separate event emitter object to emit events.


In the same working directory, create and open a new file called ticketManager.js:


```
nano ticketManager.js


```


As with the first event emitter, we need to import the EventEmitter class from the events module. Put the following code at the beginning of the file:


event-emitters/ticketManager.js
```
const EventEmitter = require("events");

```


Now, make a new TicketManager class that will soon define the method for ticket purchases:


event-emitters/ticketManager.js
```
const EventEmitter = require("events");

class TicketManager extends EventEmitter {}

```


In this case, the TicketManager class extends the EventEmitter class. This means that the TicketManager class inherits the methods and properties of the EventEmitter class. This is how it gets access to the emit() method.


In our ticket manager, we want to provide the initial supply of tickets that can be purchased. We’ll do this by accepting the initial supply in our constructor(), a special function that’s called when a new object of a class is made. Add the following constructor to the TicketManager class:


event-emitters/ticketManager.js
```
const EventEmitter = require("events");

class TicketManager extends EventEmitter {
    constructor(supply) {
        super();
        this.supply = supply;
    }
}

```


The constructor has one supply argument. This is a number detailing the initial supply of tickets we can sell. Even though we declared that TicketManager is a child class of EventEmitter, we still need to call super() to get access to the methods and properties of EventEmitter. The super() function calls the constructor of the parent function, which in this case is EventEmitter.


Finally, we create a supply property for the object with this.supply and give it the value passed in by the constructor.


Now, let’s add a buy() method that will be called when a ticket is purchased. This method will decrease the supply by one and emit an event with the purchase data.


Add the buy() method as follows:


event-emitters/ticketManager.js
```
const EventEmitter = require("events");

class TicketManager extends EventEmitter {
    constructor(supply) {
        super();
        this.supply = supply;
    }
    
    buy(email, price) {
        this.supply--;
        this.emit("buy", email, price, Date.now());
    }
}

```


In the buy() function, we take the purchaser’s email address and the price they paid for the ticket. We then decrease the supply of tickets by one. We end by emitting a buy event. This time, we emit an event with extra data: the email and price that were passed in the function as well as a timestamp of when the purchase was made.


So that our other Node.js modules can use this class, we need to export it. Add this line at the end of the file:


event-emitters/ticketManager.js
```
...

module.exports = TicketManager

```


Save and exit the file.


We’ve finished our setup for the event emitter TicketManager. Now that we’ve put things in place to push events, we can move on to reading and processing those events. To do that, we will create event listeners in the next step.


# Step 2 — Listening for Events


Node.js allows us to add a listener for an event with the on() function of an event emitter object. This listens for a particular event name and fires a callback when the event is triggered. Adding a listener typically looks like this:


```
eventEmitter.on(event_name, callback_function) {
    action
}

```



Note:: Node.js aliases the on() method with addListener(). They perform the same task. In this tutorial, we will continue to use on().

Let’s get some first-hand experience with listening to our first event. Create a new file called firstListener.js:


```
nano firstListener.js


```


As a demonstration of how the on() function works, let’s log a simple message upon receiving our first event.


First, let’s import TicketManager into our new Node.js module. Add the following code into firstListener.js:


event-emitters/firstListener.js
```
const TicketManager = require("./ticketManager");

const ticketManager = new TicketManager(10);

```


Recall that TicketManager objects need their initial supply of tickets when created. This is why we pass the 10 argument.


Now let’s add our first Node.js event listener. It will listen to the buy event. Add the following highlighted code:


event-emitters/firstListener.js
```
const TicketManager = require("./ticketManager");

const ticketManager = new TicketManager(10);

ticketManager.on("buy", () => {
    console.log("Someone bought a ticket!");
});

```


To add a new listener, we used the on() function that’s a part of the ticketManager object. The on() method is available to all event emitter objects, and since TicketManager inherits from the EventEmitter class, this method is available on all of the TicketManager instance objects.


The second argument to the on() method is a callback function, written as an arrow function. The code in this function is run after the event is emitted. In this case, we log "Someone bought a ticket!" to the console if a buy event is emitted.


Now that we set up a listener, let’s use the buy() function so that the event will be emitted. At the end of your file add this:


event-emitters/firstListener.js
```
...

ticketManager.buy("test@email.com", 20);

```


This performs the buy method with a user email of test@email.com and a ticket price of 20.


Save and exit the file.


Now run this script with node:


```
node firstListener.js


```


Your console will display this:


```
OutputSomeone bought a ticket!

```


Your first event listener worked. Let’s see what happens if we buy multiple tickets. Re-open your firstListener.js in your text editor:


```
nano firstListener.js


```


At the end of the file, make another call to the buy() function:


event-emitters/firstListener.js
```
...

ticketManager.buy("test@email.com", 20);
ticketManager.buy("test@email.com", 20);

```


Save and exit the file. Let’s run the script with Node.js to see what happens:


```
node firstListener.js


```


Your console will display this:


```
OutputSomeone bought a ticket!
Someone bought a ticket!

```


Since the buy() function was called twice, two buy events were emitted. Our listener picked up both.


Sometimes we’re only interested in listening to the first time an event was fired, as opposed to all the times it’s emitted. Node.js provides an alternative to on() for this case with the once() function.


Like on(), the once() function accepts the event name as its first argument, and a callback function that’s called when the event is fired as its second argument. Under the hood, when the event is emitted and received by a listener that uses once(), Node.js automatically removes the listener and then executes the code in the callback function.


Let’s see once() in action by editing firstListener.js:


```
nano firstListener.js


```


At the end of the file, add a new event listener using once() like the following highlighted lines:


event-emitters/firstListener.js
```
const TicketManager = require("./ticketManager");

const ticketManager = new TicketManager(10);

ticketManager.on("buy", () => {
        console.log("Someone bought a ticket!");
});

ticketManager.buy("test@email.com", 20);
ticketManager.buy("test@email.com", 20);

ticketManager.once("buy", () => {
    console.log("This is only called once");
});

```


Save and exit the file and run this program with node:


```
node firstListener.js


```


The output is the same as the last time:


```
OutputSomeone bought a ticket!
Someone bought a ticket!

```


While we added a new event listener with once(), it was added after the buy events were emitted. Because of this, the listener didn’t detect these two events. You can’t listen for events that already happened in the past. When you add a listener you can only capture events that come after.


Let’s add a couple more buy() function calls so we can confirm that the once() listener only reacts one time. Open firstListener.js in your text editor like before:


```
nano firstListener.js


```


Add the following calls at the end of the file:


event-emitters/firstListener.js
```
...

ticketManager.once("buy", () => {
    console.log("This is only called once");
});

ticketManager.buy("test@email.com", 20);
ticketManager.buy("test@email.com", 20);

```


Save and exit, then execute this program:


```
node firstListener.js


```


Your output will be:


```
OutputSomeone bought a ticket!
Someone bought a ticket!
Someone bought a ticket!
This is only called once
Someone bought a ticket!

```


The first two lines were from the first two buy() calls before the once() listener was added. Adding a new event listener does not remove previous ones, so the first event listener we added is still active and logs messages.


Since the event listener with on() was declared before the event listener with once(), we see Someone bought a ticket! before This is only called once. These two lines are both responding to the second-to-last buy event.


Finally, when the last call to buy() was made, the event emitter only had the first listener that was created with on(). As mentioned earlier, when an event listener created with once() receives an event, it is automatically removed.


Now that we have added event listeners to detect our emitters, we will see how to capture data with those listeners.


# Step 3 — Capturing Event Data


So far, you’ve set up event listeners to react to emitted events. The emitted events also pass along data. Let’s see how we can capture the data that accompanies an event.


We’ll begin by creating some new Node.js modules: an email service and a database service. They’ll be used to simulate sending an email and saving to a database respectively. We’ll then tie them all together with our main Node.js script—index.js.


Let’s begin by editing our email service module. Open the file in your text editor:


```
nano emailService.js


```


Our email service consists of a class that contains one method—send(). This method expects the email that’s emitted along with buy events. Add the following code to your file:


event-emitters/emailService.js
```
class EmailService {
    send(email) {
        console.log(`Sending email to ${email}`);
    }
}

module.exports = EmailService

```


This code creates an EmailService class that contains a send() function. In lieu of sending an actual email, it uses template literals to log a message to the console that would contain the email address of someone buying a ticket. Save and exit before moving on.


Let’s set up the database service. Open databaseService.js with your text editor:


```
nano databaseService.js


```


The database service saves our purchase data to a database via its save() method. Add the following code to databaseService.js:


event-emitters/databaseService.js
```
class DatabaseService {
    save(email, price, timestamp) {
        console.log(`Running query: INSERT INTO orders VALUES (email, price, created) VALUES (${email}, ${price}, ${timestamp})`);
    }
}

module.exports = DatabaseService

```


This creates a new DatabaseService class that contains a single save() method. Similar to the email service’s send() method, the save() function uses the data that accompanies a buy event, logging it to the console instead of actually inserting it into a database. This method needs the email of the purchaser, price of the ticket, and the time the ticket was purchased to function. Save and exit the file.


We will use our last file to bring the TicketManager, EmailService, and DatabaseService together. It will set up a listener for the buy event and will call the email service’s send() function and the database service’s save() function.


Open the index.js file in your text editor:


```
nano index.js


```


The first thing to do is import the modules we are using:


event-emitters/index.js
```
const TicketManager = require("./ticketManager");
const EmailService = require("./emailService");
const DatabaseService = require("./databaseService");

```


Next, let’s create objects for the classes we imported. We’ll set a low ticket supply of three for this demonstration:


event-emitters/index.js
```
const TicketManager = require("./ticketManager");
const EmailService = require("./emailService");
const DatabaseService = require("./databaseService");

const ticketManager = new TicketManager(3);
const emailService = new EmailService();
const databaseService = new DatabaseService();

```


We can now set up our listener with the instantiated objects. Whenever someone buys a ticket, we want to send them an email as well as saving the data to a database. Add the following listener to your code:


event-emitters/index.js
```
const TicketManager = require("./ticketManager");
const EmailService = require("./emailService");
const DatabaseService = require("./databaseService");

const ticketManager = new TicketManager(3);
const emailService = new EmailService();
const databaseService = new DatabaseService();


ticketManager.on("buy", (email, price, timestamp) => {
    emailService.send(email);
    databaseService.save(email, price, timestamp);
});

```


Like before, we add a listener with the on() method. The difference this time is that we have three arguments in our callback function. Each argument corresponds to the data that the event emits. As a reminder, this is the emitter code in the buy() function:


event-emitters/ticketManager.js
```
this.emit("buy", email, price, Date.now());

```


In our callback, we first capture the email from the emitter, then the price, and finally the Date.now() data, which we capture as timestamp.


When our listener detects a buy event, it will call the send() function from the emailService object as well as the save() function from databaseService. To test that this setup works, let’s make a call to the buy() function at the end of the file:


event-emitters/index.js
```
...

ticketManager.buy("test@email.com", 10);

```


Save and exit the editor. Now let’s run this script with node and observe what comes next. In your terminal enter:


```
node index.js


```


You will see the following output:


```
OutputSending email to test@email.com
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email.com, 10, 1588720081832)

```


The data was successfully captured and returned in our callback function. With this knowledge, you can set up listeners for a variety of emitters with different event names and data. However, there are certain nuances to handling error events with event emitters.


Next, let’s look at how to handle error events and what standards we should follow in doing so.


# Step 4 — Handling Error Events


If an event emitter cannot perform its action, it should emit an event to signal that the action failed. In Node.js, the standard way for an event emitter to signal failure is by emitting an error event.


An error event must have its name set to error. It must also be accompanied by an Error object. Let’s see a practical example of emitting an error event.


Our ticket manager decreases the supply by one every time the buy() function is called. Right now there’s nothing stopping it from selling more tickets than it has available. Let’s modify the buy() function so that if the ticket supply reaches 0 and someone wants to buy a ticket, we emit an error indicating that we’re out of stock.


Open ticketManager.js in your text editor once more:


```
nano ticketManager.js


```


Now edit the buy() function as follows:


event-emitters/ticketManager.js
```
...

buy(email, price) {
    if (this.supply > 0) {
        this.supply—;
        this.emit("buy", email, price, Date.now());
        return;
    }

    this.emit("error", new Error("There are no more tickets left to purchase"));
}
...

```


We’ve added an if statement that allows a ticket purchase if our supply is greater than zero. If we don’t have any other tickets, we’ll emit an error event. The error event is emitted with a new Error object that contains a description of why we’re throwing this error.


Save and exit the file. Let’s try to throw this error in our index.js file. Right now, we only buy one ticket. We instantiated the ticketManager object with three tickets, so we should get an error if we try to buy four tickets.


Edit index.js with your text editor:


```
nano index.js


```


Now add the following lines at the end of the file so we can buy four tickets in total:


event-emitters/index.js
```
...

ticketManager.buy("test@email.com", 10);
ticketManager.buy("test@email.com", 10);
ticketManager.buy("test@email.com", 10);
ticketManager.buy("test@email.com", 10);

```


Save and exit the editor.


Let’s execute this file to see what happens:


```
node index.js


```


Your output will be:


```
OutputSending email to test@email.com
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email.com, 10, 1588724932796)
Sending email to test@email.com
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email.com, 10, 1588724932812)
Sending email to test@email.com
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email.com, 10, 1588724932812)
events.js:196
      throw er; // Unhandled 'error' event
      ^

Error: There are no more tickets left to purchase
    at TicketManager.buy (/home/sammy/event-emitters/ticketManager.js:16:28)
    at Object.<anonymous> (/home/sammy/event-emitters/index.js:17:15)
    at Module._compile (internal/modules/cjs/loader.js:1128:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1167:10)
    at Module.load (internal/modules/cjs/loader.js:983:32)
    at Function.Module._load (internal/modules/cjs/loader.js:891:14)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:71:12)
    at internal/main/run_main_module.js:17:47
Emitted 'error' event on TicketManager instance at:
    at TicketManager.buy (/home/sammy/event-emitters/ticketManager.js:16:14)
    at Object.<anonymous> (/home/sammy/event-emitters/index.js:17:15)
    [... lines matching original stack trace ...]
    at internal/main/run_main_module.js:17:47

```


The first three buy events were processed correctly, but on the fourth buy event our program crashed. Let’s examine the beginning of the error message:


```
Output...
events.js:196
      throw er; // Unhandled 'error' event
      ^

Error: There are no more tickets left to purchase
    at TicketManager.buy (/home/sammy/event-emitters/ticketManager.js:16:28)
...

```


The first two lines highlight that an error was thrown. The comment says "Unhandled 'error' event". If an event emitter emits an error and we did not attach a listener for error events, Node.js throws the error and crashes the program.


It’s considered best practice to always listen for error events if you’re listening to an event emitter. If you do not set up a listener for errors, your entire application will crash if one is emitted. With an error listener, you can gracefully handle it.


To follow best practices, let’s set up a listener for errors. Re-open the index.js file:


```
nano index.js


```


Add a listener before we start buying tickets. Remember, a listener can only respond to events that are emitted after it was added. Insert an error listener like this:


event-emitters/index.js
```
...

ticketManager.on("error", (error) => {
    console.error(`Gracefully handling our error: ${error}`);
});

ticketManager.buy("test@email.com", 10);
ticketManager.buy("test@email.com", 10);
ticketManager.buy("test@email.com", 10);
ticketManager.buy("test@email.com", 10);

```


When we receive an error event, we will log it to the console with console.error().


Save and leave nano. Re-run the script to see our error event handled correctly:


```
node index.js


```


This time the following output will be displayed:


```
OutputSending email to test@email.com
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email.com, 10, 1588726293332)
Sending email to test@email.com
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email.com, 10, 1588726293348)
Sending email to test@email.com
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email.com, 10, 1588726293348)
Gracefully handling our error: Error: There are no more tickets left to purchase

```


From the last line, we confirm that our error event is being handled by our second listener, and the Node.js process did not crash.


Now that we’ve covered the concepts of sending and listening to events, let’s look at some additional functionality that can be used to manage event listeners.


# Step 5 — Managing Events Listeners


Event emitters come with some mechanisms to monitor and control how many listeners are subscribed to an event. To get an overview of how many listeners are processing an event, we can use the listenerCount() method that’s included in every object.


The listenerCount() method accepts one argument: the event name you want the count for. Let’s see it in action in index.js.


Open the file with nano or your text editor of choice:


```
nano index.js


```


You currently call the buy() function four times. Remove those four lines. When you do, add these two new lines so that your entire index.js looks like this:


event-emitters/index.js
```
const TicketManager = require("./ticketManager");
const EmailService = require("./emailService");
const DatabaseService = require("./databaseService");

const ticketManager = new TicketManager(3);
const emailService = new EmailService();
const databaseService = new DatabaseService();

ticketManager.on("buy", (email, price, timestamp) => {
    emailService.send(email);
    databaseService.save(email, price, timestamp);
});

ticketManager.on("error", (error) => {
    console.error(`Gracefully handling our error: ${error}`);
});

console.log(`We have ${ticketManager.listenerCount("buy")} listener(s) for the buy event`);
console.log(`We have ${ticketManager.listenerCount("error")} listener(s) for the error event`);

```


We’ve removed the calls to buy() from the previous section and instead logged two lines to the console. The first log statement uses the listenerCount() function to display the number of listeners for the buy() event. The second log statement shows how many listeners we have for the error event.


Save and exit. Now run your script with the node command:


```
node index.js


```


You’ll get this output:


```
OutputWe have 1 listener(s) for the buy event
We have 1 listener(s) for the error event

```


We only used the on() function once for the buy event and once for the error event, so this output matches our expectations.


Next, we’ll use the listenerCount() as we remove listeners from an event emitter. We may want to remove event listeners when the period of an event no longer applies. For example, if our ticket manager was being used for a specific concert, as the concert comes to an end you would remove the event listeners.


In Node.js we use the off() function to remove event listeners from an event emitter. The off() method accepts two arguments: the event name and the function that’s listening to it.



Note: Similar to the on() function, Node.js aliases the off() method with removeListener(). They both do the same thing, with the same arguments. In this tutorial, we will continue to use off().

For the second argument of the off() function, we need a reference to the callback that’s listening to an event. Therefore, to remove an event listener, its callback must be saved to some variable or constant. As it stands, we cannot remove the current event listeners for buy or error with the off() function.


To see off() in action, let’s add a new event listener that we will remove in subsequent calls. First, let’s define the callback in a variable so that we can reference it in off() later. Open index.js with nano:


```
nano index.js


```


At the end of index.js add this:


event-emitters/index.js
```
...

const onBuy = () => {
    console.log("I will be removed soon");
};

```


Now add another event listener for the buy event:


event-emitters/index.js
```
...

ticketManager.on("buy", onBuy);

```


To be sure that we successfully added that event listener, let’s print the listener count for buy and call the buy() function.


event-emitters/index.js
```
...

console.log(`We added a new event listener bringing our total count for the buy event to: ${ticketManager.listenerCount("buy")}`);
ticketManager.buy("test@email", 20);

```


Save and exit the file, then run the program:


```
node index.js


```


The following message will be displayed in the terminal:


```
OutputWe have 1 listener(s) for the buy event
We have 1 listener(s) for the error event
We added a new event listener bringing our total count for the buy event to: 2
Sending email to test@email
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email, 20, 1588814306693)
I will be removed soon

```


From the output, we see our log statement from when we added the new event listener. We then call the buy() function, and both listeners react to it. The first listener sent the email and saved data to the database, and then our second listener printed its message I will be removed soon to the screen.


Let’s now use the off() function to remove the second event listener. Re-open the file in nano:


```
nano index.js


```


Now add the following off() call to the end of the file. You will also add a log statement to confirm the number of listeners, and make another call to buy():


event-emitters/index.js
```
...

ticketManager.off("buy", onBuy);

console.log(`We now have: ${ticketManager.listenerCount("buy")} listener(s) for the buy event`);
ticketManager.buy("test@email", 20);

```


Note how the onBuy variable was used as the second argument of off(). Save and exit the file.


Now run this with node:


```
node index.js


```


The previous output will remain unchanged, but this time we will find the new log line we added confirming we have one listener once more. When buy() is called again, we will only see the output of the callback used by the first listener:


```
OutputWe have 1 listener(s) for the buy event
We have 1 listener(s) for the error event
We added a new event listener bringing our total count for the buy event to: 2
Sending email to test@email
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email, 20, 1588816352178)
I will be removed soon
We now have: 1 listener(s) for the buy event
Sending email to test@email
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email, 20, 1588816352178)

```


If we wanted to remove all events with off(), we could use the removeAllListeners() function. This function accepts one argument: the name of the event we want to remove listeners for.


Let’s use this function at the end of the file to take off the first event listener we added for the buy event. Open the index.js file once more:


```
nano index.js


```


You’ll first remove all the listeners with removeAllListeners(). You’ll then log a statement with the listener count using the listenerCount() function. To confirm it’s no longer listening, you’ll buy another ticket. When the event is emitted, nothing will react to it.


Enter the following code at the end of the file:


event-emitters/index.js
```
...

ticketManager.removeAllListeners("buy");
console.log(`We have ${ticketManager.listenerCount("buy")} listeners for the buy event`);
ticketManager.buy("test@email", 20);
console.log("The last ticket was bought");

```


Save and exit the file.


Now let’s execute our code with the node command:


```
node index.js


```


Our final output is:


```
We have 1 listener(s) for the buy event
We have 1 listener(s) for the error event
We added a new event listener bringing our total count for the buy event to: 2
Sending email to test@email
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email, 20, 1588817058088)
I will be removed soon
We now have: 1 listener(s) for the buy event
Sending email to test@email
Running query: INSERT INTO orders VALUES (email, price, created) VALUES (test@email, 20, 1588817058088)
We have 0 listeners for the buy event
The last ticket was bought

```


After removing all listeners, we no longer send emails or save to the database for ticket purchases.


# Conclusion


In this tutorial, you learned how to use Node.js event emitters to trigger events. You emitted events with the emit() function of an EventEmitter object, then listened to events with the on() and once() functions to execute code every time the event is triggered. You also added a listener for an error event and monitored and managed listeners with the listenerCount() function.


With callbacks and promises, our ticket manager system would need to be integrated with the email and database service modules to get the same functionality. Since we used event emitters, the event was decoupled from the implementations. Furthermore, any module with access to the ticket manager can observe its event and react to it. If you want Node.js modules, internal or external, to observe what’s happening with your object, consider making it an event emitter for scalability.


To learn more about events in Node.js, you can read the Node.js documentation. If you’d like to continue learning Node.js, you can return to the How To Code in Node.js series, or browse programming projects and setups on our Node topic page.


