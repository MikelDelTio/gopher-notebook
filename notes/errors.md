# Errors

The following page hosts the best practises and conventions about Go errors.

## Table of Contents

- [Handling](errors.md#handling)
- [Wrapping](errors.md#wrapping)
- [Panic & Recover](errors.md#panic--recover)
- [Exit](errors.md#exit)

## Handling

Go's philosophy is that failure is part of normal operation, and errors should be treated as first-class citizens. For
this reason, and unlike exception-based languages, it takes advance of the support for multiple return values to handle
it, by returning an error type parameter as the last return value for a function. The latter is not strictly
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
func readFile(name string) error {
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

## Panic & Recover

A panic is unexpected situation, such as a programming error (accessing out of range of an array) or an environmental
problem (out of disk space), where Go runtime is unable handled gracefully and terminates the running program.

```go
func main() {
	characters := []string{"Anakin Skywalker", "Luke Skywalker", "Leia Organa", "Han Solo", "Grogu"}
	fmt.Println("My favorite Star Wars character is:", characters[len(characters)])
}

panic: runtime error: index out of range [5] with length 5
```

Although it is possible to programmatically panic, this is something that must be avoided, whatever it takes. When an
error occurs, just return it and allow the caller to decide how to handle it, as described on
the[wrapping section](#wrapping).

In the case in which an error is irrecoverable and requires to terminate the running program, the best practise is to
handle it in the ```main``` function, log the error message and end the execution using the ```log.Fatal*``` command.

```go
func main() {
	file, err := readFile("random.json")
	if err != nil {
		panic(err)     // Bad
		log.Fatal(err) // Good
	}
	// ...
}
```

One last tip, Go provides a recover function to capture a panic, but this should not be used to emulate an exception
base error handling strategy. A program must panic only when something irrecoverable happens, so be careful about trying
to continue program execution, it would probably fail again. Use the recover capabilities to gracefully handle these
situations, for example, to log the situation and shut down using the ```log.Fatal*``` command.

```go
func main() {
	defer func() {
		if err := recover(); r != nil {
			log.Fatal(err)
		}
	}()
	panic("something irrecoverable happens")
}
```

Nonetheless, there is one situation where recovering from a panic and continuing execution can be useful, and that is to
avoid panic escapes a library boundaries. A library should never end a program, that is a decision that corresponds to
the main program, so use the recover function to capture the panic, convert into an error, and return it to signal
failure.

```go
func parseJWT(tokenString string) (*Token, error) {
	defer func() {
		if r := recover(); r != nil {
			return nil, errors.New("Error: ", r)
		}
	}()
	// ...
}
```

Sources:

- [Learning Go by Jon Bodner](https://www.oreilly.com/library/view/learning-go/9781492077206/)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#dont-panic)

## Exit

Go provides the ```os.Exit(1)``` and ```log.Fatal*``` functions to immediately finalize a program, but the latter is
recommended since also print the error message.

In any case, it should be used just in the ```main``` function, and at most once. Multiple exit sentences complicates
the control flow and makes difficult to debug errors, which ultimately hurts code readability and maintainability.
Testing is also affected as there is more than one critical point of failure to manage.

```go
func run() error {
	config, err := readConfigFromEnvironment()
	if err != nil {
		return err
	}

	fileContent, err := readFile(config.FileName)
	if err != nil {
		return err
	}

	// ...
}

func main() {
	err := run()
	if err != nil {
		log.Fatal(err)
	}

}
```

Sources:

- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#dont-panic)