# Top 10 Linux Easter Eggs

```Ubuntu``` ```Miscellaneous``` ```CentOS``` ```Fedora``` ```Debian```

# Not a Definitive List…



Often, when you log into your Linux VPS, you are looking to get some work done.  However, no one can claim that the thousands of developers who create the software available on a typical Linux machine are always completely serious.


Linux has a history of including some fun “easter eggs” in its software.  In this article, we’ll tell you about some fun commands and options to lighten up your day.  Not all of them are “easter eggs”, but we think you’ll enjoy them none-the-less.


# Text Editors



## Vim and Douglas Adams



Those of you familiar with Douglas Adams, writer of The Hitchhiker’s Guide to the Galaxy, will appreciate a relevant help option included in the vim text editor.


If you haven’t already, install vim.  In Ubuntu/Debian, you can type:


```
sudo apt-get install vim

```


In CentOS/Fedora, you can type:


```
sudo yum install vim

```


Open the editor from the command line:


```
vim

```


Type the following to access a special vim help menu:


```
:help 42

```



```
What is the meaning of life, the universe and everything?  *42*
Douglas Adams, the only person who knew what this question really was about is
now dead, unfortunately.  So now you might wonder what the meaning of death
is...

```


Type the following, twice, to exit vim:


```
:q
:q

```


## Emacs Games



Not to be outdone, Emacs, the text-editor famous for including everything but the kitchen sink, includes a surprising number of games that can be accessed from within the editor itself.


First, install emacs.  On Ubunut/Debian, this would be:


```
sudo apt-get install emacs

```


On CentOS/Fedora, execute this command instead:


```
sudo yum install emacs

```


You can find out what games are available by checking out this directory:


```
cd /usr/share/emacs/*/lisp/play
ls

```



```
5x5.elc       decipher.elc    gametree.elc   meese.elc      spook.elc
animate.elc   dissociate.elc  gomoku.elc     morse.elc      studly.elc
blackbox.elc  doctor.elc      handwrite.elc  mpuz.elc       tetris.elc
bruce.el      dunnet.elc      hanoi.elc      pong.elc       yow.elc
bubbles.elc   fortune.elc     landmark.elc   snake.elc      zone.elc
cookie1.elc   gamegrid.elc    life.elc       solitaire.elc

```


To execute them, open Emacs:


```
emacs

```


Next, type the Esc key, followed by x (for execute), and then type the name of the game you wish to start:


```
Esc-x
pong

```


<img style=“border:2px solid black; display:block;margin-left:auto;margin-right:auto” src=“https://assets.digitalocean.com/articles/easter_eggs/emacs_pong.png” alt =“Emacs pong” />


To quit Emacs when you are finished, type Ctrl, followed by x, and then Ctrl and c:


```
Ctrl-x
Ctrl-c

```


# Apt Commands



## Apt-get Cows



On Ubuntu and Debian, the apt-get package manager has had an embedded easter egg for a long time now.


If you type the help command for apt-get, you will get a hint:


```
apt-get help

```


<pre>
. . .
. . .
-c=? Read this configuration file
-o=? Set an arbitrary configuration option, eg -o dir::cache=/tmp
See the apt-get(8), sources.list(5) and apt.conf(5) manual
pages for more information and options.
<span class=“highlight”>This APT has Super Cow Powers.</span>
</pre>


The last line tells us that the easter egg is active in this version of apt.  Type:


```
apt-get moo

```



```
         (__) 
         (oo) 
   /------\/ 
  / |    ||   
 *  /\---/\ 
    ~~   ~~   
...."Have you mooed today?"...

```


## Aptitude Cows?



With apt-get’s affinity for cows, users may be curious as to whether aptitude, another apt tool, also implements a fun easter egg.


We can check the help like before:


```
aptitude help

```


<pre>
. . .
. . .
-u             Download new package lists on startup.
(terminal interface only)
-i             Perform an install run on startup.
(terminal interface only)


```
              <span class="highlight">This aptitude does not have Super Cow Powers.</span>

```


</pre>


Well that is disappointing.  Let’s try it anyways though:


```
aptitude moo

```



```
There are no Easter Eggs in this program.

```


A fairly straight forward answer.  But persistence is important.  Let’s add some verbosity:


```
aptitude -v moo

```



```
There really are no Easter Eggs in this program.

```


And again…:


```
aptitude -vv moo

```



```
Didn't I already tell you that there are no Easter Eggs in this program?

```


If you keep adding more “verbosity”, you will eventually get this:


```
aptitude -vvvvv moo

```



```
All right, you win.

                               /----\
                       -------/      \
                      /               \
                     /                |
   -----------------/                  --------\
   ----------------------------------------------

```


It doesn’t look like too much.  Let’s add another “v”:


```
aptitude -vvvvvv moo

```



```
What is it?  It's an elephant being eaten by a snake, of course.

```


This is a reference to the book The Little Prince by Antoine de Saint-Exupéry.


# Strange Options for Common Programs



There are some strange options available in some common programs that you may wish to check out.


## Insult Users with Sudo



You can configure sudo, used to elevate the privileges of a command, to insult users when they type in an incorrect password.


To do so, edit the sudoers file with a tool called visudo, which edits and validates modifications to the sudo configuration file.


```
sudo visudo

```


Near the top, add a line that reads:


```
Defaults insults

```


Save and close the file.


Next, empty the cache that stores your password for a certain amount of time and then mistype your password for a sudo command:


```
sudo -k
sudo ls

```


<pre>
[sudo] password for demo: <span class=“highlight”># Type an incorrect password here</span>
Have you considered trying to match wits with a rutabaga?
[sudo] password for demo:
My pet ferret can type better than you!
[sudo] password for demo:
Wrong!  You cheating scum!
</pre>


## Script Kiddie Output for Nmap



Nmap is a commonly used network exploration tool that can be used to perform security audits on your system.


Install it on Ubuntu/Debian with the following command:


```
sudo apt-get install nmap

```


On CentOS/Fedora, install it by entering:


```
sudo yum install nmap

```


Nmap provides you with the unusual option of being able to output its data in “script kiddie” format.


Let’s see what the normal output looks like first, by running the command against the Nmap website itself:


```
nmap scanme.nmap.org

```



```
Starting Nmap 5.21 ( http://nmap.org ) at 2013-09-18 17:43 UTC
Nmap scan report for scanme.nmap.org (74.207.244.221)
Host is up (0.072s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.40 seconds

```


Now, let’s enable the alternate output with these options:


```
nmap -oS - scanme.nmap.org

```



```
$tart|ng NMap 5.21 ( http://Nmap.org ) at 2013-09-18 17:45 UTC
Nmap $cAn r3p0rt F0r scanM3.nmaP.oRg (74.207.244.221)
Ho$t 1z Up (0.071z laT3ncy).
Not sh0wN: 998 cl0$Ed p0rt$
POrT   ST4TE $ERV!C3
22/tcp opEn  Ssh
80/tcP 0p3n  HtTp

Nmap d0n3: 1 iP AddrESz (1 h0$t Up) $canNed !n 1.34 secondz

```


Basically, it replaces certain letters with similar looking characters to emulate “hacker” language or leet-speak.


# Command-line Star Wars



There are two different network-reachable, command line Star Wars tributes accessible from your terminal.


## ASCII Art Star Wars Through Telnet



Simon Jansen, Sten Spans, and Mike Edwards created a full Star Wars tribute in ASCII (text) animated art accessible through telnet.  In layman’s terms: you can watch a text version of Star Wars in your terminal!


First, download telnet, a precursor to SSH, if it is not already installed:


On Ubuntu/Debian:


```
sudo apt-get install telnet

```


On CentOS/Fedora:


```
sudo yum install telnet

```


All you have to do from here is point telnet to the correct server:


```
telnet towel.blinkenlights.nl

```



```
                                   /~\                               
         R2-D2!                   |oo )                              
     Where are you?         #     _\=/_    #                         
                             \\  /  _  \  //                         
                              \\//|/.\|\\//                          
                               \/  \_/  \/                           
                                  |\ /|                              
                                  \_ _/                              
                                  | | |                              
                                  | | |                              
                                  []|[]                              
                                  | | |                              
  _______________________________/_]_[_\_____________________________

```


When you’ve had enough, hold Ctrl and ].  You will be given a prompt where you can type “close”:


```
Ctrl-]
close

```


## Star Wars Traceroute



A newer tribute to Star Wars has been achieved by Ryan Werber by naming the network hops to a specific address.


If you run traceroute, a program that traces the path of packets to a remote host, you will see the intro to Star Wars in the network names along the way.


Simply type:


```
traceroute -m 254 -q1 obiwan.scrye.net

```


The route will begin to populate.  After a few stops, you will begin to see the magic:


```
. . .
. . .
15  Episode.IV (206.214.251.1)  77.506 ms
16  A.NEW.HOPE (206.214.251.6)  87.194 ms
17  It.is.a.period.of.civil.war (206.214.251.9)  77.699 ms
18  Rebel.spaceships (206.214.251.14)  78.171 ms
19  striking.from.a.hidden.base (206.214.251.17)  87.624 ms
20  have.won.their.first.victory (206.214.251.22)  86.249 ms
21  against.the.evil.Galactic.Empire (206.214.251.25)  77.505 ms
22  During.the.battle (206.214.251.30)  85.622 ms
23  Rebel.spies.managed (206.214.251.33)  78.121 ms
24  to.steal.secret.plans (206.214.251.38)  77.049 ms
. . .
. . .

```


After going through the introductions to Episodes IV, V, and  VI, Ryan then continues with other avenues of entertainment:


```
99  Were.no.strangers.to.love (206.214.251.206)  77.472 ms
100  You.know.the.rules.and.so.do.I (206.214.251.209)  78.054 ms
101  A.full.commitments.what.Im.thinking.of (206.214.251.214)  78.512 ms
102  I.just.wanna.tell.you.how.Im.feeling (206.214.251.217)  79.884 ms
103  Gotta.make.you.understand (206.214.251.222)  79.427 ms
104  Never.gonna.give.you.up (206.214.251.225)  77.032 ms
105  Never.gonna.let.you.down (206.214.251.230)  78.909 ms
106  Never.gonna.run.around.and.desert.you (206.214.251.233)  80.286 ms

```


# Installing More Fun



There are a few programs that you probably don’t need for any other purpose, but can be fun if you have some time.


## Learn from your Typos



If you’ve ever accidentally typed sl when you meant to list a directory’s contents with ls, then you may want to install a program “sl”.


On Ubuntu/Debian:


```
sudo apt-get install sl

```


On CentOS/Fedora:


```
sudo yum install sl

```


Now, whenever you accidentally type “sl” instead of “ls”, you’ll have to smile:


```
sl

```



```
                          (  ) (@@) ( )  (@)  ()    @@    O     @     O     @
                     (@@@)
                 (    )
              (@@@@)

            (   )
        ====        ________                ___________
    _D _|  |_______/        \__I_I_____===__|_________|
     |(_)---  |   H\________/ |   |        =|___ ___|      _________________
     /     |  |   H  |  |     |   |         ||_| |_||     _|                \___
    |      |  |   H  |__--------------------| [___] |   =|
    | ________|___H__/__|_____/[][]~\_______|       |   -|
    |/ |   |-----------I_____I [][] []  D   |=======|____|______________________
  __/ =| o |=-~~\  /~~\  /~~\  /~~\ ____Y___________|__|________________________
   |/-=|___|=O=====O=====O=====O   |_____/~\___/          |_D__D__D_|  |_D__D__D
    \_/      \__/  \__/  \__/  \__/      \_/               \_/   \_/    \_/   \

```


A train will chug across your screen each time.


## Fun with Cowsay and Fortune



If you need some more cheap amusement at the command line, and didn’t get your fill of cows from the “apt” easter egg, you can download cowsay and fortune.


On Ubuntu/Debian:


```
sudo apt-get install fortune cowsay

```


One CentOS/Fedora:


```
sudo yum install fortune cowsay

```


Cowsay inserts any input into a word bubble and draws an ASCII cow to talk to you:


```
cowsay "hello, I'm a cow"

```



```
 __________________
< hello, I'm a cow >
 ------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```


The fortune program spits out quotations, fortunes, jokes, nonsense that can be piped into cowsay:


```
fortune | cowsay

```



```
 ________________________________________
/ Q: What looks like a cat, flies like a \
| bat, brays like a donkey, and          |
|                                        |
\ plays like a monkey? A: Nothing.       /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```


If you’re not too fond of cows, you can get other characters as well:


```
fortune | cowsay -f tux

```



```
 _____________________________________
/ You never know how many friends you \
| have until you rent a house on the  |
\ beach.                              /
 -------------------------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/

```


For a full list of the available characters, type:


```
cowsay -l

```



```
Cow files in /usr/share/cowsay/cows:
apt beavis.zen bong bud-frogs bunny calvin cheese cock cower daemon default
dragon dragon-and-cow duck elephant elephant-in-snake eyes flaming-sheep
ghostbusters gnu head-in hellokitty kiss kitty koala kosh luke-koala
mech-and-cow meow milk moofasa moose mutilated pony pony-smaller ren sheep
skeleton snowman sodomized-sheep stegosaurus stimpy suse three-eyes turkey
turtle tux unipony unipony-smaller vader vader-koala www

```


My personal favorite is the stegosaurus:


```
fortune | cowsay -f stegosaurus

```



```
 _________________________________________
/ Q: What lies on the bottom of the ocean \
\ and twitches? A: A nervous wreck.       /
 -----------------------------------------
\                             .       .
 \                           / `.   .' " 
  \                  .---.  <    > <    >  .---.
   \                 |    \  \ - ~ ~ - /  /    |
         _____          ..-~             ~-..-~
        |     |   \~~~\.'                    `./~~~/
       ---------   \__/                        \__/
      .'  O    \     /               /       \  " 
     (_____,    `._.'               |         }  \/~~~/
      `----.          /       }     |        /    \__/
            `-.      |       /      |       /      `. ,~~|
                ~-.__|      /_ - ~ ^|      /- _      `..-'   
                     |     /        |     /     ~-.     `-. _  _  _
                     |_____|        |_____|         ~ - . _ _ _ _ _>

```


As you can see, not very useful, but pretty fun.


# Conclusion



This guide probably didn’t impart any essential knowledge or improve your Linux abilities, but hopefully it helped you relax and perhaps even explore your system a little bit.


Let us know in the comments if you have any other good easter eggs or unusual, fun commands.


<div style=“text-align: right; font-size: smaller;”>By Justin Ellingwood</div>


