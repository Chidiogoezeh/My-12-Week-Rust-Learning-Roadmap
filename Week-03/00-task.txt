# Week 3 – Collections and Error Handling

Topics:

- Vec
- HashMap
- Option
- Result
- Custom errors

Example:

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err("Division by zero".into())
    } else {
        Ok(a / b)
    }
}
```

### Project

Expense Tracker CLI.

Features:

- Add expense
- Delete expense
- Calculate total

Link: https://github.com/Chidiogoezeh/expense_tracker
