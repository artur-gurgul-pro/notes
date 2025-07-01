---
layout: default
title: Bridging between languages
categories: programming
---
We can cross call functions from languages that:

- Both are compilable &rarr; Strategy: Create a binary library and call function from it.
- Library in `interpreted` and the project in `compiled` &rarr; Strategy: Create an interpreter context, then load the files and execute them.
- Library in `compiled` and the project in `interpreted` &rarr; Strategy: Use `FFI`.
- Both are interpreted
	- Probably give up. Bridging will be possible, but messy and complicated
	- Create two programs and exchange data between them using pipe, sockets, message quesues, databses etc.

### Creating shared and static library in Go

An example code that is shared [`example.go`](https://github.com/artur-gurgul/codebook):

``` go
package main

import "C"
import "fmt"

//export SayHello
func SayHello(hello *C.char) {
		fmt.Print(C.GoString(hello))
}

func main() {}
```

- The `main` function is neccecery to include into library, because the final product has to have for example the GC rutines.
- The comment starting from `//export {function name}` tells the comiler that this the function will be called from the outside.

#### Creating static library

```
go build -o example.a -buildmode=c-archive example.go
```

#### Creating dynamic library

```
go build -o example.dylib -buildmode=c-shared example.go
```

### Creating shared and static library in Swift

`point.swift`
```swift point.swift
public struct Point {
    public let x: Int
    public let y: Int

    public init(x: Int, y: Int) {
        self.x = x
        self.y = y
    }
}
```

and compile with command (module name is optional)

```sh
swiftc point.swift -emit-module  -module-name Point -emit-library -static
```

**Using**

```swift
import Point

let p = Point(x: 4, y: 20)
print("Hello library!", p.x, p.y)
```

compile with

```sh
swiftc main.swift -L ./lib/ -I ./lib/ -lpoint
```

#### Dynamic library in Swift

```
swiftc point.swift -emit-module -emit-library
```

it produces 

- `libpoint.a` 
- `point.swiftdoc` 
- `point.swiftmodule`
- `point.swiftsourceinfo`

Compile main program the same way as it has been down with the static one

Library searching paths `/usr/lib/`, `/usr/local/lib/` 

**_Create package that emits library_**

```swift
// swift-tools-version:5.3
import PackageDescription

let package = Package(
    name: "MyLibrary",
    products: [
        /// type: automatic, based on the environment
        .library(name: "MyLibrary", 
		         // type: .dynamic, .static
		         targets: ["MyLibrary"]
        ),
    ],
    targets: [
        .target(name: "MyLibrary", dependencies: []),
    ]
)
```

### Calling function from library in Go

First off we will create C++ library that we will use in out Go program.
File `example.cxx`:

```c++
#include <stdio.h>

extern "C" {

void PrintHello(const char* u) {
    printf("Hello: %s\n", u);
}

}
```

And `example.hxx`:

```c++
#pragma once
void PrintHello(const char* u)
```

`extern "C" {}` informs the compiler that we want the function names to be preserved. That is, to not "mangle" the names as is done for C++ code:

#### Creating static library

```bash
clang++ -c -Wall -o lib.o ./example.cxx
ar rc ./libexample.a ./lib.o
```

#### Creating dynamic library

```bash
clang++ -dynamiclib -o libexample.dylib example.cxx
```

## Statically linking an example library in Go

```go
package main

// #cgo CFLAGS: -I.
// #cgo LDFLAGS: -L. -lexample
//
// #include <example.hxx>
import "C"

func main() {
	C.PrintHello(C.CString("Hello Golang"))
}
```

The program is linked staticaly with libexample when you build it.

#### Example of using library with FFI in Ruby

```shell
gem install ffi
```

```ruby
require 'ffi'
module Example
  extend FFI::Library
  ffi_lib './example.dylib'
  attach_function :SayHello, [:string]
end
```

```ruby
Example.SayHello("Hello")
```

More informations about FFI: [https://en.wikipedia.org/wiki/Foreign_function_interface](https://en.wikipedia.org/wiki/Foreign_function_interface)

#### Call shared library from Python

```python
import ctypes
libc = ctypes.CDLL('./example.dylib')
libc.SayHello("Hello")
```

## Interesting websites

- [https://blog.filippo.io/building-python-modules-with-go-1-5/](https://blog.filippo.io/building-python-modules-with-go-1-5/)
- [https://id-rsa.pub/post/go15-calling-go-shared-libs-from-firefox-addon/](https://id-rsa.pub/post/go15-calling-go-shared-libs-from-firefox-addon/)