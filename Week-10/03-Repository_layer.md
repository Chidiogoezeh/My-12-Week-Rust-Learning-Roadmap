## Repository Layer

The **Repository Layer** is the part of your backend responsible for **talking to the database**.

The simplest mental model is:

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

Each layer has **one main job**:

| Layer      | Responsibility        |
| ---------- | --------------------- |
| Handler    | HTTP request/response |
| Service    | Business logic        |
| Repository | Database operations   |
| PostgreSQL | Store data            |

---

# 1. What problem does the Repository Layer solve?

Without a repository, your service might contain SQL directly:

```rust
async fn register_user(
    &self,
    email: &str,
) -> Result<User, AppError> {

    let user = sqlx::query_as!(
        User,
        "SELECT id, email FROM users WHERE email = $1",
        email
    )
    .fetch_one(&self.db)
    .await?;

    // business logic...
}
```

Now the service knows:

- business rules
- SQL
- database details

That makes the service harder to maintain.

Instead:

```text
Service
   ↓
UserRepository
   ↓
SQLx
   ↓
PostgreSQL
```

The service simply says:

> "Find this user."

The repository handles **how** to find them.

---

# 2. What should a Repository contain?

A repository normally contains database operations such as:

```text
find_by_id()
find_by_email()
create()
update()
delete()
list()
```

For example:

```rust
pub struct UserRepository {
    db: PgPool,
}
```

The repository receives the database connection pool.

---

# 3. Create a User model

For example:

```rust
#[derive(Debug)]
pub struct User {
    pub id: i32,
    pub email: String,
    pub password_hash: String,
}
```

This represents data from the `users` table.

---

# 4. Create the Repository

```rust
use sqlx::PgPool;

pub struct UserRepository {
    db: PgPool,
}

impl UserRepository {
    pub fn new(db: PgPool) -> Self {
        Self { db }
    }
}
```

Now the repository has access to PostgreSQL.

---

# 5. `find_by_id`

```rust
impl UserRepository {
    pub async fn find_by_id(
        &self,
        id: i32,
    ) -> Result<Option<User>, sqlx::Error> {

        let user = sqlx::query_as!(
            User,
            r#"
            SELECT id, email, password_hash
            FROM users
            WHERE id = $1
            "#,
            id
        )
        .fetch_optional(&self.db)
        .await?;

        Ok(user)
    }
}
```

Important part:

```rust
.fetch_optional()
```

means:

```text
User found     → Some(User)
User not found → None
```

This is useful because "user doesn't exist" isn't necessarily a database error.

---

# 6. `find_by_email`

This is very common during login and registration.

```rust
pub async fn find_by_email(
    &self,
    email: &str,
) -> Result<Option<User>, sqlx::Error> {

    let user = sqlx::query_as!(
        User,
        r#"
        SELECT id, email, password_hash
        FROM users
        WHERE email = $1
        "#,
        email
    )
    .fetch_optional(&self.db)
    .await?;

    Ok(user)
}
```

The repository doesn't decide:

> "Is this email allowed to register?"

It only answers:

> "Does a user with this email exist?"

That distinction is important.

---

# 7. `create`

```rust
pub async fn create(
    &self,
    email: &str,
    password_hash: &str,
) -> Result<User, sqlx::Error> {

    let user = sqlx::query_as!(
        User,
        r#"
        INSERT INTO users (email, password_hash)
        VALUES ($1, $2)
        RETURNING id, email, password_hash
        "#,
        email,
        password_hash
    )
    .fetch_one(&self.db)
    .await?;

    Ok(user)
}
```

The repository knows:

- which table
- which columns
- which SQL query

The service doesn't need to know those details.

---

# 8. `delete`

```rust
pub async fn delete(
    &self,
    id: i32,
) -> Result<(), sqlx::Error> {

    sqlx::query!(
        "DELETE FROM users WHERE id = $1",
        id
    )
    .execute(&self.db)
    .await?;

    Ok(())
}
```

Again, this is purely persistence logic.

---

# 9. How the Service uses the Repository

Now we have:

```rust
pub struct UserService {
    repository: UserRepository,
}
```

The service can use it:

```rust
impl UserService {
    pub async fn register(
        &self,
        email: String,
        password_hash: String,
    ) -> Result<User, AppError> {

        let existing_user =
            self.repository
                .find_by_email(&email)
                .await?;

        if existing_user.is_some() {
            return Err(AppError::Conflict);
        }

        let user = self
            .repository
            .create(&email, &password_hash)
            .await?;

        Ok(user)
    }
}
```

Notice the separation.

### Repository:

```rust
find_by_email()
create()
```

### Service:

```rust
if existing_user.is_some() {
    return Err(AppError::Conflict);
}
```

The **business rule** belongs to the service.

---

# 10. What should NOT go into the Repository?

This is important.

### Don't put business rules here

Bad:

```rust
pub async fn register_user(&self, email: &str) {
    if email.ends_with("@gmail.com") {
        // ...
    }
}
```

That's business logic.

It belongs in the **service**.

---

### Don't put HTTP logic here

Don't do this:

```rust
if user.is_none() {
    return StatusCode::NOT_FOUND;
}
```

The repository shouldn't know about HTTP.

It should return data/errors.

---

### Don't hash passwords here

Don't make the repository responsible for:

```rust
hash_password(password)
```

Password hashing is application/security logic.

The service can prepare the hash, then pass it to:

```rust
repository.create(email, password_hash)
```

---

# 11. Repository vs Service

This is probably the most important distinction for this Week.

Imagine registering a user.

### Handler

```text
Receive HTTP request
        ↓
Extract email/password
        ↓
Call service
```

### Service

```text
Check whether email exists
        ↓
Hash password
        ↓
Apply registration rules
        ↓
Ask repository to create user
```

### Repository

```text
Execute INSERT SQL
        ↓
Return created user
```

### Database

```text
Actually store the user
```

So:

```text
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

# 12. Recommended structure

Keep it simple:

```text
src/
├── main.rs
├── handlers/
│   └── user.rs
├── services/
│   └── user.rs
├── repositories/
│   └── user.rs
├── models/
│   └── user.rs
├── routes/
│   └── user.rs
└── errors.rs
```

Your repository file:

```text
repositories/user.rs
```

contains things like:

```rust
pub struct UserRepository {
    db: PgPool,
}

impl UserRepository {
    pub fn new(db: PgPool) -> Self {
        Self { db }
    }

    pub async fn find_by_id(...) { ... }

    pub async fn find_by_email(...) { ... }

    pub async fn create(...) { ... }

    pub async fn update(...) { ... }

    pub async fn delete(...) { ... }
}
```

---

# 13. Do you need Repository traits?

**Not necessarily.**

You may see something like:

```rust
trait UserRepository {
    async fn find_by_email(...);
    async fn create(...);
}
```

and then:

```rust
struct PostgresUserRepository { ... }
```

This can be useful for testing or multiple database implementations.

But **don't add this complexity just because you are learning architecture.**

For now, start with:

```rust
struct UserRepository {
    db: PgPool,
}
```

That's enough.

Later, when you need mocking/testing or multiple implementations, learn repository traits.

---

# 14. Dependency Injection connection

This connects directly to the previous topic.

Your `main.rs` creates the repository:

```rust
let user_repository = UserRepository::new(db.clone());
```

Then creates the service:

```rust
let user_service = UserService::new(user_repository);
```

Conceptually:

```text
main.rs
   │
   ├── creates PgPool
   │
   ├── creates UserRepository
   │
   └── creates UserService
             │
             └── contains UserRepository
```

The dependency flow is:

```text
PgPool
  ↓
UserRepository
  ↓
UserService
  ↓
Handler
```

---

# 15. The rule you should remember

For now, remember these three sentences:

> **Handler handles HTTP.**

> **Service handles business logic.**

> **Repository handles database persistence.**

And therefore:

```text
Handler
   ↓
Service
   ↓
Repository
   ↓
Database
```

If you understand that separation and can implement `find`, `create`, `update`, and `delete` repositories with SQLx, **you know the core Repository Layer requirement for now.**
