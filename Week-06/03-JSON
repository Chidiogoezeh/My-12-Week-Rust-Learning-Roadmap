JSON is one of the most important things to understand because REST APIs commonly use JSON to communicate between the **client and server**.

Our goal is to understand:

> **Rust struct ↔ JSON**

and:

> **JSON request → Rust struct → handler → Rust struct → JSON response**

---

# 1. What is JSON?

JSON is a text format for sending structured data.

Example:

```json
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}
```

A Rust equivalent could be:

```rust
struct User {
    id: u32,
    name: String,
    email: String,
}
```

Your API converts between these two representations.

```text
JSON
 ↓
Rust struct
 ↓
Your application
 ↓
Rust struct
 ↓
JSON
```

---

# 2. Add Serde

In Rust, **Serde** handles serialization and deserialization.

Add it to your project:

```bash
cargo add serde --features derive
cargo add serde_json
```

You should understand these two terms:

### Serialization

Rust → JSON

```text
Rust struct → JSON
```

### Deserialization

JSON → Rust struct

```text
JSON → Rust struct
```

---

# 3. Serialize a Rust struct

Start with:

```rust
use serde::Serialize;

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
}
```

The important part is:

```rust
#[derive(Serialize)]
```

It tells Serde that `User` can be converted into JSON.

For example:

```rust
let user = User {
    id: 1,
    name: String::from("Alice"),
};
```

Can become:

```json
{
  "id": 1,
  "name": "Alice"
}
```

---

# 4. Return JSON from an Axum handler

This is what you'll actually do in your REST API.

```rust
use axum::Json;
use serde::Serialize;

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
}

async fn get_user() -> Json<User> {
    let user = User {
        id: 1,
        name: String::from("Alice"),
    };

    Json(user)
}
```

Then connect it to a route:

```rust
use axum::{
    routing::get,
    Router,
};

let app = Router::new()
    .route("/user", get(get_user));
```

Request:

```text
GET /user
```

Response:

```json
{
  "id": 1,
  "name": "Alice"
}
```

### Remember

```rust
Json(user)
```

tells Axum:

> "Return this Rust value as JSON."

---

# 5. Deserializing JSON

Now let's go the other direction.

Suppose a client sends:

```json
{
  "name": "Alice",
  "email": "alice@example.com"
}
```

Create a Rust struct:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}
```

`Deserialize` tells Serde:

> "Convert incoming JSON into this Rust struct."

---

# 6. Receive JSON in an Axum handler

Use Axum's `Json` extractor:

```rust
use axum::Json;

async fn create_user(
    Json(user): Json<CreateUser>,
) -> String {
    format!("Creating user: {}", user.name)
}
```

Then:

```rust
.route("/users", post(create_user))
```

A client sends:

```json
{
  "name": "Alice",
  "email": "alice@example.com"
}
```

Axum converts it into:

```rust
CreateUser {
    name: String::from("Alice"),
    email: String::from("alice@example.com"),
}
```

Your handler can then work with normal Rust values.

---

# 7. Returning JSON after receiving JSON

This is the pattern you'll use constantly.

```rust
use axum::Json;
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct CreateUser {
    name: String,
}

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
}

async fn create_user(
    Json(input): Json<CreateUser>,
) -> Json<User> {
    let user = User {
        id: 1,
        name: input.name,
    };

    Json(user)
}
```

Client sends:

```json
{
  "name": "Alice"
}
```

Server responds:

```json
{
  "id": 1,
  "name": "Alice"
}
```

The flow is:

```text
Client
  │
  │ JSON
  ▼
Axum
  │
  │ Deserialize
  ▼
CreateUser
  │
  ▼
Handler
  │
  │ Create User
  ▼
User
  │
  │ Serialize
  ▼
JSON
  │
  ▼
Client
```

**This is the core JSON concept you need for now.**

---

# 8. JSON with multiple users

You can return a vector:

```rust
async fn get_users() -> Json<Vec<User>> {
    let users = vec![
        User {
            id: 1,
            name: String::from("Alice"),
        },
        User {
            id: 2,
            name: String::from("Bob"),
        },
    ];

    Json(users)
}
```

Response:

```json
[
  {
    "id": 1,
    "name": "Alice"
  },
  {
    "id": 2,
    "name": "Bob"
  }
]
```

Notice:

```rust
Json<Vec<User>>
```

means:

> JSON containing a vector of `User`.

---

# 9. JSON field names

By default, Rust field names become JSON field names.

```rust
#[derive(Serialize)]
struct User {
    first_name: String,
    last_name: String,
}
```

Produces:

```json
{
  "first_name": "Alice",
  "last_name": "Smith"
}
```

You can change the JSON name with Serde:

```rust
use serde::Serialize;

#[derive(Serialize)]
struct User {
    #[serde(rename = "firstName")]
    first_name: String,

    #[serde(rename = "lastName")]
    last_name: String,
}
```

Now:

```json
{
  "firstName": "Alice",
  "lastName": "Smith"
}
```

You don't need to focus heavily on this yet, but you should know it exists.

---

# 10. Optional JSON fields

Suppose the client doesn't have to provide an email.

Use `Option`:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: Option<String>,
}
```

The client can send:

```json
{
  "name": "Alice"
}
```

or:

```json
{
  "name": "Alice",
  "email": "alice@example.com"
}
```

Then:

```rust
if let Some(email) = user.email {
    println!("Email: {}", email);
}
```

This connects directly with the `Option` concept you learned earlier.

---

# 11. JSON and HTTP status codes

A good API doesn't just return JSON. It also returns the appropriate HTTP status.

For example, when creating a user:

```rust
use axum::{
    http::StatusCode,
    Json,
};

async fn create_user() -> (StatusCode, Json<User>) {
    let user = User {
        id: 1,
        name: String::from("Alice"),
    };

    (StatusCode::CREATED, Json(user))
}
```

The client gets:

```text
HTTP 201 Created
```

and:

```json
{
  "id": 1,
  "name": "Alice"
}
```

---

# 12. JSON error responses

Your API will also need to return errors as JSON.

For example:

```rust
use axum::{
    http::StatusCode,
    Json,
};
use serde::Serialize;

#[derive(Serialize)]
struct ErrorResponse {
    message: String,
}

async fn user_not_found() -> (StatusCode, Json<ErrorResponse>) {
    (
        StatusCode::NOT_FOUND,
        Json(ErrorResponse {
            message: String::from("User not found"),
        }),
    )
}
```

Response:

```text
HTTP 404 Not Found
```

```json
{
  "message": "User not found"
}
```

This is the beginning of proper API error handling. Later, you'll build a cleaner reusable error system.

---

# 13. `serde_json` vs Axum's `Json`

This can initially be confusing.

### `serde`

Handles conversion between Rust types and formats.

```rust
Serialize
Deserialize
```

### `serde_json`

Specifically handles JSON.

For example:

```rust
let json = serde_json::to_string(&user).unwrap();
```

### Axum `Json`

Handles JSON specifically in HTTP requests/responses:

```rust
Json(user)
```

For now, think of it like this:

```text
Serde
  ↓
Rust ↔ data format

serde_json
  ↓
Rust ↔ JSON

Axum Json
  ↓
HTTP ↔ JSON ↔ Rust
```

---

# 14. Complete small example

Here's a simple API that demonstrates the important concepts:

```rust
use axum::{
    routing::{get, post},
    Json, Router,
};
use serde::{Deserialize, Serialize};

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
}

#[derive(Deserialize)]
struct CreateUser {
    name: String,
}

async fn get_user() -> Json<User> {
    Json(User {
        id: 1,
        name: String::from("Alice"),
    })
}

async fn create_user(
    Json(input): Json<CreateUser>,
) -> Json<User> {
    Json(User {
        id: 2,
        name: input.name,
    })
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/user", get(get_user))
        .route("/users", post(create_user));

    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```

### `GET /user`

Response:

```json
{
  "id": 1,
  "name": "Alice"
}
```

### `POST /users`

Request:

```json
{
  "name": "Bob"
}
```

Response:

```json
{
  "id": 2,
  "name": "Bob"
}
```

---

# What you actually need to know for now

Don't try to memorize the entire Serde ecosystem. Make sure you understand these:

### 1. `Serialize`

```rust
#[derive(Serialize)]
```

**Rust → JSON**

### 2. `Deserialize`

```rust
#[derive(Deserialize)]
```

**JSON → Rust**

### 3. Axum `Json`

```rust
Json(user)
```

Return JSON.

### 4. Receiving JSON

```rust
Json(input): Json<CreateUser>
```

Extract JSON from a request.

### 5. JSON arrays

```rust
Json<Vec<User>>
```

Return multiple objects.

### 6. Optional fields

```rust
email: Option<String>
```

Allow a JSON field to be absent.

### 7. Status + JSON

```rust
(StatusCode::CREATED, Json(user))
```

Return both an HTTP status and JSON.

---

## The one concept I want you to remember

For now, this is the pattern for our REST API:

```text
                    REQUEST
                       │
                       ▼
                  JSON data
                       │
                       ▼
               Deserialize
                       │
                       ▼
                 Rust struct
                       │
                       ▼
                    Handler
                       │
                       ▼
                 Rust struct
                       │
                       ▼
                  Serialize
                       │
                       ▼
                  JSON data
                       │
                       ▼
                   RESPONSE
```

Once you understand this, **Routes + Handlers + JSON** give you the foundation for your first real REST API. PostgreSQL in Week 7 will then replace your temporary in-memory data with persistent data.
