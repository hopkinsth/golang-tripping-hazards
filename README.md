### Golang tripping hazards
---
Any time I learn something new, I always end up finding a few things that I really wish someone had told me at the beginning. If you're starting to learn Go, perhaps you'll find these tidbits useful.
---
**The defer statement**: A reserved word in Go that causes a function to execute as the last thing that happens when a function returns. You can stack as many as you want and they execute in the reverse order that they were declared. It looks something like this:
```go
func myFunction(intArg int, stringArg string) (int, error) {
	networkConn := openNetworkConnection()
	defer networkConn.Close()
	if intArg < 0 {
		errMsg := fmt.Sprintf("myFunction can only deal with positive integers, invalid argument: %v", intArg)
		return 0, errors.New(errMsg) // right after this, your deferred networkConn.Close() will execute
	}
	return intArg, nil // right after this, your deferred networkConn.Close() will execute
}
```
However, there's a complication. If the function you name in your defer statement takes arguments, and if those arguments are the return values of other functions, the argument-functions execute at the time you declare the defer, and the deferred function itself executes at function exit:
```go
func myFunction(intArg int, stringArg string) (int, error) {
	networkConn := openNetworkConnection()
	//determineHowConnShouldBeClosed() executes NOW, its return-value is stored as an
	// argument to closeMyNetworkConnection when it finally gets run.
	defer closeMyNetworkConnection(networkConn, determineHowConnShouldBeClosed())
	if intArg < 0 {
		errMsg := fmt.Sprintf("myFunction can only deal with positive integers, invalid argument: %v", intArg)
		return 0, errors.New(errMsg) // right after this, your deferred networkConn.Close() will execute
	}
	return intArg, nil // right after this, your deferred networkConn.Close() will execute
}
```
---
**The := symbol**: It's not an emoticon :shippit: it's an assignment operator that automatically infers the type. In this way:
```go
var foo int
foo = 5
```
is equivalent to:
```go
foo := 5
```
but if you're not careful, it can mess with your scoping:
```go
foo, err := someFunction() // in this case, err is nil
if true {
	bar, err := someOtherFunction() // in this case, err is not nil, but it will
                                        // go out of scope at the end of the block
}
// out here, we're dealing with the original (nil) declaration of err
if err != nil {
	fmt.Printf("we had an error: " + err.Error())
}
====================================
(this prints nothing)
```
