# Configuration

**Configuration** is how your application gets values that can change between environments without changing your source code.

Examples:

```text
Database URL
JWT secret
Server port
Environment
```

The core idea is:

```text
.env / environment variables
          ↓
       Config
          ↓
     Application
```

---

## 1. Why do we need configuration?

Bad:

```rust
let database_url = "postgres://postgres:password@localhost/myapp";
let jwt_secret = "my-super-secret-key";
let port = 3000;
```

Problems:

- secrets are inside source code
- changing environments requires editing code
- secrets could accidentally be committed to Git

Instead:

```text
Development → development values
Testing     → test values
Production  → production values
```

while your Rust code stays the same.

---

# 2. Use a `Config` struct

For now, keep configuration simple.

```rust
pub struct Config {
    pub database_url: String,
    pub jwt_secret: String,
    pub port: u16,
}
```

This gives your application one place to access configuration.

---

# 3. Environment variables

For example:

```text
DATABASE_URL=postgres://postgres:password@localhost/myapp
JWT_SECRET=some-secret
PORT=3000
```

Your Rust application reads these values.

A common development setup uses `dotenvy`.

Add it:

```toml
[dependencies]
dotenvy = "0.15"
```

Then create:

```text
.env
```

Example:

```env
DATABASE_URL=postgres://postgres:password@localhost/myapp
JWT_SECRET=development-secret
PORT=3000
```

**Do not commit `.env` to Git.**

Add:

```text
.env
```

to `.gitignore`.

---

# 4. Load the environment variables

Create:

```text
src/config.rs
```

Then:

```rust
use std::env;

pub struct Config {
    pub database_url: String,
    pub jwt_secret: String,
    pub port: u16,
}

impl Config {
    pub fn from_env() -> Result<Self, Box<dyn std::error::Error>> {
        dotenvy::dotenv().ok();

        let database_url = env::var("DATABASE_URL")?;

        let jwt_secret = env::var("JWT_SECRET")?;

        let port = env::var("PORT")
            .unwrap_or_else(|_| "3000".to_string())
            .parse::<u16>()?;

        Ok(Self {
            database_url,
            jwt_secret,
            port,
        })
    }
}
```

Now your application has a single configuration object.

---

# 5. Why use `Result`?

This:

```rust
env::var("DATABASE_URL")?
```

means:

> If `DATABASE_URL` doesn't exist, configuration loading fails.

That's good.

You **don't want your application starting with an unknown database configuration.**

For example:

```text
DATABASE_URL missing
        ↓
Config::from_env()
        ↓
Error
        ↓
Application stops
```

Better than discovering the problem later.

---

# 6. Use the configuration in `main.rs`

```rust
mod config;

use config::Config;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = Config::from_env()?;

    println!("Server port: {}", config.port);

    let db = sqlx::PgPool::connect(&config.database_url).await?;

    println!("Database connected");

    // start application...

    Ok(())
}
```

The important flow is:

```text
Config
   ↓
database_url
   ↓
PgPool
```

and:

```text
Config
   ↓
port
   ↓
Server
```

---

# 7. Configuration + Dependency Injection

This connects directly to the other topics of the week.

You can create application state:

```rust
use sqlx::PgPool;

pub struct AppState {
    pub db: PgPool,
    pub jwt_secret: String,
}
```

Then in `main.rs`:

```rust
let config = Config::from_env()?;

let db = PgPool::connect(&config.database_url).await?;

let state = AppState {
    db,
    jwt_secret: config.jwt_secret,
};
```

Then give that state to Axum.

Conceptually:

```text
             Config
                │
        ┌───────┴───────┐
        ↓               ↓
     database        jwt_secret
        ↓               ↓
      PgPool          AppState
        └───────┬───────┘
                ↓
             Handlers
```

---

# 8. Don't put configuration everywhere

Avoid this:

```rust
async fn create_user() {
    let database_url = std::env::var("DATABASE_URL").unwrap();

    // ...
}
```

And:

```rust
async fn login() {
    let jwt_secret = std::env::var("JWT_SECRET").unwrap();

    // ...
}
```

Now every part of your application is reading environment variables.

Instead:

```text
Environment
     ↓
   Config
     ↓
 AppState
     ↓
Handlers / Services / Repositories
```

Load configuration **once** during startup.

---

# 9. What belongs in configuration?

For this Week's backend, things like:

```rust
pub struct Config {
    pub database_url: String,
    pub jwt_secret: String,
    pub port: u16,
}
```

Later you might have:

```rust
pub struct Config {
    pub database_url: String,
    pub jwt_secret: String,
    pub port: u16,
    pub log_level: String,
    pub redis_url: String,
}
```

But don't add fields until your application actually needs them.

---

# 10. Development vs Production

The code should remain the same.

### Development

```env
DATABASE_URL=postgres://localhost/myapp
JWT_SECRET=development-secret
PORT=3000
```

### Production

```env
DATABASE_URL=<production-database>
JWT_SECRET=<production-secret>
PORT=8080
```

Your Rust code doesn't change:

```rust
let config = Config::from_env()?;
```

That's the main benefit.

---

# 11. Secrets

Things such as:

```text
JWT_SECRET
DATABASE_PASSWORD
API_KEY
PAYSTACK_SECRET_KEY
```

should **not** be hardcoded.

Don't do:

```rust
let jwt_secret = "abc123";
```

And don't commit:

```text
JWT_SECRET=real-production-secret
```

to GitHub.

For local development, `.env` is convenient.

For production, use your deployment platform's **environment/secret management** rather than committing a `.env` file.

---

# 12. What should Configuration NOT do?

Configuration should **not** contain business logic.

Don't do:

```rust
if user.role == "admin" {
    // ...
}
```

That's application/business logic.

Configuration answers questions like:

```text
What database should I connect to?
What port should I use?
What secret should JWT use?
What environment am I running in?
```

---

# 13. Recommended Week's structure

Keep it simple:

```text
src/
├── main.rs
├── config.rs
├── errors.rs
├── routes/
├── handlers/
├── services/
├── repositories/
└── models/
```

Your configuration responsibility is mainly:

```text
config.rs
```

with:

```rust
pub struct Config {
    pub database_url: String,
    pub jwt_secret: String,
    pub port: u16,
}

impl Config {
    pub fn from_env() -> Result<Self, Box<dyn std::error::Error>> {
        dotenvy::dotenv().ok();

        Ok(Self {
            database_url: std::env::var("DATABASE_URL")?,
            jwt_secret: std::env::var("JWT_SECRET")?,
            port: std::env::var("PORT")
                .unwrap_or_else(|_| "3000".into())
                .parse()?,
        })
    }
}
```

---

# 14. What you should know for now

You should be able to explain and implement:

### 1. Environment variables

```rust
std::env::var("DATABASE_URL")
```

### 2. `.env`

```env
DATABASE_URL=...
JWT_SECRET=...
PORT=3000
```

### 3. Config struct

```rust
struct Config {
    database_url: String,
    jwt_secret: String,
    port: u16,
}
```

### 4. Configuration loading

```rust
Config::from_env()?
```

### 5. Dependency injection

```text
Config
  ↓
AppState
  ↓
Handlers / Services / Repositories
```

### 6. Secret management

Never hardcode or commit production secrets.

---

## The Week's mental model

Put these together with the topics you've already learned:

```text
                 Environment
                      │
                      ▼
                   Config
                      │
                      ▼
                   AppState
                ┌─────┴─────┐
                ↓           ↓
             Service     Repository
                │           │
                └─────┬─────┘
                      ↓
                   Database
```

The three rules to remember:

> **Configuration loads application settings.**

> **Load configuration once at startup.**

> **Pass the required values/dependencies into the parts that need them.**

That's the core of **Configuration for now**.
