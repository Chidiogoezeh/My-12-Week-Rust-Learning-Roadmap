# Week 9 – Validation and Logging

Install:

```bash
cargo add validator
cargo add tracing
cargo add tracing-subscriber
```

Topics:

- Validation
- Structured logging
- Error handling
- HTTP middleware

Example:

```rust
#[derive(Validate)]
struct RegisterUser {
    #[validate(email)]
    email: String,
}
```

### Project

Upgrade Authentication API.

Add:

- Validation
- Logging
- Better errors
