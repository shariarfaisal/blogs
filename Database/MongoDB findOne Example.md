# MongoDB findOne Example

```Database``` ```MongoDB```

MongoDB findOne() method returns only one document that satisfies the criteria entered. If the criteria entered matches for more than one document, the method returns only one document according to natural ordering, which reflects the order in which the documents are stored in the database.


# MongoDB findOne


 MongoDB findOne() syntax is: db.collection.findOne(<criteria>, <projection>) criteria - specifies the selection criteria entered. projection - specifies the list of fields to be displayed in the returned document. Few important points about MongoDB findOne:


1. The projection parameter accepts the boolean values of 1 or true , 0 or false. If the projection fields are not specified, all the fields will be retrieved.
2. MongoDB findOne() always includes the _id field even if not specified explicitly in the projection parameter unless it is excluded.
3. MongoDB findOne() returns only a document but not a cursor.

## MongoDB findOne - Empty Query specification


This operation returns a single document from the collection specified. For example, db.car.findOne() Output:


```
{
"_id" : 2,
"name" : "Polo", "color" : "White",
"cno" : "H411", "speed" : 45, "mfdcountry" : "Japan"
}

```


Only one record is retrieved from the car collection. Note that “Polo” was inserted first into the database.


## MongoDB findOne - Query specification


This MongoDB findOne operation returns the first matching document from the specified collection along with the selection criteria entered. For example:


```
>db.car.findOne(
... {
... $or:[
... {name:"Zen"},
... {speed: {$gt:60}} ... ]
... }
... )

{
"_id" : ObjectId("546cb92393f464ed49d620db"), 
"name" : "Zen",
"color" : "JetRed",
"cno" : "H671",
"speed" : 67, 
"mfdcountry" : "Rome"
}

```


This operation searches for the car named “Zen” or for speed greater than 60 and retrieves the first document satisfying the criteria entered from the car collection.


## Projection in MongoDB findOne()


The projection parameter is also applicable for MongoDB findOne method. Let’s look at some scenarios where we can use projection in findOne.


## MongoDB findOne - specify the fields to be returned


This operation displays only the fields specified in the query. For example:


```
>db.car.findOne(
... { },
... {name:1,color:1}
... )

{ "_id" : 2, "name" : "Polo", "color" : "White" }

```


The first document from the Car collection with id, name and color fields are displayed.


## MongoDB findOne - return all the fields except the excluded one


This operation retrieves the first document excluding the fields specified in the selection criteria. For example;


```
>db.car.findOne(
... { name:"Volkswagen" },
... {_id:0, mfdcountry:0,cno:0 }
... )

{ "name" : "Volkswagen", "color" : "JetBlue", "speed" : 62 }

```


The car name with Volkswagen is retrieved excluding the id, mfdcountry and cno fields.


## MongoDB findOne result document


Cursor methods will not work in this operation as the method returns only a single document. In order to print the individual field values from the document, we can use the below code.


```
>var car = db.car.findOne(); 
> if (car) {
...  var carName = car.name;
...  print (tojson(carName));
... }

"Polo"

```


This retrieves only the name of the car “Polo”.


## MongoDB findOne Java Example


Below is the java program showing different options that we can use with MongoDB findOne(). MongoDBFindOne.java


```
package com.journaldev.mongodb;

import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBObject;
import com.mongodb.MongoClient;

import java.net.UnknownHostException;

public class MongoDBFindOne {

	// method retrieves the first document without any criteria
	public static void emptyFindOne() throws UnknownHostException {

		// Get a new connection to the db assuming that it is running
		MongoClient mongoClient = new MongoClient("localhost");

		// use test as a datbase,use your database here
		DB db = mongoClient.getDB("test");

		// fetch the collection object ,car is used here,use your own
		DBCollection coll = db.getCollection("car");

		// invoking findOne() method
		DBObject doc = coll.findOne();

		// prints the resultant document
		System.out.println(doc);
	}

	// method that retrieves the document based on the selection criteria
	public static void querySpecification() throws UnknownHostException {
		// getting a connection everytime is not needed (could be done once
		// globally).
		MongoClient mongoClient = new MongoClient("localhost");
		DB db = mongoClient.getDB("test");
		DBCollection coll = db.getCollection("car");

		// query to filter the document based on name and speed values by
		// creating new object
		DBObject query = new BasicDBObject("name", "Zen").append("speed",
				new BasicDBObject("$gt", 30));

		// resultant document fetched by satisfying the criteria
		DBObject d1 = coll.findOne(query);

		// prints the document on console
		System.out.println(d1);
	}

	public static void projectionFields() throws UnknownHostException {

		MongoClient mongoClient = new MongoClient("localhost");
		DB db = mongoClient.getDB("test");
		DBCollection coll = db.getCollection("car");

		// creates new db object
		BasicDBObject b1 = new BasicDBObject();

		// criteria to display only name and color fields in the resultant
		// document
		BasicDBObject fields = new BasicDBObject("name", 1).append("color", 1);

		// method that retrieves the documents by accepting fields and object
		// criteria
		DBObject d1 = coll.findOne(b1, fields);

		System.out.println(d1);

	}

	public static void excludeByfields() throws UnknownHostException {
		MongoClient m1 = new MongoClient("localhost");

		DB db = m1.getDB("test");
		DBCollection col = db.getCollection("car");

		// filter criteria for car name volkswagen
		DBObject query = new BasicDBObject("name", "Volkswagen");

		// excluding the fields mfdcountry,cno and id fields
		BasicDBObject fields = new BasicDBObject("mfdcountry", 0).append("cno",
				0).append("_id", 0);

		DBObject d1 = col.findOne(query, fields);
		System.out.println(d1);
	}

	public static void printDoc() throws UnknownHostException {
		MongoClient m1 = new MongoClient("localhost");
		DB db = m1.getDB("test");
		DBCollection col = db.getCollection("car");
		
		DBObject d1 = col.findOne();
		
		// prints only the name of the car
		System.out.println((d1.get("name")));
	}
	
	public static void main(String[] args) throws UnknownHostException {
		// invoking all the methods from main
		emptyFindOne();
		querySpecification();
		projectionFields();
		excludeByfields();
		printDoc();
	}

}

```


Output of above program is:


```
{ "_id" : 2.0 , "name" : "Polo" , "color" : "White" , "cno" : "H411" , "speed" : 45.0 , "mfdcountry" : "Japan"} ­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­­
{ "_id" : { "$oid" : "546cb92393f464ed49d620db"} , "name" : "Zen" , "color" : "JetRed" , "cno" : "H671" , "speed" : 67 , "mfdcountry" : "Rome"}
{ "_id" : 2.0 , "name" : "Polo" , "color" : "White"}
{ "name" : "Volkswagen" , "color" : "JetBlue" , "speed" : 62}
Polo

```


That’s all for MongoDB findOne() method, we will look into more MongoDB options in coming posts. Reference: Official Doc


