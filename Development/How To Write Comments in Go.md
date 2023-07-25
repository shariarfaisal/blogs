# How To Write Comments in Go

```Go``` ```Development```

## Introduction


Almost all programming languages have a syntax for adding comments to code, and Go is no exception. Comments are lines in a program that explain in human language how the code works or why it is written as it is. They are ignored by the compiler, but not by careful programmers. Comments add invaluable context that helps your collaborators—and your future self—to avoid pitfalls and write more maintainable code.


Ordinary comments within any package explain why that code does what it does. They are notes and warnings for a package’s developers. Doc comments summarize what each component of a package does and how it works, providing example code and command usage as well. They are the official package documentation for its users.


In this article, we’ll look at some real comments from a few Go packages to illustrate not only how comments look in Go, but what they should convey.


# Ordinary Comments


A comment in Go begins with two forward slashes (//), followed by one space (not required, but idiomatic), then the comment. It may appear just above or to the right of the code it concerns. When above, it is indented to align with the code.


This Hello World program contains a single comment on its own line:


hello.go
```
package main

import "fmt"

func main() {
	// Say hi via the console
	fmt.Println("Hello, World!")
}

```



Note: if you add a comment that does not align with the code, the gofmt tool will fix that. This tool, provided with your Go installation, formats Go code (including comments) to a universal format so that Go code everywhere looks the same and programmers cannot argue over tabs and spaces. As a Gopher (as Go enthusiasts are called), you should format your Go code continually as you write it—and certainly before committing it to version control. You can run gofmt manually (gofmt -w hello.go), but it’s more convenient to configure your text editor or IDE to run it every time your files are saved.

Since this comment is so short, it could instead appear as an inline comment just to the right of the code:


hello.go
```
. . .
	fmt.Println("Hello, World!") // Say hi via the console
. . .

```


Most comments appear on their own line unless they are very brief like this one.


Longer comments span multiple lines. Go supports C-style block comments using /* and */ tags to open and close very long comments, but these are used only in special cases. (More on that later.) Ordinary multi-line comments begin every line with //  rather than using block comment tags.


Here is some code with many comments, each indented properly. The one multi-line comment is highlighted:


color.go
```
package main

import "fmt"

const favColor string = "blue" // Could have chosen any color

func main() {
	var guess string
	// Create an input loop
	for {
		// Ask the user to guess my favorite color
		fmt.Println("Guess my favorite color:")

                // Try to read a line of input from the user.
                // Print out an error and exit, if there is one.
		if _, err := fmt.Scanln(&guess); err != nil {
			fmt.Printf("%s\n", err)
			return
		}

		// Did they guess the correct color?
		if favColor == guess {
			// They guessed it!
			fmt.Printf("%q is my favorite color!\n", favColor)
			return
		}
		// Wrong! Have them guess again.
		fmt.Printf("Sorry, %q is not my favorite color. Guess again.\n", guess)
	}
}

```


Most of these comments are actually clutter. Such a small and simple program should not contain this many comments, and most of these say something that the code itself makes obvious. You can trust other Go programmers to understand the basics of Go syntax, control flow, data types, and so on. You don’t need to write a comment announcing that the code is about to iterate over a slice or multiply two floats.


One of these comments, however, is useful.


## Good Comments Explain Why


The most useful comments in any program are those that explain not what the code does or how it does that, but why it does that. Sometimes there is no why, and even that can be useful to point out, as this inline comment does:


color.go
```
const favColor string = "blue" // Could have chosen any color

```


This comment says something that the code cannot: that the value “blue” was chosen by the programmer arbitrarily. In other words, // Feel free to change this.


Most code, though, does have a why. Here is a function in the net/http package from the Go standard library that contains two really helpful comments:


client.go
```
. . .
// refererForURL returns a referer without any authentication info or
// an empty string if lastReq scheme is https and newReq scheme is http.
func refererForURL(lastReq, newReq *url.URL) string {
	// https://tools.ietf.org/html/rfc7231#section-5.5.2
	//   "Clients SHOULD NOT include a Referer header field in a
	//    (non-secure) HTTP request if the referring page was
	//    transferred with a secure protocol."
	if lastReq.Scheme == "https" && newReq.Scheme == "http" {
		return ""
	}
	referer := lastReq.String()
	if lastReq.User != nil {
		// This is not very efficient, but is the best we can
		// do without:
		// - introducing a new method on URL
		// - creating a race condition
		// - copying the URL struct manually, which would cause
		//   maintenance problems down the line
		auth := lastReq.User.String() + "@"
		referer = strings.Replace(referer, auth, "", 1)
	}
	return referer
}
. . .

```


The first highlighted comment warns maintainers not to change the code below because it is intentionally written to comply with the RFC (official specification) for the HTTP protocol. The second highlighted comment admits that the code below is not ideal, hints at how a maintainer might try to improve it, and warns them about the danger of doing that.


Comments like these are indispensable. They stop maintainers from unwittingly introducing bugs and other problems, yet may also invite them to implement new ideas—with caution.


The comment above the func declaration is helpful too, but in a different way. Let’s explore that kind of comment next.


# Doc Comments


Comments that appear just above top-level (non-indented) declarations like package, func, const, var, and type are called doc comments. They are so named because they represent the official documentation for a package and all of its exported names.



Note: In Go, exported means the same thing public means in some languages: an exported component is one that other packages may use when importing your package. To export any top-level name in your package, all you must do is capitalize it.

## Doc Comments Explain What and How


Unlike the ordinary comments we just saw, doc comments usually do explain what the code does or how it does that. That’s because they are meant not for a package’s maintainers, but for its users, who usually don’t want to read or contribute to the code.


Users will typically read your doc comments in one of three places:


1. In their local terminal, by running go doc on an individual source file or directory.
2. On pkg.go.dev, the official hub for documentation on any public Go package.
3. On a privately-run web server, hosted by your team using the godoc tool. This tool lets your team create its own reference portal for private Go packages.

When developing your Go packages, you should write a doc comment for every exported name. (And occasionally for unexported names.) Here is a one-line doc comment in godo, the Go client library for the DigitalOcean API:


godo.go
```
// Client manages communication with DigitalOcean V2 API.
type Client struct {

```


A simple doc comment like this may seem unnecessary, but remember that it will appear alongside all of your other doc comments to comprehensively document every usable component of the package.


Here is a longer doc comment from the package:


godo.go
```
// Do sends an API request and returns the API response. The API response is JSON decoded and stored in the value
// pointed to by v, or returned as an error if an API error has occurred. If v implements the io.Writer interface,
// the raw response will be written to v, without attempting to decode it.
func (c *Client) Do(ctx context.Context, req *http.Request, v interface{}) (*Response, error) {
. . .
}

```


Doc comments for functions should clearly specify the expected format for parameters (when it is not obvious) and the format of the data the function returns. They may also summarize how the function works.


Compare the doc comment for the Do function with this comment within the function:


godo.go
```
	// Ensure the response body is fully read and closed
	// before we reconnect, so that we reuse the same TCPConnection.
	// Close the previous response's body. But read at least some of
	// the body so if it's small the underlying TCP connection will be
	// re-used. No need to check for errors: if it fails, the Transport
	// won't reuse it anyway.

```


This is like the comments we saw in the net/http package. A maintainer reading the code below this comment might wonder, “Why doesn’t this check for errors?”, and then add error checking. But the comment explains why they don’t need to do that. It is not a high-level what or how, as doc comments are.


The very highest level doc comments are package comments. Every package should contain just one package comment that gives a high-level overview of what the package is and how to use it, including code and/or command examples. The package comment may appear in any source file—and only that file—above the package <name> declaration. Often a package comment appears in its own file called doc.go.


Unlike all the other comments we’ve looked at, package comments often use /* and */ since they can be so lengthy. Here’s the beginning of the package comment for gofmt:


```
/*
Gofmt formats Go programs.
It uses tabs for indentation and blanks for alignment.
Alignment assumes that an editor is using a fixed-width font.

Without an explicit path, it processes the standard input. Given a file,
it operates on that file; given a directory, it operates on all .go files in
that directory, recursively. (Files starting with a period are ignored.)
By default, gofmt prints the reformatted sources to standard output.

Usage:

gofmt [flags] [path ...]

The flags are:

-d
Do not print reformatted sources to standard output.
If a file's formatting is different than gofmt's, print diffs
to standard output.
. . .
*/
package main

```


So what about the format for doc comments? What kind of structure can they (or must they) have?


## Doc Comments Have a Format


According to an old blog post from the Go creators:



Godoc is conceptually related to Python’s Docstring and Java’s Javadoc but its design is simpler. The comments read by godoc are not language constructs (as with Docstring) nor must they have their own machine-readable syntax (as with Javadoc). Godoc comments are just good comments, the sort you would want to read even if godoc didn’t exist.

While doc comments have no required format, they may optionally use a format that is a “simplified subset of Markdown” fully described in the Go documentation. In your doc comments, you will write in paragraphs and lists, show example code or commands in indented blocks, give links to references, and more. When docs comments are well-structured according to this format, they may be rendered into nice web pages.


Here are some comments added to the extended Hello World program greeting.go from How To Write Your First Program in Go:


greeting.go
```
// This is a doc comment for greeting.go.
//  - prompt user for name.
//   - wait for name
//    - print name.
// This is the second paragraph of this doc comment.
// `gofmt` (and `go doc`) will insert a blank line before it.
package main

import (
	"fmt"
	"strings"
)

func main() {
	// This is not a doc comment. Gofmt will NOT format it.
	//  - prompt user for name
	//   - wait for name
	//    - print name
	// This is not a "second paragraph" because this is not a doc comment.
	// It's just more lines to this non-doc comment.
	fmt.Println("Please enter your name.")
	var name string
	fmt.Scanln(&name)
	name = strings.TrimSpace(name)
	fmt.Printf("Hi, %s! I'm Go!", name)
}

```


The comment above package main is a doc comment. It is trying to use the doc comment format’s notions of paragraphs and lists, but it isn’t quite right. The gofmt tool will mold it into the format. Running gofmt greeting.go would print the following:


```
// This is a doc comment for greeting.go.
//   - prompt user for name.
//   - wait for name.
//   - print name.
//
// This is the second paragraph of this doc comment.
// `gofmt` (and `go doc`) will insert a blank line before it.
package main

import (
	"fmt"
	"strings"
)

func main() {
	// This is not a doc comment. `gofmt` will NOT format it.
	//  - prompt user for name
	//   - wait for name
	//    - print name
	// This is not a "second paragraph" because this is not a doc comment.
	// It's just more lines to this non-doc comment.
	fmt.Println("Please enter your name.")
	var name string
	fmt.Scanln(&name)
	name = strings.TrimSpace(name)
	fmt.Printf("Hi, %s! I'm Go!", name)
}

```


Notice that:


1. The items listed in the first paragraph of the doc comment are now aligned.
2. The first and second paragraphs now have a blank line between them.
3. The comment within main() has not been formatted because gofmt recognized that it is not a doc comment. (But as mentioned earlier, gofmt does align all comments to the same indentation as the code.)

Running go doc greeting.go would format and print the doc comment, but not the one within main():


Output
```
This is a doc comment for greeting.go.
  - prompt user for name.
  - wait for name.
  - print name.

This is the second paragraph of this doc comment. `gofmt` (and `go doc`) will
insert a blank line before it.

```


If you use this doc comment format consistently and appropriately, your package’s users will be grateful for your easy-to-read documentation.


Read the official reference page on doc comments to learn everything about how to write them well.


# Quickly Disabling Code


Have you ever written some new code that slowed your application down—or worse, broke everything? This is the other time to use the C-style /* and */ tags. You can quickly disable problematic code by placing one /* before it and one */ after it. Wrap these tags around the problematic code to turn it into a block comment. Then, when you have fixed whatever problem it was causing, you can re-enable the code by removing the two tags.


problematic.go
```
. . .
func main() {
    x := initializeStuff()
    /* This code is causing problems, so we're going to comment it out for now
    someProblematicCode(x)
    */
    fmt.Println("This code is still running!")
}

```


For longer chunks of code, using these tags is a lot more convenient than adding // to the beginning of every line of problematic code. As a convention, use // for ordinary comments and doc comments that will live in your code indefinitely. Use /* and */ tags only temporarily during testing (or for package comments, as mentioned before). Do not leave snippets of commented-out code in your program for long periods of time.


# Conclusion


By writing expressive comments in all of your Go programs, you are:


1. Preventing your collaborators from breaking things.
2. Helping your future self, who sometimes has forgotten why the code was originally written as it is.
3. Giving your package’s users a reference they can read without diving into your code.

