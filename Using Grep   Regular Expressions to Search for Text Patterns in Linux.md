# Using Grep   Regular Expressions to Search for Text Patterns in Linux

```Linux Basics``` ```Linux Commands```

## Introduction


The grep command is one of the most useful commands in a Linux terminal environment. The name grep stands for “global regular expression print”. This means that you can use grep to check whether the input it receives matches a specified pattern.  This seemingly trivial program is extremely powerful; its ability to sort input based on complex rules makes it a popular link in many command chains.


In this tutorial, you will explore the grep command’s options, and then you’ll dive into using regular expressions to do more advanced searching.


# Prerequisites


To follow along with this guide, you will need access to a computer running a Linux-based operating system. This can either be a virtual private server which you’ve connected to with SSH or your local machine. Note that this tutorial was validated using a Linux server running Ubuntu 20.04, but the examples given should work on a computer running any version of any Linux distribution.


If you plan to use a remote server to follow this guide, we encourage you to first complete our Initial Server Setup guide. Doing so will set you up with a secure server environment — including a non-root user with sudo privileges and a firewall configured with UFW — which you can use to build your Linux skills.


# Basic Usage


In this tutorial, you’ll use grep to search the GNU General Public License version 3 for various words and phrases.


If you’re on an Ubuntu system, you can find the file in the /usr/share/common-licenses folder. Copy it to your home directory:


```
cp /usr/share/common-licenses/GPL-3 .


```


If you’re on another system, use the curl command to download a copy:


```
curl -o GPL-3 https://www.gnu.org/licenses/gpl-3.0.txt


```


You’ll also use the BSD license file in this tutorial. On Linux, you can copy that to your home directory with the following command:


```
cp /usr/share/common-licenses/BSD .


```


If you’re on another system, create the file with the following command:


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


Now that you have the files, you can start working with grep.


In the most basic form, you use grep to match literal patterns within a text file. This means that if you pass grep a word to search for, it will print out every line in the file containing that word.


Execute the following command to use grep to search for every line that contains the word GNU:


```
grep "GNU" GPL-3


```


The first argument, GNU, is the pattern you’re searching for, while the second argument, GPL-3, is the input file you wish to search.


The resulting output will be every line containing the pattern text:


```
Output                    GNU GENERAL PUBLIC LICENSE
  The GNU General Public License is a free, copyleft license for
the GNU General Public License is intended to guarantee your freedom to
GNU General Public License for most of our software; it applies also to
  Developers that use the GNU GPL protect your rights with two steps:
  "This License" refers to version 3 of the GNU General Public License.
  13. Use with the GNU Affero General Public License.
under version 3 of the GNU Affero General Public License into a single
...
...

```


On some systems, the pattern you searched for will be highlighted in the output.


## Common Options


By default, grep will search for the exact specified pattern within the input file and return the lines it finds. You can make this behavior more useful though by adding some optional flags to grep.


If you want grep to ignore the “case” of your search parameter and search for both upper- and lower-case variations, you can specify the -i or --ignore-case option.


Search for each instance of the word license (with upper, lower, or mixed cases) in the same file as before with the following command:


```
grep -i "license" GPL-3


```


The results contain: LICENSE, license, and License:


```
Output                    GNU GENERAL PUBLIC LICENSE
 of this license document, but changing it is not allowed.
  The GNU General Public License is a free, copyleft license for
  The licenses for most software and other practical works are designed
the GNU General Public License is intended to guarantee your freedom to
GNU General Public License for most of our software; it applies also to
price.  Our General Public Licenses are designed to make sure that you
(1) assert copyright on the software, and (2) offer you this License
  "This License" refers to version 3 of the GNU General Public License.
  "The Program" refers to any copyrightable work licensed under this
...
...

```


If there was an instance with LiCeNsE, that would have been returned as well.


If you want to find all lines that do not contain a specified pattern, you can use the -v or --invert-match option.


Search for every line that does not contain the word the in the BSD license with the following command:


```
grep -v "the" BSD


```


You’ll receive this output:


```
OutputAll rights reserved.

Redistribution and use in source and binary forms, with or without
are met:
    may be used to endorse or promote products derived from this software
    without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ``AS IS'' AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
...
...

```


Since you did not specify the “ignore case” option, the last two items were returned as not having the word the.


It is often useful to know the line number that the matches occur on.  You can do this by using the -n or --line-number option. Re-run the previous example with this flag added:


```
grep -vn "the" BSD


```


This will return the following text:


```
Output2:All rights reserved.
3:
4:Redistribution and use in source and binary forms, with or without
6:are met:
13:   may be used to endorse or promote products derived from this software
14:   without specific prior written permission.
15:
16:THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ``AS IS'' AND
17:ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
...
...

```


Now you can reference the line number if you want to make changes to every line that does not contain the. This is especially handy when working with source code.


# Regular Expressions


In the introduction, you learned that grep stands for “global regular expression print”. A “regular expression” is a text string that describes a particular search pattern.


Different applications and programming languages implement regular expressions slightly differently. In this tutorial you will only be exploring a small subset of the way that grep describes its patterns.


## Literal Matches


In the previous examples in this tutorial, when you searched for the words GNU and the, you were actually searching for basic regular expressions which matched the exact string of characters GNU and the.  Patterns that exactly specify the characters to be matched are called “literals” because they match the pattern literally, character-for-character.


It is helpful to think of these as matching a string of characters rather than matching a word. This will become a more important distinction as you learn more complex patterns.


All alphabetical and numerical characters (as well as certain other characters) are matched literally unless modified by other expression mechanisms.


## Anchor Matches


Anchors are special characters that specify where in the line a match must occur to be valid.


For instance, using anchors, you can specify that you only want to know about the lines that match GNU at the very beginning of the line. To do this, you could use the ^ anchor before the literal string.


Run the following command to search the GPL-3 file and find lines where GNU occurs at the very beginning of a line:


```
grep "^GNU" GPL-3


```


This command will return the following two lines:


```
OutputGNU General Public License for most of our software; it applies also to
GNU General Public License, you may choose any version ever published

```


Similarly, you use the $ anchor at the end of a pattern to indicate that the match will only be valid if it occurs at the very end of a line.


This command will match every line ending with the word and in the GPL-3 file:


```
grep "and$" GPL-3


```


You’ll receive this output:


```
Outputthat there is no warranty for this free software.  For both users' and
  The precise terms and conditions for copying, distribution and
  License.  Each licensee is addressed as "you".  "Licensees" and
receive it, in any medium, provided that you conspicuously and
    alternative is allowed only occasionally and noncommercially, and
network may be denied when the modification itself materially and
adversely affects the operation of the network or violates the rules and
provisionally, unless and until the copyright holder explicitly and
receives a license from the original licensors, to run, modify and
make, use, sell, offer for sale, import and otherwise run, modify and

```


## Matching Any Character


The period character (.) is used in regular expressions to mean that any single character can exist at the specified location.


For example, to match anything in the GPL-3 file that has two characters and then the string cept, you would use the following pattern:


```
grep "..cept" GPL-3


```


This command returns the following output:


```
Outputuse, which is precisely where it is most unacceptable.  Therefore, we
infringement under applicable copyright law, except executing it on a
tells the user that there is no warranty for the work (except to the
License by making exceptions from one or more of its conditions.
form of a separately written license, or stated as exceptions;
  You may not propagate or modify a covered work except as expressly
  9. Acceptance Not Required for Having Copies.
...
...

```


This output has instances of both accept and except and variations of the two words. The pattern would also have matched z2cept if that was found as well.


## Bracket Expressions


By placing a group of characters within brackets (\[ and \]), you can specify that the character at that position can be any one character found within the bracket group.


For example, to find the lines that contain too or two, you would specify those variations succinctly by using the following pattern:


```
grep "t[wo]o" GPL-3


```


The output shows that both variations exist in the file:


```
Outputyour programs, too.
freedoms that you received.  You must make sure that they, too, receive
  Developers that use the GNU GPL protect your rights with two steps:
a computer network, with no transfer of a copy, is not conveying.
System Libraries, or general-purpose tools or generally available free
    Corresponding Source from a network server at no charge.
...
...

```


Bracket notation gives you some interesting options. You can have the pattern match anything except the characters within a bracket by beginning the list of characters within the brackets with a ^ character.


This example is like the pattern .ode, but will not match the pattern code:


```
grep "[^c]ode" GPL-3


```


Here’s the output you’ll receive:


```
Output  1. Source Code.
    model, to give anyone who possesses the object code either (1) a
the only significant mode of use of the product.
notice like this when it starts in an interactive mode:

```


Notice that in the second line returned, there is, in fact, the word code. This is not a failure of the regular expression or grep.  Rather, this line was returned because earlier in the line, the pattern mode, found within the word model, was found. The line was returned because there was an instance that matched the pattern.


Another helpful feature of brackets is that you can specify a range of characters instead of individually typing every available character.


This means that if you want to find every line that begins with a capital letter, you can use the following pattern:


```
grep "^[A-Z]" GPL-3


```


Here’s the output this expression returns:


```
OutputGNU General Public License for most of our software; it applies also to
States should not allow patents to restrict development and use of
License.  Each licensee is addressed as "you".  "Licensees" and
Component, and (b) serves only to enable use of the work with that
Major Component, or to implement a Standard Interface for which an
System Libraries, or general-purpose tools or generally available free
Source.
User Product is transferred to the recipient in perpetuity or for a
...
...

```


Due to some legacy sorting issues, it is often more accurate to use POSIX character classes instead of character ranges like you just used.


To discuss every POSIX character class would be beyond the scope of this guide, but an example that would accomplish the same procedure as the previous example uses the \[:upper:\] character class within a bracket selector:


```
grep "^[[:upper:]]" GPL-3


```


The output will be the same as before.


## Repeat Pattern Zero or More Times


Finally, one of the most commonly used meta-characters is the asterisk, or *, which means “repeat the previous character or expression zero or more times”.


To find each line in the GPL-3 file that contains an opening and closing parenthesis, with only letters and single spaces in between, use the following expression:


```
grep "([A-Za-z ]*)" GPL-3


```


You’ll get the following output:


```
Output Copyright (C) 2007 Free Software Foundation, Inc.
distribution (with or without modification), making available to the
than the work as a whole, that (a) is included in the normal form of
Component, and (b) serves only to enable use of the work with that
(if any) on which the executable work runs, or a compiler used to
    (including a physical distribution medium), accompanied by the
    (including a physical distribution medium), accompanied by a
    place (gratis or for a charge), and offer equivalent access to the
...
...

```


So far you’ve used periods, asterisks, and other characters in your expressions, but sometimes you need to search for those characters specifically.


## Escaping Meta-Characters


There are times where you’ll need to search for a literal period or a literal opening bracket, especially when working with source code or configuration files. Because these characters have special meaning in regular expressions, you need to “escape” these characters to tell grep that you do not wish to use their special meaning in this case.


You escape characters by using the backslash character (\) in front of the character that would normally have a special meaning.


For instance, to find any line that begins with a capital letter and ends with a period, use the following expression which escapes the ending period so that it represents a literal period instead of the usual “any character” meaning:


```
grep "^[A-Z].*\.$" GPL-3


```


This is the output you’ll see:


```
OutputSource.
License by making exceptions from one or more of its conditions.
License would be to refrain entirely from conveying the Program.
ALL NECESSARY SERVICING, REPAIR OR CORRECTION.
SUCH DAMAGES.
Also add information on how to contact you by electronic and paper mail.

```


Now let’s look at other regular expression options.


# Extended Regular Expressions


The grep command supports a more extensive regular expression language by using the -E flag or by calling the egrep command instead of grep.


These options open up the capabilities of “extended regular expressions”. Extended regular expressions include all of the basic meta-characters, along with additional meta-characters to express more complex matches.


## Grouping


One of the most useful abilities that extended regular expressions open up is the ability to group expressions together to manipulate or reference as one unit.


To group expressions together, wrap them in parentheses. If you would like to use parentheses without using extended regular expressions, you can escape them with the backslash to enable this functionality. This means that the following three expressions are functionally equivalent:


```
grep "\(grouping\)" file.txt
grep -E "(grouping)" file.txt
egrep "(grouping)" file.txt


```


## Alternation


Similar to how bracket expressions can specify different possible choices for single character matches, alternation allows you to specify alternative matches for strings or expression sets.


To indicate alternation, use the pipe character |. These are often used within parenthetical grouping to specify that one of two or more possibilities should be considered a match.


The following will find either GPL or General Public License in the text:


```
grep -E "(GPL|General Public License)" GPL-3


```


The output looks like this:


```
Output  The GNU General Public License is a free, copyleft license for
the GNU General Public License is intended to guarantee your freedom to
GNU General Public License for most of our software; it applies also to
price.  Our General Public Licenses are designed to make sure that you
  Developers that use the GNU GPL protect your rights with two steps:
  For the developers' and authors' protection, the GPL clearly explains
authors' sake, the GPL requires that modified versions be marked as
have designed this version of the GPL to prohibit the practice for those
...
...

```


Alternation can select between more than two choices by adding additional choices within the selection group separated by additional pipe (|) characters.


## Quantifiers


Like the * meta-character that matched the previous character or character set zero or more times, there are other meta-characters available in extended regular expressions that specify the number of occurrences.


To match a character zero or one times, you can use the ? character.  This makes character or character sets that came before optional, in essence.


The following matches copyright and right by putting copy in an optional group:


```
grep -E "(copy)?right" GPL-3


```


You’ll receive this output:


```
Output Copyright (C) 2007 Free Software Foundation, Inc.
  To protect your rights, we need to prevent others from denying you
these rights or asking you to surrender the rights.  Therefore, you have
know their rights.
  Developers that use the GNU GPL protect your rights with two steps:
(1) assert copyright on the software, and (2) offer you this License
  "Copyright" also means copyright-like laws that apply to other kinds of
...

```


The + character matches an expression one or more times. This is almost like the * meta-character, but with the + character, the expression must match at least once.


The following expression matches the string free plus one or more characters that are not white space characters:


```
grep -E "free[^[:space:]]+" GPL-3


```


You’ll see this output:


```
Output  The GNU General Public License is a free, copyleft license for
to take away your freedom to share and change the works.  By contrast,
the GNU General Public License is intended to guarantee your freedom to
  When we speak of free software, we are referring to freedom, not
have the freedom to distribute copies of free software (and charge for
you modify it: responsibilities to respect the freedom of others.
freedomss that you received.  You must make sure that they, too, receive
protecting users' freedom to change the software.  The systematic
of the GPL, as needed to protect the freedom of users.
patents cannot be used to render the program non-free.

```


## Specifying Match Repetition


To specify the number of times that a match is repeated, use the brace characters ({ and }). These characters let you specify an exact number, a range, or an upper or lower bounds to the amount of times an expression can match.


Use the following expression to find all of the lines in the GPL-3 file that contain triple-vowels:


```
grep -E "[AEIOUaeiou]{3}" GPL-3


```


Each line returned has a word with three vowels:


```
Outputchanged, so that their problems will not be attributed erroneously to
authors of previous versions.
receive it, in any medium, provided that you conspicuously and
give under the previous paragraph, plus a right to possession of the
covered work so as to satisfy simultaneously your obligations under this

```


To match any words that have between 16 and 20 characters, use the following expression:


```
grep -E "[[:alpha:]]{16,20}" GPL-3


```


Here’s this command’s output:


```
Output    certain responsibilities if you distribute copies of the software, or if
    you modify it: responsibilities to respect the freedom of others.
        c) Prohibiting misrepresentation of the origin of that material, or

```


Only lines containing words within that length are displayed.


# Conclusion


grep is useful in finding patterns within files or within the file system hierarchy, so it’s worth spending time getting comfortable with its options and syntax.


Regular expressions are even more versatile, and can be used with many popular programs. For instance, many text editors implement regular expressions for searching and replacing text.


Furthermore, most modern programming languages use regular expressions to perform procedures on specific pieces of data. Once you understand regular expressions, you’ll be able to transfer that knowledge to many common computer-related tasks, from performing advanced searches in your text editor to validating user input.


