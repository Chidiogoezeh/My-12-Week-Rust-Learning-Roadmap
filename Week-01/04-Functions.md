Functions are one of the core building blocks in Rust. They let you organize your code into reusable pieces, making programs easier to read, test, and maintain.

By the end of this lesson, you should understand:

- What a function is
- Declaring functions
- Calling functions
- Function parameters
- Return values
- Expressions vs. statements
- Type annotations
- Multiple parameters
- Scope
- Early returns
- Practical examples

---

# 1. What is a Function?

A **function** is a named block of code that performs a specific task.

Instead of writing the same code repeatedly, you write it once in a function and call it whenever you need it.

Think of a function as a machine:

```text
Input ───► Function ───► Output
```

Example:

```text
5, 3 ───► add() ───► 8
```

---

# 2. Declaring a Function

Functions are declared using the `fn` keyword.

```rust
fn greet() {
    println!("Hello, Rust!");
}

fn main() {
    greet();
}
```

Output

```text
Hello, Rust!
```

Here:

- `fn` declares a function.
- `greet` is the function name.
- `()` means it takes no parameters.
- `{}` contains the function body.

---

# 3. The `main()` Function

Every executable Rust program starts with the `main()` function.

```rust
fn main() {
    println!("Program started!");
}
```

Execution always begins here.

---

# 4. Calling a Function

A function does nothing until it is called.

```rust
fn say_hello() {
    println!("Hello!");
}

fn main() {
    say_hello();
    say_hello();
    say_hello();
}
```

Output

```text
Hello!
Hello!
Hello!
```

One function can be called as many times as needed.

---

# 5. Function Parameters

Parameters allow a function to receive data.

```rust
fn greet(name: &str) {
    println!("Hello, {}", name);
}

fn main() {
    greet("Alice");
    greet("Bob");
}
```

Output

```text
Hello, Alice
Hello, Bob
```

Notice that Rust requires parameter types.

Syntax:

```rust
fn function_name(parameter: Type) {
    // code
}
```

---

# 6. Multiple Parameters

Functions can accept more than one parameter.

```rust
fn add(a: i32, b: i32) {
    println!("{}", a + b);
}

fn main() {
    add(5, 7);
}
```

Output

```text
12
```

Each parameter has its own type.

---

# 7. Return Values

Many functions produce a value.

```rust
fn square(num: i32) -> i32 {
    num * num
}

fn main() {
    let result = square(6);

    println!("{}", result);
}
```

Output

```text
36
```

The `-> i32` means the function returns an `i32`.

---

# 8. Returning with `return`

You can explicitly return a value.

```rust
fn multiply(a: i32, b: i32) -> i32 {
    return a * b;
}

fn main() {
    println!("{}", multiply(4, 5));
}
```

Output

```text
20
```

Both explicit `return` and implicit returns are valid.

---

# 9. Implicit Returns (Rust Style)

Rust commonly returns the last expression without using `return`.

```rust
fn subtract(a: i32, b: i32) -> i32 {
    a - b
}
```

Notice there is **no semicolon**.

If you write:

```rust
a - b;
```

the function returns `()` (the unit type), causing a type error.

---

# 10. Expressions vs. Statements

This is an important Rust concept.

### Statement

Performs an action but doesn't produce a value.

```rust
let x = 5;
```

### Expression

Produces a value.

```rust
5 + 3
```

Functions often end with an expression:

```rust
fn double(x: i32) -> i32 {
    x * 2
}
```

---

# 11. Functions Without Return Values

Some functions simply perform an action.

```rust
fn print_message() {
    println!("Learning Rust!");
}

fn main() {
    print_message();
}
```

These return the **unit type** `()`.

---

# 12. Type Annotations

Rust requires parameter and return types.

```rust
fn area(length: f64, width: f64) -> f64 {
    length * width
}
```

Output

```text
20
```

if called with:

```rust
area(4.0, 5.0);
```

---

# 13. Scope Inside Functions

Variables declared inside a function only exist there.

```rust
fn greet() {
    let message = "Hello";

    println!("{}", message);
}

fn main() {
    greet();
}
```

Trying to use `message` in `main()` would fail.

---

# 14. Functions Calling Other Functions

Functions can call other functions.

```rust
fn greet() {
    println!("Hello!");
}

fn farewell() {
    println!("Goodbye!");
}

fn main() {
    greet();
    farewell();
}
```

Output

```text
Hello!
Goodbye!
```

---

# 15. Early Returns

Use `return` to exit a function early.

```rust
fn check_age(age: u32) {
    if age < 18 {
        println!("Access denied");
        return;
    }

    println!("Access granted");
}

fn main() {
    check_age(16);
    check_age(21);
}
```

Output

```text
Access denied
Access granted
```

---

# 16. Returning Multiple Values

Rust doesn't return multiple values directly, but tuples make it easy.

```rust
fn calculate(a: i32, b: i32) -> (i32, i32) {
    (a + b, a * b)
}

fn main() {
    let (sum, product) = calculate(4, 6);

    println!("Sum: {}", sum);
    println!("Product: {}", product);
}
```

Output

```text
Sum: 10
Product: 24
```

---

# 17. Passing Strings

Borrow a string with `&str` or `&String` to avoid taking ownership unnecessarily.

```rust
fn greet(name: &str) {
    println!("Welcome, {}!", name);
}

fn main() {
    let username = String::from("Daud");

    greet(&username);
}
```

Output

```text
Welcome, Daud!
```

You'll learn more about borrowing in the ownership section.

---

# 18. Practice Examples

### Example 1

```rust
fn say_hi() {
    println!("Hi!");
}

fn main() {
    say_hi();
}
```

---

### Example 2

```rust
fn cube(number: i32) -> i32 {
    number * number * number
}

fn main() {
    println!("{}", cube(3));
}
```

Output

```text
27
```

---

### Example 3

```rust
fn is_even(number: i32) -> bool {
    number % 2 == 0
}

fn main() {
    println!("{}", is_even(8));
    println!("{}", is_even(7));
}
```

Output

```text
true
false
```

---

# 19. Common Beginner Mistakes

### Forgetting parameter types

Incorrect

```rust
fn add(a, b) {
    a + b
}
```

Rust requires types.

Correct

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

---

### Adding a semicolon to the last expression

Incorrect

```rust
fn square(x: i32) -> i32 {
    x * x;
}
```

The semicolon turns the expression into a statement.

Correct

```rust
fn square(x: i32) -> i32 {
    x * x
}
```

---

### Calling a function before understanding ownership

```rust
fn print_name(name: String) {
    println!("{}", name);
}
```

Passing a `String` like this moves ownership. Later, you'll learn how borrowing (`&String` or `&str`) avoids unnecessary moves.

---

# Mini Project: Simple Banking Functions

```rust
fn deposit(balance: f64, amount: f64) -> f64 {
    balance + amount
}

fn withdraw(balance: f64, amount: f64) -> f64 {
    if amount > balance {
        println!("Insufficient funds.");
        return balance;
    }

    balance - amount
}

fn main() {
    let balance = 1000.0;

    let balance = deposit(balance, 500.0);
    println!("After deposit: {}", balance);

    let balance = withdraw(balance, 300.0);
    println!("After withdrawal: {}", balance);

    let balance = withdraw(balance, 1500.0);
    println!("Final balance: {}", balance);
}
```

### Expected Output

```text
After deposit: 1500
After withdrawal: 1200
Insufficient funds.
Final balance: 1200
```

---

# Challenge Exercises

1. Write a function that converts Celsius to Fahrenheit.
2. Write a function that returns the larger of two numbers.
3. Write a function that checks if a year is a leap year.
4. Write a function that calculates the area of a rectangle.
5. Write a function that returns both the minimum and maximum values from two integers using a tuple.

---

## Key takeaways

- Use `fn` to declare functions.
- Every executable Rust program starts in `main()`.
- Functions can take parameters, and every parameter must have a type.
- Use `->` to specify a return type.
- The last expression (without a semicolon) is returned automatically.
- Use `return` when you need to exit early or improve readability.
- Functions have their own scope, so variables declared inside them are not accessible outside.
- Small, focused functions make your code easier to test, reuse, and maintain.

Mastering functions prepares you for the next major topics in Rust: ownership, borrowing, structs, modules, and building real-world applications with clean, reusable code.
