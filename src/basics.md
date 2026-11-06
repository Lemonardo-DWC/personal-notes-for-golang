# Basics

Notes created by following the corresponding section of the official Go tour

## Overview

### Packages

Go programs are made up of packages

The `main` package is where programs start running

Conventionally, the package name is the same as the last element
of the import path (eg. the `"math/rand"` package contains files
that start with `package rand`)

### Imports

```go
import "fmt"
import "math"

import (
	"fmt"
	"math"
)
```

### Exports

Names are exported from packages if they begin with a capital letter

Only exported names of imported packages can be referred to

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	fmt.Println(math.Pi)
		// prints: 3.141592653589793
	fmt.Println(math.pi)
		// prints: ./prog.go:9:19: undefined: math.pi (but have Pi)
}
```

### Functions

Functions take 0 or more arguments

Types come after variable name (see
[Go's declaration syntax blog](https://go.dev/blog/gos-declaration-syntax)
for more information)

```go
func add(x int, y int) int {
	return x + y
}
```

When consecutive named function parameters share a type,
the type can be omitted until the last parameter

```go
// could use (x int, y int) instead, but it's a bit more verbose
func add(x, y int) int {
	return x + y
}
```

Functions can return any number of results

```go
package main

import "fmt"

// swaps two strings
func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b) // prints: world hello
}
```

Return values may be named, if so, they are treated as if
they were defined at the top of the function

```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}
```

A `return` statement without arguments returns the named return values.
AKA "naked" return

### Variables

`var` declares a list of variables, with the type being at the end

```go
package main

import "fmt"

var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java) // 0 false false false
}
```

`var` declarations can include initializers with one per variable

If an initializer is present, the type will be inferred if omitted.
The later short section,
[Type Inference](#type-inference),
talks about this in more detail

```go
package main

import "fmt"

var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "word"
	fmt.Println(i, j, c, python, java)
		// 1 2 true false word
}
```

Inside functions, the short assignment statement (`:=`) with implicit
type, can be used instead of a `var` declaration

```go
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "word"

	fmt.Println(i, j, k, c, python, java)
		// 1 2 3 true false word
}
```

Outside functions, every statement starts with keywords like `var`, `func`, etc.

#### Constants

Constants are declared like variables, but uses the `const` keyword

Constants can be character, string, boolean, or numeric values

Constants cannot be declared using the `:=` syntax

```go
package main

import "fmt"

const Pi = 3.14

func main() {
	const World = "世界"
	fmt.Println("Hello", World)
	fmt.Println("Happy", Pi, "Day")

	const Truth = true
	fmt.Println("Go Rules?", Truth)
}
```

### Basic Types

```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a unicode code point

float32 float64

complex64 complex128
```

> [!note]
> `int`, `uint`, and `uintptr` types are usually 32 bits wide on
> 32-bit systems and 64 bits wide on 64-bit systems. It is recommended
> to just use `int` unless there is a specific reason to use a sized
> or unsigned integer type

#### Zero Values

Variables declared without an explicit initial value are given
their zero values

```go
0       // for numeric types
false   // for boolean types
""      // for strings
nil     // for pointers, interfaces, slices, maps, channels, functions
```

#### Type Conversions

The expression `T(v)` converts the value `v` to type `T`

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)

// or

i := 42
f := float64(i)
u := uint(f)
```

Assignment between items of different types requires explicit conversions

Example:

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	var x, y int = 3, 4
	var f float64 = math.Sqrt(float64(x*x + y*y))
	var z uint = f // should be uint(f)
	fmt.Println(x, y, z)
}
```

Output:
```
./prog.go:11:15: cannot use f (variable of type float64) as uint value in variable declaration
```

#### Type Inference

When declaring a variable without specifying its type explicitly,
the variable's type is inferred from the value it is being assigned

When the right hand side is typed, the type of the new variable
will be the same type:

```go
var i int
j := i // j is of type int
```

When assigning an untyped numeric constant, the new variable may be
an `int`, `float64`, or `complex128` depending on the precision of
the constant.

```go
i := 42                 // int
f := 3.142              // float64
g := 0.867 + 0.5i       // complex128
```

### Flow Control

#### For

Go's only looping construct

`init` and `post` statements are optional

```go
// Syntax

for <init> ; <cond> ; <post> {...}

for ; <cond> ; {...}
for <cond> {...} // semicolons can be omitted if there's only a condition expression

// infinite loop
for {...}
```

Example:

```go
package main

import "fmt"

func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)

	////////////////////////////////

	prod := 1
	for ; prod < 100; {
		prod *= 2
	}
	fmt.Println(prod)

	////////////////////////////////

	diff := 1001
	for diff > 100 {
		diff -= 100
	}
	fmt.Println(diff)
}
```

#### If \[Else\]

Syntax:

```go
if <cond> {...}

// with short statement; scope is within if-block
if <statement>; <cond> {...}

// if-else, optionally with short statement; scope is within if-else-block
if ... {
	...
} else {
	...
}
```

#### Switch

`switch` provides a more concise way to write sequences of
`if - else` statements

Similar to C, C++, Java, JavaScript except Go's `switch` has
no fall-through behaviour

Switch cases don't need to be constants (can be expressions)

Values involved don't need to be integers

Evaluation order is top-down, stopping when a case succeeds

Syntax:

```go
switch <cond_var> {
case 1:
	...
case "2":
	...
case foo():
	...
default:
	...
}

// with short statement
switch <statement>; <cond_var> {
case 1:
	...
case "2":
	...
case foo():
	...
default:
	...
}

// without condition; same as 'switch true {...}'
switch {
case x < 2:
	...
case x == 2:
	...
case x > 2:
	...
default:
	...
}
```

#### Defer

Defers the execution of a function until the containing function returns

Deferred call's arguments are evaluated immediately, but the function call
is deferred

Syntax:

```go
defer f(a, b)
```

Deferred function calls are pushed onto a stack. When a function returns,
its deferred calls are executed in LIFO (last-in-first-out) order

[This blog post](https://go.dev/blog/defer-panic-and-recover)
talks about defer statements in more detail

```go
package main

import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 5; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")

	/*
		counting
		done
		4
		3
		2
		1
		0
	*/
}
```

### Pointers

The type `*T` is a pointer to a `T` value

Pointer default value is `nil`

The `&` operator generates a pointer to its operand

The `*` operator dereferences the pointer's underlying value

```go
var p *int
i := 42
p = &i

fmt.Println(*p) // read i through pointer p
*p = 21         // set i through pointer p
```

Go has no pointer arithmetic

### Structs

Structs are a collection of fields

```go
package main

import "fmt"

type Vertex struct {
	X int
	Y int
}

func main() {
	fmt.Println(Vertex{1, 2}) // prints: {1 2}
}
```

Struct fields are accessed with the `.` operator

```go
v := Vertex{1, 2}
v.X = 4

fmt.Println(v) // prints: {4 2}
```

Struct fields can also be accessed through a struct pointer

`(*p).X`

Go allows for implicit dereference

`p.X`

Example:

```go
package main

import "fmt"

type Vertex struct {
	X int
	Y int
}

func main() {
	x := Vertex{1, 2}
	p := &x
	p.X = 1e9
	fmt.Println(x) // prints: {1000000000 2}
}
```

#### Struct Literals

A struct literal denotes a newly allocated struct value by listing the
values of its fields

It is possible to only list a subset of fields using the `Name: value` syntax;
order is irrelevant when using this syntax

```go
package main

import "fmt"

type Vertex struct {
	X, Y int
}

var(
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)

func main() {
	fmt.Println(v1, p, v2, v3) // prints: {1 2} &{1 2} {1 0} {0 0}
}
```

### Arrays / Slices

The type `[n]T` is an array of `n` values that are type `T`

Array lengths are part of its type, so they cannot be resized.

```go
// an array of 10 ints
var a [10]int
```

Slices are dynamically sized, unlike arrays. Slices are much more
common than arrays

The type `[]T` is a slice with elements of type `T`

A slice is formed by specifying 2 indices, a low and high bound, colon-separated:

```go
a[low : high]
```

This selects a half open range: `[low, high)`

Slices do not store any data themselves, but instead describe a section
of some underlying array

Thus, changing the elements of a slice changes the corresponding elements
of the underlying array

Other slices sharing the same underlying array sees those changes

```go
package main

import "fmt"

func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)
	// [John Paul George Ringo]

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)
	// [John Paul] [Paul George]

	b[0] = "XXX"
	fmt.Println(a, b)
	// [John XXX] [XXX George]

	fmt.Println(names)
	// [John XXX George Ringo]

}
```

A slice literal is just an array literal without the length

```
// array literal
[3]bool{true, true, false}

// slice literal
// creates the same array as above, then builds a slice that references it
[]bool{true, true, false}
```

Example:

```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3}
	// [1 2 3]

	r := []bool{true, false, true}
	// [true false true]

	s := []struct{
		i int
		b bool
	}{
		{2, true},
		{3, false},
		{4, true},
	}
	// [{2 true} {3 false} {4 true}]
}
```

When slicing, low or high bounds may be omitted to use their defaults

Slice default values are 0 for the low bound, and the length of the
underlying slice or array for the high bound

```go
var a [10]int

// the following are equivalent
a[0:10]
a[:10]
a[0:]
a[:]
```

Slices have a `length` and `capacity`

The length is the number of elements it contains

The capacity is the number of elements in the underlying array, counting from
the first element in the slice - in other words, the maximum possible length
of the current slice

The length and capacity of a slice `s` can be obtained by: `len(s)` and `cap(s)`

Slice lengths can be extended, provided it has sufficient capacity

```go
package main

import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlices(s)

	// slice the slice to give it zero length
	s = s[:0]
	printSlices(s)

	// extend its length
	s = s[:4]
	printSlices(s)

	// drop first 2 values
	s = s[2:]
	printSlices(s)
}

func printSlices(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

The zero value of a slice is `nil`

A nil slice has a length and capacity of 0, with no underlying array

```go
package main

import "fmt"

func main() {
	var s []int

	fmt.Println(s, len(s), cap(s))
	if s == nil {
		fmt.Println("nil!")
	}

	/*
		[] 0 0
		nil!
	*/
}
```

#### Making Slices

Slices can be created with the built-in `make()` function; this is how
dynamically-sized arrays are created

The `make()` function allocates a zeroed array and returns a slice that
refers to that array

```go
a := make([]int, 5) // len(a) = 5

// to specify a capacity
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

Slices can contain any type, including other slices

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	// tic-tac-toe board
	board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

	// players take turns
	board[0][0] = "X"
	board[2][2] = "O"
	board[1][2] = "X"
	board[1][0] = "O"
	board[0][2] = "X"

	for i := 0; i < len(board); i++ {
		fmt.Printf("%s\n", strings.Join(board[i], " "))
	}

	/*
		X _ X
		O _ X
		_ _ O
	*/
}
```

#### Appending to Slices

New elements can be appended to slices using the `append()` function - check
the documentation of the built-in package
[here](https://go.dev/pkg/builtin/#append)
for more details

```go
func append(s []T, vs ...T) []T
```

`s` is the slice to append to, the rest are `T` values to append to the slice

The resulting value of `append()` is a slice containing all elements of the original
slice, followed by the additionally provided values

If the backing array of `s` is too small, a larger array will be allocated.
The returned slice will point to the newly allocated array


### Range

The `range` form of the `for` loop iterates over a slice or map

When ranging over a slice, two values are returned each iteration:
an index, and a copy of the element at that index

```go
package main

import "fmt"

var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}
```

The index or the range value can be skipped by using `_` instead

```go
for i, _ := range a
for _, v := range a

// the value variable can be omitted if only the index is needed
for i := range a
```

Example:

```go
package main

import "fmt"

func main() {
	pow := make([]int, 10)
	for i := range pow {
		pow[i] = 1 << uint(i) // == 2**i
	}
	for _, value := range pow {
		fmt.Printf("%d\n", value)
	}
}
```

### Maps

Maps map keys to values (key-value pairs)

Zero value of a map is `nil`. A `nil` map has no keys, nor can keys be added

The `make()` function returns a map of the given type, initialized and ready
for use

```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex // a map that maps string to Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}

	fmt.Println(m["Bell Labs"])
	// {40.68433 -74.39967}
}
```

Map literals are like struct literals, but keys are required

```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}

func main() {
	fmt.Println(m)
	// map[Bell Labs:{40.68433 -74.39967} Google:{37.42202 -122.08408}]
}
```

If the top-level type is just a type name, it may be omitted from the elements of the literal

```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex {
	"Bell Labs": {40.68433, -74.39967},
	"Google": {37.42202, -122.08408},
}
```

#### Map Operations

```go
// insert/update
m[key] = elem

// retrieve
elem = m[key]

// delete
delete(m, key)

// test for existing key
elem, ok = m[key]
	// ok is true if key already in m, otherwise false
	// if the key is not in the map, elem is the zero value for the map's element type

// if elem or ok hasn't been declared yet
elem, ok := m[key]
```


### Functions++

Functions are values too, and thus can be passed around

Function values may be used as arguments and return values of other functions

```go
package main

import (
	"fmt"
	"math"
)

// compute() takes a function that operates on floats to return a float
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {

	// hypot holds a function that takes 2 floats and returns a float
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}

	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```

#### Closures

Go functions may be closures

A closure is a function value that references variables from outside its body.

The function may access and assign to the referenced variables (the function
is "bound" to the variables)

Example:

```go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()

	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}

/* output:
	0 0
	1 -2
	3 -6
	6 -12
	10 -20
	15 -30
	21 -42
	28 -56
	36 -72
	45 -90
*/
```

Fibonacci Example:

```go
package main

import "fmt"

func fibonacci() func() int {
	previous, current := 0, 1
	return func() int {
		temp := previous
		previous = current
		current += temp
		return temp
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```

Fibonacci Example (using defer):

```go
package main

import "fmt"

func fibonacci() func() int {
	previous, current := 0, 1
	return func() int {
		defer func() {
			previous, current = current, current+previous
		}() // anonymous function call
		return previous
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```

## More Resources

- [Declaration Syntax](https://go.dev/blog/gos-declaration-syntax)
- [More Slices](https://go.dev/blog/slices-intro)
