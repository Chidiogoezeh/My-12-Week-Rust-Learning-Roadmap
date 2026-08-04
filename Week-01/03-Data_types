Data types are one of the foundations of Rust. Every value in Rust has a type, and the compiler uses these types to ensure your code is safe and efficient. Rust is **statically typed**, meaning types are known at compile time.

By the end of this lesson, you should understand:

- What data types are
- Scalar types
- Compound types
- Type inference
- Type annotations
- Type casting
- Numeric operations
- Common methods on types
- When to choose each type

---

# 1. What Are Data Types?

A **data type** tells Rust what kind of value a variable holds.

Examples:

- Number
- Decimal
- Character
- Text
- Boolean

```rust
fn main() {
    let age = 25;
    let name = "Alice";
    let is_student = true;
}
```

Rust knows:

- `age` → integer
- `name` → string slice (`&str`)
- `is_student` → boolean

---

# 2. Categories of Data Types

Rust groups types into two main categories:

```text
Data Types
│
├── Scalar Types
│   ├── Integer
│   ├── Floating Point
│   ├── Boolean
│   └── Character
│
└── Compound Types
    ├── Tuple
    └── Array
```

---

# 3. Integer Types

Integers are whole numbers.

Examples:

```rust
let age = 21;
let score = 100;
let temperature = -5;
```

Rust provides signed and unsigned integers.

| Type  | Size                   | Range              |
| ----- | ---------------------- | ------------------ |
| i8    | 8-bit                  | -128 to 127        |
| i16   | 16-bit                 | -32,768 to 32,767  |
| i32   | 32-bit                 | Common default     |
| i64   | 64-bit                 | Large numbers      |
| i128  | 128-bit                | Very large numbers |
| isize | Architecture dependent | Pointer-sized      |

Unsigned:

```text
u8
u16
u32
u64
u128
usize
```

These cannot store negative numbers.

Example:

```rust
fn main() {
    let age: u32 = 25;
    let balance: i64 = -1000;

    println!("{}", age);
    println!("{}", balance);
}
```

---

# 4. Integer Literals

Rust supports several number formats.

```rust
fn main() {
    let decimal = 98;
    let hex = 0xff;
    let octal = 0o77;
    let binary = 0b1111;
}
```

You can also improve readability:

```rust
let population = 1_234_567;
```

---

# 5. Floating Point Types

Floating-point numbers contain decimals.

Rust has:

```text
f32
f64
```

Example:

```rust
fn main() {
    let price = 19.99;
    let pi: f64 = 3.14159;

    println!("{}", price);
    println!("{}", pi);
}
```

Rust defaults to `f64`.

---

# 6. Boolean Type

Boolean values are either:

```text
true
false
```

Example:

```rust
fn main() {
    let logged_in = true;
    let premium = false;

    println!("{}", logged_in);
    println!("{}", premium);
}
```

Often used with conditions:

```rust
if logged_in {
    println!("Welcome!");
}
```

---

# 7. Character Type

Rust's `char` stores a single Unicode character.

```rust
fn main() {
    let grade = 'A';
    let heart = '❤';
    let emoji = '😊';

    println!("{}", grade);
    println!("{}", heart);
    println!("{}", emoji);
}
```

A `char` uses **single quotes**.

---

# 8. Strings

Rust has two common string types.

## String slice (`&str`)

Fixed, immutable text.

```rust
let name = "Alice";
```

## Owned `String`

Growable, mutable text.

```rust
let mut city = String::from("Lagos");

city.push_str(" State");

println!("{}", city);
```

Output:

```text
Lagos State
```

Use `String` when you need to modify text.

---

# 9. Tuples

A tuple groups different types together.

```rust
fn main() {
    let person = ("Alice", 25, true);

    println!("{}", person.0);
    println!("{}", person.1);
    println!("{}", person.2);
}
```

Output:

```text
Alice
25
true
```

Destructuring:

```rust
let (name, age, employed) = person;
```

---

# 10. Arrays

Arrays store multiple values of the same type.

```rust
fn main() {
    let numbers = [10, 20, 30, 40];

    println!("{}", numbers[0]);
    println!("{}", numbers[2]);
}
```

Output:

```text
10
30
```

All elements must have the same type.

---

# 11. Array Initialization

```rust
let scores = [100; 5];
```

Creates:

```text
[100, 100, 100, 100, 100]
```

---

# 12. Type Inference

Rust usually figures out the type.

```rust
let age = 30;
let salary = 1500.50;
let active = true;
```

No annotation needed.

---

# 13. Type Annotation

Sometimes you specify the type.

```rust
let age: u32 = 30;
let price: f64 = 19.99;
let active: bool = true;
```

---

# 14. Numeric Operations

Addition

```rust
let sum = 10 + 5;
```

Subtraction

```rust
let difference = 10 - 5;
```

Multiplication

```rust
let product = 10 * 5;
```

Division

```rust
let quotient = 10 / 2;
```

Remainder

```rust
let remainder = 10 % 3;
```

Example:

```rust
fn main() {
    let a = 20;
    let b = 6;

    println!("{}", a + b);
    println!("{}", a - b);
    println!("{}", a * b);
    println!("{}", a / b);
    println!("{}", a % b);
}
```

---

# 15. Integer vs Floating Division

```rust
let result = 5 / 2;
```

Output:

```text
2
```

Because both are integers.

Floating-point division:

```rust
let result = 5.0 / 2.0;
```

Output:

```text
2.5
```

---

# 16. Type Casting

Rust does not automatically convert between numeric types.

This won't compile:

```rust
let age: u32 = 20;
let price: f64 = age;
```

Instead:

```rust
let age: u32 = 20;

let price = age as f64;

println!("{}", price);
```

Output:

```text
20
```

---

# 17. Common String Methods

Length

```rust
let name = "Alice";

println!("{}", name.len());
```

Uppercase

```rust
let word = "rust";

println!("{}", word.to_uppercase());
```

Contains

```rust
let language = "Rust Programming";

println!("{}", language.contains("Rust"));
```

---

# 18. Common Array Methods

Length

```rust
let numbers = [1, 2, 3];

println!("{}", numbers.len());
```

Iterate

```rust
let numbers = [10, 20, 30];

for number in numbers {
    println!("{}", number);
}
```

---

# 19. Choosing the Right Type

| Use Case       | Type                 |
| -------------- | -------------------- |
| Age            | `u8` or `u32`        |
| Temperature    | `i32` or `f64`       |
| Price          | `f64`                |
| User name      | `String`             |
| Country code   | `&str`               |
| Is Admin       | `bool`               |
| Grade          | `char`               |
| Coordinates    | `(f64, f64)`         |
| Fixed weekdays | `["Mon", ...]` array |

---

# 20. Common Beginner Mistakes

### Mixing integer and floating-point types

```rust
let a = 5;
let b = 2.5;

let c = a + b;
```

This fails because `a` and `b` have different types.

Correct:

```rust
let c = a as f64 + b;
```

---

### Using double quotes for `char`

Wrong:

```rust
let grade = "A";
```

This is a string slice (`&str`).

Correct:

```rust
let grade = 'A';
```

---

### Array index out of bounds

```rust
let numbers = [1, 2, 3];

println!("{}", numbers[5]);
```

This will panic at runtime because index `5` doesn't exist.

---

# Mini Project: Student Information

```rust
fn main() {
    let name = String::from("Daud");
    let age: u32 = 22;
    let cgpa: f64 = 4.35;
    let graduated = false;
    let grade = 'A';

    let courses = [
        "Rust",
        "Databases",
        "Networking",
    ];

    println!("Name: {}", name);
    println!("Age: {}", age);
    println!("CGPA: {}", cgpa);
    println!("Graduated: {}", graduated);
    println!("Grade: {}", grade);

    println!("Courses:");

    for course in courses {
        println!("- {}", course);
    }
}
```

### Expected Output

```text
Name: Daud
Age: 22
CGPA: 4.35
Graduated: false
Grade: A
Courses:
- Rust
- Databases
- Networking
```

---

# Exercises

Try solving these on your own:

1. Declare variables for your name, age, height, and whether you're learning Rust. Print them.
2. Create an array of your five favorite programming languages and print each one.
3. Create a tuple containing your city, country, and ZIP/postal code. Access each element using indexing.
4. Calculate the area of a rectangle using two `f64` variables (`length` and `width`).
5. Convert an `i32` value to an `f64` using `as` and print both values.

---

# Key takeaways

- Rust has **scalar types** (`i32`, `f64`, `bool`, `char`) and **compound types** (tuples and arrays).
- Let Rust infer types where it makes the code clear, but add type annotations when they improve readability or are required.
- Use `String` for owned, editable text and `&str` for borrowed or fixed text.
- Rust does **not** perform implicit numeric conversions—use `as` when converting between numeric types.
- Choose types based on the domain of your data (for example, `bool` for flags, `u32` for IDs, `f64` for measurements, and tuples for grouping related values of different types).

Understanding Rust's data types thoroughly will make it much easier to learn ownership, borrowing, structs, enums, and collections in the next stages of foloowing the backend roadmap.
