One should understand `Option` as one simple idea:

**`Option<T>` represents a value that may or may not exist.**

This is extremely important in Rust because Rust does **not** use `null` in the same way languages like JavaScript, Java, or C# do.

# 1. The two possible values

`Option<T>` has two variants:

```rust
Some(value)
None
```

For example:

```rust
let age: Option<u32> = Some(25);
```

means:

> An age exists, and it is 25.

While:

```rust
let age: Option<u32> = None;
```

means:

> There is no age.

Think:

```text
Option<u32>
     │
     ├── Some(25)  → value exists
     │
     └── None      → no value
```

---

# 2. Why does Rust use `Option`?

Imagine you're looking for a user:

```rust
let user = find_user(10);
```

What if user `10` doesn't exist?

Instead of returning some special value like:

```text
null
```

Rust returns:

```rust
Some(user)
```

if the user exists, or:

```rust
None
```

if they don't.

This forces you to handle the possibility that the value doesn't exist.

---

# 3. Creating an `Option`

```rust
let name = Some("John");

let age: Option<u32> = Some(25);

let email: Option<String> = None;
```

Rust can usually infer the type of `Some`.

For `None`, you may need to specify the type:

```rust
let email: Option<String> = None;
```

because `None` by itself doesn't tell Rust what type the missing value would have been.

---

# 4. Checking with `match`

This is the most important thing to understand about `Option`.

```rust
let age = Some(25);

match age {
    Some(value) => println!("Age is {}", value),
    None => println!("Age is not available"),
}
```

Output:

```text
Age is 25
```

If:

```rust
let age: Option<u32> = None;
```

then:

```text
Age is not available
```

The pattern is:

```rust
match value {
    Some(value) => {
        // value exists
    }

    None => {
        // value doesn't exist
    }
}
```

You should be comfortable reading and writing this.

---

# 5. `Option` with `if let`

Often you don't care about the `None` case.

You can use:

```rust
let age = Some(25);

if let Some(value) = age {
    println!("Age is {}", value);
}
```

This means:

> If there is a value, give it to me.

For example:

```rust
let username: Option<&str> = Some("John");

if let Some(name) = username {
    println!("Hello, {}", name);
}
```

Output:

```text
Hello, John
```

---

# 6. `None`

Here's the other case:

```rust
let username: Option<&str> = None;

if let Some(name) = username {
    println!("Hello, {}", name);
}
```

Nothing is printed because there is no value.

If you want to handle both cases, use `match`:

```rust
match username {
    Some(name) => println!("Hello, {}", name),
    None => println!("No username found"),
}
```

---

# 7. `Option` and `Vec::get()`

This is where `Option` becomes practical.

Remember:

```rust
let numbers = vec![10, 20, 30];

let number = numbers.get(1);
```

`get()` returns an `Option`.

So:

```rust
numbers.get(1)
```

returns:

```text
Some(20)
```

But:

```rust
numbers.get(10)
```

returns:

```text
None
```

You can handle it:

```rust
let numbers = vec![10, 20, 30];

match numbers.get(10) {
    Some(number) => println!("Found: {}", number),
    None => println!("Number not found"),
}
```

This is one of the most important real examples of `Option`.

---

# 8. `Option` and `HashMap`

Same idea with `HashMap`.

```rust
use std::collections::HashMap;

fn main() {
    let mut users = HashMap::new();

    users.insert(1, "John");
    users.insert(2, "Mary");

    match users.get(&1) {
        Some(name) => println!("User: {}", name),
        None => println!("User not found"),
    }
}
```

`HashMap::get()` returns:

```text
Some(value)
```

if the key exists.

Or:

```text
None
```

if it doesn't.

---

# 9. Getting the value with `unwrap()`

You will see this frequently while learning Rust:

```rust
let age = Some(25);

let value = age.unwrap();

println!("{}", value);
```

Output:

```text
25
```

`unwrap()` extracts the value from `Some`.

But there's a problem.

If:

```rust
let age: Option<u32> = None;

let value = age.unwrap();
```

your program **panics**.

So don't use `unwrap()` blindly.

It's useful for simple examples and situations where you've already guaranteed the value exists, but production code generally needs deliberate handling of the `None` case.

---

# 10. `unwrap_or()`

A safer and very useful alternative is:

```rust
let age: Option<u32> = None;

let value = age.unwrap_or(0);

println!("{}", value);
```

Output:

```text
0
```

Meaning:

> Give me the value if it exists; otherwise use `0`.

Example:

```rust
let age = Some(25);

let value = age.unwrap_or(0);

println!("{}", value);
```

Output:

```text
25
```

So:

```text
Some(25) → 25
None     → 0
```

---

# 11. `is_some()` and `is_none()`

Sometimes you only want to check whether a value exists.

```rust
let age = Some(25);

println!("{}", age.is_some());
println!("{}", age.is_none());
```

Output:

```text
true
false
```

For `None`:

```rust
let age: Option<u32> = None;

println!("{}", age.is_some());
println!("{}", age.is_none());
```

Output:

```text
false
true
```

---

# 12. `Option` in a function

This is especially important for backend development.

You can make a function return `Option`.

```rust
fn find_user(id: u32) -> Option<&'static str> {
    if id == 1 {
        Some("John")
    } else {
        None
    }
}
```

Then:

```rust
fn main() {
    match find_user(1) {
        Some(name) => println!("Found: {}", name),
        None => println!("User not found"),
    }
}
```

Output:

```text
Found: John
```

But:

```rust
find_user(10)
```

would return:

```text
None
```

This pattern is extremely common.

---

# 13. A realistic backend example

Imagine a function that searches for a user:

```rust
struct User {
    id: u32,
    name: String,
}

fn find_user(id: u32) -> Option<User> {
    if id == 1 {
        Some(User {
            id: 1,
            name: String::from("John"),
        })
    } else {
        None
    }
}
```

Then:

```rust
fn main() {
    match find_user(1) {
        Some(user) => println!("User: {}", user.name),
        None => println!("User not found"),
    }
}
```

This is conceptually similar to what you'll eventually do when querying a database:

```text
Find user with ID 25
       ↓
Does user exist?
       ↓
 ┌─────┴─────┐
Yes          No
 ↓            ↓
Some(User)   None
```

---

# 14. `Option` vs `Result`

This distinction is **very important** for Week 3.

### `Option`

Use when something **may or may not exist**.

```rust
Option<T>
```

Examples:

```text
Find user → Some(user) / None
Find item → Some(item) / None
Get vector element → Some(value) / None
```

### `Result`

Use when an operation can **succeed or fail with an error**.

```rust
Result<T, E>
```

Example:

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}
```

Think:

```text
Option
   ↓
"Is there a value?"
   ↓
Some / None


Result
   ↓
"Did the operation succeed?"
   ↓
Ok / Err
```

That's the key distinction.

---

# 15. The methods you should know

For your current level, focus on these:

| Method             | Purpose                             |
| ------------------ | ----------------------------------- |
| `Some(value)`      | Represents an existing value        |
| `None`             | Represents no value                 |
| `match`            | Handle `Some` and `None`            |
| `if let Some(x)`   | Handle only the `Some` case         |
| `unwrap()`         | Extract value, but panics on `None` |
| `unwrap_or(value)` | Extract value or use a default      |
| `is_some()`        | Check if value exists               |
| `is_none()`        | Check if value doesn't exist        |

Don't worry about all the advanced `Option` methods yet.

---

# Your Week 3 target

You should be able to understand this completely:

```rust
fn find_student(id: u32) -> Option<String> {
    if id == 1 {
        Some(String::from("John"))
    } else {
        None
    }
}

fn main() {
    match find_student(1) {
        Some(name) => println!("Student: {}", name),
        None => println!("Student not found"),
    }
}
```

You should be able to explain:

```text
Option<String>
```

means:

> This function may return a `String`, or it may return nothing.

And:

```rust
Some(String::from("John"))
```

means:

> A value exists.

While:

```rust
None
```

means:

> No value exists.

### The mental model to remember

```text
Option<T>
   │
   ├── Some(T) → value exists
   │
   └── None    → value doesn't exist
```

For starters, **that's the core of `Option`**. Once you understand this, `Vec::get()`, `HashMap::get()`, and functions that search for something will make much more sense.
