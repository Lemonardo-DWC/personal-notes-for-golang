# Modules

Modules collect a set of related packages for a discrete and useful
set of functions

A basic module structure looks like this:

```
module
  ├ package 1
  │    ├ subpackage 1.1
  │    └ subpackage 1.2
  ├ package 2
  └ package 3
```

A module specifies dependencies needed to run its code,
including the version of Go and the set of other modules
it might require

## Creating a Module

Start the module with:

`go mod init example.com/module-name`

Command specs
[here](https://go.dev/ref/mod#go-mod-init),
or run `go help mod`, `go help mod init`

## Calling Code From Another Module

Import a package by its path: the module path, plus the
directory for packages outside the module root

```go
import "example.com/mymodule"         // package in the module's root directory
import "example.com/mymodule/myutil"  // package in the myutil/ subdirectory
```

Refer to an imported package by its package name (from its `package`
clause, which conventionally matches the last element of the path)

Example: `myutil.Reverse("hello")`

> [!note]
> If you're using a local module, point the dependency at its
> directory in go.mod
>
> `go mod edit -replace example.com/mymodule=/path/to/local/module`

Run `go mod tidy` to synchronize the current module's dependencies
(after a `-replace`, this also adds the matching `require` line)

## Testing

Files ending in `_test.go` indicate to `go test` that
it is a test file for the containing package

By convention, a file named `<myfile>_test.go` indicates
that it is a test file for `myfile.go`

See more on Go's testing package
[here](https://pkg.go.dev/testing)

Example test function:

```go
// test function for Foo()
func TestFoo(t *testing.T) {
	isOk := Foo(...)

	if !isOk {
		t.Errorf("error message")
	}
}
```

To run a test for a package, first `cd` to the package
directory, then run `go test`

## Compiling and Installing the Application

> `go build` - compiles the packages in addition to any dependencies
>
> `go install` - compiles and installs the main package to your Go install path

Run `go help build` and `go help install` to see more info on them

Run `go env GOBIN` to see your current install directory

You can change the install directory by using `go env -w GOBIN=/some/path/for/your/bin`

## Read Also

- [Managing Dependencies](https://go.dev/doc/modules/managing-dependencies)
- [Developing and Publishing Modules](https://go.dev/doc/modules/developing)
