Mutability is one of the core ideas in Rust and is closely tied to Rust's safety guarantees. Unlike many languages where variables are mutable by default, Rust takes the opposite approach: **everything is immutable unless you explicitly say otherwise**.

By the end of this lesson, you should understand:

* What mutability is
* Why Rust defaults to immutable variables
* How to create mutable variables
* Mutability vs. shadowing
* Mutable references
* Mutable collections
* Common mistakes
* Best practices

---

# 1. What is Mutability?

**Mutability** means that the value stored in a variable can be changed after it has been created.

Imagine a whiteboard.

An **immutable** variable is like writing with permanent marker—you can't erase or change it.

A **mutable** variable is like writing with a dry-erase marker—you can update it whenever you need to.

---

# 2. Immutable Variables (Default)

In Rust, variables are immutable unless marked with `mut`.

```rust
fn main() {
    let username = "Alice";

    println!("{}", username);
}
```

This works because `username` is never modified.

If you try to change it:

```rust
fn main() {
    let username = "Alice";

    username = "Bob";
}
```

Rust produces an error similar to:

```text
error[E0384]: cannot assign twice to immutable variable `username`
```

This is intentional.

---

# 3. Creating Mutable Variables

Use the `mut` keyword.

```rust
fn main() {
    let mut username = "Alice";

    println!("{}", username);

    username = "Bob";

    println!("{}", username);
}
```

Output:

```text
Alice
Bob
```

Here:

```rust
let mut username = "Alice";
```

means:

> "This variable is allowed to change."

---

# 4. Why Rust Prefers Immutable Variables

Rust encourages you to make things mutable **only when necessary**.

Benefits:

* Prevents accidental changes
* Makes code easier to understand
* Safer when multiple parts of a program interact
* Helps prevent bugs in concurrent programs
* Allows the compiler to make better optimizations

Example:

```rust
let tax_rate = 0.075;
```

Since a tax rate shouldn't change unexpectedly, making it immutable communicates your intent clearly.

---

# 5. Updating Mutable Variables

```rust
fn main() {
    let mut balance = 100;

    balance = 150;

    println!("{}", balance);
}
```

Output:

```text
150
```

---

# 6. Multiple Updates

```rust
fn main() {
    let mut score = 0;

    score += 10;

    score += 5;

    score -= 2;

    println!("{}", score);
}
```

Output:

```text
13
```

---

# 7. Mutable Strings

A `String` owns its data and can be modified when declared mutable.

```rust
fn main() {
    let mut name = String::from("Alice");

    name.push('!');

    println!("{}", name);
}
```

Output:

```text
Alice!
```

You can also append text.

```rust
fn main() {
    let mut message = String::from("Hello");

    message.push_str(" Rust");

    println!("{}", message);
}
```

Output:

```text
Hello Rust
```

---

# 8. Mutable Vectors

Collections often need to change.

```rust
fn main() {
    let mut numbers = vec![1, 2, 3];

    numbers.push(4);

    println!("{:?}", numbers);
}
```

Output:

```text
[1, 2, 3, 4]
```

Without `mut`, `push()` would fail because it modifies the vector.

---

# 9. Mutable HashMaps

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert("Alice", 95);
    scores.insert("Bob", 88);

    println!("{:?}", scores);
}
```

`insert()` changes the map, so it must be mutable.

---

# 10. Mutability vs. Shadowing

Many beginners confuse these concepts.

## Mutability

Same variable, new value.

```rust
fn main() {
    let mut age = 20;

    age = 21;

    println!("{}", age);
}
```

The variable itself is updated.

---

## Shadowing

Creates a completely new variable.

```rust
fn main() {
    let age = 20;

    let age = age + 1;

    println!("{}", age);
}
```

The first `age` is replaced by a new one.

---

# 11. When to Use `mut`

Use `mut` when the **same piece of data** naturally changes over time.

Good examples:

```rust
let mut score = 0;
let mut counter = 1;
let mut shopping_cart = Vec::new();
let mut username = String::new();
```

Avoid using `mut` if the value never changes.

---

# 12. Mutable References

Rust allows you to borrow data mutably.

```rust
fn add_exclamation(text: &mut String) {
    text.push('!');
}

fn main() {
    let mut message = String::from("Hello");

    add_exclamation(&mut message);

    println!("{}", message);
}
```

Output:

```text
Hello!
```

Notice two uses of `mut`:

* The variable itself is mutable.
* The function receives a mutable reference (`&mut String`).

---

# 13. Only One Mutable Reference at a Time

Rust prevents multiple mutable references to the same data at the same time.

This won't compile:

```rust
fn main() {
    let mut value = String::from("Rust");

    let a = &mut value;
    let b = &mut value;

    println!("{} {}", a, b);
}
```

Rust rejects this because two mutable references could modify the same data simultaneously.

Correct approach:

```rust
fn main() {
    let mut value = String::from("Rust");

    {
        let a = &mut value;
        a.push('!');
    }

    let b = &mut value;
    b.push('?');

    println!("{}", value);
}
```

Output:

```text
Rust!?
```

The first mutable borrow ends before the second begins.

---

# 14. Mutability with Structs

```rust
struct User {
    username: String,
    age: u32,
}

fn main() {
    let mut user = User {
        username: String::from("Alice"),
        age: 25,
    };

    user.age = 26;

    println!("{}", user.age);
}
```

Output:

```text
26
```

The entire struct is mutable, so its fields can be updated.

---

# 15. Constants Are Never Mutable

```rust
const MAX_USERS: u32 = 100;
```

This is invalid:

```rust
const mut MAX_USERS: u32 = 100;
```

Constants are always immutable because their values are fixed at compile time.

---

# 16. Common Beginner Mistakes

## Forgetting `mut`

```rust
let count = 5;

count += 1;
```

Error:

```text
cannot assign to immutable variable
```

Fix:

```rust
let mut count = 5;

count += 1;
```

---

## Using `mut` Everywhere

This works:

```rust
let mut name = "Alice";
```

But if `name` never changes, `mut` is unnecessary. Prefer the smallest amount of mutability needed.

---

## Expecting Shadowing to Behave Like Mutability

```rust
let value = 10;

let value = 20;
```

This creates a new variable; it does not modify the original one.

---

# Mini Project: Bank Account

```rust
fn main() {
    let account_holder = "Daud";
    let mut balance = 5000;

    println!("Account Holder: {}", account_holder);
    println!("Balance: ₦{}", balance);

    balance += 2500;

    println!("After Deposit: ₦{}", balance);

    balance -= 1000;

    println!("After Withdrawal: ₦{}", balance);
}
```

Expected output:

```text
Account Holder: Daud
Balance: ₦5000
After Deposit: ₦7500
After Withdrawal: ₦6500
```

---

# Practice Exercises

1. Create a mutable variable `temperature` with the value `25`. Increase it by `5` and print the result.
2. Create a mutable `Vec` containing `10`, `20`, and `30`. Add `40` and print the vector.
3. Write a function that accepts a mutable `String` reference and appends `" World"` to it.
4. Create a mutable struct representing a book with `title` and `pages`, then update the page count.

---

# Best Practices

* Default to **immutable** variables.
* Add `mut` only when a value truly needs to change.
* Prefer **shadowing** when transforming a value into a new form (especially if the type changes).
* Use mutable references (`&mut`) only when a function needs to modify the caller's data.
* Keep mutable state as small and localized as possible.

---

## Key Takeaways

* `mut` allows a variable's value to change.
* Variables are immutable by default in Rust.
* Mutability applies to owned values like `String`, `Vec`, `HashMap`, and structs.
* Mutable references (`&mut`) let functions modify borrowed data safely.
* Rust's borrowing rules prevent multiple simultaneous mutable references, eliminating many classes of bugs before your program even runs.

Understanding mutability thoroughly will make the next topics—**ownership**, **borrowing**, and **references**—much easier, because they build directly on these rules.
