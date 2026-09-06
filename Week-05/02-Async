For now, our goal with **async in Rust** is simple:

> Understand how Rust handles operations that take time, especially network requests, database queries, and timers,without blocking the entire application.

# 1. What does `async` mean?

`async` tells Rust:

> "This function performs asynchronous work and will produce its result later."

Example:

```rust
async fn say_hello() {
    println!("Hello");
}
```

But calling:

```rust
say_hello();
```

doesn't execute the function to completion.

It creates a **Future**.

To execute it and get its result, you use `.await` inside an async context:

```rust
say_hello().await;
```

---

# 2. `async` and `.await` work together

Think of it like this:

```text
async
  ↓
creates a Future
  ↓
.await
  ↓
gets the result
```

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

Output:

```text
Daud
```

### Remember

```rust
get_name()
```

→ Future

```rust
get_name().await
```

→ `String`

---

# 3. Why do we need Tokio?

Rust's standard library doesn't provide the complete async runtime you need for typical backend applications.

**Tokio** provides the runtime that executes your async tasks.

Install it:

```bash
cargo add tokio --features full
```

Then:

```rust
#[tokio::main]
async fn main() {
    println!("Hello async Rust!");
}
```

The:

```rust
#[tokio::main]
```

macro starts the Tokio runtime and allows your `main` function to be asynchronous.

---

# 4. Async waiting

Here's a simple example:

```rust
use tokio::time::{sleep, Duration};

async fn do_work() {
    println!("Starting...");

    sleep(Duration::from_secs(2)).await;

    println!("Finished!");
}

#[tokio::main]
async fn main() {
    do_work().await;
}
```

The program:

```text
Starting...
     ↓
wait 2 seconds
     ↓
Finished!
```

The important part is:

```rust
sleep(Duration::from_secs(2)).await;
```

You're asynchronously waiting for the timer.

---

# 5. Async does NOT mean "faster"

This is important.

Async doesn't automatically make:

```text
one operation
```

faster.

Its main advantage is that your program can handle **other work while an operation is waiting**.

For example, a backend server might have:

```text
Request A → waiting for PostgreSQL
Request B → waiting for external API
Request C → processing
Request D → waiting for Redis
```

An async runtime can manage these operations efficiently instead of blocking the whole server whenever one operation is waiting for I/O.

---

# 6. Async vs synchronous

### Synchronous

```rust
fn task() {
    // do something
}

fn main() {
    task();
    task();
}
```

Conceptually:

```text
Task 1 ██████████
Task 2           ██████████
```

Task 2 waits for Task 1.

### Asynchronous

```rust
async fn task() {
    // asynchronous work
}
```

Multiple asynchronous operations can make progress while others are waiting.

```text
Task 1 ███    ███
Task 2   ███    ███
Task 3     ███    ███
```

That's particularly useful for backend applications because much backend work involves **I/O**.

---

# 7. Running async tasks concurrently

Tokio provides `join!`.

```rust
use tokio::time::{sleep, Duration};

async fn task_one() {
    sleep(Duration::from_secs(2)).await;
    println!("Task one finished");
}

async fn task_two() {
    sleep(Duration::from_secs(2)).await;
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

Both tasks can progress concurrently.

The important difference is:

### Sequential

```rust
task_one().await;
task_two().await;
```

### Concurrent

```rust
tokio::join!(
    task_one(),
    task_two()
);
```

---

# 8. `tokio::spawn`

You'll also encounter `tokio::spawn`.

It starts an asynchronous task independently.

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    let task = tokio::spawn(async {
        sleep(Duration::from_secs(2)).await;
        println!("Background task finished");
    });

    println!("Main task continues...");

    task.await.unwrap();
}
```

You can think of:

```rust
tokio::spawn(...)
```

as:

> "Start this async task separately and let it make progress concurrently."

You'll eventually use this for things such as background jobs.

---

# 9. Async functions can return values

```rust
async fn get_user() -> String {
    String::from("Alice")
}

#[tokio::main]
async fn main() {
    let user = get_user().await;

    println!("User: {}", user);
}
```

Output:

```text
User: Alice
```

---

# 10. Async functions can return `Result`

This is **very important for backend development**.

```rust
async fn get_user() -> Result<String, String> {
    Ok(String::from("Alice"))
}

#[tokio::main]
async fn main() {
    match get_user().await {
        Ok(user) => println!("User: {}", user),
        Err(error) => println!("Error: {}", error),
    }
}
```

You'll see this pattern constantly when working with:

- PostgreSQL
- HTTP APIs
- Redis
- Files
- Authentication
- Axum handlers

---

# 11. Async + `?`

Once you understand `Result`, you'll frequently combine it with `.await` and `?`.

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

The important line is:

```rust
let user = get_user().await?;
```

Read it as:

> "Wait for `get_user()`. If it succeeds, give me the value. If it fails, return the error."

This combination will become extremely familiar:

```rust
.await?
```

---

# 12. Async in backend development

Imagine you're building a REST API.

A request arrives:

```text
GET /users/10
```

Your handler might eventually look conceptually like:

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
database query
      ↓
.await
      ↓
database result
      ↓
HTTP response
```

This is why async is fundamental to Rust backend development.

---

# 13. Async + HTTP requests

Later, you might make an HTTP request:

```rust
async fn fetch_data() -> Result<String, reqwest::Error> {
    let response = reqwest::get("https://example.com")
        .await?
        .text()
        .await?;

    Ok(response)
}
```

Notice that there are two `.await`s:

```rust
.await?
```

for the HTTP response, and:

```rust
.text().await?
```

for reading the response body.

You don't need to memorize this yet. Understand the pattern:

```text
Network operation
       ↓
    Future
       ↓
    .await
       ↓
    Result
```

---

# 14. Async + PostgreSQL

Later with SQLx, you'll see code like:

```rust
let user = sqlx::query_as!(
    User,
    "SELECT id, name FROM users WHERE id = $1",
    user_id
)
.fetch_one(&pool)
.await?;
```

The critical part for now is:

```rust
.await?
```

The database query is asynchronous.

You don't need to learn SQLx yet.

---

# 15. Async does not mean every function should be async

Don't do this unnecessarily:

```rust
async fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

There's no asynchronous work here.

A normal function is better:

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

Use async when the function actually performs or coordinates asynchronous operations, such as:

- Network requests
- Database queries
- Timers
- File I/O
- Async channels
- Other async functions

---

# 16. Async vs threads

Don't confuse:

```rust
async
```

with:

```rust
thread
```

Async is primarily about efficiently handling **waiting/I/O**.

Threads are operating-system execution units.

For example:

```text
Async:

Request → waiting for database
             ↓
          handle another request
```

Whereas threads are about actual execution contexts:

```text
Thread 1 → executes task A
Thread 2 → executes task B
```

Tokio can use multiple threads underneath, but you don't need to understand the runtime's threading architecture yet.

---

# 17. The most important code to practice

Create a project:

```bash
cargo new async_practice
cd async_practice
cargo add tokio --features full
```

Replace `src/main.rs` with:

```rust
use tokio::time::{sleep, Duration};

async fn task_one() {
    sleep(Duration::from_secs(2)).await;
    println!("Task one finished");
}

async fn task_two() {
    sleep(Duration::from_secs(1)).await;
    println!("Task two finished");
}

#[tokio::main]
async fn main() {
    tokio::join!(
        task_one(),
        task_two()
    );

    println!("All tasks finished");
}
```

Run:

```bash
cargo run
```

You should see:

```text
Task two finished
Task one finished
All tasks finished
```

Notice that **Task two finishes first**, even though `task_one()` appears first in the code.

That's concurrency.

---

# What you MUST know for now

You should be able to explain these without looking them up:

### `async`

> Makes a function asynchronous and causes it to return a Future.

### `.await`

> Waits for a Future to complete and gives you its output.

### Tokio

> An async runtime commonly used to execute Rust asynchronous code.

### `tokio::join!`

> Runs multiple async operations concurrently and waits for all of them.

### `tokio::spawn`

> Starts an asynchronous task that can run concurrently with other tasks.

### `Result + async`

You should understand:

```rust
let data = some_async_operation().await?;
```

### Most importantly

Understand this:

```text
async function
      ↓
    Future
      ↓
   .await
      ↓
    result
```

And for backend development:

```text
           Rust Backend
                │
          ┌─────┴─────┐
          ↓           ↓
       HTTP API    PostgreSQL
          │           │
       async        async
          │           │
          └─────┬─────┘
                ↓
              Tokio
```
