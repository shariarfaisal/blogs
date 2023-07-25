# Importing Packages in Go

```Go``` ```Development```

## Introduction


The ability to borrow and share code across different projects is foundational to any widely-used programming language—and the entire open-source community. Borrowing code empowers programmers to spend most of their time writing code that is specific to their needs, and yet often, some of their new code ends up being useful to others. They may then decide to organize those reusable parts into a unit and share it within their team or the wider programming community.


In Go, the basic unit of reusable code is called a package. Even the simplest Go program is its own package, and probably uses at least one other package. In this tutorial, you will write two tiny programs: one that uses a standard library package to generate random numbers, and another that uses a popular third-party package to generate UUIDs. Then you will optionally write a longer program that compares two similar standard library packages, importing and using both packages even though they have the same base name. Finally, you will use the goimports tool to see how to format your imports.



Note: there is also a higher-level unit of reusable code in Go: the module. Modules are versioned collections of packages. You will explore modules in a later article, How to Use Go Modules.

# Prerequisites


Before starting this tutorial, you only need to install Go. Read the right tutorial for your operating system:


- How To Install Go On Ubuntu 20.04
- How To Install Go and Set Up a Local Programming Environment on macOS
- How To Install Go and Set Up a Local Programming Environment on Windows 10

# Step 1 — Using Standard Library Packages


Like most languages, Go comes with a built-in library of reusable code that you can use for common tasks. You don’t need to write your own code to format and print strings, or send an HTTP request, for example. The Go standard library has packages for both of those tasks and many others.


The program in How To Write Your First Program In Go used the fmt and strings packages from the standard library. Let’s write another program that uses the math/rand package to generate some random numbers.


Open a new file called random.go in nano or your favorite text editor:


```
nano random.go


```


Let’s create a program that prints five random integers from zero to nine. Paste the following into your editor:


```
package main

import "math/rand"

func main() {
	for i := 0; i < 5; i++ {
		println(rand.Intn(10))
	}
}

```


This program imports the math/rand package and uses it by referencing its base name, rand. This is the name that appears in the package <pkgname> declaration at the top of each Go source file in the package.


Each iteration of the for-loop calls rand.Intn(10) to generate a random integer between zero and nine (the 10 is not inclusive), then prints the integer to the console.



Notice that the call to println() does not reference a package name. This is a builtin function that does not need to be imported. Normally you would use the fmt.Println() function from the fmt package here, but this program uses println() to introduce builtin functions.

Save the program. If you are using nano, press CTRL+X then Y and ENTER to confirm your changes. Then run the program:


```
go run random.go

```


You should see five integers from zero to nine:


```
Output1
7
7
9
1

```


It looks like the random number generator is working, yet notice that if you run the program again and again, it prints the same numbers every time rather than new random numbers as you would expect. That’s because we didn’t call the rand.Seed() function to initialize the number generator with a unique value. If you do not do this, the package behaves as if rand.Seed(1) was called, and so it will generate the same “random” numbers each time.


So you need to seed the number generator with a unique value each time the program is run. Programmers often use the current timestamp in nanoseconds. To get that, you’ll need the time package. Open random.go in your editor again and paste the following:


```
package main

import (
	"math/rand"
	"time"
)

func main() {
	now := time.Now()
	rand.Seed(now.UnixNano())
println("Numbers seeded using current date/time:", now.UnixNano())
	for i := 0; i < 5; i++ {
		println(rand.Intn(10))
	}
}


```


When importing more than one package, you can use parentheses to create a block of imports. By using a block you can avoid repeating the import keyword on every line, which makes your code cleaner.


First you are getting the current system time via the time.Now() function, which returns a Time struct. Then you are passing the time to the rand.Seed() function. That function takes a 64-bit integer (int64), so you need to use the Time.UnixNano() method on your now struct to pass in the time in nanoseconds. Finally, you are printing the time used to seed the random number generator.


Now save and run the program again:


```
go run random.go


```


You should see output similar to this:


```
OutputNumbers seeded using current date/time: 1674489465616954000
2
6
3
1
0

```


If you run the program several times, you should now see different integers each time, along with the unique integer used to seed the random number generator.


Let’s edit the program one more time to print the seed time in a more user-friendly format. Edit the line containing the first println() call to look like this:


```
	println("Numbers seeded using current date/time:", now.Format(time.StampNano))

```


Now you are calling the Time.Format() method and passing in one of many formats defined in the time package. The time.StampNano constant (const) is a string, and passing it to Time.Format() lets you print the month, day, and time, down to the nanosecond. Save and run the program one more time:


```
go run random.go

```


```
OutputNumbers seeded using current date/time: Jan 23 10:01:50.721413000
7
6
3
7
3

```


That’s nicer than seeing a huge integer representing the number of nanoseconds that have passed since January 1, 1970.


What if your program needed not random integers, but UUIDs, which many programmers use as globally unique identifiers for pieces of data across their deployment? The Go standard library doesn’t have a package for generating those, but the community does. Let’s look now at how to download and use third-party packages.


# Step 2 — Using Third-Party Packages


One of the most popular packages for generating UUIDs is github.com/google/uuid. Third-party packages are always known by their fully-qualified names, which include the site that hosts the code (e.g. github.com), the user or organization that develops it (e.g. google), and the base name (e.g. uuid). You will use a package’s fully-qualified name when importing it, when reading its documentation on pkg.go.dev, and elsewhere. When referencing it in your code statements, however, you will only use the base name.


Before downloading a package, you need to initialize a module, which is how Go manages a program’s dependencies and their versions. To initialize a module, use go mod init and pass in a fully-qualified name for your own package. If you wanted to host your module on GitHub under your username “sammy”, you could initialize your module like this:


```
go mod init github.com/sammy/random


```


This creates a file called go.mod. Let’s look at that file:


```
cat go.mod


```


```
Outputmodule github.com/sammy/random

go 1.19

```


This file must appear at the root of any Go repository that will be distributed as a Go module. It must at least define the module name and the Go version it requires. Your own Go version may be different from what is displayed above.


You will not distribute your module in this tutorial, but this step is necessary to download and use third-party packages.


Now use the go get command to download the third-party UUID module:


```
go get github.com/google/uuid


```


This downloads the latest version:


```
Outputgo: downloading github.com/google/uuid v1.3.0
go: added github.com/google/uuid v1.3.0

```


The package is placed in your local directory $GOPATH/pkg/mod/. If you do not have $GOPATH explicitly set in your shell, its default value is $HOME/go. If your local user is sammy and you’re running macOS, for example, this package will be downloaded in /Users/sammy/go/pkg/mod. You can run go env GOMODCACHE to see where Go will put downloaded modules.


Let’s view your new dependency’s own go.mod file:


```
cat /Users/sammy/go/pkg/mod/github.com/google/uuid@v1.3.0/go.mod


```


```
Outputmodule github.com/google/uuid

```


It looks like this module has no third-party dependencies; it only uses packages from the Go standard library.


Notice that the module’s version is included in its directory name. This allows you to develop against and test multiple versions of the same package, either within one program or across different programs you are writing.


Now look again at your own go.mod file:


```
cat go.mod


```


```
Outputmodule github.com/sammy/random

go 1.19

require github.com/google/uuid v1.3.0 // indirect

```


The go get command noticed the go.mod file in your current directory and updated it to reflect your program’s new dependency, including its version. Now you can use this package in your package. Open a new file called uuid.go in your text editor and paste in the following:


```
package main

import "github.com/google/uuid"

func main() {
	for i := 0; i < 5; i++ {
		println(uuid.New().String())
	}
}

```


This program is similar to random.go, but it uses github.com/google/uuid to print five UUIDs instead of using math/rand to print five integers.


Save the new file and run it:


```
go run uuid.go


```


Your output should be similar to this:


```
Output243811a3-ddc6-4e26-9649-060622bba2b0
b8129aa1-3803-4dae-bd9f-6ba8817f44b2
3ae27c71-caa8-4eaa-b8e6-e629b7c1cb49
37e06706-004d-4504-ad37-03c68252bb0f
a2da6904-a6ab-4cc2-849b-d9d25a86e373

```


The github.com/google/uuid package generates these UUIDs by using the standard library package crypto/rand, which is similar to but different from the math/rand package you used in random.go. What if you needed to use both packages? They both have a base name of rand, so how can you reference each distinct package in your code? Let’s look at that next.


# Step 3 — Importing Identically-Named Packages


The documentation for math/rand says that it actually generates pseudo-random numbers and is “unsuitable for security-sensitive work”. For that kind of work, you would use crypto/rand instead. But what if the quality of the integers’ randomness didn’t matter for your program? Maybe you really only needed arbitrary numbers.


You could write a program to compare the performance of these two rand packages, but you cannot reference both packages by the rand name throughout such a program. To get around this, Go allows you to choose an alternative local name for a package when importing it.


Here’s how to import two packages with the same base name:


```
import (
    “math/rand”
    crand “crypto/rand”
)

```


You can choose any alias you like (as long as it does not match other package names) and put it to the left of the fully-qualified package name. In this case, the alias is crand. Note that the alias does not have quotation marks around it. Throughout the rest of the source file containing this import block, you can access the crypto/rand package using your chosen name crand.


You may also import packages into your own namespace (using . as the alias) or as a blank identifier (using _ as the alias). For more on that, read the Go documentation.


To illustrate how you might want to use identically-named packages, let’s create and run a longer program that generates random integers using both packages and measures the time taken in each case. This part is optional; if you’re not interested, skip to Step 4.


## Comparing math/rand and crypto/rand (Optional)


##$ Getting A Command-Line Argument


Start by opening a third new file in your working directory named compare.go and paste in the following program:


```
package main

import "os"
import "strconv"

func main() {
	// User must pass in number of integers to generate
	if len(os.Args) < 2 {
		println("Usage:\n")
		println("  go run compare.go <numberOfInts>")
		return
	}
	n, err := strconv.Atoi(os.Args[1])
	if err != nil { // Maybe they didn't pass an integer
		panic(err) 
	}

	println("User asked for " + strconv.Itoa(n) + " integers.")
}

```


This code prepares you to generate a user-given number of pseudo-random integers using both rand packages later on. It uses the os and strconv standard library packages to convert a single command-line argument into an integer. If no argument is passed, it prints a usage statement and exits.


Run the program with a single argument of 10 to make sure it works:


```
go run compare.go 10


```


```
[seconary_label Output]
User asked for 10 integers.

```


So far, so good. Now let’s generate random integers using the math/rand package as you did before, but this time you’ll compute the time it takes to do it.


##$ Phase 1 — Measuring math/rand Performance


Remove the final println() statement and replace it with the following:


```
	// Phase 1 — Using math/rand
	// Initialize the byte slice
	b1 := make([]byte, n)
	// Get the time
	start := time.Now()
	// Initialize the random number generator
	rand.Seed(start.UnixNano())
	// Generate the pseudo-random numbers
	for i := 0; i < n; i++ {
		b1[i] = byte(rand.Intn(256)) // Where the magic happens!
	}
	// Compute the elapsed time
	elapsed := time.Now().Sub(start)
	// In case n is very large, only print a few numbers
	for i := 0; i < 5; i++ {
		println(b1[i])
	}
	fmt.Printf("Time to generate %v pseudo-random numbers with math/rand: %v\n", n, elapsed)

```


First, you are using the builtin function make() to create an empty slice of bytes ([]byte) to hold the generated integers (as bytes). Its length is the number of integers the user asked for.


Then, you are getting the current time and seeding the random number generator with it, as you did in random.go in Step 1.


After that, you are generating n pseudo-random integers between 0 and 255, converting each to a byte and putting it in your byte slice. Why integers between 0 and 255? Because the crypto/rand-using code you are about to write generates bytes, not integers of any size, and we should perform an equal comparison of the packages. A byte, which is 8 bits, may be represented by an unsigned integer from 0 to 255. (In fact, the byte type in Go is an alias for the uint8 type.)


Finally, you are only printing the first five bytes in case the user requested a very large number of integers. It’s nice to see a few integers just to convince yourself the number generator is working.


Before running the program, don’t forget to add the new packages you’re using to your import block:


```
import (
	"fmt"
	"math/rand"
	"os"
	"strconv"
	"time"
)

```


After the highlighted packages have been added, run the program:


```
go run compare.go 10


```


```
Output189
203
209
238
235
Time to generate 10 pseudo-random numbers with math/rand: 33.417µs

```


It took 33.417 microseconds to generate ten integers between 0 and 255 and store them in a byte slice. Let’s see how that compares to the performance of crypto/rand.


##$ Phase 2 — Measuring crypto/rand Performance


Before adding the code that uses crypto/rand, add the package to your import block as shown earlier:


```
import (
	"fmt"
	"math/rand"
	crand "crypto/rand"
	"os"
	"strconv"
	"time"
)

```


Then, append the following code to the end of your main() function:


```
	// Phase 2 — Using crypto/rand
	// Initialize the byte slice
	b2 := make([]byte, n)
	// Get the time (Note: no need to seed the random number generator)
	start = time.Now()
	// Generate the pseudo-random numbers
	_, err = crand.Read(b2) // Where the magic happens!
	// Compute the elapsed time
	elapsed = time.Now().Sub(start)
	// exit if error
	if err != nil {
		panic(err)
	}
	// In case n is very large, only print a few numbers
	for i := 0; i < 5; i++ {
		println(b2[i])
	}
	fmt.Printf("Time to generate %v pseudo-random numbers with crypto/rand: %v\n", n, elapsed)

```


This code mirrors the Phase 1 code as closely as possible. You are generating a byte slice of size n, getting the current time, generating the n bytes, computing the elapsed time, and finally printing five integers and the elapsed time. When using the crypto/rand package, there is no need to explicitly seed the random number generator.



Note: crypto/rand also includes an Int() function, but our example here uses Read() because that is what is used by the only snippet of example code in the package’s documentation. Feel free to explore the crypto/rand package on your own.

Your entire compare.go program should look like this:


```
package main

import (
	"fmt"
	"math/rand"
	crand "crypto/rand"
	"os"
	"strconv"
	"time"
)

func main() {
	// User must pass in number of integers to generate
	if len(os.Args) < 2 {
		println("Usage:\n")
		println("  go run compare.go <numberOfInts>")
		return
	}
	n, err := strconv.Atoi(os.Args[1])
	if err != nil { // Maybe they didn't pass an integer
		panic(err)
	}

	// Phase 1 — Using math/rand
	// Initialize the byte slice
	b1 := make([]byte, n)
	// Get the time
	start := time.Now()
	// Initialize the random number generator
	rand.Seed(start.UnixNano())
	// Generate the pseudo-random numbers
	for i := 0; i < n; i++ {
		b1[i] = byte(rand.Intn(256)) // Where the magic happens!
	}
	// Compute the elapsed time
	elapsed := time.Now().Sub(start)
	// In case n is very large, only print a few numbers
	for i := 0; i < 5; i++ {
		println(b1[i])
	}
	fmt.Printf("Time to generate %v pseudo-random numbers with math/rand: %v\n", n, elapsed)

	// Phase 2 — Using crypto/rand
	// Initialize the byte slice
	b2 := make([]byte, n)
	// Get the time (Note: no need to seed the random number generator)
	start = time.Now()
	// Generate the pseudo-random numbers
	_, err = crand.Read(b2) // Where the magic happens!
	// Compute the elapsed time
	elapsed = time.Now().Sub(start)
	// exit if error
	if err != nil {
		panic(err)
	}
	// In case n is very large, only print a few numbers
	for i := 0; i < 5; i++ {
		println(b2[i])
	}
	fmt.Printf("Time to generate %v pseudo-random numbers with crypto/rand: %v\n", n, elapsed)
}

```


Run the program with a parameter of 10 to generate ten 8-bit integers using each package:


```
go run compare.go 10


```


```
Output145
65
231
211
250
Time to generate 10 pseudo-random numbers with math/rand: 32.5µs
101
188
250
45
208
Time to generate 10 pseudo-random numbers with crypto/rand: 42.667µs

```


In this example execution, the math/rand package was a little faster than crypto/rand. Try running compare.go several times with a parameter of 10. Then try generating a thousand integers—or a million. Which package is consistently faster?


This example program is meant to show how you might use two packages with the same name and a similar purpose within the same program. It is not meant as a recommendation of one of these packages over the other. If you wanted to extend compare.go, you might use the math/stats package to compare the randomness of the bytes each package generates. Whatever program you are writing, it is up to you to evaluate different packages and choose the best one for your needs.


Finally, let’s look at how to format import declarations using the goimports tool.


# Step 4 — Using Goimports


Sometimes when you’re in the flow of programming, you forget to import the packages you’re using, or remove the ones you’re not. The goimports command-line tool not only formats your import declaration(s)—and the rest of your code, making it a more-featureful replacement for gofmt—it also adds any missing imports for packages your code references and removes imports for unused packages as well.


The tool is not shipped with Go by default, so install it now using go install:


```
go install golang.org/x/tools/cmd/goimports@latest


```


This puts the goimports binary in your $GOPATH/bin directory. If you followed the tutorial How To Install Go and Set Up a Local Programming Environment on macOS (or the matching tutorial for your operating system), this directory should already be in your $PATH. Try running the tool:


```
goimports --help


```


If you don’t see the tool’s usage statement, $GOPATH/bin is not in your $PATH. Read the Go environment setup guide for your operating system to set that up.


Once goimports is in your $PATH, remove your entire import block from random.go. Then, run goimports with the -d option to show a diff of what it wants to add:


```
goimports -d random.go


```


```
Outputsdiff -u random.go.orig random.go
--- random.go.orig	2023-01-25 16:29:38
+++ random.go	2023-01-25 16:29:38
@@ -1,5 +1,10 @@
 package main
 
+import (
+	"math/rand"
+	"time"
+)
+
 func main() {
 	now := time.Now()
 	rand.Seed(now.UnixNano())

```


Impressive, but goimports can also recognize and add third-party packages if they are installed locally via go get. Remove the import from uuid.go and run goimports on it:


```
goimports -d uuid.go


```


```
Outputdiff -u uuid.go.orig uuid.go
--- uuid.go.orig	2023-01-25 16:32:56
+++ uuid.go	2023-01-25 16:32:56
@@ -1,8 +1,9 @@
 package main
 
+import "github.com/google/uuid"
+
 func main() {
 	for i := 0; i < 5; i++ {
 		println(uuid.New().String())
 	}
 }

```


Now edit uuid.go and:


1. Add an import for math/rand, which the code does not use.
2. Change the builtin println() function to fmt.Println(), but do not add import “fmt”.

uuid.go
```
package main

import "math/rand"

func main() {
	for i := 0; i < 5; i++ {
		fmt.Println(uuid.New().String())
	}
}

```


Save the file and run goimports on it again:


```
goimports -d uuid.go


```


```
Outputdiff -u uuid.go.orig uuid.go
--- uuid.go.orig	2023-01-25 16:36:28
+++ uuid.go	2023-01-25 16:36:28
@@ -1,10 +1,13 @@
 package main
 
-import "math/rand"
+import (
+	"fmt"
 
+	"github.com/google/uuid"
+)
+
 func main() {
 	for i := 0; i < 5; i++ {
 		fmt.Println(uuid.New().String())
 	}
 }

```


The tool not only adds missing imports, it removes unnecessary ones. Notice also that it puts imports into a block within parentheses rather than using the import keyword on each line.


To write the changes to uuid.go (or any file) rather than outputting them to the terminal, use goimports with the -w option:


```
goimports -w uuid.go


```


You should configure your text editor or IDE to call goimports when saving a *.go file so your source files are always well-formatted. As mentioned earlier, goimports supersedes gofmt, so if your text editor is already using gofmt, configure it to use goimports instead.


Another thing goimports does is enforce a particular sorting order to your imports. Trying to manually maintain the order of your imports can be tedious and prone to errors, so allow goimports to handle this for you.


If the Go team changes the official format for Go source files, they will update goimports to reflect those changes, so you should periodically update goimports to ensure your code always conforms to the latest standards.


# Conclusion


In this tutorial, you created and ran two programs under ten lines long using popular packages to help you generate random numbers and UUIDs. The Go ecosystem is so rich in well-written packages that writing a new program in Go should be a pleasure, and you will find yourself writing useful programs that suit your specific needs with less effort than you might think.


Check out the next tutorial in the series, How To Write Packages in Go. Then, if you’re up for it, look ahead to How To Use Go Modules to understand how to group packages together and distribute them to others as one unit.


