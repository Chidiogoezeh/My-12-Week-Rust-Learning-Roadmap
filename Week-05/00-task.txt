# Week 5 – Async Programming

Topics:

- Futures
- Async
- Await
- Tokio
- Tasks

Example:

```rust
#[tokio::main]
async fn main() {
    println!("Hello Async");
}
```

### Project

Instead of building a concurrent URL fetcher that:

- Reads URLs
- Fetches multiple pages simultaneously
- Prints response times

Let's build on our previous project expense tracker, such that expense tracker can,
- Reads expense records, 
- Processes multiple expenses concurrently using Tokio tasks, 
- Calculates and displays expense summaries asynchronously.
