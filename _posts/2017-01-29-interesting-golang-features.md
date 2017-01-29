---
layout: post
date: 2017-01-29
title: Interesting Golang Features 
comments: true
---

I've spent the past few weeks learning the 
[Go programming language](https://golang.org/), and working on a backend service 
with a REST API written in Go. In this post, I'd like to discuss my experience 
learning Go, from the perspective of someone who has previously programmed 
primarily in Java. 

Both Go and Java are interesting and useful languages, and my intention
here is not to advocate the use of one language over the other. Rather, I want 
to outline a number of Go's features that I found interesting or noteworthy when 
I first learned about them:

### Variable Declarations
To declare a variable in most languages - like Java - you must first specify the
variable's type, and then its name. For example, to declare an integer variable
named `foobar` in Java, we'd do this:

~~~java
int foobar;
~~~

But when declaring a variable in Go, you first specify the name, and then the 
type:

~~~go
var foobar int
~~~

This is very awkward at first. Variable declaration is one of the most common
things one does when programming, and declaring the type before the name was 
engrained into my mind by years of Java programming. But the Go approach becomes 
natural after awhile.

Go is statically typed, but it can perform type inference on local variable
declarations. For example, I could create a string var named `qux` like so:

~~~go
qux := "baz"
~~~

This is quite useful, and has no equivalent in Java. Note that this syntax is only 
valid for local variables.

### Structs, Not Classes
No special constructor construct; Struct Literals
The closest thing to a class in Go is a struct. I can define a simple struct
like so:

~~~go
type YogiBear struct {
  Intelligence string
}
~~~

I can then create an instance of `YogiBear` using a "struct literal":

~~~go
yogi := YogiBear {
  Intelligence: "smarter than average bear",
}
~~~

### No Constructors
It sure would be convenient if I could set the default value of a `YogiBear`
instance to `"smarter than average"`, instead of hoping that the client knows
this is the right value for the `Intelligence` field. But, this isn't possible 
in Go.

There is no constructor construct in the Go language. Sure, it is possible to 
create a factory function that will initialize and return a struct, but you 
can't force a client to use it. This approach assumes that the client, will know
that you need to use the factory function (or know when it is safe not to). 

This can be a bit annoying at first, especially coming from Java where much
emphasis is put on creating proper constructors and ensuring objects are
initialized in a valid state. But it is futile to try to use a language with a
preconceived notion of how it should work. You cannot force a language to work 
as you expect it to, and Go is no exception. It is certainly possible to write 
good code without using constructors, it just takes time and practice.

In this case, I could use a factory function. Or, an alternative approach would 
be to define a `const` containing the default value of a `YogiBear`'s
intelligence.

### Implicit Interfaces
Go supports interfaces, much like Java. The difference is in how you implement
(aka satisfy) them. For example, suppose I have an interface called `Shape`, 
with a method to obtain the perimeter. In Java, I would need to create a 
concrete type and _explicitly_ declare that it implemented the `Shape` interface. 
This might look something like:

~~~java
class Square implements Shape {
    /* ... */
    public double perimeter() {
      return this.side * 4;
    }
}
~~~

In Go, types satisfy interfaces _implicitly_. This means that all a type needs
to do to satisfy an interface is have functions/methods with the required 
signatures. A concrete type that satisfies the `Shape` interface might look 
like this:

~~~go
type Square struct { /* ... */ }

func (s Square) Perimeter() float64 {
    return s.side * 4;
}
~~~

Notice that we don't have to write `Square implements Shape` as in Java. We just
provide the appropriate method, `perimeter` - and that's it! One common
objection to this design is that is is technically possible to accidentally 
define a type that satisfies an interface without you having intended it to do so.
This isn't really a problem in practice, though, because even if it does happen,
it is unlikely that you would somehow accidentally _use_ your type as an 
implementor of said interface without wanting to. 


### Access Control
In Java, there are four access modifiers: `public`, `protected`, `private`,
and _package-private_. These are denoted with keywords placed at the start of
a new member definition.

~~~java
public int foo;  // publicly accessible
private int bar; // privately accessible
~~~

In addition to requiring an explicit keyword, all these modifiers can be 
somewhat confusing (see [this](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html)).
And in my experience, only `public` and `private` are actually used in practice.

Go takes a different, and characteristically simpler, approach. Go supports two
"access modifiers": exported and unexported. They work at the package level,
meaning that functions, variables, and other types can be accessible only within
their packages (unexported), or outside of them as well (exported).

Keywords are not required to denote whether a variable or type is to be exported
or not. Instead, Go looks at the capitalization of the first letter of the
variable or type's name. Lowercase means unexported, uppercase means exported:

~~~go
Foo int; // exported - accessible outside package
bar int; // unexported - only accessible within package
~~~

This approach is simpler (and more succinct!).

### No Inheritance
Most "Object Oriented" programming languages support two different mechanisms
for defining relationships between new and existing types: inheritance and 
composition, "is a" and "has a" relationships.

Inheritance allows new types to inherit properties from existing types, and to 
serve a role in their place. It allows one to design "object hierarchies", in an
attempt to mirror real-world structures and relationships. The classic example 
involves vehicles: A sedan _is a_ vehicle, and a truck _is a_ vehicle. But a
sedan is not a truck, and a vehicle is not necessarily a sedan (or truck).

Composition allows new types to "have", or to "use" existing types. But, unlike
inheritance, it does not allow a new type to serve in place of an existing type
if it "has"/"uses" the existing type. The classic example here involves vehicles
as well: A sedan _has a_ steering wheel, and a truck _has a_ steering wheel. But
sedans and trucks are not steering wheels.

Go is officially considered to be a Object Oriented language, but it does not
support inheritance, just composition. At first glance, this can seem extremely
limiting, but Go's interface typing in many ways alleviates the need for
inheritance. Go also has a very interesting feature called "Embedded Types",
which is a sort of improved composition mechanism:

### Embedded Types
Go supports type embedding, which allows you to include an anonymous field in a 
struct, like so:

~~~go
type SteeringWheel struct {}
type Sedan struct {
  SteeringWheel // Notice this field has no name - it is anonymous
}
~~~

This is very useful, because as a result, it allows `Sedan` to access all of
`SteeringWheel`'s methods. For example,

~~~go
func (s *SteeringWheel) Turn() {
  /* ... */
}

sedan := Sedan {
  SteeringWheel{},
}

// This is perfectly legal
sedan.Turn()
~~~

Note, however, that this is not inheritance in disguise, because it does not 
allow `Sedan` to be used in place of `SteeringWheel`:

~~~go
func OnlyAcceptsSteeringWheels(wheel SteeringWheel) {}

// Not legal
OnlyAcceptsSteeringWheels(sedan)

// But we CAN do this
OnlyAcceptsSteeringWheels(sedan.SteeringWheel) 
~~~

### Typed Primitives
In Java, primitive types are just that: basic types that can't be extended in
any way. Java does not allow me to create a method for `int`s. There is a sort
of workaround for this in wrapper classes. Java has wrappers for all primitive
types (e.g. `Integer` & `Boolean`), and it is trivial to create your own.

But in Go, we can do something rather interesting - we can create types primitives:

~~~go
type Fahrenheit int
type Celsius int
~~~

I can now create typed `int` values! This is useful if I want to be very clear
that a function is only supposed to accept values representing temperature on
the Fahrenheit scale, say. Of course, it is trivial to cast an untyped `int` to
either of these new types, `Fahrenheit` or `Celsius`, but that requires an explicit
action to be taken on the part of the programmer, and won't happen by mistake.
As an example:

~~~go
var boiling Fahrenheit = 212 // Could also perform a cast but this is much nicer

func FtoC(f Fahrenheit) Celsius { // Emphasis return value is in Celsius scale
  return (f - 32)*5 / 9 // Can use normal arithmetic operators on typed primitives
}
~~~

It is even possible to define _methods_ on these types:

~~~go
func (f Fahrenheit) ToCelsius() Celsius {
  return (f - 32)*5 / 9
}
~~~

The idea of created typed primitives was very foreign and awkward to me when I
first encountered them. But it turns out to be very natural and useful. And, of
course, typing isn't limited to primitives. It is trivial to create new typed
`error`s:

~~~go
type PeasMixedWithCarrotsErr error
~~~

### Functions & Methods
Go supports functions _and_ methods. Functions work just as you would expect.
They take some parameters and produce a result. Methods, however, take parameters
_and_ a particular receiver type. Methods are really just functions that require
a **receiver**. For example,

~~~go
type Dog struct {}

// Methods have a receiver, and can also have a pointer
func (d *Dog) Bark(n int) {
  for i := 0; i < n; i++ {
    fmt.Println("Bark");
  }
}

func main() {
  // Make a dog bark 5 times
  var dog Dog
  dog.Bark(5)
}
~~~

Functions are first class types in Go. This means that we can pass them around
just as you would expect with any other type. So I can do this:

~~~go
func Hi() {
  fmt.Println("Hi!")
}

// Prints "Hi"
func main() {
  h := Hi
  h()
}
~~~

What we have really just done is separated the _selection_ and the _call_ of a
function (`Hi`) into two distinct steps. Because this is possible, and because
methods are really just a special type of function, we can do some neat things
to shapeshift a method such that we can use it like a typical function. Here is
an example of a **method value**:

~~~go
// Uses previous code from dog example
// Barks 5 times
func main() {
  dog := Dog{}
  b := dog.Bark // "Select" the Bark method with dog as its receiver
  b(5) // Don't have to specify receiver again, just pass in the param
}
~~~

This is particularly useful if we need to pass a method as a `func` param:

~~~go
// "Does" the func `n` times
func Do(f func(), n int) {
  for i := 0; i < n; i++ {
    f()
  }
}

func (d *Dog) Jump() {
  fmt.Println("Jump")
}

func main() {
  dog := Dog{}
  // Messy way
  Do(func() { dog.Jump() }, 5)

  // Clean way
  Do(dog.Jump, 5)
}
~~~

Go also supports **method expression** syntax. This allows us to specify a
method's receiver as the first value in the normal list of parameters, rather 
than with dot notation. This can be useful if you do not know which receiver you
want at compile time.

~~~go
// Method call with "method expression" syntax
func main() {
  dog := Dog{}
  b := (*Dog).Bark // method expression 
  b(&dog, 5)
}
~~~

### Multiple Return Values
Functions can return multiple values in Go (This is how errors are returned - see
next section). For example, I can compute the sum and the product of two values
in a function, and return both results (note that this isn't necessarily a good
use case, but it does the job of demonstrating how it works):

~~~go
func SumAndProduct(x, y int) (int, int) {
  return (x + y), (x * y)
}
// Outputs "Sum: 5, Product: 6"
func main() {
  sum, product := SumAndProduct(2, 3)
  fmt.Printf("Sum: %d, Product:%d", sum, product)
}
~~~

### Defer statements
There is a rather unique construct in go called the "defer statement". It allows
you to specify logic that you wish to have executed _after_ the function has 
returned. This is especially useful when working with multiple goroutines. Here
is an example:

~~~go
func HelloWorld() {
  defer fmt.Println("World")
  fmt.Print("Hello ")
}
~~~

Invoking `HelloWorld()` will cause the test `Hello World` (**not** `WorldHello `!)
to be displayed on stdout. This works because of the `defer` statement, which
caused the `fmt.Println("World")` to be executed only after the function has
returned, which occurs _after_ `fmt.Println("Hello ")` has been executed.

### Methods _for Functions_
Methods are just functions with a receiver parameter. Parameters are just 
instances of types. Functions are types. This means that it is possible to create
a method _for a function_. This was very strange to me when I first learned of it.
It looks something like this:

~~~go
type FuncType func(param string)

var FuncTypeVar FuncType = func(param string) {
	fmt.Println(param)
}

func (ft FuncType) executeMe() {
	ft("Executing... 1... 2... 3...")
}

func main() {
	FuncTypeVar.executeMe()
}
~~~

Output:

~~~
Executing... 1... 2... 3...
~~~

I haven't had any occasion to use this in practice, but it certainly is interesting.

### No Exceptions
Many languages support an exception mechanism whereby a function can indicate
that something has gone wrong and throw an exception up the call stack. You are 
usually forced to write code to deal with the exception, or else have your
function's execution be terminated if one is thrown to (at?) it.

Go doesn't support exceptions. Well, that is not _entirely_ true. You can
`panic`, but this mechanism is reserved for special cases, and not routine
errors that are more or less expected to occur during a program's execution (the
typical example of an "expected" error is an I/O failure). It is also not 
possible to `catch` a `panic` as you typically can do with exceptions (caveat: you 
can kind of do this using `recover` within a `defer`ed function, but it still 
isn't quite the same).

Instead of exceptions, Go has `error`s. These are just types that satisfy the
`error` interface - nothing special. They are returned from functions just as
any other value would be, but they ought to be used to indicate, well, `errors`
that occured during a function's execution. You can capture the return value in
a variable, traditionally called `err`, and check to see if it is `nil`. If not,
then you should do something in response. As an example, consider opening a file:

~~~go
file, err := os.Open("/super/cool/file")
if err != nil {
  log.Println("Failed to open file.")
}
~~~

Notice that it would have been entirely possible to have simply ignored the
error value returned by `os.Open` and not written the `if` statement at all. Go 
does not force you to check for errors. You have to make sure to capture all 
`error` values returned by functions that you call, and be responsible enough to 
handle them properly.

Programming without exceptions can certainly feel a bit quirky at first, as is
the case with many of Go's defining features.
The decision of Go's designers to not support exceptions [wasn't made lightly](https://golang.org/doc/faq#exceptions).
It is certainly one of the main points of controversy around the Go language.
My own experience is that it can be a bit more pleasant, and often makes code
easier to read, as it doesn't split up your code like `try-catch` blocks do.
Though, it can be cumbersome when you're working on a block of code that can
generate a lot of errors - such as IO operations - because you have to address
each error individually, rather than dealing with them as a whole, as you could
with a `try-catch` block.

### Arrays/Slices
Go's arrays are fixed length sequences of a type. They are unlike arrays in most
other languages because their length must be constant. Therefore, this is invalid:

~~~go
var numElements int = 21
// Produces error: non-constant array bound numElements 
var arr [numElements]string
~~~

But both of these array declarations are fine:

~~~go
const numElements = 21
var arr [numElements]string // Doesn't produce error
var arrTwo [21]string       // Also doesn't produce error
~~~

Because their size must be known at compile time, Go arrays are used very
infrequently. Far more common, however, are _slices_. Slices are basically
pointers to arrays. This means that if the underlying array of a pointer is filled,
then a new array can be allocated, the elements can be copied over, and the pointer
(aka _slice_) can be pointed at the new array. So they are essentially resizable
arrays; similar to an `ArrayList` in Java. Slices are created with the built-in
`make` function:

~~~go
var initialNumElements int = 21
// Create int slice with 21 elements to start with, but can be grown later
imASlice := make([]int, initialNumElements)
~~~

### Succinct Concurrency
Great support for concurrent programming was one of Go's primary design goals.
Threads, or `goroutines` as they are called in Go lingo, are built right into
the language itself, along with supporting mechanisms. They are not merely new
classes created as afterthoughts as in Java or C/C++. 

Creating a new `goroutine` is as simple as prefixing a function call with the
`go` keyword:

~~~go
go doAThing() // Runs func doAThing in new goroutine/thread
~~~

The brevity of this syntax stands in stark contrast to Java, where this is about
as succinct as it gets:

~~~java
new Thread(() -> doAThing()).start(); // Runs method doAThing in new thread
~~~

Go's concurrency design is based on the CSC (Communicating Sequential Processes) 
model. There is a saying in the Go community that aptly describes how Go's 
designers intended Go's excellent concurrency features to be used:

> Do not communicate by sharing memory; instead, share memory by communicating. 

Go has channels to facilitate this communication between goroutines. Channels are
essentially blocking queues for specific types. The following program illustrates
the use of channels, and prints the integers 0-21:

~~~go
func main() {
	// Make channel & waitgroup
	pipeline := make(chan int)
	var wg sync.WaitGroup

	// Launch producer & consumer in new goroutines
	wg.Add(2) // Set wg counter to 2
	go producer(pipeline, &wg)
	go consumer(pipeline, &wg)

	// Wait for producer & consumer goroutines to finish before exiting
	wg.Wait() // Blocks until wg counter is 0
}

// Send integers 0-21 to specified channel
func producer(out chan<- int, wg *sync.WaitGroup) {
	defer wg.Done() // Decrement wg counter when finished
	for i := 0; i <= 21; i++ {
		out <- i // Send i to channel for reception by consumer goroutine
	}
	close(out)
}

// Get integers from specified channel & print them
func consumer(in <-chan int, wg *sync.WaitGroup) {
	defer wg.Done() // Decrement wg counter when finished
	for i := range in { // Keep receiving next val from channel until it is closed
		fmt.Println(i)
	}
}
~~~

A roughly equivalent version of this program is shown below in Java:

~~~java
public class Main {

    public static void main(String[] args) {
        // Setup queue and threads
        BlockingQueue<Integer> bq = new LinkedBlockingDeque<>();
        Thread prod = new Thread(() -> producer(bq));
        Thread cons = new Thread(() -> consumer(bq));
        
        // Launch producer and consumer in new goroutines
        prod.start();
        cons.start();        
        
        // Wait for producer and consumer to finish before exiting
        try {
            prod.join();
            cons.join();
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
    }

    // Send integers 0-21 to specified queue
    public static void producer(BlockingQueue<Integer> out) {
        try {
            for (int i = 0; i <= 21; i++) {
                out.put(i);
            }
            out.put(-1); // Send -1 to indicate we are finished
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
    }

    // Get integers from specified queue & print them
    public static void consumer(BlockingQueue<Integer> in) {
        try {
            int i;
            while ((i = in.take()) != -1) { // Stop once receive -1
                System.out.println(i);
            }
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
    }
}
~~~

### JSON/Serialization Support
It is extremely easy to serialize an object in Go. JSON format is the most common
encoding method used in Go, in my experience, (though other formats, such as XML
are also supports) so I'll use that in my example (note that I'm neglecting to 
handle errors here):

~~~go
type YogiBear struct {
	Intelligence string
}

func main() {
	// Create a YogiBear
	yogi := YogiBear{Intelligence: "Smarter than average"}
	fmt.Printf("This is yogi: %v\n", yogi)

	// Serialize him to the file system
	b, _ := json.Marshal(yogi)
	fmt.Printf("This is yogi's JSON encoding: %s\n", string(b))
	ioutil.WriteFile("yogi.json", b, os.ModePerm)

	// Reify him
	b, _ = ioutil.ReadFile("yogi.json")
	reifiedYogi := YogiBear{}
	json.Unmarshal(b, &reifiedYogi)
	fmt.Printf("This is reifiedYogi: %v\n", reifiedYogi)
}
~~~

Output:

~~~
This is yogi: {Smarter than average}
This is yogi's JSON encoding: {"Intelligence":"Smarter than average"}
This is reifiedYogi: {Smarter than average}
~~~

### Web/HTTP Support
Go has immense support for web programming. This is not surprising since Google
initiated the development of Go (though, it is an open source project), and Google
is primarily a web services company. It would be futile to attempt to document
all of Go's web support here, but I would like to show a representative example:
it is incredibly easy to [start up a simple static file server in Go](https://github.com/golang/go/wiki/HttpStaticFiles):

~~~go
package main

import "net/http"

func main() {
    panic(http.ListenAndServe(":8080", http.FileServer(http.Dir("/usr/share/doc"))))
}
~~~

Run this, and you can access all documents located in `/user/share/doc` (of course,
you can substitute this with any file system directory) via `localhost:8080` on
your machine. Stunningly simple... There really is no comparison to Java in this
respect.

### The Go Playground
Google hosts the [Go Playground](https://play.golang.org/), which allows for easy
testing of Go code in a REPL-like environment online. It is incredibly handy when
first learning Go (and forever thereafter). It also allows you to create shareable 
links to code you write, e.g. [https://play.golang.org/p/mY2rzovmf4](https://play.golang.org/p/mY2rzovmf4). 
This allows you to embed code examples in documentation, thereby allowing users to 
experiment and run your code without ever leaving the docs, e.g. 
[https://golang.org/pkg/io/ioutil/#ReadAll](https://golang.org/pkg/io/ioutil/#ReadAll)
(click the "Example" link to view the playground snippet). It even has a virtual
file system, so you can use file IO code _within the playground_.

### Libraries and "External Code"
Java has JAR files. They allow you to write and compile your code, and create
a single binary file out of it. You can share this JAR file with others, who can 
then use it to import your classes and use their public APIs to interact with 
them without needing their source code.

Go does not have a an equivalent mechanism. For better or worse, if you wish to 
publish a Go library, then you will also have to share the source code for it.
The [`vendor` directory](https://golang.org/cmd/go/#hdr-Vendor_Directories) of a
project contains the source code of all its dependencies. This source is then
compiled when you build your project, just like the code you wrote.

### Summary
I think that Go is a very cool language. It has some quirky features, but they
generally turn out to be quite useful once you grow accustomed to them. There is
a large community of Go developers, which means it is possible to find libraries
to accomplish most tasks, if the standard libraries won't suffice. And when there
aren't native go packages available, it is always possible to create bindings for
C/C++ code (and you'll be hard pressed to find a task that somebody hasn't already
written C/C++ code to accomplish). Go is a pleasure to use, and I look forward
to working with it more in the future!
