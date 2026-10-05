For now, you do not need to learn dependency injection as a huge theoretical subject. For Rust backend development, you mainly need to understand **how to provide shared dependencies to your handlers/services without creating them everywhere**.

# Dependency Injection

## 1. What problem does dependency injection solve?

Imagine your handler needs a database connection:

```rust
async fn get_users() {
    let db = create_database_connection();

    // use db...
}
```

This is a problem because the handler is responsible for **creating** its own dependency.

Instead, create the database connection once and **give it to the handler**.

That's dependency injection.

> **Dependency injection = give a component the things it needs instead of making it create those things itself.**

---

# 2. A simple Rust example

Suppose we have a service that sends emails.

Without dependency injection:

```rust
struct EmailService;

impl EmailService {
    fn send(&self, message: &str) {
        println!("Sending email: {}", message);
    }
}

fn register_user() {
    let email_service = EmailService;

    email_service.send("Welcome!");
}
```

`register_user()` creates the `EmailService` itself.

Instead:

```rust
struct EmailService;

impl EmailService {
    fn send(&self, message: &str) {
        println!("Sending email: {}", message);
    }
}

fn register_user(email_service: &EmailService) {
    email_service.send("Welcome!");
}

fn main() {
    let email_service = EmailService;

    register_user(&email_service);
}
```

Now:

```text
main
 │
 │ creates EmailService
 ▼
register_user
 │
 │ receives EmailService
 ▼
send email
```

That's dependency injection.

---

# 3. Why is this useful in backend applications?

A real backend has many shared dependencies:

```text
Application
│
├── PostgreSQL connection pool
├── Configuration
├── Email service
├── Redis client
└── Authentication service
```

You don't want every handler creating these independently.

Instead:

```text
main.rs
   │
   ├── creates database pool
   ├── creates configuration
   ├── creates services
   │
   ▼
Application State
   │
   ├── User handler
   ├── Auth handler
   ├── Task handler
   └── etc.
```

This is the pattern you'll use heavily with **Axum**.

---

# 4. Dependency injection with Axum

Let's start with a database pool.

```rust
use axum::{
    extract::State,
    routing::get,
    Router,
};
use sqlx::PgPool;

async fn get_users(
    State(db): State<PgPool>,
) -> String {
    println!("Using database: {:?}", db);

    "Users".to_string()
}
```

Then in `main`:

```rust
#[tokio::main]
async fn main() {
    let db = PgPool::connect("postgres://user:password@localhost/app")
        .await
        .unwrap();

    let app = Router::new()
        .route("/users", get(get_users))
        .with_state(db);

    // start server...
}
```

Notice what happened.

`main` created the database pool:

```rust
let db = PgPool::connect(...).await.unwrap();
```

Then injected it into the application:

```rust
.with_state(db)
```

The handler receives it:

```rust
State(db): State<PgPool>
```

The handler **doesn't create the database connection**.

That's the important concept.

---

# 5. Multiple dependencies

Real applications usually need more than one dependency.

For example:

```rust
struct AppState {
    db: PgPool,
    jwt_secret: String,
}
```

Now our application state contains multiple dependencies.

```rust
let state = AppState {
    db,
    jwt_secret: "secret".to_string(),
};
```

Give it to Axum:

```rust
let app = Router::new()
    .route("/users", get(get_users))
    .with_state(state);
```

Then:

```rust
async fn get_users(
    State(state): State<AppState>,
) -> String {
    println!("JWT secret: {}", state.jwt_secret);

    "Users".to_string()
}
```

This is much closer to how you'll structure a real backend.

---

# 6. Shared state

There's one important Rust detail.

Axum handlers can run concurrently, so application state often needs to be safely shared.

For simple shared state, you'll commonly use:

```rust
Arc
```

For example:

```rust
use std::sync::Arc;

struct AppState {
    db: PgPool,
}

type SharedState = Arc<AppState>;
```

Then:

```rust
let state = Arc::new(AppState { db });
```

And:

```rust
let app = Router::new()
    .route("/users", get(get_users))
    .with_state(state);
```

Handler:

```rust
async fn get_users(
    State(state): State<SharedState>,
) -> String {
    // use state.db

    "Users".to_string()
}
```

You don't need to memorize `Arc` yet.

The key idea is:

> Multiple requests can safely share the same application state.

---

# 7. Dependency injection into services

This becomes even more useful when you introduce this week's architecture.

For example:

```text
src/
├── handlers/
│   └── user.rs
├── services/
│   └── user.rs
├── repository/
│   └── user.rs
├── models/
│   └── user.rs
└── main.rs
```

Your handler shouldn't contain everything.

Instead:

```text
HTTP Request
     │
     ▼
Handler
     │
     ▼
Service
     │
     ▼
Repository
     │
     ▼
PostgreSQL
```

---

# 8. Example

Repository:

```rust
struct UserRepository {
    db: PgPool,
}
```

Service:

```rust
struct UserService {
    repository: UserRepository,
}
```

Handler:

```rust
async fn create_user(
    State(service): State<UserService>,
) -> String {
    service.create_user();

    "User created".to_string()
}
```

The dependency chain becomes:

```text
main
 │
 ├── creates PgPool
 │
 ├── creates UserRepository
 │       │
 │       └── receives PgPool
 │
 ├── creates UserService
 │       │
 │       └── receives UserRepository
 │
 ▼
Axum
 │
 ▼
Handler
```

This is dependency injection in a real backend architecture.

---

# 9. Why not just create everything inside the handler?

You could do this:

```rust
async fn create_user() {
    let db = PgPool::connect(...).await.unwrap();

    let repository = UserRepository { db };

    // ...
}
```

But this causes problems.

### Problem 1 — Repetition

Every handler may create the same dependencies.

### Problem 2 — Testing

Testing becomes harder because the handler is tightly coupled to the real database.

### Problem 3 — Configuration

You don't want database URLs, API keys, etc. scattered throughout your application.

### Problem 4 — Resource management

You generally want a shared connection pool rather than creating a new database pool for every request.

---

# 10. Dependency injection makes testing easier

Suppose your service needs a payment provider.

Instead of hard-coding:

```rust
struct PaymentService;

impl PaymentService {
    fn charge(&self) {
        // Paystack API
    }
}
```

You can define what the service needs through a trait:

```rust
trait PaymentProvider {
    fn charge(&self, amount: u64);
}
```

Real provider:

```rust
struct Paystack;

impl PaymentProvider for Paystack {
    fn charge(&self, amount: u64) {
        println!("Charging Paystack: {}", amount);
    }
}
```

Test provider:

```rust
struct FakePayment;

impl PaymentProvider for FakePayment {
    fn charge(&self, amount: u64) {
        println!("Fake payment: {}", amount);
    }
}
```

Now your service can receive whichever implementation it needs.

That's dependency injection combined with **traits**.

You don't need to master advanced dependency-injection frameworks in Rust. Rust typically uses **structs, constructors, traits, and application state** instead.

---

# 11. The pattern you should learn

For this Week's backend project, aim to understand this:

```text
             main.rs
                │
        creates dependencies
                │
                ▼
           AppState
        ┌───────┼────────┐
        │       │        │
       DB     Config   Services
        │       │        │
        └───────┼────────┘
                │
                ▼
             Routes
                │
                ▼
            Handlers
                │
                ▼
            Services
                │
                ▼
           Repository
                │
                ▼
           PostgreSQL
```

The important rule is:

> **Create dependencies at the application boundary and pass them down to the components that need them.**

---

# What you should know by the end of the Week

You don't need to know every dependency-injection pattern.

You should be able to:

### 1. Explain dependency injection

> A component receives its dependencies instead of creating them itself.

### 2. Pass dependencies into functions

```rust
fn create_user(service: &UserService) {
    // ...
}
```

### 3. Create application state

```rust
struct AppState {
    db: PgPool,
}
```

### 4. Inject state into Axum handlers

```rust
async fn handler(
    State(state): State<AppState>,
) {
    // ...
}
```

### 5. Understand shared state

Know why things like:

```rust
Arc<AppState>
```

may be necessary.

### 6. Inject dependencies into services

```rust
UserService {
    repository,
}
```

### 7. Understand traits for replaceable dependencies

```rust
trait PaymentProvider {
    // ...
}
```

This is particularly important for testing and external integrations.

---

## The one mental model to remember

Don't think:

```text
Handler
   ↓
"Let me create a database connection."
```

Think:

```text
main.rs
   ↓
"Here's the database connection you need."
   ↓
Handler
```

That's **dependency injection**.

For your Rust backend roadmap, **Axum `State` + structs + constructors + traits** is enough to master this topic initially. You do **not** need a complicated DI framework.
