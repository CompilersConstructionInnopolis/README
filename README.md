# Team C++

- **Daniil Livitin** - language features developer
- **Roman Soldatov** - language features developer
- **Mikhail Martovitsky** - implement a standard library and predefined functions. Online compiler Backend developer.
- **Timur Nugaev** - tester & test writer. Online compiler Frontend developer.

## Links
- The [link](https://github.com/CompilersConstructionInnopolis/ACCPA) to the source code.
- The [link](https://github.com/CompilersConstructionInnopolis/README) to the readme with language documentation
- The [organisation](https://github.com/CompilersConstructionInnopolis) with repositories.

# Language Plus++

- Functional Lisp-like language with a static type system. 
- The language is not purely functional.

The language represents a sequence of elements. Elements can be:
- **List**. The sequence of elements. They can be considered as just a sequence of homogenous elements or a list can be of a special form which represents a list as a function. In this case, elements don't require to have the same type. The special form of a list is an S-expression, so each element is evaluated before a function call.
- **Literal**. The basic values such as integers, doubles, booleans and strings.
- **Atom**. Identifier of a variable which can be assigned to a literal or a list. An identifier can be written with the use of letters, digits and underscores. The name can't start with a digit and it must contain at least one letter.
- **Function type**. A function declaration of input argument types and return types separated by `->`
- **Tuple**. A sequence of heterogeneous elements separated by a comma `,`.

# Grammar

```
Program : Element { Element } 
List : ( Element { Element } ) 
Element : Atom | Literal | List | Tuple | Functype
Atom : Identifier
Literal : [+|-] Integer | [+|-] Double | Boolean | String
Tuple : (Element , Element {, Element})
Functype : (Element {Element} -> Element)
Identifier : { _ } Letter { Letter | DecimalDigit | _ }
Letter : Any Unicode character that represents a letter 
Integer : DecimalDigit { DecimalDigit }
DecimalDigit : 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 
Double : Integer . Integer
Boolean : true | false
String: " Characters "
Characters: { Letter }
```

# Type Checker
- Before interpretation compiler runs the type checker which analysis AST asking each node its type. Connected nodes compare their expected types with actual or infer ones. Each node has a set of rules on which type of arguments it expects and what type of argument it should return. For example, if the node is a condition which accepts a single argument of type boolean and returns some node of any type, then we take into account several things  `(cond BooleanCondition TrueBranch FalseBranch`:
- `cond` has three children nodes that represent branches(`TrueBranch` and `FalseBranch`) and `BooleanCondition`.
- `BooleanCondition` node must return a boolean.
- Branches must return the same type. For this purpose, we recursively check what each branch returns and then compare types.
- The parent node (which calls `cond`) must provide an external atom argument for `cond` of type boolean. Also, this parent node knows what `cond` returns since we checked its branches. Therefore, we can conclude what the parent node should return. Thus, if we know that e.g. `cond` returns `Int`, but its parent node should return `Double`, then it's the types error which we detected during the compilation.
- if the node has an `Auto` type, but its children or parent provides some defined type for it (e.g. `String`). Then we can conclude that argument's type `Auto` is actually a String. That's how the type inference works.

In compiler implementation, to check that connected nodes pass and return arguments of expected types, each AST node has the following methods
```
List<NodeType> getArgumentType(); // Get a list of argument types which node accepts
void setArgumentType(List<NodeType>); // Set a list of argument types which node should accept. Call this method for type inference (if node had Auto type)
NodeType getReturnType(); // Get a type of return type
void setReturnType(NodeType type); // Set return type which node should provide. Call this method for type inference (if node had Auto type)
```

## Check whether the typing is correct
- For the `function` we should check if the provided arguments are the same types as declared in the `functype` keyword.
If the argument is not Base type then we should recursively get the type of argument.
- For the variable (atom) we can use the `define` keyword to specify the variable's type.
- If the function accepts or returns an atom of type `Any` or `Num` then it can be used for different argument types (`Int` and `Double` will be acceptable for `Num`). In this case, possible exceptions would be detected only during the runtime.

## Infer the type of expressions

In our language type annotation in the source code is optional. 
If type is not specified explicitly then it should be inferred by the interpreter automatically. Alternatively, you can specify a type as unknown using `Auto`.

- if the node's argument or the return type is `Auto` and such node does not have special rules of required types (*e.g. the `cond` has a rule that condition has a type of boolean*) and its connected nodes also can't conclude their required types then the type inference won't work during the compile time. So, it's not possible to traverse through nodes and detect their types if everything has the type `Auto`.
- if connected nodes have types `Any` and `Auto` then `Auto` could be only resolved as a type of `Any`. However, we could get a type exception during the runtime because the type checker was not able to check these generic types.

# Base Types

Language has general-purpose types that are built into it:
* Numeric types (`Int`, `Double`, `Num` - parent class of `Int` and `Double`)
    - `Int` - integer number
    - `Double` - number with floating point
    - `Num` - can be acceptable either `Int`, or `Double`
* Boolean type (`Boolean`)
    - `Boolean` - has values: `true`, `false`
* Unit type (`Unit`). It indicates that a function accepts or returns nothing. 
* String type (`String`). Any symbols between quotes signs "..."
* Any type (`Any`). Parent of all types. It can be used in functions in which we don't care about the variable's type.
* Homogeneous lists. A list is a sequence of elements separated by whitespaces and enclosed by parentheses. However, there are two types of lists in general.
    1. The list with elements of the same type. So, this list is just a simple list of elements.
    2. However, the list can be interpreted as an S-expression. So, the compiler considers this list as a **function**. We call such a list a list with a special form. A list with a special form has the same syntax, but it is considered as a function definition or a function call. Its elements don't require to have the same type (since it's not a simple sequence of homogeneous elements). Therefore, such a list doesn't support classic operations on a list (e.g. `cons`, `tail`) since it represents a function.
        - **Function definition**
            - The first element is an atom representing the function name
            - The second element is a list of arguments
            - The third element is the function body
        - **Function call**
            - The first element is an atom representing the function name
            - The second element is a list of arguments
    
* Heterogeneous tuples. The tuple should have more than 1 element. The difference between tuples and lists is that tuple's elements are separated by a comma. 


# User-defined Terms and Types

* User can define aliases with the use of primitives.
* Aliases can be defined in a local scope.
```
(type Seconds Int)
(type Minutes Int)

(functype ConvertSecondsToMinutes (Seconds -> Minutes))
(func ConvertSecondsToMinutes (seconds) (divide seconds 60))
```

# Import
- To import some program with defined functions and atoms from another file just write `import file.txt`. 
- It should be written above the usage of the functions.
- From an implementation point of view, before running the lexical analysis we substitute the content of the imported file in a program row where `import` was written.

# Standard Library

Our standard library has the following functions:
* `(functype lastElement ((Any) -> Any))`
* `(functype reverse ((Any) -> (Any)))`
* `(functype getListWithoutLastElement ((Any) -> (Any)))`
* `(functype getMinElement ((Any) -> Any))`
* `(functype removeElement (Any (Any) -> (Any)))`
* `(functype sort ((Any) -> (Any)))`

To import the standard library you need to write in the source code:

`import std.txt`

After that, you can use the aforesaid functions in your code.

# Nested definitions

- Our language support nested definitions. It means that it is possible to define a local variable/function in any scope.
- From an implementation point of view, we keep the singleton stack of hashmaps. When the function should enter the local scope, we create a new hashmap and put it in the stack. All local function and atom definitions are stored in such a hashmap. To get the outer scope atom we traverse through the stack. When we leave the local scope, the top stack's hashmap is deleted.

Example:

```
(setq x 55) ; define global x = 55
(setq y 66) ; define global y = 66
(prog (x y) (
    (func helper () 42) ; define local function 'helper'
    (setq x (helper)) ; define local x = 42
    (setq y 6) ; define local y = 6
    (x y) ; (42 6)
))
; the 'helper' function doesn't exist anymore
(x y) ; These x and y are the global ones (55 66)
```

# Simple Constraint-Based Type Inference

Reserved word `Auto` is used for specifying omitted types

```
; Define a function 'sum'
(functype sum (Auto Auto -> Auto)) 
(func sum (a b) (plus a b))

; Compiler checks that the function 'sum' is used for integers.
; Therefore, 'sum' is resolved as (Int Int -> Int) function
(sum 5 4) 

; Since 'sum' was inferred as (Int Int -> Int) function, here you'll get a compile error because arguments must be integers, not doubles.
; (sum 4.4 2.3)
```

# Typing rules

* Type of the function : 
Everything before `->` is the type of argument of the function
Everything after `->` is returned type of function
```
(functype sum (Num Num -> Num)) ; function takes 2 arguments of type Num and return type Num

(functype example ((Num -> Num) -> Num)) ; argument of the function is a function that takes Num and return Num and returned type of the function is Num
```







Comment lines `;`
```
; Everything after the semicolon is a comment on this line
; The compiler will ignore this part of the code 
```

Create a variable `setq`
`(functype setq (Any Any -> Unit))`
```
(setq a 5) ; a has type Int
(setq b 5.3) ; b has type Double
(setq c true); c has type Boolean
(setq d "hello, world"); d has type String
```

Reassignment
```
(setq x 5) ; We declared and initialized a variable x which has type Int
(setq x 6); x reassigned a value. It's still has a type Int
; (setq x 8.0) compile error. Can't reassign to Double since x was declared as Int
(setq a 8)
(setq x a); now x has a value of 8
```

`define` keyword
`(functype define (Any Any -> Unit))`
The user can't define an atom of types: `Unit`, `Any`, `Num`
```
(define x Int) ; declare a variable e with type Int
(setq x 5); assign an integer value
```

```
(define x Int); In futher usage x must be Int
; (setq x true) compile error
(setq x 5); valid assignment since 5 is integer
```

`functype` keyword
- The syntax: (`functype` (`list of arguments types` -> `return type`)).
- You can declare the `functype` for a particular function only once above the `func`.
- `Unit` argument can be used only as a single type.
```
(functype unitExample1 (Unit -> Int)) ; can use Unit as argument
(functype unitExample1 (Unit Double -> Int)) ; can't use Unit. It should be single.
(functype myEasyFunction (Int -> Double)) ; a function which takes Int as an argument and return Double

(functype myMediumFunction (Int Boolean -> Double)) ; a function which takes two arguments: Int and Boolean and returns Double.

(functype myHardFunction ((Int -> Boolean) -> Double)) ; a function which takes a function as an argument and returns Double. The argument function is a function which takes Int and returns Boolean.

(functype myInterestingFunction ((Int -> Boolean) -> (Double -> String))) ; a function which takes a function as an argument and returns a function. The argument function accepts Int and return Bolean. The result function is a function which takes Double and returns String

```

Creare a function `func`
The syntax: (`func` `function name` `list of arguments` `function body`)
```
(functype squareSum)
(func squareSum (a b)
(
    (plus (times a a) (times b b))
))

(setq x (squareSum 2 3)) ; call the function squareSum with arguments 2 and 3. Assign result to x
x ; should output 13
```

```
(functype getBestNumber (Unit -> Int)) ; The function doesn't reqire any argument and returns an integer.
(func getBestNumber () 42)
(getBestNumber) ; this line will call function getBestNumber and outputs 42
```

```
(setq x 3)
(functype incrementByValue (Int -> Unit))
(func incrementByValue (n)
    (setq x (plus x n))
)
; The function accepts an argument n and increments outer scope variable x by n. The function itself returns nothing
```

```
(setq x 3)
(functype sideEffect (Unit -> Unit))
(func sideEffect ()
    (setq x (plus x 1))
)
; The function accepts nothing and returns nothing. It just increments outer scope variable x
```

Arithmetic functions
```
(functype plus (Num Num -> Num))
(functype minus (Num Num -> Num))
(functype times (Num Num -> Num))
(functype divide (Num Num -> Num))
```

Logical operators
```
(functype and (Boolean Boolean -> Boolean))
(functype or (Boolean Boolean -> Boolean))
(functype xor (Boolean Boolean -> Boolean))
(functype not (Boolean -> Boolean))
```

Comparisons
```
(functype equal (Any Any -> Boolean))
(nonequal false (Any Any -> Boolean))
(nonequal (Any Any -> Boolean))
(less (Num Num -> Boolean))
(lesseq (Num Num -> Boolean))
(greater (Num Num -> Boolean))
(greatereq (Num Num -> Boolean))
```

`Cond` keyword
Syntax: (`cond` `Boolean condition` `result if true` `result if false`)
The result for `true` and `false` branches must have the same type.
```
(setq a 5)
(setq b 6)
(cond (less a b) ; condition
    (plus a b) ; the branch if condition is true
    (minus a b) ; the branch if condition is false
)
```

```
(cond (less 4 3)
    45.2
    false
) ; compile error since the return type must be the same for both branches
```

Lists. The list is homogeneous.
If the first element is a function name then such a list is considered as a function call, the rest list's elements are function arguments.

```
(plus 1 2) ; this is a function call

(1 2 3) ; this is a list of integers

; (1 5.23 true) compile error. The list must be homogeneous

() ; empty list

(define x (Int)) ; x is a list of integers
(setq x (1 2 3)) ; now x is equal to (1 2 3)
; (setq x (true false)) complie error since x is a list of integers, not booleans
```

For function arguments, we denote a list as `(T)` where `T` - is the list type.
```
(functype getMaxElement ((Int) -> Int)) ; the function accepts a list of integers and returns an integer number

(functype getMaxElement ((Int) -> (Int))) ; the function accepts a list of integers and returns a list of integers
```
 
Operations on list
```
(functype cons (Any (Any) -> (Any)))
(functype tail ((Any) -> (Any)))
(functype isempty ((Any) -> Boolean))
(functype length ((Any) -> Int))

(cons 3 ()); returns a list with one element (3)
(cons 1 (2 3)); returns a list (1 2 3)

(head (1 2 3)); returns the first element

(tail (1 2 3)); returns a list without the first element

(isempty (1 2 3)); returns false
(isempty ()); returns true

(length (1 2 3)); returns 3
(length ()); returns 0
```

`lambda` keyword.
Syntax: (`lambda` `list of arguments` `function body`)
```
(define myFunc (Int -> Int))
(setq myFunc (lambda (p) (plus p 1)))

(functype applyFunction (Int (Int -> Int) -> Int)) ; the function takes an integer and some function as an arguments and returns intger number.
(func applyFunction (x f)
    (f x)
)
(applyFunction 3 (lamda (p) (times p p)))
```

`let binding`
Syntax: (`let` `list of defined  variables` `in body`)
```
(func getMax (currentMax list)
    (cond (isempty list)
        currentMax
        (let 
            (
                (first (head list))
                (restList (tail list))
            )
            (cond (greater first currentMax)
                (getMax first restList)
                (getMax currentMax restList))
        )    
    )
)

(setq lst (2 4 1 3))
(getMax (head lst) (tail lst))
```

`tuples`

```
(define x (Int, Double, Int))
(setq x (3, 2.2, 30))
(getAt x 2) ; returns 2.2
(setAt x 2 4.4) ; now x = (3, 4.4, 30)
```

Currying function
```
(functype moreThanX (Int (Int) -> (Int)))
(func moreThanX (x lst) (
    (cond (isepmty lst) 
        ()
        (let 
            (
                (first (head lst))
                (restLst (tail lst))
            )
            (cond (greater first x) 
                    (cons first (moreThanX x restLst))
                    (moreThanX x restLst)
            )
        )
    )
)

(setq moreThan5 (moreThanX 5)) ; moreThan5 can be considered as a function similar to moreThanX, but the first argument is already passed as 5.

(setq initialList (1 2 5 6 7))

(setq listResult (moreThan5 initialList))

; listResult will have values '(6 7)
```

# Interpreter

In the [interpreter](https://github.com/CompilersConstructionInnopolis/ACCPA/tree/main/src/main/java/interpreter) package there are classes related to AST interpretation.

* [AtomsTable.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/AtomsTable.java) saves defined atoms and their values. This class is represented as a singleton. Atoms are stored in some local context. Atoms with similar names shadows atoms from the outer scope. Each local context is represented as a HashMap. The hierarchy of local contexts is presented as a Stack. When you enter into some function or loop the interpreter calls the` introduceLocalContext()` function to create a new temporary local context which will be deleted after leaving the form (`leaveLocalContext()` will be called).
* [FunctionsTable.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/FunctionsTable.java) is similar to [AtomsTable.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/AtomsTable.java), but it stores defined user-functions.
* [PredefinedFunction.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/PredefinedFunction.java) and [DefinedFunction.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/DefinedFunction.java) classes are utility functions to interpret a list as a function call and evaluate it.

# Build project

***Online interpreter***
You can test the interpreter on this [website](https://plus-plus-plus.netlify.app)

***Instructions for building the project***

 
1. Сlone a repository of our project.
2. Navigate to the folder of the project using the terminal.
3. Run the command `mvn clean install`. This command will create a ‘target’ folder and an executable .jar file inside.
 
***Instructions for using the prototype***

1. Create a build of the application (see the previous step)
2. Navigate to the created ‘target’ folder.
3. Run a command: `java -jar Compilers_Innopolis-1.0-SNAPSHOT.jar ../code/main.txt`
    
> Note: before running a command you need to enter a code of the program inside the code/main.txt file. You can specify another path to the program file.


# Demo program

```
; sorting example
(functype getMinElement ((Int) -> Int)) 
(functype removeElement (Int (Int) -> (Int)))
(functype sort ((Int) -> (Int)))

(func getMinElement (lst)
(cond (isempty (tail lst))
        (head lst)
        (cond (less (head lst) (getMinElement (tail lst)))
        (head lst)
    (getMinElement (tail lst))))
)

(func removeElement (el lst)
(cond (equal (head lst) el) (tail lst) (cons (head lst) (removeElement el (tail lst))))
)

(func sort (lst)
(cond (isempty (tail lst))
    lst
(cons (getMinElement lst) (sort (removeElement (getMinElement lst) lst)) )
)
)

(setq sortedList (sort initialList))
sortedList


(func getMax (currentMax list)
    (cond (isempty list)
        currentMax
        (let 
            (
                (first (head list))
                (restList (tail list))
            )
            (cond (greater first currentMax)
                (getMax first restList)
                (getMax currentMax restList))
        )    
    )
)


(define superMaxElement Auto)
(setq superMaxElement (getMax (head sortedList) (tail sortedList)))

(functype getMaxElementTwice (Unit -> Auto))
(func getMaxElementTwice () (times 2 (getMax (head sortedList) (tail sortedList))))
```
