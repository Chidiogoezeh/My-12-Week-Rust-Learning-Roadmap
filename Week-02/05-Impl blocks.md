# 1. What is an `impl` block?

`impl` means **implementation**.

It is where you define behavior for a type, such as a `struct`.

```rust
struct User {
    name: String,
    age: u32,
}

impl User {
    fn introduce(&self) {
        println!("Hi, I'm {}", self.name);
    }
}
```

Then you can call the method:

```rust
fn main() {
    let user = User {
        name: String::from("Daud"),
        age: 25,
    };

    user.introduce();
}
```

Think:

```text
struct → defines what data something has

impl   → defines what that thing can do
```

---

# 2. Methods with `&self`

The most common form you'll use is:

```rust
impl User {
    fn introduce(&self) {
        println!("I'm {}", self.name);
    }
}
```

`&self` means:

> Borrow this particular `User` without taking ownership.

Example:

```rust
let user = User {
    name: String::from("Daud"),
    age: 25,
};

user.introduce();
println!("{}", user.name);
```

The `user` is still available after calling the method.

---

# 3. Methods with `&mut self`

Use `&mut self` when the method needs to modify the struct.

```rust
struct User {
    name: String,
}

impl User {
    fn change_name(&mut self, new_name: String) {
        self.name = new_name;
    }
}
```

Usage:

```rust
fn main() {
    let mut user = User {
        name: String::from("Daud"),
    };

    user.change_name(String::from("John"));

    println!("{}", user.name);
}
```

Output:

```text
John
```

Remember:

```rust
&self
```

→ read/borrow

```rust
&mut self
```

→ modify

---

# 4. Methods with `self`

You can also use:

```rust
fn consume(self)
```

This **takes ownership** of the struct.

```rust
struct User {
    name: String,
}

impl User {
    fn consume(self) {
        println!("Goodbye {}", self.name);
    }
}
```

Usage:

```rust
let user = User {
    name: String::from("Daud"),
};

user.consume();

// user can no longer be used here
```

You should understand this because it connects directly to **Rust ownership**.

---

# 5. Associated Functions

An `impl` block can contain functions that don't take `self`.

The most common example is `new()`.

```rust
struct User {
    name: String,
    age: u32,
}

impl User {
    fn new(name: String, age: u32) -> User {
        User {
            name,
            age,
        }
    }
}
```

Call it with:

```rust
let user = User::new(String::from("Daud"), 25);
```

Notice the difference:

```rust
user.introduce();   // method
User::new(...);     // associated function
```

A simple rule:

> If it needs an existing object → method.

> If it creates/operates on the type itself → associated function.

---

# 6. A Practical Backend Example

Imagine you're building a user system.

```rust
struct User {
    id: u32,
    username: String,
    email: String,
}

impl User {
    fn new(id: u32, username: String, email: String) -> User {
        User {
            id,
            username,
            email,
        }
    }

    fn display(&self) {
        println!("{} - {}", self.username, self.email);
    }

    fn change_email(&mut self, email: String) {
        self.email = email;
    }
}
```

Usage:

```rust
fn main() {
    let mut user = User::new(
        1,
        String::from("daud"),
        String::from("daud@example.com"),
    );

    user.display();

    user.change_email(String::from("new@example.com"));

    user.display();
}
```

This pattern will feel very familiar when you start building backend services.

---

# 7. Multiple `impl` Blocks

You can have multiple `impl` blocks for the same struct.

```rust
struct User {
    name: String,
}

impl User {
    fn new(name: String) -> User {
        User { name }
    }
}

impl User {
    fn greet(&self) {
        println!("Hello {}", self.name);
    }
}
```

Rust treats them as part of the same implementation.

You generally don't need this until your code becomes larger.

---

# 8. `impl` Becomes Important in Backend Development

You'll eventually see things like:

```rust
impl UserService {
    async fn create_user(...) {
        // ...
    }

    async fn get_user(...) {
        // ...
    }
}
```

Or:

```rust
impl UserRepository {
    async fn find_by_id(...) {
        // database query
    }
}
```

So you'll commonly have:

```text
User
  ↓
User methods

UserService
  ↓
Business logic

UserRepository
  ↓
Database operations
```

This is why you should understand `impl` well before moving into your Axum/API weeks.

---

# 9. `impl` + Traits — Just Know the Idea for Now

Later you'll learn traits.

You'll see:

```rust
trait Printable {
    fn print(&self);
}

impl Printable for User {
    fn print(&self) {
        println!("{}", self.name);
    }
}
```

This is different from:

```rust
impl User {
    fn print(&self) {}
}
```

The first means:

> "User implements the `Printable` trait."

No need to go deep into traits yet. Just recognize the syntax.

---

# What you should master now

Based on the roadmap, make sure you can explain and use:

| Syntax                | Meaning                            |
| --------------------- | ---------------------------------- |
| `impl User`           | Add behavior to `User`             |
| `&self`               | Borrow/read the instance           |
| `&mut self`           | Mutably borrow/modify the instance |
| `self`                | Take ownership of the instance     |
| `User::new()`         | Associated function                |
| `user.method()`       | Instance method                    |
| `impl Trait for User` | Implement a trait                  |

### The most important mental model

```rust
struct User {
    name: String,
}
```

defines **what a `User` is**.

```rust
impl User {
    fn greet(&self) {
        println!("Hello {}", self.name);
    }
}
```

defines **what a `User` can do**.

If you understand that distinction, plus `self`, `&self`, `&mut self`, and associated functions, you've learned the important `impl` material for this stage of your roadmap.
