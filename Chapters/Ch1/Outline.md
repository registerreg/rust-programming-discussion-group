# Introduction and Chapter 1: Getting Started

Rust challenges the traditional separation of low-level and high-level programming by balancing powerful technical capacity and great developer experience while also allowing the option to control low-level details.

Ideal for teams of developers with various programming experience due to the compiler's gatekeeper role by refusing to compile elusive bugs.  More logic programming, less bug chasing.

Also includes Cargo, a dependency manager and build tool, Rustfmt, a code style tool, and the Rust Language Server which allows for IDE integration for code completion and error messages.

Ideal for students for many of the same reasons but also due to the topics such as operating systems development that is covered by Rust and the Rust Programming Language book.

Ideal for companies for a wide variety of solutions, such as command line tools, web services, DevOps tooling, embedded devices, audio and video analysis and transcoding, cryptocurrencies, bioinformatics, search engines, IoT, applications, machine learning, and major parts of Firefox.

Ideal for open source developers and encouraged to contribute to the language.

Ideal for speed and stability.  Not only the speed in which you can write a program, but the speed of the language itself.  Through the compiler, stability can be ensured during feature additions and/or refactoring.

Book assumes you've written code in another programming language before.

## Chapter 1: Getting Started

- Installing Rust on Linux, MacOS, and Windows
- Writing a "Hello, World!"
- Using cargo, Rust's package manager and build system

## 1.1: Installation

Installing on Linux or MacOS:
```bash
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Command will download a script to install the `rustup` tool, which installs the latest version of Rust.  If successful, `Rust is installed now. Great!` will appear.

Running a script directly from curl is risky though, so you can download via curl and run the script yourself later.

You can also use other system package managers, such as Homebrew, MacPorts, or pkgsrc for MacOS, Chocolatey or Scoop for Windows, and likely, apt or yum for Linux.

Installing on Windows:
Go to https://www.rust-lang.org/tools/install

For all operating systems, you will need a C compiler if you don't already have it, such as xcode for MacOS (`xcode-select install` to install) or C++ build tools for Visual Studio (Build Tools for Visual Studio 2019, ensure C++ build tools is selected)

`rustup update` to update and `rustup self uninstall` to uninstall

To confirm you have Rust installed, use `rustc --version` and it should return the version of Rust you have installed.  If not and you are certain you installed, it also might be that Rust is not in your PATH and you will need to add it.

Rust includes a copy of the documentation if you need to read it online/locally, `rustup doc`

## 1.2 Hello, World!

Start by making the directory:
```bash
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Make a new file called `main.rs`. Rust file extension is `.rs`.  Rust (community standard or compiler?) also expects underscores in filenames if using more than one word (`hello_world.rs`)

```Rust
fn main() {
    println!("Hello, world!");
}
```

To run this new file:

```bash
$ rustc main.rs
$ ./main
Hello, world!
```

### Parts of the code

Defining a function in Rust:
```Rust
fn main() {

}
```
`main` specifically is the first function that runs in every executable Rust program.  There are no parameters in this example, but if there were parameters, they'd go in the parenthesis.  Function body is wrapped in curly brackets, typically with a space between the parenthesis and curly bracket and first curly is on the same line.  However, if you or your team like some other style, you can use `rustfmt` to enforce that style.

Function body:
```Rust
    println!("Hello, world!");
```

Rust style is to indent with four spaces, no tabs.  `println!` is a Rust macro, if it were a function instead, there would be no exclamation mark.  Next is what we pass to the macro, which is "Hello, world!" string and then end with a semicolon.  Most lines in Rust will end in a semicolon to indicate the expression is over.


### Running the code
```bash
$ rustc main.rs
```
compiles the code by using `rustc` and the name of the source file.  This is similar to `gcc` or `clang` for C or C++ developers.  After compiling successfully, Rust outputs a binary executable.

You can see the executable by running `ls` in the command line in the same folder you are running `rustc`.

```bash
$ ./main # or .\main.exe on Windows
```
runs the binary executable, which runs the main function and outputs the string.

Rust is an _ahead-of-time compiled_ language, which means it has to be compiled before run, which is different than some other languages like Ruby, Python, or Javascript.

## 1.3 Hello, Cargo!
Cargo is Rust's build system and package manager.  Typically used to manage the project due to its extensibility.  Since the Hello, World! code didn't have any dependencies, Cargo would only be able to help in the building aspect.

To make sure you have an updated version and `cargo` is available in your PATH:
```bash
$ cargo --version
```

To make a new project with `cargo`:
```bash
$ cargo new hello_cargo
$ cd hello_cargo
```

Cargo will generate two files, a _Cargo.toml_ file and a _src_ directory with a _main.rs_ file inside.

It will also initialize a new Git repository for the folder with a _.gitignore_ file.

### Cargo.toml
```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2018"

[dependencies]
```

The file is in the TOML format, which is Cargo's configuration format.

`[package]` is the first header of the file and contains the first required values for Cargo to compile a project, the name, version of the project, and edition of Rust to use.

`[dependencies]` is where your dependencies will be listed as you add them.  In Rust, packages are called _crates_.

### src/main.rs

Cargo will put a Hello, world! program in the `src/main.rs` as the initial file in the `src` directory.  Cargo expects all source files to be in the `src` directory and the top-level directory to hold files such as README, LICENSE, etc.

### Running with Cargo

From the top level directory of the project, run `cargo build`.  Cargo will create an executable file in `target/debug/hello_cargo` which you can run with `./target/debug/hello_cargo`.

You can also run with `cargo run` instead.  If there are changes to the source code, running `cargo run` will also have cargo build the project again before running.

There's also `cargo check` which is like a dry-run, it will compile and check for any issues, but won't make a new executable.  You can run this while you're developing to continually check if any issues appear.

When you are ready to build for production, `cargo build --release` will compile with optimizations.  The executable will also be in `target/release` instead of `target/debug`

Cargo has its own separate documentation as well that is worth reviewing for more information.
