Newt Language Design
====================

This document contains informal notes on the design of the language. Below
you can find a brief overview of the features, which are discussed in more
detail in their respective documents.

Principles
----------

Newt is supposed to be a teaching language. Therefore the design and 
development process are guided with the following principles in mind:

1. **Easy to get into.** No complicated configuration process should be
required to get into coding in Newt. Most that is allowed is a 
straightforward "Download and install" process that takes care of the basics 
for you. Ideally a web interface should be created for people who would 
just want to check it out.

2. **Fun and quick coding.** Process of writing programs in Newt should be
fun and quick. This means you should be able to create a simple animation
in as few lines of code as possible and a simple game in fewer than 100 lines.
You should never feel like you're not able to or don't know how to write
some basic things. This requires a standard library with a lot of basic
operations implemented and containing simple frameworks for pretty much 
anything a beginner would like to code.

3. **Least annoyance.** This is connected to the previous principle. The
tools should be annoying as little as possible. This means fast startup,
no distractions, fast compilation/interpretation etc.

4. **Intuition and consistency over familiarity.** Language constructs should 
be first of all easy to understand for a beginner and consistent with each
other. Similarity to other programming languages is less important.

Standard Library
----------------

* system calls, 
* simple CLI handling, 
* algorithms and data structures,
* basic graphics, input and audio for simple 2d animations and games,
* network library,
* http and database libraries

Program structure
-----------------

Here's a Hello World program written in Newt:

    use console

    program HelloWorld {
      console.writeln("Hello, world!")
    }

Variables and constants
-----------------------

Variables are created using the `var` keyword and a type annotation. If
an initial value is provided, the type annotation is optional.

    var a : int
    var x : real <- 1.5
    var y <- 9.75

Constants are created in a similar fashion, except using a `const` keyword,

    const pi : real <- 3.1415926535
    const phi <- 1.61

Primitive data types include: `int` (maximum-size integer), 
`real` (double-precision floating point), `bool` (`true`/`false`) and `string`.

Vectors
-------

Newt uses resizable arrays (vectors) as its main container type. Vectors are
created like normal variables. Vector type is denoted `[T]`, where `T` is
the type of each element. Vector literals are written with curly braces. 

    var primes : [int] <- {2, 3, 5, 7, 11, 13}
    var fib <- {1, 1, 2, 3, 5, 8, 13}

Individual elements in vectors can be accessed using the square-bracket
notation. Negative numbers and vectors of integers are allowed as subscript 
values.
  
    var numbers <- {4, 8, 15, 16, 23, 42}
    console.writeln(numbers[3]) // => 16
    console.writeln(numbers[-1]) // => 42
    console.writeln(numbers[{1, 3, 5}]) // => [8, 16, 42]
    console.writeln(numbers[range(2,4)]) // => [8, 15, 16]

Some basic operations on vectors are implemented as functions.

Conditionals
------------

If..else statements are similar to the ones in C-like languages, except
brackets around conditions are ommitted. Conditions can be combined using
`and`, `or` and `not` operators.

    if n % 15 == 0 {
      console.writeln('FizzBuzz')
    } else if n % 5 == 0 {
      console.writeln('Buzz')
    } else if n % 3 == 0 {
      console.writeln('Fizz')
    } else {
      console.writeln(n)
    }

Curly braces after `if` are required. They can be ommitted after `else` if
the following statement is an `if` statement (so that `else if` is allowed).

Loops
-----

There are 4 types of loop in Newt: `while`, `repeat`, `foreach`, `forever`.

`while` repeats a block of code as long as the condition is true. Brackets
around condition are not required, curly braces around the code block are.

    while n < 10 {
      console.writeln(n)
      n <- n+1
    }

`repeat` takes an expression which evaluates to an integer and repeats the
code block that many times. The number of iterations is calculated once,
before the loop is entered. Curly braces are, again, required.

    repeat 5 {
      console.writeln(5)
    }


    var n <- 3
    repeat 2*n { // repeat 6 times
      console.writeln(n)
      n <- n+1 // this doesn't change the number of repetitions
    }

`foreach` executes a code block for each element in a collection. Curly braces
are required.

    foreach i in {1, 2, 3} {
      console.writeln(i)
    }

`forever` repeats a code block until a `break` is encountered or the program
is closed (by `Ctrl-C`, `Alt-F4` or similar). Curly braces are required.

    forever {
      foreach e in entities {
        e.draw()
        e.update()
      }
    }

References
----------

References are pointers but with no pointer arithmetic. References are created
using the `&` operator, dereferencing happens with star operator and reference
type is annotated with a star as well, eg. `*int`.

Functions
---------

Functions are defined as follows:

    fun fact(n : int) -> int { // takes an integer n and returns an integer
      if n < 2 {
        return 1
      }

      return n * fact(n-1)
    }

The arrow and return type can be ommitted if nothing is returned.

    fun doSomething(n : int) {
      repeat n {
        console.writeln("Doing something")
      }
    }

Ideally, functions should be first-class objects.

    fun map(vals : [int], f : int -> int) -> [int] {
      var result : [int]
      foreach v in vals {
        result <- append(result, f(v))
      }

      return result
    }

Functions are pass-by-value, so you need to explicitly pass a reference
if you want to operate on the actual variable and not its copy.

Structs
-------

Structs consist of fields (variables) and optional methods (functions).

    struct Rectangle {
      var x, y, w, h : int

      fun area() -> int {
        return w*h
      }
    }

Fields and methods with names starting with `_` are hidden. This may not be
enforced in the early implementations, but probably will be in the final 
version.

Visible fields and methods are accessed using a dot operator, both for
variables and references:

    var rect <- Rectangle(0, 0, 10, 20)
    var rectRef <- &Rectangle(0, 0, 20, 30)

    console.writef("%d %d\n", rect.area(), rectRef.area()) // => 200 600

Two default constructors are created for each struct:

1. Empty constructor takes no arguments and invokes an empty constructor
for each field.

        var rect <- Rectangle() // or
        var rect : Rectangle
    
        console.writeln(rect.area()) // => 0

1. Value constructor takes as many arguments as there are fields in the
struct, with the same types, and uses the passed values to initialise fields.

        var rect <- Rectangle(100, 100, 50, 10)
    
        console.writeln(rect.area()) // => 500

New constructors can be defined as `_construct` methods. Note that the name
starts with an underscore, so the method itself is hidden and cannot be
invoked directly from outside. It is only accessible via the constructor syntax.

    struct Rectangle {
      var x, y, w, h : int

      fun _construct (width : int, height : int) {
        x <- 0
        y <- 0
        w <- width
        h <- height
      }

      fun area() -> int {
        return w*h
      }
    }

    program Rectangles {
      var r <- Rectangle(10, 30)
      console.writeln(r.area()) // => 300
    }

Generics
--------

Type abstractions will be available but the exact specifics are yet to be
decided.

Interfaces
----------

Would be nice to have, but no specifics are decided yet.

Object-oriented programming
---------------------------

Ideally this would be included in the language but there are no specifics
decided yet.

Modules
-------

They are required for the standard library to work the way I intend, so they
will be present. Currently the syntax for module definition looks like this:

    module math

    fun power(n : int, k : int) {
      res <- 1
      repeat k {
        res <- res * n
      }
    }

 Then if you wanted to use your module in a program, you would write

    use console
    use math

    program PowerExample {
      console.writeln(math.power(2, 5)) // => 32
    }

Name of the module is required if you import the entire module. However,
it is possible to import only selected parts of the module.

    use console.writeln
    use math.power

    program PowerExample2 {
      writeln(power(2,5)) // => 32
    }

Whether a wildcard use clause (e.g. `use console.*`) will be included is
yet to be decided.

You can combine several use clauses into one:

    use console, math.power

    program PowerExample2 {
      console.writeln(power(2,5)) // => 32
    }
