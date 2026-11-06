# Methods and Interfaces

Notes created by following the corresponding section of the official Go tour

## Methods

Go has no classes, but it's possible to define methods on types

A method is just a function with a special receiver argument

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

// Abs() has a receiver argument v of type Vertex
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}
```

Methods can be declared on non-struct types as well

```go
type MyFloat float64

func (f MyFloat) foo() {
	...
}
```
> [!note]
> It is only possible to define a method with a receiver that
> is defined in the same package as the method. It is not possible
> to define a method whose type is defined in another package
> (such as built-in types like `int`)

### Pointer Receivers

Methods with pointer receivers can be defined by using
the literal `*T` syntax, for some type `T`. Note that `T` itself cannot
be a pointer (eg. `*int`)

Methods with pointer receivers can modify the value to which the
receiver points

Methods with value receivers operate on a copy of the original value
(this is the same behaviour as if it were a normal function argument)

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
	v.X *= f
	v.Y *= f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}
```

### Methods and Pointer Indirection

Functions with a pointer argument must take a pointer

```go
func foo(v *Vertex, n int) {...}

var v Vertex

foo(v, 4) // compiler error
foo(&v, 4) // ok
```

Methods with pointer receivers take either a value or a pointer

```go
func (v *Vertex) foo(n int) {...}

var v Vertex

v.foo(4) // ok
p := &v
p.foo(4) // ok

```

As a convenience, Go interprets `v.foo(4)` as `(&v).foo(4)` since
foo has a pointer receiver

The same thing happens for functions that take value arguments
and methods with value receivers

```go
func foo(v Vertex) {...}

var v Vertex
foo(v)  // ok
foo(&v) // compiler error

//////////////////////////////

func (v Vertex) bar() {...}

var u Vertex
u.bar() // ok
p := &u
p.bar() // ok
```

In this case, `p.bar()` is interpreted as `(*p).bar()`

### Value or Pointer Receiver?

Use pointer receivers when modifying the value its receiver
points to, or for efficiency
(eg. avoiding the copying of a large struct on each method call)

Generally, all methods on a given type should have either value
or pointer receivers, but not a mix of both


## Interfaces

An interface type is defined as a set of method signatures

```go
package main

import (
	"fmt"
	"math"
)

type Abser interface {
	Abs() float64
}

func main() {
	var a Abser
	f := MyFloat(-math.Sqrt2)
	v := Vertex{3, 4}

	a = f  // a MyFloat implements Abser
	a = &v // a *Vertex implements Abser

	a = v // COMPILE ERROR
		// v is type Vertex, not *Vertex
		// and does not implement Abser
	
	fmt.Println(a.Abs())
}

type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

Interfaces are implemented implicitly

A type implements an interface by implementing its methods.
There is no explicit declaration of intent by using a keyword
like `implements`

Implicit interfaces decouple the definition of an interface
from its implementation, which could then appear in any
package without prearrangement

```go
package main

import "fmt"

type I interface {
	M()
}

type T struct {
	S string
}

// this method means type T implements interface I
func (t T) M() {
	fmt.Println(t.S)
}

func main() {
	var i I = T{"hello"}
	i.M()
}
```

#### Interface Values

Interface values under the hood can be thought of as a tuple

`(value, type)`

An interface value holds a value of a specific underlying concrete type

Calling a method on an interface value executes the method of the same
name on its underlying type

```go
package main

import (
	"fmt"
	"math"
)

type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	fmt.Println(t.S)
}

type F float64

func (f F) M() {
	fmt.Println(f)
}

func main() {
	var i I

	i = &T{"hello"}
	describe(i)
	i.M()

	i = F(math.Pi)
	describe(i)
	i.M()
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```

If the concrete value inside the interface itself is `nil`,
the method will be called with a nil receiver

In some languages this causes a null pointer exception,
but methods in Go are commonly written to handle nil receivers gracefully

```go
func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}
```

> [!note]
> an interface value that holds a nil concrete value is itself non-nil

##### Nil Interface Values

A nil interface value holds neither a value nor concrete type

Calling a method on a nil interface is a run-time error as there
is no type inside the interface tuple to indicate which concrete
method to call

```go
package main

import "fmt"

type I interface {
	M()
}

func main() {
	var i I
	describe(i)
	i.M() // RUNTIME ERROR
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```

#### Empty Interfaces

The empty interface specifies zero methods:

`interface{}`

An empty interface may hold values of any type (every type
implements at least 0 methods)

`any` is an alias for `interface{}`

Empty interfaces are useful for handling values of unknown type.
For example, `fmt.Print` takes any number of arguments of type `any`

```go
package main

import "fmt"

func main() {
	var i interface{}
	describe(i)
	// (<nil>, <nil>)

	i = 42
	describe(i)
	// (42, int)

	i = "hello"
	describe(i)
	// (hello, string)
}

func describe(i interface{}) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```

#### Type Assertions

Type assertions provide access to an interface's underlying concrete value

`t := i.(T)`

This statement asserts that the interface value `i` holds the
concrete type `T` and assigns the underlying `T` value to the
variable `t`

> [!note]
> if `i` does not hold a `T`, the statement will trigger a panic

A type assertion returns two values, which allows for testing whether
an interface holds a specific type:

- the underlying value
- a boolean value reporting whether or not the assertion succeeded

`t, ok := i.(T)`

If `i` holds a `T`, then `t` will be the underlying value and `ok`
will be true

Otherwise, `ok` will be false and `t` will be the zero value of type
`T`, and no panic occurs

#### Type Switches

A type switch is a construct that allows for a series of type assertions

```go
switch v := i.(type) {
case T:
	// v has type T
case S:
	// v has type S
default:
	// no match; v has same type as i
}
```

#### Stringers

A `Stringer` is a type that can describe itself as a string.
The `fmt` package (among others) looks for this interface to
print values

```go
type Stringer interface {
	String() string
}
```

Example:

```go
package main

import "fmt"

type Person struct {
	Name string
	Age int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
	p := Person{Name: "Bob", Age: 52}
	z := Person{"Zaphod Beeblebrox", 9001}

	fmt.Println(p, z)
	// Bob (52 years) Zaphod Beeblebrox (9001 years)
}
```

#### Errors

Go programs express error state with `error` values

The `error` type is a built-in interface like `Stringer`.
As with `fmt.Stringer`, the `fmt` package looks for the `error`
interface when printing values

```go
type error interface {
	Error() string
}
```

It is common for functions to return an error value; calling
code should implement checks for whether or not the error
is `nil` as needed

```go
i, err := strconv.Atoi("42")
if err != nil {
	fmt.Printf("couldn't convert number: %v\n", err)
	return
}
fmt.Println("converted integer:", i)
```

A nil `error` denotes success; failure otherwise

#### Readers

The `io` package specifies the `io.Reader` interface, which represents
the read end of a stream

The standard Go library contains many implementations of this interface,
including files, network connections, compressors, ciphers, among others

The `io.Reader` interface has a `Read` method:

`func (T) Read(b []byte) (n int, err error)`

`Read` populates the given byte slice with data and returns the number
of bytes populated and an error value. An `io.EOF` error is returned
when the stream ends

Example:

```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")

	// consumes output of r 8 bytes at a time
	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

Rot13 Example:

[what is rot13](https://en.wikipedia.org/wiki/ROT13)

```go
package main

import (
	"io"
	"os"
	"strings"
)

type rot13Reader struct {
	r io.Reader
}

func (rr rot13Reader) Read(s []byte) (int, error) {

	n, err := rr.r.Read(s)

	for i := range s[:n] {
		s[i] = rot13(s[i])
	}

	return n, err
}

func rot13(b byte) byte {
	switch {
	case 'A' <= b && b <= 'Z':
		return 'A' + (b-'A'+13)%26
	case 'a' <= b && b <= 'z':
		return 'a' + (b-'a'+13)%26
	default:
		return b
	}
}

func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}
```

#### Images

Package
[image](https://go.dev/pkg/image/#Image)
defines the `Image` interface:

```go
package image

type Image interface {
	ColorModel() color.Model
	Bounds() Rectangle
	At(x, y int) color.Color
}
```

> [!note]
> the `Rectangle` return value of the `Bounds()` method is actually
> an `image.Rectangle`, as the declaration is inside package `image`

`color.Color` and `color.Model` types are also interfaces, see their specs
[here](https://go.dev/pkg/image/color/)

Example:

```go
package main

import (
	"fmt"
	"image"
)

func main() {
	m := image.NewRGBA(image.Rect(0, 0, 100, 100))
	fmt.Println(m.Bounds())
	fmt.Println(m.At(0, 0).RGBA())
}
```

Example 2:

```go
package main

import (
	"image"
	"image/color"
	"golang.org/x/tour/pic"
)

type MyImage struct {
	W, H int
}

func (mi MyImage) ColorModel() color.Model {
	return color.RGBAModel
}

func (mi MyImage) Bounds() image.Rectangle {
	return image.Rect(0, 0, mi.W, mi.H)
}

func (mi MyImage) At(x, y int) color.Color {
	v := uint8((x*x + 2*x*y + y*y) / (y + 1))
	return color.RGBA{v, v, 255, 255}
}

func main() {
	m := MyImage{250, 250}
	pic.ShowImage(m)
}
```


