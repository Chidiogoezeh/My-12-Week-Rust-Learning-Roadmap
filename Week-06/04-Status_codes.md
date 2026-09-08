For **REST API**, we should understand status codes well enough to know **what response your API should send and why**.

## 1. What is a status code?

An HTTP status code tells the client **what happened to the request**.

For example:

```text
GET /users/1

200 OK
```

means the request succeeded and the user was found.

While:

```text
GET /users/999

404 Not Found
```

means the request was understood, but that user doesn't exist.

---

# 2. The main status code groups

You don't need to memorize every HTTP status code.

Focus on these:

| Code  | Meaning               | When to use                               |
| ----- | --------------------- | ----------------------------------------- |
| `200` | OK                    | Successful request                        |
| `201` | Created               | Resource successfully created             |
| `204` | No Content            | Successful request with no response body  |
| `400` | Bad Request           | Client sent invalid data                  |
| `401` | Unauthorized          | Authentication is required/invalid        |
| `403` | Forbidden             | Authenticated but not allowed             |
| `404` | Not Found             | Resource doesn't exist                    |
| `409` | Conflict              | Request conflicts with existing data      |
| `422` | Unprocessable Content | Data format is valid but validation fails |
| `500` | Internal Server Error | Server-side problem                       |

For now, **200, 201, 204, 400, 404, and 500** are the most important.

---

# 3. Returning a status code in Axum

Axum provides `StatusCode`.

```rust
use axum::http::StatusCode;

async fn hello() -> (StatusCode, &'static str) {
    (StatusCode::OK, "Hello World")
}
```

Response:

```text
HTTP/1.1 200 OK

Hello World
```

---

# 4. `200 OK`

Use `200` when the request succeeds.

For example, getting users:

```rust
use axum::{http::StatusCode, Json};
use serde::Serialize;

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
}

async fn get_user() -> (StatusCode, Json<User>) {
    (
        StatusCode::OK,
        Json(User {
            id: 1,
            name: "Alice".to_string(),
        }),
    )
}
```

The client receives:

```json
{
  "id": 1,
  "name": "Alice"
}
```

with:

```text
200 OK
```

**Typical use:**

- GET succeeded
- PUT succeeded
- PATCH succeeded

---

# 5. `201 Created`

Use `201` when you successfully create something.

For example:

```rust
use axum::{http::StatusCode, Json};
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
) -> (StatusCode, Json<User>) {
    (
        StatusCode::CREATED,
        Json(User {
            id: 1,
            name: input.name,
        }),
    )
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

**Remember:**

```text
POST → successfully creates resource → 201
```

---

# 6. `204 No Content`

Use `204` when the operation succeeded but there is **nothing to return**.

For example, deleting a user:

```rust
use axum::http::StatusCode;

async fn delete_user() -> StatusCode {
    StatusCode::NO_CONTENT
}
```

Response:

```text
204 No Content
```

There should be no response body.

Typical example:

```text
DELETE /users/1
        ↓
    204 No Content
```

---

# 7. `400 Bad Request`

Use `400` when the client sends an invalid request.

Example:

```rust
use axum::http::StatusCode;

async fn create_user() -> (StatusCode, &'static str) {
    (
        StatusCode::BAD_REQUEST,
        "Invalid request",
    )
}
```

Response:

```text
400 Bad Request
```

Example situations:

```text
Missing required data
Invalid request structure
Invalid parameter
```

---

# 8. `404 Not Found`

One of the most important API status codes.

Use it when the requested resource doesn't exist.

```rust
use axum::http::StatusCode;

async fn get_user() -> (StatusCode, &'static str) {
    (
        StatusCode::NOT_FOUND,
        "User not found",
    )
}
```

Response:

```text
404 Not Found
```

For example:

```text
GET /users/999
```

If user `999` doesn't exist:

```text
404 Not Found
```

---

# 9. `500 Internal Server Error`

This means something went wrong **on the server**.

```rust
use axum::http::StatusCode;

async fn something_failed() -> (StatusCode, &'static str) {
    (
        StatusCode::INTERNAL_SERVER_ERROR,
        "Internal server error",
    )
}
```

Response:

```text
500 Internal Server Error
```

For example, later when you have PostgreSQL:

```text
API
 ↓
Database query
 ↓
Database error
 ↓
500 Internal Server Error
```

Don't use `500` just because the user's request is invalid. `500` means **the server failed**.

---

# 10. Status code + JSON

In a real REST API, you'll commonly return a status code together with JSON.

```rust
use axum::{http::StatusCode, Json};
use serde::Serialize;

#[derive(Serialize)]
struct ErrorResponse {
    message: String,
}

async fn user_not_found() -> (StatusCode, Json<ErrorResponse>) {
    (
        StatusCode::NOT_FOUND,
        Json(ErrorResponse {
            message: "User not found".to_string(),
        }),
    )
}
```

Response:

```text
404 Not Found
```

```json
{
  "message": "User not found"
}
```

This is a pattern you should become comfortable with.

---

# 11. Common REST API pattern

You should understand this table:

```text
GET     /users       → 200 OK
GET     /users/1     → 200 OK
GET     /users/999   → 404 Not Found

POST    /users       → 201 Created

PUT     /users/1     → 200 OK

PATCH   /users/1     → 200 OK

DELETE  /users/1     → 204 No Content
```

And errors:

```text
Invalid request       → 400 Bad Request
Not authenticated     → 401 Unauthorized
Not allowed           → 403 Forbidden
Resource missing      → 404 Not Found
Duplicate/conflict    → 409 Conflict
Validation failed     → 422
Server failure        → 500 Internal Server Error
```

---

# 12. Complete small example

This brings **routes + handlers + JSON + status codes** together:

```rust
use axum::{
    http::StatusCode,
    routing::{get, post, delete},
    Json,
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

async fn get_user() -> (StatusCode, Json<User>) {
    (
        StatusCode::OK,
        Json(User {
            id: 1,
            name: "Alice".to_string(),
        }),
    )
}

async fn create_user(
    Json(input): Json<CreateUser>,
) -> (StatusCode, Json<User>) {
    (
        StatusCode::CREATED,
        Json(User {
            id: 2,
            name: input.name,
        }),
    )
}

async fn delete_user() -> StatusCode {
    StatusCode::NO_CONTENT
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/user", get(get_user))
        .route("/user", post(create_user))
        .route("/user", delete(delete_user));

    let listener =
        tokio::net::TcpListener::bind("127.0.0.1:3000")
            .await
            .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```

---

## 13. What you should know by the end of this Week's topics

You don't need every HTTP status code. Make sure you can:

- Understand what an HTTP status code represents
- Use `StatusCode` in Axum
- Return `200 OK`
- Return `201 Created`
- Return `204 No Content`
- Return `400 Bad Request`
- Return `404 Not Found`
- Understand `401`, `403`, `409`, `422`, and `500`
- Return **status code + JSON**
- Choose an appropriate status code for different API operations

### Mental model

```text
Client
  ↓
Request
  ↓
Route
  ↓
Handler
  ↓
Result
  ↓
Status Code + Response Body
```

The key thing to remember is:

> **The status code tells the client whether the request succeeded, failed because of the client, or failed because of the server.**
