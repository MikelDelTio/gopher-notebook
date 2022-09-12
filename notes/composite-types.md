# Composite Types

The following page hosts the best practises and conventions about Go composite types.

## Table of Contents

- [Structs](composite-types.md#structs)
    - [Initialization](composite-types.md#initialization)
    - [Marshalling & Unmarshalling](composite-types.md#marshalling--unmarshalling)

## Structs

The following section hosts the best practises and conventions about Go structs.

### Initialization

Go provides several ways to initialize a struct, but the recommendation is to use the map literal style, in which both
the name and value of the desired fields must be explicitly defined. Not declared fields get its zero value.

The main advantages are that it makes clear what value is being assigned to what field, favoring the readability and
maintainability, while allows to add new fields to the struct, without getting compilation errors on the existing
codebase.

```go
type customer struct {
	name string
	age  int
}

customer := customer{name: "Bob", age: 32}
```

The struct literal style instead, declares a value for each struct field in the order they were defined in the struct,
so any change to it forces to update its initialization at all existing point, or the program will not compile. Although
this style may seem more comfortable for small structs, the recommendation is not to use it for the previously described
reasons.

```go
type customer struct {
	name    string
	surname string
	age     int
}

customer := customer{"Bob", 32} // Bad
customer := customer{name: "Bob", age: 32} // Ok
```

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#use-field-names-to-initialize-structs)

### Marshalling & Unmarshalling

Go provides tags, that is, one or more key/pair values separated by spaces, to define how structs should be marshalled
and unmarshalled to/from JSON. These tags allow to specify the name of the JSON field that should be associated with the
struct field, ignore empty fields or even unwanted fields, among others.

```go
type Macbook struct {
	id          int    `json:"-"`
	model       string `json:"model"`
	displayInch int    `json:"display_inch"`
	price       float  `json:"price, omitempty"`
}
```

Although Go's default behavior is to match JSON object field with the name of the Go struct field, the best practise and
also the convention among Go community is to explicitly annotate all the fields with the corresponding tag, even if they
are identical, guarding against accidental refactoring or renaming fields.

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#use-field-tags-in-marshaled-structs)