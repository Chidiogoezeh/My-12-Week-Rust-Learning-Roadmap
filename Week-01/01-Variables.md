Variables are one of the first and most important concepts in Rust. They differ from many other languages because **variables are immutable by default**, which helps prevent accidental bugs.

By the end of this lesson, you should understand:

- What a variable is
- Variable declaration
- Immutability vs. mutability
- Shadowing
- Variable scope
- Constants
- Type annotations
- Naming conventions
- Basic data types stored in variables

---

# 1. What is a Variable?

A **variable** is a named location in memory used to store data.

Think of it as a labeled box:

```text
Name
 ┌─────────┐
 │ "Alice" │
 └─────────┘
```

Whenever you use `name`, Rust retrieves the value `"Alice"`.

---

# 2. Declaring Variables

In Rust, variables are declared with `let`.

```rust
fn main() {
    let name = "Alice";

    println!("{}", name);
}
```

Output

```text
Alice
```

Here:

- `let` declares the variable.
- `name` is the variable name.
- `"Alice"` is the value.

---

# 3. Variables Are Immutable by Default

This is one of Rust's defining features.

```rust
fn main() {
    let age = 20;

    age = 21;
}
```

This will **not compile**.

You'll get an error similar to:

```text
cannot assign twice to immutable variable
```

Rust prevents changing values unless you explicitly allow it.

---

# 4. Mutable Variables

Use `mut` if you want a variable to change.

```rust
fn main() {
    let mut age = 20;

    age = 21;

    println!("{}", age);
}
```

Output

```text
21
```

Here,

```rust
let mut age = 20;
```

means:

> "This variable is allowed to change."

---

# 5. Why Rust Prefers Immutable Variables

Imagine this code:

```rust
let username = "Alice";
```

If nobody can change `username`, then every part of your program knows it will always remain `"Alice"`.

Benefits:

- Fewer bugs
- Safer concurrency
- Easier debugging
- More predictable code

---

# 6. Multiple Variables

```rust
fn main() {
    let name = "Alice";
    let age = 25;
    let country = "Nigeria";

    println!("{}", name);
    println!("{}", age);
    println!("{}", country);
}
```

Output

```text
Alice
25
Nigeria
```

---

# 7. Variable Naming Rules

Good:

```rust
let first_name = "Alice";
let total_score = 90;
let is_logged_in = true;
```

Bad:

```rust
let FirstName = "Alice"; // looks like a type
let total-score = 90;    // '-' is subtraction
let 1name = "Alice";     // cannot start with a number
```

Rust uses **snake_case** for variables.

Examples:

```text
first_name
user_id
account_balance
total_price
```

---

# 8. Type Inference

Rust usually figures out the type automatically.

```rust
fn main() {
    let age = 25;
}
```

Rust knows `age` is an integer.

Similarly,

```rust
let pi = 3.14;
```

Rust infers a floating-point type.

---

# 9. Explicit Type Annotation

You can specify the type yourself.

```rust
fn main() {
    let age: u32 = 25;
}
```

Syntax:

```rust
let variable_name: Type = value;
```

Examples

```rust
let score: i32 = 95;
let price: f64 = 19.99;
let active: bool = true;
let letter: char = 'A';
```

---

# 10. Changing the Value of a Mutable Variable

```rust
fn main() {
    let mut count = 1;

    println!("{}", count);

    count = 2;

    println!("{}", count);

    count = 3;

    println!("{}", count);
}
```

Output

```text
1
2
3
```

---

# 11. Shadowing

Shadowing creates a **new variable with the same name**.

```rust
fn main() {
    let number = 10;

    let number = number + 5;

    println!("{}", number);
}
```

Output

```text
15
```

The second `number` is a new variable that shadows the first one.

---

# 12. Shadowing vs Mutability

### Mutable variable

```rust
fn main() {
    let mut number = 10;

    number = 20;
}
```

Same variable, new value.

### Shadowing

```rust
fn main() {
    let number = 10;

    let number = 20;
}
```

A new variable replaces the old one.

---

# 13. Changing Types with Shadowing

Mutability **cannot** change a variable's type.

This won't compile:

```rust
let mut age = 20;
age = "Twenty";
```

But shadowing can:

```rust
fn main() {
    let value = "42";

    let value: i32 = value.parse().unwrap();

    println!("{}", value);
}
```

Output

```text
42
```

This is common when converting input from a string to a number.

---

# 14. Constants

Constants are declared with `const`.

```rust
const PI: f64 = 3.1415926535;

fn main() {
    println!("{}", PI);
}
```

Differences between `const` and `let`:

| `let`                    | `const`                        |
| ------------------------ | ------------------------------ |
| Can be mutable (`mut`)   | Always immutable               |
| Value decided at runtime | Value known at compile time    |
| Lower snake_case         | UPPER_SNAKE_CASE by convention |

Examples:

```rust
const MAX_USERS: u32 = 100;
const TAX_RATE: f64 = 0.075;
```

---

# 15. Variable Scope

Variables exist only within the block (`{}`) where they are declared.

```rust
fn main() {
    let name = "Alice";

    {
        let age = 20;

        println!("{}", age);
    }

    println!("{}", name);
}
```

This works because `age` is only used inside its block.

But:

```rust
fn main() {
    {
        let age = 20;
    }

    println!("{}", age);
}
```

will fail because `age` is out of scope.

---

# 16. Practice Examples

### Example 1

```rust
fn main() {
    let city = "Port Harcourt";

    println!("{}", city);
}
```

---

### Example 2

```rust
fn main() {
    let mut balance = 100;

    balance += 50;

    println!("{}", balance);
}
```

Output

```text
150
```

---

### Example 3

```rust
fn main() {
    let spaces = "    ";

    let spaces = spaces.len();

    println!("{}", spaces);
}
```

Output

```text
4
```

Notice how shadowing lets `spaces` change from a `&str` to a `usize`.

---

# 17. Common Beginner Mistakes

**Forgetting `mut`**

```rust
let count = 1;
count = 2;
```

**Using `=` instead of `==` in comparisons**

```rust
if age == 18 {
    println!("Adult");
}
```

**Trying to use a variable outside its scope**

```rust
{
    let x = 5;
}

println!("{}", x);
```

---

# Mini Project: Student Profile

```rust
fn main() {
    let name = "Daud";
    let age: u32 = 22;
    let mut score = 78.5;
    let graduated = false;

    println!("Name: {}", name);
    println!("Age: {}", age);
    println!("Score: {}", score);
    println!("Graduated: {}", graduated);

    score = 85.0;

    println!("Updated Score: {}", score);
}
```

### Expected Output

```text
Name: Daud
Age: 22
Score: 78.5
Graduated: false
Updated Score: 85
```

---

## Key takeaways

- Use `let` to declare variables.
- Variables are **immutable by default**.
- Use `mut` only when a value needs to change.
- Use **shadowing** (`let` with the same name) when you want a new variable, especially if its type changes.
- Prefer descriptive `snake_case` names.
- Let Rust infer types unless an explicit type improves clarity or is required.
- Use `const` for compile-time values that never change.
- Remember that variables only exist within their scope.

Mastering these concepts will make the next topics—ownership, borrowing, functions, and structs—much easier to understand.
