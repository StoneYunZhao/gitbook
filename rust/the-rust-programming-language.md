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



