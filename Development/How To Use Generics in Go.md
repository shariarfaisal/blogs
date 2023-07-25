# How To Use Generics in Go

```Development``` ```Go```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


In Go 1.18, the language introduced a new feature called generic types (commonly known by the shorter term, generics) that had been on Go developers’ wish list for some time. In programming, a generic type is a type that can be used in conjunction with multiple other types. Typically in Go, if you want to be able to use two different types for the same variable, you’d need to use either a specific interface, such as io.Reader, or use interface{}, which allows any value to be used. Using interface{} can make it challenging to work with those types, though, because you need to translate between several other potential types to interact with them. Using generic types allows you to interact directly with your types, leading to cleaner and easier-to-read code.


In this tutorial, you will create a program that interacts with a deck of cards. You’ll begin by creating a deck that uses interface{} to interact with the cards, and then you will update it to use generic types. After those updates, you will add a second type of card to your deck using generics, and then you will update your deck to constrain its generic type to only support card types. Finally, you will create a function that uses your cards and supports generic types.


# Prerequisites


To follow this tutorial, you will need:


- 
Go version 1.18 or greater installed. To set this up, follow the How To Install Go tutorial for your operating system.

- 
A solid understanding of the Go language, such as variables, functions, struct types, for loops, and slices. If you’d like to read more about these concepts, our How To Code in Go series has a number of tutorials covering these and more.


# Go Collections Without Generics


One powerful feature of Go is its ability to flexibly represent many types using interfaces. A lot of code written in Go can work well using just the functionality interfaces provide. This is one reason why Go existed for so long without support for generics.


In this tutorial, you will create a Go program that simulates getting a random PlayingCard from a Deck of cards. In this section, you’ll use an interface{} to allow the Deck to interact with any type of card. Later in the tutorial, you’ll update your program to use generics, so that you can better understand the differences between them and recognize when one is a better option than the other.


The type systems in programming languages can generally be classified into two different categories: typing and type checking. A language can use either strong or weak typing, and static or dynamic type checking. Some languages use a mix of these, but Go fits pretty well into the strongly-typed and statically-checked languages. Being strongly-typed means that Go ensures a value in a variable matches the type of the variable, so you can’t store an int value in a string variable, for example. As a statically-checked type system, Go’s compiler will check these type rules when it compiles the program instead of while the program is running.


A benefit of using a strongly-typed, statically-checked language like Go is that the compiler lets you know about any potential mistakes before your program is released, avoiding certain “invalid type” runtime errors. This does add a limitation to Go programs, though, because you must know the types you intend to use before compiling your program. One way to handle this is by using the interface{} type. The reason an interface{} type works for any value is because it doesn’t define any required methods for the interface (signified by the empty {}), so any type matches the interface.


To start creating your program using an interface{} to represent your cards, you’ll need a directory to keep the program’s directory in. In this tutorial, you’ll use a directory named projects.


First, make the projects directory and navigate to it:


```
mkdir projects
cd projects


```


Next, make the directory for your project and navigate to it. In this case, use the directory generics:


```
mkdir generics
cd generics


```


Inside the generics directory, use nano, or your favorite editor, to open the main.go file:


```
nano main.go


```


In the main.go file, begin by adding your package declaration and import the packages you’ll need:


main.go
```
package main

import (
	"fmt"
	"math/rand"
	"os"
	"time"
)

```


The package main declaration tells Go to compile your program as a binary so you can run it directly, and the import statement tells Go which packages you’ll be using in the later code.


Now, define your PlayingCard type and its associated functions and methods:


main.go
```
...

type PlayingCard struct {
	Suit string
	Rank string
}

func NewPlayingCard(suit string, card string) *PlayingCard {
	return &PlayingCard{Suit: suit, Rank: card}
}

func (pc *PlayingCard) String() string {
	return fmt.Sprintf("%s of %s", pc.Rank, pc.Suit)
}

```


In this snippet, you define a struct named PlayingCard with the properties Suit and Rank, to represent the cards from a deck of 52 playing cards. The Suit will be one of Diamonds, Hearts, Clubs, or Spades, and the Rank will be A, 2, 3, and so on through K.


You also defined a NewPlayingCard function to act as the constructor for the PlayingCard struct, and a String method, which will return the rank and suit of the card using fmt.Sprintf.


Next, create your Deck type with the AddCard and RandomCard methods, as well as a NewPlayingCardDeck function to create a *Deck filled with all 52 playing cards:


main.go
```
...

type Deck struct {
	cards []interface{}
}

func NewPlayingCardDeck() *Deck {
	suits := []string{"Diamonds", "Hearts", "Clubs", "Spades"}
	ranks := []string{"A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"}

	deck := &Deck{}
	for _, suit := range suits {
		for _, rank := range ranks {
			deck.AddCard(NewPlayingCard(suit, rank))
		}
	}
	return deck
}

func (d *Deck) AddCard(card interface{}) {
	d.cards = append(d.cards, card)
}

func (d *Deck) RandomCard() interface{} {
	r := rand.New(rand.NewSource(time.Now().UnixNano()))

	cardIdx := r.Intn(len(d.cards))
	return d.cards[cardIdx]
}

```


In the Deck you define above, you create a field called cards to hold a slice of cards. Since you want the deck to be able to hold multiple different types of cards, you can’t just define it as []*PlayingCard, though. You define it as []interface{} so it can hold any type of card you may create in the future. In addition to the []interface{} field on the Deck, you also create an AddCard method that accepts the same interface{} type to append a card to the Deck’s cards field.


You also create a RandomCard method that will return a random card from the Deck’s cards slice. This method uses the math/rand package to generate a random number between 0 and the number of cards in the cards slice. The rand.New line creates a new random number generator using the current time as a source of randomness; otherwise, the random number could be the same every time. The r.Intn(len(d.cards)) line uses the random number generator to generate an int value between 0 and the number provided. Since the Intn method doesn’t include the parameter value in the range of numbers, there’s no need to subtract 1 from the length to account for starting from 0. Finally, RandomCard returns the card value at the index of the random number.



Warning: Be careful which random number generator you use in your programs. The math/rand package is not cryptographically secure and shouldn’t be used for security-sensitive programs. The crypto.rand package, however, does provide a random number generator that can be used for these purposes.

Finally, the NewPlayingCardDeck function returns a *Deck value populated with all the cards in a playing card deck. You use two slices, one with all the available suits and one with all the available ranks, and then loop over each value to create a new *PlayingCard for each combination before adding it to the deck using AddCard. Once the cards for the deck are generated, the value is returned.


Now that you have your Deck and PlayingCard set up, you can create your main function to use them to draw cards:


main.go
```
...

func main() {
	deck := NewPlayingCardDeck()

	fmt.Printf("--- drawing playing card ---\n")
	card := deck.RandomCard()
	fmt.Printf("drew card: %s\n", card)

	playingCard, ok := card.(*PlayingCard)
	if !ok {
		fmt.Printf("card received wasn't a playing card!")
		os.Exit(1)
	}
	fmt.Printf("card suit: %s\n", playingCard.Suit)
	fmt.Printf("card rank: %s\n", playingCard.Rank)
}

```


In the main function, you first create a new deck of playing cards using the NewPlayingCardDeck function and assign it to the deck variable. Then, you use fmt.Printf to print that you’re drawing a card and use deck’s RandomCard method to get a random card from the deck. After, you use fmt.Printf again to print the card you drew from the deck.


Next, since the type of the card variable is interface{}, you need to use a type assertion to get a reference to the card as its original *PlayingCard type. If the type in the card variable is not a *PlayingCard type, which it should be given how your program is written right now, the value of ok will be false and your program will print an error message using fmt.Printf and exit with an error code of 1 using os.Exit. If it is a *PlayingCard type, you then print out the playingCard’s Suit and Rank values using fmt.Printf.


Once you have all your changes saved, you can run your program using go run with main.go, the name of the file to run:


```
go run main.go


```


In the output of your program, you should see a card randomly chosen from the deck, as well as the card suit and rank:


```
Output--- drawing playing card ---
drew card: Q of Diamonds
card suit: Diamonds
card rank: Q

```


Since the card is drawn randomly from the deck, your output will likely be different from the output shown above, but you should see similar output. The first line is printed just before drawing the random card from the deck, and then the second line is printed once the card is drawn. You can see the output of the card is using the value returned by PlayingCard’s String method. Lastly, you can see the two lines of suit and rank output printed after asserting your interface{} card value into a *PlayingCard value.


In this section, you created a Deck that uses interface{} values to store and interact with any value, and a PlayingCard type to act as cards within that Deck. You then used the Deck and PlayingCards to choose a random card from the deck and print out information about that card.


To access specific information about the *PlayingCard value you drew, though, you needed to do some extra work to convert the interface{} type into a *PlayingCard type with access to the Suit and Rank fields. Using the Deck this way will work, but can also result in errors if a value other than *PlayingCard is added to the Deck. By updating your Deck to use generics, you can benefit from Go’s strong types and static type checking while still having the flexibility accepting interface{} values provides.


# Go Collections With Generics


In the previous section, you created a collection using a slice of interface{} types. But to use those values, you needed to do some extra work to translate the values from an interface{} into the actual type of those values. However, using generics, you can create one or more type parameters, which act almost like function parameters, but they can hold types as values instead of data. In this way, generics provide a way to substitute a different type for the type parameters every time the generic type is used. This is where generic types get their name. Since the generic type can be used with multiple types, and not just a specific type like io.Reader or interface{}, it’s generic enough to fit several use cases.


In this section, you will update your Deck type to be a generic type that can use any specific type of card when you create an instance of the Deck instead of using an interface{}.


To make your first update, open your main.go file and remove the os package import:


main.go
```
package main

import (
	"fmt"
	"math/rand"
	// "os" package import is removed
	"time"
)

```


As you’ll see in the later updates, you won’t need to use the os.Exit function any more so it’s safe to remove this import.


Next, update your Deck struct to be a generic type:


main.go
```
...

type Deck[C any] struct {
	cards []C
}

```


This update introduces the new syntax used by generics to create placeholder types, or type parameters, within your struct declaration. You can almost think of these type parameters as similar to the parameters you’d include in a function. When calling a function, you provide values for each function parameter. Likewise, when creating a generic type value, you provide types for the type parameters.


You’ll see after the name of your struct, Deck, that you’ve added a statement inside square brackets ([]). These square brackets allow you to define one or more of these type parameters for your struct.


In the case of your Deck type, you only need one type parameter, named C, to represent the type of the cards in your deck. By declaring C any inside your type parameters, your code says, “create a generic type parameter named C that I can use in my struct, and allow it to be any type”. Behind the scenes, the any type is actually an alias to the interface{} type. This makes generics easier to read, and you don’t need to use C interface{}. Your deck only needs one generic type to represent the cards, but if you require additional generic types, you could add them using a comma-separated syntax, such as C any, F any. The name you use for your type parameters can be anything you’d like if it isn’t reserved, but they are typically short and capitalized.


Lastly, in your update to the Deck declaration, you updated the type of the cards slice in the struct to use the C type parameter. When using generics, you can use your type parameters anywhere you typically put a specific type. In this case, you want your C parameter to represent each card in the slice, so you put the [] slice type declaration followed by the C parameter declaration.


Next, update your Deck type’s AddCard method to use the generic type you defined. For now, you’ll skip over updating the NewPlayingCardDeck function, but you will be coming back to it shortly:


main.go
```
...

func (d *Deck[C]) AddCard(card C) {
	d.cards = append(d.cards, card)
}

```


In the update to Deck’s AddCard method, you first added the [C] generic type parameter to the method’s receiver. This lets Go know the name of the type parameter you’ll be using elsewhere in the method declaration and follows a similar square bracket syntax as the struct declaration. In this case, though, you don’t need to provide the any constraint because it was already provided in the Deck’s declaration. Then, you updated the card function parameter to use the C placeholder type instead of the original interface{} type. This allows the method to use the specific type C will eventually become.


After updating the AddCard method, update the RandomCard method to use the C generic type as well:


main.go
```
...

func (d *Deck[C]) RandomCard() C {
	r := rand.New(rand.NewSource(time.Now().UnixNano()))

	cardIdx := r.Intn(len(d.cards))
	return d.cards[cardIdx]
}

```


This time, instead of using the C generic type as a function parameter, you’ve updated the method to return the value C instead of interface{}. Aside from updating the receiver to include [C], this is the only update you need to make to the function. Since the cards field on Deck was already updated in the struct declaration, when this method returns a value from cards, it’s returning a value of type C.


Now that your Deck type is updated to use generics, go back to your NewPlayingCardDeck function and update it to use the generic Deck type for *PlayingCard types:


main.go
```
...

func NewPlayingCardDeck() *Deck[*PlayingCard] {
	suits := []string{"Diamonds", "Hearts", "Clubs", "Spades"}
	ranks := []string{"A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"}

	deck := &Deck[*PlayingCard]{}
	for _, suit := range suits {
		for _, rank := range ranks {
			deck.AddCard(NewPlayingCard(suit, rank))
		}
	}
	return deck
}

...

```


Most of the code in the NewPlayingCardDeck stays the same, but now that you’re using a generic version of Deck, you need to specify the type you’d like to use for C when using the Deck. You do this by referencing your Deck type as you normally would, whether it’s Deck or a reference like *Deck, and then providing the type that should replace C by using the same square brackets you used when initially declaring the type parameters.


For the NewPlayingCardDeck return type, you still use *Deck as you did before, but this time, you also include the square brackets and *PlayingCard. By providing [*PlayingCard] for the type parameter, you’re saying that you want the *PlayingCard type in your Deck declaration and methods to replace the value of C. This means that the type of the cards field on Deck essentially changes from []C to []*PlayingCard.


Similarly, when creating a new instance of Deck, you also need to provide the type replacing C. Where you might usually use &Deck{} to make a new reference to Deck, you instead include the type inside square brackets to end up with &Deck[*PlayingCard]{}.


Now that your types have been updated to use generics, you can update your main function to take advantage of them:


main.go
```
...

func main() {
	deck := NewPlayingCardDeck()

	fmt.Printf("--- drawing playing card ---\n")
	playingCard := deck.RandomCard()
	fmt.Printf("drew card: %s\n", playingCard)
	// Code removed
	fmt.Printf("card suit: %s\n", playingCard.Suit)
	fmt.Printf("card rank: %s\n", playingCard.Rank)
}

```


This time your update is to remove code because you no longer need to assert an interface{} value into a *PlayingCard value. When you updated Deck’s RandomCard method to return C and updated NewPlayingCardDeck to return *Deck[*PlayingCard], it changed RandomCard to return a *PlayingCard value instead of interface{}. When RandomCard returns *PlayingCard, it means the type of playingCard is also *PlayingCard instead of interface{} and you can access the Suit or Rank fields right away.


To see your program running after you’ve saved your changes to main.go, use the go run command again:


```
go run main.go


```


You should see output similar to the following output, but the card drawn will likely be different:


```
Output--- drawing playing card ---
drew card: 8 of Hearts
card suit: Hearts
card rank: 8

```


Even though the output is the same as the previous version of your program using interface{}, the code itself is a little cleaner and avoids potential errors. You no longer need to do the assertion to the *PlayingCard type, avoiding additional error handling. In addition, by saying the instance of your Deck can only contain *PlayingCard, there’s no possibility for a value other than *PlayingCard being added to the cards slice.


In this section, you updated your Deck struct to be a generic type, providing more control over the types of cards each instance of your deck can contain. You also updated the AddCard and RandomCard methods to either accept a generic argument or return a generic value. Then, you updated the NewPlayingCardDeck to return a *Deck containing *PlayingCard cards. Finally, you removed the error handling in your main function because you didn’t need it anymore.


Now that your Deck is updated to be generic, it’s possible to use it to hold any type of cards you’d like. In the next section, you take advantage of this flexibility by adding a new type of card to your program.


# Using Generics With Multiple Types


Once you have a generic type created, like your Deck, you can use it with any other type. When you created an instance of your generic Deck and wanted it to be used with *PlayingCard types, the only thing you needed to do was specify that type when creating the value. To support a different type, you swap the *PlayingCard type with the new type you’d like to use.


In this section, you will create a new TradingCard struct type to represent a different type of card. Then, you will update your program to create a Deck of *TradingCards.


To create your TradingCard type, open your main.go file again and add the definition:


main.go
```
...

import (
	...
)

type TradingCard struct {
	CollectableName string
}

func NewTradingCard(collectableName string) *TradingCard {
	return &TradingCard{CollectableName: collectableName}
}

func (tc *TradingCard) String() string {
	return tc.CollectableName
}

```


This TradingCard is similar to your PlayingCard, but instead of having the Suit and Rank fields, it has a CollectableName field to keep track of the trading card’s name. It also includes the NewTradingCard constructor function and String method, similar to PlayingCard.


Now, create the NewTradingCardDeck constructor for a Deck filled with *TradingCards:


main.go
```
...

func NewPlayingCardDeck() *Deck[*PlayingCard] {
	...
}

func NewTradingCardDeck() *Deck[*TradingCard] {
	collectables := []string{"Sammy", "Droplets", "Spaces", "App Platform"}

	deck := &Deck[*TradingCard]{}
	for _, collectable := range collectables {
		deck.AddCard(NewTradingCard(collectable))
	}
	return deck
}

```


When you create or return the *Deck this time, you’ve replaced *PlayingCard with *TradingCard, but that’s the only change you need to make to the deck. You have a slice of special DigitalOcean collectables, which you then loop over to add each *TradingCard to the deck. The deck’s AddCard method still works the same way, but this time it accepts the *TradingCard value from NewTradingCard. If you tried passing it a value from NewPlayingCard, the compiler would give you an error because it would be expecting a *TradingCard, but you’re providing a *PlayingCard.


Lastly, update your main function to create a new Deck of *TradingCards, draw a random card with RandomCard, and print out the card’s information:


main.go
```
...

func main() {
	playingDeck := NewPlayingCardDeck()
	tradingDeck := NewTradingCardDeck()

	fmt.Printf("--- drawing playing card ---\n")
	playingCard := playingDeck.RandomCard()
	...
	fmt.Printf("card rank: %s\n", playingCard.Rank)

	fmt.Printf("--- drawing trading card ---\n")
	tradingCard := tradingDeck.RandomCard()
	fmt.Printf("drew card: %s\n", tradingCard)
	fmt.Printf("card collectable name: %s\n", tradingCard.CollectableName)
}

```


In this last update, you create a new trading card deck using NewTradingCardDeck and store it in tradingDeck. Then, since you’re still using the same Deck type as before, you can use RandomCard to get a random trading card from the deck and print out the card. You can also reference and print out the CollectableName field directly on the tradingCard because the generic Deck you’re using has defined C as *TradingCard.


This update also shows the value of using generics. To support an entirely new card type, you didn’t have to change the Deck at all. The type parameters of Deck allowed you to provide the type of card to use when you created an instance of the Deck, and from that point forward, any interactions with that Deck value use the *TradingCard type instead of the *PlayingCard type.


To see your updated code in action, save your changes and run your program again with go run:


```
go run main.go


```


Then, review your output:


```
Output--- drawing playing card ---
drew card: Q of Diamonds
card suit: Diamonds
card rank: Q
--- drawing trading card ---
drew card: App Platform
card collectable name: App Platform

```


Once the program finishes running, you should see output similar to the output above, just with different cards. You’ll see two cards being drawn: the original playing card and the new trading card you added.


In this section, you added a new TradingCard type to represent a different type of card than your original PlayingCard. Once you added the TradingCard type, you created the NewTradingCardDeck constructor to create and populate the deck with trading cards. Finally, you updated your main method to use the new trading card deck and print out information about the random card being drawn.


Aside from creating a new function, NewTradingCardDeck, to populate a Deck with different cards, you didn’t need to make any other updates to your Deck to support an entirely new card type. This is the power of generic types. You can write your code once and re-use it for multiple other types of similar data. One problem with your current Deck, though, is that it could be used for any type, due to the C any declaration you have. This may be something you want so that you can create a deck of int values using &Deck[int]{}. But if you want your Deck to contain only cards, you’ll need a way to restrict the data types allowed for C.


# Restricting Generic Types


Often, you don’t want or need any restrictions on the types being used by generics because you don’t necessarily care about the specific data. Other times, though, you need to be able to restrict the types used by a generic. For example, if you’re creating a generic Sorter type, you may want to restrict its generic types to those with a Compare method, so the Sorter can compare the items it’s holding. If you don’t include that restriction, the values may not even have a Compare method, and your Sorter won’t know how to compare them.


In this section, you’ll create a new Card interface for your cards to implement, and then update your Deck to only allow Card types to be added.


To begin the updates, open the main.go file and add your Card interface:


main.go
```
...

import (
...
)

type Card interface {
	fmt.Stringer

	Name() string
}

```


Your Card interface is defined the same as any other Go interface you may have used in the past; there are no special requirements to use it with generics. In this Card interface, you’re saying that for something to be considered a Card, it must implement the fmt.Stringer type (it must have the String method your cards already have), and it must also have a Name method that returns a string value.


Next, update your TradingCard and PlayingCard types to add a new Name method, in addition to the existing String methods, so they implement the Card interface:


main.go
```
...

type TradingCard struct {
	...
}

...

func (tc *TradingCard) Name() string {
	return tc.String()
}

...

type PlayingCard struct {
	...
}

...

func (pc *PlayingCard) Name() string {
	return pc.String()
}

```


The TradingCard and PlayingCard already have String methods that implement the fmt.Stringer interface. So to implement the Card interface, you only need to add the new Name method. Also, since fmt.Stringer is already implemented to return the names of the cards, you can just return the result of the String method for Name.


Now, update your Deck so it only allows Card types to be used for C:


main.go
```
...

type Deck[C Card] struct {
	cards []C
}

```


Before this update, you had C any for the type restriction (known as the type constraint), which isn’t much of a restriction. Since any means the same as interface{}, it allowed any type in Go to be used for the C type parameter. Now that you’ve replaced any with your new Card interface, the Go compiler will ensure that any types used for C implement Card when you compile your program.


Since you’ve added this restriction, you can now use any methods provided by Card inside your Deck type’s methods. If you wanted RandomCard to also print out the name of the card being drawn, it would be able to access the Name method because it’s part of the Card interface. You’ll see this in action in the next section.


These few updates are the only ones you need to make to restrict your Deck type to only using Card values. After saving your changes, run your updated program using go run:


```
go run main.go


```


Then, once your program is finished running, review the output:


```
Output--- drawing playing card ---
drew card: 5 of Clubs
card suit: Clubs
card rank: 5
--- drawing trading card ---
drew card: Droplets
card collectable name: Droplets

```


You’ll see that, aside from choosing different cards, the output hasn’t changed. Since your updates only restricted the values to types you were already using, your program’s functionality didn’t change.


In this section, you added a new Card interface and updated both TradingCard and PlayingCard to implement the interface. You also updated Deck’s type constraint to limit its type parameters to only types that implement the Card interface.


So far you’ve only created a generic struct type, though. In addition to creating generic types, Go also allows you to create generic functions.


# Creating Generic Functions


Generic functions in Go have a very similar syntax to other generic types in Go. When you consider that other generic types have type parameters, making generic functions is a matter of adding a second set of parameters to those functions.


In this section, you will create a new printCard generic function, and use that function to print out the name of the card provided.


To implement your new function, open your main.go file and make the following updates:


main.go
```
...

func printCard[C any](card C) {
	fmt.Println("card name:", card.Name())
}

func main() {
	...
	fmt.Printf("card collectable name: %s\n", tradingCard.CollectableName)

	fmt.Printf("--- printing cards ---\n")
	printCard[*PlayingCard](playingCard)
	printCard(tradingCard)
}

```


In the printCard function, you’ll see the familiar square bracket syntax for the generic type parameters, followed by the regular function parameters in the parentheses. Then, in the main function, you use the printCard function to print out both a *PlayingCard and a *TradingCard.


You may notice that one of the calls to printCard includes the [*PlayingCard] type parameter, while the second one doesn’t include the same [*TradingCard] type parameter. The Go compiler is able to figure out the intended type parameter by the value you’re passing in to the function parameters, so in cases like this, the type parameters are optional. If you wanted to, you could also remove the [*PlayingCard] type parameter.


Now, save your changes and run your program again using go run::


```
go run main.go


```


This time, though, you’ll see a compiler error:


```
Output# command-line-arguments
./main.go:87:33: card.Name undefined (type C has no field or method Name)

```


In the type parameter for the printCard function, you have any as the type constraint for C. When Go compiles the program, it expects to see only methods defined by the any interface, where there are none. This is the benefit of using specific type constraints on type parameters. In order to access the Name method on your card types, you need to tell Go the only types being used for the C parameter are Cards.


Update your main.go file one last time to replace the any type constraint with the Card type constraint:


main.go
```
...

func printCard[C Card](card C) {
	fmt.Println("card name:", card.Name())
}

```


Then, save your changes and run your program using go run:


```
go run main.go


```


Your program should now run successfully:


```
Output--- drawing playing card ---
drew card: 6 of Hearts
card suit: Hearts
card rank: 6
--- drawing trading card ---
drew card: App Platform
card collectable name: App Platform
--- printing cards ---
card name: 6 of Hearts
card name: App Platform

```


In this output, you’ll see both cards being drawn and printed as you’re familiar with, but now the printCard function also prints out the cards and uses their Name method to get the name to print.


In this section, you created a new generic printCard function that can take any Card value and print the name. You also saw how using a type constraint of any instead of Card or another specific value can affect the methods available.


# Conclusion


In this tutorial, you created a new program with a Deck that could return a random card from the deck as an interface{} value, as well as a PlayingCard type to represent a playing card in the deck. Then, you updated your Deck type to support generic type parameters and were able to remove some error checking because the generic type ensured that type of error couldn’t occur anymore. After that, you created a new TradingCard type to represent a different type of card your Deck could support, as well as creating decks of each type of card and returning a random card from each deck. Next, you added a type constraint to your Deck to ensure only types that implemented the Card interface could be added to the deck. Finally, you created a generic printCard function that could print the name of any Card value using the Name method.


Using generics in your code can dramatically clean up the amount of code required to support multiple types for the same code, but it’s also possible to overuse them. There are performance and code legibility trade-offs in using generics instead of an interface as a value, so if you can use an interface instead of generics it’s best to prefer the interface. However, generics are still very powerful tools that can make a developer’s life easier when used in cases where they excel.


If you’d like to learn more about how to use generics in Go, the Tutorial: Getting started with generics tutorial on the Go website goes into greater detail on how to use them, as well as additional ways to use type constraints beyond only interfaces.


This tutorial is also part of the DigitalOcean How to Code in Go series. The series covers a number of Go topics, from installing Go for the first time to how to use the language itself.


