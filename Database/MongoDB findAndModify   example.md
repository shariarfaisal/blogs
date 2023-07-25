# MongoDB findAndModify   example

```Database``` ```MongoDB```

MongoDB findAndModify() method modifies and returns a single document based upon the selection criteria entered. The returned document does not show the updated content by default. If the records matching the criteria does not exist in the database, a new record will be inserted if the upsert is set to true.


# MongoDB findAndModify()


The syntax for mongodb findAndModify method is as follows.


```
db.collection.findAndModify({
    query: <document>,
    sort: <document>,
    new: <boolean>,
    fields: <document>,
    upsert: <boolean>
})

```


The description of the parameters are as follows. query: Defines the selection criteria as to which record needs modification. sort: Determines which document should be modified when the selection criteria retrieves multiple documents. new: indicates that the modified document will be displayed. fields: specifies the set of fields to be returned. upsert: creates a new document if the selection criteria fails to retrieve a document.


## MongoDB findAndModify important points


Some of the things to keep in mind while using the findAndModify MongoDB calls are:


- If the entered selection criteria does not fetch any document and if upsert is set to true, then the specified values are inserted and a new document is created.
- If the entered selection criteria does not fetch any document while performing update or remove operations, the output returned is null.
- If the new option is set to false and sort operation is not mentioned, the output returned is null.
- If the new option is set to false and sort operation is specified, the output is empty.

## MongoDB findAndModify Example


Now let us see some examples demonstrating the usage of the findAndModify API. First, let us create some test data to start with. We will create a new car document with name, color, car no (cno), speed and manufactured country(mfdcountry) fields through the mongo console.


```
db.car.insert(
[
{ _id: 1, name: "Alto", color: "Red",cno: "H410",speed:40,mfdcountry: "India"},
{ _id: 2, name: "Polo", color: "White",cno: "H411",speed:45,mfdcountry: "Japan" },
{ _id: 3, name: "Audi", color: "Black",cno: "H412",speed:50,mfdcountry: "Germany" }
]
)

```


Let us now verify that the data was actually inserted using mongodb find :


```
db.car.find()
{ "_id" : 1, "name" : "Alto", "color" : "Red", "cno" : "H410", "speed" : 40, "mfdcountry" : "India" }
{ "_id" : 2, "name" : "Polo", "color" : "White", "cno" : "H411", "speed" : 45, "mfdcountry" : "Japan" }
{ "_id" : 3, "name" : "Audi", "color" : "Black", "cno" : "H412", "speed" : 50, "mfdcountry" : "Germany" }

```


Moving on to the actual usage of find and modify, we depict different possibilities. Case 1: The document exists in the database


```
db.car.findAndModify({
query: { name: "Alto" },
sort: { cno: 1 },
update: { $inc: { speed: 10 } },
})

```


1. The query finds a document in the car collection where the name field has the value Alto.
2. The sort orders the results of the query in ascending order. If multiple documents meet the query condition, the method will select for modification the first document as ordered by this sort.
3. The update increments the value of the speed field by 10.
4. The method returns the original document selected for this update :

Output:


```
{
"_id" : 1,
"name" : "Alto",
"color" : "Red",
"cno" : "H410",
"speed" : 40,
"mfdcountry" : "India"
}

```


Case 2: The new option set to true (returns the updated data set)


```
db.car.findAndModify({
query: { name: "HondaCity", color: "Silver", cno:"H415" ,speed: 25 },
sort: { cno: 1 },
update: { $inc: { speed: 20 } },
upsert: true,
new: true
})

```


Output:


```
{
"_id" : ObjectId("546c9f347256eabc40c9da1c"),
"cno" : "H415",
"color" : "Silver",
"name" : "HondaCity",
"speed" : 45
}

```


Note that speed is being displayed as 45 which is the updated value since we set new to be true. If this is not set, the speed field will be shown as 20 , the old value though it gets updated in the database. Case 3: The upsert is set to true


```
db.car.findAndModify({
query: { name: "WagonR" },
sort: { cno: 1 },
update: { $inc: { speed: 5 } },
upsert: <strong>true</strong>
})

```


Output:


```
db.car.find();
{ "_id" : 1, "name" : "Alto", "color" : "Red", "cno" : "H410", "speed" : 50, "mfdcountry" : "India" }
{ "_id" : 2, "name" : "Polo", "color" : "White", "cno" : "H411", "speed" : 45, "mfdcountry" : "Japan" }
{ "_id" : 3, "name" : "Audi", "color" : "Black", "cno" : "H412", "speed" : 50, "mfdcountry" : "Germany" }
{ "_id" : ObjectId("546c7c7d6be0faf69ee36546"), "name" : "WagonR", "speed" : 5 }

```


The car name with WagonR is not in the db hence a new document is created in database. The method returns an empty document { } if the sort option is specified. If sort option is not included then the method returns null. Apart from these, this can also be used for doing a Sort and Remove operation. If the remove field is set to true, the car name with the specified criteria will be removed from the database.


```
db.car.findAndModify(
{
query: { name: "Alto" },
sort: { cno: 1 },
remove: true
}
)

```


Output:


```
{
"_id" : 1,
"name" : "Alto",
"color" : "Red",
"cno" : "H410",
"speed" : 50,
"mfdcountry" : "India"
}

```


The remove field is set to true the document with the name “Alto” gets deleted from the database.


## MongoDB findAndModify Java Example


The operations depicted above is all manual performed using mongo shell. The same can be done programmatically in Java as below. Download the mongo driver jar and add it to your classpath. First, we need to establish a connection to the mongodb server using the Mongo client. The name of the database should be specified for the connection as a parameter. If the database does not exists, a new database will is created. After this, we will add a few records into the database and do a findAndModify operation and then verify if the records are actually updated. Below program works with mongo java driver versions 2.x.


```
package com.journaldev.mongodb;

import java.net.UnknownHostException;

import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;
import com.mongodb.DBObject;
import com.mongodb.MongoClient;

public class MongoDBFindAndModify {

	public static void main(String[] args) throws UnknownHostException {
		// Get a new connection to the db assuming it is running.
		MongoClient mongoClient = new MongoClient("localhost");
		// use test as the database. Use your database here.
		DB db = mongoClient.getDB("test");

		DBCollection coll = db.getCollection("car");
		
		// insert some test data to start with.
		BasicDBObject obj = new BasicDBObject();
		obj.append("name", "Volkswagen");
		obj.append("color", "JetBlue");
		obj.append("cno", "H672");
		obj.append("speed", 62);
		obj.append("mfdcountry", "Italy");
		coll.insert(obj);
		
		// findAndModify operation. Update colour to blue for cars having speed
		// < 45
		DBObject query = new BasicDBObject("speed",
				new BasicDBObject("$lt", 45));
		DBObject update = new BasicDBObject();
		update.put("$set", new BasicDBObject("color", "Blue"));
		DBCursor cursor = coll.find();
		try {
			while (cursor.hasNext()) {
				System.out.println(cursor.next());
			}
		} finally {
			cursor.close();
		}
		coll.findAndModify(query, update);
	}
}

```


Output:


```
//Test Data Before Insert
{ "_id" : 2.0 , "name" : "Polo" , "color" : "White" , "cno" : "H411" , "speed" : 45.0 , "mfdcountry" : "Japan"}
{ "_id" : 3.0 , "name" : "Audi" , "color" : "Black" , "cno" : "H412" , "speed" : 50.0 , "mfdcountry" : "Germany"}


//Test Data After insert
{ "_id" : 2.0 , "name" : "Polo" , "color" : "White" , "cno" : "H411" , "speed" : 45.0 , "mfdcountry" : "Japan"}
{ "_id" : 3.0 , "name" : "Audi" , "color" : "Black" , "cno" : "H412" , "speed" : 50.0 , "mfdcountry" : "Germany"}
{ "_id" : { "$oid" : "546cc76093f404729e2e946e"} , "name" : "Volkswagen" , "color" : "JetBlue" , "cno" : "H672" , "speed" : 62 , "mfdcountry" : "Italy"}


/*Test Data Before findandModify
{ "_id" : 2.0 , "name" : "Polo" , "color" : "White" , "cno" : "H411" , "speed" : 45.0 , "mfdcountry" : "Japan"}
{ "_id" : 3.0 , "name" : "Audi" , "color" : "Black" , "cno" : "H412" , "speed" : 50.0 , "mfdcountry" : "Germany"}
{ "_id" : { "$oid" : "546cc76093f404729e2e946e"} , "name" : "Volkswagen" , "color" : "JetBlue" , "cno" : "H672" , "speed" : 62 , "mfdcountry" : "Italy"}
{ "_id" : { "$oid" : "546c7c7d6be0faf69ee36546"} , "name" : "WagonR" , "speed" : 5.0}


/*Test Data After findAndModify
{ "_id" : 2.0 , "name" : "Polo" , "color" : "White" , "cno" : "H411" , "speed" : 45.0 , "mfdcountry" : "Japan"}
{ "_id" : 3.0 , "name" : "Audi" , "color" : "Black" , "cno" : "H412" , "speed" : 50.0 , "mfdcountry" : "Germany"}
{ "_id" : { "$oid" : "546cc76093f404729e2e946e"} , "name" : "Volkswagen" , "color" : "JetBlue" , "cno" : "H672" , "speed" : 62 , "mfdcountry" : "Italy"}
{ "_id" : { "$oid" : "546c7c7d6be0faf69ee36546"} , "name" : "WagonR" , "speed" : 5.0 , "color" : "Blue"}

```


If you are using MongoDB java driver versions 3.x, then use below program.


```
package com.journaldev.mongodb.main;

import java.net.UnknownHostException;

import org.bson.Document;
import org.bson.conversions.Bson;

import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
 
public class MongoDBFindAndModify {
 
    public static void main(String[] args) throws UnknownHostException {
        // Get a new connection to the db assuming it is running.
        MongoClient mongoClient = new MongoClient("localhost");
        
        // use test as the database. Use your database here.
        MongoDatabase db = mongoClient.getDatabase("test");
 
        MongoCollection<Document> coll = db.getCollection("car");
         
        // insert some test data to start with.
        Document obj = new Document();
        obj.append("name", "Volkswagen");
        obj.append("color", "JetBlue");
        obj.append("cno", "H672");
        obj.append("speed", 62);
        obj.append("mfdcountry", "Italy");
        coll.insertOne(obj);
         
        // findAndModify operation. Update color to blue for cars having speed > 45
        Bson query = new Document("speed",
                new Document("$gt", 45));
        Bson update = new Document("$set", 
        			new Document("color", "Blue"));
        
        System.out.println("before update");
        findAndPrint(coll);
        
        coll.findOneAndUpdate(query, update);
        
        System.out.println("after update of color field");
        findAndPrint(coll);
        
        mongoClient.close();

    }

	private static void findAndPrint(MongoCollection<Document> coll) {
		FindIterable<Document> cursor = coll.find();
        
		for (Document d : cursor)
			System.out.println(d);
	}
}

```


Below image shows the output of above program, notice the change in color field.  That’s all for MongoDB findAndModify example, we will look into more MongoDB operations in the coming posts. Reference: MongoDB Official Documentation


