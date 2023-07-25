# How To Create a Real-Time App with Socket IO  Angular  and Node js

```Angular``` ```Node.js```

## Introduction


WebSocket is the internet protocol that allows for full-duplex communication between a server and clients. This protocol goes beyond the typical HTTP request and response paradigm. With WebSockets, the server may send data to a client without the client initiating a request, thus allowing for some very interesting applications.


In this tutorial, you will build a real-time document collaboration application (similar to Google Docs). We’ll be using the Socket.IO Node.js server framework and Angular 7 to accomplish this.


You can find the complete source code for this example project on GitHub.


# Prerequisites


To complete this tutorial, you will need:


- Node.js installed locally, which you can do by following How to Install Node.js and Create a Local Development Environment.
- A modern web browser that supports WebSocket.

This tutorial was originally written in an environment consisting of Node.js v8.11.4, npm v6.4.1, and Angular v7.0.4.


This tutorial was verified with Node v14.6.0, npm v6.14.7, Angular v10.0.5, and Socket.IO v2.3.0.


# Step 1 — Setting Up the Project Directory and Creating the Socket Server


First, open your terminal and create a new project directory that will hold both our server and client code:


```
mkdir socket-example


```


Next, change into the project directory:


```
cd socket-example


```


Then, create a new directory for the server code:


```
mkdir socket-server


```


Next, change into the server directory.


```
cd socket-server


```


Then, initialize a new npm project:


```
npm init -y


```


Now, we will install our package dependencies:


```
npm install express@4.17.1 socket.io@2.3.0 @types/socket.io@2.1.10 --save


```


These packages include Express, Socket.IO, and @types/socket.io.


Now that you have completed setting up the project, you can move on to writing code for the server.


First, create a new src directory:


```
mkdir src


```


Now, create a new file called app.js in the src directory, and open it using your favorite text editor:


```
nano src/app.js


```


Start with the require statements for Express and Socket.IO:


socket-server/src/app.js
```
const app = require('express')();
const http = require('http').Server(app);
const io = require('socket.io')(http);

```


As you can tell, we’re using Express and Socket.IO to set up our server. Socket.IO provides a layer of abstraction over native WebSockets. It comes with some nice features, such as a fallback mechanism for older browsers that do not support WebSockets, and the ability to create rooms. We’ll see this in action in a minute.


For the purposes of our real-time document collaboration application, we will need a way to store documents. In a production setting, you would want to use a database, but for the scope of this tutorial, we will use an in-memory store of documents:


socket-server/src/app.js
```
const documents = {};

```


Now, let’s define what we want our socket server to actually do:


socket-server/src/app.js
```
io.on("connection", socket => {
  // ...
});

```


Let’s break this down. .on('...') is an event listener. The first parameter is the name of the event, and the second one is usually a callback executed when the event fires, with the event payload.


The first example we see is when a client connects to the socket server (connection is a reserved event type in Socket.IO).


We get a socket variable to pass to our callback to initiate communication to either that one socket or to multiple sockets (i.e., broadcasting).


safeJoin


We will set up a local function (safeJoin) that takes care of joining and leaving rooms:


socket-server/src/app.js
```
io.on("connection", socket => {
  let previousId;

  const safeJoin = currentId => {
    socket.leave(previousId);
    socket.join(currentId, () => console.log(`Socket ${socket.id} joined room ${currentId}`));
    previousId = currentId;
  };

  // ...
});

```


In this case, when a client has joined a room, they are editing a particular document. So if multiple clients are in the same room, they are all editing the same document.


Technically, a socket can be in multiple rooms, but we don’t want to let one client edit multiple documents at the same time, so if they switch documents, we need to leave the previous room and join the new room. This little function takes care of that.


There are three event types that our socket is listening for from the client:


- getDoc
- addDoc
- editDoc

And two event types that are emitted by our socket to the client:


- document
- documents

getDoc


Let’s work on the first event type - getDoc:


socket-server/src/app.js
```
io.on("connection", socket => {
  // ...

  socket.on("getDoc", docId => {
    safeJoin(docId);
    socket.emit("document", documents[docId]);
  });

  // ...
});

```


When the client emits the getDoc event, the socket is going to take the payload (in our case, it’s just an id), join a room with that docId, and emit the stored document back to the initiating client only. That’s where socket.emit('document', ...) comes into play.


addDoc


Let’s work on the second event type - addDoc:


socket-server/src/app.js
```
io.on("connection", socket => {
  // ...

  socket.on("addDoc", doc => {
    documents[doc.id] = doc;
    safeJoin(doc.id);
    io.emit("documents", Object.keys(documents));
    socket.emit("document", doc);
  });

  // ...
});

```


With the addDoc event, the payload is a document object, which, at the moment, consists only of an id generated by the client. We tell our socket to join the room of that ID so that any future edits can be broadcast to anyone in the same room.


Next, we want everyone connected to our server to know that there is a new document to work with, so we broadcast to all clients with the io.emit('documents', ...) function.


Note the difference between socket.emit() and io.emit() - the socket version is for emitting back to only initiating the client, the io version is for emitting to everyone connected to our server.


editDoc


Let’s work on the third event type - editDoc:


socket-server/src/app.js
```
io.on("connection", socket => {
  // ...

  socket.on("editDoc", doc => {
    documents[doc.id] = doc;
    socket.to(doc.id).emit("document", doc);
  });

  // ...
});

```


With the editDoc event, the payload will be the whole document at its state after any keystroke. We’ll replace the existing document in the database and then broadcast the new document to only the clients that are currently viewing that document. We do this by calling socket.to(doc.id).emit(document, doc), which emits to all sockets in that particular room.


Finally, whenever a new connection is made, we broadcast to all the clients to ensure the new connection receives the latest document changes when they connect:


socket-server/src/app.js
```
io.on("connection", socket => {
  // ...

  io.emit("documents", Object.keys(documents));

  console.log(`Socket ${socket.id} has connected`);
});

```


After the socket functions are all set up, pick a port and listen on it:


socket-server/src/app.js
```
http.listen(4444, () => {
  console.log('Listening on port 4444');
});

```


Run the following command in your terminal to start the server:


```
node src/app.js


```


We now have a fully-functioning socket server for document collaboration!


# Step 2 — Installing @angular/cli and Creating the Client App


Open a new terminal window and navigate to the project directory.


Run the following commands to install the Angular CLI as a devDependency:


```
npm install @angular/cli@10.0.4 --save-dev


```


Now, use the @angular/cli command to create a new Angular project, with no Angular Routing and with SCSS for styling:


```
ng new socket-app --routing=false --style=scss


```


Then, change into the server directory:


```
cd socket-app


```


Now, we will install our package dependencies:


```
npm install ngx-socket-io@3.2.0 --save


```


ngx-socket-io is an Angular wrapper over Socket.IO client libraries.


Then, use the @angular/cli command to generate a document model, a document-list component, a document component, and a document service:


```
ng generate class models/document --type=model
ng generate component components/document-list
ng generate component components/document
ng generate service services/document


```


Now that you have completed setting up the project, you can move on to writing code for the client.


## App Module


Open app.modules.ts:


```
nano src/app/app.module.ts


```


And import FormsModule, SocketioModule, and SocketioConfig:


socket-app/src/app/app.module.ts
```
// ... other imports
import { FormsModule } from '@angular/forms';
import { SocketIoModule, SocketIoConfig } from 'ngx-socket-io';

```


And before your @NgModule declaration, define config:


socket-app/src/app/app.module.ts
```
const config: SocketIoConfig = { url: 'http://localhost:4444', options: {} };

```


You’ll notice that this is the port number that we declared earlier in the server’s app.js.


Now, add to your imports array, so it looks like:


socket-app/src/app/app.module.ts
```
@NgModule({
  // ...
  imports: [
    // ...
    FormsModule,
    SocketIoModule.forRoot(config)
  ],
  // ...
})

```


This will fire off the connection to our socket server as soon as AppModule loads.


## Document Model and Document Service


Open document.model.ts:


```
nano src/app/models/document.model.ts


```


And define id and doc:


socket-app/src/app/models/document.model.ts
```
export class Document {
  id: string;
  doc: string;
}

```


Open document.service.ts:


```
nano src/app/services/document.service.ts


```


And add the following in the class definition:


socket-app/src/app/services/document.service.ts
```
import { Injectable } from '@angular/core';
import { Socket } from 'ngx-socket-io';
import { Document } from 'src/app/models/document.model';

@Injectable({
  providedIn: 'root'
})
export class DocumentService {
  currentDocument = this.socket.fromEvent<Document>('document');
  documents = this.socket.fromEvent<string[]>('documents');

  constructor(private socket: Socket) { }

  getDocument(id: string) {
    this.socket.emit('getDoc', id);
  }

  newDocument() {
    this.socket.emit('addDoc', { id: this.docId(), doc: '' });
  }

  editDocument(document: Document) {
    this.socket.emit('editDoc', document);
  }

  private docId() {
    let text = '';
    const possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';

    for (let i = 0; i < 5; i++) {
      text += possible.charAt(Math.floor(Math.random() * possible.length));
    }

    return text;
  }
}

```


The methods here represent each emit the three event types that the socket server is listening for. The properties currentDocument and documents represent the events emitted by the socket server, which is consumed on the client as an Observable.


You may notice a call to this.docId(). This is a little private method that generates a random string to assign as the document id.


## Document List Component


Let’s put the list of documents in a sidenav. Right now, it’s only showing the docId - a random string of characters.


Open document-list.component.html:


```
nano src/app/components/document-list/document-list.component.html


```


And replace the contents with the following:


socket-app/src/app/components/document-list/document-list.component.html
```
<div class='sidenav'>
    <span
      (click)='newDoc()'
    >
      New Document
    </span>
    <span
      [class.selected]='docId === currentDoc'
      (click)='loadDoc(docId)'
      *ngFor='let docId of documents | async'
    >
      {{ docId }}
    </span>
</div>

```


Open document-list.component.scss:


```
nano src/app/components/document-list/document-list.component.scss


```


And add some styles:


socket-app/src/app/components/document-list/document-list.component.scss
```
.sidenav {
  background-color: #111111;
  height: 100%;
  left: 0;
  overflow-x: hidden;
  padding-top: 20px;
  position: fixed;
  top: 0;
  width: 220px;

  span {
    color: #818181;
    display: block;
    font-family: 'Roboto', Tahoma, Geneva, Verdana, sans-serif;
    font-size: 25px;
    padding: 6px  8px  6px  16px;
    text-decoration: none;

    &.selected {
      color: #e1e1e1;
    }

    &:hover {
      color: #f1f1f1;
      cursor: pointer;
    }
  }
}

```


Open document-list.component.ts:


```
nano src/app/components/document-list/document-list.component.ts


```


And add the following in the class definition:


socket-app/src/app/components/document-list/document-list.component.ts
```
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Observable, Subscription } from 'rxjs';

import { DocumentService } from 'src/app/services/document.service';

@Component({
  selector: 'app-document-list',
  templateUrl: './document-list.component.html',
  styleUrls: ['./document-list.component.scss']
})
export class DocumentListComponent implements OnInit, OnDestroy {
  documents: Observable<string[]>;
  currentDoc: string;
  private _docSub: Subscription;

  constructor(private documentService: DocumentService) { }

  ngOnInit() {
    this.documents = this.documentService.documents;
    this._docSub = this.documentService.currentDocument.subscribe(doc => this.currentDoc = doc.id);
  }

  ngOnDestroy() {
    this._docSub.unsubscribe();
  }

  loadDoc(id: string) {
    this.documentService.getDocument(id);
  }

  newDoc() {
    this.documentService.newDocument();
  }
}

```


Let’s start with the properties. documents will be a stream of all available documents. currentDocId is the id of the currently selected document. The document list needs to know what document we’re on, so we can highlight that doc id in the sidenav. _docSub is a reference to the Subscription that gives us the current or selected doc. We need this so we can unsubscribe in the ngOnDestroy lifecycle method.


You’ll notice the methods loadDoc() and newDoc() don’t return or assign anything. Remember, these fire off events to the socket server, which turns around and fires an event back to our Observables. The returned values for getting an existing document or adding a new document are realized from the Observable patterns above.


## Document Component


This will be the document editing surface.


Open document.component.html:


```
nano src/app/components/document/document.component.html


```


And replace the contents with the following:


socket-app/src/app/components/document/document.component.html
```
<textarea
  [(ngModel)]='document.doc'
  (keyup)='editDoc()'
  placeholder='Start typing...'
></textarea>

```


Open document.component.scss:


```
nano src/app/components/document/document.component.scss


```


And change some styles on the default HTML textarea:


socket-app/src/app/components/document/document.component.scss
```
textarea {
  border: none;
  font-size: 18pt;
  height: 100%;
  padding: 20px  0  20px  15px;
  position: fixed;
  resize: none;
  right: 0;
  top: 0;
  width: calc(100% - 235px);
}

```


Open document.component.ts:


```
src/app/components/document/document.component.ts


```


And add the following in the class definition:


socket-app/src/app/components/document/document.component.ts
```
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs';
import { startWith } from 'rxjs/operators';

import { Document } from 'src/app/models/document.model';
import { DocumentService } from 'src/app/services/document.service';

@Component({
  selector: 'app-document',
  templateUrl: './document.component.html',
  styleUrls: ['./document.component.scss']
})
export class DocumentComponent implements OnInit, OnDestroy {
  document: Document;
  private _docSub: Subscription;

  constructor(private documentService: DocumentService) { }

  ngOnInit() {
    this._docSub = this.documentService.currentDocument.pipe(
      startWith({ id: '', doc: 'Select an existing document or create a new one to get started' })
    ).subscribe(document => this.document = document);
  }

  ngOnDestroy() {
    this._docSub.unsubscribe();
  }

  editDoc() {
    this.documentService.editDocument(this.document);
  }
}

```


Similar to the pattern we used in the DocumentListComponent above, we’re going to subscribe to the changes for our current document, and fire off an event to the socket server whenever we change the current document. This means that we will see all the changes if any other client is editing the same document we are, and vice versa. We use the RxJS startWith operator to give a little message to our users when they first open the app.


## AppComponent


Open app.component.html:


```
nano src/app.component.html


```


And compose the two custom components by replacing the contents with the following:


socket-app/src/app.component.html
```
<app-document-list></app-document-list>
<app-document></app-document>

```


# Step 3 — Viewing the App in Action


With our socket server still running in a terminal window, let’s open a new terminal window and start our Angular app:


```
ng serve


```


Open more than one instance of http://localhost:4200 in separate browser tabs and see it in action.





Now, you can create new documents and see them update in both browser windows. You can make a change in one browser window and see the change reflected in the other browser window.


# Conclusion


In this tutorial, you have completed an initial exploration into using WebSocket. You used it to build a real-time document collaboration application. It supports multiple browser sessions to connect to a server and update and modify multiple documents.


If you’d like to learn more about Angular, check out our Angular topic page for exercises and programming projects.


If you’d like to learn more about Socket.IO, check out Integrating Vue.js and Socket.IO.


Further WebSocket projects include real-time chat applications. See How To Build a Realtime Chat App with React and GraphQL.


