# golang cheatsheet
### 0. Types
1. ```bool```, all numeric and ```string``` are defined types
2. ```byte``` is alias for ```uint8```
3. ```rune``` is alias for ```int32```
4. ```int32``` and ```int``` are not the same type even though they may have the same size on a particular architecture.
5. Default values:
```
false for booleans
0 for numeric types
"" for strings
nil for pointers, functions, interfaces, slices, channels, and maps
```
### 1. Type assignability
```golang
var y T = x // x's type is V
```
1. V is identical to T
2. V and T have identical underlying types and at least one of V or T is not a defined type 
3. With channels you also can make one-direction channel from two-direction channel
4. Assigning to interface type T value implementing T
5. It’s allowed to assign nil to variable which is a pointer, function, slice, map, channel or interface
6. x is assignable to type T if x is an untyped constant representable by a value of type T
### 2. Declarations
1. Shadowing - an identifier declared in a block may be redeclared in an inner block. While the identifier of the inner declaration is in scope, it denotes the entity declared by the inner declaration:
```go
var x = 0
if x == 0 {
 var x = “text”
 fmt.Println(x) // x == “text”
}
fmt.Println(x) // x == 0
```
2. Short variable declaration. It can be used only in function body:
```go
y := 1 // compile error
var y = 1 // it’s ok
func f() {
  x := 2 // it’s ok
}
```
3. Unlike regular variable declarations, a short variable declaration may redeclare variables that were defined in the same block. There are two requirements: at least one declared variable is new and redeclared variables must save their types
4. Redeclaration does not introduce a new variable; it just assigns a new value to the original
### 3. Method declarations
1. Receiver type must be a defined type T or a pointer to a defined type T. T is called the receiver **base type**
2. A receiver base type cannot be a pointer or interface type
3. Receiver base type must be defined in the same package as the method
4. The method is said to be bound to its receiver base type and the method name is visible only within selectors for type T or *T.
5. For a base type, the non-blank names of methods bound to it must be unique.(As well as the names of methods from whole method set)
6. If the base type is a struct type, the non-blank method names and field names must be distinct.(But promoted field can has the same name. Also, a method from embedded struct can has name which field from other level of embedding already has)
7. A non-blank receiver identifier must be unique in the method signature.
8. If the receiver's value is not referenced inside the body of the method, its identifier may be omitted in the declaration.
### 4. Method sets
1. A type may have a method set associated with it
2. The method set of an interface type is its interface
3. The method set of any other type T consists of all methods declared with receiver type T
4. The method set of the corresponding pointer type *T is the set of all methods declared with receiver *T or T - that is, it also contains the method set of T
5. Promoted methods are included in the method set of the struct. Let we have struct type S and a defined type T. Promoted methods are included in the method set of the struct as in next two rules.
6. If S contains an embedded field T, the method sets of S and *S both include promoted methods with receiver T. The method set of *S also includes promoted methods with receiver *T.
7. If S contains an embedded field *T, the method sets of S and *S both include promoted methods with receiver T or *T.
8. In a method set, each method must have a unique non-blank method name.
9. A defined type does not inherit any methods bound to the given type, but the method set of an interface type or of elements of a composite type remains unchanged:
```go
// A Mutex is a data type with two methods, Lock and Unlock.
type Mutex struct         { /* Mutex fields */ }
func (m *Mutex) Lock()    { /* Lock implementation */ }
func (m *Mutex) Unlock()  { /* Unlock implementation */ }

// NewMutex has the same composition as Mutex but its method set is empty.
type NewMutex Mutex

// The method set of PtrMutex's underlying type *Mutex remains unchanged,
// but the method set of PtrMutex is empty.
type PtrMutex *Mutex

// The method set of *PrintableMutex contains the methods Lock and Unlock bound to its embedded field Mutex.
type PrintableMutex struct {
  Mutex
}

// MyBlock is an interface type. Therereby has the same method set as Block.
type MyBlock Block
```
### 5. Selectors
```go
x.f // (1), of type T 
```
1. For a primary expression (1) that is not a package name, the selector expression
denotes the field of the value ```x``` or method from method set of type ```T```. if x was of type *T it would be the same syntax for selecting fields from x.
2. A selector f may denote a field or method f from the method set of type T. The number of embedded fields traversed to reach f is called its depth in T. The depth of a field or method f declared in T is zero. The depth of a field or method f declared in an embedded field A in T is the depth of f in A plus one.
3. For a value x of type ```T``` ```x.f``` denotes the field or method at the shallowest depth in ```T``` where there is such an f. If there is not exactly one f with shallowest depth, the selector expression is illegal.
4. For a value x of type ```I``` where ```I``` is an interface type, ```x.f``` denotes the actual method with name ```f``` of the dynamic value of ```x```. If there is no method with name ```f``` in the method set of ```I```, the selector expression is illegal.
5. As an exception, if the type of ```x``` is a defined pointer type and ```(*x).f``` is a valid selector expression denoting a field (but not a method), ```x.f``` is shorthand for ```(*x).f```.
6. In all other cases, ```x.f``` is illegal.
7. If ```x``` is of pointer type and has the value ```nil``` and ```x.f``` denotes a struct field, assigning to or evaluating ```x.f``` causes a run-time panic.
8. If ```x``` is of interface type and has the value ```nil```, calling or evaluating the method ```x.f``` causes a run-time panic.
### 6. Method expressions
```go
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
```
1. If M is in the method set of type T, T.M is a function that is callable as a regular function with the same arguments as M prefixed by an additional argument that is the receiver of the method.
2. The expression ```T.Mv``` yields a function equivalent to Mv but with an explicit receiver as its first argument; it has signature ```func(tv T, a int) int```
3. The expression ```(*T).Mp``` yields a function value representing Mp with signature ```func(tp *T, f float32) float32```. Mutation of ```tp``` can be performed.
4. The expression ```(*T).Mv``` yields a function value representing Mv with signature ```func(tv *T, a int) int```. But behind the scenes go dereference ```tv``` and pass the obtained value to the underlying method. So, you cannot mutate ```tv``` anyway.
5. The expression ```T.Mp``` is illegal and yelds compilation error mecause ```Mp``` is not in method set of ```T```
6. It is legal to derive a function value from a method of an interface type. The resulting function takes an explicit receiver of that interface type.
### 7. Method values
```go
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
var pt *T
func makeT() T
```
1. If the expression ```x``` has static type ```T``` and ```M``` is in the method set of type ```T```, ```x.M``` is called a method value. The method value ```x.M``` is a function value that is callable with the same arguments as a method call of ```x.M```. The expression x is evaluated and saved during the evaluation of the method value; the saved value is then used as the receiver in any calls, which may be executed later.
2. The expression ```t.Mv``` yields a function value of type ```func(int) int```.
3. The expression ```pt.Mp``` yields a function value of type ```func(float32) float32```. In this case mutation of pt is available.
4. The expression```pt.Mv``` is also legal because ```Mv``` is in method set of ```pt```. In this case the expression will be equivalent to ```(*t).Mv```.
5. Despite of the fact that ```Mp``` is not in method set of ```T``` the expression ```t.Mp``` is also legal because it will be replaced with ```(&t).Mp```. ```t``` must be addressable, so the following code will not compile: 
```go
f := makeT().Mp   // invalid: result of makeT() is not addressable
```
6. It's also possible to create method values from interface values:
```go
var i interface { M(int) } = myVal
f := i.M; f(7)  // like i.M(7)
```
### 8. Interfaces
1. Types implement interfaces implicitly
2. Interface value gives access only to methods of its interface type:
```golang
type I interface {
    M1()
}
type T int64
func (T) M1() {}
func (T) M2() {}
func main() {
    var i I = T(10)
    i.M1()
    i.M2() // i.M2 undefined (type I has no field or method M2)
}
```
2. Default value for interfaces is ```nil```
3. Besides having static type what is interface type itself, every interface value has dynamic type and dynamic value. It's a type of a value that is assigned to the given interface value. If interface value is nil, dynamic type is ```nil``` as well as dynamic value is ```nil```. Note that it's possible to have non-nil interface value with ```nil``` dynamic value:
```golang
var i I // some interface type
var v V // some user-defined type that implements I
i = v // from now i has dynamic type V and dynamic value nil. Note that this is not the same that having i == nil
```
4. Go has special empty interface type:
```golang
 interface {}
```
Empty interface is automatically implemented by any type. So, value of any type can be assigned to such interface variable
5. It’s allowed to embedded other interfaces — either defined in the same package or imported:
```golang
import "fmt"
type I interface {
     m1()
}
type J interface {
    m2()
    I
    fmt.Stringer
}
```
6. Circular embedding of interfaces is disallowed and will be detected while compilation
7. In case of embedding interface from other package that includes non-exported methods, embracing interface will include non-exported methods.
### 9. Type assertions
```golang
x.(T) // x is of type I
```
1. For an expression x of interface type and a type T, the primary expression ```x.(T)``` asserts that x is not nil and that the value stored in ```x``` is of type ```T```. The notation ```x.(T)``` is called a type assertion.
2. If ```I``` is not interface type the expression is invalid.
3. If ```T``` is not interface type and ```T``` doesn't implement ```I``` the expression is invalid.
4. If ```T``` is not interface type and dynamic value from ```x``` is identical to ```T```, result is value of type ```T```.
5. If ```T``` is interface type and type of dynamic value of x implements ```T```, result is value of type ```T```.
6. In every other case run-time error occurs(e.g. if ```x``` is ```nil```, if ype of dynamic value of x doesn't implement ```T```)
7. A type assertion used in the special form ```var v, ok = x.(T)``` prevents any panicking. The value of ```ok``` is ```true``` if the assertion is correct. Otherwise it is ```false``` and the value of ```v``` is the ```nil``` value for type T.
### 10. Type switch
```golang
switch i := x.(type) { // The TypeSwitchGuard with a short variable declaration (1)
    case nil:
	printString("x is nil")                // type of i is type of x (interface{})
    case int:
	printInt(i)                            // type of i is int
    case float64:
	printFloat64(i)                        // type of i is float64
    case func(int) float64:
	printFunction(i)                       // type of i is func(int) float64
    case bool, string:
	printString("type is bool or string")  // type of i is type of x (interface{})
    default:
	printString("don't know the type")     // type of i is type of x (interface{})
}
```
1. A type switch is switch that compares types rather than values. Cases match actual types T against the dynamic type of the expression x. The types listed in the cases of a type switch must all be different.
2. The TypeSwitchGuard (1) may include a short variable declaration. When that form is used, the variable is declared in the implicit block of each clause. In clauses with a case listing exactly one type, the variable has that type; otherwise, the variable has the type of the expression in the TypeSwitchGuard.
3. Instead of a type, a case may use the predeclared identifier ```nil```; that case is selected when the expression in the TypeSwitchGuard is a ```nil``` interface value. There may be at most one nil case.
4. The type switch guard may be preceded by a [simple statement](https://medium.com/golangspec/simple-statement-notion-in-go-b8afddfc7916), which executes before the guard is evaluated:
```golang
var v I1 = T1{}
switch aux := 1; v.(type) {
case nil:
    fmt.Println("nil")
case T1:
    fmt.Println("T1", aux)
case T2:
    fmt.Println("T2", aux)
}
```
### 11. Channels
1. The  the type of the value to be sent must be assignable to the channel's element type.
2. A send on a closed channel proceeds by causing a run-time panic.
3. A send on a nil channel blocks forever.
4. Sending to or closing a closed channel causes a run-time panic.
5. It's not possible to call built-in ```close``` on receive-only channel(cause a compile error)
6. 
### 12. Defer statement
1. A ```defer``` statement invokes a function whose execution is deferred. Cases when ```defer``` invokes given function:
```
  1. The surrounding function returns
  2. The surrounding function executed a return statement
  3. The surrounding function reached the end of its function body
  4. The corresponding goroutine is panicking
 ```
 2. The expression must be a function or method call
 3. Calls of built-in functions are restricted
 4. Each time a ```defer``` statement executes, the function value and parameters to the call are evaluated as usual and saved anew but the actual function is not invoked. Instead, deferred functions are invoked immediately before the surrounding function returns, in the reverse order they were deferred.
 5. If the surrounding function returns through an explicit return statement, deferred functions are executed **after** any result parameters are set by that return statement but **before** the function returns to its caller.
 6. If a deferred function value evaluates to nil, execution panics when the function is invoked, not when the ```defer``` statement is executed.
 7. If the deferred function is a function literal and the surrounding function has named result parameters that are in scope within the literal, the deferred function may access and modify the result parameters before they are returned:
```golang
// f returns 42
func f() (result int) {
	defer func() {
		// result is accessed after it was set to 6 by the return statement
		result *= 7
	}()
	return 6
}
```
### 13. Handling panics
1. ```recover``` function allows a program to manage behavior of a panicking goroutine. Suppose a function ```G``` defers a function ```D``` that calls recover and a panic occurs in a function on the same goroutine in which ```G``` is executing. When the running of deferred functions reaches ```D```, the return value of ```D```'s call to recover will be the value passed to the call of panic. If ```D``` returns normally, without starting a new panic, the panicking sequence stops. In that case, the state of functions called between ```G``` and the call to panic is discarded(so, return value of ```G``` in this case will be default value for ```G``` return value type), and normal execution resumes. Any functions deferred by ```G``` before ```D``` are then run and ```G```'s execution terminates by returning to its caller:
```golang
// input after execution this: 
// recovered err: Example error
// DD is called as usual
// x: 0
package main

import "fmt"

func DD() {
	fmt.Println("DD is called as usual")
}

func D() {
	var err = recover()
	fmt.Println("recovered err:", err)
}

func G() int {
	defer DD()
	defer D()
	panic("Example error")
	return 10
}

func main() {
	var x = G()
	fmt.Println("x:", x)
}
```
2. The return value of recover is nil if any of the following conditions holds:
```
panic's argument was nil;
the goroutine is not panicking;
recover was not called directly by a deferred function.
```
### 14. Pointers
1. A pointer type denotes the set of all pointers to variables of a given type. This type called the base type of the pointer. Note that there are no any difference between pointer type and defined type that is made from pointer:
```go
type Pint *int

var x = 1
var px *int = &x // type of px is *int. Base type of *int is int
var mypx Pint = px // type of mypx is Pint. Base type of Pint is also int
```
### 15. Functions
1. A function declaration may omit the body. Such a declaration provides the signature for a function implemented outside Go, such as an assembly routine:
```go
func flushICache(begin, end uintptr)  // implemented externally
```
2. If the function's signature declares result parameters, the function body's statement list must end in a terminating statement(it’s not only return, see documentation).
3. The final incoming parameter in a function signature may have a type prefixed with .... A function with such a parameter is called variadic and may be invoked with zero or more arguments for that parameter:
```golang
func(a, b int, z float64, opt ...interface{}) (success bool)
```
4. The built-in functions do not have standard Go types, so they can only appear in call expressions; **they cannot be used as function values**.
If f is variadic with a final parameter p of type ...T, then within f the type of p is equivalent to type []T. If f is invoked with no actual arguments for p, the value passed to p is nil. Otherwise, the value passed is a new slice of type []T with a new underlying array whose successive elements are the actual arguments, which all must be assignable to T. The length and capacity of the slice is therefore the number of arguments bound to p and may differ for each call.
### 16. Labels and break, continue, goto statements
1. Labels can be used for *goto*, *break* and *continue* statements
2. It’s optional for *break*, *continue* statements, but required for *goto*
3. Label’s scope is full function body, not only lines that appears after label declaration:
```go
func main() {
   fmt.Println(1)
   goto End
   fmt.Println(2)
End:
   fmt.Println(3)
}
```
4. Scope doesn’t contain nested functions so it’s illegal to write:
```go
 //compile error
 func() { 
    fmt.Println(“Nested function”)
    goto End
 }()
End:
```
5. Labels are not block scoped. So it’s impossible to redeclare label inside nested block:
```go
//compile error
goto X
X:
   {
   X:
   }
```
6. Identifiers of the labels live in a separate space so they don’t conflict with i.e. variables identifiers:
```go
x := 1
goto x
x:
fmt.Println(x)
```
7. break can be used in *select*, *switch*, *for* statements
8. continue can be used only in *for* statement
9. Label for break statement must be the one associated with enclosing for, switch or select statement. So, It’s impossible to compile:
```go
///compile error
FirstLoop:
for i := 0; i < 10; i++ {
}
for i := 0; i < 10; i++ {
  break FirstLoop
}
```
10. Using of continue is the same but it’s can be used only with *for*
11. *goto* can move control only within the same function
12. Executing the *goto* statement must not cause any variables to come into scope that were not already in scope at the point of the goto:
```go
// compile error
goto L
	v := 3
L:
```
13. *goto* cannot move into other block:
```go
// compile error
goto Block
{
Block:
    v := 0
    fmt.Println(v)
}
```
### 17. Program initialization
```golang
var (
	a = c + b  // == 9
	b = f()    // == 4
	c = f()    // == 5
	d = 3      // == 5 after initialization has finished
)

func f() int {
	d++
	return d
}
```
1. Package-level variable initialization run iteratively. Each iteration selects the variable earliest in declaration order **which has no dependencies on uninitialized variables**.
2. If any variables are still uninitialized when package-level initialization ends that means that those variables are part of one or more initialization cycles, and the program is not valid.
3. Multiple variables on the left-hand side of a variable declaration initialized by single (multi-valued) expression on the right-hand side are initialized together: If any of the variables on the left-hand side is initialized, all those variables are initialized in the same step:
```golang
var x = a
var a, b = f() // a and b are initialized together, before x is initialized
```
4. The declaration order of variables declared in multiple files is determined by the order in which the files are presented to the compiler: Variables declared in the first file are declared before any of the variables declared in the second file, and so on.
5. Each package performs initialization only once, no matter how many times this package has been imported.
6. Variables may also be initialized using functions named init declared in the package block, with no arguments and no result parameters.
```golang
func init() { … }
```
7. Multiple such functions may be defined per package, even within a single source file. In the package block, the init identifier can be used only to declare init functions, but the identifier itself is not declared. Thus init functions cannot be referred to from anywhere in a program.
8. Order of initialization:
```
1. initialization of imported packages
2. assigning initial values to all package-level variables
3. Calling all init functions in the order they appear in the source, possibly in multiple files, as presented to the compiler.
```
9. Package initialization (variable initialization and the invocation of init functions) happens in a single goroutine, sequentially, one package at a time. An init function may launch other goroutines, which can run concurrently with the initialization code. However, initialization always sequences the init functions: it will not invoke the next one until the previous one has returned.
### 18. Other
1. Scope of importing packages is file block
2. Identifiers has declared outside of any function are visible across the whole package (the scope is the package block)
3. Types can be recursive, but only with nested pointer types:
```go
// correct type definition
type X struct {
   name string
   next *X
}
// incorrect(compile error cause of recursive non-pointer field)
type X struct {
   name string
   next X
}
```
