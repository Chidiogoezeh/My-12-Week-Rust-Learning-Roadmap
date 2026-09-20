For **Authentication**, you should understand registration as:

> **Receive user details → validate → check email → hash password → save user → return safe response**

## 1. Registration request

Client sends:

```json
{
  "email": "chidi@example.com",
  "password": "mypassword123"
}
```

Rust request model:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
pub struct RegisterRequest {
    pub email: String,
    pub password: String,
}
```

---

## 2. Validate the input

At minimum, understand that you should validate:

- Email is present and properly formatted.
- Password meets your minimum requirements.
- Email is normalized consistently, usually lowercase.

Simple beginner example:

```rust
if request.email.trim().is_empty() {
    return Err(RegisterError::InvalidInput);
}

if request.password.len() < 8 {
    return Err(RegisterError::InvalidInput);
}
```

Later, you can use the `validator` crate for proper validation.

---

## 3. Check whether the email already exists

Before creating the user:

```rust
let existing_user = sqlx::query!(
    r#"
    SELECT id
    FROM users
    WHERE email = $1
    "#,
    request.email
)
.fetch_optional(&pool)
.await?;

if existing_user.is_some() {
    return Err(RegisterError::EmailAlreadyExists);
}
```

You should also have a **UNIQUE constraint** on the database column:

```sql
email TEXT UNIQUE NOT NULL
```

The database constraint is important because application-level checks alone can have race conditions.

---

## 4. Hash the password

**Never store the original password.**

Use the `hash_password()` function you learned earlier:

```rust
let password_hash = hash_password(&request.password)?;
```

The database stores something like:

```text
$argon2id$v=19$m=19456,t=2,p=1$...
```

Not:

```text
mypassword123
```

---

## 5. Insert the user

Example:

```rust
let user = sqlx::query!(
    r#"
    INSERT INTO users (email, password_hash)
    VALUES ($1, $2)
    RETURNING id, email
    "#,
    request.email,
    password_hash
)
.fetch_one(&pool)
.await?;
```

Notice that the original password is **not** inserted.

---

## 6. Return a safe response

You can return:

```rust
#[derive(Serialize)]
pub struct RegisterResponse {
    pub id: i64,
    pub email: String,
}
```

Then:

```rust
Json(RegisterResponse {
    id: user.id,
    email: user.email,
})
```

Do **not** return:

```rust
password
password_hash
```

---

# 7. Complete registration flow

The important sequence is:

```text
POST /register
      ↓
Receive email + password
      ↓
Validate input
      ↓
Check email
      ↓
Hash password with Argon2
      ↓
Save email + password_hash
      ↓
Return user information
```

Example handler structure:

```rust
pub async fn register(
    State(pool): State<PgPool>,
    Json(request): Json<RegisterRequest>,
) -> Result<Json<RegisterResponse>, RegisterError> {

    // 1. Validate
    if request.password.len() < 8 {
        return Err(RegisterError::InvalidInput);
    }

    // 2. Check existing email
    let existing = sqlx::query!(
        "SELECT id FROM users WHERE email = $1",
        request.email
    )
    .fetch_optional(&pool)
    .await?;

    if existing.is_some() {
        return Err(RegisterError::EmailAlreadyExists);
    }

    // 3. Hash password
    let password_hash = hash_password(&request.password)?;

    // 4. Save user
    let user = sqlx::query!(
        r#"
        INSERT INTO users (email, password_hash)
        VALUES ($1, $2)
        RETURNING id, email
        "#,
        request.email,
        password_hash
    )
    .fetch_one(&pool)
    .await?;

    // 5. Return safe response
    Ok(Json(RegisterResponse {
        id: user.id,
        email: user.email,
    }))
}
```

You don't need to memorize this entire handler yet. Understand the **five operations**.

---

## 8. Registration vs Login

This distinction is important for now:

| Registration     | Login                |
| ---------------- | -------------------- |
| Create user      | Authenticate user    |
| Validate input   | Validate input       |
| Check email      | Find email           |
| Hash password    | Verify password hash |
| Store user       | Create JWT           |
| Return user info | Return JWT           |

The overall authentication flow becomes:

```text
REGISTER
email + password
      ↓
Argon2 hash
      ↓
Database


LOGIN
email + password
      ↓
Find user
      ↓
Verify Argon2 hash
      ↓
Create JWT
      ↓
Return JWT
```

### What you should know for now

You should be able to explain and implement:

1. `POST /register`
2. `RegisterRequest`
3. Input validation
4. Checking for an existing email
5. **Argon2 password hashing**
6. PostgreSQL `INSERT`
7. Unique email constraint
8. Never storing plaintext passwords
9. Never returning passwords/password hashes
10. Returning a safe registration response

**You do not need to add email verification, password reset, refresh tokens, roles/RBAC, or rate limiting yet.** Those can come later after the basic registration + login flow is solid.
