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

variables, references are immutable by default.

use `mut` before the variable name to make a variable mutable.

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

### 5.1 Defining and Instantiating Structs



### 5.2 An Example Program Using Structs



### 5.3 Method Syntax



