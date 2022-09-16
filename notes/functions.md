# Functions

The following page hosts the best practises and conventions about Go functions.

## Table of Contents

- [Naming](functions.md#naming)

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