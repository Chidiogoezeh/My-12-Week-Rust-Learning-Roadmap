Ownership is **the most important concept in Rust**. If you understand ownership well, borrowing, lifetimes, and Rust's memory safety become much easier.

Unlike languages with a garbage collector (Java, Go, C#) or manual memory management (C/C++), Rust uses **ownership** to manage memory safely **without a garbage collector**.

By the end of this lesson, you should understand:

- What ownership is
- Why Rust uses ownership
- The three ownership rules
- Moving ownership
- Copy types
- Cloning
- Ownership and functions
- Returning ownership
- References (preview)
- Common ownership errors
- Best practices

---

# 1. What is Ownership?

Ownership is Rust's way of managing memory.

Think of ownership like owning a house.

```text
House
  │
  ▼
Alice
```

Alice owns the house.

Only **one owner** can own the house at a time.

If Alice sells it to Bob:

```text
House
  │
  ▼
Bob
```

Alice no longer owns it.

The same idea applies to Rust variables.

---

# 2. Why Does Rust Need Ownership?

Most programming languages use one of two approaches:

- **Garbage Collection (GC)** – Automatically frees unused memory (Java, Go, C#).
- **Manual Memory Management** – The programmer allocates and frees memory (C/C++).

Rust does neither.

Instead, Rust knows exactly **who owns every value** and automatically frees memory when the owner goes out of scope.

Benefits:

- No memory leaks (in normal safe Rust)
- No dangling pointers
- No double frees
- High performance
- Memory safety without a garbage collector

---

# 3. The Three Ownership Rules

Every value in Rust follows these rules:

1. **Each value has exactly one owner.**
2. **There can only be one owner at a time.**
3. **When the owner goes out of scope, the value is dropped (memory is freed).**

These three rules explain nearly every ownership-related error.

---

# 4. Scope and Ownership

```rust
fn main() {
    {
        let name = String::from("Alice");

        println!("{}", name);

    } // name is dropped here
}
```

When execution leaves the block, Rust automatically frees the memory used by `name`.

No `free()`.

No garbage collector.

No memory leak.

---

# 5. Stack vs Heap (Simplified)

Rust stores different values in different places.

Stack:

```text
Age = 25
```

Heap:

```text
String
 ┌─────────────┐
 │ Alice       │
 └─────────────┘
```

Simple values (integers, booleans, chars) live on the stack.

Strings (`String`) store their text on the heap.

Ownership mainly matters for heap-allocated values.

---

# 6. Moving Ownership

Consider this:

```rust
fn main() {
    let name1 = String::from("Alice");

    let name2 = name1;

    println!("{}", name2);
}
```

This works.

But:

```rust
fn main() {
    let name1 = String::from("Alice");

    let name2 = name1;

    println!("{}", name1);
}
```

Produces an error similar to:

```text
borrow of moved value: `name1`
```

Why?

Ownership moved from `name1` to `name2`.

After:

```rust
let name2 = name1;
```

The situation becomes:

```text
name1  (Incorrect)

name2 ─────► "Alice"
```

Only `name2` owns the data now.

---

# 7. Why Doesn't Rust Copy Strings?

Suppose Rust copied heap data automatically:

```text
name1 ─────► "Alice"

name2 ─────► "Alice"
```

Now who frees the memory?

Both?

Neither?

That could lead to crashes or memory corruption.

Instead, Rust moves ownership.

---

# 8. Copy Types

Not every type moves.

Simple types implement the `Copy` trait.

Example:

```rust
fn main() {
    let x = 10;

    let y = x;

    println!("{}", x);
    println!("{}", y);
}
```

Output

```text
10
10
```

Because integers are copied.

Types that are `Copy` include:

- Integers
- Floating-point numbers
- Booleans
- Characters
- Tuples containing only `Copy` types

---

# 9. String is Different

```rust
fn main() {
    let a = String::from("Rust");

    let b = a;
}
```

Ownership moves.

The original variable becomes invalid.

---

# 10. Cloning

Sometimes you want two independent copies.

Use `.clone()`.

```rust
fn main() {
    let a = String::from("Rust");

    let b = a.clone();

    println!("{}", a);
    println!("{}", b);
}
```

Output

```text
Rust
Rust
```

Memory layout:

```text
a ─────► "Rust"

b ─────► "Rust"
```

These are separate heap allocations.

---

# 11. Move vs Clone

Move:

```rust
let b = a;
```

- Fast
- No duplication
- Old variable becomes invalid

Clone:

```rust
let b = a.clone();
```

- Copies data
- Uses more memory
- Both variables remain valid

---

# 12. Ownership and Functions

Ownership also moves into functions.

```rust
fn print_name(name: String) {
    println!("{}", name);
}

fn main() {
    let person = String::from("Alice");

    print_name(person);

    println!("{}", person);
}
```

Error:

```text
borrow of moved value: `person`
```

Ownership moved into `print_name`.

---

# 13. Returning Ownership

One solution is to return ownership.

```rust
fn print_name(name: String) -> String {
    println!("{}", name);

    name
}

fn main() {
    let person = String::from("Alice");

    let person = print_name(person);

    println!("{}", person);
}
```

Ownership is returned to the caller.

---

# 14. References (Preview)

Instead of moving ownership, you can borrow the value.

```rust
fn print_name(name: &String) {
    println!("{}", name);
}

fn main() {
    let person = String::from("Alice");

    print_name(&person);

    println!("{}", person);
}
```

Output

```text
Alice
Alice
```

Ownership never leaves `person`.

Borrowing is covered in detail in the next lesson.

---

# 15. Ownership with Multiple Variables

```rust
fn main() {
    let a = String::from("Hello");

    let b = a;

    let c = b;

    println!("{}", c);
}
```

Ownership chain:

```text
a (Incorrect)

b (Incorrect)

c ─────► "Hello"
```

Only the final owner is valid.

---

# 16. Ownership Inside a Scope

```rust
fn main() {
    let name = String::from("Alice");

    {
        let owner = name;

        println!("{}", owner);
    }
}
```

When `owner` leaves scope, the memory is freed.

`name` no longer owns the value.

---

# 17. Common Beginner Mistakes

### Using a moved value

```rust
let a = String::from("Rust");

let b = a;

println!("{}", a);
```

---

### Forgetting to clone

```rust
let a = String::from("Rust");

let b = a.clone();

println!("{}", a);
```

---

### Passing ownership accidentally

```rust
fn consume(text: String) {
    println!("{}", text);
}
```

If you still need `text` afterwards, pass a reference instead:

```rust
fn consume(text: &String) {
    println!("{}", text);
}
```

---

# 18. Best Practices

- Don't call `.clone()` unless you truly need another owned copy.
- Prefer borrowing (`&T`) when functions only need to read data.
- Remember that primitive types like `i32` and `bool` are copied automatically.
- Read compiler ownership errors carefully—they usually explain exactly what's happening.
- Think of ownership as a transfer of responsibility for cleaning up memory.

---

# Mini Project: Book Ownership

```rust
fn display_book(book: &String) {
    println!("Book: {}", book);
}

fn main() {
    let book = String::from("The Rust Programming Language");

    display_book(&book);

    println!("Still owned by main: {}", book);

    let backup = book.clone();

    println!("Backup copy: {}", backup);
}
```

### Expected Output

```text
Book: The Rust Programming Language
Still owned by main: The Rust Programming Language
Backup copy: The Rust Programming Language
```

---

# Visual Summary

### Move

```text
Before

owner1 ─────► "Rust"

After

owner1 (Incorrect)

owner2 ─────► "Rust"
```

### Clone

```text
owner1 ─────► "Rust"

owner2 ─────► "Rust"
```

### Borrow

```text
Owner ─────► "Rust"
              ▲
              │
          Reference
```

The owner remains responsible for the memory.

---

# Key Takeaways

- Every value in Rust has **one owner**.
- Ownership can **move** from one variable to another.
- When the owner goes out of scope, Rust automatically frees the memory.
- Simple types like integers are **copied**, not moved.
- Heap-allocated types like `String` are **moved** by default.
- Use `.clone()` when you need an independent copy.
- Functions take ownership unless you pass a **reference**.
- Borrowing (`&T`) lets you use data without taking ownership and is the foundation for writing efficient Rust code.

Once ownership makes sense, you're ready for the next crucial topic in the roadmap: **borrowing and references**, where you'll learn how Rust lets multiple parts of your program safely access data without transferring ownership.
