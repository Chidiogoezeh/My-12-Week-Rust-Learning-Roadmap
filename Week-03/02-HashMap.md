What is `HashMap` in Rust

A `HashMap` stores **key-value pairs**.

Think of it like:

```text
key       value
----------------
"John"    25
"Mary"    30
"David"   22
```

In Rust:

```rust
use std::collections::HashMap;
```

---

## 1. Creating a `HashMap`

```rust
use std::collections::HashMap;

fn main() {
    let mut ages = HashMap::new();

    ages.insert("John", 25);
    ages.insert("Mary", 30);
    ages.insert("David", 22);

    println!("{:?}", ages);
}
```

The important method here is:

```rust
insert()
```

It adds a key-value pair.

---

# 2. Understanding key and value

In:

```rust
ages.insert("John", 25);
```

```text
"John" → key
25     → value
```

A `HashMap` requires:

- Keys to have one type
- Values to have one type

For example:

```rust
HashMap<String, u32>
```

means:

```text
String = key
u32    = value
```

---

# 3. Getting a value with `get()`

```rust
use std::collections::HashMap;

fn main() {
    let mut ages = HashMap::new();

    ages.insert("John", 25);
    ages.insert("Mary", 30);

    let age = ages.get("John");

    println!("{:?}", age);
}
```

Output:

```text
Some(25)
```

Notice that `get()` returns an `Option`.

If the key doesn't exist:

```rust
let age = ages.get("David");

println!("{:?}", age);
```

Output:

```text
None
```

This is important because it connects `HashMap` directly to what you're learning about **`Option`**.

---

# 4. Using `match` with `get()`

This is a very useful pattern.

```rust
use std::collections::HashMap;

fn main() {
    let mut ages = HashMap::new();

    ages.insert("John", 25);
    ages.insert("Mary", 30);

    match ages.get("John") {
        Some(age) => println!("John is {} years old", age),
        None => println!("John was not found"),
    }
}
```

Output:

```text
John is 25 years old
```

---

# 5. Updating a value

If the key already exists, `insert()` replaces the old value.

```rust
let mut ages = HashMap::new();

ages.insert("John", 25);
ages.insert("John", 26);

println!("{:?}", ages.get("John"));
```

Output:

```text
Some(26)
```

So:

```rust
insert()
```

means:

> Add the key if it doesn't exist; replace its value if it does.

---

# 6. Checking whether a key exists

Use:

```rust
contains_key()
```

Example:

```rust
let mut ages = HashMap::new();

ages.insert("John", 25);

if ages.contains_key("John") {
    println!("John exists");
}
```

Output:

```text
John exists
```

This is useful when you only care whether something exists.

---

# 7. Removing a value

Use:

```rust
remove()
```

Example:

```rust
let mut ages = HashMap::new();

ages.insert("John", 25);
ages.insert("Mary", 30);

ages.remove("John");

println!("{:?}", ages);
```

Now John is gone.

---

# 8. Getting the number of entries

Use:

```rust
len()
```

Example:

```rust
let mut ages = HashMap::new();

ages.insert("John", 25);
ages.insert("Mary", 30);

println!("{}", ages.len());
```

Output:

```text
2
```

---

# 9. Checking if it's empty

```rust
let ages: HashMap<&str, i32> = HashMap::new();

println!("{}", ages.is_empty());
```

Output:

```text
true
```

---

# 10. Looping through a `HashMap`

You can loop through key-value pairs:

```rust
use std::collections::HashMap;

fn main() {
    let mut ages = HashMap::new();

    ages.insert("John", 25);
    ages.insert("Mary", 30);
    ages.insert("David", 22);

    for (name, age) in &ages {
        println!("{} is {}", name, age);
    }
}
```

You might get:

```text
Mary is 30
John is 25
David is 22
```

**Don't rely on the order.**

A `HashMap` does not guarantee that entries will come out in the order you inserted them.

---

# 11. `HashMap<String, String>`

This is closer to what you'll encounter in backend development.

```rust
use std::collections::HashMap;

fn main() {
    let mut user = HashMap::new();

    user.insert(
        String::from("name"),
        String::from("John"),
    );

    user.insert(
        String::from("email"),
        String::from("john@example.com"),
    );

    println!("{:?}", user.get("name"));
}
```

Output:

```text
Some("John")
```

---

# 12. HashMap with a struct

You can also store structs as values.

```rust
use std::collections::HashMap;

struct User {
    name: String,
    age: u32,
}

fn main() {
    let mut users = HashMap::new();

    users.insert(
        1,
        User {
            name: String::from("John"),
            age: 25,
        },
    );

    users.insert(
        2,
        User {
            name: String::from("Mary"),
            age: 30,
        },
    );

    if let Some(user) = users.get(&1) {
        println!("{} is {}", user.name, user.age);
    }
}
```

Here:

```text
HashMap<u32, User>
```

means:

```text
User ID → User
```

This is a very useful mental model for backend development.

---

# 13. Why `&1` with `get()`?

Notice:

```rust
users.get(&1)
```

rather than:

```rust
users.get(1)
```

`get()` receives a **reference to the key**.

So if your key is:

```rust
u32
```

you normally provide:

```rust
&u32
```

Example:

```rust
users.get(&1)
```

This is another small example of Rust's **borrowing/reference** system.

---

# 14. The most useful pattern: `entry()`

You don't need to master `entry()` yet, but you should understand this common pattern.

Suppose you're counting words:

```rust
use std::collections::HashMap;

fn main() {
    let mut counts = HashMap::new();

    let words = vec!["apple", "apple", "banana", "apple"];

    for word in words {
        let count = counts.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", counts);
}
```

Result will contain approximately:

```text
{
    "apple": 3,
    "banana": 1
}
```

The important part is:

```rust
counts.entry(word).or_insert(0)
```

It means:

> If this key doesn't exist, insert `0`. Otherwise, use the existing value.

Then:

```rust
*count += 1;
```

increments the value.

You don't need to go deeply into the mechanics of `entry()` yet.

---

# 15. `HashMap` vs `Vec`

This distinction is important.

### `Vec`

Use when you have a **list of items**:

```rust
let users = vec![
    "John",
    "Mary",
    "David",
];
```

Think:

```text
[John, Mary, David]
```

### `HashMap`

Use when you need to associate one thing with another:

```rust
let ages = HashMap::from([
    ("John", 25),
    ("Mary", 30),
]);
```

Think:

```text
John → 25
Mary → 30
```

---

# What you should know for Week 3

Focus on these:

| Concept          | What you should know                         |
| ---------------- | -------------------------------------------- |
| `HashMap::new()` | Create an empty map                          |
| `insert()`       | Add/update key-value pair                    |
| `get()`          | Retrieve a value                             |
| `contains_key()` | Check if key exists                          |
| `remove()`       | Remove a key-value pair                      |
| `len()`          | Number of entries                            |
| `is_empty()`     | Check if empty                               |
| `for` loop       | Iterate over entries                         |
| `entry()`        | Insert a default/update existing value       |
| `Option`         | Understand why `get()` returns `Some`/`None` |

## Target exercise

You should be able to build this without copying:

```rust
use std::collections::HashMap;

fn main() {
    let mut students = HashMap::new();

    students.insert(1, "John");
    students.insert(2, "Mary");
    students.insert(3, "David");

    if let Some(name) = students.get(&2) {
        println!("Student: {}", name);
    }

    students.remove(&3);

    for (id, name) in &students {
        println!("{}: {}", id, name);
    }

    println!("Total students: {}", students.len());
}
```

If you understand **key/value pairs, `insert()`, `get()`, `Some`/`None`, `remove()`, `contains_key()`, iteration, and the basic idea behind `entry()`**, you've learned enough `HashMap` for begginer level.

No need to study hashing algorithms, custom hashers, `BTreeMap`, `HashSet`, or `Entry` internals yet. Those can come later when they are actually needed.
