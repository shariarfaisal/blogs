# How To Use Vim for Advanced Editing of Plain Text or Code on a VPS

```Linux Basics``` ```System Tools```

## Introduction



Vim, an improvement on the classic vi text editor, is extremely powerful at editing code and plain text.  While it may seem obtuse and difficult at first, it is one of the most efficient ways of editing text due to its language like command syntax.


In a previous article, we discussed how to install vim and perform basic editing.  In this document, we will continue to more advanced topics that may help demonstrate the versatility available when editing.


We will assume that you have installed vim and are familiar with the basic movement and editing commands discussed in the article linked above.


# Advanced Navigation



Before beginning new material, let’s review just a little navigation that we learned in the previous article:


- 
Basic Movements

h: left
l: right
j: down
k: up


- h: left
- l: right
- j: down
- k: up
- 
Other Movements

gg: top of the document
G: bottom of document or to line number if a number is placed in front of G
w: next word
e: end of word
0: beginning of line
$: end of line


- gg: top of the document
- G: bottom of document or to line number if a number is placed in front of G
- w: next word
- e: end of word
- 0: beginning of line
- $: end of line

As you can see, we already have quite a few movement commands at our disposal.  However, we can direct movement in other ways as well.


We can move the cursor to different areas in the currently visible portion of the page by using these commands:


- 
H: Move cursor to the top of the currently visible page (think “high”)

- 
M: Move cursor to the middle of the currently visible page

- 
L: Move cursor to the bottom of the currently visible page (think “low”)


If we want to move the page instead of the cursor (as in scrolling), we can use these commands:


- 
CTRL-D: Page down

- 
CTRL-U: Page up

- 
CTRL-E: Scroll down one line

- 
CTRL-Y: Scroll up one line


We can also navigate by logical blocks of information.  This can be useful if you are typing regular text instead of code.  These are similar to the word and line navigation commands.


- 
): Move to start of next sentence

- 
(: Move to start of last sentence

- 
}: Move to start of next paragraph (as delimited by a blank line)

- 
{: Move to start of last paragraph (as delimited by a blank line)


You can also define your own points in the file to jump to.  You can set marks at any point in a file.  You can then reference those marks to either jump to that point or pass that point to a command that accepts movements:


- 
m: Typing “m” followed by a letter creates a mark reference by that letter.

Lower-case letters are specific to the current document, while upper-case letters can only be used once (they can be used to jump to sections in different documents.


- Lower-case letters are specific to the current document, while upper-case letters can only be used once (they can be used to jump to sections in different documents.
- 
': The single quote followed by a mark letter (previously defined with the “m” as above), will move the cursor to the beginning of the line containing that mark.

- 
`: The back-tick followed by a mark letter will move the cursor to the exact position of the mark.


These commands allow you to place a mark and then yank, delete, or format on the area defined between the current position and the mark.  This allows for very fine-grained control over editing options.


# How To Manage Documents



Often when you’re working, whether on a software project or a term paper, you want to be able to reference multiple documents at once.  Vim has a few different ways of doing this, depending on how you wish to work.


## How To Manage Buffers



One way to manage multiple files is through buffers.  Buffers usually represent a file open for editing.  They are basically everything that vim has open currently and can get access to easily.


We open multiple files with vim like this:


```
vim file1 file2 file3

```


Each of these files is opened in its own buffer.Currently, we can only see the first file.


We can see which buffers we have available by typing :buffers.


```
:buffers

```



```
:buffers
  1 %a  "file1"             line 1
  2     "file2"             line 0
  3     "file3"             line 0
Press ENTER or type command to continue

```


If we wish to check out the next buffer, we can type :bn.  This will change to the next buffer.  Similarly, we can switch to a buffer by number (in the first column above), or name, by typing b followed by the number or name.  This works even if the file name is not complete.


Here are some of the commands to manage buffers:


- 
:buffers: List available buffers

- 
:ls: Same as above

- 
:bn: Switch to next buffer

- 
:bp: Switch to previous buffer

- 
:bfirst: Switch to first buffer

- 
:blast: Switch to last buffer

- 
:bdelete: Delete the current buffer

- 
:badd: Open a new buffer with the filename that follows

- 
:e: Edit another file in a new buffer and switch to it.


## How To Manage Windows



A separate control mechanism that vim has for managing multiple files is the concept of windows or views.  This allows you to split the current editing area into different windows so that you can view multiple buffers at the same time.


To split the current workspace into separate windows, you can type :split or :sp.  This opens a new window above the current one and changes focus to that window.  You can change the buffer shown in the new window by using the buffer commands shown above.


Here are some commands that we can use to create and manage windows:


- 
:sp: Split the current window in two.  The same buffer will be shown in each window initially.

Precede the “sp” with a number to set the new window height.


- Precede the “sp” with a number to set the new window height.
- 
:vs: Split the current window vertically.  The same buffer will be shown in each window initially.

Precede the “vs” with a number to set the new window width.


- Precede the “vs” with a number to set the new window width.
- 
CTRL-ww: Change focus to the next window

- 
CTRL-w(movement): Change focus to the window in the direction (h,j,k,l) indicated

- 
CTRL-wc: Close the current window

- 
CTRL-w+: Increase size of current window

- 
CTRL-w-: Decrease size of current window

- 
CTRL-w=: Set all windows to equal size

- 
#CTRL-w_: Sets the height to the size indicated by the preceding “#”

- 
:only: Close all windows but the current one

- 
CTRL-wn: Opens a new window with a new buffer


## How To Manage Tabs



A third concept for managing multiple documents within vim is that of tabs.  Unlike many programs, in vim, tabs can contain windows, not the other way around.  Tabs can contain windows, which act as viewports into buffers.


We can manage the window layout of each tab separately.  To create tabs we can use the :tabnew command to open a new tab.


Some easy ways to manage tabs are:


- 
:tabnew: Open new tab

- 
:tabclose: Close current tab

- 
:tabn: Switch to next tab

- 
gt: Switch to next tab

- 
:tabp: Switch to previous tab

- 
gT: Switch to previous tab

- 
:tab ball: Open all buffers in individual tabs

- 
:tabs: List all available tabs


With shuffling around with buffers, windows, and tabs, it sometimes becomes confusing which file you are currently viewing.  A quick way to find out the filename you’re currently viewing is to type:


- CTRL-g: Displays current file name

# Document Specific Commands



Depending on what kind of documents you are dealing with, vim has certain functionality that may help you.


## Plain Text



If you are editing plain text documents, vim can assist you in a variety of ways.  One of the features that is essential for this function is spell check.


To turn on spell checking within vim, you can type:


```
:set spell

```


To set the language that is being used, you can type:


```
:set spelllang=[language abbreviation]

```


Now, your document will be checked for spelling.  The normal squiggly line will appear under misspelled words.  This is how you use it.


To jump back and forth between misspelled words, type:


```
]s    # Jump to next mistake
[s    # Jump to previous mistake

```


Once your cursor is over a misspelled word, you can see spelling suggestions by typing:


```
z=

```


This will give you a list of possible matches.  You can select the option you want by choosing the associated number, or you can press ENTER to keep the word as it is.


If you want to mark a word as not being misspelled, you can add it to one of the spelling list.  Vim maintains two spelling lists, a regular list and a temporary list that will be used for the current session.


To add the word to the “good” words lists, use one of these commands:


```
zg    # Adds word to regular dictionary
zG    # Adds word to the current session dictionary

```


If you accidentally add a word, you can remove it by going to the word and typing:


```
zug   # Remove word from regular dictionary
zuG   # Remove word from the current session dictionary

```


If you find yourself having to type out long words of phrases often, you can add an abbreviation.


If we type :ab followed by an abbreviation and an expansion, vim will input the expansion whenever we type the abbreviation followed by a space.


For instance, if we are sticklers who follow Richard Stallman’s example of correcting any usage of “Linux” with “GNU/Linux”, we can create an abbreviation that automatically does that:


```
:ab Linux GNU/Linux

```


Now, when we type “Linux”, vim will automatically substitute “GNU/Linux”.


```
Linux is an operating system.

```


Changes to:


```
GNU/Linux is an operating system.

```


If we find ourselves talking specifically about the kernel though, where only the word Linux would be appropriate, we can cancel the expansion by typing CTRL-V before typing the space.


```
GNU/Linux is an operating system with Linux(CTRL-V) as a kernel.

```


If we no longer wish to use this abbreviation, we can remove it with this command:


```
:una Linux 

```


Now our “Linux” will remain “Linux”.


Another thing you might have to do from time to time is insert characters that are not on a traditional qwerty keyboard.  We call these “digraphs”.  You can see a list of vim’s digraph by typing:


```
:digraphs

```



```
NU ^@  10    SH ^A   1    SX ^B   2    EX ^C   3    ET ^D   4    EQ ^E   5
AK ^F   6    BL ^G   7    BS ^H   8    HT ^I   9    LF ^@  10    VT ^K  11
FF ^L  12    CR ^M  13    SO ^N  14    SI ^O  15    DL ^P  16    D1 ^Q  17
D2 ^R  18    D3 ^S  19    D4 ^T  20    NK ^U  21    SY ^V  22    EB ^W  23
CN ^X  24    EM ^Y  25    SB ^Z  26    EC ^[  27    FS ^\  28    GS ^]  29
RS ^^  30    US ^_  31    SP     32    Nb #   35    DO $   36    At @   64

```


Now, you can insert any of the characters on the right column by typing CTRL-k followed by the two letters on the left column.


For instance, on my computer, to enter a British pound sign, I can type this when in insert mode:


```
CTRL-k Pd

```



```
£

```


## Source Code



If you are coding, there are a number of different things that will help you interact with your code.


One of the most basic is syntax highlighting.  You can enable syntax highlighting by typing:


```
:syntax on

```


This should set up syntax highlighting for your file based on the file extension that is detected.  If you would like to change the language that is being highlighted, you can do so by setting the language with:


```
:set filetype=[language]

```


If you would like to use a system utility to modify some lines in your file, you can call it by using the ! command in normal mode.


This command accepts a motion and then sends it to the command that follows.


```
![motion] filter

```


For example, to sort the lines from the current position to the end of the file, you can type:


```
!G sort

```


Sort is a Linux command that sorts the input, alphabetically by default.


If we want to insert the output of a command into the file, navigate to a blank line where you want the output.  Type:


```
!!command

```


This will place the output of the specified command into the document.


If we want to see the results of a command, but do not wish to insert it into the document, we can also use the command mode (:) version, which would be:


```
:!command

```


This will show you the results of the command, but will return you to your unaltered document when finished.


# Reducing Repetition



Often in editing or creating any kind of file, you will find yourself repeating many of the same or similar operations.  Luckily, vim provides some ways to save groups of commands into macros.


To begin recording a macro, you can type q followed by a letter to reference the macro.


```
qa    # will save macro "a"

```


Any commands you type will now be recorded as part of the macro.  To end the macro, you can type q again.


So if we type:


```
qa0c3wDELETED<esc>q

```


This would start a macro (saved as “a”), go to the start of the line, and replace the next three words with the word “DELETED”.  It then exits insert mode and ends the macro.


To play this macro back, starting at the current cursor position, use the @ character followed by the macro reference:


```
@a

```


This will replay the macro commands starting at the current position.


If we wish to create a macro that ends in insert mode, we have to end the macro in a different way (typing “q” will just insert a q).  We can execute a normal mode command in insert mode by preceding it with CTRL-O.


Thus, if we want to change the contents of the first parentheses on this line, you could have a macro that says:


```
qi0f(lct)<CTRL-O>q

```


This creates a macro “i”.  The macro moves to the beginning of the current line.  It then finds the opening parenthesis and moves to the right one character (to move inside of the parentheses).  It then changes everything until the closing parenthesis.  With vim in insert mode awaiting replacement text, we press CTRL-O followed by q to end the macro, leaving us in insert mode ready to replace the text.


# Conclusion



You should now have an idea of some more complicated ways that vim can help you.  While this might seem like a lot, it is only scratching the surface.


There are plenty of functions that we have not touched, and you don’t need to know everything.  You will learn what is important based on how you choose to use vim.  The more you practice and use it every day, the more natural it will feel, and the more powerful it will become.


<div class=“author”>By Justin Ellingwood</div>


