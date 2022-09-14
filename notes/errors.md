# Errors

The following page hosts the best practises and conventions about Go errors.

## Table of Contents

- [Handling](errors.md#handling)
- [Wrapping](errors.md#wrapping)

## Handling

Go's philosophy is that failure is part of normal operation, and errors should be treated as first-class citizens. For
this reason, and unlike exception-based languages, it takes advance of the support for multiple return values to handle
errors, by returning an error type parameters as the last return value for a function. The latter is not strictly
necessary, but it is a convention among the Go's community.

Thus, the calling function is responsible for checking the return error argument, comparing it to nil, and taking
appropriate action, such as propagate it, so that becomes a failure of the calling routine, retry the failed operation,
probably with a delay between tries, or print the error and stop the program gracefully, an extremely discouraged
option, except for the main package of a program.

In case there are multiple return parameters, zero value is explicitly assigned on the return statement. Do not use
previously declared variables, unless it is really necessary, since forces you to read backwards the entire code base to
find the last assignments, and can contain a non-zero value that confuses the function caller.

```go
func getUsers() ([]User, error) {
	dbUsers, err := getUsersFromDatabase()
	if err != nil {
		return nil, err
	}
	users, err := parseDatabaseUsers(dbUsers)
	if err != nil {
		return nil, err
	}
	return users, nil
}
```

As has been said, there are some error cases where partial results can be useful. For example, the ```io.ReadAll```
function returns read data until an error or EOF, so the caller function can process the incomplete information before
handling the error. Anyway, this unusual behavior should be clearly documented.

```go
func ReadAll(r Reader) ([]byte, error) {
	b := make([]byte, 0, 512)
	for {
		if len(b) == cap(b) {
			// Add more capacity (let append pick how much).
			b = append(b, 0)[:len(b)]
		}
		n, err := r.Read(b[len(b):cap(b)])
		b = b[:len(b)+n]
		if err != nil {
			if err == EOF {
				err = nil
			}
			return b, err
		}
	}
}
```

This error handling style may seem unnatural to those developers used to work with exception based programing languages,
but there are solid software engineering principles underlying this approach.

First, although the exception handling style may produce shorter code, that doesn't mean it's easier to maintain. In
fact, usually the opposite happens. Go bets on an idiomatic style that favors the readability and maintainability, even
if it takes more lines.

Seconds, exceptions produces at least one new code path, which could be invisible on those languages that don’t include
a throw exception declaration in the function signature. Imagine what happens when they aren't properly handled.
Instead, Go uses the ordinary control-flow mechanisms like if and return to respond to errors. Requires to pay more
attention, but on the other hand, reduces unexpected behaviours.

Last but not least, Go's error handling style is governed by the same compiler rules as the business code, that is, the
developers must read all declared variables, in this case, the error ones, and either take previously described actions
or explicitly ignore it by using an underscore (_) symbol. It goes without saying that ignoring possible errors is never
a good practice.

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [The Go Programming Language by Alan A. A. Donovan and Brian W. Kernighan](https://www.gopl.io)

## Wrapping

Error wrapping consist on providing additional context to an error before propagating it, such as the name of the
function that received the error, or the operation it was trying to perform, so the callers have more information to
take more informed decisions. Thus, these are some advices about whether and how an error should wrap the original.

First, it is bad practice to wrap an error on every calling function to form an error chain that mimics a stacktrace.
That goes against Go's error philosophy. Instead, return the original error as-is, as long as the underlying error
message provides enough information to track down where it came from.

```go
func readFile(name string) (err error) {
	file, err := os.Open(name)
	if err != nil {
		return err
	}
	...
}

error: open /home/workspace/test.file: The system cannot find the file specified
```

If not so, add useful context to the error message where possible, picking between the ```%w``` or ```%v``` verbs
on ```fmt.Errorf()``` function.

Thus, use the ```%w``` verb when the caller should have access to the underlying original error, since it provides
an ```Unwrap``` method to recover it. This is a good default for the 99.99% of cases, except when you must support an
1.13 older version of Go, or [xerrors](https://golang.org/x/xerrors) is not available. Don't forget to properly document
those cases in which the wrapped error is a known var or type, so that the caller can get the most out of it.

```go
func fromJSON(data []byte) (*User, error) {
	var user User
	err := json.Unmarshal(data, &user)
	if err != nil {
		return nil, fmt.Errorf("user from JSON: %w", err)
	}
	return &user, nil
}
```

Finally, use the ```%v``` to obfuscate the underlying error and hide implementation details, so that the caller will be
unable to match it.

```go
func fromJSON(data []byte) (*User, error) {
	var user User
	err := json.Unmarshal(data, &user)
	if err != nil {
		return nil, fmt.Errorf("user from JSON: %v", err)
	}
	return &user, nil
}
```

Sources:

- [The Go Blog](https://go.dev/blog/go1.13-errors)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#error-wrapping)