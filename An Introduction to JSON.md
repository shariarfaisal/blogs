# An Introduction to JSON

```JavaScript``` ```Development``` ```Conceptual```

## Introduction


JSON, short for JavaScript Object Notation, is a format for sharing data. As its name suggests, JSON is derived from the JavaScript programming language, but it’s available for use by many languages including Python, Ruby, PHP, and Java. JSON is usually pronounced like the name “Jason.”


JSON is also readable, lightweight, offers a good alternative to XML, and requires much less formatting. This informational guide will discuss the data you can use in JSON files and the general structure and syntax of this format.


# Understanding Syntax and Structure


JSON uses the .json extension when it stands alone, and when it’s defined in another file format (as in .html), it can appear inside of quotes as a JSON string, or it can be an object assigned to a variable. This format transmits between web server and client or browser.


A JSON object is a key-value data format that is typically rendered in curly braces. When you’re working with JSON, you’ll likely come across JSON objects in a .json file, but they can also exist as a JSON object or string within the context of a program.


Here is an example of a JSON object:


```
{
  "first_name" : "Sammy",
  "last_name" : "Shark",
  "location" : "Ocean",
  "online" : true,
  "followers" : 987 
}

```


Although this is a short example, and JSON can be many lines long, this demonstrates that the format is generally set up with two curly braces (or curly brackets) that are represented with { } on either end of it, and with key-value pairs populating the space between. Most data used in JSON ends up being encapsulated in a JSON object.


Key-value pairs have a colon between them as in "key" : "value". Each key-value pair is separated by a comma, so the middle of a JSON lists as follows: "key" : "value", "key" : "value", "key": "value". In the previous example, the first key-value pair is "first_name" : "Sammy".


JSON keys are on the left side of the colon. They need to be wrapped in double quotation marks, as in "key", and can be any valid string. Within each object, keys need to be unique. These key strings can include whitespaces, as in "first name", but that can make it harder to access when you’re programming, so it’s best to use underscores, as in "first_name".


JSON values are found to the right of the colon. At the granular level, these need to be one of following six data types:


- strings
- numbers
- objects
- arrays
- Booleans (true or false)
- null

At the broader level, values can also be made up of the complex data types of JSON object or array, which is discussed in the next section.


Each of the data types that are passed in as values into JSON will maintain their own syntax, meaning strings will be in quotes, but numbers will not be.


With .json files, you’ll typically get a format expanded over multiple lines, but JSON can also be written all in one line, as in the following example:


```
{ "first_name" : "Sammy", "last_name": "Shark",  "online" : true, }

```


This is more common within another file type or when you encounter a JSON string.


Writing the JSON format on multiple lines often makes it much more readable, especially when dealing with a large data set. Because JSON ignores whitespace between its elements, you can space out your colons and key-value pairs in order to make the data even more human readable:


```
{ 
  "first_name"  :  "Sammy", 
  "last_name"   :  "Shark", 
  "online"      :  true 
}

```


It is important to keep in mind that though they appear similar, a JSON object is not the same format as a JavaScript object, so though you can use functions within JavaScript objects, you cannot use them as values in JSON. The most important attribute of JSON is that it can be readily transferred between programming languages in a format that all of the participating languages can work with. In contrast, JavaScript objects can only be worked with directly through the JavaScript programming language.


JSON can become increasingly complex with hierarchies that are comprised of nested objects and arrays. Next, you’ll learn more about these complex structures.


# Working with Complex Types in JSON


JSON can store nested objects in JSON format in addition to nested arrays. These objects and arrays will be passed as values assigned to keys, and may be comprised of key-value pairs as well.


## Nested Objects


In the following users.json file, for each of the four users ("sammy", "jesse", "drew", "jamie") there is a nested JSON object passed as the value for each of them, with its own nested keys of "username" and "location" that relate to each of the users. Each user entry in the following code block is an example of a nested JSON object:


users.json
```
{ 
  "sammy" : {
    "username"  : "SammyShark",
    "location"  : "Indian Ocean",
    "online"    : true,
    "followers" : 987
  },
  "jesse" : {
    "username"  : "JesseOctopus",
    "location"  : "Pacific Ocean",
    "online"    : false,
    "followers" : 432
  },
  "drew" : {
    "username"  : "DrewSquid",
    "location"  : "Atlantic Ocean",
    "online"    : false,
    "followers" : 321
  },
  "jamie" : {
    "username"  : "JamieMantisShrimp",
    "location"  : "Pacific Ocean",
    "online"    : true,
    "followers" : 654
  }
}

```


In this example, curly braces are used throughout to form a nested JSON object with associated username and location data for each of the four users. As with any other value, when using objects, commas are used to separate elements.


## Nested Arrays


Data can also be nested within the JSON format by using JavaScript arrays that are passed as a value. JavaScript uses square brackets [ ] on either end of its array type. Arrays are ordered collections and can contain values of different data types.


For example, you may use an array when dealing with a lot of data that can be grouped together, like when there are various websites and social media profiles associated with a single user.


With the first nested array, a user profile for "Sammy" may be represented as follows:


user_profile.json
```
{ 
  "first_name" : "Sammy",
  "last_name" : "Shark",
  "location" : "Ocean",
  "websites" : [
    {
      "description" : "work",
      "URL" : "https://www.digitalocean.com/"
    },
    {
      "desciption" : "tutorials",
      "URL" : "https://www.digitalocean.com/community/tutorials"
    }
  ],
  "social_media" : [
    {
      "description" : "twitter",
      "link" : "https://twitter.com/digitalocean"
    },
    {
      "description" : "facebook",
      "link" : "https://www.facebook.com/DigitalOceanCloudHosting"
    },
    {
      "description" : "github",
      "link" : "https://github.com/digitalocean"
    }
  ]
}

```


The "websites" key and "social_media" key each use an array to nest information belonging to Sammy’s two website links and three social media profile links. You can identify that those are arrays because of the use of square brackets.


Using nesting within your JSON format allows you to work with more complicated and hierarchical data.


# Comparing JSON to XML


XML, or eXtensible Markup Language, is a way to store accessible data that can be read by both humans and machines. The XML format is available for use across many programming languages.


In many ways, XML is similar to JSON, but it requires much more text, making it longer and more time-consuming to read and write. XML must also be parsed with an XML parser, but JSON can be parsed with a standard function. Also, unlike JSON, XML cannot use arrays.


Here’s an example of the XML format:


users.xml
```
<users>
    <user>
        <username>SammyShark</username> <location>Indian Ocean</location>
    </user>
    <user>
        <username>JesseOctopus</username> <location>Pacific Ocean</location>
    </user>
    <user>
        <username>DrewSquir</username> <location>Atlantic Ocean</location>
    </user>
    <user>
        <username>JamieMantisShrimp</username> <location>Pacific Ocean</location>
    </user>
</users>

```


Now, compare the same data rendered in JSON:


users.json
```
{"users": [
  {"username" : "SammyShark", "location" : "Indian Ocean"},
  {"username" : "JesseOctopus", "location" : "Pacific Ocean"},
  {"username" : "DrewSquid", "location" : "Atlantic Ocean"},
  {"username" : "JamieMantisShrimp", "location" : "Pacific Ocean"}
] }

```


JSON is much more compact and does not require end tags while XML does. Additionally, XML is not making use of an array as this example of JSON does (which you can tell through the use of square brackets).


If you are familiar with HTML, you’ll notice that XML is quite similar in its use of tags. While JSON is leaner and less verbose than XML and quick to use in many situations, including AJAX applications, you first want to understand the type of project you’re working on before deciding what data structures to use.


# Conclusion


JSON is a lightweight format that enables you to share, store, and work with data. As a format, JSON has been experiencing increased support in APIs, including the Twitter API. JSON is also a natural format to use in JavaScript and has many implementations available for use in various popular programming languages. You can read the full language support on the “Introducing JSON” site.


Because you likely won’t be creating your own .json files but procuring them from other sources, it is important to think less about JSON’s structure and more about how to best use JSON in your programs. For example, you can convert CSV or tab-delimited data that you may find in spreadsheet programs into JSON by using the open-source tool Mr. Data Converter. You can also convert XML to JSON and vice versa with the Creative Commons-licensed utilities-online.info site.


Finally, when translating other data types to JSON, or creating your own, you can validate your JSON with JSONLint, and test your JSON in a web development context with JSFiddle.


