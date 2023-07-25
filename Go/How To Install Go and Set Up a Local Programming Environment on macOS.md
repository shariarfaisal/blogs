# How To Install Go and Set Up a Local Programming Environment on macOS

```macOS``` ```Go``` ```Development```

## Introduction


Go is a programming language that was born out of frustration. At Google, developers were tired of having to make tradeoffs when picking the language for a new project. Some languages executed efficiently but took a long time to compile, while others were easy to write but ran inefficiently in production. So Google invented Go and designed the language to have it all: fast compilation, fast execution, easy to write, and easy to deploy.


While Go is a versatile language that can be used for many kinds of projects, from web applications to command-line tools, it is particularly well suited for distributed systems and microservice architectures, earning it a reputation as the language of the cloud. It helps the modern programmer do more with a strong set of tooling, removing debates over formatting by making the format part of the language specification, as well as making deployment easy by compiling each program and all of its dependencies into a single binary. Go is easy to learn, with a very small set of keywords, which makes it a great choice for beginner and veteran developers alike.


In this introductory tutorial, you will install Go on your local macOS machine and run your first program to prove that the installation worked.


# Prerequisites


You need a macOS computer with administrative access that is connected to the internet.


# Step 1 — Opening Terminal


The macOS Terminal is an application you can use to access the command line interface. You can find it by going into Finder, navigating to the Applications folder, and then into the Utilities folder. From here, double-click the Terminal.


Now that you have opened up Terminal, you can download and install Xcode, a package of developer tools that you will need in order to install Go.


# Step 2 — Installing Xcode


Xcode is an integrated development environment (IDE) that comprises software development tools for macOS. You can check if Xcode is already installed by typing the following in the Terminal:


```
xcode-select -p


```


The following output means that Xcode is already installed:


```
Output/Library/Developer/CommandLineTools

```


If you received an error, install Xcode from the App Store and accept the default options.


Once Xcode is installed, return to your Terminal window. Next, you’ll need to install Xcode’s separate Command Line Tools app, which you can do by typing:


```
xcode-select --install


```


At this point, Xcode and its Command Line Tools app are fully installed, and you are ready to install the package manager Homebrew.


# Step 3 — Installing and Setting Up Homebrew


While the macOS Terminal is very similar to Linux terminals and those of other Unix systems, it does not ship with an official command-line package manager like Linux distributions do. A package manager helps you install software, upgrade and configure it, and uninstall it, either interactively from a terminal or within scripts. There are a few open-source (and unofficial) package managers for macOS, and Homebrew has emerged as one of the most popular. It offers a quick and flexible way to install and update Go on macOS.


To install Homebrew, run this in Terminal:


```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"


```


This command downloads a script from GitHub and installs Homebrew. If you need to enter your password, note that your keystrokes will not display in the Terminal window but they will be recorded. Simply press the return key once you’ve entered your password. Otherwise press y for “yes” whenever you are prompted to confirm the installation.


Once the installation is complete, you’ll put Homebrew’s directories at the top of the PATH environment variable so that anything you install via Homebrew will take precedence over any identically-named program installed by default on macOS (if there is one). Since macOS does not ship with Go, putting Homebrew at the top of your PATH is not strictly necessary in this case, but to accommodate other cases, many developers prefer to add Homebrew to the top of their PATH.


To do that, create or open the file ~/.zprofile with the command-line text editor nano:


```
nano ~/.zprofile


```



Note: If you are running a version of macOS older than 10.15 Catalina, your Terminal likely uses the Bash shell (/bin/bash) rather than the Z-shell (/bin/zsh). In that case, you need to create or open the file ~/.bash_profile instead of ~/.zprofile. To check which shell you are using, run echo $SHELL.

Add the following line to the file:


```
eval "$(/opt/homebrew/bin/brew shellenv)"

```


Exit nano by typing CTRL+x, and when prompted to save the file press y and then ENTER.


Now activate these changes:


```
source ~/.zprofile


```


You can make sure Homebrew was successfully installed by typing:


```
brew doctor


```


If no updates are required at this time, the output will read:


```
OutputYour system is ready to brew.

```


Otherwise, you may get a warning to run another command such as brew update to ensure that your installation of Homebrew is up to date.


Once Homebrew is ready, you can install Go.


# Step 4 — Installing Go


You can search all available Homebrew packages with the brew search command. For the purpose of this tutorial, you will search for Go-related packages or modules:


```
brew search golang


```



Note: Do not run brew search go, as it will return too many results. The Go language is often called Golang, so use golang as the search term to narrow down results.

The Terminal will output a list of what you can install:


```
Outputgolang	golang-migrate golangci-lint glslang

```


You want that first result: golang. Install it now:


```
brew install golang


```


The installation may take a few minutes. When it’s done, check the version of Go that you installed:


```
go version


```


Homebrew should have installed the latest stable version of Go. At the time of this writing, that version is 1.19.4.


To update Go in the future, you can run these two commands to first update Homebrew and then update Go: (You don’t have to do this now, as you just installed the latest version.)


```
brew update
brew upgrade golang


```


brew update will update the formulae for Homebrew itself, ensuring you have the latest information for packages you want to install. brew upgrade golang will update the golang package to the latest release.


With Go installed, you are ready to compile and run your first program.


# Step 6 — Writing Hello World in Go


This section will not explain anything about Go programming. The goal is only to compile and run the simplest program imaginable to convince yourself that Go is working.


From your home directory, create a new file using a text editor like nano:


```
nano hello.go


```


Paste in this program:


```
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}

```


Exit nano by typing CTRL+x, and when prompted to save the file press y and then ENTER.


Then compile and run the program with this single command:


```
go run hello.go


```


You should see this output:


```
OutputHello, World!

```


Go is alive! You’re ready to embark on your adventures in Go.


# Conclusion


This tutorial provided the briefest introduction to the Go programming language. You installed Go and ran your first program. To learn more about and expand upon your Hello World program, read How to Write Your First Program in Go next.


