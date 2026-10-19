# HTTP Middleware in Rust

For now, you should understand **what middleware is, why it exists, how it works in Axum, and how to use it for common backend concerns**.

You do not need to learn advanced middleware architecture yet.

## 1. What is middleware?

Middleware is code that runs **around your HTTP request handlers**.

Instead of putting the same logic inside every handler, middleware handles it centrally.

For example:

```text
Request
   |
   v
Middleware
   |
   v
Handler
   |
   v
Response
   |
   v
Middleware
   |
   v
Client
```

Common middleware responsibilities:

- Logging requests
- Authentication
- Authorization
- CORS
- Request IDs
- Rate limiting
- Timeouts
- Adding response headers

---

# 2. Why use middleware?

Without middleware:

```rust
async fn get_users() {
    log_request();
    check_auth();
    // actual logic
}

async fn get_products() {
    log_request();
    check_auth();
    // actual logic
}
```

You repeat the same code.

With middleware:

```text
Request
   |
   v
Authentication middleware
   |
   v
Logging middleware
   |
   v
Handler
```

The handlers can focus on their actual business logic.

---

# 3. Axum middleware

Axum uses middleware through Tower/Tower HTTP.

A simple example:

```rust
use axum::{
    routing::get,
    Router,
};
use tower_http::trace::TraceLayer;

async fn hello() -> &'static str {
    "Hello"
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(hello))
        .layer(TraceLayer::new_for_http());

    // start server...
}
```

`TraceLayer` is middleware that provides HTTP request/response tracing.

---

# 4. Middleware runs before and after the handler

Conceptually:

```text
Request
   |
   v
Middleware starts
   |
   v
Handler runs
   |
   v
Middleware continues
   |
   v
Response
```

This makes middleware useful for things such as measuring request duration.

For example:

```text
Request received
      |
      v
Start timer
      |
      v
Handler
      |
      v
Stop timer
      |
      v
Log duration
      |
      v
Response
```

---

# 5. Authentication middleware

One of the most important backend uses is authentication.

Suppose you have:

```text
GET /profile
```

You don't want every handler manually checking the JWT.

Instead:

```text
Request
   |
   v
JWT Middleware
   |
   | valid
   v
Profile Handler
```

If the token is invalid:

```text
Request
   |
   v
JWT Middleware
   |
   | invalid
   v
401 Unauthorized
```

---

# 6. Simple custom middleware

Axum provides `middleware::from_fn`.

Example:

```rust
use axum::{
    middleware,
    extract::Request,
    response::Response,
    Router,
};

async fn auth_middleware(
    request: Request,
    next: middleware::Next,
) -> Response {
    println!("Checking authentication");

    let response = next.run(request).await;

    response
}
```

Then attach it:

```rust
let app = Router::new()
    .route("/profile", get(profile))
    .layer(middleware::from_fn(auth_middleware));
```

Now requests to `/profile` pass through:

```text
Request
   |
   v
auth_middleware
   |
   v
/profile handler
```

---

# 7. Middleware can stop the request

Middleware doesn't have to call the handler.

For example:

```rust
use axum::{
    http::StatusCode,
    middleware,
    extract::Request,
    response::{Response, IntoResponse},
};

async fn auth_middleware(
    request: Request,
    next: middleware::Next,
) -> Response {

    let authenticated = false;

    if !authenticated {
        return StatusCode::UNAUTHORIZED.into_response();
    }

    next.run(request).await
}
```

If authentication fails:

```text
Request
   |
   v
Auth middleware
   |
   | failed
   v
401
```

The handler is never executed.

---

# 8. Reading an Authorization header

A common authentication pattern is:

```text
Authorization: Bearer <JWT>
```

Middleware can inspect it:

```rust
async fn auth_middleware(
    request: Request,
    next: middleware::Next,
) -> Response {

    let auth_header = request
        .headers()
        .get("Authorization");

    if auth_header.is_none() {
        return StatusCode::UNAUTHORIZED.into_response();
    }

    next.run(request).await
}
```

This example only checks that the header exists.

A real authentication middleware would:

1. Read the header.
2. Extract the Bearer token.
3. Verify the JWT.
4. Extract the user ID.
5. Put the authenticated user information into the request.
6. Continue to the handler.

---

# 9. Middleware can pass information to handlers

This is important for authentication.

Suppose JWT contains:

```text
user_id = 42
```

Middleware verifies it and wants the handler to know:

```text
Authenticated user = 42
```

Axum allows request extensions for this.

Conceptually:

```rust
request.extensions_mut().insert(user_id);
```

Then the handler can extract it.

For example:

```rust
use axum::extract::Extension;

async fn profile(
    Extension(user_id): Extension<i32>,
) -> String {
    format!("User ID: {}", user_id)
}
```

The flow becomes:

```text
JWT
 |
 v
Middleware
 |
 | verifies token
 |
 | inserts user_id = 42
 v
Handler
 |
 v
Extension(user_id)
```

You do not need to build a complete JWT middleware during this lesson. Just understand this pattern because you will use it in Week 8/9 authentication work.

---

# 10. Global vs route-specific middleware

This distinction is important.

### Global middleware

Applies to the whole application:

```rust
let app = Router::new()
    .route("/", get(home))
    .route("/users", get(users))
    .layer(TraceLayer::new_for_http());
```

Every route gets the middleware.

Useful for:

- Request logging
- Tracing
- CORS
- General headers

### Route-specific middleware

Apply middleware only to protected routes.

Conceptually:

```rust
let protected_routes = Router::new()
    .route("/profile", get(profile))
    .route("/settings", get(settings))
    .layer(middleware::from_fn(auth_middleware));

let app = Router::new()
    .route("/", get(home))
    .merge(protected_routes);
```

Now:

```text
GET /
    |
    v
Home handler


GET /profile
    |
    v
Auth middleware
    |
    v
Profile handler
```

This is a very common pattern.

---

# 11. Middleware order matters

Suppose you have:

```rust
.layer(logging)
.layer(auth)
```

Middleware forms a chain.

Conceptually:

```text
Request
   |
   v
Logging
   |
   v
Auth
   |
   v
Handler
```

The exact execution order can depend on how layers are composed, so don't blindly assume the order from visual code order. When middleware order matters, verify the Axum/Tower composition you're using.

The important concept is:

> Middleware forms a processing chain.

---

# 12. Middleware vs handler

A useful rule:

### Middleware

Use for concerns that apply across multiple endpoints.

Examples:

```text
Authentication
Logging
CORS
Request ID
Rate limiting
Timeouts
```

### Handler

Use for endpoint-specific work.

Examples:

```text
Create user
Get user
Update profile
Create task
Delete task
```

Don't put your actual business logic into middleware.

---

# 13. Middleware + structured logging

This connects directly to the previous topic.

For example:

```rust
use tracing::info;

async fn logging_middleware(
    request: Request,
    next: middleware::Next,
) -> Response {

    info!(
        method = %request.method(),
        uri = %request.uri(),
        "Incoming request"
    );

    let response = next.run(request).await;

    info!(
        status = %response.status(),
        "Request completed"
    );

    response
}
```

Now you can see information such as:

```text
Incoming request method=GET uri=/users
Request completed status=200
```

This is much more useful than scattered `println!` calls.

---

# 14. Middleware + error handling

Middleware can also return errors/status codes before the handler runs.

For example:

```rust
if !authenticated {
    return StatusCode::UNAUTHORIZED.into_response();
}
```

Your overall application can therefore look like:

```text
Request
   |
   v
Logging middleware
   |
   v
Authentication middleware
   |
   | failure
   +------> 401
   |
   | success
   v
Handler
   |
   v
Service
   |
   v
Repository
   |
   v
Database
```

This connects the three Week's topics:

```text
Validation
Error handling
Middleware
```

---

# 15. What you should know about `Next`

This:

```rust
next.run(request).await
```

means:

> Pass the request to the next layer/handler.

If you don't call it, the request stops at the current middleware.

For example:

```rust
async fn middleware(
    request: Request,
    next: middleware::Next,
) -> Response {

    // Before handler

    let response = next.run(request).await;

    // After handler

    response
}
```

This is the basic middleware pattern you should remember.

---

# HTTP Middleware Checklist

You should be able to explain and use:

- What middleware is
- Why middleware exists
- Request → middleware → handler → response
- `middleware::from_fn`
- `middleware::Next`
- `next.run(request).await`
- Returning a response without calling the handler
- Global middleware
- Route-specific middleware
- Authentication middleware
- Reading request headers
- Passing data through request extensions
- `TraceLayer`
- Middleware and structured logging
- Middleware and error handling
- Middleware ordering
- Middleware vs handler responsibilities

### The core pattern

Remember this:

```rust
async fn middleware(
    request: Request,
    next: middleware::Next,
) -> Response {

    // Before handler

    let response = next.run(request).await;

    // After handler

    response
}
```

And the core mental model:

```text
Request
   ↓
Middleware
   ↓
Middleware
   ↓
Handler
   ↓
Response
```

For begginner level, **authentication middleware + request logging + route-specific middleware** are the three areas you should be able to implement confidently.
