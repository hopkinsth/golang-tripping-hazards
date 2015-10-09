### Golang tripping hazards
---
#####Any time I learn something new, I always end up finding a few things that I really wish someone had told me at the beginning. If you're starting to learn Go, perhaps you'll find these tidbits useful.
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
**The := symbol**: It's not an emoticon it's an assignment operator that automatically infers the type. In this way:
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
	fmt.Printf("we had an error")
}
```
(this prints nothing)

if you find yourself writing something like that, you probably meant this instead:
```go
foo, err := someFunction() // err is nil
if true {
	var bar int
	bar, err = someOtherFunction() //err is not nill
}
if err != nil {
	fmt.Printf("we had an error")
}
```
(this prints we had an error)

---
**Pointers** Pointers are always a tripping hazard for new developers, and Go is no exception. Instead of re-inventing that blog, I recommend you just go read this: https://www.golang-book.com/books/intro/8

---
**The Go compiler is more picky about style than other languages you may have worked with**: I don't have much concrete advice here, but it's something that bugged me when I started. If you're someone who likes your curly brackets ({) on the next line, that's a compile error, and you just have to learn to live with it. Fortunately, there's an automatic formatter called "fmt" that fixes everything for you. On the commandline, run: _go fmt_ and it will fix the formatting in all of your go files.

---
**Same-line statements are possible but they can get confusing for scope reasons**: Go allows you to put multiple statements on the same line separated by semi-colons. I haven't seen this often, but there are a couple of cases where I do. You can use it to check to see if something is in a map:
```go
myMap := make(map[string]string)
myMap["foo"] = "bar"
if value, ok := myMap["baz"]; !ok {
	// the "truthiness" of this if statement is in the last (right-most) statement
	fmt.Printf("baz does not seem to be in myMap\n")
} else {
	fmt.Printf("we found baz, its value is %v\n", value)
}
// be aware that value is not defined in this scope, outside the if/else block
```
And this will check to see if an operation returned an error:
```go
if result, err := someFunction(); err != nil {
	fmt.Printf("had a problem running someFunction. It returned a value of %v and an error, %v\n", result, err.Error())
}
// be aware that because result is declared inside an if-statement, it is not in scope outside the if-block.
```
---
**Postfix Increments**: They're statements, not operators as they are in other languages. fmt.Printf("%v\n", foo++) doesn't work.
---
