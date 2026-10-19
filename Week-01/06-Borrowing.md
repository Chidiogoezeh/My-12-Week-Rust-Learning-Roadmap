Borrowing is one of the most important concepts in Rust. It works hand-in-hand with **ownership** and is one of the reasons Rust can guarantee memory safety **without a garbage collector**.

If you truly understand borrowing, you'll find it much easier to build reliable backend applications.

By the end of this lesson, you should understand:

- What borrowing is
- Why borrowing exists
- References (`&`)
- Mutable references (`&mut`)
- Borrowing rules
- Immutable vs mutable borrowing
- Dangling references
- Borrowing with functions
- Borrowing with structs
- Common compiler errors
- Best practices

---

# 1. What is Borrowing?

Borrowing means **temporarily allowing another part of your program to use a value without taking ownership of it**.

Think of it like borrowing a library book.

- The library owns the book.
- You borrow it.
- You use it.
- You return it.
- The library still owns it.

Rust variables work the same way.

---

# 2. Why Borrowing Exists

Imagine this code:

```rust
fn main() {
    let name = String::from("Alice");

    print_name(name);

    println!("{}", name);
}

fn print_name(name: String) {
    println!("{}", name);
}
```

This **does not compile**.

Why?

Because:

```rust
print_name(name);
```

moves ownership of `name` into the function.

After that,

```rust
println!("{}", name);
```

tries to use a value that no longer belongs to `main`.

You'll get an error similar to:

```text
borrow of moved value: `name`
```

---

# 3. Borrowing with References

Instead of moving ownership, borrow the value.

```rust
fn main() {
    let name = String::from("Alice");

    print_name(&name);

    println!("{}", name);
}

fn print_name(name: &String) {
    println!("{}", name);
}
```

Output

```text
Alice
Alice
```

Notice:

```rust
&name
```

means

> "Borrow this value."

The function receives

```rust
&String
```

instead of

```rust
String
```

Ownership never changes.

---

# 4. References

The symbol

```rust
&
```

creates a reference.

```rust
let name = String::from("Alice");

let reference = &name;
```

Now

```rust
reference
```

points to

```rust
name
```

without owning it.

Visual representation:

```text
name
 ┌────────────┐
 │ "Alice"    │
 └────────────┘
        ▲
        │
reference
```

---

# 5. Multiple Immutable Borrows

Rust allows many immutable references.

```rust
fn main() {
    let city = String::from("Lagos");

    let a = &city;
    let b = &city;
    let c = &city;

    println!("{}", a);
    println!("{}", b);
    println!("{}", c);
}
```

Output

```text
Lagos
Lagos
Lagos
```

Reading is always safe.

---

# 6. Mutable Borrowing

If you want to modify data, use

```rust
&mut
```

Example

```rust
fn main() {
    let mut name = String::from("Alice");

    change_name(&mut name);

    println!("{}", name);
}

fn change_name(name: &mut String) {
    name.push_str(" Smith");
}
```

Output

```text
Alice Smith
```

Notice:

Variable:

```rust
let mut name
```

Borrow:

```rust
&mut name
```

Parameter:

```rust
&mut String
```

All three are necessary.

---

# 7. Borrowing Rules

Rust has only a few rules, but they're extremely important.

## Rule 1

You can have:

Many immutable references

OR

One mutable reference

Never both simultaneously.

---

## Allowed

```rust
let x = String::from("Hello");

let a = &x;
let b = &x;
let c = &x;
```

Three immutable borrows.

Perfectly fine.

---

## Also allowed

```rust
let mut x = String::from("Hello");

let a = &mut x;
```

One mutable borrow.

Also fine.

---

## Not allowed

```rust
let mut x = String::from("Hello");

let a = &x;

let b = &mut x;
```

Compiler error.

Why?

Someone is reading while someone else is writing.

Rust prevents data races before your program ever runs.

---

# 8. Another Illegal Example

```rust
let mut text = String::from("Rust");

let a = &mut text;

let b = &mut text;
```

Two mutable borrows.

Not allowed.

Who gets to modify it first?

Rust doesn't allow that ambiguity.

---

# 9. Borrow Ends After Last Use

This surprises beginners.

```rust
fn main() {
    let mut text = String::from("Rust");

    let r1 = &text;

    println!("{}", r1);

    let r2 = &mut text;

    r2.push_str(" Language");

    println!("{}", r2);
}
```

This works.

Why?

Because

```rust
r1
```

is no longer used after the first `println!`.

The immutable borrow ends there.

---

# 10. Borrowing vs Cloning

Borrowing

```rust
let name = String::from("Alice");

let other = &name;
```

No copy.

Very fast.

---

Cloning

```rust
let name = String::from("Alice");

let other = name.clone();
```

Creates another String.

Uses more memory.

Borrow whenever possible.

Clone only when necessary.

---

# 11. Borrowing in Functions

Instead of

```rust
fn calculate(text: String)
```

prefer

```rust
fn calculate(text: &String)
```

Example

```rust
fn length(text: &String) -> usize {
    text.len()
}

fn main() {
    let word = String::from("Backend");

    let size = length(&word);

    println!("{}", size);

    println!("{}", word);
}
```

Output

```text
7
Backend
```

Ownership stays in `main`.

---

# 12. Returning References

This is illegal:

```rust
fn create() -> &String {
    let text = String::from("Hello");

    &text
}
```

Why?

`text` disappears when the function ends.

Returning a reference to it would create a **dangling reference**.

Rust refuses to compile it.

---

Correct

```rust
fn create() -> String {
    String::from("Hello")
}
```

Ownership moves safely to the caller.

---

# 13. Borrowing Structs

```rust
struct User {
    username: String,
}

fn print_user(user: &User) {
    println!("{}", user.username);
}

fn main() {
    let user = User {
        username: String::from("alice"),
    };

    print_user(&user);

    println!("{}", user.username);
}
```

Ownership never changes.

---

# 14. Mutable Struct Borrowing

```rust
struct User {
    username: String,
}

fn rename(user: &mut User) {
    user.username = String::from("bob");
}

fn main() {
    let mut user = User {
        username: String::from("alice"),
    };

    rename(&mut user);

    println!("{}", user.username);
}
```

Output

```text
bob
```

---

# 15. Borrowing Slices

Instead of borrowing an entire `String`, you can borrow part of it.

```rust
fn first_word(text: &str) -> &str {
    &text[..5]
}

fn main() {
    let word = String::from("Hello World");

    println!("{}", first_word(&word));
}
```

Output

```text
Hello
```

`&str` is a string slice—a borrowed view into a string. You'll use `&str` frequently in Rust APIs.

---

# 16. Common Compiler Errors

### Error 1: Borrow after move

```rust
let name = String::from("Alice");

let another = name;

println!("{}", name);
```

Fix: borrow instead of moving, or clone if you need two owned copies.

---

### Error 2: Mutable and immutable borrow together

```rust
let mut text = String::from("Rust");

let a = &text;

let b = &mut text;
```

Fix: make sure the immutable borrow is no longer used before creating the mutable borrow.

---

### Error 3: Two mutable borrows

```rust
let mut value = String::new();

let a = &mut value;

let b = &mut value;
```

Fix: use one mutable borrow at a time.

---

# 17. Real Backend Example

Suppose you have a function that logs a request.

Without borrowing:

```rust
fn log_request(path: String) {
    println!("{}", path);
}
```

This takes ownership of the `String`, meaning the caller can't use it afterward.

With borrowing:

```rust
fn log_request(path: &str) {
    println!("Request: {}", path);
}

fn main() {
    let path = String::from("/users");

    log_request(&path);

    println!("Handling {}", path);
}
```

Output:

```text
Request: /users
Handling /users
```

This pattern is common in web frameworks like Axum because it avoids unnecessary copying while keeping ownership clear.

---

# 18. Mini Project: Library Book

```rust
struct Book {
    title: String,
}

fn display(book: &Book) {
    println!("Title: {}", book.title);
}

fn rename(book: &mut Book) {
    book.title.push_str(" (2nd Edition)");
}

fn main() {
    let mut book = Book {
        title: String::from("Rust Programming"),
    };

    display(&book);

    rename(&mut book);

    display(&book);
}
```

Output

```text
Title: Rust Programming
Title: Rust Programming (2nd Edition)
```

---

# Best Practices

- Borrow (`&T`) when a function only needs to read data.
- Borrow mutably (`&mut T`) only when a function needs to modify data.
- Prefer borrowing over cloning to avoid unnecessary allocations.
- Use `&str` for read-only string parameters instead of `&String` when possible, because it accepts both string literals and `String` values.
- Let the compiler guide you—its borrow checker errors often point directly to the ownership issue.

---

# Practice Exercises

1. Write a function that takes a borrowed `String` and returns its length.
2. Write a function that borrows a `Vec<i32>` and returns the largest number.
3. Create a `User` struct and write a function that prints it using an immutable borrow.
4. Write a function that takes a mutable borrow of a `User` and updates the email address.
5. Create a function that accepts `&str` and prints a welcome message. Call it with both a string literal and a `String`.

Master borrowing before moving on to lifetimes. Once borrowing feels natural, lifetimes become much easier to understand because they simply describe **how long borrowed references are guaranteed to remain valid**.
