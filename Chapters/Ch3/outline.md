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






