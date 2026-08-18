**Tuple structs** are a small Rust feature, but they're worth understanding because they teach you how Rust can create **strongly typed wrappers around existing values**. That becomes particularly useful in backend development for IDs, coordinates, money values, domain types, etc.

# 1. What is a Tuple Struct?

A tuple struct is a struct that has **unnamed fields**.

A normal struct has named fields:

```rust
struct User {
    id: u32,
    name: String,
}
```

A tuple struct doesn't name its fields:

```rust
struct UserId(u32);
```

You can think of it as:

```text
UserId
  │
  └── u32
```

The syntax is:

```rust
struct StructName(Type1, Type2, Type3);
```

For example:

```rust
struct Coordinates(f64, f64);
```

This creates a type containing two `f64` values.

---

# 2. Creating a Tuple Struct

Given:

```rust
struct UserId(u32);
```

You create an instance like this:

```rust
let user_id = UserId(100);
```

For:

```rust
struct Coordinates(f64, f64);
```

you can do:

```rust
let location = Coordinates(4.8156, 7.0498);
```

---

# 3. Accessing Tuple Struct Fields

Because the fields don't have names, you access them using their **position**.

```rust
struct Coordinates(f64, f64);

fn main() {
    let location = Coordinates(4.8156, 7.0498);

    println!("Latitude: {}", location.0);
    println!("Longitude: {}", location.1);
}
```

The indexes start at `0`:

```text
location.0 → first field
location.1 → second field
```

This is similar to a tuple:

```rust
let coordinates = (4.8156, 7.0498);

println!("{}", coordinates.0);
println!("{}", coordinates.1);
```

But there's a **very important difference**.

---

# 4. Tuple vs Tuple Struct

Consider a normal tuple:

```rust
let user_id = 100;
let product_id = 100;
```

You could simply have:

```rust
let user_id: u32 = 100;
let product_id: u32 = 100;
```

Both are just `u32`.

But tuple structs allow you to create **different types**.

```rust
struct UserId(u32);
struct ProductId(u32);
```

Now:

```rust
let user_id = UserId(100);
let product_id = ProductId(100);
```

Even though both internally contain a `u32`, Rust considers them different types.

This is one of the biggest reasons tuple structs are useful.

---

# 5. Preventing Mistakes with IDs

Imagine you have:

```rust
struct UserId(u32);
struct ProductId(u32);
```

Then:

```rust
fn find_user(id: UserId) {
    println!("Finding user {}", id.0);
}
```

You can do:

```rust
let user_id = UserId(10);

find_user(user_id);
```

But this:

```rust
let product_id = ProductId(10);

find_user(product_id);
```

will produce a compiler error.

Why?

Because:

```text
UserId ≠ ProductId
```

even though:

```text
UserId contains u32
ProductId contains u32
```

This is **type safety**.

And type safety is one of Rust's biggest strengths.

---

# 6. Why This Matters for Backend Development

Imagine you're building an API with:

```text
Users
Products
Orders
Events
Tickets
```

You might have:

```rust
struct UserId(u64);
struct ProductId(u64);
struct OrderId(u64);
struct EventId(u64);
struct TicketId(u64);
```

Now your functions can be very explicit:

```rust
fn get_user(id: UserId) {
    // ...
}

fn get_product(id: ProductId) {
    // ...
}

fn get_order(id: OrderId) {
    // ...
}
```

This prevents accidentally passing an `OrderId` where a `UserId` is expected.

That's a useful pattern in production Rust applications.

---

# 7. Tuple Struct with Multiple Fields

Tuple structs can contain multiple fields.

```rust
struct User(String, u32);
```

Here:

```text
User
├── field 0 → String
└── field 1 → u32
```

Create one:

```rust
let user = User(
    String::from("Daud"),
    25,
);
```

Access:

```rust
println!("Name: {}", user.0);
println!("Age: {}", user.1);
```

---

# 8. Different Data Types

Each field can have a different type.

```rust
struct Product(String, f64, bool);
```

Create one:

```rust
let product = Product(
    String::from("Laptop"),
    1200.50,
    true,
);
```

Access:

```rust
println!("Name: {}", product.0);
println!("Price: {}", product.1);
println!("Available: {}", product.2);
```

So tuple structs don't have to contain just one value.

---

# 9. Tuple Structs vs Normal Structs

Compare these:

### Normal struct

```rust
struct User {
    id: u32,
    name: String,
    age: u8,
}
```

Access:

```rust
user.id
user.name
user.age
```

### Tuple struct

```rust
struct User(u32, String, u8);
```

Access:

```rust
user.0
user.1
user.2
```

### Which is better?

For something like a user, the normal struct is generally better:

```rust
struct User {
    id: u32,
    name: String,
    age: u8,
}
```

because:

```rust
user.name
```

is much clearer than:

```rust
user.1
```

Use tuple structs when the fields are naturally understood by their position or when you're creating a **new semantic type around an existing value**.

---

# 10. Tuple Structs and Functions

You can pass tuple structs into functions.

```rust
struct UserId(u32);

fn print_user_id(id: UserId) {
    println!("User ID: {}", id.0);
}

fn main() {
    let id = UserId(42);

    print_user_id(id);
}
```

Output:

```text
User ID: 42
```

---

# 11. Borrowing Tuple Structs

Just like normal structs, tuple structs follow Rust's ownership rules.

```rust
struct UserId(u32);

fn print_user_id(id: &UserId) {
    println!("User ID: {}", id.0);
}

fn main() {
    let id = UserId(42);

    print_user_id(&id);

    // Still usable because we borrowed it.
    println!("Again: {}", id.0);
}
```

Notice:

```rust
&UserId
```

means we're borrowing the tuple struct.

---

# 12. Mutable Tuple Structs

You can modify fields if the instance is mutable.

```rust
struct UserId(u32);

fn main() {
    let mut id = UserId(10);

    println!("{}", id.0);

    id.0 = 20;

    println!("{}", id.0);
}
```

Output:

```text
10
20
```

The same rule applies as with normal structs:

```rust
let mut id
```

is required.

---

# 13. Tuple Structs and `impl`

You can implement methods on tuple structs.

```rust
struct UserId(u32);

impl UserId {
    fn new(id: u32) -> UserId {
        UserId(id)
    }

    fn value(&self) -> u32 {
        self.0
    }
}
```

Then:

```rust
fn main() {
    let id = UserId::new(100);

    println!("User ID: {}", id.value());
}
```

Output:

```text
User ID: 100
```

This is a pattern you'll see in more advanced Rust.

---

# 14. A More Realistic Backend Example

Suppose you're building an API.

Instead of:

```rust
fn get_user(id: u64) {
    // ...
}
```

you can create:

```rust
struct UserId(u64);
```

Then:

```rust
fn get_user(id: UserId) {
    println!("Getting user {}", id.0);
}
```

Now:

```rust
let user_id = UserId(123);

get_user(user_id);
```

Your function communicates something much more meaningful:

```rust
fn get_user(id: UserId)
```

rather than:

```rust
fn get_user(id: u64)
```

The first tells you:

> "This function expects a User ID."

The second only tells you:

> "This function expects some unsigned integer."

---

# 15. Domain Types

This idea is called a **domain type** or **newtype pattern**.

For example:

```rust
struct Email(String);
struct Username(String);
struct Password(String);
```

All three contain a `String`, but they're different types.

```rust
let email = Email(String::from("daud@example.com"));
let username = Username(String::from("daud"));
let password = Password(String::from("secret"));
```

You could then create functions such as:

```rust
fn send_email(email: Email) {
    // ...
}
```

Now you can't accidentally do:

```rust
send_email(username);
```

because:

```text
Username ≠ Email
```

This gives your backend code stronger type safety.

---

# 16. Newtype Pattern

The **newtype pattern** is essentially wrapping one type inside a tuple struct.

For example:

```rust
struct UserId(u64);
```

is a new type around:

```text
u64
```

This gives you a separate type identity.

Other examples:

```rust
struct OrderId(u64);
struct ProductId(u64);
struct EventId(u64);
struct Money(f64);
struct Email(String);
```

This is a very useful Rust design pattern.

---

# 17. Deriving `Debug`

Just like normal structs, you can derive traits.

```rust
#[derive(Debug)]
struct UserId(u64);
```

Then:

```rust
fn main() {
    let id = UserId(100);

    println!("{:?}", id);
}
```

Output:

```text
UserId(100)
```

For pretty printing:

```rust
println!("{:#?}", id);
```

---

# 18. Tuple Structs and `Copy`

For small types like IDs, you may eventually see:

```rust
#[derive(Debug, Clone, Copy)]
struct UserId(u64);
```

Now:

```rust
let id1 = UserId(100);
let id2 = id1;

println!("{:?}", id1);
println!("{:?}", id2);
```

works because `UserId` implements `Copy`.

Why?

Because `u64` itself is a simple `Copy` type.

Don't worry about mastering `Copy` yet. Just understand that you'll encounter it when creating small wrapper types.

---

# 19. Tuple Structs in Your REST API

When you eventually build your Axum API, you might have:

```rust
struct UserId(i64);
```

and your database may return:

```text
user_id = 42
```

You could turn it into:

```rust
let user_id = UserId(42);
```

Then your application can distinguish:

```rust
UserId
OrderId
EventId
TicketId
```

even if they're all backed by `i64`.

This becomes especially useful in larger applications where many IDs have the same underlying type.

---

# 20. A Complete Example

Here's a small example that combines the important concepts:

```rust
#[derive(Debug, Clone, Copy)]
struct UserId(u64);

impl UserId {
    fn new(id: u64) -> Self {
        Self(id)
    }

    fn value(&self) -> u64 {
        self.0
    }
}

fn find_user(id: UserId) {
    println!("Looking for user with ID: {}", id.value());
}

fn main() {
    let user_id = UserId::new(42);

    println!("User ID: {:?}", user_id);

    find_user(user_id);

    // Because UserId implements Copy,
    // we can still use it here.
    println!("ID again: {}", user_id.value());
}
```

The important pieces are:

```rust
struct UserId(u64);
```

Creates the type.

```rust
impl UserId
```

Adds functionality.

```rust
fn new(id: u64) -> Self
```

Creates a new `UserId`.

```rust
self.0
```

Accesses the underlying `u64`.

```rust
#[derive(Debug, Clone, Copy)]
```

adds useful capabilities.

---

# 21. What You Should Understand Before Moving On

For your 12-week roadmap, you **don't need to become an expert in tuple structs**.

You should be able to explain and use:

### Essential

- What a tuple struct is.
- Difference between a tuple and tuple struct.
- How to define one.
- How to instantiate one.
- How to access fields using `.0`, `.1`, etc.
- How to modify fields.
- How ownership and borrowing apply to them.

### Important for backend development

- Why `struct UserId(u64)` is different from `u64`.
- The **newtype pattern**.
- Why strong types prevent bugs.
- How to create methods with `impl`.
- How to derive `Debug`, `Clone`, and `Copy`.
- Why domain-specific types can make backend code safer.

---

# 22. The Most Important Example to Remember

If you remember only one backend-oriented example, make it this:

```rust
struct UserId(u64);
struct OrderId(u64);
struct ProductId(u64);

fn get_user(id: UserId) {
    println!("Getting user {}", id.0);
}

fn get_order(id: OrderId) {
    println!("Getting order {}", id.0);
}

fn main() {
    let user_id = UserId(10);
    let order_id = OrderId(20);
    let product_id = ProductId(30);

    get_user(user_id);
    get_order(order_id);

    // This would NOT compile:
    // get_user(product_id);
}
```

Even though all three contain a `u64`, Rust treats them as different types.

**That's the real value of tuple structs: they let you give ordinary values meaningful, distinct types.**

For your backend journey, you'll eventually see this same idea applied to **user IDs, database IDs, email addresses, money, UUIDs, request types, and domain objects**.
