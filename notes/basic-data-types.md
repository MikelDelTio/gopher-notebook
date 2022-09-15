# Primitive Types

The following page hosts the best practises and conventions about Go basic data types.

## Table of Contents

- [Constants](basic-data-types.md#constants)
    - [Naming](basic-data-types.md#constants---naming)
    - [Declaration](basic-data-types.md#constants---declaration)
- [Enumerations](basic-data-types.md#enumerations)
    - [Declaration](basic-data-types.md#enumerations---declaration)

## Constants

The following section hosts the best practises and conventions about Go constants.

### Constants - Naming

Constants in Go are like in any other programming language: `Numeric`, `Boolean`, `String`, `Rune`
or `Built-in Function` type variables whose expression is evaluated at compile time, and could not be changed at run
time.

However, they are not named with uppercase characters separated by underscores. Instead, camel case naming convention is
employed, just like for all other [variables](program-structure.md#variables---naming). These names should be
descriptive, making it clear what the value represents.

```go
const STATUS_OK = 200    // Bad
const StatusOk int = 200 // Bad
const StatusOK = 200     // Good
```

### Constants - Declaration

Again, just like for all other [variables](program-structure.md#variables---declaration), the best practise is to
exclude the underlying type both from name and declaration in Go constants, but in this case not only because the
compiler infers the type, but also because performs an automatic conversion, if the value can be represented in the
target type at least.

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

## Enumerations

The following section hosts the best practises and conventions about Go enumerations.

### Enumerations - Declaration

Go does not have support for enumerations like in the most programming languages, that is, types that only contain a
limited set of values. Instead, it has iota, which allows to assign an increasing value to a set of custom type int
constants.

```go
type Season int64

const (
	Summer Season = iota // Value: 0
	Autumn               // Value: 1
	Winter               // Value: 2
	Spring               // Value: 3
)
```

However, it should not be used as common enumerations, since they are referred by value rather than by name, so in case
of inserting a new identifier in the middle list, all the subsequent ones will be affected, and therefore, the
application could fail.

```go
type Season int64

const (
	None   Season = iota // Value: 0
	Summer               // Value: 1
	Autumn               // Value: 2
	Winter               // Value: 3
	Spring               // Value: 4
)
```

Thus, the best practise for enumerations is to explicitly write the constant values instead of using iota, since allows
to insert new values at any moment without the risk of breaking the application. Leave iota for internal purposes and
just to differentiate the type of set of values, but not the value itself.

```go
type Season int64

const (
	Summer Season = 0
	Autumn        = 1
	Winter        = 2
	Spring        = 3
)
```

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)