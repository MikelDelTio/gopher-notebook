# Methods

The following page hosts the best practises and conventions about Go methods.

## Table of Contents

- [Receiver](methods.md#receiver)
- [Getters & Setters](methods.md#getters--setters)

## Receiver

Methods in Go are declared like ordinary functions, with one addition: the receiver specification, an extra parameter
between the ```func``` keyword and the method name.

First, the convention among Go's community is to use an abbreviation as the receiver name, specifically, the first
letter of each word that makes up the name of the struct. The use of ```this``` or ```self``` keywords is non-idiomatic
and should be avoided.

```go
type Course struct{}

func (course Course) Enroll() {} // Bad, the receiver name should be an abbreviation, no the full name
func (this Course) Enroll() {}   // Bad, the receiver name should not be this
func (self Course) Enroll() {}   // Bad, the receiver name should not be self

func (c Course) Enroll() {}      // Good
```

Secondly, receivers could be of pointer or value type, just like any parameter, so the same rules should be applied to
determine when to use each kind of them.

Thus, the pointer type is the preferred choice when the method needs to modify the receiver or use nil as zero value. It
also could provide a better performance when it contains lots of data, but there is no a default value to determine how
much kb / mb should the receiver be, so you will need to benchmark it.

In any case, if only one method satisfied any of the above conditions, the best practise is to be consistent and use
pointer receivers for all methods, even the ones that don’t modify the receiver.

```go
type Employee struct {
	salary float32
}

func (e *Employee) AnnualRaise() {
	e.salary = e.salary * 1.10
}
```

Otherwise, the value type receiver should be used, which is the only way to ensure data immutability.

```go
type Record struct {
	value []byte
}

func New(value []byte) Record {
	return Record{value: value}
}

func (r Record) Value() []byte {
	return r.value
}
```

Sources:

- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments#receiver-type)
- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)
- [When to use pointers in Go by Dylan Meeus](https://medium.com/@meeusdylan/when-to-use-pointers-in-go-44c15fe04eac)

## Getters & Setters

Go's convention is to directly access structs fields, instead of writing getter and setter methods. Methods should be
reserved just for business logic.

```go
type User struct {
	Name string
}

func main() {
	user := User{}
	user.Name = "Bob"
	fmt.Printf("The user's name is %s", user.Name)
}
```

Nonetheless, there are exceptions like when you need them to meet an interface or when it is not a direct assignment
operation, either because of the complexity or because multiple fields are involved. In these cases, Get prefix (or
similar) should be avoided, but setter word in maintained.

```go
type Nameable interface {
	Name() string                
}

type User struct {
	name string
}

func (u *User) Name() string {
	return u.name
}

func (u *User) SetName(name string) {
	u.name = name
	// ...
}
```

Sources:

- [Effective Go](https://go.dev/doc/effective_go#Getters)
- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)