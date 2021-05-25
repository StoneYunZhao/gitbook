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
```

