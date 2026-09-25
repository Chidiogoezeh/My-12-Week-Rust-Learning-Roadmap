For backend development, you should understand **what validation is, where it happens, how to define rules, and how to return useful errors to the client**.

# Validation in Rust

## 1. What is validation?

Validation means checking whether data supplied by a client is acceptable **before your application processes or stores it**.

For example, a registration request might contain:

```json
{
  "username": "chidi",
  "email": "chidi@example.com",
  "password": "123"
}
```

Your backend might require:

- username isn't empty
- email has a valid format
- password is at least 8 characters

If the request doesn't satisfy those rules, the server should reject it.

Conceptually:

```text
Client
   ↓
Request
   ↓
Validation
   ↓
Valid? ── No ──→ 400 Bad Request
   │
  Yes
   ↓
Business logic
   ↓
Database
```

**Important:** validation is not the same thing as authentication or authorization.

- **Validation:** Is the data acceptable?
- **Authentication:** Who are you?
- **Authorization:** Are you allowed to do this?

---

# 2. Simple validation without a library

Before using `validator`, understand how validation works manually.

Suppose we have:

```rust
struct RegisterUser {
    username: String,
    email: String,
    password: String,
}
```

We can write:

```rust
fn validate_user(user: &RegisterUser) -> Result<(), String> {
    if user.username.is_empty() {
        return Err("Username cannot be empty".to_string());
    }

    if user.email.is_empty() {
        return Err("Email cannot be empty".to_string());
    }

    if user.password.len() < 8 {
        return Err("Password must be at least 8 characters".to_string());
    }

    Ok(())
}
```

Then:

```rust
fn main() {
    let user = RegisterUser {
        username: "chidi".to_string(),
        email: "chidi@example.com".to_string(),
        password: "123".to_string(),
    };

    match validate_user(&user) {
        Ok(()) => println!("User is valid"),
        Err(error) => println!("Validation error: {}", error),
    }
}
```

Because the password is only three characters:

```text
Validation error: Password must be at least 8 characters
```

### What you should understand from this

The important pattern is:

```rust
Result<(), String>
```

The function says:

> "If validation succeeds, give me `Ok(())`. If it fails, give me an error."

This is the foundation you need to understand before introducing a validation crate.

---

# 3. Why use a validation library?

Imagine you have 20 fields:

```text
username
email
password
first_name
last_name
phone
age
address
...
```

Writing every validation rule manually can become repetitive.

A validation library allows you to describe the rules directly on your struct.

We'll use the `validator` crate.

Install it:

```bash
cargo add validator --features derive
```

---

# 4. Basic `validator` example

```rust
use validator::Validate;

#[derive(Validate)]
struct RegisterUser {
    #[validate(length(min = 3))]
    username: String,

    #[validate(email)]
    email: String,

    #[validate(length(min = 8))]
    password: String,
}
```

Now we can validate:

```rust
let user = RegisterUser {
    username: "ch".to_string(),
    email: "wrong-email".to_string(),
    password: "123".to_string(),
};

if let Err(errors) = user.validate() {
    println!("{:#?}", errors);
}
```

The library checks the rules for us.

---

# 5. Common validation rules

These are the main ones you should know.

### Minimum length

```rust
#[validate(length(min = 3))]
username: String,
```

### Maximum length

```rust
#[validate(length(max = 50))]
username: String,
```

### Minimum and maximum

```rust
#[validate(length(min = 3, max = 50))]
username: String,
```

### Email

```rust
#[validate(email)]
email: String,
```

### Range

For numbers:

```rust
#[validate(range(min = 18, max = 100))]
age: u32,
```

### Regex

For specific formats:

```rust
#[validate(regex(path = *PHONE_REGEX))]
phone: String,
```

You don't need to memorize every validation rule. Learn the common ones and look up specialized rules when needed.

---

# 6. Validation in an Axum API

Now we connect validation to what you've already learned in previous weeks.

Suppose we have:

```rust
use axum::Json;
use serde::Deserialize;
use validator::Validate;

#[derive(Deserialize, Validate)]
struct RegisterRequest {
    #[validate(length(min = 3))]
    username: String,

    #[validate(email)]
    email: String,

    #[validate(length(min = 8))]
    password: String,
}
```

Our handler receives it:

```rust
async fn register(
    Json(payload): Json<RegisterRequest>,
) {
    // validation happens here
}
```

We can validate:

```rust
if let Err(errors) = payload.validate() {
    println!("{:#?}", errors);
    return;
}
```

Then:

```rust
async fn register(
    Json(payload): Json<RegisterRequest>,
) {
    if let Err(errors) = payload.validate() {
        println!("{:#?}", errors);
        return;
    }

    println!("User is valid!");

    // Continue to business logic...
}
```

The important idea is:

```text
HTTP request
     ↓
Deserialize JSON
     ↓
Validate
     ↓
Business logic
     ↓
Database
```

---

# 7. Returning a proper HTTP response

We don't want to simply print the validation error.

The client should receive something like:

```json
{
  "error": "Validation failed"
}
```

with HTTP status:

```text
400 Bad Request
```

A simple Axum version:

```rust
use axum::{
    http::StatusCode,
    Json,
};
```

Then:

```rust
async fn register(
    Json(payload): Json<RegisterRequest>,
) -> (StatusCode, Json<String>) {

    if let Err(errors) = payload.validate() {
        return (
            StatusCode::BAD_REQUEST,
            Json(errors.to_string()),
        );
    }

    (
        StatusCode::OK,
        Json("User is valid".to_string()),
    )
}
```

That's enough for you to understand the basic flow.

---

# 8. Better validation errors

Eventually, you want an API response like:

```json
{
  "error": "Validation failed",
  "fields": {
    "email": "Invalid email",
    "password": "Must be at least 8 characters"
  }
}
```

This is much more useful to frontend developers and API consumers.

But **don't jump into building a complicated custom error system yet**.

For now, understand the basic progression:

```text
Validation fails
       ↓
validator::ValidationErrors
       ↓
Convert to API error
       ↓
HTTP 400
       ↓
JSON response
```

Later, when you learn centralized error handling, you'll make this cleaner.

---

# 9. Validation vs database constraints

This is **very important** for backend development.

Suppose:

```rust
#[validate(email)]
email: String,
```

checks that the email looks valid.

That doesn't mean the email is unique.

For example:

```text
john@example.com
```

might be correctly formatted but already belong to another user.

So you need both:

### Application validation

```rust
#[validate(email)]
email: String,
```

and a database constraint:

```sql
CREATE UNIQUE INDEX users_email_unique
ON users(email);
```

Think of it this way:

```text
Validation
    ↓
"Does this data look acceptable?"

Database constraints
    ↓
"Can this data actually exist in the database?"
```

**Never rely exclusively on application-level validation for important data integrity rules.**

---

# 10. Validation vs sanitization

These are also different.

### Validation

Asks:

> Is this input acceptable?

Example:

```text
email = "john@example.com"
```

### Sanitization

Changes or cleans input.

For example:

```text
"  john@example.com  "
```

might become:

```text
"john@example.com"
```

Don't confuse the two.

---

# 11. What should you validate?

For your backend projects, you'll commonly validate:

### Registration

```text
username → minimum length
email → valid email
password → minimum length
```

### Login

```text
email → valid format
password → not empty
```

### Creating an event

```text
title → required
description → required
capacity → positive number
date → valid date
```

### Creating a ticket

```text
event_id → required
quantity → positive number
```

---

# 12. Where should validation happen?

For a typical backend:

```text
                HTTP Request
                     │
                     ▼
              Deserialize JSON
                     │
                     ▼
                 Validation
                     │
               ┌─────┴─────┐
               │           │
            Invalid       Valid
               │           │
               ▼           ▼
            400         Service
                           │
                           ▼
                       Repository
                           │
                           ▼
                       PostgreSQL
```

Your handler shouldn't blindly send client input directly to the database.

---

# 13. What you should be able to write by the end of the week

You should be comfortable writing something like:

```rust
use serde::Deserialize;
use validator::Validate;

#[derive(Debug, Deserialize, Validate)]
struct CreateUserRequest {
    #[validate(length(min = 3, max = 30))]
    username: String,

    #[validate(email)]
    email: String,

    #[validate(length(min = 8))]
    password: String,
}
```

Then:

```rust
async fn create_user(
    Json(payload): Json<CreateUserRequest>,
) {
    if let Err(errors) = payload.validate() {
        // Return 400
        println!("{:#?}", errors);
        return;
    }

    // Validated data can now move
    // into your service layer.
}
```

And understand exactly what happens:

```text
JSON
 ↓
CreateUserRequest
 ↓
validate()
 ↓
invalid → 400
       OR
valid
 ↓
Service
 ↓
Database
```

---

# Your validation checklist

Don't move on until you understand these:

- [ ] What validation is
- [ ] Why validation is needed
- [ ] `Result` for handling validation failures
- [ ] Manual validation
- [ ] `validator` crate
- [ ] `#[derive(Validate)]`
- [ ] `#[validate(...)]`
- [ ] `length`
- [ ] `email`
- [ ] `range`
- [ ] Basic regex validation
- [ ] Calling `.validate()`
- [ ] Returning HTTP `400 Bad Request`
- [ ] Validation vs authentication
- [ ] Validation vs authorization
- [ ] Validation vs database constraints
- [ ] Validation vs sanitization
- [ ] Where validation belongs in an API request flow

### The one principle to remember

> **Never trust data just because it came from your API client. Validate it before your business logic or database operations.**

For your Rust backend roadmap, that's essentially the **core of validation**.
