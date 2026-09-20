For now, you don't need to learn every JWT detail. You need to understand **what a JWT is, how to create one after login, how to verify it, and how to use it to protect Axum routes**.

# JWT Authentication

## 1. What is a JWT?

JWT stands for **JSON Web Token**.

After a user successfully logs in:

```text
email + password
       ↓
verify password
       ↓
   successful
       ↓
 create JWT
       ↓
send JWT to client
```

The client then sends the JWT when accessing protected endpoints:

```http
Authorization: Bearer <token>
```

Your server verifies the token before allowing access.

---

# 2. What does a JWT look like?

A JWT looks roughly like this:

```text
xxxxx.yyyyy.zzzzz
```

It has three parts:

```text
HEADER.PAYLOAD.SIGNATURE
```

For example:

```text
eyJhbGciOiJIUzI1NiJ9
.
eyJzdWIiOiIxMjMiLCJleHAiOjE3...
.
signature...
```

You don't need to memorize the encoded strings.

Understand the three parts:

### Header

Describes the token.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload

Contains claims.

```json
{
  "sub": "123",
  "exp": 1790000000
}
```

### Signature

Allows your server to detect whether the token has been modified.

---

# 3. Important: JWT is not encryption

This is very important.

A normal JWT payload is **encoded, not encrypted**.

Therefore, don't put sensitive information inside it.

Bad:

```json
{
  "password": "mypassword123"
}
```

Good:

```json
{
  "sub": "123",
  "exp": 1790000000
}
```

Usually:

- `sub` → user ID
- `exp` → expiration time
- `iat` → issued-at time

---

# 4. Install the JWT crate

For your project:

```bash
cargo add jsonwebtoken
cargo add serde --features derive
cargo add chrono
```

You'll use:

```rust
use jsonwebtoken::{
    encode,
    decode,
    EncodingKey,
    DecodingKey,
    Header,
    Validation,
};
```

---

# 5. Create JWT claims

Create:

```text
src/auth.rs
```

Start with:

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct Claims {
    pub sub: String,
    pub exp: usize,
}
```

`sub` identifies the user.

`exp` tells you when the token expires.

---

# 6. Create a JWT

For learning, we'll use an environment variable for the secret.

```rust
use jsonwebtoken::{
    encode,
    EncodingKey,
    Header,
};

pub fn create_token(user_id: &str, secret: &str) -> Result<String, jsonwebtoken::errors::Error> {
    let claims = Claims {
        sub: user_id.to_string(),
        exp: 1_800_000_000,
    };

    encode(
        &Header::default(),
        &claims,
        &EncodingKey::from_secret(secret.as_bytes()),
    )
}
```

Then:

```rust
fn main() {
    let token = create_token("123", "my-secret-key").unwrap();

    println!("{}", token);
}
```

You'll get something like:

```text
eyJhbGciOiJIUzI1NiJ9...
```

---

# 7. Don't hard-code the secret

This:

```rust
let secret = "my-secret-key";
```

is only for learning.

Don't put a real JWT secret directly into your source code.

Instead use:

```text
JWT_SECRET=some-long-random-secret
```

in your `.env`:

```text
DATABASE_URL=postgres://...
JWT_SECRET=your-long-random-secret
```

Then load it with `dotenvy`.

```bash
cargo add dotenvy
```

Later:

```rust
dotenvy::dotenv().ok();

let secret = std::env::var("JWT_SECRET")
    .expect("JWT_SECRET must be set");
```

And **never commit `.env` to Git**.

Put this in `.gitignore`:

```text
.env
```

---

# 8. Verify a JWT

Creating the token isn't enough.

When the client sends the token back, your server needs to verify it.

```rust
use jsonwebtoken::{
    decode,
    DecodingKey,
    Validation,
};

pub fn verify_token(
    token: &str,
    secret: &str,
) -> Result<Claims, jsonwebtoken::errors::Error> {
    let token_data = decode::<Claims>(
        token,
        &DecodingKey::from_secret(secret.as_bytes()),
        &Validation::default(),
    )?;

    Ok(token_data.claims)
}
```

Usage:

```rust
let claims = verify_token(token, secret)?;

println!("User ID: {}", claims.sub);
```

If the token is invalid or expired, verification fails.

---

# 9. The complete authentication flow

This is the most important thing to understand.

### Registration

```text
POST /register
       ↓
email + password
       ↓
hash password with Argon2
       ↓
store user + password_hash
```

### Login

```text
POST /login
       ↓
email + password
       ↓
find user
       ↓
verify password with Argon2
       ↓
correct?
   ↓       ↓
 yes       no
 ↓         ↓
create     reject
JWT        login
 ↓
return JWT
```

### Access protected route

```text
GET /profile
Authorization: Bearer <JWT>
       ↓
extract token
       ↓
verify JWT
       ↓
valid?
 ↓       ↓
yes      no
 ↓        ↓
handler   401
```

---

# 10. JWT with Axum

This is where JWT becomes useful for backend development.

Suppose you have:

```text
GET /profile
```

You want only authenticated users to access it.

A simplified handler could eventually look like:

```rust
use axum::http::HeaderMap;

async fn profile(headers: HeaderMap) -> &'static str {
    let auth = headers
        .get("Authorization")
        .and_then(|value| value.to_str().ok());

    match auth {
        Some(value) => {
            println!("{}", value);
            "Profile"
        }
        None => "Missing token",
    }
}
```

But **don't stop here**. A production API should extract and verify the Bearer token in middleware/extractors rather than repeating this logic in every handler.

---

# 11. Bearer token

The client normally sends:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

There are two pieces:

```text
Authorization:
```

and:

```text
Bearer <JWT>
```

Your server extracts the token after `Bearer`.

Conceptually:

```text
"Bearer abc123"
       ↓
    remove
    "Bearer "
       ↓
    "abc123"
       ↓
   verify JWT
```

---

# 12. JWT middleware

The goal is to have something like:

```text
Request
   ↓
JWT middleware
   ↓
verify token
   ↓
valid?
 ┌───┴───┐
 ↓       ↓
Yes      No
 ↓       ↓
handler  401
```

For example:

```rust
async fn protected_route() -> &'static str {
    "You are authenticated"
}
```

The JWT middleware sits **before** this handler.

That means your handler doesn't need to worry about parsing JWTs.

---

# 13. What should the JWT contain?

For your project, keep it simple:

```rust
#[derive(Debug, Serialize, Deserialize)]
pub struct Claims {
    pub sub: String,
    pub exp: usize,
}
```

You could have:

```json
{
  "sub": "42",
  "exp": 1790000000
}
```

Where:

```text
sub = user ID
exp = expiration time
```

That's enough for your first authentication system.

---

# 14. JWT secret

The server uses the same secret to:

### Create

```text
claims + secret
      ↓
    JWT
```

### Verify

```text
JWT + secret
      ↓
valid / invalid
```

Therefore:

**Keep the secret private.**

Never put it in:

- GitHub
- frontend JavaScript
- `Cargo.toml`
- README
- source code

---

# 15. What you should know by the end of the week

You should be able to explain:

### 1. What JWT is

A signed token used to carry authentication claims between a client and server.

### 2. JWT structure

```text
Header.Payload.Signature
```

### 3. JWT isn't encryption

Don't put passwords or sensitive information in the payload.

### 4. Creating a token

```rust
encode(...)
```

### 5. Verifying a token

```rust
decode(...)
```

### 6. Claims

At minimum:

```text
sub
exp
```

### 7. Bearer authentication

```http
Authorization: Bearer <JWT>
```

### 8. Protected routes

```text
Request
 ↓
JWT verification
 ↓
valid?
 ↓
handler
```

### 9. Secret management

Use:

```text
JWT_SECRET
```

from environment variables.

### 10. How JWT connects with password hashing

This is the key relationship:

```text
              REGISTER
                 │
                 ▼
            Argon2id hash
                 │
                 ▼
             PostgreSQL
                 │
                 │
              LOGIN
                 │
                 ▼
        Verify Argon2id hash
                 │
                 ▼
          Password correct?
                 │
                YES
                 │
                 ▼
             Create JWT
                 │
                 ▼
              Client
                 │
                 │ Authorization: Bearer JWT
                 ▼
            Protected API
                 │
                 ▼
            Verify JWT
                 │
                 ▼
              Handler
```

**That's the core of JWT authentication you need for now.** Once you understand this flow, the next practical step is to combine **Argon2 + JWT + Axum + PostgreSQL** into your authentication API.
