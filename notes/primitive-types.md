# Primitive Types

The following page hosts the best practises and conventions about Go primitive types.

## Table of Contents

- [Naming](primitive-types.md#naming)

## Naming

Variables in Go follows [MixedCaps](https://go.dev/doc/effective_go#mixed-caps) naming convention, that is, words
separated with a single capitalized letter. The first letter can be uppercase or lowercase, depending on whether the
item is accessible or not outside the package.

This rule must be followed even when it breaks conventions in other languages, like the use of capital letters and
underscores to name constants.

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