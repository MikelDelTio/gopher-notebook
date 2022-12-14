# Basic Data Types

The following page hosts the best practises and conventions about Go basic data types.

## Table of Contents

- [Integers](basic-data-types.md#integers)
    - [Typing](basic-data-types.md#integers---typing)
- [Floats](basic-data-types.md#floats)
    - [Typing](basic-data-types.md#floats---typing)
    - [Limitation](basic-data-types.md#floats---limitation)
- [Constants](basic-data-types.md#constants)
    - [Naming](basic-data-types.md#constants---naming)
    - [Declaration](basic-data-types.md#constants---declaration)
- [Enumerations](basic-data-types.md#enumerations)
    - [Declaration](basic-data-types.md#enumerations---declaration)

## Integers

The following section hosts the best practises and conventions about Go integers.

### Integers - Typing

Go provides multiple types of signed and unsigned integers, all of them from 1 to 8 bytes, which can be
overwhelming-decor at first. That's why the following guidelines it is so recommended to decide which one to use
for each situation.

| Name   | Range                                       |
|--------|---------------------------------------------|
| int8   | –128 to 127                                 |
| int16  | –32768 to 32767                             |
| int32  | –2147483648 to 2147483647                   |
| int64  | –9223372036854775808 to 9223372036854775807 |
| uint8  | 0 to 255                                    |
| uint16 | 0 to 65535                                  |
| uint32 | 0 to 4294967295                             |
| uint64 | 0 to 18446744073709551615                   |

First of all, remember that type just should be explicitly specified to initialize a variable to its zero value, or
when the default type for the assignment is not the wanted type for the variable.

Thus, if the use case absolutely requires a well-defined integer of a specific size or sign, for example, to implement a
particular network protocol specification, or to support a certain device architecture, use it, but never for trivial
reasons or personal preferences.

```go
// package socket. cmsghdr_linux_32bit.go 
func (h *cmsghdr) set(l, lvl, typ int) {
	h.Len = uint32(l)
	h.Level = int32(lvl)
	h.Type = int32(typ)
}

// package socket. cmsghdr_linux_64bit.go 
func (h *cmsghdr) set(l, lvl, typ int) {
	h.Len = uint64(l)
	h.Level = int32(lvl)
	h.Type = int32(typ)
}
```

On the other hand, if the use case requires to work with any integer type, it is no longer necessary to declare a pair
of functions with ```int64``` and ```uint64``` for the parameter type, and force callers to make types conversion, as
can still been seen on FormatInt/FormatUint functions in the strconv package of the Go standard library.

```go
func FormatUint(i uint64, base int) string {
	if fastSmalls && i < nSmalls && base == 10 {
		return small(int(i))
	}
	_, s := formatBits(nil, i, base, false, false)
	return s
}

func FormatInt(i int64, base int) string {
	if fastSmalls && 0 <= i && i < nSmalls && base == 10 {
		return small(int(i))
	}
	_, s := formatBits(nil, uint64(i), base, i < 0, false)
	return s
}
```

Currently, the most elegant option is to take advantage of the generics introduced in Go 1.18, and use
the ```constraints.Integer``` as function parameter type to support any kind of integer.

```go
func add[T constraints.Integer](x, y T) T {
	return x + y
}

func main() {
	var num1 int8 = 1
	var num2 int8 = 1
	fmt.Println(add(num1, num2)) // Good, prints 2

	var num3 int16 = 2
	var num4 int16 = 2
	fmt.Println(add(num3, num4)) // Good, prints 4
}
```

For all other use cases, just use ```int``` or ```uint```, that are the natural or most efficient size for signed and
unsigned integers on a particular platform. (32 or 64 bit depending on the CPU architecture)

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)

## Floats

The following section hosts the best practises and conventions about Go integers.

### Floats - Typing

Go uses the IEEE 754 specification to provide 32 or 64 bits floats, but unlike
for [integers](basic-data-types.md#integers---typing), the guideline to decide which one to use for each situation is
very simple.

| Name    | Largest absolute value                         | Smallest (nonzero) absolute value              |
|---------|------------------------------------------------|------------------------------------------------|
| float32 | 3.40282346638528859811704183484516925440e+38   | 1.401298464324817070923729583289916131280e-45  |
| float64 | 1.797693134862315708145274237317043567981e+308 | 4.940656458412465441765687928682213723651e-324 |

The convention is to use ```float64``` as the preferred type, unless you have to be compatible with an existing format.
The main reasons are that it provides a higher level of precision, 6 digits against 15 approximately , and that the
difference in memory consumption is insignificantly for most use cases. Thus, the use of ```float32```type will be
considered a premature optimization until proven otherwise.

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)

### Floats - Limitation

Although Go floats provides a huge range of precession, they are not exact (stores the nearest approximation), and they
can not be compared using ```==```  or ```!=``` operators. Instead, a maximum allowed variance is employed to check if
the difference between two floats is less than that.

Therefore, they only should be used in those situations where inexact values are acceptable, like graphics and
scientific operations, and never to represent money or for monetary operation.

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)

## Constants

The following section hosts the best practises and conventions about Go constants.

### Constants - Naming

Constants in Go are like in any other programming language: `Numeric`, `Boolean`, `String`, `Rune`
or `Built-in Function` type variables whose expression is evaluated at compile time, and could not be changed at run
time.

However, they are not named with uppercase characters separated by underscores. Instead, camel case naming convention is
employed, just like for all other [variables](program-structure.md#variables---naming). These names should be
descriptive, making it clear what the value represents, but without any reference to the underlying type.

```go
const STATUS_OK_INT = 200 // Bad, the constant name should not contain the underlying type
const STATUS_OK = 200     // Bad, the constant name should not contain the underscore symbol
const StatusOk int = 200  // Bad, the constant should not be declared using the long form style
const StatusOK = 200      // Good
```

Sources:

- [Pro Go by Adam Freeman](https://link.springer.com/book/10.1007/978-1-4842-7355-5)

### Constants - Declaration

It is a best practise to exclude the underlying type also from the Go constants declaration, again, just like for all
other [variables](program-structure.md#variables---declaration), not only because the compiler infers the type, but also
because allows to perform an automatic conversion, if the value can be represented in the target type at least (This
conversion does not happen on non-constant variables).

```go
var num1 float32 = 99.99
num2 := 2
fmt.Println("Total:", num1+num2) // Good, prints 101,99

const num1 float32 = 99.99
const num2 int = 2
fmt.Println("Sum:", num1+num2)   // Error, mismatched types float32 and int

const num1 float32 = 99.99
const num2 = 2
fmt.Println("Sum:", num1+num2)   // Good, prints 101,99
```

Sources:

- [Effective Go](https://go.dev/doc/effective_go#constants)
- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
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

Thus, the best practise for enumerations is to explicitly declare the constant values instead of using iota, since
allows to insert new values at any moment without the risk of breaking the application. Leave iota for internal purposes
and just to differentiate the type of set of values, but not the value itself.

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