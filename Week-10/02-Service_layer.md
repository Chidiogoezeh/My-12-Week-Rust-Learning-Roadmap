The service layer is one of the most important architectural concepts to understand because it separates **HTTP handling** from **business logic**.

You don't need a complicated architecture. Learn this simple flow:

```text
HTTP Request
     ↓
Handler
     ↓
Service
     ↓
Repository
     ↓
PostgreSQL
```

# 1. What is a service layer?

The **service layer contains your application's business logic**.

For example, suppose your API has:

```http
POST /users
```

The handler should deal with HTTP:

- Read the request
- Validate/extract data
- Call the service
- Return an HTTP response

The service should deal with the actual operation:

- Check whether the email exists
- Hash the password
- Create the user
- Apply business rules

The repository should deal with the database:

- `INSERT`
- `SELECT`
- `UPDATE`
- `DELETE`

---

# 2. Without a service layer

You might initially write:

```rust
async fn register_user(
    State(db): State<PgPool>,
    Json(request): Json<RegisterRequest>,
) -> Result<Json<User>, AppError> {

    // Check email
    let existing = sqlx::query!(
        "SELECT id FROM users WHERE email = $1",
        request.email
    )
    .fetch_optional(&db)
    .await?;

    if existing.is_some() {
        return Err(AppError::Conflict);
    }

    // Hash password
    let password_hash = hash_password(&request.password)?;

    // Insert user
    let user = sqlx::query_as!(
        User,
        "INSERT INTO users (email, password_hash)
         VALUES ($1, $2)
         RETURNING id, email",
        request.email,
        password_hash
    )
    .fetch_one(&db)
    .await?;

    Ok(Json(user))
}
```

It works.

But now your **HTTP handler is doing everything**:

```text
Handler
├── HTTP
├── Business logic
├── Password hashing
└── Database operations
```

As the application grows, this becomes difficult to maintain.

---

# 3. With a service layer

Instead:

```text
Handler
   ↓
UserService
   ↓
UserRepository
   ↓
PostgreSQL
```

The handler becomes much smaller.

```rust
async fn register(
    State(service): State<UserService>,
    Json(request): Json<RegisterRequest>,
) -> Result<Json<User>, AppError> {

    let user = service.register(request).await?;

    Ok(Json(user))
}
```

That's what you want.

---

# 4. The service

Now put the business logic inside `UserService`.

```rust
struct UserService {
    repository: UserRepository,
}
```

Then:

```rust
impl UserService {
    async fn register(
        &self,
        request: RegisterRequest,
    ) -> Result<User, AppError> {

        let existing = self
            .repository
            .find_by_email(&request.email)
            .await?;

        if existing.is_some() {
            return Err(AppError::Conflict);
        }

        let password_hash =
            hash_password(&request.password)?;

        let user = self
            .repository
            .create_user(
                &request.email,
                &password_hash,
            )
            .await?;

        Ok(user)
    }
}
```

Now the service owns the **business process**.

---

# 5. The repository

The repository handles database operations.

```rust
struct UserRepository {
    db: PgPool,
}
```

Then:

```rust
impl UserRepository {

    async fn find_by_email(
        &self,
        email: &str,
    ) -> Result<Option<User>, AppError> {

        let user = sqlx::query_as!(
            User,
            "SELECT id, email
             FROM users
             WHERE email = $1",
            email
        )
        .fetch_optional(&self.db)
        .await?;

        Ok(user)
    }

    async fn create_user(
        &self,
        email: &str,
        password_hash: &str,
    ) -> Result<User, AppError> {

        let user = sqlx::query_as!(
            User,
            "INSERT INTO users (email, password_hash)
             VALUES ($1, $2)
             RETURNING id, email",
            email,
            password_hash
        )
        .fetch_one(&self.db)
        .await?;

        Ok(user)
    }
}
```

Notice something important:

The repository **doesn't decide whether an email is allowed**.

It simply performs database operations.

---

# 6. Responsibilities

This is the distinction you should remember:

| Layer      | Responsibility |
| ---------- | -------------- |
| Handler    | HTTP           |
| Service    | Business logic |
| Repository | Database       |
| Model      | Data structure |

### Handler

```rust
async fn register(...) {
    // HTTP concerns
}
```

### Service

```rust
async fn register(...) {
    // Business rules
}
```

### Repository

```rust
async fn create_user(...) {
    // Database query
}
```

---

# 7. Real example: Task management

Suppose you have:

```http
POST /tasks
```

A user wants to create a task.

### Handler

```rust
async fn create_task(
    State(service): State<TaskService>,
    Json(request): Json<CreateTaskRequest>,
) -> Result<Json<Task>, AppError> {

    let task = service.create_task(request).await?;

    Ok(Json(task))
}
```

The handler doesn't decide anything about task ownership.

---

## Service

```rust
impl TaskService {

    async fn create_task(
        &self,
        request: CreateTaskRequest,
    ) -> Result<Task, AppError> {

        if request.title.trim().is_empty() {
            return Err(AppError::Validation(
                "Title is required".into()
            ));
        }

        let task = self
            .repository
            .create(
                request.user_id,
                &request.title,
            )
            .await?;

        Ok(task)
    }
}
```

The service decides:

> Is this task valid?

That's business logic.

---

## Repository

```rust
impl TaskRepository {

    async fn create(
        &self,
        user_id: i64,
        title: &str,
    ) -> Result<Task, AppError> {

        let task = sqlx::query_as!(
            Task,
            "INSERT INTO tasks (user_id, title)
             VALUES ($1, $2)
             RETURNING id, user_id, title",
            user_id,
            title
        )
        .fetch_one(&self.db)
        .await?;

        Ok(task)
    }
}
```

The repository simply says:

> Insert this task into PostgreSQL.

---

# 8. What counts as business logic?

This is important.

Business logic answers questions like:

### User registration

```text
Does this email already exist?
Is the password valid?
Should this user be created?
```

### Payments

```text
Can this payment be processed?
Has this payment already been processed?
Should the user receive a refund?
```

### Event ticketing

```text
Is the event still open?
Are tickets available?
Can this user buy another ticket?
How many tickets can they purchase?
```

### Task management

```text
Does this user own the task?
Can this task be completed?
Can a completed task be edited?
```

Those decisions belong in the **service layer**.

---

# 9. What should NOT be in the service?

Don't put HTTP-specific code there.

Avoid:

```rust
HttpStatusCode::BAD_REQUEST
```

or:

```rust
Json(...)
```

or:

```rust
Request<Body>
```

The service shouldn't care whether the request came from:

```text
REST API
     ↓
GraphQL
     ↓
CLI
     ↓
Background job
```

The service should simply perform the operation.

For example:

```rust
service.register(request).await
```

---

# 10. Why this is useful

Imagine tomorrow you create:

```text
POST /users
```

and also a background job:

```text
create_user_from_import()
```

Without a service layer, you might duplicate the registration logic.

With a service:

```text
REST Handler ──────┐
                   │
                   ▼
              UserService
                   ▲
                   │
Import Job ────────┘
```

Both use the same business logic.

---

# 11. Where dependency injection fits

This connects directly to what we discussed previously.

You create your dependencies in `main.rs`:

```rust
let repository = UserRepository {
    db: pool.clone(),
};

let service = UserService {
    repository,
};
```

Then give the service to Axum:

```rust
let app = Router::new()
    .route("/users", post(register))
    .with_state(service);
```

The handler receives it:

```rust
async fn register(
    State(service): State<UserService>,
    Json(request): Json<RegisterRequest>,
) -> Result<Json<User>, AppError> {

    let user = service.register(request).await?;

    Ok(Json(user))
}
```

So:

```text
main.rs
   │
   ├── PostgreSQL pool
   │
   ▼
Repository
   │
   ▼
Service
   │
   ▼
Handler
   │
   ▼
HTTP response
```

---

# 12. Recommended Weeks structure

Don't over-engineer it.

Start with:

```text
src/
├── main.rs
├── routes/
│   └── user.rs
├── handlers/
│   └── user.rs
├── services/
│   └── user.rs
├── repositories/
│   └── user.rs
├── models/
│   └── user.rs
└── errors.rs
```

The important flow is:

```text
routes
  ↓
handlers
  ↓
services
  ↓
repositories
  ↓
database
```

---

# 13. What you should know by the end of the Week

You should be able to explain:

### Service layer

> A service layer contains application/business logic and coordinates operations between handlers and repositories.

### Handler

> Handles HTTP concerns.

### Service

> Handles business rules and application operations.

### Repository

> Handles persistence/database operations.

And you should be comfortable writing:

```rust
struct UserService {
    repository: UserRepository,
}
```

and:

```rust
impl UserService {
    async fn register(
        &self,
        request: RegisterRequest,
    ) -> Result<User, AppError> {
        // business logic
    }
}
```

while keeping your handler simple:

```rust
async fn register(
    State(service): State<UserService>,
    Json(request): Json<RegisterRequest>,
) -> Result<Json<User>, AppError> {

    Ok(Json(service.register(request).await?))
}
```

## The mental model

Remember just this:

```text
HANDLER
"What HTTP request did I receive?"

        ↓

SERVICE
"What should the application do?"

        ↓

REPOSITORY
"How do I get/save the data?"

        ↓

DATABASE
"Store the data."
```

That is the level of **service-layer knowledge you need for the Week**. No need for a DI framework, CQRS, event sourcing, or other architectural patterns yet.
