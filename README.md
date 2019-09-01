# golang cheatsheet

### 1. Type assignability
```golang
var y T = x // x's type is V
```
1. V is identical to T
2. V and T have identical underlying types and at least one of V or T is not a defined type
3. With channels you also can make one-direction channel from two-direction channel
4. Assigning to interface type T value implementing T
5. It’s allowed to assign nil to variable which is a pointer, function, slice, map, channel or interface type
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
5. For a base type, the non-blank names of methods bound to it must be unique.
6. If the base type is a struct type, the non-blank method and field names must be distinct.
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
### 5. Method expressions
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
### 6. Method values
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
6. It's alsoe possible to create method values from interface values:
```go
var i interface { M(int) } = myVal
f := i.M; f(7)  // like i.M(7)
```
### 7. Pointers
1. A pointer type denotes the set of all pointers to variables of a given type. This type called the base type of the pointer. Note that there are no any difference between pointer type and defined type that is made from pointer:
```go
type Pint *int

var x = 1
var px *int = &x // type of px is *int. Base type of *int is int
var mypx Pint = px // type of mypx is Pint. Base type of Pint is also int
```
### 8. Functions
1. A function declaration may omit the body. Such a declaration provides the signature for a function implemented outside Go, such as an assembly routine:
```go
func flushICache(begin, end uintptr)  // implemented externally
```
2. If the function's signature declares result parameters, the function body's statement list must end in a terminating statement(it’s not only return, see documentation).
### 9. Labels and break, continue, goto statements
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
### 10. Other
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
