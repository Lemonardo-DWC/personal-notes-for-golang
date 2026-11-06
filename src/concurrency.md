# Concurrency

Notes created by following the corresponding section of the official Go tour

## Goroutines

A goroutine is a lightweight thread managed by the Go runtime

```go
// starts a new goroutine running foo(x, y, z)
go foo(x, y, z)
```

`foo`, `x`, `y`, and `z` are evaluated in the current goroutine,
but the execution of `foo` happens in the new goroutine

Goroutines run in the same address space, thus access to shared
memory must be synchronized

The `sync` package provides useful primitives, although there are
other primitives that are often sufficient (next section)

## Channels

Channels are a typed conduit through which values can be sent and
received with the channel operator: `<-`

The direction of the arrow indicates the data flow

```go
ch <- v     // send v to channel ch
v := <- ch  // receive from ch and assign value to v
```

Like maps and slices, channels must be created before use:

`ch := make(chan int)`

By default, sends and receives block until the other side is ready,
which is nice for synchronizing goroutines without explicit locks
or condition variables

Example:

```go
// sums up numbers in a slice, distributing work between two goroutines

package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c

	fmt.Println(x, y, x+y)
}
```

### Buffered Channels

Channels can be buffered by providing a second argument as the
buffer length

`ch := make(chan int, 100)`

Sends to a buffered channel block only when the buffer is full

Receives block when the buffer is empty

Example:

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

> [!tip]
> Overfilling the buffer in this example causes a fatal error
> where all goroutines are asleep (deadlock)

### Range and Close

A sender can `close` a channel to indicate that no more values
will be sent

Receivers can test whether a channel has been closed by assigning
a second parameter to the receive expression

`v, ok := <-ch`

`ok` is `false` if there are no more values to receive and the channel
is closed

The loop `for i := range c` receives values from the channel
repeatedly until it's closed

Example:

```go
package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```

> [!note]
> Channels should only ever be closed by senders, and never
> the receiver. Sending on a closed channel will cause a panic

> [!note]
> Unlike files, channels don't usually need to be closed.
> It is only necessary when the receiver must be notified
> about no more incoming values, such as to terminate a
> `range` loop

### Select

The `select` statement lets a goroutine wait on multiple
communication operations

A select blocks until one of its cases can run, then it
executes that case. It chooses one at random if multiple
are ready

```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()

	fibonacci(c, quit)
}
```

#### Default Selection

The `default` case in a `select` runs if no other case is ready

Useful for trying a send or receive without blocking

```go
select {
case i := <-c:
	// use i
default:
	// receiving from c would block
}
```

Example:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	start := time.Now()
	tick := time.Tick(100 * time.Millisecond)
	boom := time.After(500 * time.Millisecond)
	elapsed := func() time.Duration {
		return time.Since(start).Round(time.Millisecond)
	}

	for {
		select {
		case <-tick:
			fmt.Printf("[%6s] tick.\n", elapsed())
		case <-boom:
			fmt.Printf("[%6s] BOOM!\n", elapsed())
			return
		default:
			fmt.Printf("[%6s]     .\n", elapsed())
			time.Sleep(50 * time.Millisecond)
		}
	}
}
```

### Exercise Examples

#### Equivalent binary trees

Example implementation of checking if two binary trees store
the same sequence. See docs for `Tree`
[here](https://godoc.org/golang.org/x/tour/tree#Tree)

```go
package main

import (
	"fmt"
	"golang.org/x/tour/tree" // just holds a left, right, and node value (int in this case)
)

// Performs an in-order traversal of tree t, sending
// all values from the tree to the channel ch.
func Walk(t *tree.Tree, ch chan int) {
	walker(t, ch)
	close(ch)
}

// helper for in-order traversal of the tree
func walker(t *tree.Tree, ch chan int) {
	if t == nil {
		return
	}

	walker(t.Left, ch)
	ch <- t.Value

	walker(t.Right, ch)
}

// Same determines whether the trees
// t1 and t2 contain the same values.
func Same(t1, t2 *tree.Tree) bool {

	ch1, ch2 := make(chan int), make(chan int)

	go Walk(t1, ch1)
	go Walk(t2, ch2)

	// checks sequences in 'parallel'
	for {
		node1, ok1 := <-ch1
		node2, ok2 := <-ch2

		// trees of different size or sequence
		if ok1 != ok2 || node1 != node2 {
			return false
		}

		// finished, all good
		if !ok1 {
			return true
		}
	}
}

func main() {
	fmt.Println(Same(tree.New(1), tree.New(1))) // true
	fmt.Println(Same(tree.New(1), tree.New(2))) // false
}
```

### sync.Mutex

Channels are good for communication among goroutines,
but there are cases where we only need to make sure
only one goroutine accesses a variable at a time to avoid
conflicts.

This concept is known as 'mutual exclusion', the conventional
name for the data structure that provides it is 'mutex'

Go provides mutual exclusion in its standard library with
`sync.Mutex` and methods: `Lock`, `Unlock`

A block of code can be designated to be executed in mutual
exclusion by surrounding it with a `Lock` and `Unlock` call

`defer` can additionally be used to help ensure unlocking
of the mutex

Example:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mu.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mu.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second) // just to make sure counter goes to 1000
	fmt.Println(c.Value("somekey"))
}
```


