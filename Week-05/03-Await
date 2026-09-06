For now, we only need to understand `.await` well enough to use asynchronous functions confidently in Tokio and, later, Axum and SQLx.

# 1. What is `.await`?

`.await` tells Rust:

> **"Wait for this Future to finish, then give me its result."**

Example:

```rust
async fn get_name() -> String {
    String::from("Daud")
}

#[tokio::main]
async fn main() {
    let name = get_name().await;

    println!("{}", name);
}
```

Here:

```rust
get_name()
```

produces a **Future**.

Then:

```rust
get_name().await
```

waits for that Future and produces:

```text
String
```

So:

```text
get_name()          → Future
get_name().await    → String
```

---

# 2. `.await` can only be used in async code

This is valid:

```rust
#[tokio::main]
async fn main() {
    let result = get_data().await;
}
```

This is **not** valid:

```rust
fn main() {
    let result = get_data().await;
}
```

Why?

Because `.await` requires an **async context**.

---

# 3. `.await` doesn't mean "block the entire program"

This is one of the most important things to understand.

When you write:

```rust
sleep(Duration::from_secs(5)).await;
```

you're saying:

> "This task needs to wait for 5 seconds."

During that waiting period, the async runtime can work on other tasks.

Example:

```rust
use tokio::time::{sleep, Duration};

async fn task_one() {
    println!("Task one started");

    sleep(Duration::from_secs(3)).await;

    println!("Task one finished");
}

async fn task_two() {
    println!("Task two started");

    sleep(Duration::from_secs(1)).await;

    println!("Task two finished");
}

#[tokio::main]
async fn main() {
    tokio::join!(
        task_one(),
        task_two()
    );
}
```

Output:

```text
Task one started
Task two started
Task two finished
Task one finished
```

While `task_one` was waiting, `task_two` was able to make progress.

---

# 4. `.await` gets the Future's output

Suppose:

```rust
async fn get_number() -> i32 {
    42
}
```

Then:

```rust
let number = get_number().await;
```

`number` is:

```rust
i32
```

Not:

```rust
Future
```

Think:

```text
Future<i32>
     ↓
  .await
     ↓
    i32
```

---

# 5. `.await` with `Result`

This is extremely important for backend development.

```rust
async fn get_user() -> Result<String, String> {
    Ok(String::from("Alice"))
}

#[tokio::main]
async fn main() {
    let result = get_user().await;

    match result {
        Ok(user) => println!("User: {}", user),
        Err(error) => println!("Error: {}", error),
    }
}
```

After:

```rust
get_user().await
```

you have:

```rust
Result<String, String>
```

Then you handle the `Result`.

---

# 6. `.await?`

In backend development, you'll very frequently see:

```rust
let user = get_user().await?;
```

Example:

```rust
async fn get_user() -> Result<String, String> {
    Ok(String::from("Alice"))
}

async fn run() -> Result<(), String> {
    let user = get_user().await?;

    println!("{}", user);

    Ok(())
}

#[tokio::main]
async fn main() {
    run().await.unwrap();
}
```

Understand this line:

```rust
let user = get_user().await?;
```

It means:

1. Call `get_user()`.
2. Wait for the Future with `.await`.
3. If successful, extract the value.
4. If there's an error, return that error from `run()`.

You will see this constantly with database and HTTP code.

---

# 7. Multiple `.await`s

You can await several operations:

```rust
async fn get_user() -> String {
    String::from("Alice")
}

async fn get_email() -> String {
    String::from("alice@example.com")
}

#[tokio::main]
async fn main() {
    let user = get_user().await;
    let email = get_email().await;

    println!("{} {}", user, email);
}
```

These are **sequential**:

```text
get_user()
    ↓
.await
    ↓
get_email()
    ↓
.await
```

The second operation starts after the first has completed.

---

# 8. `.await` with concurrent operations

If the operations don't depend on each other, you can run them concurrently:

```rust
async fn get_user() -> String {
    String::from("Alice")
}

async fn get_email() -> String {
    String::from("alice@example.com")
}

#[tokio::main]
async fn main() {
    let (user, email) = tokio::join!(
        get_user(),
        get_email()
    );

    println!("{} {}", user, email);
}
```

Now both Futures can make progress concurrently.

This matters when your backend needs to perform independent I/O operations.

---

# 9. `.await` with HTTP requests

Later you'll use libraries such as `reqwest`.

Conceptually:

```rust
let response = reqwest::get("https://example.com")
    .await?;
```

The important part is:

```rust
.await
```

The HTTP request is asynchronous.

The Future represents the operation:

```text
Send HTTP request
       ↓
     Future
       ↓
     .await
       ↓
 HTTP response
```

---

# 10. `.await` with PostgreSQL

When you reach SQLx, you'll see something like:

```rust
let user = sqlx::query!("SELECT * FROM users")
    .fetch_one(&pool)
    .await?;
```

Again, the important concept is:

```rust
.await?
```

The database operation is asynchronous.

```text
SQL query
   ↓
Future
   ↓
.await
   ↓
Database result
```

---

# 11. `.await` doesn't automatically make operations concurrent

This is a common mistake.

This:

```rust
let a = task_a().await;
let b = task_b().await;
```

is sequential.

It does **not** mean both operations run concurrently.

For concurrent execution, use something like:

```rust
let (a, b) = tokio::join!(
    task_a(),
    task_b()
);
```

### Remember:

```rust
task_a().await;
task_b().await;
```

→ **Sequential**

```rust
tokio::join!(
    task_a(),
    task_b()
);
```

→ **Concurrent**

---

# 12. `.await` vs `await`

In Rust, you don't write:

```rust
await get_data();
```

You write:

```rust
get_data().await;
```

`.await` is an **operator applied to a Future**.

---

# 13. The backend pattern you need to recognize

When you eventually build an Axum API, you'll encounter patterns like:

```rust
async fn get_user() -> Result<User, Error> {
    let user = database_query().await?;

    Ok(user)
}
```

The flow is:

```text
HTTP request
     ↓
async handler
     ↓
database_query()
     ↓
   Future
     ↓
   .await
     ↓
database result
     ↓
HTTP response
```

This is the practical purpose of `.await` in your backend journey.

---

# What you MUST know for now

You should be able to explain these clearly:

### `.await`

> Waits for a Future to complete and produces its output.

### Example

```rust
let data = get_data().await;
```

### `.await?`

> Waits for a Future returning `Result`, extracts the successful value, or returns the error.

```rust
let data = get_data().await?;
```

### Sequential

```rust
let a = task_a().await;
let b = task_b().await;
```

### Concurrent

```rust
let (a, b) = tokio::join!(
    task_a(),
    task_b()
);
```

### `.await` requires async context

```rust
async fn run() {
    let data = get_data().await;
}
```

---

## Your mental model

Memorize this:

```text
async function
      ↓
    Future
      ↓
   .await
      ↓
    result
```

And when you see:

```rust
let result = something().await?;
```

read it as:

> **"Run this asynchronous operation, wait for its result, and return the error if it fails."**

That's all you need to know about `.await` for now before moving deeper into **Tokio and concurrent tasks**.
