For now, we don't need to learn every detail of cryptography. For password hashing, we should understand **what it is, why it is necessary, and how to implement it correctly in Rust**.

# Password Hashing

## 1. What is password hashing?

When a user registers, **never store their actual password** in your database.

Bad:

```text
email: chidi@example.com
password: mypassword123
```

Instead, you hash the password:

```text
email: chidi@example.com
password_hash: $argon2id$v=19$m=19456,t=2,p=1$...
```

A hash is designed to be **one-way**.

You can do:

```text
password → hash
```

but you should not be able to simply do:

```text
hash → password
```

The database therefore stores the **hash**, not the original password.

---

# 2. Why not use SHA-256?

Don't do this:

```rust
use sha2::{Digest, Sha256};

let hash = Sha256::digest(password.as_bytes());
```

SHA-256 is designed to be fast.

That's useful for many cryptographic purposes, but **bad for password storage** because attackers can try enormous numbers of passwords very quickly.

Password hashing algorithms are deliberately expensive.

For this project, use **Argon2id**.

---

# 3. What is Argon2?

**Argon2** is a password-hashing algorithm designed specifically for protecting passwords.

You'll commonly see:

```text
Argon2id
```

For your Week's project, the important idea is simply:

```text
User password
      ↓
    Argon2id
      ↓
Password hash
      ↓
   Database
```

---

# 4. Install Argon2

Inside your Rust project:

```bash
cargo add argon2
cargo add password-hash
cargo add rand
```

---

# 5. Hashing a password

Create:

```text
src/password.rs
```

Then:

```rust
use argon2::{
    password_hash::{
        rand_core::OsRng,
        PasswordHash,
        PasswordHasher,
        PasswordVerifier,
        SaltString,
    },
    Argon2,
};

pub fn hash_password(password: &str) -> Result<String, argon2::password_hash::Error> {
    let salt = SaltString::generate(&mut OsRng);

    let argon2 = Argon2::default();

    let password_hash = argon2.hash_password(password.as_bytes(), &salt)?;

    Ok(password_hash.to_string())
}
```

You can then do:

```rust
fn main() {
    let password = "mypassword123";

    let hash = hash_password(password).unwrap();

    println!("{}", hash);
}
```

You'll get something resembling:

```text
$argon2id$v=19$m=19456,t=2,p=1$...
```

**Don't worry about memorizing that format.** Argon2 stores the parameters and salt in the encoded hash.

---

# 6. What is a salt?

A **salt** is random data added when hashing a password.

For example:

```text
password: hello123
salt: X7kP...
```

Argon2 combines them:

```text
hello123 + X7kP...
       ↓
    Argon2id
       ↓
     hash
```

The important thing is that **each password gets a unique random salt**.

That's why you should let the password-hashing library generate it:

```rust
let salt = SaltString::generate(&mut OsRng);
```

Don't create your own predictable salt.

---

# 7. Verifying a password

When the user logs in, you **don't decrypt the stored hash**.

Instead:

```text
Login password
      ↓
Argon2id + stored salt
      ↓
Compare with stored hash
```

Add this:

```rust
pub fn verify_password(
    password: &str,
    stored_hash: &str,
) -> Result<bool, argon2::password_hash::Error> {
    let parsed_hash = PasswordHash::new(stored_hash)?;

    Ok(Argon2::default()
        .verify_password(password.as_bytes(), &parsed_hash)
        .is_ok())
}
```

Then:

```rust
fn main() {
    let password = "mypassword123";

    let hash = hash_password(password).unwrap();

    let correct = verify_password("mypassword123", &hash).unwrap();
    let incorrect = verify_password("wrongpassword", &hash).unwrap();

    println!("{}", correct);
    println!("{}", incorrect);
}
```

Output:

```text
true
false
```

---

# 8. How this fits into registration

Your registration flow becomes:

```text
POST /register
       ↓
Receive password
       ↓
Hash password with Argon2id
       ↓
Store hash in PostgreSQL
       ↓
Return success
```

For example, your user model might eventually look like:

```rust
struct User {
    id: i32,
    email: String,
    password_hash: String,
}
```

Notice:

```rust
password_hash: String
```

not:

```rust
password: String
```

---

# 9. How login works

The login flow is:

```text
POST /login
      ↓
email + password
      ↓
Find user in PostgreSQL
      ↓
Get stored password_hash
      ↓
verify_password()
      ↓
Correct?
   ↙       ↘
 Yes        No
 ↓          ↓
Create     Reject
JWT        login
```

The important code is:

```rust
let valid = verify_password(
    &login.password,
    &user.password_hash,
)?;

if !valid {
    return Err("Invalid credentials");
}
```

Then, **only after the password is verified**, you'll create the JWT.

---

# 10. What you should understand by the end of the Week

You should be able to explain these six things:

### ① Hashing ≠ encryption

Password hashing is intended to be one-way.

### ② Never store plaintext passwords

Store:

```text
password_hash
```

not:

```text
password
```

### ③ Use a password-specific hashing algorithm

For this project:

```text
Argon2id
```

### ④ Use a random salt

Argon2 handles this for you when you generate a `SaltString`.

### ⑤ Registration hashes

```rust
let hash = hash_password(password)?;
```

Then save `hash` to PostgreSQL.

### ⑥ Login verifies

```rust
verify_password(password, stored_hash)
```

You **don't hash the login password yourself and compare strings**; use the password-hashing library's verifier.

---

## Your Week practice task

Before connecting PostgreSQL, build this small program:

```text
Register
   ↓
username + password
   ↓
hash password
   ↓
store username + hash in memory

Login
   ↓
username + password
   ↓
find stored hash
   ↓
verify password
   ↓
Login successful / Invalid credentials
```

Once you can build that without copying the solution line-for-line, you're ready to integrate password hashing into your **Axum + PostgreSQL authentication API**.
