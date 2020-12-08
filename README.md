# golang-studygroup

## Code Packages and Package Imports
To use the exported resources (functions, types, variables and named constants, etc) in a specified package, the package must first be imported, except the [ builtin standard code package](https://golang.org/pkg/builtin/).

<br />

## Introduction of Package Import

### Sample 1

```golang
package main

import "fmt"

func main() {
    fmt.Println("Go has", 25, "keywords.")
}
```
* The main entry function of a program must be put in a package named main.
* The third line imports the fmt standard package by using the import is a keyword. The identifier fmt is the package name.
*  The form aImportName.AnExportedIdentifier is called a qualified identifier. AnExportedIdentifier is called an unqualified identifier.
* only [exported](https://go101.org/article/keywords-and-identifiers.html#identifier) resources in a package can be used in the source file which imports the package.

</br>**Note**</br>
Two built-in functions, print and println, are not recommended to be used in the production environment, for they are not guaranteed to stay in the future Go versions.
</br></br>


### Sample 2

```golang
package main

import "fmt"
import "math/rand"

func main() {
	fmt.Printf("Next random number is %v.\n", rand.Uint32())
}
```
* The `math/rand` package, which is a sub-package of the math standard package.
* `Printf` function must take at least one argument.
</br></br>

### Sample 3
```golang
package main

// Multiple packages can be imported together.
import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	 // Set the random seed.
	rand.Seed(time.Now().UnixNano())
	fmt.Printf("Next random number is %v.\n", rand.Uint32())
}
```
* If multiple packages are imported into a source file, we can group them in one import declaration by enclosing them in a `()`.

<br/>

## More About fmt.Printf Format Verbs
In Go 101, only the following listed format verbs will be used.
* %v, which will be replaced with the general string representation of the corresponding argument.
* %T, which will be replaced with the type name or type literal of the corresponding argument.
* %x, which will be replaced with the hex string representation of the corresponding argument. Note, the hex string representations for values of some kinds of types are not defined. Generally, the corresponding arguments of %x should be strings, integers, integer arrays or integer slices (arrays and slices will be explained in a later article).
* %s, which will be replaced with the string representation of the corresponding argument. The corresponding argument should be a string or byte slice.
* Format verb %% represents a percent sign.
### Sample 4
```golang
package main

import "fmt"

func main() {
	a, b := 123, "Go"
	fmt.Printf("a == %v == 0x%x, b == %s\n", a, a, b)
	fmt.Printf("type of a: %T, type of b: %T\n", a, b)
	fmt.Printf("1%% 50%% 99%%\n")
}
```
Result will be:
```
a == 123 == 0x7b, b == Go
type of a: int, type of b: string
1% 50% 99%
```
<br/>

## Package Folder, Package Import Path and Package Dependencies
a package whose import path containing an internal folder name is viewed as a special package. It can only be imported by the packages in and under the direct parent directory of the internal folder. For example, package `.../a/b/c/internal/d/e/f` and `.../a/b/c/internal` can only be imported by the packages whose import paths have a `.../a/b/c` prefix.
<br/>

### Module
A module can be viewed as a collection of packages which have a common root (a package tree). Each module is associated with an root import path and a semantic version.
* 1.11 introduced a `GO111MODULE` environment var to control if module feature is on/off

```
_ GOPATH
  |_ src
     |_ x
        |_ vendor
        |  |_ w
        |     |_ foo
        |        |_ foo.go    // package foo
        |_ y
        |  |_ vendor
        |  |  |_ w
        |  |     |_ foo
        |  |        |_ foo.go // package foo
        |  |_ y.go            // package y
        |_ z
        |  |_ z.go            // package z
        |_ x.go               // package x
```
For example, when the modules feature is off, then for the following hierarchical directory structure,
* the import paths of the two foo packages are both w/foo.
* the import paths of the x, y and z packages are x, x/y and x/z, respectively.
<br/>

**Note**
* when the file y.go imports a package with import path as w/foo, the imported package is the package with folder GOPATH/src/x/y/vendor/w/foo.
* when the x.go or z.go file imports a package with import path w/foo, the imported package is the package with folder GOPATH/src/x/vendor/w/foo.
<br/>

When the modules feature is on, the root import path of a module is often (but not required to be) specified in a go.mod file which is directly contained in the root package folder of the module. We often use the root import path to identify the module. The root import path is the common prefix of all packages in the module.

```
_ MyProject
     |_ go.mod                // module example.com/mypkg
     |_ vendor
     |  |_ w
     |     |_ foo
     |        |_ foo.go       // package foo
     |_ x
        |_ y
        |  |_ vendor
        |  |  |_ w
        |  |     |_ foo
        |  |        |_ foo.go // package foo
        |  |_ y.go            // package y
        |_ z
        |  |_ z.go            // package z
        |_ x.go               // package x
```
For example, when the modules feature is on, then in the module identified with `example.com/mypkg` shown blow,
* the import path of the first foo package is w/foo. The MyProject/vendor folder is viewed as a special folder.
* the import path of the other foo package is`example.com/mypkg/x/y/vendor/w/foo`. Note, the MyProject/x/y/vendor folder is viewed as a normal package folder.
* the import paths of the x, y and z packages are example.com/mypkg/x, example.com/mypkg/x/y and example.com/mypkg/x/z, respectively.
<br/>

**Note**
* when the x.go, y.go or z.go files import a package with import path w/foo, the imported package is always the package with folder MyProject/vendor/w/foo.
<br/><br/>

### Go doesn't support circular package dependencies. 
If package a depends on package b and package b depends on package c, then source files in package c can't import package a and b, and source files in package b can't import package a.

## The init Functions

```golang
package main

import "fmt"

func init() {
	fmt.Println("hi,", bob)
}

func main() {
	fmt.Println("bye")
}

func init() {
	fmt.Println("hello,", smith)
}

func titledName(who string) string {
	return "Mr. " + who
}

var bob, smith = titledName("Bob"), titledName("Smith")
```
Output
```
hi, Mr. Bob
hello, Mr. Smith
bye
```
## Resource Initialization Order
At run time, a package will be loaded after all its dependency packages. Each package will be loaded once and only once.

* All init functions in all involved packages in a program will be invoked sequentially. 
* An init function in an importing package will be invoked after all the init functions declared in the dependency packages of the importing package for sure. 
* All init functions will be invoked before invoking the main entry function.
* The invocation order of the init functions in the same source file is from top to bottom. 
* Go specification recommends, but doesn't require, to invoke the init functions in different source files of the same package by the alphabetical order of filenames of their containing source files. 

## Full Package Import Forms
```golang
// import importname "path/to/package"
import fmt "fmt"        // <=> import "fmt"
import rand "math/rand" // <=> import "math/rand"
import time "time"      // <=> import "time"
```

The full import declaration form is not used widely. However, sometimes we must use it. 
* For example, if a source file imports two packages with the same name, to avoid making compiler confused, we must use the full import form to set a custom importname for at least one package in the two.


```golang
package main

import (
	. "fmt"
	. "time"
)

func main() {
	Println("Current time:", Now())
}
```
***Generally, dot imports are not recommended to be used in formal projects.***

The importname in the full form import declaration can be a dot (.). Such imports are called dot imports. To use the exported elements in the packages being dot imported, the prefix part in qualified identifiers must be omitted.

<br/>

```golang
package main

import _ "net/http/pprof"

func main() {
	... // do somethings
}
```

* The importname in the full form import declaration can be the blank identifier (_). 
* Such imports are called anonymous imports (some articles elsewhere also call them blank imports). 
* The importing source files `can't use the exported resources` in anonymously imported packages. The purpose of anonymous imports is to initialize the imported packages (each of init functions in the anonymously imported packages will be called once).

## Each Non-Anonymous Import Must Be Used at Least Once

```golang
package main

import (
	"net/http" // error: imported and not used
	. "time"   // error: imported and not used
)

import (
	format "fmt"  // okay: it is used once below
	_ "math/rand" // okay: it is not required to be used
)

func main() {
	format.Println() // use the imported "fmt" package
}
```