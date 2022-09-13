# Composite Types

The following page hosts the best practises and conventions about Go composite types.

## Table of Contents

- [Structs](composite-types.md#structs)
    - [Initialization](composite-types.md#initialization)
    - [Fields Ordering](composite-types.md#fields-ordering)
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

### Fields Ordering

Properly ordering struct fields allows to improve both application’s performance and memory usage, due to modern CPU
works efficiently when the data is naturally aligned. Let's explain this.

First, computers stores data at an address equals to the multiple of the data size. That is, a 2 bytes data can be
stored in 0, 2 or 4 bytes address, while a 4 bytes data can be stored in a 0, 4 or 8 bytes. Go follows the "required
alignment" methodology, so a struct field memory address size is equals to the memory required by the largest field in
the struct. The empty bytes between the memory addresses is called padding.

```go
type customer struct {
	verified bool    // size: 1 byte,  padding: 7 bytes
	id       float64 // size: 8 bytes, padding: 0 bytes
	age      int32   // size: 4 bytes, padding: 4 bytes
}

Total memory consuption: 24 bytes
```

However, multiple fields can be stored in the same memory address, as long as their size is less than the required
alignment. Therefore, properly ordering struct fields allows to reduce the memory consumption.

```go
type customer struct {
	id       float64 // size: 8 bytes, padding: 0 bytes
	verified bool    // size: 1 byte,  padding: 3 bytes
	age      int32   // size: 4 bytes, padding: 0 bytes
}

Total memory consuption: 16 bytes
```

On the other hand, CPU reads and writes data from memory addresses in the multiple of its word size. In a 32bit
architecture a word is equivalent to 4 byte memory block, while in a 64bit architecture the memory block is 8 byte.
Thus:

- When a struct field memory address size is smaller than the CPU word, the processor has to use one cycle to access
  field.
- When a struct field memory address size is greater than the CPU word, the processor has to use two or more cycles to
  access field, depending on its size.
- When a memory address stores multiple struct fields and its size is equals or smaller to the CPU word, the processor
  can use a single cycle to access as many fields as fit in a single world.

Therefore, if a structure is not optimally word-aligned, the CPU has to perform extra operations to load the value into
a register so that it can be used.

One last tip, tools like [fieldalignment](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/fieldalignment) could
be so useful to find structs to optimize on your codebase.

Sources:

- [awstip.com](https://awstip.com/optimizing-memory-by-changing-the-order-of-struct-field-485106504087)
- [betterprogramming.pub](https://betterprogramming.pub/how-to-speed-up-your-struct-in-golang-76b846209587)
- [itnext.io](https://itnext.io/structure-size-optimization-in-golang-alignment-padding-more-effective-memory-layout-linters-fffdcba27c61)
- [wagslane.dev](https://wagslane.dev/posts/go-struct-ordering/)

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