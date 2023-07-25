# How To Use the AWK language to Manipulate Text in Linux

```Linux Basics``` ```System Tools``` ```Linux Commands```

## Introduction


Linux utilities often follow the Unix philosophy of design.  Tools are encouraged to be small, use plain text files for input and output, and operate in a modular manner.  Because of this legacy, we have great text processing functionality with tools like sed and awk.


awk is both a programming language and text processor that you can use to manipulate text data in very useful ways.  In this guide, you’ll explore how to use the awk command line tool and how to use it to process text.


# Basic Syntax


The awk command is included by default in all modern Linux systems, so you do not need to install it to begin using it.


awk is most useful when handling text files that are formatted in a predictable way.  For instance, it is excellent at parsing and manipulating tabular data.  It operates on a line-by-line basis and iterates through the entire file.


By default, it uses whitespace (spaces, tabs, etc.) to separate fields.  Luckily, many configuration files on your Linux system use this format.


The basic format of an awk command is:


```
awk '/search_pattern/ { action_to_take_on_matches; another_action; }' file_to_parse


```


You can omit either the search portion or the action portion from any awk command.  By default, the action taken if the “action” portion is not given is “print”.  This simply prints all lines that match.


If the search portion is not given, awk performs the action listed on each line.


If both are given, awk uses the search portion to decide if the current line reflects the pattern, and then performs the actions on matches.


In its simplest form, you can use awk like cat to print all lines of a text file out to the screen.


Create a favorite_food.txt file which lists the favorite foods of a group of friends:


```
echo "carrot sandy
wasabi luke
sandwich brian
salad ryan
spaghetti jessica" > favorite_food.txt


```


Now use the awk command to print the file to the screen:


```
awk '{print}' favorite_food.txt


```


You’ll see the file printed to the screen:


```
Outputcarrot sandy
wasabi luke
sandwich brian
salad ryan
spaghetti jessica

```


This isn’t very useful.  Let’s try out awk’s search filtering capabilities by searching through the file for the text “sand”:


```
awk '/sand/' favorite_food.txt


```


```
Outputcarrot sandy
sandwich brian

```


As you can see, awk now only prints the lines that have the characters “sand” in them.


Using regular expressions, you can target specific parts of the text. To display only the line that starts with the letters “sand”, use the regular expression ^sand:


```
awk '/^sand/' favorite_food.txt


```


This time, only one line is displayed:


```
Outputsandwich brian

```


Similarly, you can use the action section to specify which pieces of information you want to print.  For instance, to print only the first column, use the following command:


```
awk '/^sand/ {print $1;}' favorite_food.txt


```


```
Outputsandwich

```


You can reference every column (as delimited by whitespace) by variables associated with their column number.  For example, the The first column is $1, the second is $2, and you can reference the entire line with $0.


# Internal Variables and Expanded Format


The awk command uses some internal variables to assign certain pieces of information as it processes a file.


The internal variables that awk uses are:


- FILENAME: References the current input file.
- FNR: References the number of the current record relative to the current input file.  For instance, if you have two input files, this would tell you the record number of each file instead of as a total.
- FS: The current field separator used to denote each field in a record.  By default, this is set to whitespace.
- NF: The number of fields in the current record.
- NR: The number of the current record.
- OFS: The field separator for the outputted data. By default, this is set to whitespace.
- ORS: The record separator for the outputted data.  By default, this is a newline character.
- RS: The record separator used to distinguish separate records in the input file.  By default, this is a newline character.

You can change the values of these variables at will to match the needs of your files.  Usually you do this during the initialization phase of your processing.


This brings us to another important concept.  The awk syntax is slightly more complex than what you’ve used so far  There are also optional BEGIN and END blocks that can contain commands to execute before and after the file processing, respectively.


This makes our expanded syntax look something like this:


```
awk 'BEGIN { action; }
/search/ { action; }
END { action; }' input_file


```


The BEGIN and END keywords are specific sets of conditions, just like the search parameters. They match before and after the document has been processed.


This means that you can change some of the internal variables in the BEGIN section. For instance, the /etc/passwd file is delimited with colons (:) instead of whitespace.


To print out the first column of this file, execute the following command:


```
awk 'BEGIN { FS=":"; }
{ print $1; }' /etc/passwd


```


```
Outputroot
daemon
bin
sys
sync
games
man
. . .

```


You can use the BEGIN and END blocks to print information about the fields you are printing. Use the following command to transform the data from the file into a table, nicely spaced with tabs using \t:


```
awk 'BEGIN { FS=":"; print "User\t\tUID\t\tGID\t\tHome\t\tShell\n--------------"; }
{print $1,"\t\t",$3,"\t\t",$4,"\t\t",$6,"\t\t",$7;}
END { print "---------\nFile Complete" }' /etc/passwd


```


You’ll see this output:


```
OutputUser		UID		GID		Home		Shell
--------------
root 		 0 		 0 		 /root 		 /bin/bash
daemon 		 1 		 1 		 /usr/sbin 		 /bin/sh
bin 		 2 		 2 		 /bin 		 /bin/sh
sys 		 3 		 3 		 /dev 		 /bin/sh
sync 		 4 		 65534 		 /bin 		 /bin/sync
. . .
---------
File Complete

```


As you can see, you can format things quite nicely by taking advantage of some of awk’s features.


Each of the expanded sections are optional.  In fact, the main action section itself is optional if another section is defined.  For example, you  can do things like this:


```
awk 'BEGIN { print "We can use awk like the echo command"; }'


```


And you’ll see this output:


```
OutputWe can use awk like the echo command

```


Now let’s look at how to look for text within fields of the output.


# Field Searching and Compound Expressions


In one of the previous examples, you printed the line in the favorite_food.txt file that began with “sand”.  This was easy because you were looking for the beginning of the entire line.


What if you wanted to find out if a search pattern matched at the beginning of a field instead?


Create a new version of the favorite_food.txt file which adds an item number in front of each person’s food:


```
echo "1 carrot sandy
2 wasabi luke
3 sandwich brian
4 salad ryan
5 spaghetti jessica" > favorite_food.txt


```


If you want to find all foods from this file that begin with “sa”, you might begin by trying something like this:


```
awk '/sa/' favorite_food.txt


```


This shows all lines that contain “sa”:


```
Output1 carrot sandy
2 wasabi luke
3 sandwich brian
4 salad ryan

```


Here, you are matching any instance of “sa” in the word.  This ends up including things like “wasabi” which has the pattern in the middle, or “sandy” which is not in the column you want.  In this case you’re only interested in words beginning with “sa” in the second column.


You can tell awk to only match at the beginning of the second column by using this command:


```
awk '$2 ~ /^sa/' favorite_food.txt


```


As you can see, this allows us to only search at the beginning of the second column for a match.


The field_num ~ part specifies that awk should only pay attention to the second column.


```
Output3 sandwich brian
4 salad ryan

```


You can just as easily search for things that do not match by including the “!” character before the tilde (~).  This command will return all lines that do not have a food that starts with “sa”:


```
awk '$2 !~ /^sa/' favorite_food.txt


```


```
Output1 carrot sandy
2 wasabi luke
5 spaghetti jessica

```


If you decide later on that you are only interested in lines that don’t start with “sa” and the item number is less than 5, you could use a compound expression like this:


```
awk '$2 !~ /^sa/ && $1 < 5' favorite_food.txt


```


This introduces a few new concepts.  The first is the ability to add additional requirements for the line to match by using the && operator.  Using this, you can combine an arbitrary number of conditions for the line to match. In this case, you’re using this operator to add a check that the value of the first column is less than 5.


You’ll see this output:


```
Output1 carrot sandy
2 wasabi luke

```


You can use awk to process files, but you can also work with the output of other programs.


# Processing Output from Other Programs


You can use the awk command to parse the output of other programs rather than specifying a filename. For example, you can use awk to parse out the IPv4 address from the ip command.


The ip a command displays the IP address, broadcast address, and other information about all the network interfaces on your machine. To display the information for the interface called eth0, use this command:


```
ip a s eth0 


```


You’ll see the following results:


```
Output2571: eth0@if2572: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:0b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.11/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

```


You can use awk to target the inet line and then print out just the IP address:


```
ip a s eth0 | awk -F '[\/ ]+' '/inet / {print $3}'


```


The -F flag tells awk to delimit by forward slashes or spaces using the regular expression [\/ ]+. This splits the line     inet 172.17.0.11/16 into separate fields. The IP address is in the third field because the spaces at the start of the line also count as a field, since you delimited by spaces as well as slashes. Note that awk treated consecutive spaces as a single space in this case.


The output shows the IP address:


```
Output172.17.0.11

```


You’ll find many places where you can use awk to search or parse the output of other commands.


# Conclusion


By now, you should have a basic understanding of how you can use the awk command to manipulate, format, and selectively print text files and streams of text.  Awk is a much larger topic though, and is actually an entire programming language complete with variable assignment, control structures, built-in functions, and more. You can use it within your own scripts to format text in a reliable way.


To learn more about awk, you can read the free public-domain book by its creators which goes into much more detail.


