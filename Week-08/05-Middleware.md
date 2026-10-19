For **Authentication**, you should understand middleware mainly as the layer that **protects routes by checking authentication before the handler runs**.

## 1. What is middleware?

Middleware is code that runs **between the incoming request and your route handler**.

Without middleware:

```text
Request → Handler → Response
```

With authentication middleware:

```text
Request
   ↓
Auth Middleware
   ↓
Verify JWT
   ↓
Handler
   ↓
Response
```

If the JWT is invalid:

```text
Request
   ↓
Auth Middleware
   ↓
Invalid JWT
   ↓
401 Unauthorized
```

The handler never runs.

---

# 2. Why use middleware?

Instead of doing this in every protected handler:

```rust
// Extract token
// Verify token
// Check user
// Then perform handler logic
```

you do it once in middleware.

For example:

```text
POST /register    → public
POST /login       → public

GET /profile      → auth middleware
GET /tasks        → auth middleware
POST /tasks       → auth middleware
DELETE /tasks/:id → auth middleware
```

---

# 3. What authentication middleware does

For your project, understand this flow:

```text
Request
   ↓
Read Authorization header
   ↓
Extract Bearer token
   ↓
Verify JWT
   ↓
Get user ID from `sub`
   ↓
Allow request
```

If anything fails:

```text
401 Unauthorized
```

---

# 4. Authorization header

The client sends:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

The middleware needs to extract:

```text
eyJhbGciOiJIUzI1NiIs...
```

and verify it using your JWT secret.

---

# 5. Simple Axum middleware example

Install Axum if you haven't already:

```bash
cargo add axum
```

A simplified middleware can look like this:

```rust
use axum::{
    http::{Request, StatusCode},
    middleware::Next,
    response::Response,
};

pub async fn auth_middleware(
    request: Request<axum::body::Body>,
    next: Next,
) -> Result<Response, StatusCode> {
    let auth_header = request
        .headers()
        .get("Authorization")
        .and_then(|value| value.to_str().ok());

    let Some(auth_header) = auth_header else {
        return Err(StatusCode::UNAUTHORIZED);
    };

    let Some(token) = auth_header.strip_prefix("Bearer ") else {
        return Err(StatusCode::UNAUTHORIZED);
    };

    // Verify JWT here
    let _claims = verify_token(token, "your-secret")
        .map_err(|_| StatusCode::UNAUTHORIZED)?;

    Ok(next.run(request).await)
}
```

The important part is:

```rust
Ok(next.run(request).await)
```

That means:

> "Authentication succeeded. Continue to the actual handler."

---

# 6. Protect routes

With Axum:

```rust
use axum::{
    middleware,
    routing::get,
    Router,
};

let protected_routes = Router::new()
    .route("/profile", get(profile))
    .route("/tasks", get(get_tasks))
    .layer(middleware::from_fn(auth_middleware));
```

Now:

```text
GET /profile
      ↓
auth_middleware
      ↓
profile handler
```

and:

```text
GET /tasks
      ↓
auth_middleware
      ↓
get_tasks handler
```

Both routes require a valid JWT.

---

# 7. Public vs protected routes

A typical application:

```rust
let app = Router::new()
    .route("/register", post(register))
    .route("/login", post(login))
    .merge(protected_routes);
```

So:

```text
/register → public
/login    → public

/profile  → protected
/tasks    → protected
```

You **do not** want authentication middleware on `/register` or `/login`, because users don't have a token yet.

---

# 8. Passing the user ID to the handler

This is an important concept.

Your JWT contains:

```rust
pub struct Claims {
    pub sub: String,
    pub exp: usize,
}
```

Suppose:

```text
sub = "42"
```

The middleware verifies the JWT and knows:

```text
Authenticated user = 42
```

In a production-style Axum application, you normally pass that authenticated identity to the handler using an **extractor/request extensions**, rather than making the handler decode the JWT again.

Conceptually:

```text
Request
   ↓
Middleware
   ↓
JWT verified
   ↓
user_id = 42
   ↓
Handler receives user_id
```

For example, the middleware can insert it into request extensions:

```rust
use axum::http::Request;

request.extensions_mut().insert(user_id);
```

Then a handler can retrieve it:

```rust
let user_id = request.extensions().get::<i64>();
```

In Axum, a custom extractor is usually a cleaner production approach, but you don't need to master that yet.

---

# 9. Authentication vs authorization

Know the difference:

**Authentication:**

> "Who are you?"

JWT verification answers this.

**Authorization:**

> "Are you allowed to do this?"

For example:

```text
Authenticated user → Can access /profile
Authenticated user → Can create their own task
Admin             → Can delete another user's account
```

For now, focus primarily on **authentication middleware**.

RBAC/permissions can come later.

---

# 10. What you should know for now

You should be able to explain:

1. What middleware is.
2. Why authentication belongs in middleware.
3. How to read the `Authorization` header.
4. What `Bearer <token>` means.
5. How to extract the JWT.
6. How to verify the JWT.
7. How to return `401 Unauthorized`.
8. How `next.run(request)` continues the request.
9. How to attach middleware to protected Axum routes.
10. How to get the authenticated user's ID from the JWT.

The core architecture you should understand is:

```text
                    ┌── /register → handler
Request → Router ───┤
                    ├── /login    → handler
                    │
                    └── /profile
                          ↓
                    Auth Middleware
                          ↓
                     Verify JWT
                          ↓
                    Profile Handler
```

**That is enough middleware knowledge for now.** Don't dive into complex middleware stacks, custom extractors, RBAC, refresh tokens, or advanced request extensions until your basic JWT-protected route works.
