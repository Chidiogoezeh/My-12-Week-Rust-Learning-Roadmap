For now, we should understand Tokio as the **async runtime that runs your Rust asynchronous code**.

We do **not** need to learn Tokio internals yet.

## 1. What is Tokio?

**Tokio is an asynchronous runtime for Rust.**

Rust's `async`/`.await` syntax alone doesn't execute asynchronous work. Tokio provides the runtime that schedules and runs those async tasks.

Think:

```text
Your async code
     ↓
 async / await
     ↓
   Tokio
     ↓
Runs and schedules async tasks
```

For backend development, Tokio is extremely important because frameworks like **Axum** use it for asynchronous HTTP servers.

---

# 2. Add Tokio to your project

In `Cargo.toml`:

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

Then:

```bash
cargo build
```

For now, `features = ["full"]` is fine. Later, you'll learn how to enable only the features you need.

---

# 3. `#[tokio::main]`

Normally, Rust starts with:

```rust
fn main() {
}
```

For an async program, you can use:

```rust
#[tokio::main]
async fn main() {
    println!("Hello from Tokio!");
}
```

`#[tokio::main]` sets up a Tokio runtime for your program.

Without a runtime, you can't simply do:

```rust
async fn main() {}
```

because Rust needs something to execute the async code.

---

# 4. Running an async function

Example:

```rust
async fn say_hello() {
    println!("Hello!");
}

#[tokio::main]
async fn main() {
    say_hello().await;
}
```

Important:

```rust
say_hello()
```

returns a **Future**.

```rust
say_hello().await
```

runs/waits for that Future to complete.

---

# 5. Tokio's `sleep`

Tokio provides async utilities such as timers.

```rust
use tokio::time::{sleep, Duration};

async fn do_work() {
    println!("Working...");

    sleep(Duration::from_secs(2)).await;

    println!("Done!");
}

#[tokio::main]
async fn main() {
    do_work().await;
}
```

The important part is:

```rust
sleep(Duration::from_secs(2)).await;
```

This is an **async sleep**.

It does **not** block the entire Tokio runtime while waiting.

---

# 6. Tokio and concurrency

This is one of the most important things to understand for now.

Suppose you have:

```rust
async fn task_one() {
    sleep(Duration::from_secs(2)).await;
    println!("Task one finished");
}

async fn task_two() {
    sleep(Duration::from_secs(1)).await;
    println!("Task two finished");
}
```

If you do:

```rust
task_one().await;
task_two().await;
```

they run sequentially:

```text
Task 1 → 2 seconds
Task 2 → 1 second

Total ≈ 3 seconds
```

But Tokio allows you to run independent async operations concurrently.

---

# 7. `tokio::join!`

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
    tokio::join!(task_one(), task_two());

    println!("Both finished");
}
```

Approximate execution:

```text
Task 1 ──────────────┐
                     │
Task 2 ────────┐     │
               ↓     ↓
             finished
```

Total time is roughly **2 seconds**, not 3.

### When to use `join!`

Use it when you have **independent async operations that you want to run concurrently**.

For example:

```text
Get user
Get user's orders
Get user's notifications
        ↓
    concurrently
```

---

# 8. `tokio::spawn`

`tokio::spawn` creates a separate asynchronous task.

Example:

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    let task = tokio::spawn(async {
        sleep(Duration::from_secs(2)).await;
        println!("Background task finished");
    });

    println!("Main function continues");

    task.await.unwrap();
}
```

Think of:

```rust
tokio::spawn(...)
```

as:

> "Tokio, start this async task separately and let my current code continue."

---

# 9. `spawn` vs `join!`

Know this distinction:

### `join!`

```rust
tokio::join!(task_one(), task_two());
```

Use when you want to **wait for multiple operations together**.

### `spawn`

```rust
tokio::spawn(async {
    // work
});
```

Use when you want to **create a separate Tokio task**.

Simple mental model:

```text
join!
 ↓
Run multiple futures concurrently
and wait for all of them

spawn
 ↓
Create a separate async task
```

---

# 10. Tokio + `Result`

This is extremely important for backend development.

```rust
use tokio::time::{sleep, Duration};

async fn get_user() -> Result<String, String> {
    sleep(Duration::from_secs(1)).await;

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

You'll frequently see this pattern in real Rust backend code:

```rust
let user = get_user().await?;
```

For example:

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

You should become comfortable with:

```rust
.await?
```

because you'll use it constantly with databases, HTTP requests, file operations, etc.

---

# 11. Tokio's role in a backend

This is the most important practical connection for your roadmap.

Imagine an API request:

```text
Client
  ↓
Axum
  ↓
Tokio
  ↓
Database query
  ↓
PostgreSQL
```

While PostgreSQL is processing the query, Tokio can allow other tasks to make progress.

For example:

```text
Request A → waiting for database
                  ↓
Request B → processing
                  ↓
Request C → waiting for database
                  ↓
Request A → database finished
```

That's why asynchronous runtimes are valuable for backend servers handling lots of I/O.

---

# 12. Tokio HTTP example

Later you'll use Tokio with libraries such as `reqwest`.

Conceptually:

```rust
async fn fetch_data() -> Result<String, reqwest::Error> {
    let response = reqwest::get("https://example.com").await?;

    let body = response.text().await?;

    Ok(body)
}
```

Notice the repeated pattern:

```rust
request.await?
      ↓
response.await?
      ↓
result
```

This is the same Tokio + async + await model you'll use with databases and APIs.

---

# 13. What you should know for now

You should be able to explain these without looking them up:

| Concept              | What you should know                                           |
| -------------------- | -------------------------------------------------------------- |
| Tokio                | Async runtime for Rust                                         |
| Runtime              | Executes/schedules async tasks                                 |
| `#[tokio::main]`     | Starts Tokio runtime for your `main` function                  |
| `async`              | Creates a Future                                               |
| `.await`             | Waits for a Future to complete                                 |
| `tokio::time::sleep` | Async timer                                                    |
| `tokio::join!`       | Run multiple futures concurrently and wait for them            |
| `tokio::spawn`       | Create a separate async task                                   |
| `Result` + async     | Common pattern for fallible async operations                   |
| `.await?`            | Await operation and propagate errors                           |
| Concurrency          | Multiple async tasks can make progress without blocking on I/O |

---

## Mental model

Memorize this:

```text
async
  ↓
Future
  ↓
Tokio runtime
  ↓
.await
  ↓
result
```

And for concurrency:

```text
                 ┌── Task A
Tokio runtime ───┼── Task B
                 └── Task C
```

### Don't study yet

**If you can build a small program using `#[tokio::main]`, `async`, `.await`, `sleep`, `join!`, `spawn`, and `Result`/`.await?`, you know the Tokio fundamentals you need for now.**
