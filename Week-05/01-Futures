For now, we aim to understand **what a Future is, how it gets executed, and how `async`/`.await` work with Tokio**.

# 1. What is a Future?

A `Future` represents **a value that will be available later**.

Think of it like ordering food:

```text
You order food
      ↓
Food is being prepared
      ↓
You wait/do something else
      ↓
Food is ready
```

In Rust, the "food being prepared" is a **Future**.

A Future doesn't necessarily do the work immediately. It represents work that can produce a result later.

---

# 2. A Future is a Rust trait

Rust has a `Future` trait:

```rust
trait Future {
    type Output;

    // simplified
}
```

The important part is:

```rust
type Output;
```

It represents the type of value the Future eventually produces.

For example:

```text
Future<Output = String>
```

means:

> "This Future will eventually produce a `String`."

---

# 3. `async fn` creates a Future

Consider:

```rust
async fn get_name() -> String {
    String::from("Alice")
}
```

Calling it:

```rust
let name = get_name();
```

does **not** give you a `String`.

It gives you a **Future**.

Conceptually:

```text
get_name()
    ↓
Future
    ↓
.await
    ↓
String
```

That's why you use:

```rust
let name = get_name().await;
```

Now `name` is a `String`.

---

# 4. What does `.await` mean?

`.await` means:

> "Wait until this Future produces its result."

Example:

```rust
async fn get_name() -> String {
    String::from("Alice")
}

#[tokio::main]
async fn main() {
    let name = get_name().await;

    println!("{}", name);
}
```

Output:

```text
Alice
```

The important distinction is:

```rust
get_name()
```

→ Future

while:

```rust
get_name().await
```

→ actual result

---

# 5. Why can't we use `.await` inside normal `fn`?

This won't work:

```rust
fn main() {
    let name = get_name().await;
}
```

because `.await` can only be used inside an `async` context.

That's why we commonly use Tokio:

```rust
#[tokio::main]
async fn main() {
    let name = get_name().await;

    println!("{}", name);
}
```

`#[tokio::main]` sets up Tokio's runtime so your asynchronous code can execute.

---

# 6. Future doesn't automatically execute

This is an important concept.

Consider:

```rust
async fn say_hello() {
    println!("Hello");
}

fn main() {
    let future = say_hello();
}
```

You haven't actually completed the asynchronous operation.

You created a Future.

You need an async runtime such as Tokio to execute it:

```rust
#[tokio::main]
async fn main() {
    say_hello().await;
}
```

Now:

```text
Future
   ↓
Tokio runtime
   ↓
Future executes
   ↓
"Hello"
```

---

# 7. `async` vs `await`

Remember this simple rule:

### `async`

Creates a Future.

```rust
async fn fetch_data() -> String {
    String::from("data")
}
```

### `.await`

Gets the result from the Future.

```rust
let data = fetch_data().await;
```

So:

```rust
async fn fetch_data() -> String {
    String::from("data")
}

#[tokio::main]
async fn main() {
    let data = fetch_data().await;

    println!("{}", data);
}
```

---

# 8. Futures can return `Result`

This is extremely important for backend development.

You will frequently see:

```rust
async fn get_user() -> Result<String, String> {
    Ok(String::from("Alice"))
}
```

Then:

```rust
#[tokio::main]
async fn main() {
    let result = get_user().await;

    match result {
        Ok(user) => println!("User: {}", user),
        Err(error) => println!("Error: {}", error),
    }
}
```

You'll eventually use this pattern constantly with:

- Database queries
- HTTP requests
- Authentication
- File operations
- External APIs

---

# 9. Futures and delays

Let's use Tokio's timer to see asynchronous waiting.

Add Tokio:

```bash
cargo add tokio --features full
```

Then:

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

Output:

```text
Starting...
```

After 2 seconds:

```text
Finished!
```

The important thing is that:

```rust
sleep(...).await;
```

is an asynchronous wait.

---

# 10. Sequential Futures

Suppose you have two operations:

```rust
use tokio::time::{sleep, Duration};

async fn task_one() {
    sleep(Duration::from_secs(2)).await;
    println!("Task one done");
}

async fn task_two() {
    sleep(Duration::from_secs(2)).await;
    println!("Task two done");
}

#[tokio::main]
async fn main() {
    task_one().await;
    task_two().await;
}
```

The total time is roughly:

```text
2 seconds
   +
2 seconds
   =
4 seconds
```

because you're waiting for one Future before starting the other.

---

# 11. Running Futures concurrently

This is where Futures become powerful for backend development.

Tokio provides `join!`:

```rust
use tokio::time::{sleep, Duration};

async fn task_one() {
    sleep(Duration::from_secs(2)).await;
    println!("Task one done");
}

async fn task_two() {
    sleep(Duration::from_secs(2)).await;
    println!("Task two done");
}

#[tokio::main]
async fn main() {
    tokio::join!(
        task_one(),
        task_two()
    );
}
```

Both operations can make progress concurrently.

Instead of roughly:

```text
Task 1: ██████████ 2s
Task 2:           ██████████ 2s

Total: ~4s
```

you get:

```text
Task 1: ██████████
Task 2: ██████████

Total: ~2s
```

This is a major reason async programming is useful for backend servers.

---

# 12. Concurrency ≠ parallelism

Don't confuse these.

### Concurrency

Multiple tasks make progress during overlapping periods.

```text
Task A ███   ███
Task B   ███   ███
```

### Parallelism

Tasks literally execute at the same time on different CPU cores.

```text
CPU 1: █████████
CPU 2: █████████
```

Tokio's async programming is primarily about **concurrency**, which is especially useful for I/O-heavy backend work.

---

# 13. Futures and HTTP requests

This is where you'll start seeing the practical value.

Imagine:

```rust
async fn fetch_user() -> String {
    // HTTP request
    String::from("User data")
}
```

You might eventually write:

```rust
let user = fetch_user().await;
```

The HTTP request can spend time waiting for the server/network.

Instead of blocking the entire program during that wait, the async runtime can work on other tasks.

This is one of the fundamental ideas behind asynchronous web servers.

---

# 14. Futures and databases

The same concept applies to PostgreSQL.

Eventually you'll write things like:

```rust
let user = sqlx::query!("SELECT * FROM users")
    .fetch_one(&pool)
    .await?;
```

Notice:

```rust
.await
```

The database operation is asynchronous.

While the database is responding, Tokio can allow other tasks to make progress.

You don't need to understand SQLx yet. Just recognize the pattern:

```text
Database query
      ↓
Future
      ↓
.await
      ↓
Database result
```

---

# 15. What you should understand about Futures for now

For your roadmap, **this is enough**:

### Must know

- What a Future represents.
- `async fn` returns a Future.
- `.await` waits for a Future's result.
- Futures need an executor/runtime to run.
- Tokio provides the runtime you'll use.
- Async code is useful for I/O-heavy applications.
- Multiple Futures can run concurrently.
- `tokio::join!` can wait for multiple Futures concurrently.
- Async functions commonly return `Result`.

### You don't need yet

Don't get bogged down in:

- Implementing `Future` manually
- `Pin`
- `Poll`
- `Waker`
- Writing your own executor
- Advanced async trait patterns
- Runtime internals

Those are useful eventually, but **not necessary for your current goal of becoming a Rust backend developer**.

---

# Mental model

Keep this picture in your head:

```text
              async fn
                 │
                 ▼
              Future
                 │
                 │ .await
                 ▼
              Result
```

And for backend applications:

```text
HTTP request
     │
     ▼
  async handler
     │
     ├──────► Database Future
     │             │
     │           .await
     │             │
     │             ▼
     │        Database result
     │
     └──────► HTTP response
```

That's the core of what you need to understand before moving into **Tokio → Axum → REST APIs**.
