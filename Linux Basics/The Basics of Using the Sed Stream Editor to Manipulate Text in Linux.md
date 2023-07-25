# The Basics of Using the Sed Stream Editor to Manipulate Text in Linux

```Linux Basics``` ```Linux Commands```

## Introduction


The sed command, short for stream editor, performs editing operations on text coming from standard input or a file. sed edits line-by-line and in a non-interactive way.


This means that you make all of the editing decisions as you are calling the command, and sed executes the directions automatically. This may seem confusing or unintuitive, but it is a very powerful and fast way to transform text, especially as part of a script or automated workflow.


This tutorial will cover some basic operations and introduce you to the syntax required to operate this editor. You will almost certainly never replace your regular text editor with sed, but it will probably become a welcomed addition to your text editing toolbox.



Note: This tutorial uses the GNU version of sed found on Ubuntu and other Linux operating systems. If you’re using macOS, you’ll have the BSD version which has different options and arguments. You can install the GNU version of sed with Homebrew using brew install gnu-sed.

# Basic Usage


sed operates on a stream of text that it reads from either a text file or from standard input (STDIN). This means that you can send the output of another command directly into sed for editing, or you can work on a file that you’ve already created.


You should also be aware that sed outputs everything to standard out (STDOUT) by default. That means that, unless redirected, sed will print its output to the screen instead of saving it in a file.


The basic usage is:


```
sed [options] commands [file-to-edit]


```


In this tutorial, you’ll use a copy of the BSD Software License to experiment with sed. On Ubuntu, execute the following commands to copy the BSD license file to your home directory so you can work with it:


```
cd
cp /usr/share/common-licenses/BSD .


```


If you don’t have a local copy of the BSD license, create one yourself with this command:


```
cat << 'EOF' > BSD
Copyright (c) The Regents of the University of California.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:
1. Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.
3. Neither the name of the University nor the names of its contributors
   may be used to endorse or promote products derived from this software
   without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ``AS IS'' AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED.  IN NO EVENT SHALL THE REGENTS OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
SUCH DAMAGE.
EOF


```


Let’s use sed to view the contents of the BSD license file. sed sends its results to the screen by default, which means you can use it as a file reader by passing it no editing commands. Try executing the following command:


```
sed '' BSD


```


You’ll see the BSD license displayed to the screen:


```
OutputCopyright (c) The Regents of the University of California.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:
1. Redistributions of source code must retain the above copyright
    notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
    notice, this list of conditions and the following disclaimer in the
    documentation and/or other materials provided with the distribution.
...
...

```


The single quotes contain the editing commands you pass to sed. In this case, you passed it nothing, so sed printed each line it received to standard output.


sed can use standard input rather than a file. Pipe the output of the cat command into sed to produce the same result:


```
cat BSD | sed ''


```


You’ll see the output of the file:


```
OutputCopyright (c) The Regents of the University of California.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:
1. Redistributions of source code must retain the above copyright
    notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
    notice, this list of conditions and the following disclaimer in the
    documentation and/or other materials provided with the distribution.
. . .
. . .

```


As you can see, you can operate on files or streams of text, like the ones produced when piping output with the pipe (|) character, just as easily.


# Printing Lines


In the previous example, you saw that input passed into sed without any operations would print the results directly to standard output.


Let’s explore sed’s explicit print command, which you specify by using the p character within single quotes.


Execute the following command:


```
sed 'p' BSD


```


You’ll see each line of the BSD file printed twice:


```
OutputCopyright (c) The Regents of the University of California.
Copyright (c) The Regents of the University of California.
All rights reserved.
All rights reserved.


Redistribution and use in source and binary forms, with or without
Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
modification, are permitted provided that the following conditions
are met:
are met:
. . .
. . .

```


sed automatically prints each line by default, and then you’ve told it to print lines explicitly with the “p” command, so you get each line printed twice.


If you examine the output closely, you’ll see that it has the first line twice, followed by the second line twice, etc, which tells you that sed operates on data line by line. It reads a line, operates on it, and outputs the resulting text before repeating the process on the next line.


You can clean up the results by passing the -n option to sed, which suppresses the automatic printing:


```
sed -n 'p' BSD


```


```
OutputCopyright (c) The Regents of the University of California.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:
1. Redistributions of source code must retain the above copyright
    notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
    notice, this list of conditions and the following disclaimer in the
    documentation and/or other materials provided with the distribution.
. . .
. . .

```


We now are back to printing each line once.


The examples so far can hardly be considered editing (unless you wanted to print each line twice…). Next you’ll explore how sed can modify the output by targeting specific sections of the text data.


# Using Address Ranges


Addresses let you target specific parts of a text stream. You can specify a specific line or even a range of lines.


Let’s have sed print the first line of the file. Execute the following command:


```
sed -n '1p' BSD


```


The first line prints to the screen:


```
OutputCopyright (c) The Regents of the University of California.

```


By placing the number 1 before the print command, you told sed the line number to operate on. You can just as easily print five lines (don’t forget the “-n”):


```
sed -n '1,5p' BSD


```


You’ll see this output:


```
OutputCopyright (c) The Regents of the University of California.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions

```


You’ve just given an address range to sed. If you give sed an address, it will only perform the commands that follow on those lines. In this example, you’ve told sed to print line 1 through line 5. You could have specified this in a different way by giving the first address and then using an offset to tell sed how many additional lines to travel, like this:


```
sed -n '1,+4p' BSD


```


This will result in the same output, because you told sed to start at line 1 and then operate on the next 4 lines as well.


If you want to print every other line, specify the interval after the ~ character. The following command prints every other line in the BSD file, starting with line 1:


```
sed -n '1~2p' BSD


```


Here’s the output you’ll see:


```
OutputCopyright (c) The Regents of the University of California.

modification, are permitted provided that the following conditions
1. Redistributions of source code must retain the above copyright
2. Redistributions in binary form must reproduce the above copyright
    documentation and/or other materials provided with the distribution.
    may be used to endorse or promote products derived from this software
. . .
. . .

```


You can use sed to delete text from the output as well.


# Deleting Text


You can perform text deletion where you previously were specifying text printing by changing the p command to the d command.


In this case, you no longer need the -n command because sed will print everything that is not deleted. This will help you see what’s going on.


Modify the last command from the previous section to make it
delete every other line starting with the first:


```
sed '1~2d' BSD


```


The result is that you see every line you were not given last time:


```
OutputAll rights reserved.
Redistribution and use in source and binary forms, with or without
are met:
    notice, this list of conditions and the following disclaimer.
    notice, this list of conditions and the following disclaimer in the
3. Neither the name of the University nor the names of its contributors
    without specific prior written permission.
. . .
. . .

```


It is important to note here that our source file is not being affected.  It is still intact. The edits are output to our screen.


If we want to save our edits, we can redirect standard output to a file like so:


```
sed '1~2d' BSD > everyother.txt


```


Now open the file with cat:


```
cat everyother.txt


```


You see the same output that you saw onscreen previously:


```
OutputAll rights reserved.
Redistribution and use in source and binary forms, with or without
are met:
    notice, this list of conditions and the following disclaimer.
    notice, this list of conditions and the following disclaimer in the
3. Neither the name of the University nor the names of its contributors
    without specific prior written permission.
. . .
. . .

```


The sed command does not edit the source file by default, but you can change this behavior by passing the -i option, which means “perform edits in-place.” This will alter the source file.



Warning: Using the -i switch will overwrite the original file, so you should use this with care. Perform the operations without the -i switch first and then run the command again with -i once you have what you want, create a backup of the original file, or redirect the output to a file. It’s very easy to accidentally alter the original file with the -i switch.

Let’s try it by editing the everyother.txt file you just created, in-place. Let’s further reduce the file by deleting every other line
again:


```
sed -i '1~2d' everyother.txt


```


If you use cat to display the file with cat everyother.txt, you’ll see that the file has been edited.


The -i option can be dangerous.  Thankfully, sed gives you the ability to create a backup file prior to editing.


To create a backup file prior to editing, add the backup extension directly after the “-i” option:


```
sed -i.bak '1~2d' everyother.txt


```


This creates a backup file with the .bak extension, and then edits the original file in-place.


Next you’ll look at how to use sed to perform search and replace operations.


# Substituting Text


Perhaps the most well-known use for sed is substituting text. sed can search for text patterns using regular expressions, and then replace the found text with something else.


You can learn more about regular expressions by following the Using Grep Regular Expressions to Search for Text Patterns in Linux.


In its most basic form, you can change one word to another word using the following syntax:


```
's/old_word/new_word/'

```


The s is the substitute command. The three slashes (/) are used to separate the different text fields. You can use other characters to delimit the fields if it would be more helpful.


For instance, if you were trying to change a website name, using another delimiter would be helpful since URLs contain slashes.


Execute the following command to print a URL with echo and modify it with sed, using the underscore (_) character as the delimiter:


```
echo "http://www.example.com/index.html" | sed 's_com/index_org/home_'


```


This replaces com/index with org/home. The output shows the modifed URL:


```
Outputhttp://www.example.org/home.html

```


Do not forget the final delimiter, or sed will complain. If you ran this command:


```
echo "http://www.example.com/index.html" | sed 's_com/index_org/home'


```


You’d see this output:


```
Outputsed: -e expression #1, char 20: unterminated `s' command

```


Let’s create a new file to practice some substitutions. Execute the following command to create a new text file called song.txt:


```
echo "this is the song that never ends
yes, it goes on and on, my friend
some people started singing it
not knowing what it was
and they'll continue singing it forever
just because..." > song.txt


```


Now let’s substitute the expression on with forward. Use the following command:


```
sed 's/on/forward/' song.txt


```


The output looks like this:


```
Outputthis is the sforwardg that never ends
yes, it goes forward and on, my friend
some people started singing it
not knowing what it was
and they'll cforwardtinue singing it forever
just because...

```


You can see a few notable things here. First, is that sed replaced patterns, not words. The on within song is changed to forward.


The other thing to notice is that on line 2, the second on was not changed to forward.


This is because by default, the s command operates on the first match in a line and then moves to the next line. To make sed replace every instance of on instead of just the first on each line, you must pass an optional flag to the substitute command.


Provide the g flag to the substitute command by placing it after the substitution set:


```
sed 's/on/forward/g' song.txt


```


You’ll see this output:


```
Outputthis is the sforwardg that never ends
yes, it goes forward and forward, my friend
some people started singing it
not knowing what it was
and they'll cforwardtinue singing it forever
just because...

```


Now the substitute command changes every instance.


If you only wanted to change the second instance of “on” that sed finds on each line, then you would use the number 2 instead of the g:


```
sed 's/on/forward/2' song.txt


```


This time the other lines are unchanged, as they don’t have a second occurrence:


```
Outputthis is the song that never ends
yes, it goes on and forward, my friend
some people started singing it
not knowing what it was
and they'll continue singing it forever
just because...

```


If you only want to see which lines were substituted, use the -n option again to suppress automatic printing.


You can then pass the p option to the substitute command to print lines where substitution took place.


```
sed -n 's/on/forward/2p' song.txt


```


The line that changed prints to the screen:


```
Outputyes, it goes on and forward, my friend

```


As you can see, you can combine the flags at the end of the command.


If you want the search process to ignore case, you can pass it the “i” flag.


```
sed 's/SINGING/saying/i' song.txt


```


Here’s the output you’ll see:


```
Outputthis is the song that never ends
yes, it goes on and on, my friend
some people started saying it
not knowing what it was
and they'll continue saying it forever
just because...

```


## Replacing and Referencing Matched Text


If you want to find more complex patterns with regular expressions, you have a number of different methods of referencing the matched pattern in the replacement text.


For instance, to match from the beginning of the line to at, use the following command:


```
sed 's/^.*at/REPLACED/' song.txt


```


You’ll see this output:


```
OutputREPLACED never ends
yes, it goes on and on, my friend
some people started singing it
REPLACED it was
and they'll continue singing it forever
just because...

```


You can see that the wildcard expression matches from the beginning of the line to the last instance of at.


Since you don’t know the exact phrase that will match in the search string, you can use the & character to represent the matched text in the replacement string.


Let’s put parentheses around the matched text:


```
sed 's/^.*at/(&)/' song.txt


```


You’ll see this output:


```
Output(this is the song that) never ends
yes, it goes on and on, my friend
some people started singing it
(not knowing what) it was
and they'll continue singing it forever
just because...

```


A more flexible way of referencing matched text is to use escaped parentheses to group sections of matched text.


Every group of search text marked with parentheses can be referenced by an escaped reference number. For instance, the first parentheses group can be referenced with \1, the second with \2 and so on.


In this example, we’ll switch the first two words of each line:


```
sed 's/\([a-zA-Z0-9][a-zA-Z0-9]*\) \([a-zA-Z0-9][a-zA-Z0-9]*\)/\2 \1/' song.txt


```


You’ll see this output:


```
Outputis this the song that never ends
yes, goes it on and on, my friend
people some started singing it
knowing not what it was
they and'll continue singing it forever
because just...

```


As you can see, the results are not perfect. For instance, the second line skips the first word because it has a character not listed in our character set. Similarly, it treated they'll as two words in the fifth line.


Let’s improve the regular expression to be more accurate:


```
sed 's/\([^ ][^ ]*\) \([^ ][^ ]*\)/\2 \1/' song.txt


```


You’ll see this output:


```
Outputis this the song that never ends
it yes, goes on and on, my friend
people some started singing it
knowing not what it was
they'll and continue singing it forever
because... just

```


This is much better than last time. This groups punctuation with the associated word.


Notice how we repeat the expression inside the parentheses (once without the * character, and then once with it). This is because the * character matches the character set that comes before it zero or more times. This means that the match with the wildcard would be considered a “match” even if the pattern is not found.


To ensure that sed finds the text at least once, you must match it once without the wildcard before employing the wildcard.


# Conclusion


In this tutorial you explored the sed command. You printed specific lines from the file, searched for text, deleted lines, overwrote the original file, and used regular expressions to replace text.  You should be able to see already how you can quickly transform a text document using properly constructed sed commands.


In the next article in this series, you will explore some more advanced features.


