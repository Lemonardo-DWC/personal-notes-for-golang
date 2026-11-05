# Generics

## Type Parameters

Go functions can work on multiple types by using type parameters

Example:

`func Index[T comparable](s []T, x T) int`

This declaration means `s` is a slice of any type `T` that fulfills
the built-in constraint `comparable`. `x` is also of the same type

`comparable` is a useful constraint that makes it possible to use
comparison operators like `==` and `!=` on values of this type

Example:

```go
package main

import "fmt"

// return index of x in s; -1 if not found
func Index[T comparable](s []T, x T) int {
	for i, v := range s {
		if v == x {
			return i
		}
	}

	return -1
}

func main() {
	si := []int{10, 20, 15, -10}
	fmt.Println(Index(si, 15))
	// 2

	ss := []string{"foo", "bar", "baz"}
	fmt.Println(Index(ss, "hello"))
	// -1
}
```

## Generic Types

Go also has generic types. A type can be parameterized with a type
parameter, which can be useful for implementing generic data structures

Example:

```go
package main

import (
	"fmt"
	"strings"
)

// List represents a singly-linked list that holds values of any type
type List[T any] struct {
	next *List[T]
	val  T
}

// random struct for our linked list
type Blah struct {
	s  string
	ia []int
}

func main() {

	// create linked list of 10 items
	head := List[Blah]{next: nil, val: Blah{"first", []int{0, 0, 0}}}
	for i := 1; i < 10; i++ {
		head.Push(Blah{"later", []int{i, i + 1, i + 2}})
	}

	head.Each(func(b *Blah) {
		b.s = "changed"
		b.ia[0] = 999
	})

	fmt.Println(&head)

	fmt.Println(head.Len())
}

// append to end of list
func (l *List[T]) Push(v T) {
	p := l
	for p.next != nil {
		p = p.next
	}

	p.next = &List[T]{next: nil, val: v}
}

// return length of list
func (l *List[T]) Len() int {
	length := 0

	if l != nil {
		length = 1
	} else {
		return 0
	}

	for p := l; p.next != nil; p = p.next {
		length++
	}

	return length
}

// string representation of whole list
func (l *List[T]) String() string {
	var sb strings.Builder

	for p := l; p != nil; p = p.next {
		fmt.Fprintf(&sb, "%v\n", p.val)
	}

	return sb.String()
}

// apply a function on each item in the list
func (l *List[T]) Each(f func(*T)) {
	for p := l; p != nil; p = p.next {
		f(&p.val)
	}
}
```
