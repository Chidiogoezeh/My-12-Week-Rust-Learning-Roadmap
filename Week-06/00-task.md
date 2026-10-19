# Week 6 – First REST API

Install:

```bash
cargo add axum
cargo add tokio --features full
cargo add serde --features derive
cargo add serde_json
```

Topics:

- Routes
- Handlers
- JSON
- Status codes

Endpoints:

```text
GET /users

GET /users/:id

POST /users

PUT /users/:id

DELETE /users/:id
```

### Project

Update the previously built cli project, to reusable API

User API

No database yet.

Store data in memory.
