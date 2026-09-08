For now, we only need to understand the core idea:

> **A handler is the Rust function that receives an HTTP request, does something with it, and returns an HTTP response.**

The flow is:

```text
Client
   ↓
GET /users/5
   ↓
Router
   ↓
get_user()
   ↓
Response
```

# 1. A basic handler

Start with the simplest possible handler:

```rust
async fn hello() -> &'static str {
    "Hello, World!"
}
```

Connect it to a route:

```rust
use axum::{
    routing::get,
    Router,
};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(hello));

    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}

async fn hello() -> &'static str {
    "Hello, World!"
}
```

When a client sends:

```text
GET /
```

Axum calls:

```rust
hello()
```

and the response is:

```text
Hello, World!
```

---

# 2. Why handlers use `async`

You'll notice:

```rust
async fn hello()
```

instead of:

```rust
fn hello()
```

Web servers spend a lot of time waiting for things such as:

- Database queries
- Other APIs
- Files
- Network requests

`async` allows Rust to handle these operations efficiently.

You'll learn more about this with **Tokio**. For now, just understand that **Axum handlers are commonly async functions**.

---

# 3. Returning different types of responses

A handler can return a string:

```rust
async fn hello() -> &'static str {
    "Hello"
}
```

It can return a `String`:

```rust
async fn greeting() -> String {
    String::from("Hello, Daud")
}
```

Or:

```rust
async fn greeting() -> String {
    format!("Hello, {}", "World")
}
```

Axum converts these into HTTP responses.

---

# 4. Returning JSON

This is **very important for backend development**.

Your API normally returns JSON rather than plain text.

Add Serde:

```bash
cargo add serde --features derive
cargo add serde_json
```

Create a struct:

```rust
use serde::Serialize;

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
}
```

Then return JSON:

```rust
use axum::Json;

async fn get_user() -> Json<User> {
    Json(User {
        id: 1,
        name: String::from("Alice"),
    })
}
```

Route:

```rust
.route("/user", get(get_user))
```

The response will look like:

```json
{
  "id": 1,
  "name": "Alice"
}
```

### Important concept

`Serialize` tells Serde:

> "Convert this Rust struct into a format such as JSON."

`Json<User>` tells Axum:

> "Return this value as a JSON HTTP response."

---

# 5. Receiving JSON from a client

Handlers don't just return data. They can **receive data**.

Suppose the client sends:

```json
{
  "name": "Alice"
}
```

Create a struct:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct CreateUser {
    name: String,
}
```

Then:

```rust
use axum::Json;

async fn create_user(
    Json(user): Json<CreateUser>,
) -> String {
    format!("Created user: {}", user.name)
}
```

Route:

```rust
.route("/users", post(create_user))
```

The important part is:

```rust
Json(user): Json<CreateUser>
```

Axum takes the incoming JSON and converts it into your Rust struct.

So:

```text
JSON request
      ↓
   Axum
      ↓
CreateUser struct
      ↓
 handler
```

---

# 6. Path parameters

Remember your route from the previous lesson:

```rust
.route("/users/:id", get(get_user))
```

A client could request:

```text
GET /users/25
```

The handler can get `25` using `Path`:

```rust
use axum::extract::Path;

async fn get_user(
    Path(id): Path<u32>,
) -> String {
    format!("User ID: {}", id)
}
```

Now:

```text
GET /users/25
```

returns:

```text
User ID: 25
```

This is one of the most important handler concepts to understand.

---

# 7. Query parameters

You will also encounter URLs like:

```text
GET /users?page=2
```

The `page=2` part is a **query parameter**.

Create a struct:

```rust
use axum::extract::Query;
use serde::Deserialize;

#[derive(Deserialize)]
struct Pagination {
    page: u32,
}
```

Handler:

```rust
async fn get_users(
    Query(params): Query<Pagination>,
) -> String {
    format!("Page: {}", params.page)
}
```

Route:

```rust
.route("/users", get(get_users))
```

Request:

```text
GET /users?page=2
```

Result:

```text
Page: 2
```

You don't need complicated query handling yet. Just understand how Axum extracts query parameters.

---

# 8. Returning HTTP status codes

A real API needs more than `200 OK`.

For example:

```text
200 OK
201 Created
400 Bad Request
401 Unauthorized
404 Not Found
500 Internal Server Error
```

Axum allows you to return a status code together with your response.

```rust
use axum::http::StatusCode;

async fn create_user() -> (StatusCode, &'static str) {
    (StatusCode::CREATED, "User created")
}
```

The client receives:

```text
HTTP 201 Created
```

with:

```text
User created
```

---

# 9. Status code + JSON

You'll commonly do this:

```rust
use axum::{
    http::StatusCode,
    Json,
};
use serde::Serialize;

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
}

async fn create_user() -> (StatusCode, Json<User>) {
    let user = User {
        id: 1,
        name: String::from("Alice"),
    };

    (StatusCode::CREATED, Json(user))
}
```

Response:

```text
201 Created
```

```json
{
  "id": 1,
  "name": "Alice"
}
```

---

# 10. Accessing request headers

Handlers can also access HTTP headers.

For example:

```rust
use axum::http::HeaderMap;

async fn check_headers(headers: HeaderMap) -> String {
    format!("Headers received: {}", headers.len())
}
```

Route:

```rust
.route("/headers", get(check_headers))
```

You'll use headers heavily later for things such as:

````text
Authorization: Bearer <token>

---

# 11. Combining extractors

This is where handlers become useful.

Imagine:

```text
POST /users/25
````

with:

```json
{
  "name": "Alice"
}
```

You can get both the path parameter and JSON body:

```rust
use axum::{
    extract::Path,
    Json,
};
use serde::Deserialize;

#[derive(Deserialize)]
struct UpdateUser {
    name: String,
}

async fn update_user(
    Path(id): Path<u32>,
    Json(user): Json<UpdateUser>,
) -> String {
    format!("Updating user {} to {}", id, user.name)
}
```

A request:

```text
POST /users/25
```

with:

```json
{
  "name": "Alice"
}
```

produces:

```text
Updating user 25 to Alice
```

This demonstrates an important concept:

> **A handler can extract different parts of an HTTP request.**

---

# 12. Handler parameters are extractors

You'll frequently see Axum handlers like:

```rust
async fn create_user(
    Json(user): Json<CreateUser>,
)
```

or:

```rust
async fn get_user(
    Path(id): Path<u32>,
)
```

or:

```rust
async fn get_users(
    Query(params): Query<Pagination>,
)
```

These are called **extractors**.

They extract information from the incoming HTTP request.

The main ones you need for now are:

| Extractor   | Gets              |
| ----------- | ----------------- |
| `Path`      | `/users/25`       |
| `Query`     | `?page=2`         |
| `Json`      | JSON request body |
| `HeaderMap` | HTTP headers      |

You don't need to memorize every Axum extractor right now.

---

# 13. A simple User API

Now let's put the concepts together.

```rust
use axum::{
    extract::{Path, Json},
    http::StatusCode,
    routing::{get, post, put, delete},
    Router,
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

async fn get_user(
    Path(id): Path<u32>,
) -> String {
    format!("Getting user {}", id)
}

async fn create_user(
    Json(user): Json<CreateUser>,
) -> (StatusCode, Json<User>) {
    let user = User {
        id: 3,
        name: user.name,
    };

    (StatusCode::CREATED, Json(user))
}

async fn update_user(
    Path(id): Path<u32>,
    Json(user): Json<CreateUser>,
) -> String {
    format!("Updating user {} to {}", id, user.name)
}

async fn delete_user(
    Path(id): Path<u32>,
) -> String {
    format!("Deleting user {}", id)
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/users", get(get_users).post(create_user))
        .route(
            "/users/:id",
            get(get_user)
                .put(update_user)
                .delete(delete_user),
        );

    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```

Don't worry about making this production-ready yet. **The purpose is to understand the relationship between routes, handlers, requests, and responses.**

---

# The mental model you should remember

When building an Axum backend, think:

```text
             HTTP REQUEST
                  │
                  ▼
              ┌───────┐
              │ Router│
              └───┬───┘
                  │
                  ▼
              ┌─────────┐
              │ Handler │
              └────┬────┘
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
        Path     Query     JSON
          │        │        │
          └────────┼────────┘
                   ▼
              Business Logic
                   │
                   ▼
              HTTP RESPONSE
                   │
                   ▼
                Client
```

Later, your handler will call a **service**, which may call a **repository**, which talks to **PostgreSQL**:

```text
Request
   ↓
Router
   ↓
Handler
   ↓
Service
   ↓
Repository
   ↓
PostgreSQL
   ↓
Repository
   ↓
Service
   ↓
Handler
   ↓
JSON Response
```

But **don't build all of that now**.

## Handler checklist

You should be able to explain and write:

- [ ] A basic `async` handler
- [ ] Returning a string
- [ ] Returning JSON with `Json`
- [ ] Receiving JSON with `Json<T>`
- [ ] Extracting path parameters with `Path`
- [ ] Extracting query parameters with `Query`
- [ ] Reading headers
- [ ] Returning HTTP status codes
- [ ] Combining extractors in one handler
- [ ] Connecting handlers to routes

If you understand those, you've learned **enough about Axum handlers for now**. The next logical topic is **JSON request/response handling**, because that's what turns your simple Axum handlers into an actual REST API.
