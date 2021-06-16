---
description: 'https://doc.rust-lang.org/book/title-page.html'
---

# The Rust Programming Language

## 1. Getting Started

### 1.2 Hello, World!

If you’re using more than one word in your filename, use an underscore to separate them.

Rust style is to indent with four spaces, not a tab.

Using a `!` means that you’re calling a macro instead of a normal function.

Most lines of Rust code end with a semicolon.

Rust is an _ahead-of-time compiled_ language, meaning you can compile a program and give the executable to someone else, and they can run it even without having Rust installed. 

### 1.3 Hello, Cargo!

In Rust, packages of code are referred to as _crates_.

```text
cargo new
cargo build [--release /* make your Rust code run faster */]
cargo run
cargo check // much faster than cargo build
cargo update
cargo doc --open
```

## 2. Programming a Guessing Game

`let` statement, which is used to create a _variable_.

Variables, references are immutable by default.

Use `mut` before the variable name to make a variable mutable.

An associated function is implemented on a type, rather than on a particular instance.

The `&` indicates that is a _reference_, which gives you a way to let multiple parts of your code access one piece of data without needing to copy that data into memory multiple times.

An enumeration is a type that can have a fixed set of values, and those values are called the enum’s _variants_.

_binary crate_, which is an executable. _library crate_, which contains code intended to be used in other programs.

{% embed url="https://github.com/StoneYunZhao/the-rust-programming-language/tree/master/guessing\_game" %}

## 3. Common Programming Concepts

### 3.1 Variables and Mutability

By default variables are **immutable**.

```rust
let x = 5; // immutable
let mut y = 6; // mutable


// Rust’s naming convention for constants is to 
// use all uppercase with underscores between words.
const MAX_POINTS: u32 = 100_000;
```

Like immutable variables, **constants** are values that are bound to a name and are not allowed to change. But there are some difference:

* Not allowed to use `mut` with constants.
* When using the `const` keyword, the type of the value _must_ be annotated.
* Constants can be declared in any scope, including the global scope.
* Constants may be set only to a constant expression, not the result of a function call or any other value that could only be computed at runtime.

You can declare a new variable with the same name as a previous variable. That means the first variable is _**shadowed**_ by the second, which means that the second variable’s value is what appears when the variable is used. 

```rust
let x = 5; // shadowed by the second x
let x = x + 1;

let spaces = "   ";
let spaces = spaces.len(); // type changed
```

Shadowing is different from marking a variable as `mut`. By using `let`, we can perform a few transformations on a value but have the variable be immutable after those transformations have been completed. We can change the type of the value but reuse the same name.

### 3.2 Data Types

Rust is a _**statically typed**_ **language**, which means that it must know the types of all variables at compile time. The compiler can usually **infer** what type we want to use based on the value and how we use it. In cases when many types are possible, we must add a type annotation.

A _**scalar**_ **type** represents a single value. Rust has four primary scalar types: **integers, floating-point numbers, Booleans, and characters**.

| Length | Signed | Unsigned |
| :---: | :---: | :---: |
| 8-bit | `i8` | `u8` |
| 16-bit | `i16` | `u16` |
| 32-bit | `i32` | `u32` |
| 64-bit | `i64` | `u64` |
| 128-bit | `i128` | `u128` |
| arch | `isize` | `usize` |

Each signed variant can store numbers from -2^\(n - 1\) to 2^\(n - 1\) - 1 inclusive. Unsigned variants can store numbers from 0 to 2^\(n - 1\).

When you’re compiling in debug mode, Rust includes checks for integer overflow that cause your program to _panic_ at runtime if this behavior occurs. When you’re compiling in release mode with the `--release` flag, Rust does _not_ include checks for integer overflow that cause panics.

Rust’s **floating-point types** are `f32` and `f64`. The **default type** is `f64` because on modern CPUs it’s roughly the same speed as `f32` but is capable of more precision.

A **Boolean type** in Rust has two possible values: `true` and `false`. Booleans are one byte in size.

Rust’s `char` type is the language’s most primitive alphabetic type. `char` literals are specified with single quotes. Rust’s `char` type is four bytes in size and represents a Unicode Scalar Value.

_**Compound types**_ can group multiple values into one type. Rust has two primitive compound types: **tuples and arrays**.

A **tuple** is a general way of grouping together a number of values with a variety of types into one compound type. Tuples have a fixed length: once declared, they cannot grow or shrink in size.

We can use pattern matching to destructure a tuple value. We can access a tuple element directly by using a period \(`.`\) followed by the index of the value we want to access. 

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);

let tup = (500, 6.4, 1);
let (x, y, z) = tup; // pattern matching

let five_hundred = tup.0;
```

Unlike a tuple, every element of an **array** must have the same type. Arrays in Rust have a **fixed length**, like tuples. Arrays are useful when you want your data **allocated on the stack** rather than the heap.

A **vector** is a similar collection type provided by the standard library that _is_ allowed to grow or shrink in size.

When you attempt to access an element using indexing, Rust will check that the index you’ve specified is less than the array length at runtime.

```rust
let a = [1, 2, 3, 4, 5];
let a: [i32; 5] = [1, 2, 3, 4, 5];
let a = [3; 5]; // [3, 3, 3, 3, 3]

let first = a[0];
let second = a[1];
```

### 3.3 Functions

Rust code uses _**snake case**_ as the conventional style for function and variable names. In snake case, all letters are lowercase and underscores separate words.

In function signatures, you _must_ declare the type of each parameter.

Function bodies are made up of a series of statements optionally ending in an expression. Rust is an **expression-based language**.

_**Statements**_ are instructions that perform some action and do not return a value. _**Expressions**_ evaluate to a resulting value.

Function definitions are statements. Calling a function is an expression. Calling a macro is an expression. The block that we use to create new scopes, `{}`, is an expression. **Expressions do not include ending semicolons**.

```rust
let y = 6; // statement
let x = (let y = 6); // compile error

let y = {
    let x = 3;
    x + 1 // NOTICE: without a semicolon at the end
};
```

The return value of the function is synonymous with the value of the final expression in the block of the body of a function. You can return early from a function by using the `return` keyword and specifying a value.

```rust
fn plus_one(x: i32) -> i32 {
    x + 1
}
```

### 3.4 Comments

```rust
// comments
```

### 3.5 Control Flow

`if` is an expression. You must be explicit and always provide `if` with a Boolean as its condition. The values that have the potential to be results from each arm of the `if` must be the same type.

```rust
let number = if condition { 5 } else { 6 };
```

Rust has three kinds of loops: `loop`, `while`, and `for`.

The **`loop`** keyword tells Rust to execute a block of code over and over again forever or until you explicitly tell it to stop. 

You can add the value you want returned after the `break` expression you use to stop the loop; that value will be returned out of the loop so you can use it.

```rust
let result = loop {
    counter += 1;

    if counter == 10 {
        break counter * 2;
    }
};

while number != 0 {
    println!("{}!", number);

    number -= 1;
}
```

You can use a `for` loop and execute some code for each item in a collection.

```rust
let a = [10, 20, 30, 40, 50];

for element in a.iter() {
    println!("the value is: {}", element);
}

// range over 3, 2, 1
for number in (1..4).rev() {
    println!("{}!", number);
}
```

## 4. Understanding Ownership

Ownership enables Rust to make memory safety guarantees without needing a garbage collector.

### 4.1 What is Ownership?

Memory is managed through a system of ownership with a set of rules that the compiler checks at compile time. None of the ownership features slow down your program while it’s running.

The **stack** stores values in the order it gets them and removes the values in the opposite order. All data stored on the stack must have a known, fixed size. Pushing to the stack is faster than allocating on the heap. Accessing data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there.

**Ownership rules**:

* Each value in Rust has a variable that’s called its _owner_.
* There can only be one owner at a time.
* When the owner goes out of scope, the value will be dropped.

A **scope** is the range within a program for which an item is valid.

The memory is automatically returned once the variable that owns it goes out of scope.

Rust calls `drop` automatically at the closing curly bracket.

```rust
let s1 = String::from("hello");
let s2 = s1; // s1 moved into s2


println!("s1: {}", s1) // compile error because s1 is invalid
```

Rust will never automatically create “deep” copies of your data. If we _do_ want to deeply copy the heap data of the `String`, not just the stack data, we can use a common method called `clone`.

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

Types such as integers that have a known size at compile time are stored entirely on the stack.

Rust has a special annotation called the `Copy` trait that we can place on types like integers that are stored on the stack. If a type implements the `Copy` trait, an older variable is still usable after assignment. Rust won’t let us annotate a type with the `Copy` trait if the type, or any of its parts, has implemented the `Drop` trait. Any group of simple scalar values can implement `Copy`. Here are some of the types that implement `Copy`:

* All the integer types, such as `u32`.
* The Boolean type, `bool`, with values `true` and `false`.
* All the floating point types, such as `f64`.
* The character type, `char`.
* Tuples, if they only contain types that also implement `Copy`. For example, `(i32, i32)` implements `Copy`, but `(i32, String)` does not.

The semantics for passing a value to a function are similar to those for assigning a value to a variable. Passing a variable to a function will **move** or **copy**. 

Returning values can also transfer ownership.

The ownership of a variable follows the same pattern every time: assigning a value to another variable moves it. When a variable that includes data on the heap goes out of scope, the value will be cleaned up by `drop` unless the data has been moved to be owned by another variable.

### 4.2 References and Borrowing

_**References**_ allow you to refer to some value without taking ownership of it.

The opposite of referencing by using `&` is _dereferencing_, which is accomplished with the dereference operator, `*`. 

When functions have references as parameters instead of the actual values, we won’t need to return the values in order to give back ownership, because we never had ownership.

We call having references as function parameters _**borrowing**_. 

Just as variables are immutable by default, so are references. We’re not allowed to modify something we have a reference to.

You can have only one **mutable reference** to a particular piece of data in a particular scope. The benefit of having this restriction is that Rust can prevent data races at compile time.

```rust
// compile error
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;
} // r1 goes out of scope here, so we can make a new reference with no problems.

let r2 = &mut s;
```

We _also_ cannot have a mutable reference while we have an immutable one.

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM

println!("{}, {}, and {}", r1, r2, r3);
```

A reference’s scope starts from where it is introduced and continues through the last time that reference is used.

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// r1 and r2 are no longer used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```

The compiler guarantees that references will never be dangling references: if you have a reference to some data, the compiler will ensure that the data will not go out of scope before the reference to the data does.

```rust
// compile error: this function's return type contains a borrowed value, 
// but there is no value for it to be borrowed from.
for it to be borrowed from.
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle returns a reference to a String
    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.


// solve method
fn no_dangle() -> String {
    let s = String::from("hello");

    s
} // Ownership is moved out, and nothing is deallocated.
```

### 4.3 The Slice Type

Another data type that does not have ownership is the _slice_. Slices let you reference a contiguous sequence of elements in a collection rather than the whole collection.

A _string slice_ is a reference to part of a `String`. Internally, the slice data structure stores the starting position and the length of the slice. String slice range indices must occur at valid UTF-8 character boundaries.

```rust
let s = String::from("hello world");

let s1 = &s[0..5];
let s2 = &s[..6];
let s3 = &s[7..];
let s4 = &s[..];
```

String literals is stored inside the binary. The type of `s` here is `&str`: it’s a slice pointing to that specific point of the binary.

```rust
let s = "Hello, world!"; // immutable reference.
```

Defining a function to take a string slice instead of a reference to a `String` makes our API more general and useful without losing any functionality.

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3]; // type: &[i32]
```

## 5. Using Structs to Structure Related Data

A _struct_, or _structure_, is a custom data type that lets you name and package together multiple related values that make up a meaningful group.

### 5.1 Defining and Instantiating Structs

Rust doesn’t allow us to mark only certain fields as mutable.

Tuple structs have the added meaning the struct name provides but don’t have names associated with their fields; rather, they just have the types of the fields.

_**unit-like structs**_ ****behave similarly to `()`, the unit type. 

```rust
fn main() {
    let mut user1 = User {
        username: String::from("someone@example.com"),
        active: true,
    };

    user1.username = String::from("anotheremail@example.com");

    let user2 = User {
        username: String::from("anotherusername567"),
        // struct update syntax: the remaining fields not explicitly set
        // should have the same value as the fields in the given instance.
        ..user1
    };

    // tuple struct
    struct Color(i32, i32, i32);

    let black = Color(0, 0, 0);
}

fn build_user(username: string) -> User {
    User {
        username, // field init shorthand syntax
        active: true,
    }
}

struct User {
    username: String, // field
    active: bool,
}
```

### 5.2 An Example Program Using Structs

```rust
println!("{}", user); // compile error
println!("{:?}", user); // should annotate with #[derive(Debug)]
println!("{:#?}", user); // should annotate with #[derive(Debug)]
```

### 5.3 Method Syntax

_Methods_ are similar to functions. But are different from functions in that they’re defined within the context of a struct \(or an enum or a trait object\), and their first parameter is always `self`, which represents the instance of the struct the method is being called on.

Having a method that takes ownership of the instance by using just `self` as the first parameter is rare; this technique is usually used when the method transforms `self` into something else and you want to prevent the caller from using the original instance after the transformation.

**Associated functions** are often used for constructors that will return a new instance of the struct. And let you namespace functionality that is particular to your struct without having an instance available.

Each struct is allowed to have multiple `impl` blocks.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // If we wanted to change the instance that we’ve called the method on
    // as part of what the method does, we’d use &mut self as the first parameter.
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }

    // associated functions: namespaced by the struct
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );

    let sq = Rectangle::square(3);
}
```

## 6. Enums and Pattern Matching

Enums allow you to define a type by enumerating its possible _variants_. 

### 6.1 Defining an Enum

You can put any kind of data inside an enum variant: strings, numeric types, or structs.

```rust
enum IPAddrKind {
    V4,
    V6,
}

enum IpAddr {
    // each variant can have different types and amounts of associated data.
    V4(u8, u8, u8, u8),
    V6(String),
}

impl IpAddr {
    fn call(&self) {
        // body
    }
}

fn main() {
    let four = IpAddrKind::V4;

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));

    home.call();
}
```

Rust **doesn’t have the null feature** that many other languages have, but it does have an enum that can encode the concept of a value being present or absent. This enum is `Option<T>` included in the **prelude**.

If we use `None` rather than `Some`, we need to tell Rust what type of `Option<T>` we have.

```rust
enum Option<T> {
    Some(T),
    None,
}

let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

Everywhere that a value has a type that isn’t an `Option<T>`, you _can_ safely assume that the value isn’t null. This was a deliberate design decision for Rust to limit null’s pervasiveness and increase the safety of Rust code.

{% embed url="https://doc.rust-lang.org/std/option/enum.Option.html" %}

### 6.2 The `match` Control Flow Operator

`match` allows you to compare a value against a series of patterns and then execute code based on which pattern matches. Patterns can be made up of literal values, variable names, wildcards, and many other things.

Match arms can bind to the parts of the values that match the pattern.

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => { // patterns that bind to value
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

Matches in Rust are _exhaustive_: we must exhaust every last possibility in order for the code to be valid. 

The `_` pattern will match any value. 

### 6.3 Concise Control Flow with `if let`

The `if let` syntax lets you combine `if` and `let` into a less verbose way to handle values that match one pattern while ignoring the rest. 

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}

// simplified by if let

let some_u8_value = Some(0u8);
if let Some(3) = some_u8_value {
    println!("three");
}
```

you can think of `if let` as syntax sugar for a `match` that runs code when the value matches one pattern and then ignores all other values.

```rust
let coin = Coin::Penny;
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}


// equivalent to if let ... else ...

let coin = Coin::Penny;
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

## 7. Managing Growing Projects with Packages, Crates and Modules

A package can contain multiple binary crates and optionally one library crate.

### 7.1 Packages and Crates

A **crate** is a binary or library. The _crate root_ is a source file that the Rust compiler starts from and makes up the root module of your crate. 

A _**package**_ is one or more crates that provide a set of functionality. A package contains a _Cargo.toml_ file that describes how to build those crates.

A package _must_ contain zero or one library crates, and no more. It can contain as many binary crates as you’d like, but it must contain at least one crate \(either library or binary\).

`cargo new XX` command will give us a package.

Cargo follows conventions:

* _**src/main.rs**_: is the crate root of a binary crate with the same name as the package. 
* _**src/lib.rs**_: is the crate root of a library crate with the same name as the package.
* _**src/bin**_ **directory**_:_ each file will be a separate binary crate.

If a package contains _src/main.rs_ and _src/lib.rs_, it has two crates: a library and a binary, both with the same name as the package.

### 7.2 Defining Modules to Control Scope and Privacy

Modules can hold definitions for other items, such as structs, enums, constants, traits, functions.

```rust
// src/lib.rs
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
    }
}


// module tree
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

If module A is contained inside module B, we say that module A is the _child_ of module B and that module B is the _parent_ of module A. Notice that the entire module tree is rooted under the implicit module named `crate`.

### 7.3 Paths for Referring to an Item in the Module Tree

A path can take two forms:

* An _absolute path_ starts from a crate root by using a crate name or a literal `crate`.
* A _relative path_ starts from the current module and uses `self`, `super`, or an identifier in the current module.

All items \(functions, methods, structs, enums, modules, and constants\) are private by default. Items in a parent module can’t use the private items inside child modules, but items in child modules can use the items in their ancestor modules\(include siblings\).

Making the module public doesn’t make its contents public. The `pub` keyword on a module only lets code in its ancestor modules refer to it.

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }

    mod serving {
        fn serve_order() {}

        mod back_of_house {
            fn fix_incorrect_order() {
                super::serve_order();
            }
        }
    
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

If we use `pub` before a struct definition, we make the struct public, but the struct’s fields will still be private.

If we make an enum public, all of its variants are then public.

### 7.4 Bringing Paths into Scope with the `use` Keyword

We can bring a path into a scope once and then call the items in that path as if they’re local items with the `use` keyword.

Paths brought into scope with `use` also check privacy.

You can also bring an item into scope with `use` and a relative path.

Bringing the function’s parent module into scope with `use` so we have to specify the parent module when calling the function makes it clear that the function isn’t locally defined while still minimizing repetition of the full path. On the other hand, when bringing in structs, enums, and other items with `use`, it’s idiomatic to specify the full path. 

After the path, we can specify `as` and a new local name, or alias, for the type.

When we bring a name into scope with the `use` keyword, the name available in the new scope is private. To enable the code that calls our code to refer to that name as if it had been defined in that code’s scope, we can combine `pub` and `use`. 

The standard library \(`std`\) is also a crate that’s external to our package. The name of the standard library crate is `std`.

We can use nested paths to bring the same items into scope in one line. We do this by specifying the common part of the path, followed by two colons, and then curly brackets around a list of the parts of the paths that differ.

If we want to bring _all_ public items defined in a path into scope, we can specify that path followed by `*`, the glob operator.  Glob can make it harder to tell what names are in scope and where a name used in your program was defined.

```rust
use crate::front_of_house::hosting; // parent module

use std::collections::HashMap; // full path

use std::io::Result as IoResult;

pub use crate::front_of_house::hosting;

use std::{cmp::Ordering, io};

use std::io::{self, Write};

use std::collections::*;
```

### 7.5 Separating Modules into Different Files

Using a semicolon after `mod xxx` rather than using a block tells Rust to load the contents of the module from another file with the same name as the module. 

The `mod` keyword declares modules, and Rust looks in a file with the same name as the module for the code that goes into that module.

{% embed url="https://github.com/StoneYunZhao/the-rust-programming-language/tree/master/modules" %}

## 8. Common Collections

### 8.1 Storing Lists of Values with Vectors

Vectors allow you to store more than one value in a single data structure that puts all the values next to each other in memory. Vectors can only store values of the same type.

```rust
// initialize
let v: Vec<i32> = Vec::new();
let v = vec![1, 2, 3];

// update
let mut v = Vec::new();
v.push(5);

// get
let third: &i32 = &v[2];
let opt: Option<&i32> = v.get(2);

// iterate
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}

let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

Adding a new element onto the end of the vector might require allocating new memory and copying the old elements to the new space.

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0]; // immutable borrow

v.push(6); // compile error: mutable borrow

println!("The first element is: {}", first);
```

When we need to store elements of a different type in a vector, we can define and use an enum! If you don’t know the exhaustive set of types the program will get at runtime to store in a vector, the enum technique won’t work. Instead, you can use a trait object.

### 8.2 Storing UTF-8 Encoded Text with Strings

Strings are implemented as a collection of bytes.

Rust has only one string type in the core language, which is the string slice `str` that is usually seen in its borrowed form `&str`.

The `String` type, which is provided by Rust’s standard library rather than coded into the core language, is a growable, mutable, owned, UTF-8 encoded string type.

Rust’s standard library also includes a number of other string types, such as `OsString`, `OsStr`, `CString`, and `CStr`. 

You can conveniently use the `+` operator or the `format!` macro to concatenate `String` values.

```rust
// initialize
let mut s = String::new();
let s2 = "initial contents".to_string();
let s3 = String::from("initial contents");

// update
s.push_str("bar");
s.push('l');

let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used

let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3); // doesn’t take ownership of any of its parameters
```

The `+` operator uses the `add` method, whose signature looks something like this:

```rust
fn add(self, s: &str) -> String {
```

The compiler can _coerce_ the `&String` argument into a `&str`. 

Rust strings don’t support indexing.

A `String` is a wrapper over a `Vec<u8>`.

 You can use `[]` with a range to create a string slice containing particular bytes:

```rust
let hello = "Здравствуйте";
let s = &hello[0..4]; // s is &str
```

If you need to perform operations on individual Unicode scalar values, the best way to do so is to use the `chars` method. The `bytes` method returns each raw byte.

### 8.3 Storing Keys with Associated Values in  Hash Maps

The type `HashMap<K, V>` stores a mapping of keys of type `K` to values of type `V`. 

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);

let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];
let mut scores: HashMap<_, _> =
    teams.into_iter().zip(initial_scores.into_iter()).collect();
    
for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

For types that implement the `Copy` trait, like `i32`, the values are copied into the hash map. For owned values like `String`, the values will be moved and the hash map will be the owner of those values.

The `or_insert` method on `Entry` is defined to return a mutable reference to the value for the corresponding `Entry` key if that key exists, and if not, inserts the parameter as the new value for this key and returns a mutable reference to the new value. 

```rust
let mut scores = HashMap::new();

// insert or overwriting
scores.insert(String::from("Blue"), 10);

// insert or ignore if key exists
scores.entry(String::from("Blue")).or_insert(50);
```

## 9. Error Handling

Rust groups errors into two major categories: _recoverable_ and _unrecoverable_ errors.

Rust has the type `Result<T, E>` for recoverable errors and the `panic!` macro that stops execution when the program encounters an unrecoverable error.

### 9.1 Unrecoverable Errors with `panic`!

When the `panic!` macro executes, your program will print a failure message, unwind and clean up the stack, and then quit.

You can switch from unwinding to aborting upon a panic by adding `panic = 'abort'` to the appropriate `[profile]` sections in your _Cargo.toml_ file. 

We can set the `RUST_BACKTRACE` environment variable to get a backtrace of exactly what happened to cause the error. 

### 9.2 Recoverable Errors with `Result`

The `Result` enum and its variants have been brought into scope by the prelude.

```rust
let res = File::open("hello.txt");
res.unwrap_or_else(...);
res.unwrap();
res.expect("xxx");
```

The `?` placed after a `Result` value is defined to work in almost the same way as the `match` expressions.  If the value of the `Result` is an `Ok`, the value inside the `Ok` will get returned from this expression, and the program will continue. If the value is an `Err`, the `Err` will be returned from the whole function as if we had used the `return` keyword.

Error values that have the `?` operator called on them go through the `from` function. When the `?` operator calls the `from` function, the error type received is converted into the error type defined in the return type of the current function. As long as each error type implements the `from` function to define how to convert itself to the returned error type, the `?` operator takes care of the conversion automatically.

We’re only allowed to use the `?` operator in a function that returns `Result` or `Option` or another type that implements `std::ops::Try`. The `main` function is special:

```rust
fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())
}
```

### 9.3 To `panic!` or Not to `panic!`

Use `panic` for examples, prototype code, and tests.

It would also be appropriate to call `unwrap` when you have some other logic that ensures the `Result` will have an `Ok` value, but the logic isn’t something the compiler understands.

It’s advisable to have your code panic when it’s possible that your code could end up in a bad state. A _bad state_ is when some assumption, guarantee, contract, or invariant has been broken:

* The bad state is not something that’s _expected_ to happen occasionally.
* Your code after this point needs to rely on not being in this bad state.
* There’s not a good way to encode this information in the types you use.

If someone calls your code and passes in values that don’t make sense, the best choice might be to call `panic!` and alert the person using your library to the bug in their code so they can fix it during development. Similarly, `panic!` is often appropriate if you’re calling external code that is out of your control and it returns an invalid state that you have no way of fixing.

When failure is expected, it’s more appropriate to return a `Result` than to make a `panic!` call. 

When your code performs operations on values, your code should verify the values are valid first and panic if the values aren’t valid.

Having lots of error checks in all of your functions would be verbose and annoying. Fortunately, you can use Rust’s type system to do many of the checks for you.

## 10. Generic Types, Traits, and Lifetimes

### 10.1 Generic Data Types

```rust
fn largest<T>(list: &[T]) -> T

struct Point<T> {
    x: T,
    y: T,
}

// implement for all types
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
    
    // not same as in struct
    fn mixup<U>(self, other: Point<U>) -> (T, U) {
        (self.x, other.y)
    }
}

// impelement only for f32
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Generic type parameters in a struct definition aren’t always the same as those you use in that struct’s method signatures.

Rust implements generics in such a way that your code doesn’t run any slower using generic types than it would with concrete types.

Rust accomplishes this by performing **monomorphization** of the code that is using generics at compile time. _Monomorphization_ is the process of turning generic code into specific code by filling in the concrete types that are used when compiled. The process of monomorphization makes Rust’s generics **extremely efficient at runtime**.

The compiler looks at all the places where generic code is called and generates code for the concrete types the generic code is called with.

### 10.2 Traits: Defining Shared Behavior

A _trait_ tells the Rust compiler about functionality a particular type has and can share with other types. 

```rust
pub trait Summary {
    fn summarize(&self) -> String;

    // with default implementation
    fn summarize2(&self) -> String {
        String::from("Read more...")
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

// traits as parameters
pub fn notify(item: &impl Summary) {}

// trait bound syntax, same as notify
pub fn notify2<T: Summary>(item: &T) {}

// multiple traits
pub fn notify3(item: &(impl Summary + Display)) {}

// trait bound with + syntax, same as notify3
pub fn notify4<T: Summary + Display>(item: &T) {}

// where syntax
fn f1<T, U>(t: &T, u: &U) -> ()
    where T: Display + Clone,
          U: Clone + Debug {}

// return traits
fn f2() -> impl Summary {
    Tweet {
        username: "a".to_string(),
        content: "b".to_string(),
    }
}
```

We can implement a trait on a type only if either the trait or the type is local to our crate. But we can’t implement external traits on external types. This rule ensures that other people’s code can’t break your code and vice versa.

Default implementations can call other methods in the same trait, even if those other methods don’t have a default implementation.

**You can only use `impl Trait` if you’re returning a single type**. 

By using a trait bound with an `impl` block that uses generic type parameters, we can implement methods conditionally for types that implement the specified traits.

```rust
struct Pair<T> {
    x: T,
    y: T,
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

We can also conditionally implement a trait for any type that implements another trait. Called _blanket implementations_.

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

Rust check errors **at compile time** if we called a method on a type which didn’t define the method. Dynamic languages check it at runtime, like Java. So it improves performance.

### 10.3 Validating References with Lifetimes

Every reference in Rust has a _lifetime_, which is the scope for which that reference is valid. Most of the time, lifetimes are implicit and inferred.

The Rust compiler has a _borrow checker_ that compares scopes to determine whether all borrows are valid.

Lifetime annotations don’t change how long any of the references live. Lifetime annotations describe the relationships of the lifetimes of multiple references to each other without affecting the lifetimes.

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

When a function has references to or from code outside that function, it becomes almost impossible for Rust to figure out the lifetimes of the parameters or return values on its own. The lifetimes might be different each time the function is called. This is why we need to annotate the lifetimes manually.

```rust
// it means that the lifetime of the reference returned by the longest function
// is the same as the smaller of the lifetimes of the references passed in.
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// This annotation means an instance of ImportantExcerpt
// can’t outlive the reference it holds in its part field.
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {

    // the first elision rule
    fn level(&self) -> i32 {
        3
    }

    // the third lifetime elision rule
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

When returning a reference from a function, the lifetime parameter for the return type needs to match the lifetime parameter for one of the parameters.

Ultimately, lifetime syntax is about connecting the lifetimes of various parameters and return values of functions.

The Rust team programmed some patterns into the compiler’s code so the borrow checker could infer the lifetimes in these situations and wouldn’t need explicit annotations. Called the _**lifetime elision rules**_.

Lifetimes on function or method parameters are called _**input lifetimes**_, and lifetimes on return values are called _**output lifetimes**_.

The compiler uses three rules to figure out what lifetimes references have when there aren’t explicit annotations:

* The first rule is that each parameter that is a reference gets its own lifetime parameter.
* The second rule is if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters.
* The third rule is if there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output lifetime parameters.

One special lifetime we need to discuss is `'static`, which means that this reference _can_ live for the entire duration of the program. 

```rust
// generic + trait bounds + lifetimes
fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
    where
        T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### Summary

**Generic type** parameters let you apply the code to different types. **Traits and trait bounds** ensure that even though the types are generic, they’ll have the behavior the code needs. You learned how to use **lifetime annotations** to ensure that this flexible code won’t have any dangling references. And all of this analysis happens at **compile time**, which doesn’t affect runtime performance!

## 11. Writing Automated Tests

### 11.1 How to Write Tests

Attributes are metadata about pieces of Rust code. To change a function into a test function, add `#[test]` on the line before `fn`. 

The `cargo test` command runs all tests in our project.

Tests fail when something in the test function panics. Each test is run in a new thread, and when the main thread sees that a test thread has died, the test is marked as failed.

The `assert!` macro, provided by the standard library, is useful when you want to ensure that some condition in a test evaluates to `true`.

You can’t use the `#[should_panic]` annotation on tests that use `Result<T, E>`. 

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
        assert_ne!(2 + 3, 4);
        assert!(1 + 2 == 3);
        assert!(1 == 1, "assert with custom message: {}", 123)
    }

    #[test]
    #[should_panic]
    fn should_panic() {
        panic!("asf")
    }

    #[test]
    #[should_panic(expected = "too long")]
    fn should_panic2() {
        panic!("too long a")
    }

    #[test]
    fn result() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("failed"))
        }
    }
}
```

### 11.2 Controlling How Tests Are Run

When you run multiple tests, by default they run in parallel using threads.

By default, if a test passes, Rust’s test library captures anything printed to standard output.

The module in which a test appears becomes part of the test’s name, so we can run all the tests in a module by filtering on the module’s name.

You can annotate the time-consuming tests using the `ignore` attribute to exclude them.

```bash
cargo test -- --test-threads=1
cargo test -- --show-output
cargo test ${fn_name_pattern}
cargo test -- --ignored # include ignored tests
```

### 11.3 Test Organization

The Rust community thinks about tests in terms of two main categories: _unit tests_ and _integration tests_. **Unit tests** are small and more focused, testing one module in isolation at a time, and can test private interfaces. **Integration tests** are entirely external to your library and use your code in the same way any other external code would, using only the public interface and potentially exercising multiple modules per test.

The purpose of unit tests is to test each unit of code in isolation from the rest of the code to quickly pinpoint where code is and isn’t working as expected. You’ll put unit tests in the _src_ directory in each file with the code that they’re testing.

The `#[cfg(test)]` annotation on the tests module tells Rust to compile and run the test code only when you run `cargo test`, not when you run `cargo build`. The attribute `cfg` stands for _configuration_ and tells Rust that the following item should only be included given a certain configuration option. 

To create integration tests, you first need a _tests_ directory. Each file in the `tests` directory is a separate crate.

```bash
cargo test --test ${file_name} # run particular test file
```

_tests/common/mod.rs_ is a naming convention that Rust understands. Naming the file this way tells Rust not to treat the `common` module as an integration test file.

Files in subdirectories of the _tests_ directory don’t get compiled as separate crates or have sections in the test output.

If our project is a binary crate that only contains a _src/main.rs_ file and doesn’t have a _src/lib.rs_ file, we can’t create integration tests in the _tests_ directory and bring functions defined in the _src/main.rs_ file into scope with a `use` statement. We can provide a binary have a straightforward _src/main.rs_ file that calls logic that lives in the _src/lib.rs_ file. 

## 12. An I/O Project: Building a Command Line Program

{% embed url="https://github.com/StoneYunZhao/the-rust-programming-language/tree/master/minigrep" %}

### 12.3 Refactoring to Improve Modularity and Error Handling

The Rust community has developed a process to use as a guideline for splitting the separate concerns of a binary program when `main` starts getting large. The process has the following steps:

* Split your program into a _main.rs_ and a _lib.rs_ and move your program’s logic to _lib.rs_.
* As long as your command line parsing logic is small, it can remain in _main.rs_.
* When the command line parsing logic starts getting complicated, extract it from _main.rs_ and move it to _lib.rs_.

The responsibilities that remain in the `main` function after this process should be limited to the following:

* Calling the command line parsing logic with the argument values
* Setting up any other configuration
* Calling a `run` function in _lib.rs_
* Handling the error if `run` returns an error

This pattern is about separating concerns: _main.rs_ handles running the program, and _lib.rs_ handles all the logic of the task at hand. Because you can’t test the `main` function directly, this structure lets you test all of your program’s logic by moving it into functions in _lib.rs_.

### 12.4 Developing the Library’s Functionality with Test-Driven Development

Test-driven development \(TDD\) process:

1. Write a test that fails and run it to make sure it fails for the reason you expect.
2. Write or modify just enough code to make the new test pass.
3. Refactor the code you just added or changed and make sure the tests continue to pass.
4. Repeat from step 1!

## 13. Functional Language Features: Iterators and Closures

### 13.1 Closures: Anonymous Functions that Can Capture Their Environment

Rust’s closures are anonymous functions you can save in a variable or pass as arguments to other functions. Unlike functions, closures can capture values from the scope in which they’re defined.

Closures don’t require you to annotate the types of the parameters or the return value like `fn` functions do. Closures are usually short and relevant only within a narrow context rather than in any arbitrary scenario. Within these limited contexts, the compiler is reliably able to infer the types of the parameters and the return type. 

As with variables, we can add type annotations if we want to increase explicitness and clarity at the cost of being more verbose than is strictly necessary.

Closure definitions will have one concrete type inferred for each of their parameters and for their return value.

Each closure instance has its own unique anonymous type: that is, even if two closures have the same signature, their types are still considered different.

The `Fn` traits are provided by the standard library. All closures implement at least one of the traits: `Fn`, `FnMut`, or `FnOnce`.

Closures can capture values from their environment in three ways, which directly map to the three ways a function can take a parameter: **taking ownership**, **borrowing mutably**, and **borrowing immutably** :

* `FnOnce` consumes the variables it captures from its enclosing scope, known as the closure’s _environment_. To consume the captured variables, the closure must take ownership of these variables and move them into the closure when it is defined. The `Once` part of the name represents the fact that the closure can’t take ownership of the same variables more than once, so it can be called only once.
* `FnMut` can change the environment because it mutably borrows values.
* `Fn` borrows values from the environment immutably.

When you create a closure, Rust infers which trait to use based on how the closure uses the values from the environment. All closures implement `FnOnce` because they can all be called at least once. Closures that don’t move the captured variables also implement `FnMut`, and closures that don’t need mutable access to the captured variables also implement `Fn`. 

If you want to force the closure to take ownership of the values it uses in the environment, you can use the `move` keyword before the parameter list. This technique is mostly useful when passing a closure to a new thread to move the data so it’s owned by the new thread.

> `move` closures may still implement `Fn` or `FnMut`, even though they capture variables by move. This is because the traits implemented by a closure type are determined by what the closure does with captured values, not how it captures them. The `move` keyword only specifies the latter.

```rust
fn add_one_v1(x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x| { x + 1 };
let add_one_v4 = |x| x + 1;

// compile error
let example_closure = |x| x;
let s = example_closure(String::from("hello"));
let n = example_closure(5);

// move keyword
let x = vec![1, 2, 3];
let equal_to_x = move |z| z == x;


struct Cacher<T>
    where
        T: Fn(u32) -> u32,
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
    where
        T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }
}
```

### 13.2 Processing a Series of Items with Iterators

In Rust, iterators are _lazy_, meaning they have no effect until you call methods that consume the iterator to use it up. 

All iterators implement a trait named `Iterator` that is defined in the standard library.

```rust
pub trait Iterator {
    type Item;
    
    fn next(&mut self) -> Option<Self::Item>;
}
```

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
}
```

Note that we needed to make `v1_iter` mutable: calling the `next` method on an iterator changes internal state that the iterator uses to keep track of where it is in the sequence. We didn’t need to make `v1_iter` mutable when we used a `for` loop because the loop took ownership of `v1_iter` and made it mutable behind the scenes.

The `iter` method produces an iterator over immutable references. If we want to create an iterator that takes ownership of `v1` and returns owned values, we can call `into_iter` instead of `iter`. Similarly, if we want to iterate over mutable references, we can call `iter_mut` instead of `iter`.

Methods that call `next` are called _**consuming adaptors**_, because calling them uses up the iterator. Other methods defined on the `Iterator` trait, known as _**iterator adaptors**_, allow you to change iterators into different kinds of iterators. You can chain multiple calls to iterator adaptors to perform complex actions in a readable way. But because all iterators are lazy, you have to call one of the consuming adaptor methods to get results from calls to iterator adaptors.

### 13.4 Comparing Performance: Loops vs. Iterators

iterators, although a high-level abstraction, get compiled down to roughly the same code as if you’d written the lower-level code yourself. Iterators are one of Rust’s _zero-cost abstractions_, by which we mean using the abstraction imposes no additional runtime overhead. 

_Unrolling_ is an optimization that removes the overhead of the loop controlling code and instead generates repetitive code for each iteration of the loop.

You can use iterators and closures without fear! Rust make code seem like it’s higher level but don’t impose a runtime performance penalty for doing so.

## 14. More About Cargo and Crates.io

### 14.1 Customizing Builds with Release Profiles

Cargo has two main profiles: the `dev` profile Cargo uses when you run `cargo build` and the `release` profile Cargo uses when you run `cargo build --release`.

The `opt-level` setting controls the number of optimizations Rust will apply to your code, with a range of 0 to 3.

### 14.2 Publishing a Crate to Crates.io

Documentation comments use three slashes, `///`, instead of two and support Markdown notation for formatting the text.

```bash
cargo doc
cargo doc --open
cargo login XXX
cargo publish

cargo yank --vers 1.0.1
cargo yank --vers 1.0.1 --undo
```

Some commonly used sections: 

* Examples.
* Panics.
* Errors.
* Safety.

Running `cargo test` will run the code examples in your documentation as tests!

Another style of doc comment, `//!`, adds documentation to the item that contains the comments rather than adding documentation to the items following the comments. 

You can re-export items to make a public structure that’s different from your private structure by using `pub use`. Re-exporting takes a public item in one location and makes it public in another location, as if it were defined in the other location instead.

Before publishing, you’ll need to add some metadata to your crate by adding it to the `[package]` section of the crate’s _Cargo.toml_ file.

Yanking a version prevents new projects from starting to depend on that version while allowing all existing projects that depend on it to continue to download and depend on that version.

### 14.3 Cargo Workspaces

A _workspace_ is a set of packages that share the same _Cargo.lock_ and output directory. 

The crates in a workspace are meant to depend on each other.

Cargo doesn’t assume that crates in a workspace will depend on each other, so we need to be explicit about the dependency relationships between the crates.

```rust
cargo run -p ${crate}
cargo test -p ${crate}
```

You must change to each crate’s directory and run `cargo publish` on each crate in the workspace to publish the crates.

### 14.4 Installing Binaries from Crates.io with `cargo install`

The `cargo install` command allows you to install and use binary crates locally.

You can only install packages that have binary targets. 

### 14.5 Extending Cargo with Custom Commands

If a binary in your `$PATH` is named `cargo-something`, you can run it as if it was a Cargo subcommand by running `cargo something`.

Custom commands like this are also listed when you run `cargo --list`. 

## 15. Smart Pointers

_Smart pointers_ are data structures that not only act like a pointer but also have additional metadata and capabilities. 

References are pointers that only borrow data; in contrast, in many cases, smart pointers _own_ the data they point to.

The characteristic that distinguishes a smart pointer from an ordinary struct is that smart pointers implement the `Deref` and `Drop` traits.

The most common smart pointers in the standard library:

* `Box<T>` for allocating values on the heap
* `Rc<T>`, a reference counting type that enables multiple ownership
* `Ref<T>` and `RefMut<T>`, accessed through `RefCell<T>`, a type that enforces the borrowing rules at runtime instead of compile time

### 15.1 Using `Box<T>` to Point to Data on the Heap

Boxes don’t have performance overhead. You’ll use them most often in these situations:

* When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size
* When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so
* When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type.

Just like any owned value, when a box goes out of scope, it will be deallocated. The deallocation happens for the box \(stored on the stack\) and the data it points to \(stored on the heap\).

_recursive type_, where a value can have as part of itself another value of the same type. 

To determine how much space to allocate for enum type, Rust goes through each of the variants to see which variant needs **the most space**.

The `Box<T>` type is a smart pointer because it implements the `Deref` trait, which allows `Box<T>` values to be treated like references. When a `Box<T>` value goes out of scope, the heap data that the box is pointing to is cleaned up as well because of the `Drop` trait implementation. 

### 15.2 Treating Smart Pointers Like Regular References with the `Deref` Trait

Implementing the `Deref` trait allows you to customize the behavior of the _dereference operator_, `*`.

The `deref` method gives the compiler the ability to take a value of any type that implements `Deref` and call the `deref` method to **get a `&` reference** that it knows how to dereference. If the `deref` method returned the value directly instead of a reference to the value, the value would be moved out of `self`. 

```rust
// the type of y implemented deref trait
*y ==> *(y.deref())

&y ==> y.deref() // deref coercion
```

Deref coercion happens automatically when we pass a reference to a particular type’s value as an argument to a function or method that doesn’t match the parameter type in the function or method definition. A sequence of calls to the `deref` method converts the type we provided into the type the parameter needs.

When the `Deref` trait is defined for the types involved, Rust will analyze the types and use `Deref::deref` as many times as necessary to get a reference to match the parameter’s type. The number of times that `Deref::deref` needs to be inserted is resolved at compile time.

You can use the `DerefMut` trait to override the `*` operator on mutable references.

Rust does deref coercion when it finds types and trait implementations in three cases:

* From `&T` to `&U` when `T: Deref<Target=U>`
* From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
* From `&mut T` to `&U` when `T: Deref<Target=U>`

### 15.3 Running Code on Cleanup with the `Drop` Trait

`Drop` lets you customize what happens when a value is about to go out of scope.  The `Drop` trait is included in the prelude.

Variables are dropped in the reverse order of their creation.

Rust doesn’t let you call the `Drop` trait’s `drop` method manually; instead you have to call the `std::mem::drop` function provided by the standard library which is prelude if you want to force a value to be dropped before the end of its scope.

The ownership system that makes sure references are always valid also ensures that `drop` gets called only once when the value is no longer being used.

### 15.4 `Rc<T>`, the Reference Counted Smart Pointer

The `Rc<T>` type keeps track of the number of references to a value which determines whether or not a value is still in use. 

Note that `Rc<T>` is only for use in single-threaded scenarios.

```rust
Rc::new(xx);
Rc::clone(xx);
Rc::strong_count(xx);
```

### 15.5 `RefCell<T>` and the Interior Mutability Pattern

_Interior mutability_ is a design pattern in Rust that allows you to mutate data even when there are immutable references to that data. The pattern uses `unsafe` code inside a data structure to bend Rust’s usual rules that govern mutation and borrowing. 

With `RefCell<T>`, the borrowing rules’ invariants are enforced _at runtime_. The `RefCell<T>` type is useful when you’re sure your code follows the borrowing rules but the compiler is unable to understand and guarantee that.

`RefCell<T>` is only for use in single-threaded scenarios and will give you a compile-time error if you try using it in a multithreaded context.

When creating immutable and mutable references, we use the `&` and `&mut` syntax, respectively. With `RefCell<T>`, we use the `borrow` and `borrow_mut` methods, which are part of the safe API that belongs to `RefCell<T>`. The `borrow` method returns the smart pointer type `Ref<T>`, and `borrow_mut` returns the smart pointer type `RefMut<T>`.

The `RefCell<T>` keeps track of how many `Ref<T>` and `RefMut<T>` smart pointers are currently active. Every time we call `borrow`, the `RefCell<T>` increases its count of how many immutable borrows are active. When a `Ref<T>` value goes out of scope, the count of immutable borrows goes down by one. Just like the compile-time borrowing rules, `RefCell<T>` lets us have many immutable borrows or one mutable borrow at any point in time.

If you have an `Rc<T>` that holds a `RefCell<T>`, you can get a value that can have multiple owners _and_ that you can mutate!

```rust
// x's type is RefCell<T>
x.borrow();
x.borrow_mut();
```

### 15.6 Reference Cycles Can Leak Memory

If you have `RefCell<T>` values that contain `Rc<T>` values or similar nested combinations of types with interior mutability and reference counting, you must ensure that you don’t create cycles; you can’t rely on Rust to catch them. 

You can create a _weak reference_ to the value within an `Rc<T>` instance by calling `Rc::downgrade` and passing a reference to the `Rc<T>`. Calling `Rc::downgrade` increases the `weak_count` by 1. The `weak_count` doesn’t need to be 0 for the `Rc<T>` instance to be cleaned up.

Strong references are how you can share ownership of an `Rc<T>` instance. Weak references don’t express an ownership relationship. Weak references will be broken once the strong reference count of values involved is 0.

Because the value that `Weak<T>` references might have been dropped, to do anything with the value that a `Weak<T>` is pointing to, you must make sure the value still exists. Do this by calling the `upgrade` method on a `Weak<T>` instance, which will return an `Option<Rc<T>>`.

```rust
Week::new();
Rc::downgrade(&x); // x's type is Rc<T>
y.upgrade(); // y's type is Week<T>
Rc::weak_count();
```

## 16. Fearless Concurrency

_Concurrent programming_, where different parts of a program execute independently, and _parallel programming_, where different parts of a program execute at the same time. 

* _**Message-passing**_ **concurrency**, where channels send messages between threads
* _**Shared-state**_ **concurrency**, where multiple threads have access to some piece of data

### 16.1 Using Threads to Run Code Simultaneously

Programming language-provided threads are known as _green_ threads. The Rust standard library only provides an implementation of 1:1 threading.

The calls to `thread::sleep` force a thread to stop its execution for a short duration, allowing a different thread to run.

The return type of `thread::spawn` is `JoinHandle`. A `JoinHandle` is an owned value that, when we call the `join` method on it, will wait for its thread to finish. 

We can use the `move` keyword before the parameter list of a closure to force the closure to take ownership of the values it uses in the environment. The `move` closure is often used alongside `thread::spawn` because it allows you to use data from one thread in another thread.

```rust
let v = vec![1, 2, 3];

let handle = thread::spawn(move || {
    println!("vector: {:?}", v)
});

handle.join().unwrap();
```

### 16.2 Using Message Passing to Transfer Data Between Threads

One major tool Rust has for accomplishing message-sending concurrency is the _channel._ A channel is said to be _closed_ if either the transmitter or receiver half is dropped.

We create a new channel using the `mpsc::channel` function; `mpsc` stands for _multiple producer, single consumer_.

`recv` will block the main thread’s execution and wait until a value is sent down the channel. The `try_recv` method doesn’t block.

The `send` function takes ownership of its parameter.

```rust
let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    tx.send(String::from("hi")).unwrap();
});

let rev = rx.recv().unwrap();

println!("got: {}", rev)


let tx1 = tx.clone(); // tx1 could be used in another thread to send value
```

### 16.3 Shared-State Concurrency

_Mutex_ is an abbreviation for _mutual exclusion_, as in, a mutex allows only one thread to access some data at any given time. The lock is a data structure that is part of the mutex that keeps track of who currently has exclusive access to the data.

`Mutex<T>` is a smart pointer. The call to `lock` _returns_ a smart pointer called `MutexGuard`. The `MutexGuard` smart pointer implements `Deref` to point at our inner data; the smart pointer also has a `Drop` implementation that releases the lock automatically when a `MutexGuard` goes out of scope.

`Arc<T>` _is_ a type like `Rc<T>` that is safe to use in concurrent situations.

 to allow us to mutate contents inside an `Rc<T>`, we use `Mutex<T>` to mutate contents inside an `Arc<T>`.

```rust
let m = Mutex::new(5);

{
    let mut num = m.lock().unwrap(); // num is MutexGuard<i32>
    *num = 6;
} // release lock automatically by Drop implementation
```

```rust
let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();

        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("Result: {}", *counter.lock().unwrap());
```

### 16.4 Extensible Concurrency with the `Sync` and `Send` Traits

Two concurrency concepts are embedded in the language: the `std::marker` traits `Sync` and `Send`.

The `Send` marker trait indicates that ownership of values of the type implementing `Send` can be transferred between threads. Any type composed entirely of `Send` types is automatically marked as `Send` as well. Almost all primitive types are `Send`, aside from raw pointers.

The `Sync` marker trait indicates that it is safe for the type implementing `Sync` to be referenced from multiple threads. Similar to `Send`, primitive types are `Sync`, and types composed entirely of types that are `Sync` are also `Sync`.

So any type `T` is `Sync` if `&T` is `Send`.

## 17. Object Oriented Programming Features of Rust

### 17.1 Characteristics of Object-Oriented Languages

OOP languages share certain common characteristics, namely **objects, encapsulation, and inheritance**.

Object-oriented programs are made up of objects. An _object_ packages both data and the procedures that operate on that data. The procedures are typically called _methods_ or _operations_.

**Objects**: structs and enums have data, and `impl` blocks provide methods on structs and enums. 

**Encapsulation**: we can use the `pub` keyword to decide which modules, types, functions, and methods in our code should be public, and by default everything else is private. 

_Inheritance_ is a mechanism whereby an object can inherit from another object’s definition, thus gaining the parent object’s data and behavior without you having to define them again.

**Inheritance**: Rust not support. There is no way to define a struct that inherits the parent struct’s fields and method implementations. But you can use other solution in Rust: 

* You can share Rust code using default trait method implementations instead. We can also override the default implementation.
* Rust use trait objects instead of inheritance. Go to next section for detail.

> To many people, **polymorphism\(多态\)** is synonymous with inheritance. But it’s actually a more general concept that refers to code that can work with data of multiple types. For inheritance, those types are generally subclasses.

### 17.2 Using Trait Objects That Allow for Values of Different Types

A trait object points to both an instance of a type implementing our specified trait as well as a table used to look up trait methods on that type at runtime. We create a trait object by specifying some sort of **pointer**, such as a `&` reference or a `Box<T>` smart pointer, then the `dyn` keyword, and then specifying the relevant trait. 

Wherever we use a trait object, Rust’s type system will ensure **at compile time** that any value used in that context will implement the trait object’s trait.

A generic type parameter with trait bounds can only be substituted with one concrete type at a time, whereas trait objects allow for multiple concrete types to fill in for the trait object at runtime.

```rust
pub trait Draw {
    fn draw(&self);
}

// Sreen1 use trait objects
// allow for multiple concrete types
pub struct Screen1 {
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen1 {
    pub fn run(&self) {
        for component in self.components.iter(){
            component.draw();
        }
    }
}

// Secreen2 use generic type parameter
// T can only be substituted with only one concrete type
pub struct Screen2<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen2<T>
    where
        T: Draw,
{
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

* **static dispatch**: monomorphization. The compiler generates nongeneric implementations of functions and methods for each concrete type that we use in place of a generic type parameter.
* **dynamic dispatch**: the compiler emits code that at runtime will figure out which method to call.

When we use trait objects, Rust must use dynamic dispatch. At runtime, Rust uses the pointers inside the trait object to know which method to call. There is a runtime cost. Dynamic dispatch also prevents the compiler from choosing to inline a method’s code, which in turn prevents some optimizations.

You can only make _object-safe_ traits into trait objects. A trait is object safe if all the methods defined in the trait have the following properties:

* The return type isn’t `Self`.
* There are no generic type parameters.

The `Self` keyword is an alias for the type we’re implementing the traits or methods on. Trait objects must be object safe because once you’ve used a trait object, Rust no longer knows the concrete type that’s implementing that trait. If a trait method returns the concrete `Self` type, but a trait object forgets the exact type that `Self` is, there is no way the method can use the original concrete type. The same is true of generic type parameters that are filled in with concrete type parameters when the trait is used: the concrete types become part of the type that implements the trait. When the type is forgotten through the use of a trait object, there is no way to know what types to fill in the generic type parameters with.

```rust
// not object safe
pub trait Clone {
    fn clone(&self) -> Self;
}
```

### 17.3 Implementing an Object-Oriented Design Pattern

{% embed url="https://github.com/StoneYunZhao/the-rust-programming-language/blob/master/oop/src/post.rs" %}

## 18. Patterns and Matching

A pattern consists of some combination of the following:

* Literals
* Destructured arrays, enums, structs, or tuples
* Variables
* Wildcards
* Placeholders

### 18.1 All the Places Patterns Can Be Used

One requirement for `match` expressions is that they need to be _exhaustive_ in the sense that all possibilities for the value in the `match` expression must be accounted for. A variable name matching any value can never fail and thus covers every remaining case. A particular pattern `_` will match anything, but it never binds to a variable.

```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

`if let` expressions mainly as a shorter way to write the equivalent of a `match` that only matches one case. `if let` can also introduce shadowed variables in the same way that `match` arms can. The downside of using `if let` expressions is that the compiler doesn’t check exhaustiveness, whereas with `match` expressions it does.

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {}, as the background", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age { // introduced a new shadowed age variable
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

The `while let` conditional loop allows a `while` loop to run for as long as a pattern continues to match. 

```rust
let mut stack = Vec::new();

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

In a `for` loop, the pattern is the value that directly follows the keyword `for`, so in `for x in y` the `x` is the pattern.

```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

```rust
let PATTERN = EXPRESSION;
let x = 5;
let (x, y, z) = (1, 2, 3); // destructure a tuple
```

Function parameters can also be patterns. We can also use patterns in closure parameter lists in the same way as in function parameter lists.

```rust
// The values &(3, 5) match the pattern &(x, y)
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

### 18.2 Refutability: Whether a Pattern Might Fail to Match

Patterns come in two forms: refutable and irrefutable. Patterns that will match for any possible value passed are _**irrefutable**_. Patterns that can fail to match for some possible value are _**refutable**_.

Function parameters, `let` statements, and `for` loops can only accept irrefutable patterns, because the program cannot do anything meaningful when values don’t match. 

The `if let` and `while let` expressions accept refutable and irrefutable patterns, but the compiler warns.

Match arms must use refutable patterns, except for the last arm.

```rust
let Some(x) = some_option_value; // compile error: must use irrefutable pattern

// warn: using irrefutable here doesn't make sense
if let x = 5 {
    println!("{}", x);
};
```

### 18.3 Pattern Syntax

The `..=` syntax allows us to match to an inclusive range of values. 

Ranges are only allowed with numeric values or `char` values.

```rust
// matching literals
let x = 1;
match x {
    1 => println!("one"),
    2 => println!("two"),
    _ => println!("others"),
}

// matching named variables
let x = Some(5);
match x {
    Some(y) => println!("y = {:?}", y),
    _ => println!("others"),
}

// multiple pattern
let x = 1;
match x {
    1 | 2 => println!("one or two"),
    _ => println!("others"),
}

// matching ranges of values
let x = 5;
match x {
    1..=5 => println!("one through five"),
    _ => println!("others"),
}
```

We can also use patterns to destructure structs, enums, tuples, and references to use different parts of these values.

The pattern to destructure an enum should correspond to the way the data stored within the enum is defined.

We can mix, match, and nest destructuring patterns in even more complex ways.

```rust
// destructuring structs
let p = Point { x: 0, y: 7 };
let Point { x: a, y: b } = p; // a and b are variables
let Point { x, y } = p; // x and y are variables
match p { // destruct with literal values
    Point { x, y: 0 } => println!("y is zero"),
    Point { x: 0, y } => println!("x is zero"),
    Point { x, y } => println!("others")
}

// destructuring enums
// mentioned before, like Option<T>, Result<T, E>

// destructuring nested structs and enums
let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));
match msg {
    Message::ChangeColor(Color::Rgb(r, g, b)) => println!("rgb");
    Message::ChangeColor(Color::Hsv(g, s, v)) => println!("hsv"),
    _ => println!("others"),
}

// destructuring structs and tuples
let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
```

It’s sometimes useful to ignore values in a pattern, such as in the last arm of a `match`.

There is a subtle difference between using only `_` and using a name that starts with an underscore. The syntax `_x` still binds the value to the variable, whereas `_` doesn’t bind at all.

The `..` pattern ignores any parts of a value that we haven’t explicitly matched in the rest of the pattern. Using `..` must be unambiguous.

```rust
// ignoring an entire value with _
fn foo(_: i32, y: i32) {}

// ignoring parts of a value with a nested _
let numbers = (2, 4, 8, 16, 32);
match numbers {
    (first, _, third, _, fifth) => println!("numbers"),
}

// ignoring an unused variable by starting its name with _
let _x = 5; // no compile warning even though _x is not used anywhere

// ignoring remaining parts of a value with ..
let numbers = (2, 4, 8, 16, 32);
match numbers {
    (first, .., last) => println!("first and last"),
}
```

A _match guard_ is an additional `if` condition specified after the pattern in a `match` arm that must also match, along with the pattern matching, for that arm to be chosen. The condition can use variables created in the pattern.

You can also use the _or_ operator `|` in a match guard to specify multiple patterns; the match guard condition will apply to all the patterns. 

```rust
let num = Some(4);
match num {
    Some(x) if x < 5 => println!("x < 5"),
    Some(x) => println!("other x"),
    _ => println!("others"),
}
```

The _at_ operator \(`@`\) lets us create a variable that holds a value at the same time we’re testing that value to see whether it matches a pattern. Using `@` lets us test a value and save it in a variable within one pattern.

```rust
enum Msg {
    Hello { id: i32 },
}

let msg = Msg::Hello { id: 5 };
match msg {
    Msg::Hello { id: id_var @ 3..=7, } => println!("id: {}", id_var),
    Msg::Hello { id: 10..=12 } => println!("10 to 12"),
    Msg::Hello { id } => println!("other"),
}
```

## 19. Advanced Features

### 19.1 Unsafe Rust

Unsafe Rust exists because, by nature, static analysis is conservative.

To switch to unsafe Rust, use the `unsafe` keyword and then start a new block that holds the unsafe code. You can take five actions in unsafe Rust:

* Dereference a raw pointer
* Call an unsafe function or method
* Access or modify a mutable static variable
* Implement an unsafe trait
* Access fields of `union`s

`unsafe` doesn’t turn off the borrow checker or disable any other of Rust’s safety checks. The `unsafe` keyword only gives you access to these five features that are then not checked by the compiler for memory safety. Any errors related to memory safety must be within an `unsafe` block. It’s best to enclose unsafe code within a safe abstraction and provide a safe API.

#### Deference a Raw Pointer

Unsafe Rust has two new types called _raw pointers_ that are similar to references. Raw pointers can be immutable or mutable and are written as `*const T` and `*mut T`, respectively. _immutable_ means that the pointer can’t be directly assigned to after being dereferenced.

Different from references and smart pointers, raw pointers:

* Are allowed to ignore the borrowing rules by having both immutable and mutable pointers or multiple mutable pointers to the same location
* Aren’t guaranteed to point to valid memory
* Are allowed to be null
* Don’t implement any automatic cleanup

You can get greater performance or the ability to interface with another language or hardware from raw pointers.

We can create raw pointers in safe code; we just can’t dereference raw pointers outside an unsafe block.

Situations using raw pointers: 

* interfacing with C code
* building up safe abstractions that the borrow checker doesn’t understand

```rust
let address = 0x012345usize;
let r = address as *const i32;

let mut num = 5;
let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

#### Calling an Unsafe Function or Method

Bodies of unsafe functions are effectively `unsafe` blocks, so to perform other unsafe operations within an unsafe function, we don’t need to add another `unsafe` block.

```rust
unsafe fn dangerous() {}

unsafe {
    dangerous()
}
```

#### Creating a Safe Abstraction over Unsafe Code

```rust
impl<T, A: Allocator> Vec<T, A> {

    #[inline]
    pub fn split_at_spare_mut(&mut self) -> (&mut [T], &mut [MaybeUninit<T>]) {
        // SAFETY:
        // - len is ignored and so never changed
        let (init, spare, _) = unsafe { self.split_at_spare_mut_with_len() };
        (init, spare)
    }
}
```

**Using extern Functions to Call External Code**

Rust has a keyword, `extern`, that facilitates the creation and use of a _Foreign Function Interface \(FFI\)_. ****An FFI is a way for a programming language to define functions and enable a different \(foreign\) programming language to call those functions.

Functions declared within `extern` blocks are always unsafe to call from Rust code.

The ABI\(_application binary interface_\) defines how to call the function at the assembly level.

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

unsafe {
    println!("Absolute value of -3 according to C: {}", abs(-3));
}
```

We can also use `extern` to create an interface that allows other languages to call Rust functions.

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

#### Accessing or Modifying a Mutable Static Variable <a id="accessing-or-modifying-a-mutable-static-variable"></a>

In Rust, global variables are called _static_ variables. Static variables can only store references with the `'static` lifetime. Accessing an immutable static variable is safe. Accessing and modifying mutable static variables is _unsafe_. 

With mutable data that is globally accessible, it’s difficult to ensure there are no data races, which is why Rust considers mutable static variables to be unsafe.

Constants .vs static variables

* values in a static variable have a fixed address in memory. Constants are allowed to duplicate their data whenever they’re used.
* static variables can be mutable.

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}

static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}
```

#### Implementing an Unsafe Trait <a id="implementing-an-unsafe-trait"></a>

A trait is unsafe when at least one of its methods has some invariant that the compiler can’t verify.

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

The compiler implements traits automatically if our types are composed entirely of `Send` and `Sync` types. If we implement a type that contains a type that is not `Send` or `Sync`, and we want to mark that type as `Send` or `Sync`, we must use `unsafe`. 

#### Accessing Fields of a Union <a id="accessing-fields-of-a-union"></a>

 A `union` is similar to a `struct`, but only one declared field is used in a particular instance at one time. Unions are primarily used to interface with unions in C code. 

Accessing union fields is unsafe because Rust can’t guarantee the type of the data currently being stored in the union instance.

### 19.2 Advanced Traits

#### Associated Types

_Associated types_ connect a type placeholder with a trait such that the trait method definitions can use these placeholder types in their signatures. The implementor of a trait will specify the concrete type to be used in this type’s place for the particular implementation. That way, we can define a trait that uses some types without needing to know exactly what those types are until the trait is implemented.

We can’t implement a trait on a type multiple times.

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {}
}
```

#### Default Generic Type Parameters and Operator Overloading

When we use generic type parameters, we can specify a default concrete type for the generic type. The syntax is `<PlaceholderType=ConcreteType>`.

_Operator overloading_ is customizing the behavior of an operator \(such as `+`\) in particular situations. You can overload the operations and corresponding traits listed in `std::ops` by implementing the traits associated with the operator. 

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

#### Fully Qualified Syntax

The compiler defaults to calling the method that is directly implemented on the type.

You’re allowed to omit any part of this syntax that Rust can figure out from other information in the program.

```rust
// fully qualified syntax
<Type as Trait>::function(receiver_if_method, next_arg, ...);

Pilot::fly(&person); // Omit Type

<Dog as Animal>::baby_name();
```

#### Supertraits

```rust
// work only for types that also implement Display
trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string(); // use Display trait's functionality
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

#### newtype pattern

**orphan rule**: we’re allowed to implement a trait on a type as long as either the trait or the type are local to our crate.

There is no runtime performance penalty for using this pattern, and the wrapper type is elided at compile time.

```rust
// wrapper type
struct Wrapper(Vec<String>); // tuple struct

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}
```

### 19.3 Advanced Types

The new type can expose a public API that is different from the API of the private inner type. The newtype pattern is a lightweight way to achieve encapsulation to hide implementation details.

```rust
// type alias
type Kilometers = i32;
type Thunk = Box<dyn Fn() + Send + 'static>;
type Result<T> = std::result::Result<T, std::io::Error>;
```

Functions that return never are called _diverging functions_. Expressions of type `!` can be coerced into any other type. 

```rust
// never type
fn bar() -> ! {
    // --snip--
}
```

Rust needs to know how much memory to allocate for any value of a particular type, and all values of a type must use the same amount of memory. It’s not possible to create a variable holding a dynamically sized type.

So although a `&T` is a single value that stores the memory address of where the `T` is located, a `&str` is _two_ values: the address of the `str` and its length.

The golden rule of dynamically sized types is that we must always put values of dynamically sized types behind a pointer of some kind.

Every trait is a dynamically sized type. To use traits as trait objects, we must put them behind a pointer, such as `&dyn Trait` or `Box<dyn Trait>` \(`Rc<dyn Trait>` would work too\).

Rust has a particular trait called the `Sized` trait to determine whether or not a type’s size is known at compile time. This trait is automatically implemented for everything whose size is known at compile time. In addition, Rust implicitly adds a bound on `Sized` to every generic function. 

```rust
// compile error
let s1: str = "Hello there!";
let s2: str = "How's it going?";

// ok
let s1: &str = "Hello there!";
```

By default, generic functions will work only on types that have a known size at compile time. However, you can use the following special syntax to relax this restriction. This syntax is only available for `Sized`, not any other traits.

```rust
// parameter must be &T
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

### 19.4 Advanced Functions and Closures

#### Function Pointers

Functions coerce to the type `fn` \(with a lowercase f\), not to be confused with the `Fn` closure trait. The `fn` type is called a _function pointer_. 

```rust
fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}
```

Unlike closures, `fn` is a type rather than a trait, so we specify `fn` as the parameter type directly rather than declaring a generic type parameter with one of the `Fn` traits as a trait bound.

Function pointers implement all three of the closure traits \(`Fn`, `FnMut`, and `FnOnce`\), so you can always pass a function pointer as an argument for a function that expects a closure.

Tuple structs and tuple-struct enum variants use `()` as initializer syntax, which looks like a function call. The initializers are actually implemented as functions. We can use these initializer functions as function pointers.

```rust
enum Status {
    Value(u32),
    Stop,
}

let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
```

#### Return Closures

Closures are represented by traits, which means you can’t return closures directly. You’re not allowed to use the function pointer `fn` as a return type.

```rust
// compile error
fn returns_closure() -> dyn Fn(i32) -> i32 {
    |x| x + 1
}

// ok
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

### 19.5 Macros

The term _macro_ refers to a family of features in Rust: _**declarative**_ **macros with `macro_rules!`** and **three kinds of** _**procedural**_ **macros**:

* Custom **`#[derive]` macros** that specify code added with the `derive` attribute used on structs and enums
* **Attribute-like macros** that define custom attributes usable on any item
* **Function-like macros** that look like function calls but operate on the tokens specified as their argument

#### The Difference Between Maros and Functions

Fundamentally, macros are a way of writing code that writes other code, which is known as _metaprogramming_. All of macros _expand_ to produce more code than the code you’ve written manually.

A function signature must declare the number and type of parameters the function has. Macros, on the other hand, can take a variable number of parameters.

Macros are expanded before the compiler interprets the meaning of the code.

You must define macros or bring them into scope _before_ you call them in a file, as opposed to functions you can define anywhere and call anywhere.

#### Declarative Macros with `macro_rules!` for General Metaprogramming

**Declarative macros** compare a value to patterns that are associated with particular code: in this situation, the value is the literal Rust source code passed to the macro; the patterns are compared with the structure of that source code; and the code associated with each pattern, when matched, replaces the code passed to the macro.

To define a macro, you use the `macro_rules!` construct. 

The `#[macro_export]` annotation indicates that this macro should be made available whenever the crate in which the macro is defined is brought into scope. Without this annotation, the macro can’t be brought into scope.

```rust
#[macro_export]
macro_rules! vec {
    () => (
        $crate::__rust_force_expr!($crate::vec::Vec::new())
    );
    ($elem:expr; $n:expr) => (
        $crate::__rust_force_expr!($crate::vec::from_elem($elem, $n))
    );
    ($($x:expr),+ $(,)?) => (
        $crate::__rust_force_expr!(<[_]>::into_vec(box [$($x),+]))
    );
}
```

There are some strange edge cases with `macro_rules!`. In the future, Rust will have a second kind of declarative macro and `macro_rules!` will be effectively deprecated. 

#### Procedural Macros for Generating Code from Attributes <a id="procedural-macros-for-generating-code-from-attributes"></a>

**Procedural macros** accept some code as an input, operate on that code, and produce some code as an output rather than matching against patterns and replacing the code with other code as declarative macros do.

When creating procedural macros, the definitions must reside in their own crate with a special crate type.

```rust
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream { }
```

#### Write a Custom `derive` Macro <a id="how-to-write-a-custom-derive-macro"></a>

Rust doesn’t have reflection capabilities.

The convention for structuring crates and macro crates is as follows: for a crate named `foo`, a custom derive procedural macro crate is called `foo_derive`.

{% embed url="https://github.com/StoneYunZhao/the-rust-programming-language/tree/master/ch19\_macros" %}

```rust
#[proc_macro_derive(XXX)]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream { ... }

#[derive(XXX)]
sturct AAA;
```

#### Attribute-like macros <a id="attribute-like-macros"></a>

Attribute-like macros allow you to create new attributes.

`derive` only works for structs and enums; attributes can be applied to other items as well, such as functions.

```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream { ... }

#[route(GET, "/")]
fn index() { ... }
```

#### Function-like macros <a id="function-like-macros"></a>

Function-like macros define macros that look like function calls.

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream { ... }

let sql = sql!(SELECT * FROM posts WHERE id=1);
```

## 20. Final Project: Building a Multithreaded Web Server



