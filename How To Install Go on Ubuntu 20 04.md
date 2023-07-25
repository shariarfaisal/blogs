# How To Install Go on Ubuntu 20 04

```Ubuntu``` ```Go``` ```Ubuntu 20.04```

## Introduction


Go, sometimes referred to as “Golang”, is an open-source programming language that was released by Google in 2012. Google’s intention was to create a programming language that could be learned quickly.


Since its release, Go has become highly popular among developers and is used for various applications ranging from cloud or server-side applications, to artificial intelligence and robotics. This tutorial outlines how to download and install the latest version of Go (currently version 1.16.7) on an Ubuntu 20.04 server, build the famous Hello, World! application, and make your Go code into an executable binary for future use.


# Prerequisites


This tutorial requires an Ubuntu 20.04 system configured with a non-root user with sudo privileges and a firewall as described in Initial Server Setup with Ubuntu 20.04.


# Step 1 — Installing Go


In this step, you will install Go on your server.


First, connect to your Ubuntu server via ssh:


```
ssh sammy@your_server_ip


```


Next, navigate to the official Go downloads page in your web browser. From there, copy the URL for the current binary release’s tarball.


As of this writing, the latest release is go1.16.7. To install Go on an Ubuntu server (or any Linux server, for that matter), copy the URL of the file ending with linux-amd64.tar.gz.


Now that you have your link ready, first confirm that you’re in the home directory:


```
cd ~


```


Then use curl to retrieve the tarball, making sure to replace the highlighted URL with the one you just copied. The -O flag ensures that this outputs to a file, and the L flag instructs HTTPS redirects, since this link was taken from the Go website and will redirect here before the file downloads:


```
curl -OL https://golang.org/dl/go1.16.7.linux-amd64.tar.gz


```


To verify the integrity of the file you downloaded, run the sha256sum command and pass it to the filename as an argument:


```
sha256sum go1.16.7.linux-amd64.tar.gz


```


This will return the tarball’s SHA256 checksum:


```
Outputgo1.16.7.linux-amd64.tar.gz
7fe7a73f55ba3e2285da36f8b085e5c0159e9564ef5f63ee0ed6b818ade8ef04  go1.16.7.linux-amd64.tar.gz

```


If the checksum matches the one listed on the downloads page, you’ve done this step correctly.


Next, use tar to extract the tarball. This command includes the -C flag which instructs tar to change to the given directory before performing any other operations. This means that the extracted files will be written to the /usr/local/ directory, the recommended location for installing Go…  The x flag tells tar to extract, v tells it we want verbose output (a listing of the files being extracted), and f tells it we’ll specify a filename:


```
sudo tar -C /usr/local -xvf go1.16.7.linux-amd64.tar.gz


```


Although /usr/local/go is the recommended location for installing Go, some users may prefer or require different paths.


# Step 2 — Setting Go Paths


In this step, you will set paths in your environment.


First, set Go’s root value, which tells Go where to look for its files.
You can do this by editing the .profile file, which contains a list of commands that the system runs every time you log in.


Use your preferred editor to open .profile, which is stored in your user’s home directory. Here, we’ll use nano:


```
sudo nano ~/.profile


```


Then, add the following information to the end of your file:


sudo nano ~/.profile
```
. . .
export PATH=$PATH:/usr/local/go/bin

```


After you’ve added this information to your profile, save and close the file. If you used nano, do so by pressing CTRL+X, then Y, and then ENTER.


Next, refresh your profile by running the following command:


```
source ~/.profile


```


After, check if you can execute go commands by running go version:


```
go version


```


This command will output the release number of whatever version of Go is installed on your system:


```
Outputgo version go1.16.7 linux/amd64

```


This output confirms that you are now running Go on your server.


# Step 3 — Testing Your Install


Now that Go is installed and the paths are set for your server, you can try creating your Hello, World! application to ensure that Go is working.


First, create a new directory for your Go workspace, which is where Go will build its files:


```
mkdir hello


```


Then move into the directory you just created:


```
cd hello


```


When importing packages, you have to manage the dependencies through the code’s own module. You can do this by creating a go.mod file with the go mod init command:


```
go mod init your_domain/hello


```


Next, create a Hello, World! Go file in your preferred text editor:


```
nano hello.go


```


Add the following text into your hello.go file:


hello.go
```
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}

```


Then, save and close the file by pressing CTRL+X, then Y, and then ENTER.


Test your code to check that it prints the Hello, World! greeting:


```
go run .


```


```
OutputHello, World!

```


The go run command compiles and runs the Go package from a list of .go source files from the new hello directory you created and the path you imported. But, you can also use go build to make an executable file that can save you some time.


# Step 4 — Turning Your Go Code Into a Binary Executable


The go run command is typically used as a shortcut for compiling and running a program that requires frequent changes. In cases where you’ve finished your code and want to run it without compiling it each time, you can use go build to turn your code into an executable binary. Building your code into an executable binary consolidates your application into a single file with all the support code necessary to execute the binary. Once you’ve built the binary executable, you can then run go install to place your program on an executable file path so you can run it from anywhere on your system. Then, your program will successfully print Hello, World! when prompted and you won’t need to compile the program again.


Try it out and run go build. Make sure you run this from the same directory where your hello.go file is stored:


```
go build


```


Next, run ./hello to confirm the code is working properly:


```
./hello


```


```
OutputHello, World!

```


This confirms that you’ve successfully turned your hello.go code into an executable binary. However, you can only invoke this binary from within this directory. If you wanted to run this program from a different location on your server, you would need to specify the binary’s full file path in order to execute it.


Typing out a binary’s full file path can quickly become tedious. As an alternative, you can run the go install command. This is similar to go build but instead of leaving the executable in the current directory, go install places it in the $GOPATH/bin directory, which will allow you to run it from any location on your server.


In order to run go install successfully, you must pass it the install path of the binary you created with go build. To find the binary’s install path, run the following go list command:


```
go list -f ‘{{.Target}}’


```


go list generates a list of any named Go packages stored in the current working directory. The f flag will cause go list to return output in a different format based on whatever package template you pass to it. This command tells it to use the Target template, which will cause go list to return the install path of any packages stored in this directory:


```
Output‘/home/sammy/go/bin/hello

```


This is the install path of the binary file you created with go build. This means that the directory where this binary is installed is /home/sammy/go/bin/.


Add this install directory to your system’s shell path. Be sure to change the highlighted portion of this command to reflect the install directory of the binary on your system, if different:


```
export PATH=$PATH:/home/sammy/go/bin/


```


Finally, run go install to compile and install the package:


```
go install


```


Try running this executable binary by just entering hello


```
hello


```


```
OutputHello, World!

```


If you received the Hello, World! output, you’ve successfully made your Go program executable from both a specific and unspecified path on your server.


# Conclusion


By downloading and installing the latest Go package and setting its paths, you now have a system to use for Go development. You can find and subscribe to additional articles on installing and using Go within our “Go” tag


