---
description: 'https://doc.rust-lang.org/stable/rust-by-example/'
---

# Rust by Example

## 1. Hello World

### 1.2 Formatted Print

* `fmt::Debug`: Uses the `{:?}` marker. Format text for debugging purposes.
* `fmt::Display`: Uses the `{}` marker. Format text in a more elegant, user friendly fashion.

#### 1.2.1 Debug

_All_ types can `derive` \(automatically create\) the `fmt::Debug` implementation. This is not true for `fmt::Display` which must be manually implemented.

Rust also provides "pretty printing" with `{:#?}`.

#### 1.2.2 Display

`fmt::Display` is not implemented for `Vec<T>` or for any other generic containers.

`{:b}` requires `fmt::Binary` to be implemented.

## 2. Primitives

#### Scalar Types <a id="scalar-types"></a>

* signed integers: `i8`, `i16`, `i32`, `i64`, `i128` and `isize` \(pointer size\)
* unsigned integers: `u8`, `u16`, `u32`, `u64`, `u128` and `usize` \(pointer size\)
* floating point: `f32`, `f64`
* `char` Unicode scalar values like `'a'`, `'α'` and `'∞'` \(4 bytes each\)
* `bool` either `true` or `false`
* and the unit type `()`, whose only possible value is an empty tuple: `()`

Despite the value of a unit type being a tuple, it is not considered a compound type because it does not contain multiple values.

#### Compound Types <a id="compound-types"></a>

* arrays like `[1, 2, 3]`
* tuples like `(1, true)`

Integers default to `i32` and floats to `f64`.

### 2.2 Tuples

A tuple is a collection of values of different types.

### 2.3 Arrays and Slices

An array is a collection of objects of the same type `T`, stored in contiguous memory. Arrays are created using brackets `[]`, and their length, which is known at compile time, is part of their type signature `[T; length]`.

Slices are similar to arrays, but their length is not known at compile time. Instead, a slice is a two-word object, the first word is a pointer to the data, and the second word is the length of the slice.

## 3. Custom Types

Rust custom data types are formed mainly through the two keywords:

* `struct`: define a structure
* `enum`: define an enumeration

Constants can also be created via the `const` and `static` keywords.

### 3.1 Structures

```rust
// A unit struct
struct Unit;

// A tuple struct
struct Pair(i32, f32);

// Classic struct
struct Point {
    x: f32,
    y: f32,
}
```

### 3.2 Enums

The `enum` keyword allows the creation of a type which may be one of a few different variants. Any variant which is valid as a `struct` is also valid as an `enum`.

The most common place you'll see this is in `impl` blocks using the `Self` alias.

```rust
enum VeryVerboseEnumOfThingsToDoWithNumbers {
    Add,
    Subtract,
}

// distinguish self and Self
impl VeryVerboseEnumOfThingsToDoWithNumbers {
    fn run(&self, x: i32, y: i32) -> i32 {
        match self {
            Self::Add => x + y,
            Self::Subtract => x - y,
        }
    }
}

// C-like enums
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}
```

### 3.3 Constants

* `const`: An unchangeable value \(the common case\).
* `static`: A possibly `mut`able variable with `'static` lifetime. The static lifetime is inferred and does not have to be specified. Accessing or modifying a mutable static variable is `unsafe`.

```rust
static LANGUAGE: &str = "Rust";
const THRESHOLD: i32 = 10;
```

## 4. Variable Bindings

Values \(like literals\) can be bound to variables, using the `let` binding.

### 4.1 Mutability

Variable bindings are immutable by default, but this can be overridden using the `mut` modifier.

### 4.2 Scope and Shadowing

Variable bindings have a scope, and are constrained to live in a _block_. 

### 4.3 Declare First

It's possible to declare variable bindings first, and initialize them later.

## 5. Types

Rust provides no implicit type conversion \(coercion\) between primitive types. But, explicit type conversion \(casting\) can be performed using the `as` keyword.

Numeric literals can be type annotated by adding the type as a suffix.

The `type` statement can be used to give a new name to an existing type.

## 6. Conversion

The `From` trait allows for a type to define how to create itself from another type.

The [`Into`](https://doc.rust-lang.org/std/convert/trait.Into.html) trait is simply the reciprocal of the `From` trait.

Similar to [`From` and `Into`](https://doc.rust-lang.org/stable/rust-by-example/conversion/from_into.html), [`TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html) and [`TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html) are generic traits for converting between types.

## 7. Expressions

If the last expression of the block ends with a semicolon, the return value will be `()`.

## 8. Flow of Control

if/else, loop/break/continue, while, for/in, match, if let, while let

## 9. Functions

### 9.1 Methods

Methods are functions attached to objects. These methods have access to the data of the object and its other methods via the `self` keyword.

### 9.2 Closures

Closures are functions that can capture the enclosing environment.

## 10. Modules

By default, the items in a module have private visibility, but this can be overridden with the `pub` modifier.

## 11. Crates

## 12. Cargo

## 13. Attributes

An attribute is metadata applied to some module, crate or item. This metadata can be used to/for:

* conditional compilation of code
* set crate name, version and type \(binary or library\)
* disable lints \(warnings\)
* enable compiler features \(macros, glob imports, etc.\)
* link to a foreign library
* mark functions as unit tests
* mark functions that will be part of a benchmark

When attributes apply to a whole crate, their syntax is `#![crate_attribute]`, and when they apply to a module or item, the syntax is `#[item_attribute]` .

Attributes can take arguments with different syntaxes:

* `#[attribute = "value"]`
* `#[attribute(key = "value")]`
* `#[attribute(value)]`

### 13.2 Crates

The `crate_type` attribute can be used to tell the compiler whether a crate is a binary or a library \(and even which type of library\), and the `crate_name` attribute can be used to set the name of the crate.

Both the `crate_type` and `crate_name` attributes have **no** effect whatsoever when using Cargo. Since Cargo is used for the majority of Rust projects, this means real-world uses of `crate_type` and `crate_name` are relatively limited.

## 14. Generics

### 14.1 Functions

Using generic functions sometimes requires explicitly specifying type parameters. This may be the case if the function is called where the return type is generic, or if the compiler doesn't have enough information to infer the necessary type parameters.

A function call with explicitly specified type parameters looks like: `fun::<A, B, ...>()`.

### 14.4 Bounds

When working with generics, the type parameters often must use traits as _bounds_ to stipulate what functionality a type implements. 

A consequence of how bounds work is that even if a `trait` doesn't include any functionality, you can still use it as a bound. `Eq` and `Copy` are examples of such `trait`s from the `std` library.

### 14.5 Multiple bounds

Multiple bounds for a single type can be applied with a `+`.

### 14.6 Where clauses

`where` clauses can apply bounds to arbitrary types, rather than just to type parameters.

### 14.9 Phantom type parameters

A phantom type parameter is one that doesn't show up at runtime, but is checked statically \(and only\) at compile time.

## 15. Scoping Rules

### 15.1 RAII

Variables in Rust do more than just hold data in the stack: they also _own_ resources, e.g. `Box<T>` owns memory in the heap. Rust enforces RAII \(Resource Acquisition Is Initialization\), so whenever an object goes out of scope, its destructor is called and its owned resources are freed.

The notion of a destructor in Rust is provided through the `Drop` trait. The destructor is called when the resource goes out of scope. 

### 15.2 Ownership and moves

**Resources can only have one owner**. Note that not all variables own resources \(e.g. references\).

When doing assignments \(`let x = y`\) or passing function arguments by value \(`foo(x)`\), the _ownership_ of the resources is transferred. In Rust-speak, this is known as a _**move**_. After moving resources, the previous owner can no longer be used.

### 15.3 Borrowing

Most of the time, we'd like to access data without taking ownership over it. To accomplish this, Rust uses a _**borrowing**_ mechanism. Instead of passing objects by value \(`T`\), objects can be passed by reference \(`&T`\).

The compiler statically guarantees \(via its borrow checker\) that references _always_ point to valid objects. That is, while references to an object exist, the object cannot be destroyed.

Mutable data can be **mutably borrowed** using `&mut T`. This is called a _mutable reference_ and gives read/write access to the borrower. In contrast, `&T` borrows the data via an immutable reference, and the borrower can read the data but not modify it.

`&'static str` is a reference to a string allocated in read only memory. string literals have type `&'static str`

Data can be immutably borrowed any number of times, but while immutably borrowed, the original data can't be mutably borrowed. On the other hand, only _one_ mutable borrow is allowed at a time. The original data can be borrowed again only _after_ the mutable reference has been used for the last time.

When doing pattern matching or destructuring via the `let` binding, the **`ref` keyword** can be used to take references to the fields of a struct/tuple. 

## 16. Traits

## 17. `macro_rules!`

## 18. Error Handling

## 19. Std library types

## 20. Std misc

## 21. Testing

## 22. Unsafe Operations

## 23. Compatibility

## 24. Meta



