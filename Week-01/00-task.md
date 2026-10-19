# Week 1 – Rust Fundamentals

## Topics

- Installing tools
- Variables
- Mutability
- Data types
- Functions
- Ownership
- Borrowing
- References
- Shadowing

Example:

```rust
fn main() {
    let name = String::from("Alice");
    greet(&name);
}

fn greet(name: &String) {
    println!("{}", name);
}
```

### Project

Build a simple **CLI Calculator**.

Features:

- Addition
- Subtraction
- Multiplication
- Division

Link: https://github.com/Chidiogoezeh/rust_calculator