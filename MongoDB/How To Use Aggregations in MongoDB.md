# How To Use Aggregations in MongoDB

```Databases``` ```MongoDB``` ```Data Analysis``` ```NoSQL```

The author selected the Open Internet/Free Speech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


MongoDB is a database management system that allows you to store large amounts of data in documents that are held within larger structures known as collections. You can run queries on collections to retrieve a subset of  documents matching given conditions, but MongoDB’s query mechanism doesn’t allow you to group or transform the returned data. This means your options for performing meaningful data analysis with MongoDB’s query mechanism alone are limited.


As with many other database systems, MongoDB allows you to perform a variety of aggregation operations. These allow you to process data records in a variety of ways, such as grouping data, sorting data into a specific order, or restructuring returned documents, as well as filtering data as one might with a query.


MongoDB provides aggregation operations through aggregation pipelines — a series of operations that process data documents sequentially. In this tutorial, you’ll learn by example how to use the most common features of the aggregation pipelines. You’ll filter, sort, group, and transform documents, and then use all these features together to form a multi-stage processing pipeline.


# Prerequisites


To follow this tutorial, you will need:


- A server with a regular, non-root user with sudo privileges and a firewall configured with UFW. This tutorial was validated using a server running Ubuntu 20.04, and you can prepare your server by following this initial server setup tutorial for Ubuntu 20.04.
- MongoDB installed on your server. To set this up, follow our tutorial on How to Install MongoDB on Ubuntu 20.04.
- Your server’s MongoDB instance secured by enabling authentication and creating an administrative user. To secure MongoDB like this, follow our tutorial on How To Secure MongoDB on Ubuntu 20.04.
- Familiarity with querying MongoDB collections and filtering results. To learn how to use MongoDB queries, follow the tutorial How To Create Queries in MongoDB.


Note: The linked tutorials on how to configure your server, install, and then secure MongoDB installation refer to Ubuntu 20.04. This tutorial concentrates on MongoDB itself, not the underlying operating system. It will generally work with any MongoDB installation regardless of the operating system as long as authentication has been enabled.

# Understanding Aggregation Pipelines


When working with a database management system, any time you want to retrieve data from the database you must execute an operation known as a query. However, queries only return the data that already exists in the database. In order to analyze your data to find patterns or other information about the data — rather than the data itself — you’ll often need to perform another kind of operation known as an aggregation.


Aggregations group data from multiple sources and then process that data in some way to return a single result. In a relational database, the database management system will typically pull data from multiple rows in the same table to execute an aggregate function. In a document-oriented database like MongoDB, though, the database will pull data from multiple documents in the same collection.


MongoDB enables you to perform aggregation operations through the mechanism called aggregation pipelines. These are built as a sequential series of declarative data processing operations known as stages. Each stage inspects and transforms the documents as they pass through the pipeline, feeding the transformed results into the subsequent stages for further processing. Documents from a chosen collection enter the pipeline and go through each stage, where the output coming from one stage forms the input for the next one and the final result comes at the end of the pipeline.


It may help to think of this process like vegetables going through an assembly line in a restaurant kitchen. In this analogy, vegetables go through a set of stations, each responsible for a single action: washing, peeling, chopping, cooking, and plating. Likewise, data entering an aggregation pipeline must go through a number of stages, each of which are responsible for a specific operation.


Stages can perform operations on data such as:


- filtering: This resembles  queries, where the list of documents is narrowed down through a set of criteria
- sorting: You can reorder documents based on a chosen field
- transforming: The ability to change the structure of documents means you can remove or rename certain fields, or perhaps rename or group fields within an embedded document for legibility
- grouping: You can also process multiple documents together to form a summarized result

Pipeline stages do not need to produce the same number of documents they receive. Continuing with our kitchen analogy, imagine a chopping station that takes whole vegetables and passes them along as multiple slices, or a quality control station that rejects vegetables with faults and passes over to the next station only a handful of the healthy ones. Similarly, stages can generate new documents or filter out existing ones from the collection that was entered into the start of the pipeline. Additionally, the same stage can appear more than once in an aggregation pipeline, applying multiple operations one after another.


In the following steps, you’ll prepare a test database to serve as an example data set. You’ll then learn how to use a few of the most common aggregation pipeline stages individually. Finally, you’ll combine these stages together to form a complete example pipeline.


# Step 1 — Preparing the Test Data


In order to learn how aggregation pipelines work and how to use them, this step outlines how to  open the MongoDB shell to connect to your locally-installed MongoDB instance. It also explains how to create a sample collection and insert a few sample documents into it. This guide will use this sample data to illustrate different a few of MongoDB’s different types of aggregation stages.


To create this sample collection, connect to the MongoDB shell as your administrative user. This tutorial follows the conventions of the prerequisite MongoDB security tutorial and assumes the name of this administrative user is AdminSammy and its authentication database is admin. Be sure to change these details in the following command to reflect your own setup, if different:


```
mongo -u AdminSammy -p --authenticationDatabase admin


```


Enter the password you set during installation to gain access to the shell. After providing the password, your prompt will change to a greater-than sign (>).



Note: On a fresh connection, the MongoDB shell will automatically connect to the test database by default. You can safely use this database to experiment with MongoDB and the MongoDB shell.
Alternatively, you could also switch to another database to run all of the example commands given in this tutorial. To switch to another database, run the use command followed by the name of your database:
use database_name



To understand how the aggregation pipelines work, you’ll need a collection of documents with multiple fields of different types you can filter, sort, group, and summarize in different ways. This guide will use a sample collection describing the twenty most populated cities in the world. These documents will have the same format as the following sample document, which describes the city of Tokyo:


The Tokyo document
```
{
    "name": "Tokyo",
    "country": "Japan",
    "continent": "Asia",
    "population": 37.400
}

```


This document contains the following information:


- name: the city’s name.
- country: the country where the city is located.
- continent: the continent where the city is located.
- population: the city’s population, in millions.

Run the following insertMany() method in the MongoDB shell to simultaneously create a collection named cities and insert twenty sample documents into it. These documents describe the twenty most populated cities in the world:


```
db.cities.insertMany([
    {"name": "Seoul", "country": "South Korea", "continent": "Asia", "population": 25.674 },
    {"name": "Mumbai", "country": "India", "continent": "Asia", "population": 19.980 },
    {"name": "Lagos", "country": "Nigeria", "continent": "Africa", "population": 13.463 },
    {"name": "Beijing", "country": "China", "continent": "Asia", "population": 19.618 },
    {"name": "Shanghai", "country": "China", "continent": "Asia", "population": 25.582 },
    {"name": "Osaka", "country": "Japan", "continent": "Asia", "population": 19.281 },
    {"name": "Cairo", "country": "Egypt", "continent": "Africa", "population": 20.076 },
    {"name": "Tokyo", "country": "Japan", "continent": "Asia", "population": 37.400 },
    {"name": "Karachi", "country": "Pakistan", "continent": "Asia", "population": 15.400 },
    {"name": "Dhaka", "country": "Bangladesh", "continent": "Asia", "population": 19.578 },
    {"name": "Rio de Janeiro", "country": "Brazil", "continent": "South America", "population": 13.293 },
    {"name": "São Paulo", "country": "Brazil", "continent": "South America", "population": 21.650 },
    {"name": "Mexico City", "country": "Mexico", "continent": "North America", "population": 21.581 },
    {"name": "Delhi", "country": "India", "continent": "Asia", "population": 28.514 },
    {"name": "Buenos Aires", "country": "Argentina", "continent": "South America", "population": 14.967 },
    {"name": "Kolkata", "country": "India", "continent": "Asia", "population": 14.681 },
    {"name": "New York", "country": "United States", "continent": "North America", "population": 18.819 },
    {"name": "Manila", "country": "Philippines", "continent": "Asia", "population": 13.482 },
    {"name": "Chongqing", "country": "China", "continent": "Asia", "population": 14.838 },
    {"name": "Istanbul", "country": "Turkey", "continent": "Europe", "population": 14.751 }
])


```


The output will contain a list of object identifiers assigned to the newly inserted objects.


```
Output{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("612d1e835ebee16872a109a4"),
                ObjectId("612d1e835ebee16872a109a5"),
                ObjectId("612d1e835ebee16872a109a6"),
                ObjectId("612d1e835ebee16872a109a7"),
                ObjectId("612d1e835ebee16872a109a8"),
                ObjectId("612d1e835ebee16872a109a9"),
                ObjectId("612d1e835ebee16872a109aa"),
                ObjectId("612d1e835ebee16872a109ab"),
                ObjectId("612d1e835ebee16872a109ac"),
                ObjectId("612d1e835ebee16872a109ad"),
                ObjectId("612d1e835ebee16872a109ae"),
                ObjectId("612d1e835ebee16872a109af"),
                ObjectId("612d1e835ebee16872a109b0"),
                ObjectId("612d1e835ebee16872a109b1"),
                ObjectId("612d1e835ebee16872a109b2"),
                ObjectId("612d1e835ebee16872a109b3"),
                ObjectId("612d1e835ebee16872a109b4"),
                ObjectId("612d1e835ebee16872a109b5"),
                ObjectId("612d1e835ebee16872a109b6"),
                ObjectId("612d1e835ebee16872a109b7")
        ]
}

```


You can verify that the documents were properly inserted by running the find() method on the cities collection with no arguments. This will retrieve all the documents in the collection:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("612d1e835ebee16872a109a4"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
. . .

```


With the sample data in place, you can continue on to the next step to learn how to build an aggregation pipeline using the $match stage.


# Step 2 — Using the $match Aggregation Stage


To create an aggregation pipeline, you can use can use MongoDB’s aggregate() method. This method uses a syntax that is fairly similar to the find() method used to query data in a collection, but aggregate() accepts one or more stage names as arguments. This step focuses on how to use the $match aggregation stage.


Whether you want to do light document structure processing, summarizing, or complex transformations, you’ll usually want to focus your analysis on just a selection of documents matching specific criteria. $match can be used to narrow down the list of documents at any given step of a pipeline, and can be used to ensure that all subsequent operations will be executed on a limited list of entries.


As an example, run the following operation. This will construct an aggregation pipeline using a single $match stage without any particular filtering query:


```
db.cities.aggregate([
    { $match: { } }
])


```


The aggregate() method executed on the cities collection instructs MongoDB to run an aggregation pipeline passed as the method argument. Because aggregation pipelines are multi-step processes, the argument is a list of stages, hence the use of square brackets [] denoting an array of multiple elements.


Each element inside this array is an object describing a processing stage. The stage here is written as { $match: { } }. In this document describing the processing stage, the key $match refers to the stage type, and the value { } describes its parameters. In our example, the $match stage uses the empty query document as its parameter and is the only stage in the whole processing pipeline.


Remember that $match narrows down the list of documents from the collection. With no filtering parameters applied, MongoDB will return the list of all cities from the collection:


```
Output{ "_id" : ObjectId("612d1e835ebee16872a109a4"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("612d1e835ebee16872a109a5"), "name" : "Mumbai", "country" : "India", "continent" : "Asia", "population" : 19.98 }
{ "_id" : ObjectId("612d1e835ebee16872a109a6"), "name" : "Lagos", "country" : "Nigeria", "continent" : "Africa", "population" : 13.463 }
{ "_id" : ObjectId("612d1e835ebee16872a109a7"), "name" : "Beijing", "country" : "China", "continent" : "Asia", "population" : 19.618 }
{ "_id" : ObjectId("612d1e835ebee16872a109a8"), "name" : "Shanghai", "country" : "China", "continent" : "Asia", "population" : 25.582 }
. . .

```


Next, run the aggregate() method again, but this time include a query document as a parameter to the $match stage. Any valid query document can be  used here.


You can think of using the $match stage as equivalent to querying the collection with find() as described in the How To Create Queries in MongoDB tutorial listed in the Prerequisites section. The biggest difference is that $match can be used multiple times in the aggregation pipeline, allowing you to query documents that have already been processed and transformed earlier in the pipeline. You’ll learn more about using the same stage multiple times in the same aggregation pipeline later on in this guide.


Run the following aggregate() method. This example includes a $match stage to select only cities from North America:


```
db.cities.aggregate([
    { $match: { "continent": "North America" } }
])


```


This time the { "continent": "North America" } query document appears as the parameter to the $match stage. Consequently, MongoDB returns two cities from North America:


```
Output{ "_id" : ObjectId("612d1e835ebee16872a109b0"), "name" : "Mexico City", "country" : "Mexico", "continent" : "North America", "population" : 21.581 }
{ "_id" : ObjectId("612d1e835ebee16872a109b4"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


This command returns the same output as the following one which instead uses the find() method to query the database:


```
db.cities.find({ "continent": "North America" })


```


The previous aggregate() method only returns two cities, so there isn’t much to experiment with. To return more results, alter this command so it returns cities from North America and Asia:


```
db.cities.aggregate([
    { $match: { "continent": { $in: ["North America", "Asia"] } } }
])


```


Notice that the query document syntax is once again identical to how you’d retrive the same data using the find() method. This time MongoDB returns 14 different cities:


```
Output{ "_id" : ObjectId("612d1e835ebee16872a109a4"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("612d1e835ebee16872a109a5"), "name" : "Mumbai", "country" : "India", "continent" : "Asia", "population" : 19.98 }
. . .

```


With that, you’ve learned how to execute an aggregation pipeline and using the $match stage to narrow down the collection’s documents. Continue reading to learn how to build more complex pipelines by using $sort stage to order the results and by combining multiple stages together.


# Step 3 — Using the $sort Aggregation Stage


The $match stage is useful for narrowing down the list of documents that are moved on to the next aggregation stage. However, $match does nothing to change or transform the data as it passes through the pipeline.


When querying the database, it’s common to expect a certain order when retrieving the results. Using the standard query mechanism, you can specify the document order by appending a sort() method to the end of a find() query. For example, to retrieve every city in the collection and sort them in descending order by population, you could run an operation like this:


```
db.cities.find().sort({ "population": -1 })


```


MongoDB will return each city starting with Tokyo, followed by Delhi, Seoul, and so on:


```
Output{ "_id" : ObjectId("612d1e835ebee16872a109ab"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("612d1e835ebee16872a109b1"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("612d1e835ebee16872a109a4"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
. . .

```


You can alternatively sort the documents in an aggregation pipeline by including a $sort stage. To illustrate this, run the following aggregate() method. This follows a similar syntax to the previous examples that used a $match stage:


```
db.cities.aggregate([
    { $sort: { "population": -1 } }
])


```


Again, the list of stages used in an aggregation pipeline is passed as an array of stage definitions between a pair of square brackets ([]). This example’s stage definition only includes a single $sort stage as the key, with its value being a document holding the sorting parameters. Any valid sort document can be used here.


MongoDB will return the same result set as the previous find() operation since using an aggregation pipeline with just a sorting stage is equivalent to a standard query with a sort order applied:


```
Output{ "_id" : ObjectId("612d1e835ebee16872a109ab"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("612d1e835ebee16872a109b1"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("612d1e835ebee16872a109a4"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
. . .

```


Suppose that you want to retrieve cities just from North America sorted by population in ascending order. To do so, you could apply two processing stages one after the other: the first to narrow down the result set with a filtering $match stage and then a second to apply the required ordering using a $sort stage:


```
db.cities.aggregate([
    { $match: { "continent": "North America" } },
    { $sort: { "population": 1 } }
])


```


Notice the two separate stages in the command syntax, separated by a comma within the stages array.


This time, MongoDB will return documents representing New York and Mexico City, the only two cities from North America, starting with New York as it has a lower population:


```
Output{ "_id" : ObjectId("612d1e835ebee16872a109b4"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }
{ "_id" : ObjectId("612d1e835ebee16872a109b0"), "name" : "Mexico City", "country" : "Mexico", "continent" : "North America", "population" : 21.581 }

```


To obtain these results, MongoDB first passed the document collection through the $match stage, filtered the documents against the query criteria, and then forwarded the results to the next stage in line responsible for sorting the results. Just like the $match stage, $sort can appear multiple times in the aggregation pipeline and can sort documents by any field you might need, including fields that will only appear in the document structure during the aggregation.



Note: When running filtering and sorting stages at the beginning of the aggregation pipeline, before any projection, grouping, or other transformation stages, MongoDB will use indexes to maximize the performance just like it would with standard query. You can learn more about indexes in our guide on How to Use Indexes in MongoDB.

# Step 4 — Using the $group Aggregation Stage


The $group aggregation stage is responsible for grouping and summarizing documents. It takes in multiple documents and arranges them into several separate batches based on grouping expression values and outputs a single document for each distinct batch. The output documents hold information about the group and can contain additional computed fields like sums or averages across the list of documents from the group.


To illustrate, run the following aggregate() method. This includes a $group stage that will group the resulting documents by the continent in which each city is located:


```
db.cities.aggregate([
    { $group: { "_id": "$continent" } }
])


```


In MongoDB, every document must have an _id field to be used as a primary key. Recall from Step 1 that the insertMany() method used to create the sample collection didn’t include this field in any sample documents. This is because MongoDB automatically creates this field and generates unique identification numbers in the form of ObjectId fields. For $group stages within an aggregation pipeline, though, it is required that you specify an _id field with a valid expression.


This aggregate() method, though, does specify an _id value; namely, each value found in the continent fields of each document in the cities collection. Any time you want to refer the values of a field in an aggregation pipeline like this, you must precede the name of the field with a dollar sign ($). In MongoDB, this is referred to as a field path, as it directs the operation to the appropriate field where it can find the values to be used in the pipeline stage.


Here in this example, "$continent" tells MongoDB to take the continent field from the original document and use its value to construct the expression value in the aggregation pipeline. MongoDB will output a single document for each unique value of that grouping expression:


```
Output{ "_id" : "Africa" }
{ "_id" : "Asia" }
{ "_id" : "South America" }
{ "_id" : "Europe" }
{ "_id" : "North America" }

```


This example outputs a single document for each of the five continents represented in the collection. By default, the grouping stage doesn’t include any additional fields from the original document, since it wouldn’t know how or from which document to source the other values.


You can, however, specify multiple single-field values in a grouping expression. The following example method will group documents based on the values in the continent and country documents:


```
db.cities.aggregate([
    {
        $group: {
            "_id": {
                "continent": "$continent",
                "country": "$country"
            }
        }
    }
])


```


Notice that the _id field for this example’s grouping expression uses an embedded document which, in turn, has two fields inside: one for the continent name and another for the country name. Both fields refer to fields from the original documents using the field path dollar sign notation.


This time MongoDB returns 14 results as there are 14 distinct country-continents pairs in the collection:


```
Output{ "_id" : { "continent" : "Europe", "country" : "Turkey" } }
{ "_id" : { "continent" : "South America", "country" : "Argentina" } }
{ "_id" : { "continent" : "Asia", "country" : "Bangladesh" } }
{ "_id" : { "continent" : "Asia", "country" : "Philippines" } }
{ "_id" : { "continent" : "Asia", "country" : "South Korea" } }
{ "_id" : { "continent" : "Asia", "country" : "Japan" } }
{ "_id" : { "continent" : "Asia", "country" : "China" } }
{ "_id" : { "continent" : "North America", "country" : "United States" } }
{ "_id" : { "continent" : "North America", "country" : "Mexico" } }
{ "_id" : { "continent" : "Africa", "country" : "Nigeria" } }
{ "_id" : { "continent" : "Asia", "country" : "India" } }
{ "_id" : { "continent" : "Asia", "country" : "Pakistan" } }
{ "_id" : { "continent" : "Africa", "country" : "Egypt" } }
{ "_id" : { "continent" : "South America", "country" : "Brazil" } }

```


These results aren’t ordered in any meaningful way. As you work with data more and more, you’ll likely encounter situations where you want to perform more complex data analysis. To this end, MongoDB provides a number of accumulator operators which allow you to find more granular details about your data. An accumulator operator, sometimes just referred to simply as an accumulator, is a special type of operation that maintains its value or state as it passes through an aggregation pipeline, such as a sum or average of more than one value.


To illustrate, run the following aggregate() method. This method’s $group stage creates the required _id grouping expression as well as three additional computed fields. These computed fields all include an accumulator operator and its value. Here’s a breakdown of these computed fields:


- highest_population: this field contains the maximum population value in the group. The $max accumulator operator computes the maximum value for "$population" across all documents in a group.
- first_city: contains the name of the first city in the group. The $first accumulator operator takes the value of "$name" from the first document appearing in the group. Notice that since the list of documents is now unordered, this doesn’t automatically make it the city with the highest population, but rather the first city MongoDB finds within each group.
- cities_in_top_20: holds the number of cities in the collection for each continent-country pair. To accomplish this, the $sum accumulator operator is used to compute the sum of all the pairs in the list. In this example, the sum takes one for each document and doesn’t refer to a particular field in the source document.

You can add as many additional computed fields as needed for your use case, but for now run this example query:


```
db.cities.aggregate([
    {
        $group: {
            "_id": {
                "continent": "$continent",
                "country": "$country"
            },
            "highest_population": { $max: "$population" },
            "first_city": { $first: "$name" },
            "cities_in_top_20": { $sum: 1 }
        }
    }
])


```


MongoDB returns the following 14 documents, one for each unique group defined by the grouping expression:


```
Output{ "_id" : { "continent" : "North America", "country" : "United States" }, "highest_population" : 18.819, "first_city" : "New York", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "Asia", "country" : "Philippines" }, "highest_population" : 13.482, "first_city" : "Manila", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "North America", "country" : "Mexico" }, "highest_population" : 21.581, "first_city" : "Mexico City", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "Africa", "country" : "Nigeria" }, "highest_population" : 13.463, "first_city" : "Lagos", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "Asia", "country" : "India" }, "highest_population" : 28.514, "first_city" : "Mumbai", "cities_in_top_20" : 3 }
{ "_id" : { "continent" : "Asia", "country" : "Pakistan" }, "highest_population" : 15.4, "first_city" : "Karachi", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "Africa", "country" : "Egypt" }, "highest_population" : 20.076, "first_city" : "Cairo", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "South America", "country" : "Brazil" }, "highest_population" : 21.65, "first_city" : "Rio de Janeiro", "cities_in_top_20" : 2 }
{ "_id" : { "continent" : "Europe", "country" : "Turkey" }, "highest_population" : 14.751, "first_city" : "Istanbul", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "Asia", "country" : "Bangladesh" }, "highest_population" : 19.578, "first_city" : "Dhaka", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "South America", "country" : "Argentina" }, "highest_population" : 14.967, "first_city" : "Buenos Aires", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "Asia", "country" : "South Korea" }, "highest_population" : 25.674, "first_city" : "Seoul", "cities_in_top_20" : 1 }
{ "_id" : { "continent" : "Asia", "country" : "Japan" }, "highest_population" : 37.4, "first_city" : "Osaka", "cities_in_top_20" : 2 }
{ "_id" : { "continent" : "Asia", "country" : "China" }, "highest_population" : 25.582, "first_city" : "Beijing", "cities_in_top_20" : 3 }

```


The field names in the returned documents correspond to the computed field names in the grouping stage document. To examine the results more closely, let’s narrow our focus to a single document:


A summarized document representing Japan
```
{ "_id" : { "continent" : "Asia", "country" : "Japan" }, "highest_population" : 37.4, "first_city" : "Osaka", "cities_in_top_20" : 2 }

```


The _id field holds the grouping expression values for Japan and Asia. The cities_in_top_20 field shows that two Japanese cities are on the list of the 20 most populated cities. Recall from Step 1 that you only added two documents representing cities in Japan (Tokyo and Osaka), so this value is correct. The highest_population corresponds to the population of Tokyo, which is indeed the higher population of the two.


However, the first_city shows Osaka and not Tokyo, as one might expect. That’s because the grouping stage used a list of source documents that were not ordered by population, so it couldn’t guarantee the logical meaning of “first” in that scenario. Osaka was processed by the pipeline first and hence appears in the first_city field value.


You’ll learn how to change that by strategically combining sorting and grouping stages in Step 6. For now, though, you can continue on to Step 5 which outlines how to transform the structure of documents in a pipeline using projections.



Note: In addition to the three described in this step, there are several more accumulator operators available in MongoDB that can be used for a variety of aggregations. To learn more about grouping and accumulator operators, we encourage you to follow the official MongoDB documentation.

# Step 5 — Using the $project Aggregation Stage


When working with aggregation pipelines, you’ll sometimes want to return only a few of a document collection’s multiple fields or change the structure slightly to move some fields into embedded documents. You could use this strategy to redact fields that should not be included in a report, or to prepare results in a format ready for certain application requirements.


Say, for example, that you’d like to retrieve the population for each of the cities in the sample collection, but you’d like the results to take the following format:


Required document structure
```
{
    "location" : {
        "country" : "South Korea",
        "continent" : "Asia"
    },
    "name" : "Seoul",
    "population" : 25.674
}

```


The location field contains the country and continent pair, the city’s name and population are shown in name and population fields, respectively, and the document identifier _id doesn’t appear in the outputted document.


You can use the $project stage to construct new document structures in an aggregation pipeline, thereby altering the way resulting documents appear in the result set.


To illustrate, run the following aggregate() method which includes a $project stage:


```
db.cities.aggregate([
    {
        $project: {
            "_id": 0,
            "location": {
                "country": "$country",
                "continent": "$continent",
            },
            "name": "$name",
            "population": "$population"
        }
    }
])


```


The value for this $project stage is a projection document describing the output structure. These projection documents follow the same format as those used in queries, constructed as inclusion projections or exclusion projections. The projection document keys correspond to the keys from input documents entering the $project stage.


When the projection document contains keys with 1 as their values, it describes the list of fields that will be included in the result. If, on the other hand, projection keys are set to 0, the projection document describes the list of fields that will be excluded from the result.


In an aggregation pipeline, projections can also include additional computed fields. In such cases, the projection automatically becomes an inclusion projection, and only the _id field can be suppressed by appending "_id": 0 to the projection document. Computed fields use the dollar sign field path notation for their values and can refer to the values from input documents.


In this example, the document identifier is suppressed with "_id": 0, the name and population are computed fields referring to the name and population fields from the input documents, respectively. The location field becomes an embedded document with two additional keys: country and continent, referring to fields from the input documents.


Using this projection stage, MongoDB will return the following documents:


```
Output{ "location" : { "country" : "South Korea", "continent" : "Asia" }, "name" : "Seoul", "population" : 25.674 }
{ "location" : { "country" : "India", "continent" : "Asia" }, "name" : "Mumbai", "population" : 19.98 }
{ "location" : { "country" : "Nigeria", "continent" : "Africa" }, "name" : "Lagos", "population" : 13.463 }
{ "location" : { "country" : "China", "continent" : "Asia" }, "name" : "Beijing", "population" : 19.618 }
{ "location" : { "country" : "China", "continent" : "Asia" }, "name" : "Shanghai", "population" : 25.582 }
{ "location" : { "country" : "Japan", "continent" : "Asia" }, "name" : "Osaka", "population" : 19.281 }
{ "location" : { "country" : "Egypt", "continent" : "Africa" }, "name" : "Cairo", "population" : 20.076 }
{ "location" : { "country" : "Japan", "continent" : "Asia" }, "name" : "Tokyo", "population" : 37.4 }
{ "location" : { "country" : "Pakistan", "continent" : "Asia" }, "name" : "Karachi", "population" : 15.4 }
{ "location" : { "country" : "Bangladesh", "continent" : "Asia" }, "name" : "Dhaka", "population" : 19.578 }
{ "location" : { "country" : "Brazil", "continent" : "South America" }, "name" : "Rio de Janeiro", "population" : 13.293 }
{ "location" : { "country" : "Brazil", "continent" : "South America" }, "name" : "São Paulo", "population" : 21.65 }
{ "location" : { "country" : "Mexico", "continent" : "North America" }, "name" : "Mexico City", "population" : 21.581 }
{ "location" : { "country" : "India", "continent" : "Asia" }, "name" : "Delhi", "population" : 28.514 }
{ "location" : { "country" : "Argentina", "continent" : "South America" }, "name" : "Buenos Aires", "population" : 14.967 }
{ "location" : { "country" : "India", "continent" : "Asia" }, "name" : "Kolkata", "population" : 14.681 }
{ "location" : { "country" : "United States", "continent" : "North America" }, "name" : "New York", "population" : 18.819 }
{ "location" : { "country" : "Philippines", "continent" : "Asia" }, "name" : "Manila", "population" : 13.482 }
{ "location" : { "country" : "China", "continent" : "Asia" }, "name" : "Chongqing", "population" : 14.838 }
{ "location" : { "country" : "Turkey", "continent" : "Europe" }, "name" : "Istanbul", "population" : 14.751 }

```


Each document now follows the new format transformed through the projection stage.


Now that you’ve learned how to use $project stage to construct a new document structure for the documents going through an aggregation pipeline, you’re ready to combine all the pipeline stages covered throughout this guide in a single aggregation pipeline.


# Step 6 — Putting All the Stages Together


You’re now ready to join together all the stages that you’ve practiced using in the previous steps to form a fully functional aggregation pipeline that both filters and transforms documents.


Suppose the task at hand is to find the most populated city for each country in country in Asia and North America and return both its name and population. The results should be sorted by the highest population, returning countries with the largest cities first, and you are interested only in countries where the most populated city crosses the threshold of 20 million people. Lastly, the document structure you aim for should replicate the following:


Example document
```
{
    "location" : {
        "country" : "Japan",
        "continent" : "Asia"
    },
    "most_populated_city" : {
        "name" : "Tokyo",
        "population" : 37.4
    }
}

```


To illustrate how to retrieve a data set that would satisfy these requirements, this step outlines how to build the appropriate aggregation pipeline.


Begin by running the following query, which filters the initial documents coming from the cities collection so the result set will only contain countries in Asia and North America. Even though it would be possible to narrow down the document selection later, doing so upfront will optimize the pipeline’s efficiency. After all, limiting the number of documents to be processed early will minimize the amount of processing needed in later stages:


```
db.cities.aggregate([
    {
        $match: {
            "continent": { $in: ["North America", "Asia"] }
        }
    }
])


```


This pipeline’s $match stage will only find cities in North America and Asia, and the documents representing these cities will be returned in their full original structure and with default ordering:


```
Output{ "_id" : ObjectId("612d1e835ebee16872a109a4"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("612d1e835ebee16872a109a5"), "name" : "Mumbai", "country" : "India", "continent" : "Asia", "population" : 19.98 }
{ "_id" : ObjectId("612d1e835ebee16872a109a7"), "name" : "Beijing", "country" : "China", "continent" : "Asia", "population" : 19.618 }
{ "_id" : ObjectId("612d1e835ebee16872a109a8"), "name" : "Shanghai", "country" : "China", "continent" : "Asia", "population" : 25.582 }
. . .

```


In Step 4, you learned that passing an unordered list of documents to the grouping stage can have unexpected results if you need to access fields from the first document in the group. You must do this to find the name of the most populated city later, so to head off this problem you can order the cities from the highest to the lowest population by following the $match stage with a $sort stage:


```
db.cities.aggregate([
    {
        $match: {
            "continent": { $in: ["North America", "Asia"] }
        }
    },
    {
        $sort: { "population": -1 }
    }
])


```


The second pipeline stage in this aggregate() method tells MongoDB to order the documents by population in descending order, as indicated by the { "population": -1 } sorting document.


Once again the returned documents have the same structure, but this time Tokyo comes first since it has the highest population:


```
Output{ "_id" : ObjectId("612d1e835ebee16872a109ab"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
. . .

```


You now have the list of cities sorted by the population coming from the expected continents, so the next necessary action for this scenario is to group cities by their countries, choosing only the most populated city from each group. To do so, add a $group stage to the pipeline:


```
db.cities.aggregate([
    {
        $match: {
            "continent": { $in: ["North America", "Asia"] }
        }
    },
    {
        $sort: { "population": -1 }
    },
    {
        $group: {
            "_id": {
                "continent": "$continent",
                "country": "$country"
            },
            "first_city": { $first: "$name" },
            "highest_population": { $max: "$population" }
        }
    }
])


```


This new stage’s grouping expression tells MongoDB to group cities by unique continent and country pairs. For each group, two computed values summarize the groups. The highest_population value uses the $max accumulator operator to find the highest population in the group. The first_city gets the name of the first city in the group of documents. Thanks to the previously applied sorting stage, you can be sure this first city will also the most populated city in the group and that it will match the numerical population value.


Adding this $group stage changes the number of documents returned by this method as well as their structure. This time, the method only returns nine documents, as there are only nine unique country and continent pairs in the previously filtered cities list. Each document corresponds to one of these pairs, and consists of the grouping expression value in the _id field and two computed fields:


```
Output{ "_id" : { "continent" : "North America", "country" : "United States" }, "first_city" : "New York", "highest_population" : 18.819 }
{ "_id" : { "continent" : "Asia", "country" : "China" }, "first_city" : "Shanghai", "highest_population" : 25.582 }
{ "_id" : { "continent" : "Asia", "country" : "Japan" }, "first_city" : "Tokyo", "highest_population" : 37.4 }
{ "_id" : { "continent" : "Asia", "country" : "South Korea" }, "first_city" : "Seoul", "highest_population" : 25.674 }
{ "_id" : { "continent" : "Asia", "country" : "Bangladesh" }, "first_city" : "Dhaka", "highest_population" : 19.578 }
{ "_id" : { "continent" : "Asia", "country" : "Philippines" }, "first_city" : "Manila", "highest_population" : 13.482 }
{ "_id" : { "continent" : "Asia", "country" : "India" }, "first_city" : "Delhi", "highest_population" : 28.514 }
{ "_id" : { "continent" : "Asia", "country" : "Pakistan" }, "first_city" : "Karachi", "highest_population" : 15.4 }
{ "_id" : { "continent" : "North America", "country" : "Mexico" }, "first_city" : "Mexico City", "highest_population" : 21.581 }

```


Notice that the resulting documents for each group are not ordered by the population value. New York comes first, but the second city — Shanghai — has a population of almost 7 million people more. Also, several countries have the most populated cities below the expected threshold of 20 million.


Remember that filtering and sorting stages can appear multiple times in the pipeline. Also, for each aggregation stage, the last stage’s output is the next stage’s input. Use another $match stage to filter the groups to contain only countries with the cities satisfying population minimum of 20 million:


```
db.cities.aggregate([
    {
        $match: {
            "continent": { $in: ["North America", "Asia"] }
        }
    },
    {
        $sort: { "population": -1 }
    },
    {
        $group: {
            "_id": {
                "continent": "$continent",
                "country": "$country"
            },
            "first_city": { $first: "$name" },
            "highest_population": { $max: "$population" }
        }
    },
    {
        $match: {
            "highest_population": { $gt: 20.0 }
        }
    }
])


```


This filtering $match stage refers to the highest_population field available in the documents coming from the grouping stage, even though such a field is not part of the structure of the original documents.


This time, five countries appear in the output:


```
Output{ "_id" : { "continent" : "Asia", "country" : "China" }, "first_city" : "Shanghai", "highest_population" : 25.582 }
{ "_id" : { "continent" : "Asia", "country" : "Japan" }, "first_city" : "Tokyo", "highest_population" : 37.4 }
{ "_id" : { "continent" : "Asia", "country" : "South Korea" }, "first_city" : "Seoul", "highest_population" : 25.674 }
{ "_id" : { "continent" : "Asia", "country" : "India" }, "first_city" : "Delhi", "highest_population" : 28.514 }
{ "_id" : { "continent" : "North America", "country" : "Mexico" }, "first_city" : "Mexico City", "highest_population" : 21.581 }

```


Next, sort the results according by their highest_population value. To do so, add another $sort stage:


```
db.cities.aggregate([
    {
        $match: {
            "continent": { $in: ["North America", "Asia"] }
        }
    },
    {
        $sort: { "population": -1 }
    },
    {
        $group: {
            "_id": {
                "continent": "$continent",
                "country": "$country"
            },
            "first_city": { $first: "$name" },
            "highest_population": { $max: "$population" }
        }
    },
    {
        $match: {
            "highest_population": { $gt: 20.0 }
        }
    },
    {
        $sort: { "highest_population": -1 }
    }
])


```


The document structure doesn’t change, and MongoDB still returns five documents corresponding to the country groups. This time, however, Japan appears first since Tokyo is the most populated city of all in the data set:


```
Output{ "_id" : { "continent" : "Asia", "country" : "Japan" }, "first_city" : "Tokyo", "highest_population" : 37.4 }
{ "_id" : { "continent" : "Asia", "country" : "India" }, "first_city" : "Delhi", "highest_population" : 28.514 }
{ "_id" : { "continent" : "Asia", "country" : "South Korea" }, "first_city" : "Seoul", "highest_population" : 25.674 }
{ "_id" : { "continent" : "Asia", "country" : "China" }, "first_city" : "Shanghai", "highest_population" : 25.582 }
{ "_id" : { "continent" : "North America", "country" : "Mexico" }, "first_city" : "Mexico City", "highest_population" : 21.581 }

```


The last requirement is to transform the document structure to match the sample shown previously. For your review, here’s that sample once more:


Example document
```
{
    "location" : {
        "country" : "Japan",
        "continent" : "Asia"
    },
    "most_populated_city" : {
        "name" : "Tokyo",
        "population" : 37.4
    }
}

```


This sample’s location embedded document resembles the _id grouping expression value, as both include country and continent fields. The most populated city name and population are nested as an embedded document under the most_populated_city field. This is different from the grouping results, where all computed fields are top-level fields.


To transform the results to align with this structure, add a $project stage to the pipeline:


```
db.cities.aggregate([
    {
        $match: {
            "continent": { $in: ["North America", "Asia"] }
        }
    },
    {
        $sort: { "population": -1 }
    },
    {
        $group: {
            "_id": {
                "continent": "$continent",
                "country": "$country"
            },
            "first_city": { $first: "$name" },
            "highest_population": { $max: "$population" }
        }
    },
    {
        $match: {
            "highest_population": { $gt: 20.0 }
        }
    },
    {
        $sort: { "highest_population": -1 }
    },
    {
        $project: {
            "_id": 0,
            "location": {
                "country": "$_id.country",
                "continent": "$_id.continent",
            },
            "most_populated_city": {
                "name": "$first_city",
                "population": "$highest_population"
            }
        }
    }
])


```


This $project stage first suppresses the _id field from appearing in the output. Then it creates a location field as a nested document containing two fields: country and continent. Using the dollar-sign notation, each of these fields refers to values from the input documents. "$_id.country" pulls values from the country field from inside the _id embedded document of the input, and $_id.continent pulls values from its continent field.  most_populated_city follows a similar structure, nesting the name and population fields inside. These refer to the top-level fields first_city and highest_population, respectively.


This projection stage effectively constructs an entirely new structure for the output, which is as follows:


```
Output{ "location" : { "country" : "Japan", "continent" : "Asia" }, "most_populated_city" : { "name" : "Tokyo", "population" : 37.4 } }
{ "location" : { "country" : "India", "continent" : "Asia" }, "most_populated_city" : { "name" : "Delhi", "population" : 28.514 } }
{ "location" : { "country" : "South Korea", "continent" : "Asia" }, "most_populated_city" : { "name" : "Seoul", "population" : 25.674 } }
{ "location" : { "country" : "China", "continent" : "Asia" }, "most_populated_city" : { "name" : "Shanghai", "population" : 25.582 } }
{ "location" : { "country" : "Mexico", "continent" : "North America" }, "most_populated_city" : { "name" : "Mexico City", "population" : 21.581 } }

```


This output meets all the requirements defined at the beginning of this step:


- It only includes cities from Asia and North America in the lists.
- For each country and continent pair, a single city is selected, and it’s the city with the highest population.
- The selected city’s name and population are listed.
- Cities are sorted from the most populated to least populated.
- The output format is altered to align with the example document.

# Conclusion


In this article, you familiarized yourself with aggregation pipelines, a MongoDB feature for multi-step document processing including filtering, sorting, summarization, and transformation. You’ve used $match, $sort, $group, and $project aggregation stages one-by-one and jointly to execute an example reporting scenario processing input documents to present aggregated data. You’ve also familiarized yourself with computed fields and accumulator operators to find maximums and sums from groups of documents.


This tutorial describes only a small subset of aggregation pipeline features provided by MongoDB to process and transform data. There are more processing stages available, and each of the stages described in this article can be used in additional and different ways. We encourage you to study the official official MongoDB documentation to learn more about aggregation pipelines and how they can help you work with data stored in the database.


