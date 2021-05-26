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



