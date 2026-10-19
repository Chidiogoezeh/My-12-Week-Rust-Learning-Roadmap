# 12-Week Rust Backend Development Journey

A hands-on journey from learning Rust fundamentals to building a production-style Personal Finance API using Rust, Axum, PostgreSQL, and SQLx.

The goal of this programme was to develop practical backend engineering skills through progressive, project-based learning. Rather than building a new application every week, I used the Expense Tracker as my main project from Week 3 onward, gradually evolving it into a Personal Finance API.

## Final Project: Personal Finance API

**Project:** Expense Tracker → Personal Finance API

**Personal Finance API Repository:** (https://github.com/Chidiogoezeh/expense_tracker)

The final project extends a basic expense tracker into a backend API for recording income and expenses, managing monthly budgets, and understanding spending patterns.

### Core Features

- **Authentication:** User registration, login, password hashing, and JWT authentication.
- **User management:** Profile access and user-specific data.
- **Financial transactions:** Create, list, update, and delete income and expense records.
- **Categories:** Organize financial transactions and analyze spending by category.
- **Budgets:** Create and manage monthly budgets and track spending against budget limits.
- **Dashboard and reports:** Summarize income, expenses, net cash flow, and category spending.
- **Database:** PostgreSQL persistence, SQLx queries, and database migrations.
- **Security:** Input validation, authentication middleware, and user ownership checks.
- **Error handling and logging:** Consistent HTTP error responses and structured logging with `tracing`.
- **Testing:** Unit tests and API integration tests for important workflows.
- **Deployment preparation:** Docker, Docker Compose, health checks, automated CI, API documentation, and test-environment deployment.

## Technology Stack

| Purpose                   | Technology                 |
| ------------------------- | -------------------------- |
| Programming language      | Rust                       |
| Async runtime             | Tokio                      |
| Web framework             | Axum                       |
| Serialization             | Serde                      |
| Database                  | PostgreSQL                 |
| Database access           | SQLx                       |
| Authentication            | JWT                        |
| Password hashing          | Argon2                     |
| Environment configuration | dotenvy                    |
| Validation                | validator                  |
| Structured logging        | tracing                    |
| Testing                   | cargo test                 |
| Containerization          | Docker, Docker Compose     |
| CI                        | GitHub Actions             |
| API documentation         | OpenAPI, Swagger UI        |
| Deployment                | Render, Railway, or Fly.io |

## 12-Week Learning Roadmap

Each week focused on a practical implementation milestone. From Week 3 onward, the Expense Tracker remained the central project, allowing each new concept to build on the previous work.

### Week 1 — Rust Fundamentals

**Topics**

- Installing and configuring the Rust toolchain.
- Variables, mutability, and data types.
- Functions and control flow.
- Ownership, borrowing, and references.
- Shadowing.

**Project:** CLI Calculator

Built a command-line calculator supporting addition, subtraction, multiplication, and division.

**Week 1 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-01)

### Week 2 — Structs, Enums, and Traits

**Topics**

- Structs and tuple structs.
- Enums and pattern matching.
- `match` expressions.
- Implementation blocks.
- Traits.

**Project:** Student Management CLI

Built a command-line application for adding, listing, and deleting student records.

**Week 2 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-02)

### Week 3 — Collections and Error Handling

**Topics**

- `Vec` and `HashMap`.
- `Option` and `Result`.
- Handling invalid input and errors.
- Basic business logic.

**Project milestone:** Expense Tracker CLI

Started the Expense Tracker with functionality for adding and deleting expenses and calculating the total.

**Week 3 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-03)

### Week 4 — Cargo, Packages, and Modules

**Topics**

- Cargo packages and crates.
- Module organization.
- Visibility and code reuse.
- External dependencies.

**Project milestone:** Organized the Expense Tracker codebase into reusable modules and explored Rust project structure.

**Week 4 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-04)

### Week 5 — Asynchronous Programming

**Topics**

- Futures, `async`, and `.await`.
- Tokio runtime.
- Asynchronous tasks and concurrency.

**Project milestone:** Applied asynchronous programming concepts in preparation for network-facing Rust backend services.

**Week 5 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-05)

### Week 6 — REST APIs with Axum

**Topics**

- Routes and handlers.
- HTTP methods and status codes.
- JSON request and response bodies.
- In-memory application state.

**Project milestone:** Began developing the Expense Tracker as a REST API using Axum.

**Week 6 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-06)

### Week 7 — PostgreSQL and SQLx

**Topics**

- PostgreSQL database setup.
- Connection pools.
- SQL queries and database access with SQLx.
- Database migrations.
- Persistent data storage.

**Project milestone:** Extended the Expense Tracker API to persist financial data in PostgreSQL.

**Week 7 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-07)

### Week 8 — Authentication

**Topics**

- Password hashing with Argon2.
- JWT generation and verification.
- Registration and login.
- Authentication middleware.
- Protected endpoints.

**Project milestone:** Added authentication to the Expense Tracker backend, including user registration, login, and protected profile access.

**Week 8 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-08)

### Week 9 — Validation, Logging, and Error Handling

**Topics**

- Request validation with `validator`.
- Structured logging with `tracing`.
- HTTP middleware.
- Consistent error responses.
- Database error handling.

**Project milestone:** Improved the Expense Tracker's input validation, error handling, and observability.

**Week 9 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-09)

### Week 10 — Production-Style Backend Architecture

**Topics**

- Separation of handlers, services, and repositories.
- Dependency injection.
- Application state management.
- Environment-based configuration.
- Separation of business logic from database access.

**Project milestone:** Refactored the Expense Tracker into a clearer backend architecture that could support additional financial features without unnecessary complexity.

**Week 10 Repository:** (https://github.com/Chidiogoezeh/My-12-Week-Rust-Learning-Roadmap/tree/master/Week-10)

### Week 11 — Personal Finance API

**Project:** Evolving the Expense Tracker into a Personal Finance API.

**Implemented Topics and features**

- User registration, login, and JWT authentication.
- Income and expense transaction management.
- Transaction categories and financial summaries.
- Monthly budgets and budget usage.
- PostgreSQL and SQLx migrations.
- User data isolation and ownership checks.
- Structured logging and consistent error handling.
- Unit and API integration testing.

**Milestone:** Expanded the original expense-tracking concept into a personal finance backend designed to support income tracking, spending analysis, and monthly budgeting.

**Repository:** [See final project]

### Week 12 — Containerization, CI, API Documentation, and Deployment

**Topics and tasks**

- Multi-stage Docker builds and `.dockerignore`.
- Docker Compose with PostgreSQL and persistent storage.
- Database migrations and service readiness.
- Health checks and graceful shutdown.
- End-to-end testing of core financial workflows.
- GitHub Actions for formatting, Clippy, tests, and release builds.
- OpenAPI documentation and Swagger UI.
- Deployment to a test environment with HTTPS.
- Secure environment configuration and deployment documentation.

**Milestone:** Prepare the Personal Finance API for reliable local execution, automated checks, and deployment to a test environment.

**Repository:** [See final project]

## Future Development

After completing the core Personal Finance API, planned extensions include:

### Financial Provider Integrations

- Design a provider-neutral adapter interface.
- Implement a mock financial provider.
- Validate and import external transactions.
- Prevent duplicate imports with database constraints.
- Reconcile imported transactions with existing financial records.
- Integrate a real provider when API access and authorization are available.

### Kubernetes

- Learn Pods, Deployments, Services, ConfigMaps, and Secrets.
- Configure readiness and liveness probes.
- Deploy the API to a test environment.
- Understand container orchestration beyond local Docker Compose development.

These are future extensions rather than prerequisites for completing the 12-week programme.

## Repository Structure

The final project's structure may evolve as implementation progresses. The intended separation of responsibilities is:

```text
personal-finance-api/
├── src/
│   ├── auth/
│   ├── transaction/
│   ├── budget/
│   ├── middleware.rs
│   ├── config.rs
│   ├── error.rs
│   ├── state.rs
│   └── main.rs
├── migrations/
├── tests/
├── .env.example
├── .gitignore
├── Cargo.toml
├── Dockerfile
├── compose.yaml
└── README.md
```

Only include files and modules that actually exist in the repository; this is the intended organizational direction, not a claim that every item is already implemented.

## Learning Outcomes

Through this project-based programme, I worked toward developing practical skills in:

- Writing Rust code using ownership, borrowing, types, and error handling.
- Building asynchronous REST APIs with Axum and Tokio.
- Persisting data with PostgreSQL and SQLx.
- Implementing authentication and protecting user-specific resources.
- Organizing backend code into maintainable modules.
- Validating input and handling errors consistently.
- Adding structured logging and automated tests.
- Containerizing an application and automating quality checks.
- Preparing and deploying a backend service.

## Learning Resources

- [The Rust Programming Language](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Axum Documentation](https://docs.rs/axum/latest/axum/)
- [Tokio Documentation](https://tokio.rs/)
- [SQLx Documentation](https://github.com/launchbadge/sqlx)
- [The Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)

## Conclusion

This 12-week journey focused on learning Rust backend development through progressive implementation rather than isolated tutorials alone.

The first two weeks introduced Rust fundamentals through small CLI applications. From Week 3 onward, the Expense Tracker became the central learning project, evolving through collections, modular design, asynchronous programming, REST APIs, PostgreSQL, authentication, validation, logging, and backend architecture.

The final milestone was transforming the Expense Tracker into a Personal Finance API and preparing it for containerized execution, automated testing, documentation, and deployment.

The long-term goal is to keep improving the project with reliable financial workflows, stronger testing, and carefully scoped integrations while maintaining a simple, secure, and maintainable backend architecture.
