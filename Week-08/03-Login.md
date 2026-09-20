For now, you should understand exactly what happens from the moment the user submits their email/password until the server returns a JWT.

# Login

## 1. What is login?

A login request typically contains:

```json
{
  "email": "chidi@example.com",
  "password": "mypassword123"
}
```

The server then:

```text
Email + password
       ↓
Find user in database
       ↓
Get stored password_hash
       ↓
Verify password
       ↓
Correct?
   ↓        ↓
 YES        NO
  ↓          ↓
Create JWT   Reject
  ↓
Return JWT
```

That's the entire concept.

---

# 2. Login request model

With Serde:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
pub struct LoginRequest {
    pub email: String,
    pub password: String,
}
```

Axum can deserialize incoming JSON into this struct.

For example:

```http
POST /login
Content-Type: application/json
```

```json
{
  "email": "chidi@example.com",
  "password": "mypassword123"
}
```

---

# 3. Find the user

Your database contains something like:

```text
users
--------------------------------
id
email
password_hash
```

The important point is:

**You search by email, not by password.**

With SQLx:

```rust
let user = sqlx::query_as!(
    User,
    r#"
        SELECT id, email, password_hash
        FROM users
        WHERE email = $1
    "#,
    request.email
)
.fetch_optional(&pool)
.await?;
```

`fetch_optional()` is useful because the email might not exist.

---

# 4. What happens if the user doesn't exist?

Don't return something overly specific like:

```text
Email doesn't exist
```

A login endpoint should generally return the same generic authentication failure for both:

- unknown email
- incorrect password

For example:

```text
Invalid credentials
```

This helps avoid revealing which email addresses have accounts.

Conceptually:

```rust
let user = match user {
    Some(user) => user,
    None => return Err(LoginError::InvalidCredentials),
};
```

---

# 5. Verify the password

Remember that the database contains:

```text
password_hash
```

not:

```text
password
```

Use your password verification function:

```rust
let valid = verify_password(
    &request.password,
    &user.password_hash,
)?;
```

Then:

```rust
if !valid {
    return Err(LoginError::InvalidCredentials);
}
```

You **do not decrypt the hash**.

You also shouldn't manually hash the login password and compare the resulting strings. Let Argon2's verifier handle the stored salt and parameters.

---

# 6. Create the JWT

Once the password is correct:

```rust
let token = create_token(
    &user.id.to_string(),
    &jwt_secret,
)?;
```

Now you have:

```text
User verified
     ↓
Create JWT
     ↓
Return token
```

---

# 7. Login response

A simple response can be:

```rust
use serde::Serialize;

#[derive(Serialize)]
pub struct LoginResponse {
    pub token: String,
}
```

Then:

```rust
Json(LoginResponse { token })
```

The client receives:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

The client can then use that token for protected requests.

---

# 8. Complete login handler

Here's the basic structure you'll want to understand:

```rust
use axum::{extract::State, Json};
use serde::{Deserialize, Serialize};
use sqlx::PgPool;

#[derive(Deserialize)]
pub struct LoginRequest {
    pub email: String,
    pub password: String,
}

#[derive(Serialize)]
pub struct LoginResponse {
    pub token: String,
}

pub async fn login(
    State(pool): State<PgPool>,
    Json(request): Json<LoginRequest>,
) -> Result<Json<LoginResponse>, LoginError> {

    let user = sqlx::query_as!(
        User,
        r#"
            SELECT id, email, password_hash
            FROM users
            WHERE email = $1
        "#,
        request.email
    )
    .fetch_optional(&pool)
    .await
    .map_err(|_| LoginError::Database)?;

    let user = match user {
        Some(user) => user,
        None => return Err(LoginError::InvalidCredentials),
    };

    let valid = verify_password(
        &request.password,
        &user.password_hash,
    )
    .map_err(|_| LoginError::InvalidCredentials)?;

    if !valid {
        return Err(LoginError::InvalidCredentials);
    }

    let token = create_token(
        &user.id.to_string(),
        &std::env::var("JWT_SECRET")
            .expect("JWT_SECRET must be set"),
    )
    .map_err(|_| LoginError::Token)?;

    Ok(Json(LoginResponse { token }))
}
```

You don't need to memorize this. Understand the sequence.

---

# 9. The complete login flow

This is what you should be able to draw from memory:

```text
POST /login
     │
     ▼
{ email, password }
     │
     ▼
Find user by email
     │
     ├── Not found ──────► Invalid credentials
     │
     ▼
Get password_hash
     │
     ▼
Verify password with Argon2
     │
     ├── Wrong ──────────► Invalid credentials
     │
     ▼
Create JWT
     │
     ▼
Return JWT
```

---

# 10. What should NOT happen

### Don't store the login password

```rust
println!("{}", request.password);
```

Don't log passwords.

### Don't return the password

```json
{
  "email": "...",
  "password": "..."
}
```

### Don't put the password in the JWT

```json
{
  "sub": "123",
  "password": "..."
}
```

### Don't return different errors

Avoid:

```text
Email not found
```

versus:

```text
Wrong password
```

Use:

```text
Invalid credentials
```

### Don't create a JWT before verifying the password

The order matters:

```text
Verify password
      ↓
Create JWT
```

not:

```text
Create JWT
      ↓
Verify password
```

---

# 11. How login connects everything you've learned

You now have three separate pieces:

### Password hashing

Registration:

```text
password
   ↓
Argon2
   ↓
password_hash
   ↓
PostgreSQL
```

### Login

```text
email + password
       ↓
PostgreSQL
       ↓
password_hash
       ↓
Argon2 verification
```

### JWT

After successful verification:

```text
user_id
   ↓
JWT
   ↓
client
```

Then future requests:

```text
JWT
 ↓
verify
 ↓
protected endpoint
```

---

# 12. Your Week login exercise

Before moving on, build this endpoint:

```text
POST /login
```

### Request

```json
{
  "email": "chidi@example.com",
  "password": "password123"
}
```

### Successful response

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

### Failed response

```json
{
  "error": "Invalid credentials"
}
```

Your implementation should perform exactly these four operations:

```text
1. Find user by email
2. Verify password with Argon2
3. Create JWT
4. Return JWT
```

If you can implement and explain those four steps, you understand **login** at the level you need for now.
