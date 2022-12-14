# Functions

The following page hosts the best practises and conventions about Go functions.

## Table of Contents

- [Naming](functions.md#naming)
- [Grouping Parameters](functions.md#grouping-parameters)
- [Named Result Parameters](functions.md#named-result-parameters)
- [Naked Returns](functions.md#naked-returns)
- [Init](functions.md#init)

## Naming

Functions in Go follows the same naming convention as [variables](program-structure.md#variables---naming), except that
test and benchmark functions starts with ```Test``` or ```Benchmark``` word.

```go
func Hello_Word() {}                     // Bad, the function name should not contain the undescore symbol
func HelloWord()  {}                     // Good

func HelloWord(t *testing.T) {}          // Bad, the test function name should begin with Test word
func TestHelloWord(t *testing.T) {}      // Good

func HelloWord(b *testing.B) {}          // Bad, the benchmark function name should begin with Benchmark word
func BenchmarkHelloWord(b *testing.B) {} // Good
```

Sources:

- [pkg.go.dev](https://pkg.go.dev/testing)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#function-names)

## Grouping Parameters

Go allows to group a sequence of input parameters of the same type, so that the type itself is written only
once, producing an idiomatic codebase that favors the readability and maintainability.

```go
func minus(x int, y int) int {} // Bad, parameters of the same type should be grouped
func minus(x, y int) int {}     // Good
```

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [Pro Go by Adam Freeman](https://link.springer.com/book/10.1007/978-1-4842-7355-5)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)

## Named Result Parameters

Go allows to specify names to the result parameters, which automatically are pre-declared and initialized with their
types zero values when the function begins, and it can be returned using a [naked return statement](#naked-returns).

```go
func concat(str1, str2 string) (result string) {
	result = str1 + str1
	return result
}
```

Although some might think that this technique makes the codebase shorter and clearer, or even consider it documentation,
it does also have some potential corner cases that discourage its use in most cases.

First, by not declaring explicitly the return parameters, it is easier to shadow them, so you have to be very careful
and use the ```=``` operator to update their value every time. The following code snippet is a good example of this,
where false (boolean zero value) is always returned, regardless of the input parameter, since result is shadowed.

```go
func isOdd(x int) (result bool) {
	if x%2 == 0 {
		result := true // Bad, the result variable is shadowed
		println(result)
	}
	return result
}
```

Secondly, specifying names to the result parameters does not imply that must be returned. That is, if any other values
are assigned to the return statement, they will be returned as well, even though they were never assigned to the named
return parameters. This makes the code confusing.

```go
func mul(x, y int) (result int) {
	result = x * y
	return 5
}
```

Even so, there are some specific situations where this feature can be helpful, such as to modify a return argument in a
defer block. For example, closing a file in a defer function, allows to properly handle the error, and update the named
return parameter.

```go
func readFile(name string) (err error) {
	f, err := os.Open(name)
	if err != nil {
		return err
	}

	defer func() {
		cerr := f.Close()
		if err == nil {
			err = cerr
		}
	}()
    
    // ...
}
```

It could be also useful to differentiate return arguments in those cases where they share the underlying type.

```go
func Split(path string) (string, string) {}   // Bad, a named result parameter should be used to differentiate same type return arguments
func Split(path string) (dir, file string) {} // Good
```

In any case, note that its use is not recommended beyond these two exceptions.

Sources:

- [CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments#named-result-parameters)
- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)

## Naked Returns

A naked return is a return statement without arguments in which named return values are automatically returned.

```go
func div(x, y int) (result int) {
	result = x / y
	return
}
```

This return style is strongly discouraged, since makes the code base even more confusing than the [named result
parameters](#named-result-parameters), difficulting the readability and maintainability.

On the one hand, it looks like the function does not return values, when it does. On the other hand, makes it hard to
understand what values are actually returned, forcing you to read backwards the entire code base to find the last
assignments of the variables.

```go
func getRandomNumber() (result int) {
	result = rand.Intn(100)
	if result > 16 {
		result = result + 256
	} else {
		result = 512
	}
	return
}
```

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)

## Init

Go provides a lifecycle hook named ```init```, a package level function that takes no parameters, returns no values, and
runs the first time the package is loaded. As its name suggests, it is used to initialize package level variables before
business functions are executed, usually with the aim of reducing its execution time.

```go
var db *Database

func init() {
	conn, err := sql.Open("postgres", "connectionURI")
	if err != nil {
		log.Fatal(err)
	}
	db = &Database{Connection: conn}
}
```

Unfortunately, it should not be used. It is a bad practice to have mutable states at the top level of a package, since
makes harder to understand how the application works. That is, functions no longer depend only on their input
parameters, but also on external values.

Moreover, this data could be modified concurrently by other executions or even functions, producing data races that lead
to unexpected errors, and Go does not provide a way to enforce that value does not change after first assignment
on ```init``` function.

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#avoid-init)