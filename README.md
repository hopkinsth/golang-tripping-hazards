### Golang tripping hazards
---
Any time I learn something new, I always end up finding a few things that I really wish someone had told me at the beginning. If you're starting to learn Go, perhaps you'll find these tidbits useful.

1. **The defer statement**: A reserved word in Go that causes a function to execute as the last thing that happens when a function returns. You can stack as many as you want and they execute in the reverse order that they were declared. It looks something like this:
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
