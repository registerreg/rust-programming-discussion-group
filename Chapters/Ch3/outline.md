# Ch3 Common Programming Concepts

covers concepts familiar to every programming language and how they work in Rust. topics: 
* variables
* basic types 
* functions
* comments
* control flow

like with most programming languages, there are some keywords that you can't use for variable or type names:
Rust Reserved Keywords: https://doc.rust-lang.org/book/appendix-01-keywords.html

## Variables and Mutability

Callback to ch2 for an important fact about Rust, variables are immutable by defaut. enables: 
* safety for developer (don't have to keep track of state mutations of a variable across codebase)
* easy concurrency (if two threads never mutate the same state, it is much much easier)

example: 
```Rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

running this with `cargo run` should result in the following error: 

```sh
$ cargo build
   Compiling no_type_annotations v0.1.0 (file:///projects/no_type_annotations)
error[E0282]: type annotations needed
 --> src/main.rs:2:9
  |
2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^ consider giving `guess` a type

For more information about this error, try `rustc --explain E0282`.
error: could not compile `no_type_annotations` due to previous error
```

this is the type of error you get when you try to reassign an immutable variable.

It is very hard to track down bugs that result from part of some codebase assuming that the value 
referenced by a variable will never change and then another part changing the value. It is 
even harder to track when the value changes only sometimes. The compiler enforces that values
don't change by default so that you, the developer, don't have to.

But Rust lets you override this default behavior with `mut` keyword. Here is the same example again
with `mut` which doesn't throw the compiler error: 

```Rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

output:
```sh
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6
```

another tradeoff that would make mutability more attractive is for large data structures. if you
had to create a new copy of part or all of the structure with every change, this could be a 
performance hit. However, with small data structures, immutability is probably still easier to reason
about with data structures. 

### Constants

using `const` instead of `let` makes your identifier a constant instead of a variable. Constants cannot
be mutable, `mut` not allowed, and they also must be declared with a type annotation. a few other things: 
* constants can be declared at any scope level, including global
* constants cannot be set to an expression that requires runtime evaluation, for how to follow this
  rule see https://doc.rust-lang.org/reference/const_eval.html

example of a constant:
```Rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

By convention, constant identifiers are all caps with underscores for spaces, UPPER_SNAKE_CASE.

As with other programming languages, constants are good for things that would otherwise be hardcoded, 
because they will have a good name and you will only have to change them in one place.

### Shadowing

Shadowing an assignment is possible in Rust, it works the way you might expect coming from other scoped
languages. The binding of a variable corresponds the declaration in the innermost scope and closest to its 
usage. for example: 

```Rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}
```

in the above example, the first time it is printed, x will be 12, then the second time it will go back to 6.

Shadowing almost looks like it is violating the compiler enforced default of immutability. Just note that 
in this example, `let` was used each time x was shadowed, so we were creating a new x, not mutating the value it 
held. Shadowing also lets you change the type of a variable when you shadow it again. This lets you use the 
same identifier to set up a value with one type and then use that to create a value of a different type and 
store it in a variable with the same name. `mut` doesn't let you change the type of a variable once it is declared,
only its value, so if you wanted to achieve this trick with mutable variables you could not.

## Data Types

### Functions

The function naming convention for Rust is `snake_case`. As with C family languages, you have argument lists in () 
parentheses, and you have curly braces enclosing function bodies that tell the compiler where the function's boundaries
are. 

an example function:
```Rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

### Parameters

Book mentions the formal definition of parameters as being the names used in the signature and then the term
arguments for the concrete values passed into the function when it is invoked, but then goes on to say that 
they are interchanged colloquially. Some rules about parameters: 
* the Rust compiler requires you to provide type annotations for your parameters in a function signature,
and this is because the type inference the compiler gets from parameter types allows it to infer the parameter's
types without specific declarations in most other places in the code. 
* you need to comma separate the parameter declarations in your function signature.

### Statements and Expressions

In Rust, functions are made up of a series of statements ending in an optional expression. 

Definitions of statement vs. expression
* Statements: instructions that perform an action and don't return any value
* Expressions: evaluate to some resulting value

example statement (the function definiton is a statement and the first line of the function is one too.): 
```Rust
fn main() {
    let y = 6;
}
```

since statments can't return values, so you can't use them on the right hand side of assignments, as is done here:
```Rust
fn main() {
    let x = (let y = 6);
}
```

Rust is an expression based language and so most lines will evaluate to some value. the examples of statements above
provide the central ways that statements are used, most of the rest of it will be expressions. 

In Rust, you can indicate that a line is an expression by not putting a semicolon at the end of it, which will cause it
to return a value. example of a scope block which makes up an expression:

```Rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

### Functions with Return Values

Rust returns the final expression of its function implicitly, though you can use the `return` keyword to do it early.
You specify the function's type with an `->` symbol. you can write a function whose body consists solely of a number, 
that will be treated as the expression that is to be implicitly returned: 

```Rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {}", x);
}
```

If you take an expression that is the last line of a function and put a semicolon on it, that tells the compiler
you want that line to be a statement, and if you have the type of the return value listed in the function signature
as something like `-> i32`, then the compiler will throw a type mismatch error indicating that you should have returned 
`()` which represents the unit type.

## Comments

comment syntax in Rust is `//` and from that point the comment continues to the end of the line

```Rust
// this is an example of a Rust comment
```

There are also documentation comments but they won't be covered until Chapter 14 of the book. 

## Control Flow

This covers conditional and loop constructs in Rust

### if Expressions

these are really similar to the way they look in C family languages, except no parenthesis around the conditional
which is more like python and other languages. 

```Rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

The blocks of code that happen depending on the outcome of the conditional are often called 'arms'.
Else blocks are optional, so you can have a lone if statement that just doesn't execute if its conditional
evaluates to false.

Rust does not have 'truthiness' like javascript, ruby and python. so the boolean expression you have in your conditional
must evaluate to a Bool typed result. The presence of something is not implicitly `true`, the absence not `false`.

for multiple branches of the conditional you have `else if` for everything between the if and the else. 

In chapter 6 they will introduce the `match` construct which is for more powerful alternatives to complex conditionals

### Using if in a let Statement

You can use an if on the right hand side of a let statement that assigns to a variable. when you do this, you put the
value that should be set on the variable in curly braces for both outcomes of the branch. example:

```Rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {}", number);
}
```

Note that the possible values to set for number in the above must both be the same type. If 6 had been "six" then
you would get a compiler error. This is because making it possible for a variable to be one of two different types
depending on somethng that might only be known at runtime is not allowed, the compiler must resolve the type of 
the variable assignment. 

### Repetition With Loops


