# Chapter 2: Programming a Guessing Game

A program that will generate a random integer between 1 and 100.  It will then prompt the player to enter a guess.  After a guess is entered, the program will indicate whether the guess is too low or too high.  If the guess is correct, the game will print a congratulatory message and exit.

```bash
$ cargo new guessing_game
$ cd guessing_game
```

main.rs (Part 1, User Input)
```Rust
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

To prompt and receive the response, we need the `io` library from the standard library (`std`).  By default, Rust has a few items defined in the standard library that it brings in scope in every program which is called the prelude.  If your type isn't in the prelude, you have to include it with `use`.

`let mut guess = String::new()` is defining a new variable to store the user input.  In Rust, variables are immutable by default (Chapter 3), but to make it mutable, you add `mut` before the name.

```Rust
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

`String::new()` is function that returns a new instance of a `String`.  The `::` syntax indicates that `new` is an associated function of `String` type.  _Associated functions_ are functions implemented on a type.  `new` creates a new, empty string.

`io::stdin` allows us to handle user input with a `Stdin` instance.  We can use `io` because we did `use std::io` but if we didn't, `std::io::stdin` would also work.  `.read_line(&mut guess)` calls the read_line method on the `Stdin` instance to get the input from the user.  Specifying `&mut guess` informs it where to store the user input.  `&` indicates that the argument is a reference.  References allows a way to let multiple parts of the code access one piece of data but they are immutable, which is why `&guess` wasn't used in favor of `&mut guess`.

`.expect("Failed to read line);`:  `read_line` will return `io::Result`, there are a few `Result` types, which are _enumerations_ (enums) which can have a fixed set of possibilities or _variants_.  `Result`'s variants are `Ok` or `Err`, `Ok` meaning it worked, `Err` meaning it did not and contains information as to why.  `io::Result` specifically has an `.expect` method that will be called with either variant, but when the variant is `Err`, `expect` will cause the program to crash and display the message passed as a parameter, but if it's `Ok` it will simply return the user input.

If there is no `expect`, the program will compile, but give a warning to encourage error handling with the `expect` method.

`println!("You Guessed: {}", guess);`: `{}` works as a placeholder, replacing with the value you put after the string. You can use more than one, and they will match in the order of curly brackets to values.

Running this first part of code:
```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

`rand` crate (package) has the code needed for the random number generation.  Add `rand` to the Cargo.toml file:

```toml
[dependencies]
rand = "0.8.3"
```

Specifying `"0.8.3"` will also mean `"^0.8.3"` for Cargo, which means it will use any version between `0.8.3` and `0.9.0`.  Run `cargo build` to build the project but to also acquire the `rand` crate.  It will get and compile not only the `rand` crate, but anything `rand` also depends on.  They will be downloaded from crates.io, which is the crates are posted from other developers.

Cargo.lock file can ensure that you will build the project with the same versions of dependencies.  This file is created the first time `cargo build` is run.  Unless you change the Cargo.lock file, the project will always use the `"0.8.3"` version of `rand`.

You can update your dependencies to the latest in the `^` range by using `cargo update`.  If you want to update past that range, you need to change it in your Cargo.toml file.

main.rs (Part 2, Generating Random Number)

```Rust
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

In order to use the new `rand` crate, we add `use rand::Rng`.

`let secret_number = rand::thread_rng().gen_range(1..101);`: First, `rand::thread_rng()` is a random number generator seeded by the operating system, which is then utilized by the second, `gen_range` to generate a random number within that range with the seed.  The parameter is in the form of `start..end` and inclusive on lower and upper so `1..101` ends up being a number between 1 and 100.

The added `println!` is simply there to ensure the random number generator works before we move on.  Running the program a few times with `cargo run` to make sure that the number is being generated and random is a good idea before moving on.

main.rs (Part 3, Comparing the Numbers)
```Rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // --snip--

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

A new `use` statement to add `std::cmp::Ordering`!  `Ordering` is an enum that has the variants `Less`, `Greater`, and `Equal`.  Then the five lines to attempt to `match` the `guess` and the `secret_number`.  The `.cmp` method compares the two numbers and returns a variant of the `Ordering` enum.  A `match` expression is made up of _arms_.  An arm consists of a pattern to match against and the code that should be run if the value given to match fits the arm's pattern.  So for instance, if the `cmp` method returns `Ordering::Less`, the `match` method goes down the `Ordering::Less` arm to `println!("Too small!)`, whereas with the other cases, it would go down those arms and perform the code in their respective arms.

If we were to compile here, it would not work since `secret_number` is a integer and `guess` is a `String`.  We can convert `guess` into a number by adding `let guess: u32 = guess.trim().parse().expect("Please type a number!");` after the `io::stdin` method.  In Rust, it is possible to define a new value on top of the same name as the old one by _shadowing_ (Chapter 3).  The `trim()` part will remove any newlines that the terminal might've added when the user hit the enter button.  `parse()` will attempt to parse the string into `u32` (unsigned, 32-bit integer).  Since it's highly possible this will error if anything but a number was entered, `parse()` has a `Result` type that we can use with `expect()` again.  If we did not get a number, `expect()` will crash the program again and output the string that is passed to the method.

Running the program now will work (but also make sure to check the new parsing by passing a letter or word), but it's only possible to guess once.

main.rs (Part 3, Looping)
```Rust
   // --snip--

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => println!("You win!"),
        }
    }
}
```

Now the program will loop constantly, but there's no exit.  Users can exit with  keyboard interrupt (ctrl-c) but adding a break for the `Ordering::Equal` arm will work better:

```Rust
        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

main.rs (Part 4, Handling Invalid Input)
```Rust
        // --snip--

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        // --snip--
```

Instead of an `expect` call, it can be a `match` and have two arms for the variants, `Ok` and `Err`.  Ok would simply return the number given, while `Err` would `continue` and simply loop for another guess.  The `_` underscore character is a catch all for all possible parameters.

main.rs (Final)
```Rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```
