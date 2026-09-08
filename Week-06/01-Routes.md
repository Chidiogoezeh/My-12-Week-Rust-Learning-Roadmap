For now, we don't need to learn every feature of Axum. For **Routes**, our goal is to understand how an HTTP request gets connected to a Rust function.

Think of a route as:

> **HTTP method + URL path → handler function**

For example:

```text
GET /users
     ↓
get_users()
```

## 1. What is a route?

A route defines what your API should do when a client requests a particular URL.

Example:

```rust
use axum::{
    routing::get,
    Router,
};

async fn hello() -> &'static str {
    "Hello, World!"
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(hello));

    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```

This route:

```rust
.route("/", get(hello))
```

means:

```text
GET /
   ↓
hello()
```

When you visit:

```text
http://127.0.0.1:3000/
```

you get:

```text
Hello, World!
```

---

# 2. HTTP methods

You need to understand the common HTTP methods because routes use them.

| Method | Purpose               | Example           |
| ------ | --------------------- | ----------------- |
| GET    | Retrieve data         | `GET /users`      |
| POST   | Create data           | `POST /users`     |
| PUT    | Replace/update data   | `PUT /users/1`    |
| PATCH  | Partially update data | `PATCH /users/1`  |
| DELETE | Delete data           | `DELETE /users/1` |

In Axum:

```rust
use axum::routing::{get, post, put, delete};
```

You then connect them to handlers:

```rust
Router::new()
    .route("/users", get(get_users))
    .route("/users", post(create_user))
    .route("/users/:id", put(update_user))
    .route("/users/:id", delete(delete_user));
```

So:

```text
GET    /users       → get_users()
POST   /users       → create_user()
PUT    /users/:id   → update_user()
DELETE /users/:id   → delete_user()
```

---

# 3. Multiple routes

For now, User API might look like this:

```rust
let app = Router::new()
    .route("/users", get(get_users))
    .route("/users", post(create_user))
    .route("/users/:id", get(get_user))
    .route("/users/:id", put(update_user))
    .route("/users/:id", delete(delete_user));
```

This is the basic CRUD routing pattern you should understand.

```text
GET    /users       → Get all users
GET    /users/1     → Get user 1
POST   /users       → Create user
PUT    /users/1     → Update user 1
DELETE /users/1     → Delete user 1
```

---

# 4. Route parameters

This is particularly important for backend development.

Consider:

```rust
.route("/users/:id", get(get_user))
```

`:id` is a **path parameter**.

A request such as:

```text
GET /users/25
```

means:

```text
id = 25
```

You can retrieve it using Axum's `Path` extractor:

```rust
use axum::extract::Path;

async fn get_user(Path(id): Path<u32>) -> String {
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

### Important idea

This:

```rust
"/users/:id"
```

is a pattern.

This:

```text
/users/25
```

is an actual request.

---

# 5. Multiple path parameters

You can have more than one.

```rust
.route("/users/:user_id/posts/:post_id", get(get_post))
```

Handler:

```rust
async fn get_post(
    Path((user_id, post_id)): Path<(u32, u32)>
) -> String {
    format!("User {} - Post {}", user_id, post_id)
}
```

Request:

```text
GET /users/10/posts/5
```

Result:

```text
User 10 - Post 5
```

You don't need to spend much time on complicated nested routes yet. Just understand the concept.

---

# 6. Route handlers

A **handler** is the function that executes when a route matches.

Example:

```rust
async fn get_users() -> &'static str {
    "List of users"
}
```

Then:

```rust
.route("/users", get(get_users))
```

The relationship is:

```text
Client
  ↓
GET /users
  ↓
Router
  ↓
get_users()
  ↓
Response
```

This distinction is important:

### Route

```rust
.route("/users", get(get_users))
```

### Handler

```rust
async fn get_users() -> &'static str {
    "List of users"
}
```

The **route determines when the function is called**.

The **handler determines what happens**.

---

# 7. Different handlers for the same path

You can have:

```rust
.route("/users", get(get_users))
.route("/users", post(create_user))
```

This is completely normal.

The URL is the same:

```text
/users
```

but the HTTP method determines the handler.

```text
GET /users
     ↓
get_users()

POST /users
     ↓
create_user()
```

That's why an API can have multiple operations on the same resource.

---

# 8. Route organization

As your application grows, you don't want everything in `main.rs`.

Eventually you'll have:

```text
src/
├── main.rs
├── routes.rs
└── handlers.rs
```

For example, `routes.rs`:

```rust
use axum::{
    routing::{get, post},
    Router,
};

use crate::handlers::{create_user, get_users};

pub fn user_routes() -> Router {
    Router::new()
        .route("/users", get(get_users))
        .route("/users", post(create_user))
}
```

And your `main.rs` can use it:

```rust
let app = user_routes();
```

**But don't over-engineer this for now.** Start with routes directly in `main.rs`, understand how they work, and then separate them when the number of routes starts growing.

---

# 9. Route nesting

You'll eventually want to group related routes.

For example:

```text
/api/users
/api/users/1
/api/products
/api/products/1
```

You can create an API router:

```rust
let api = Router::new()
    .route("/users", get(get_users))
    .route("/products", get(get_products));

let app = Router::new()
    .nest("/api", api);
```

Now:

```text
GET /api/users
GET /api/products
```

reach the appropriate handlers.

Again, **understand this concept but don't make it complicated yet.**

---

# 10. Current target

By the end of the Routes section, you should be comfortable writing something like:

```rust
use axum::{
    routing::{get, post, put, delete},
    Router,
};

async fn get_users() -> &'static str {
    "Get users"
}

async fn get_user() -> &'static str {
    "Get one user"
}

async fn create_user() -> &'static str {
    "Create user"
}

async fn update_user() -> &'static str {
    "Update user"
}

async fn delete_user() -> &'static str {
    "Delete user"
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/users", get(get_users).post(create_user))
        .route("/users/:id", get(get_user).put(update_user).delete(delete_user));

    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```

The important thing isn't memorizing the syntax. You should be able to look at this:

```rust
.route("/users", get(get_users).post(create_user))
.route("/users/:id", get(get_user).put(update_user).delete(delete_user))
```

and understand:

```text
GET    /users       → get_users
POST   /users       → create_user

GET    /users/:id   → get_user
PUT    /users/:id   → update_user
DELETE /users/:id   → delete_user
```

### Your Routes checklist

For now, make sure you understand these **7 things**:

- [ ] What an HTTP route is
- [ ] GET, POST, PUT, PATCH, DELETE
- [ ] Connecting routes to handlers
- [ ] Route parameters (`:id`)
- [ ] Extracting parameters with `Path`
- [ ] Multiple routes for the same resource
- [ ] Basic route organization/nesting

Once you understand those, you're ready for the next topic: **Handlers**. That's where you'll start taking the request, processing it, and returning proper API responses.
