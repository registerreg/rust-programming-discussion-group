# Chapter 5: Using Structs to Structure Related Data

## Defining and Instantiating Structs

Structs are like Rust's tuples, because they both can hold multiple, related values of different types.
Structs give names to their values though and they don't rely on the declaration order to access these values.

The structure is familiar, just a collection of fields and their values with type information. 

```Rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {}
```

Then, to declare an instance of the struct, 

```Rust
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
}
```

You access and (if the instance is mutable) mutate fields of the struct with dot notation. *Note* struct instances
are either completely immutable or completely mutable. You can't have an immutable struct with some mutable fields.

### Using the Field Init Shorthand

When setting values for fields with variables where the name of the variable is the same as the name of the field, rust lets you leave out the field name in your instantiation. 

```Rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}

fn main() {
    let user1 = build_user(
        String::from("someone@example.com"),
        String::from("someusername123"),
    );
}
```

### Creating Instances From Other Instances With Struct Update Syntax

In something that looks very much like the 'spread' operator in javascript or the 'splat' 
operator in python but with a twist, `...` in rust lets you copy over fields from another instance. However,
it can also notice that you have fields of the same name in your list and only give you the fields you didn't
name. 

```Rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    // --snip--

    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

*callback to ownership chapter* see when user2 gets instantiated with user1, this is done as a 'move'
and so the compiler won't let you use user1 after that point. This is because we didn't assign new values of both
of the String objects in user1 for user2. If the boolean and int values were the only values these objects had in
common, user1 would still be usable.

### Using Tuple Structs without Named Fields To Create Different Types

Sometimes you don't need names, because they are obvious. RGB color values or x,y,z coordinate holding objects
are examples of this. in this case you just use types and leave out the names, and when instantiating them
it looks more like a tuple. These structs essentially work like tuples except that you can't assign an instance
of one of these structs to a variable that was defined as another tuple struct type.

```Rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

### Unit Structs Without Fields

This is introduced early and its facility will become clearer later in the book. Structs without 
fields are useful when you want to make a trait with no data, just behavior. 

```Rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

*Back to Ownership again* another callout to something that will make more sense later. Structs need to use owned types
in their declaration. The compiler will complain if we use references like a slice type. Because without the concept of Lifetimes the compiler can't ensure that the data stays valid as long as the struct is valid, in order to do its fancy 
memory management rust needs to have this restriction on what types can be in a struct. 

```Rust
struct User {
    active: bool,
    username: &str,
    email: &str,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
}
```

```sh
$ cargo run
   Compiling structs v0.1.0 (file:///projects/structs)
error[E0106]: missing lifetime specifier
 --> src/main.rs:3:15
  |
3 |     username: &str,
  |               ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     active: bool,
3 ~     username: &'a str,
  |

error[E0106]: missing lifetime specifier
 --> src/main.rs:4:12
  |
4 |     email: &str,
  |            ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     active: bool,
3 |     username: &str,
4 ~     email: &'a str,
  |

For more information about this error, try `rustc --explain E0106`.
error: could not compile `structs` due to 2 previous errors
```

## An Example of a Program Using Structs

Area of a rectangle is the use case. 

```Rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

Main problem here, area function takes width and height separately but in reality they are 
two attributes of a rectangle. So we could make this more apparent with a tuple: 

```Rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

now width and height are combined into dimensions, but we don't know which is which or what they
represent because they are unnamed unsigned 32 bit integers. We can do better. 

```Rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

this is the most clear about what we a taking area of and what the
components are. Also notice we pass the instance of the rectangle struct by
reference to 'borrow' it (ownership terms) and it is immutable since we 
don't need to mutate height or width.

### Adding Useful Functionality With Derived Traits

Programmer defined structs won't implement the display formatter which is 
the default method used by the println! macro. You need another way to print. 
If you try to use prinln! the compiler will tell you to try `:?` for pretty 
print. This won't work either, you need to enable the `Debug` trait first.

```Rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    // println!("rect1 is {}", rect1 throws compile error)
    // and so does the below statement without the outer 
    // attribute #[derive(Debug)] added above
    println!("rect1 is {:?}", rect1);
}
```

adding `:#?` istead of `:?` will additionally style the output with whitespace and
new lines for larger structs to be more readable.

An alternative is the `debug!` macro which will temporarily take ownership of the expression
that is passed to it and then print the file and line number and expressoin and then return ownership. 
Because debug! only temporarily takes ownership, you can use debug! inline on the right hand side of an 
assignment and it won't affect how the expression is used after debug! is done with it. 

```Rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```

will have the output: 

```sh
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/rectangles`
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```

## Method Syntax

 