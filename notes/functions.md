# Functions

The following page hosts the best practises and conventions about Go functions.

## Table of Contents

- [Naming](functions.md#naming)
- [Named Result Parameters](functions.md#named-result-parameters)

## Naming

Functions in Go follows the same naming convention as [variables](program-structure.md#variables---naming), except that
test and benchmark functions starts with ```Test``` or ```Benchmark``` word.

```go
func Hello_Word() {} // Bad
func HelloWord()  {} // Good

func HelloWord(t *testing.T) {}     // Bad
func TestHelloWord(t *testing.T) {} // Good

func HelloWord(b *testing.B) {}          // Bad
func BenchmarkHelloWord(b *testing.B) {} // Good
```

Sources:

- [pkg.go.dev](https://pkg.go.dev/testing)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#function-names)

## Named Result Parameters

Go allows to specify names to the result parameters, which automatically are pre-declared and initialized with their
types zero values when the function begins, and it can be returned using a naked return statement.

```go
func concat(str1, str2 string) (result string) {
	result = str1 + str1
	return result
}
```

While sometimes can help to make the codebase shorter and clearer, specially if the function returns multiple parameters
of the same type, they do have some potential corner cases.

First, by not declaring explicitly the return parameters, it is easier to shadow it, so you have to be very careful and
use the ```=``` operator to update its value every time. The following code snippet is a good example of this, where
false (boolean zero value) is always returned, regardless of the input parameter, since result is shadowed.

```go
func isOdd(a int) (result bool) {
	if a%2 == 0 {
		result := true // Shadowed variable
		println(result)
	}
	return result
}
```

Secondly, specifying names to the result parameters does not imply that must be returned. That is, if any other values
are assigned to the return statement, they will be returned as well, even though they were never assigned to the named
return parameters. This makes the code confusing.

```go
func sum(a, b int) (result int) {
	result = a + b
	return 5
}
```

Even so, there is one situation where it can be helpful, and that is to modify a return argument in a defer block. For
example, closing a file in a defer function, allows to properly handle the error, and update the named return parameter.

```go
func readFile(name string) (err error) {
	file, err := os.Open(name)
	if err != nil {
		return err
	}

	defer func() {
		cerr := file.Close()
		if err == nil {
			err = cerr
		}
	}()
    
    // ...
}
```

Sources:

- [CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments#named-result-parameters)
- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)