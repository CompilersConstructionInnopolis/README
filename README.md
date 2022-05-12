# Team C++
- **Mikhail Martovitsky** - implement a standard library of predefined functions
- **Roman Soldatov** - language features developer
- **Timur Nugaev** - tester & test writer
- **Daniil Livitin** - language features developer

# Language Plus++

- Functional Lisp-like language with static type system. 
- The language is not purely functional. E.g. variables might change their values inside the function call.

# Build project

***Instructions for building the project***

 
1. Сlone a repository of our project.
2. Navigate to the folder of the project using the terminal.
3. Run the command `mvn clean install`. This command will create a ‘target’ folder and an executable .jar file inside.
 
***Instructions for using the prototype***

1. Create a build of the application (see previous step)
2. Navigate to the created ‘target’ folder.
3. Run a command: `java -jar Compilers_Innopolis-1.0-SNAPSHOT.jar ../Tests/program.txt`
	
> Note: before running a command you need to enter a code of the program inside Tests/program.txt file.


# Interpreter

In the [interpreter](https://github.com/CompilersConstructionInnopolis/ACCPA/tree/main/src/main/java/interpreter) package there are classes related to AST interpretation.

* [AtomsTable.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/AtomsTable.java) saves defined atoms and their values. This class is represented as a singleton. Atoms are stored in some local context. Atoms with similar name shadows atoms from outer scope. Each local context is perpesented as a HashMap. The hirerachy of local contexts is presented as a Stack. When you enter into some function or loop the interpreter calls `introduceLocalContext()` function to create a new temporary local context which will be deleted after leaving the form (`leaveLocalContext()` will be called).
* [FunctionsTable.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/FunctionsTable.java) is similar to [AtomsTable.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/AtomsTable.java), but it stores defined user-functions.
* [PredefinedFunction.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/PredefinedFunction.java) and [DefinedFunction.java](https://github.com/CompilersConstructionInnopolis/ACCPA/blob/main/src/main/java/interpreter/DefinedFunction.java) classes are utility functions to interpret a list as a function call and evaluate it.

# Grammar

```
Program : Element { Element } 
List : ( Element { Element } ) 
Element : Atom | Literal | List 
Atom : Identifier
Literal : [+|-] Integer | [+|-] Double | Boolean | String | null 
Identifier : Letter { Letter | DecimalDigit }
Letter : Any Unicode character that represents a letter 
Integer : DecimalDigit { DecimalDigit }
DecimalDigit : 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 
Double : Integer . Integer
Boolean : true | false
String: " Characters "
Characters: { Letter }
```

# Type Checker

## Check whether the typing is correct
- For the `function` we should check if the provided arguments are the same types as declared in the `functype` keyword.
If the argument is not Base type then we should recursively get the type of argument.
- For the variable we can use `define` keyword to specify variable's type.


## Infer the type of expressions

In our language type annotation in the source code is optional. 
If type is not specified explicitly then it should be inferred by interpreter automatically. 


# Base Types

Language has general-purpose types that are built into it:
* Numeric types (`Int`, `Double`, `Num` - parent class of `Int` and `Double`)
* Boolean type (`Boolean`)
* Unit type (`Unit`). It indicates that a function will return nothing. 
* String type (`String`). Any symbols
* Homogeneous lists
* Heterogeneous tuples. The tuple should have more than 1 element.


# User-defined Terms and Types

* User can define aliases with a use of primitives.
```
(type Seconds Int)
(type Minutes Int)

(functype GetMinutes (Int -> Int))
(func GetMinutes (seconds) (divide seconds 60))
```

# Standard Library

Our standart library will be substituted by such definitions:
* `map` - Type is: `((Any -> Any) (Any) -> (Any))`
* `filter` - Type is: `((Any -> Boolean) (Any) -> (Any))`
* `foldl` - Type is: `((Any Any -> Any) Any (Any) -> (Any))`
* `length` - Type is: `((Any) -> Int)`
* `isempty` - Type is: `((Any) -> Boolean)`

To import standart library you need to write in source code:

`(import standart.txt)`

After that you can use aforesaid functions in your code.

# Nested definitions

Our language support nested definitions. It means that it is possible to define a local variable/function in any scope.

Example:

```
(setq x 55)
(setq y 55)
(prog (x y) (
    (func helper () 42)
    (setq x (helper))
    (setq y 6)
    (x y) ; (42 6)
))
(x y) ; (55 55)
```

# Simple Constraint-Based Type Inference

Reserved word `auto` is used for specifying ommitted types

```
(functype sum (auto auto -> auto))
(func sum (a b) (plus a b))
(sum 5 4)
(sum 5.5 4) ; Error?
(sum 4.4 2.3) ; Error?
```

# Typing rules

* Type of the function : 
Everything before `->` is types of arguments of function
Everything after `->` is returned type of function
```
(functype sum (Num Num -> Num)) ; function takes 2 arguments of type Num and return type Num

(functype example ((Num -> Num) -> Num)) ; argument of the function is a function that takes Num and return Num and returned type of the function is Num
```







Comment lines `;`
```
; Everything after semicolon is a comment on this line
; The compiler will ignore this part of code 
```

Create a variable `setq`
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
(setq x a); now x has value 8
```

`define` keyword
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
The syntax: (`functype` (`list of arguments types` -> `return type`)).
You can declare the `functype` for a particular function only once above the `func`.
```
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
; The function accepts an argument n and increments outerscope variable x by n. The function itself returns nothing
```

```
(setq x 3)
(functype sideEffect (Unit -> Unit))
(func sideEffect ()
    (setq x (plus x 1))
)
; The function accepts nothing and returns nothing. It just increments outerscope variable x
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
Result for `true` and `false` branches must have the same type.
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

Lists. List is homogeneous.
If the first element is a function name then such list is considered as a function call, the rest list's elements are function arguments.

```
(plus 1 2) ; this is a function call

(1 2 3) ; this is a list of integers

; (1 5.23 true) compile error. List must be homogeneous

() ; empty list

(define x (Int)) ; x is a list of integers
(setq x (1 2 3)) ; now x is equal to (1 2 3)
; (setq x (true false)) complie error since x is a list of integers, not booleans
```

For function arguments we denote a list as `(T)` where `T` - is the list type.
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
                (setq first (head list))
                (setq restList (tail list))
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
(getI x 2) ; returns 2.2
(setI x 2 4.4) ; now x = (3, 4.4, 30)
```

Currying function
```
(functype moreThanX (Int (Int) -> (Int)))
(func moreThanX (x lst) (
	(cond (isepmty lst) 
        ()
        (let 
            (
                (setq first (head lst))
                (setq restLst (tail lst))
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
```