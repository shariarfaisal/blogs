# AWK command in Linux Unix

```UNIX/Linux```

AWK is suitable for pattern search and processing. The script runs to search one or more files to identify matching patterns and if the said patterns perform specific tasks. In this guide, we take a look into AWK Linux command and see what it can do.


# What Operations can AWK do?


- Scanning files line by line
- Splitting each input line into fields
- Comparing input lines and fields to patterns
- Performing specified actions on matching lines

# AWK Command Usefulness


- Changing data files
- Producing formatted reports

# Programming Concepts for awk command


- Format output lines
- Conditional and loops
- Arithmetic and string operations

# AWK Syntax


```
$ awk options 'selection _criteria {action }' input-file > output-file

```


To demonstrate more about AWK usage, we are going to use the text file called file.txt  1st column => Item, 2nd column => Model 3rd column => Country 4th column => Cost


# Awk Command Examples


## Printing specific columns


To print the 2nd and 3rd columns, execute the command below.


```
$ awk '{print $2 "\t" $3}' file.txt

```


Output 


## Printing all lines in a file


If you wish to list all the lines and columns in a file, execute


```
$ awk ' {print $0}' file.txt

```


Output 


## Printing all lines that match a specific pattern


if you want to print lines that match a certain pattern, the syntax is as shown


```
$ awk '/variable_to_be_matched/ {print $0}' file.txt

```


For instance, to match all entries with the letter ‘o’, the syntax will be


```
$ awk '/o/ {print $0}' file.txt

```


Output  To match all entries with the letter ‘e’


```
$ awk '/e/ {print $0}' file.txt

```


Output 


## Printing columns that match a specific pattern


When AWK locates a pattern match, the command will execute the whole record. You can change the default by issuing an instruction to display only certain fields. For example:


```
$ awk '/a/ {print $3 "\t" $4}' file.txt

```


The above command prints the 3rd and 4th columns where the letter ‘a’ appears in either of the columns Output 


## Counting and Printing Matched Pattern


You can use AWK to count and print the number of lines for every pattern match. For example, the command below counts the number of instances a matching pattern appears


```
$ awk '/a/{++cnt} END {print "Count = ", cnt}' file.txt

```


Output 


## Print Lines with More or less than a No. of Characters


AWK has a built-in length function that returns the length of the string. From the command $0 variable stores the entire line and in the absence of a body block, the default action is taken, i.e., the print action. Therefore, in our text file, if a line has more than 18 characters, then the comparison results true, and the line is printed as shown below.


```
$ awk 'length($0) > 20' file.txt

```


Output 


## Saving output of AWK to a different file


If you wish to save the output of your results, use the > redirection operator. For example


```
$ awk '/a/ {print $3 "\t" $4}' file.txt > Output.txt

```


You can verify the results using the cat command as shown below


```
$ cat output.txt

```


Output 


# Conclusion


AWK is another simple programming script that you can use to manipulate text in documents or perform specific functions. The shared commands are a few or the many you are yet to know or come across.


