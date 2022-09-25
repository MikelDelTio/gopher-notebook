# Program Structure

The following page hosts the best practises and conventions about Go program structure.

## Table of Contents

- [Packages](program-structure.md#packages)
    - [Naming](program-structure.md#packages---naming)
- [Variables](program-structure.md#variables)
    - [Naming](program-structure.md#variables---naming)
    - [Declaration](program-structure.md#variables---declaration)

## Packages

The following section hosts the best practises and conventions about Go packages.

### Packages - Naming

Package names in Go should be short and concise, a lower-case single-word singular terms that could
not contain capitals or underscores. By convention, these name should match with the base name of its source directory,
that is, a package located at ```src/compress/gzip``` is imported as ```compress/gzip``` but its name is ```gzip```.

```go
compress/gzips           // Bad, the package name should be written in singular
compress/gZip            // Bad, the package name should be written in lower-case
compress/gzip_compresser // Bad, the package name should not contain the underscore symbol
compress/gzipcompresser  // Bad, the package name should be a single-word term

compress/gzip            // Good
```

Don't worry about duplications, although package name is default the name for imports, does not have to be unique
neither
locally nor globally. In the event of a collision, aliasing could be used to provide an alternative name.

```go
import "runtime/trace"
import nettrace "golang.net/x/trace"
```

It is important to note that the elements of a package, such as [structs](composite-types.md#structs)
or [functions](functions.md), are externally referenced using the package name, so that name should be omitted from the
identifiers.

```go
package http

type HTTPClient struct {} // Bad, the package name should be ommited on package elements such as structs
type Client struct {}     // Good

func NewHTTPClient() {}   // Bad, the package name should be ommited on package elements such as functions
func NewClient() {}       // Good
```

Finally, avoid meaningless package names like util, shared, common or lib, among other, there are totally uninformative.

Sources:

- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments#package-names)
- [Effective Go](https://go.dev/doc/effective_go#package-names)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#package-names)

## Variables

The following section hosts the best practises and conventions about Go variables.

### Variables - Naming

Variables in Go follows [MixedCaps](https://go.dev/doc/effective_go#mixed-caps) naming convention, that is, words
separated with a single capitalized letter. The first letter can be uppercase or lowercase, depending on whether the
item is accessible or not outside the package.

This rule must be followed even when it breaks conventions in other languages, like the use of capital letters and
underscores to name [constants](basic-data-types.md#constants---naming).

```go
var first_name string // Bad, the variable name should not contain the underscore symbol
var firstName string  // Good
```

For words in names that are initialism or acronyms, a consistent case is applied, that is, they would be written
completely in uppercase or lowercase, depending on the first letter of the word.

```go
var HttpServer string  // Bad, acronyms should be written in complety uppercase or lowercase
var HTTPServer string  // Good

var jSONContent []byte // Bad, acronyms should be written in complety uppercase or lowercase
var jsonContent []byte // Good

var customerId int     // Bad, acronyms should be written in complety uppercase or lowercase
var customerID int     // Good
```

These names should be concise, but shorter than long, specially when the scope or even the life cycle of the variable is
limited. As commented in [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments#initialisms),
the basic rule is: the further from its declaration that a name is used, the more descriptive the name must be.

The following code snippet extracted from the ```ioutils``` package is a good example of this. Limited scope variables
such as loop indices and values are represent with a single letter, while a more descriptive name is employed for the
main input parameter of the function.

In the other hand, file variables is named as ```f```, following the convention among Go community to describe common
variables like ```file```, ```reader``` o```writer```, among many others, with a single-letter abbreviation, especially
when the previously mentioned rule is fulfilled.

```go
func ReadDir(dirname string) ([]fs.FileInfo, error) {
	f, err := os.Open(dirname)
	if err != nil {
		return nil, err
	}
	list, err := f.Readdir(-1)
	f.Close()
	if err != nil {
		return nil, err
	}
	sort.Slice(list, func(i, j int) bool { return list[i].Name() < list[j].Name() })
	return list, nil
}
```

Regardless of size, variable type should be excluded from the name. After all, Go is strongly typed, so it is
unnecessary to track the underlying type on the name.

```go
usersMap := make(map[User]int) // Bad, variable name should not contain the underlying type
users := make(map[User]int)    // Good
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

### Variables - Declaration

Go provides several ways to declare variables, both basic data and composite types, and each one communicates something
about how it is used, but the most common convention within functions is to use the ```:=``` operator. Just follow the
KISS (keeping it simple stupid) design principle whenever possible.

```go
var hello string = "world" // Bad, a non-zero value variable should be initialized using the := operator
var hello = "world"        // Bad, a non-zero value variable should be initialized using the := operator
hello := "world"           // Good
```

Even so, there are some cases in which it should not be used, such as to initialize a variable to its zero value, where
the long form style is recommended. It shows clearer the intended zero value.

```go
found := false // Bad, a zero value variable should be initialized using the log form style
var found bool // Good
```

The long form style is also the suitable way when the default type for the assignment is not the wanted type for the
variable. It defines in a more idiomatic way, without requiring an explicit type conversion.

```go
buffer := byte(256)   // Bad, explicit type conversion should not be used in variables initialization
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