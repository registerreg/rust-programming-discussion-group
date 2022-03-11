# Chapter 4: Understanding Ownership

Ownership is the rules that dictate memory usage for Rust programs.  While some programs use garbage collection and others have the developers allocate the memory in the code, but Rust uses ownership instead to manage memory.  If any of the ownership rules are broken, the compiler won't work and ownership is designed to not slow down operation.

Ownership is difficult to understand at the start, but developers get used to the concepts eventually.

## Stack and Heap
System programming languages like Rust require you to think about stack and heap and whether a value is on the heap or stack determines how the operations work.  The stack and heap are parts of memory available at runtime.  The stack stores values in the order they were received and removes them in a LIFO fashion. Adding to the stack is called pushing and removing is called popping.

The heap is not as organized, when you request memory from the heap, the memory allocator finds space and returns a pointer, which is called allocating.  Since the memory in the heap is fixed and has an address (pointer), you can put that pointer on the stack.  Pushing to the stack is faster than heap because space doesn't need to be found in the heap and retrieving is also slower since you have to follow a pointer.

Managing the heap and stack are things that the ownership does.

## Ownership Rules
- Each value in Rust has a variable that’s called its owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

## Variable Scope

`let s = "hello";`

`s` is a string literal, the variable is valid from the point it's defined (into scope) to end of scope (out of scope).

### String Type
All types previously mentioned were of static sizes which could be added and removed from stack easily, but String type can be a good explainer for values on heap.  You use String in instances where you know a string is not a literal and can be dynamic, like taking user input.

`let s = String::from("hello");`

Literals are able to be hardcoded because they don't change which makes them fast and efficient.  Non-literal Strings though are mutable and since they can change size, they need to be stored on the heap instead.

- The memory must be requested from the memory allocator at runtime.
- We need a way of returning this memory to the allocator when we’re done with our String.

The first part is done when we call String::from.  The second part would typically be done by a garbage collector but there's issues with GCs, like forgetting to call them and wasting memory.  Instead, Rust frees that memory the moment the String goes out of scope.  Rust calls a special function called `drop` when the scope ends, but it's also possible for developers to call that directly.

```Rust
let x = 5;
let y = x;
```
Both of these variables would end up on the stack because they are known to be fixed length

```Rust
let x = String::from("hello");
let y = x;
```
In this instance, hello data would be stored in the heap and the variables would be pointers to that heap location with size and capacity as well.  Particularly in this case as well, both pointers would point to the same location in the heap because Rust does not copy the heap.

At the end of scope, both variables would be attempted to be dropped which is called a double free error because freeing memory twice can cause corruption.  To ensure safety, after y is declared, x no longer exists so only y is dropped at end of scope.

Instead of shallow copy or deep copy in other languages, Rust does what's called a move, which basically means the data heap is no longer associated to x pointer, but only y pointer.  Rust will never deep copy automatically so the copying of a variable is actually inexpensive performance wise.

If you do want to deep copy, you can use the `clone` method.

```Rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

What's interesting is that in the initial integer copy, even though we'd assume the x variable would no longer exist due to double free avoiding, it actually does because integers are fixed length and Rust doesn't bother worrying in that case.

Instead of using Drop in those cases, Copy is available instead and automatically done by Rust.  You can see some types below that use Copy automatically:

- All the integer types, such as u32.
- The Boolean type, bool, with values true and false.
- All the floating point types, such as f64.
- The character type, char.
- Tuples, if they only contain types that also implement Copy. For example, (i32, i32) implements Copy, but (i32, String) does not.

## Ownership and Functions
Passing a variable to a function is the same as assigning a variable a value, so it will essentially move or copy automatically.

```Rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

## Return Values and Scope
```Rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

It is possible to use a value without transferring ownership by passing tuples, but it is better to use references.  You can see an example of the tuples implementation, but it's too much for something that's been solved by references:
```Rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

## References and Borrowing
The issue with the above tuple example is that we have to return the `String` to the calling function so we can still use the `String` after the call to `calculate_length`, because the `String` was moved into `calculate_length`.  Instead we can not take ownership and instead just read the data in the heap using a reference, which is a read-only pointer to the pointer which points to the data.

```Rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, nothing happens.
```

In this example, the ampersands indicate references which will have its pointer point to the owned pointer.  It is also possible to dereference using `*` instead but that will be discussed later in the book.

Since calculate_length doesn't have ownership of s, it won't drop the variable because it's just a reference.  This is called borrowing.  If you try to change a mutable variable that is a reference, by default Rust will error stating that it cannot be changed due to the variable being prefixed by &.

However, with a few tweaks, we can create a mutable reference by specifying the variable as a &mut reference.  But you can only have one mutable reference to specific data at a time.  This is to avoid data races at compile time:

- Two or more pointers access the same data at the same time.
- At least one of the pointers is being used to write to the data.
- There’s no mechanism being used to synchronize access to the data.

```Rust
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
```

A reference's scope is available from the moment it's created to the last time it's used so the below example won't cause an issue:

```Rust
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!("{}", r3);
```

## Dangling References
In other languages, it's possible to have a dangling pointer, but in Rust, it's not possible to have a dangling reference because Rust makes sure that data in the heap is available until after the last use of the reference.

```Rust
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```
The solution to the above is to just return s, not a reference.

## Slices
Slices let you reference a contiguous sequence of elements in a collection rather than the whole collection. Lets say you are trying to solve for returning the first word in a sentence, but if that sentence only has one word, return the whole string.

```Rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word will get the value 5

    s.clear(); // this empties the String, making it equal to ""

    // word still has the value 5 here, but there's no more string that
    // we could meaningfully use the value 5 with. word is now totally invalid!
}
```

`enumerate` wraps the iter to return a tuple where first element is index and second is a reference to the element.  Because we now have a tuple, we can be destructive with the tuple.  We can now search through all these tuples for the one that is a space.  This implementation means that word and s are not related at all so word can still exist.

## String slice
Instead, we can use a String slice to reference part of a string.

```Rust
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
```

Where s has a normal pointer with size and capacity, the slice is actually a pointer with just size and it points to a different location in the data.

So with slice instead, we can do this:

```Rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```
