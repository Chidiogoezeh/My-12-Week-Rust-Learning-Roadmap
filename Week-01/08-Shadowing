**Shadowing** is an important Rust concept because it looks similar to mutability, but it behaves differently. Understanding this distinction will help for better understanding with **ownership, borrowing, type conversions, and data processing in backend applications**.

# Rust Shadowing

## 1. What is Shadowing?

**Shadowing means declaring a new variable with the same name as an existing variable.**

You use `let` again:

```rust
fn main() {
    let x = 10;

    let x = 20;

    println!("{}", x);
}
```

Output:

```text
20
```

What happened?

```text
First variable:
x → 10

Second variable:
x → 20
```

The second `x` **shadows** the first `x`.

The original variable still existed for the appropriate scope, but you can no longer refer to it by the name `x` from that point onward.

---

# 2. Shadowing vs Mutability

This distinction is extremely important.

### Mutability

```rust
fn main() {
    let mut x = 10;

    x = 20;

    println!("{}", x);
}
```

Here, you're changing the **same variable**.

```text
x
│
├── 10
│
└── 20
```

### Shadowing

```rust
fn main() {
    let x = 10;

    let x = 20;

    println!("{}", x);
}
```

Here, you're creating a **new variable** with the same name.

```text
x₁ → 10

x₂ → 20
```

The key syntax difference:

```rust
// Mutation
x = 20;

// Shadowing
let x = 20;
```

---

# 3. Why Is Shadowing Useful?

One of the biggest reasons Rust developers use shadowing is **transforming a value while keeping a meaningful name**.

For example, suppose you receive a number as text:

```rust
let input = "42";
```

Initially:

```text
input → &str
```

You can parse it:

```rust
let input: i32 = input.parse().unwrap();
```

Now:

```text
input → i32
```

Complete example:

```rust
fn main() {
    let input = "42";

    let input: i32 = input.parse().unwrap();

    println!("{}", input);
}
```

Output:

```text
42
```

This is something you **cannot do with `mut`**.

---

# 4. Shadowing Allows Type Changes

Consider:

```rust
fn main() {
    let value = "100";

    let value: i32 = value.parse().unwrap();

    println!("{}", value);
}
```

Initially:

```text
value = "100"
type = &str
```

After shadowing:

```text
value = 100
type = i32
```

This won't work with `mut`:

```rust
fn main() {
    let mut value = "100";

    value = 100;
}
```

Rust will reject it because a mutable variable must retain the same type.

---

# 5. Shadowing Multiple Times

You can shadow a variable more than once.

```rust
fn main() {
    let number = 10;

    let number = number + 5;

    let number = number * 2;

    let number = number - 4;

    println!("{}", number);
}
```

Let's follow it:

```text
number = 10
number = 15
number = 30
number = 26
```

Output:

```text
26
```

This style can be very readable when you're progressively transforming data.

---

# 6. Shadowing Doesn't Require `mut`

Notice:

```rust
let number = 10;
let number = number + 5;
```

Neither variable is mutable.

That's perfectly valid.

Why?

Because you're **creating a new variable**, not changing the existing one.

---

# 7. Shadowing with Different Types

This is particularly useful in backend programming.

```rust
fn main() {
    let user_id = "123";

    let user_id: u64 = user_id.parse().unwrap();

    println!("User ID: {}", user_id);
}
```

First:

```text
user_id → String/text
```

Then:

```text
user_id → u64
```

This is common when processing:

- HTTP parameters
- Form input
- JSON data
- Environment variables
- Database values

---

# 8. Shadowing and Strings

Here's another useful example:

```rust
fn main() {
    let username = "  daud  ";

    let username = username.trim();

    println!("{}", username);
}
```

Output:

```text
daud
```

You can progressively transform data:

```rust
let username = "  daud  ";
let username = username.trim();
let username = username.to_uppercase();

println!("{}", username);
```

Output:

```text
DAUD
```

Each `let username` creates a new binding.

---

# 9. Shadowing in a Function

Shadowing is also common when preparing data inside functions.

```rust
fn calculate_price() {
    let price = "100";

    let price: f64 = price.parse().unwrap();

    let price = price * 1.075;

    println!("Price: {}", price);
}

fn main() {
    calculate_price();
}
```

Conceptually:

```text
"100"
  ↓
100.0
  ↓
107.5
```

The variable name `price` remains meaningful throughout the transformation.

---

# 10. Shadowing Inside Blocks

Shadowing can also happen within scopes.

```rust
fn main() {
    let x = 10;

    {
        let x = 20;

        println!("Inside: {}", x);
    }

    println!("Outside: {}", x);
}
```

Output:

```text
Inside: 20
Outside: 10
```

The inner `x` shadows the outer `x` **only inside that block**.

Think of it as:

```text
Outside scope
x = 10
│
├── Inside scope
│   x = 20
│
└── Back outside
    x = 10
```

---

# 11. Shadowing vs Scope

These concepts are related but different.

### Scope

Determines **where a variable can be used**.

```rust
{
    let x = 10;
}
```

`x` only exists within that block.

### Shadowing

Determines what happens when you create another variable with the same name.

```rust
let x = 10;
let x = 20;
```

The second `x` shadows the first.

---

# 12. Shadowing with Expressions

You can use the previous value to create the next one:

```rust
fn main() {
    let price = 100;

    let price = price + 10;

    let price = price * 2;

    println!("{}", price);
}
```

Result:

```text
220
```

Calculation:

```text
100
 ↓ +10
110
 ↓ ×2
220
```

---

# 13. Shadowing with Functions

You can also use functions during transformations:

```rust
fn main() {
    let name = "daud";

    let name = name.to_uppercase();

    println!("{}", name);
}
```

Output:

```text
DAUD
```

---

# 14. Shadowing Is Especially Useful with Input

Imagine you're building a backend API.

A user sends:

```text
age = "25"
```

HTTP input often arrives as text.

You might do:

```rust
let age = "25";

let age: u32 = age.parse().unwrap();

println!("Age: {}", age);
```

Now you have:

```text
HTTP input
   ↓
"25"
   ↓
parse()
   ↓
25
   ↓
u32
```

This pattern becomes extremely useful when working with:

- REST APIs
- Query parameters
- Path parameters
- Environment variables
- Configuration
- Database data

---

# 15. A Backend Example

Imagine an endpoint receives:

```text
GET /users/42
```

The `"42"` part can initially be represented as text.

You might transform it into an integer:

```rust
fn main() {
    let user_id = "42";

    let user_id: u64 = user_id.parse().unwrap();

    println!("Looking for user {}", user_id);
}
```

Output:

```text
Looking for user 42
```

Later, when you learn Axum, you'll see similar transformations when working with request parameters.

---

# 16. Shadowing and Immutability

This is an important Rust design principle.

Consider:

```rust
let name = "daud";

let name = name.trim();

let name = name.to_uppercase();
```

We never use:

```rust
let mut name
```

Why?

Because none of the individual variables need to change.

Instead, we're saying:

> "Give me a new version of this value."

This works nicely with Rust's emphasis on safety and predictable data flow.

---

# 17. Common Mistake

Don't confuse:

```rust
let x = 10;
let x = 20;
```

with:

```rust
let mut x = 10;
x = 20;
```

They look similar, but they're different operations.

### Shadowing

```rust
let x = 10;
let x = 20;
```

Creates a new binding.

### Mutation

```rust
let mut x = 10;
x = 20;
```

Changes the value stored in the existing mutable binding.

---

# 18. When Should You Use Shadowing?

Use shadowing when you're **transforming a value**.

For example:

```rust
let input = "42";
let input: i32 = input.parse().unwrap();
```

Or:

```rust
let username = "  DAUD ";
let username = username.trim();
let username = username.to_lowercase();
```

Shadowing is particularly appropriate when the **conceptual identity remains the same**, but its representation changes.

---

# 19. When Should You Use `mut`?

Use `mut` when you genuinely need the **same variable to change over time**.

For example:

```rust
let mut total = 0;

total += 10;
total += 20;
total += 30;

println!("{}", total);
```

Here, `total` represents an accumulating value, so mutation makes sense.

---

# 20. A Realistic Backend Example

Imagine calculating an order total:

```rust
fn main() {
    let price = "1000";

    let price: f64 = price.parse().unwrap();

    let tax = price * 0.075;

    let total = price + tax;

    println!("Price: {}", price);
    println!("Tax: {}", tax);
    println!("Total: {}", total);
}
```

Output:

```text
Price: 1000
Tax: 75
Total: 1075
```

Notice the flow:

```text
"1000"
   ↓
parse
   ↓
1000.0
   ↓
calculate tax
   ↓
75.0
   ↓
calculate total
   ↓
1075.0
```

This type of transformation is something you'll encounter constantly in backend development.

---

# 21. Important Rules to Remember

### Rule 1

Shadowing uses `let`:

```rust
let x = 10;
let x = 20;
```

### Rule 2

Mutation doesn't use `let` again:

```rust
let mut x = 10;
x = 20;
```

### Rule 3

Shadowing can change types:

```rust
let x = "10";
let x: i32 = x.parse().unwrap();
```

### Rule 4

Mutation cannot change types:

```rust
let mut x = "10";
x = 10; // Incorrect
```

### Rule 5

Shadowing can happen repeatedly:

```rust
let x = 1;
let x = x + 1;
let x = x * 2;
```

### Rule 6

Inner scopes can shadow outer variables:

```rust
let x = 10;

{
    let x = 20;
}
```

---

# Practice Exercises

Before moving on, try these yourself.

### Exercise 1 — Basic Shadowing

What does this print?

```rust
fn main() {
    let x = 5;
    let x = x + 10;
    println!("{}", x);
}
```

---

### Exercise 2 — Type Conversion

Complete this:

```rust
fn main() {
    let age = "25";

    // Convert age to u32 using shadowing

    println!("{}", age);
}
```

Expected output:

```text
25
```

---

### Exercise 3 — String Transformation

Write code that transforms:

```text
"   rust backend   "
```

into:

```text
"RUST BACKEND"
```

using shadowing.

---

### Exercise 4 — Mutation or Shadowing?

Decide which is more appropriate:

**A:**

```rust
let mut count = 0;

count += 1;
count += 1;
count += 1;
```

**B:**

```rust
let input = "500";
let input: i32 = input.parse().unwrap();
```

**Answer:** A uses **mutation**; B uses **shadowing**.

---

## What you should be able to do before moving on

For your 12-week roadmap, don't just memorize the definition. You should be comfortable writing code like:

```rust
fn main() {
    let input = "  42  ";

    let input = input.trim();

    let input: i32 = input.parse().unwrap();

    let input = input * 2;

    println!("{}", input);
}
```

And be able to explain the transformation:

```text
"  42  "       → &str
     ↓ trim
"42"            → &str
     ↓ parse
42              → i32
     ↓ multiply
84              → i32
```

If you understand **why each `let` is there and why `mut` isn't needed**, you've got the core of Rust shadowing.
