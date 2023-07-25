# Arrays in Shell Scripts

```UNIX/Linux```

Knowing how to work with arrays in shell scripts will help you work with larger datasets in a much efficient manner. But what are arrays and how can you create arrays? Let’s find out!


# What are Arrays?


If you already have a basic understanding of any programming language, you know what arrays are. But for the uninitiated, let’s go over the basics of arrays and learn how to work with them.


Variables store single data elements. Arrays, on the other hand, can store a virtually unlimited number of data elements. When working with a large amount of data, variables can prove to be very inefficient and it’s very helpful to get hands-on with arrays.


Let’s learn how to create arrays in shell scripts.


# Creating Arrays in Shell Scripts


There are two types of arrays that we can work with, in shell scripts.


- Indexed Arrays - Store elements with an index starting from 0
- Associative Arrays - Store elements in key-value pairs

The default array that’s created is an indexed array. If you specify the index names, it becomes an associative array and the elements can be accessed using the index names instead of numbers.


Declaring Arrays:


```
root@ubuntu:~# declare -A assoc_array
root@ubuntu:~# assoc_array[key]=value
OR
root@ubuntu:~# declare -a indexed_array
root@ubuntu:~# indexed_array[0]=value

```


Notice the uppercase and lowercase letter a. Uppercase A is used to declare an associative array while lowercase a is used to declare an indexed array.


The declare keyword is used to explicitly declare arrays but you do not really need to use them. When you’re creating an array, you can simply initialize the values based on the type of array you want without explicitly declaring the arrays.


# Working with Arrays in Shell Scripts


Now that you know how to create arrays, let’s learn how to work with arrays. Since these are collections of data elements, we can work with loops and arrays at the same time to extract the required data points.


## 1. Accessing Array Elements Individually


Since we know that each data point is being indexed individually, we can access all the array elements by specifying the array index as shown below:


```
assoc_array[element1]="Hello World"
echo ${assoc_array[element1]}

```


Associative Arrays In Shell Script
Similarly, let’s access some indexed array elements. We can specify all the elements for the index array by delimiting with spaces because the index is automatically generated for each of those elements.


```
index_array=(1 2 3 4 5 6)
echo ${index_array[0]}

```


Index Arrays In Shell Scripts
As you can see, the first element is automatically printed based on index 0.


## 2. Reading Array Elements Sequentially


This is going to be an easy task if you know for loops already. If you don’t we’ll cover them in a future tutorial. We’ll make use of the while or for loops in shell scripts to work through the array elements. Copy the script below and save it as <filename>.sh


```
#!/bin/bash
index_array=(1 2 3 4 5 6 7 8 9 0)

for i in ${index_array[@]}
do
        echo $i
done

```


The above script will output the following:


Looping Over Arrays In Shell Scripts
Now you might have noticed the index_array[@] and if you’re wondering what the @ symbol is for, we’re going to go over the same right now.


# Built-in Operations for Arrays in Shell Scripts


Now that you learned how to access elements individually and using for loops, let’s learn the different operations that are available by default for arrays.


## 1. Access All Elements of an Array


We learned how to access elements by providing the index or the key of the array. But if we want to print all the elements at the same time or work with all the elements, we can use another operator which is the [@] symbol.


As you noticed in the example above, I used this symbol when I wanted to loop through all the array elements using the for loop.


```
echo ${assoc_array[@]}

```


The above will print all the elements that are stored within the assoc array.


## 2. Count the Number of Elements in an Array


Similar to the @ symbol above, we have the # symbol which can be prefixed to an array name to provide us the count of the elements stored in the array. Let’s see how it works.


```
echo ${#index_array[@]}

```


If you want to count the number of characters used for a particular element, we can simply replace the @ symbol with the index.


## 3. Delete Individual Array Elements


We know how to add array elements and print them too. Let’s learn how to delete specific elements. For this purpose, we’ll use the unset keyword.


```
unset index_array[1]

```


Replace the array name and the index ID in the above code example and you’ve removed the array element that you desire. Pretty simple isn’t it?


# Conclusion


Shell scripts are pretty vast and can replace any function that you can perform on the terminal with the right person writing the script. Some additional functionalities of arrays in shell scripts also include being able to work with regex (Regular Expressions). We can use various regular expressions to manipulate array elements within shell scripts.


For now, we hope you have a good understanding of creating and working with arrays and will be able to use arrays in your scripting. Comment below to let us know what you think, and if you have any questions about this topic.


