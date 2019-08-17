# golang cheatsheet

## 1. Type assignability // var T y = x where x type is V
1. V is identical to T
2. V and T have identical underlying types and at least one of V or T is not a defined type
3. With channels you also can make one-direction channel from two-direction channel
4. Assigning to interface type T value implementing T
5. It’s allowed to assign nil to variable which is a pointer, function, slice, map, channel or interface type
6. x is assignable to type T if x is an untyped constant representable by a value of type T

## 2. Labels and break, continue, goto statements

1. Labels can be used for goto, break and continue statements
2. It’s optional for goto and break, continue statements, but required for goto
3. Label’s scope is full function body, not only lines that appears after label declaration:
    func main() {
   fmt.Println(1)
   goto End
   fmt.Println(2)
End:
   fmt.Println(3)
}
4. Scope doesn’t contain nested functions so it’s illegal to write:
    //compile error
    func() { 
       fmt.Println(“Nested function”)
       goto End
   }()
End:
5. Labels are not block scoped. So it’s impossible to redeclare label inside nested block:
//compile error
goto X
X:
   {
   X:
   }
6. Identifiers of the labels live in a separate space so they don’t conflict with i.e. variables identifiers: 
x := 1
   goto x
x:
   fmt.Println(x)
7. break can be used in select, switch, for statements
8. continue can be used only in for statement
9. Label for break statement must be the one associated with enclosing for, switch or select statement. So, It’s impossible to compile:
        ///compile error
       FirstLoop:
   for i := 0; i < 10; i++ {
   }
   for i := 0; i < 10; i++ {
       break FirstLoop
   }
10. Using of continue is the same but it’s can be used only with for 


















3. Other
1. Scope of importing packages is file block
2. Identifiers has declared outside of any function are visible across the whole package (the scope is the package block)
3. Types can be recursive, but only with nested pointer types:
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








