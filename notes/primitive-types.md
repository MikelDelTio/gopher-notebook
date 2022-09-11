# Primitive Types

The following page hosts the best practises and conventions about Go primitive types.

## Table of Contents

- [Naming](primitive-types.md#naming)
- [Declaration](primitive-types.md#declaration)
- [Constants](primitive-types.md#constants)

## Naming

Variables in Go follows [MixedCaps](https://go.dev/doc/effective_go#mixed-caps) naming convention, that is, words
separated with a single capitalized letter. The first letter can be uppercase or lowercase, depending on whether the
item is accessible or not outside the package.

This rule must be followed even when it breaks conventions in other languages, like the use of capital letters and
underscores to name [constants](primitive-types.md#constants).

```go
var first_name string // Bad
var firstName string  // Good
```

For words in names that are initialism or acronyms, a consistent case is applied, that is, they would be written
completely in uppercase o lowercase, depending on the first letter of the word.

```go
var HttpServer string // Bad
var HTTPServer string // Good

var jSONContent []byte // Bad
var jsonContent []byte // Good

var customerId int // Bad
var customerID int // Good
```

Finally, avoid using Go's [predeclared identifiers](https://go.dev/ref/spec#Predeclared_identifiers), such as types(
bool, byte, int...) or functions (copy, delete, len, make...) as variable names, since will shadow the original meaning
and introduce hard to find bugs, especially in those cases where the compiler is not able to detect it.

```go
func main() {
	fmt.Println(false) // Prints false
	false := 999.99
	fmt.Println(false) // Prints 999.99
}
```

Sources:

- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments#initialisms)
- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#avoid-using-built-in-names)

## Declaration

Go provides several ways to declare variables, both primitive and composite types, and each one communicates something
about how it is used, but the most common convention within functions is to use the ```:=``` operator. Just follow the
KISS (keeping it simple stupid) design principle whenever possible.

```go
var hello string = "world" // Bad
var hello = "world"        // Bad
hello := "world"           // Good
```

Even so, there are some cases in which it should not be used, such as to initialize a variable to its zero value, where
the long form style is recommended. It shows clearer the intended zero value.

```go
found := false // Bad
var found bool // Good
```

The long form style is also the suitable way when the default type for the assignment is not the wanted type for the
variable. It defines in a more idiomatic way, without requiring an explicit type conversion.

```go
buffer := byte(256)   // Bad
var buffer byte = 256 // Good
```

One last tip, pay special attention to the scope when using the ```:=``` operator, if you don't want to shadow the
variable at least. That is, this operator allows to assign values to both new and existing variables, so it could create
a new variable or reuse existing one, depending on the code block.

```go
func main() {
	x := 1
	if true {
		x := 2
		fmt.Println(x) // Prints 2
	}
	fmt.Println(x) // Prints 1
}
```

In those cases, explicitly declare the conflicting variable with the var keyword, and use the ```=``` operator to update
its value every time. It looks clearer which variables are new.

```go
func main() {
	var x = 1
	if true {
		x = 2
		fmt.Println(x) // Prints 2
	}
	fmt.Println(x) // Prints 2
}
```

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)

## Constants

Constants in Go are like in any other programming language: `Numeric`, `Boolean`, `String`, `Rune`
or `Built-in Function` type variables whose expression is evaluated at compile time, and could not be changed at run
time.

However, they are not named with uppercase characters separated by underscores. Instead, camel case naming convention is
employed, just like for all other [variables](primitive-types.md#naming). These names should be descriptive, making it
clear what the value represents.

```go
const STATUS_OK = 200    // Bad
const StatusOk int = 200 // Bad
const StatusOK = 200     // Good
```

As can be seen, the underlying type is excluded both from name and declaration, again, just like for all
other [variables](primitive-types.md#declaration), but in this case the Go compiler not only infers the type, but also
performs an automatic conversion, if the value can be represented in the target type at least.

```go
func main() {
	const num1 float32 = 99.99
	const num2 int = 2               // Error, mismatched types float32 and int
	const num2 = 2                   // Good, prints 101,99
	fmt.Println("Total:", num1+num2) 
}
```

Sources:

- [Effective Go](https://go.dev/doc/effective_go#constants)
- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [Pro Go by Adam Freeman](https://link.springer.com/book/10.1007/978-1-4842-7355-5)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)
