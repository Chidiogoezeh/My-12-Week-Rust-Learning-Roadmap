For now, you should understand **why structured logging exists, how to create logs in Rust, how to attach useful context, and how to control log levels**.

For your Rust backend stack, we'll use **`tracing` + `tracing-subscriber`**.

# Structured Logging

## 1. What is logging?

Logging is your application's way of recording what is happening while it runs.

For example:

```text
User registered
User logged in
Database connection failed
Request took 250ms
```

When your API is running on your computer, you can see these messages in the terminal.

When your API is deployed, logs help you understand what happened when something goes wrong.

---

# 2. What is structured logging?

A normal log might look like:

```text
User chidi logged in
```

A structured log contains **separate pieces of information**:

```text
user_id=42 action="login" email="chidi@example.com"
```

Instead of putting everything into one sentence, you attach **fields** to the event.

This makes logs easier for logging systems to search, filter, and analyze.

Think:

```text
Normal logging:

"Chidi logged in successfully"

Structured logging:

action="login"
user_id=42
success=true
```

---

# 3. Why use `tracing`?

Rust's `tracing` ecosystem is commonly used for structured application logging.

Install:

```bash
cargo add tracing
cargo add tracing-subscriber --features fmt,env-filter
```

You'll primarily use:

```rust
tracing
```

and:

```rust
tracing-subscriber
```

---

# 4. Your first log

Start with:

```rust
use tracing::info;

fn main() {
    info!("Application started");
}
```

But you won't see anything yet because we haven't configured a subscriber.

---

# 5. Configure the logger

Create:

```rust
use tracing_subscriber;

fn main() {
    tracing_subscriber::fmt::init();

    tracing::info!("Application started");
}
```

Run:

```bash
cargo run
```

You'll see something similar to:

```text
2026-09-25T14:00:00.123Z  INFO test_project: Application started
```

The exact timestamp and formatting will vary.

---

# 6. Log levels

You need to know these four:

| Level    | Meaning                                        |
| -------- | ---------------------------------------------- |
| `error!` | Something failed                               |
| `warn!`  | Something unusual/problematic                  |
| `info!`  | Normal important application event             |
| `debug!` | Detailed information useful during development |

### `info!`

```rust
tracing::info!("User registered");
```

### `debug!`

```rust
tracing::debug!("Processing registration request");
```

### `warn!`

```rust
tracing::warn!("Login attempt failed");
```

### `error!`

```rust
tracing::error!("Database connection failed");
```

You don't need to use `debug` for everything.

---

# 7. Structured fields

This is where structured logging becomes useful.

Instead of:

```rust
tracing::info!("User 42 logged in");
```

write:

```rust
tracing::info!(
    user_id = 42,
    action = "login",
    "User logged in"
);
```

Now the log contains structured fields:

```text
user_id=42 action="login" User logged in
```

The important part is:

```rust
user_id = 42
action = "login"
```

These are **fields**.

---

# 8. Multiple fields

For example:

```rust
let user_id = 42;
let email = "chidi@example.com";

tracing::info!(
    user_id,
    email,
    action = "login",
    "User logged in successfully"
);
```

This gives your logging system useful information such as:

```text
user_id=42
email="chidi@example.com"
action="login"
```

---

# 9. Don't log sensitive information

This is extremely important for backend development.

### Never do this:

```rust
tracing::info!(
    email = user.email,
    password = user.password,
    "User login"
);
```

Passwords should **never** appear in logs.

Be careful with:

- Passwords
- JWTs
- API keys
- Access tokens
- Refresh tokens
- Credit/debit card information
- Secrets

Instead:

```rust
tracing::info!(
    user_id = user.id,
    "User login successful"
);
```

---

# 10. Logging an error

Suppose a database operation fails:

```rust
match create_user().await {
    Ok(user) => {
        tracing::info!(
            user_id = user.id,
            "User created"
        );
    }

    Err(error) => {
        tracing::error!(
            error = ?error,
            "Failed to create user"
        );
    }
}
```

Notice:

```rust
error = ?error
```

The `?` tells `tracing` to use the `Debug` representation.

You'll frequently see this when logging Rust errors.

---

# 11. `?` vs `%`

You'll encounter this:

```rust
tracing::error!(error = ?error, "Something failed");
```

and:

```rust
tracing::info!(email = %email, "User logged in");
```

For now, remember:

```text
?value
```

→ Debug formatting

```text
%value
```

→ Display formatting

You don't need to go deeper than that for now.

---

# 12. Logging inside an Axum handler

Suppose you have:

```rust
async fn register(
    Json(payload): Json<RegisterRequest>,
) {
    tracing::info!("Registration request received");

    // registration logic
}
```

Better:

```rust
async fn register(
    Json(payload): Json<RegisterRequest>,
) {
    tracing::info!(
        email = %payload.email,
        "Registration request received"
    );

    // registration logic
}
```

Again, **don't log the password**.

---

# 13. Logging validation errors

This connects directly to what you just learned.

```rust
async fn register(
    Json(payload): Json<RegisterRequest>,
) {
    if let Err(errors) = payload.validate() {
        tracing::warn!(
            errors = ?errors,
            "Registration validation failed"
        );

        // Return 400
        return;
    }

    tracing::info!("Registration validation passed");

    // Continue...
}
```

Now your logs tell you:

```text
WARN Registration validation failed
```

instead of silently returning an error.

---

# 14. Request-level logging

For backend APIs, you eventually want to know:

```text
HTTP method
URL
status
duration
```

For example:

```text
method=POST
path=/users
status=201
duration_ms=42
```

With Axum, middleware can handle this.

A simple example uses `TraceLayer` from Tower HTTP.

Install:

```bash
cargo add tower-http --features trace
```

Then:

```rust
use tower_http::trace::TraceLayer;

let app = Router::new()
    .route("/users", post(create_user))
    .layer(TraceLayer::new_for_http());
```

Now HTTP requests can automatically produce tracing information.

This is something you should understand conceptually for now; you don't need to build an elaborate logging system yet.

---

# 15. `#[instrument]`

This is another important `tracing` feature.

Suppose:

```rust
async fn create_user() {
    tracing::info!("Creating user");
}
```

You can instead use:

```rust
use tracing::instrument;

#[instrument]
async fn create_user() {
    tracing::info!("Creating user");
}
```

`#[instrument]` creates a **span** around the function.

A span represents a period of time during which something is happening.

For example:

```text
create_user
    │
    ├── validate user
    ├── hash password
    ├── insert database
    └── return response
```

This becomes very useful as your application becomes more complex.

---

# 16. Adding fields to `#[instrument]`

You can attach useful information:

```rust
#[instrument]
async fn get_user(user_id: i32) {
    tracing::info!("Fetching user");
}
```

The log can contain:

```text
get_user{user_id=42}
```

You can also tell `tracing` which arguments **not** to record:

```rust
#[instrument(skip(password))]
async fn create_user(email: String, password: String) {
    // ...
}
```

This is particularly important when a function receives sensitive information.

---

# 17. Controlling log levels

You don't always want every debug message in production.

Set a filter:

```rust
use tracing_subscriber::EnvFilter;

fn main() {
    tracing_subscriber::fmt()
        .with_env_filter(
            EnvFilter::new("info")
        )
        .init();

    tracing::debug!("Debug message");
    tracing::info!("Info message");
}
```

With `info`, you'll see:

```text
INFO Info message
```

but not:

```text
DEBUG Debug message
```

You can also use the environment variable:

### Windows CMD

```cmd
set RUST_LOG=debug
cargo run
```

Then debug logs will appear.

For PowerShell:

```powershell
$env:RUST_LOG="debug"
cargo run
```

---

# 18. Logging in your backend architecture

As your week's architecture develops:

```text
Request
   ↓
Handler
   ↓
Service
   ↓
Repository
   ↓
PostgreSQL
```

logs might appear at meaningful points:

```rust
tracing::info!("Request received");

tracing::info!("Creating user");

tracing::debug!("Validating user");

tracing::debug!("Hashing password");

tracing::info!("User created");

tracing::error!(
    error = ?error,
    "Database operation failed"
);
```

Don't log every single line.

Log **important events and failures**.

---

# 19. What NOT to do

### Don't use `println!` as your application's logging system

Instead of:

```rust
println!("User created");
```

use:

```rust
tracing::info!("User created");
```

`println!` is fine for simple CLI programs, but structured backend logging should use `tracing`.

---

### Don't log sensitive data

Bad:

```rust
tracing::info!(
    password = %password,
    "User login"
);
```

Good:

```rust
tracing::info!(
    user_id = user.id,
    "User login successful"
);
```

---

### Don't log everything as `error!`

Bad:

```rust
tracing::error!("User registered");
tracing::error!("Request received");
```

Use appropriate levels:

```rust
tracing::info!("User registered");
```

---

# 20. What you should know by the end of the week

You should be comfortable with:

### Basic logging

```rust
tracing::info!("Server started");
tracing::warn!("Something unusual happened");
tracing::error!("Database failed");
tracing::debug!("Processing request");
```

### Structured fields

```rust
tracing::info!(
    user_id = 42,
    action = "login",
    "User logged in"
);
```

### Error logging

```rust
tracing::error!(
    error = ?error,
    "Database operation failed"
);
```

### Initialization

```rust
tracing_subscriber::fmt::init();
```

### Filtering

```text
RUST_LOG=debug
```

### Function spans

```rust
#[instrument]
async fn create_user() {
    // ...
}
```

### HTTP request tracing

Understand what:

```rust
TraceLayer::new_for_http()
```

does.

### Security

Know **what must never appear in logs**.

---

# The mental model to remember

Don't think of structured logging as:

> "Printing messages to the terminal."

Think of it as:

> **Recording important events and their context so you can understand what your backend was doing.**

For example:

```rust
tracing::info!(
    user_id = 42,
    action = "login",
    "Login successful"
);
```

The message tells you **what happened**:

```text
Login successful
```

The fields tell you **the context**:

```text
user_id=42
action=login
```

That's the core of structured logging you need for now.
