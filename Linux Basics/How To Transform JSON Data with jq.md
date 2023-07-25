# How To Transform JSON Data with jq

```Linux Basics``` ```Linux Commands``` ```System Tools```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


When working with big JSON files, it can be hard to find and manipulate the information you need. You could copy and paste all relevant snippets to calculate totals manually, but this is a time-consuming process and could be prone to human error. Another option is to use general-purpose tools for finding and manipulating information. All modern Linux systems come installed with three established text processing utilities: sed, awk, and grep. While these commands are helpful when working with loosely structured data, other options exist for machine-readable data formats like JSON.


jq, a command-line JSON processing tool, is a good solution for dealing with machine-readable data formats and is especially useful in shell scripts. Using jq can aid you when you need to manipulate data. For example, if you run a curl call to a JSON API, jq can extract specific information from the server’s response. You could also incorporate jq into your data ingestion process as a data engineer. If you manage a Kubernetes cluster, you could use the JSON output of kubectl as an input source for jq to extract the number of available replicas for a specific deployment.


In this article, you will use jq to transform a sample JSON file about ocean animals. You’ll apply data transformations using filters and merge pieces of transformed data into a new data structure. By the end of the tutorial, you will be able to use a jq script to answer questions about the data you have manipulated.


# Prerequisites


To complete this tutorial, you will need the following:


- jq, a JSON parsing and transformation tool. It is available from the repositories for all major Linux distributions. If you are using Ubuntu, run sudo apt install jq to install it.
- An understanding of JSON syntax, which you can refresh in An Introduction to JSON.

# Step 1 — Executing Your First jq Command


In this step, you will set up your sample input file and test the setup by running a jq command to generate an output of the sample file’s data. jq can take input from either a file or a pipe. You will use the former.


You’ll begin by generating the sample file. Create and open a new file named seaCreatures.json using your preferred editor (this tutorial uses nano):


```
nano seaCreatures.json


```


Copy the following contents into the file:


seaCreatures.json
```
[
    { "name": "Sammy", "type": "shark", "clams": 5 },
    { "name": "Bubbles", "type": "orca", "clams": 3 },
    { "name": "Splish", "type": "dolphin", "clams": 2 },
    { "name": "Splash", "type": "dolphin", "clams": 2 }
]

```


You’ll work with this data for the rest of the tutorial. By the end of the tutorial, you will have written a one-line jq command that answers the following questions about this data:


- What are the names of the sea creatures in list form?
- How many clams do the creatures own in total?
- How many of those clams are owned by dolphins?

Save and close the file.


In addition to an input file, you will need a filter that describes the exact transformation you’d like to do. The . (period) filter, also known as the identity operator, passes the JSON input unchanged as output.


You can use the identity operator to test whether your setup works. If you see any parse errors, check that seaCreatures.json contains valid JSON.


Apply the identity operator to the JSON file with the following command:


```
jq '.' seaCreatures.json


```


When using jq with files, you always pass a filter followed by the input file. Since filters may contain spacing and other characters that hold a special meaning to your shell, it is a good practice to wrap your filter in single quotation marks. Doing so tells your shell that the filter is a command parameter. Rest assured that running jq will not modify your original file.


You’ll receive the following output:


```
Output[
  {
    "name": "Sammy",
    "type": "shark",
    "clams": 5
  },
  {
    "name": "Bubbles",
    "type": "orca",
    "clams": 3
  },
  {
    "name": "Splish",
    "type": "dolphin",
    "clams": 2
  },
  {
    "name": "Splash",
    "type": "dolphin",
    "clams": 2
  }
]

```


By default, jq will pretty print its output. It will automatically apply indentation, add new lines after every value, and color its output when possible. Coloring may improve readability, which can help many developers as they examine JSON data produced by other tools. For example, when sending a curl request to a JSON API, you may want to pipe the JSON response into jq '.' to pretty print it.


You now have jq up and running. With your input file set up, you’ll manipulate the data using a few different filters in order to compute the values of all three attributes: creatures, totalClams, and totalDolphinClams. In the next step, you’ll find the information from the creatures value.


# Step 2 — Retrieving the creatures Value


In this step, you will generate a list of all sea creatures, using the creatures value to find their names. At the end of this step, you will have generated the following list of names:


```
Output[
  "Sammy",
  "Bubbles",
  "Splish",
  "Splash"
],

```


Generating this list requires extracting the names of the creatures and then merging them into an array.


You’ll have to refine your filter to get the names of all creatures and discard everything else. Since you’re working on an array, you’ll need to tell jq you want to operate on the values of that array instead of the array itself. The array value iterator, written as .[], serves this purpose.


Run jq with the modified filter:


```
jq '.[]' seaCreatures.json


```


Every array value is now output separately:


```
Output{
  "name": "Sammy",
  "type": "shark",
  "clams": 5
}
{
  "name": "Bubbles",
  "type": "orca",
  "clams": 3
}
{
  "name": "Splish",
  "type": "dolphin",
  "clams": 2
}
{
  "name": "Splash",
  "type": "dolphin",
  "clams": 2
}

```


Instead of outputting every array item in full, you’ll want to output the value of the name attribute and discard the rest. The pipe operator | will allow you to apply a filter to each output. If you have used find | xargs on the command line to apply a command to every search result, this pattern will feel familiar.


A JSON object’s name property can be accessed by writing .name. Combine the pipe with the filter and run this command on seaCreatures.json:


```
jq '.[] | .name' seaCreatures.json


```


You’ll notice that the other attributes have disappeared from the output:


```
Output"Sammy"
"Bubbles"
"Splish"
"Splash"

```


By default, jq outputs valid JSON, so strings will appear in double quotation marks (""). If you need the string without double quotes, add the -r flag to enable raw output:


```
jq -r '.[] | .name' seaCreatures.json


```


The quotation marks have disappeared:


```
OutputSammy
Bubbles
Splish
Splash

```


You now know how to extract specific information from the JSON input. You’ll use this technique to find other specific information in the next step and then to generate the creatures value in the final step.


# Step 3 — Computing the totalClams Value with map and add


In this step, you’ll find the total information for how many clams the creatures own. You can calculate the answer by aggregating a few pieces of data. Once you’re familiar with jq, this will be faster than manual calculations and less prone to human error. The expected value at the end of this step is 12.


In Step 2, you extracted specific bits of information from a list of items. You can reuse this technique to extract the values of the clams attribute. Adjust the filter for this new attribute and run the command:


```
jq '.[] | .clams' seaCreatures.json


```


The individual values of the clams attribute will be output:


```
Output5
3
2
2

```


To find the sum of individual values, you will need the add filter. The add filter works on arrays. However, you are currently outputting array values, so you must wrap them in an array first.


Surround your existing filter with [] as follows:


```
jq '[.[] | .clams]' seaCreatures.json


```


The values will appear in a list:


```
Output[
  5,
  3,
  2,
  2
]

```


Before applying the add filter, you can improve the readability of your command with the map function, which also makes it easier to maintain. Iterating over an array, applying a filter to each of those items, and then wrapping the results in an array can be achieved with one map invocation. Given an array of items, map will apply its argument as a filter to each item. For example, if you apply the filter map(.name) to [{"name": "Sammy"}, {"name": "Bubbles"}], the resulting JSON object will be ["Sammy", "Bubbles"].


Rewrite the filter to generate an array to use a map function instead, then run it:


```
jq 'map(.clams)' seaCreatures.json


```


You will receive the same output as before:


```
Output[
  5,
  3,
  2,
  2
]

```


Since you have an array now, you can pipe it into the add filter:


```
jq 'map(.clams) | add' seaCreatures.json


```


You’ll receive a sum of the array:


```
Output12

```


With this filter, you have calculated the total number of clams, which you’ll use to generate the totalClams value later. You’ve written filters for two out of three questions. You have one more filter to create, after which you can generate the final output.


# Step 4 — Computing the totalDolphinClams Value with the add Filter


Now that you know how many clams the creatures own, you can identify how many of those clams the dolphins have. You can generate the answer by adding only the values of array elements that satisfy a specific condition. The expected value at the end of this step is 4, which is the total number of clams the dolphins have. In the final step, the resulting value will be used by the totalDolphinClams attribute.


Instead of adding all clams values as you did in Step 3, you’ll count only clams held by creatures with the "dolphin" type. You’ll use the select function to select a specific condition: select(condition). Any input for which the condition evaluates to true is passed on. All other input is discarded. If, for example, your JSON input is "dolphin" and your filter is select(. == "dolphin"), the output would be "dolphin". For the input "Sammy", the same filter would output nothing.


To apply select to every value in an array, you can pair it with map. In doing so, array values that don’t satisfy the condition will be discarded.


In your case, you only want to retain array values whose type value equals "dolphin". The resulting filter is:


```
jq 'map(select(.type == "dolphin"))' seaCreatures.json


```


Your filter will not match Sammy the shark and Bubbles the orca, but it will match the two dolphins:


```
Output[
  {
    "name": "Splish",
    "type": "dolphin",
    "clams": 2
  },
  {
    "name": "Splash",
    "type": "dolphin",
    "clams": 2
  }
]

```


This output contains the number of clams per creature, as well as some information that isn’t relevant. To retain only the clams value, you can append the name of the field to the end of map’s parameter:


```
jq 'map(select(.type == "dolphin").clams)' seaCreatures.json


```


The map function receives an array as input and will apply map’s filter (passed as an argument) to each array element. As a result, select gets called four times, once per creature. The select function will produce output for the two dolphins (as they match the condition) and omit the rest.


Your output will be an array containing only the clams values of the two matching creatures:


```
Output[
  2,
  2
]

```


Pipe the array values into add:


```
jq 'map(select(.type == "dolphin").clams) | add' seaCreatures.json


```


Your output will return the sum of the clams values from creatures of the "dolphin" type:


```
Output4

```


You’ve successfully combined map and select to access an array, select array items matching a condition, transform them, and sum the result of that transformation. You can use this strategy to calculate totalDolphinClams in the final output, which you will do in the next step.


# Step 5 — Transforming Data to a New Data Structure


In the previous steps, you wrote filters to extract and manipulate the sample data. Now, you can combine these filters to generate an output that answers your questions about the data:


- What are the names of the sea creatures in list form?
- How many clams do the creatures own in total?
- How many of those clams are owned by dolphins?

To find the names of the sea creatures in list form, you used the map function: map(.name). To find how many clams the creatures own in total, you piped all clams values into the add filter: map(.clams) | add. To find how many of those clams are owned by dolphins, you used the select function with the .type == "dolphin" condition: map(select(.type == "dolphin").clams) | add.


You’ll combine these filters into one jq command that does all of the work. You will create a new JSON object that merges the three filters in order to create a new data structure that displays the information you desire.


As a reminder, your starting JSON file matches the following:


seaCreatures.json
```
[
    { "name": "Sammy", "type": "shark", "clams": 5 },
    { "name": "Bubbles", "type": "orca", "clams": 3 },
    { "name": "Splish", "type": "dolphin", "clams": 2 },
    { "name": "Splash", "type": "dolphin", "clams": 2 }
]

```


Your transformed JSON output will generate the following:


```
Final Output{
  "creatures": [
    "Sammy",
    "Bubbles",
    "Splish",
    "Splash"
  ],
  "totalClams": 12,
  "totalDolphinClams": 4
}

```


Here is a demonstration of the syntax for the full jq command with empty input values:


```
jq '{ creatures: [], totalClams: 0, totalDolphinClams: 0 }' seaCreatures.json


```


With this filter, you create a JSON object containing three attributes:


```
Output{
  "creatures": [],
  "totalClams": 0,
  "totalDolphinClams": 0
}

```


That’s starting to look like the final output, but the input values are not correct because they have not been pulled from your seaCreatures.json file.


Replace the hard-coded attribute values with the filters you created in each prior step:


```
jq '{ creatures: map(.name), totalClams: map(.clams) | add, totalDolphinClams: map(select(.type == "dolphin").clams) | add }' seaCreatures.json


```


The above filter tells jq to create a JSON object containing:


- A creatures attribute containing a list of every creature’s name value.
- A totalClams attribute containing a sum of every creature’s clams value.
- A totalDolphinClams attribute containing a sum of every creature’s clams value for which type equals "dolphin".

Run the command, and the output of this filter should be:


```
Output{
  "creatures": [
    "Sammy",
    "Bubbles",
    "Splish",
    "Splash"
  ],
  "totalClams": 12,
  "totalDolphinClams": 4
}

```


You now have a single JSON object providing relevant data for all three questions. Should the dataset change, the jq filter you wrote will allow you to re-apply the transformations at any time.


# Conclusion


When working with JSON input, jq can help you perform a wide variety of data transformations that would be difficult with text manipulation tools like sed. In this tutorial, you filtered data with the select function, transformed array elements with map, summed arrays of numbers with the add filter, and learned how to merge transformations into a new data structure.


To learn about jq advanced features, dive into the jq reference documentation. If you often work with non-JSON command output, you can explore our guides on sed, awk or grep for information on text processing techniques that will work on any format.


