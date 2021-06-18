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



