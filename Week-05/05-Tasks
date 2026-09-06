For now, we should understand **Tasks** as the individual units of asynchronous work that Tokio can run concurrently.

## 1. What is a Task?

A **Tokio task** is an independently scheduled piece of asynchronous work.

You create one with:

```rust
tokio::spawn(...)
```

Example:

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    tokio::spawn(async {
        sleep(Duration::from_secs(2)).await;
        println!("Task finished");
    });

    println!("Main continues");

    sleep(Duration::from_secs(3)).await;
}
```

Here:

```rust
tokio::spawn(async {
    // task
});
```

creates a Tokio task.

---

## 2. Why use Tasks?

Tasks allow your program to handle multiple pieces of async work concurrently.

For example, a backend server might have:

```text
Task 1 → Handle HTTP request
Task 2 → Handle another HTTP request
Task 3 → Send an email
Task 4 → Process a database operation
```

While one task is waiting for I/O, Tokio can run another task.

---

## 3. `tokio::spawn`

Basic syntax:

```rust
let handle = tokio::spawn(async {
    // asynchronous work
});
```

`spawn` returns a **JoinHandle**.

The handle allows you to wait for the task:

```rust
let handle = tokio::spawn(async {
    println!("Hello from task");
});

handle.await.unwrap();
```

So:

```text
tokio::spawn()
      ↓
creates Task
      ↓
returns JoinHandle
      ↓
handle.await
      ↓
wait for Task
```

---

# 4. Tasks run concurrently

Example:

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
    let a = tokio::spawn(task_one());
    let b = tokio::spawn(task_two());

    a.await.unwrap();
    b.await.unwrap();

    println!("Both tasks finished");
}
```

Execution is roughly:

```text
Task 1 ──────────────── 2 seconds
Task 2 ─────── 1 second

Total ≈ 2 seconds
```

They don't have to wait for each other to start.

---

# 5. `spawn` vs normal function call

This:

```rust
task_one().await;
task_two().await;
```

is sequential.

```text
Task 1 → finish → Task 2 → finish
```

But this:

```rust
let a = tokio::spawn(task_one());
let b = tokio::spawn(task_two());
```

starts them as separate Tokio tasks.

```text
Task 1 ──────────→
Task 2 ───────→
```

That's the key difference.

---

# 6. Getting a value back from a Task

A task can return a value.

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    let handle = tokio::spawn(async {
        sleep(Duration::from_secs(1)).await;

        42
    });

    let result = handle.await.unwrap();

    println!("{}", result);
}
```

Output:

```text
42
```

The task returns `42`, and:

```rust
handle.await.unwrap()
```

gets the value.

---

# 7. Tasks can return `Result`

This is especially important for backend development.

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    let handle = tokio::spawn(async {
        sleep(Duration::from_secs(1)).await;

        Ok::<String, String>(String::from("User found"))
    });

    let result = handle.await.unwrap();

    match result {
        Ok(message) => println!("{}", message),
        Err(error) => println!("Error: {}", error),
    }
}
```

There are actually **two possible failure levels** here:

```text
JoinHandle
    ↓
Task itself could fail/panic
    ↓
Result returned by the task could also be Err
```

You'll encounter this pattern later, although you don't need to go deeply into it for now.

---

# 8. `spawn` does NOT mean "parallel"

This distinction is important.

Tokio tasks are primarily about **concurrency**.

```text
Concurrency:
Task A makes progress
Task B makes progress
Task C makes progress
```

That doesn't automatically mean they are running simultaneously on different CPU cores.

For now:

> **Tokio tasks are lightweight concurrent units of async work.**

That's the definition you should remember.

---

# 9. Tasks are useful for I/O

Tasks are particularly useful when work involves waiting:

```text
HTTP request
    ↓
waiting...

Database query
    ↓
waiting...

File operation
    ↓
waiting...
```

Instead of blocking the entire application, Tokio can schedule other tasks while those operations are waiting.

This is one reason async Rust is useful for backend servers.

---

# 10. A practical example

Imagine you want to send two notifications:

```rust
use tokio::time::{sleep, Duration};

async fn send_email() {
    sleep(Duration::from_secs(2)).await;
    println!("Email sent");
}

async fn send_sms() {
    sleep(Duration::from_secs(1)).await;
    println!("SMS sent");
}

#[tokio::main]
async fn main() {
    let email_task = tokio::spawn(send_email());
    let sms_task = tokio::spawn(send_sms());

    email_task.await.unwrap();
    sms_task.await.unwrap();

    println!("Notifications completed");
}
```

Both operations can make progress concurrently.

---

# 11. What happens if you don't await the task?

Consider:

```rust
#[tokio::main]
async fn main() {
    tokio::spawn(async {
        println!("Task running");
    });
}
```

The program may finish before you can rely on that task completing.

A spawned task is not automatically something your `main` function waits for.

If you need its result/completion, keep the handle:

```rust
let handle = tokio::spawn(async {
    println!("Task running");
});

handle.await.unwrap();
```

---

# 12. `spawn` vs `join!`

You should know both:

### `join!`

```rust
tokio::join!(task_one(), task_two());
```

Runs multiple futures concurrently and waits for them.

### `spawn`

```rust
let task = tokio::spawn(task_one());
```

Creates a separate Tokio task.

Simple distinction:

```text
join!
→ concurrent futures

spawn
→ separate Tokio task
```

You don't need to overthink this distinction yet.

---

# What you should know for now

You should be comfortable with:

- What a Tokio task is
- `tokio::spawn`
- `JoinHandle`
- `handle.await`
- Running multiple tasks concurrently
- Returning values from tasks
- Basic `Result` handling
- Difference between sequential execution and concurrent tasks
- Difference between `spawn` and `join!`
- Why tasks are useful for I/O-heavy backend applications

### Your mental model

```text
async function
      ↓
    Future
      ↓
 tokio::spawn()
      ↓
   Tokio Task
      ↓
 handle.await
      ↓
    Result
```
