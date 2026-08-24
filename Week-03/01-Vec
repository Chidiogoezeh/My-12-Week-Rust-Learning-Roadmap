We do not need to learn every feature of `Vec<T>`, but should understand how to **create, add, access, modify, remove, loop through, and check** a vector.

What is a `Vec<T>` in Rust?

A `Vec<T>` (vector) is a **growable list of values of the same type**.

Think of it like an array that can grow and shrink.

```rust
let numbers: Vec<i32> = Vec::new();
```

Or more commonly:

```rust
let numbers = vec![10, 20, 30];
```

The important thing is that all elements have the same type:

```rust
let numbers = vec![10, 20, 30]; // Vec<i32>
```

This would not work:

```rust
let numbers = vec![10, "hello", 30]; // Incorrect
```

---

## 1. Creating a `Vec`

### Empty vector

```rust
let mut students: Vec<String> = Vec::new();
```

`mut` is necessary because we're going to change the vector.

### Vector with initial values

```rust
let numbers = vec![10, 20, 30, 40];
```

You don't normally need to specify the type because Rust can infer it.

---

# 2. Adding values — `push()`

Use `push()` to add an item to the end.

```rust
let mut numbers = Vec::new();

numbers.push(10);
numbers.push(20);
numbers.push(30);

println!("{:?}", numbers);
```

Output:

```text
[10, 20, 30]
```

Think:

```text
push = add to the end
```

---

# 3. Accessing values

You can access an element using its **index**.

Important: Rust indexes start at **0**.

```rust
let numbers = vec![10, 20, 30];

println!("{}", numbers[0]);
println!("{}", numbers[1]);
println!("{}", numbers[2]);
```

Output:

```text
10
20
30
```

So:

```text
index:    0    1    2
value:   10   20   30
```

---

# 4. Safer access with `get()`

You can also use:

```rust
let numbers = vec![10, 20, 30];

let number = numbers.get(1);

println!("{:?}", number);
```

Output:

```text
Some(20)
```

If the index doesn't exist:

```rust
let number = numbers.get(10);

println!("{:?}", number);
```

Output:

```text
None
```

This is important because:

```rust
numbers[10]
```

would panic if the index doesn't exist.

Whereas:

```rust
numbers.get(10)
```

returns:

```rust
None
```

This connects directly to the **`Option`** topic you'll learn in Week 3.

---

# 5. Changing a value

The vector must be mutable.

```rust
let mut numbers = vec![10, 20, 30];

numbers[0] = 100;

println!("{:?}", numbers);
```

Output:

```text
[100, 20, 30]
```

---

# 6. Removing a value — `remove()`

```rust
let mut numbers = vec![10, 20, 30];

numbers.remove(1);

println!("{:?}", numbers);
```

Output:

```text
[10, 30]
```

We removed the value at index `1`.

```text
Before:
[10, 20, 30]
     ↑
   index 1

After:
[10, 30]
```

---

# 7. Removing the last value — `pop()`

`pop()` removes the last element.

```rust
let mut numbers = vec![10, 20, 30];

let last = numbers.pop();

println!("{:?}", numbers);
println!("{:?}", last);
```

Output:

```text
[10, 20]
Some(30)
```

Notice something important:

```rust
pop()
```

returns an `Option`.

Why?

Because the vector might be empty.

```rust
let mut numbers: Vec<i32> = Vec::new();

let value = numbers.pop();

println!("{:?}", value);
```

Output:

```text
None
```

---

# 8. Checking the length

Use:

```rust
len()
```

Example:

```rust
let numbers = vec![10, 20, 30];

println!("{}", numbers.len());
```

Output:

```text
3
```

---

# 9. Checking whether it's empty

Use:

```rust
is_empty()
```

Example:

```rust
let numbers: Vec<i32> = Vec::new();

println!("{}", numbers.is_empty());
```

Output:

```text
true
```

After adding something:

```rust
let mut numbers = Vec::new();

numbers.push(10);

println!("{}", numbers.is_empty());
```

Output:

```text
false
```

---

# 10. Looping through a `Vec`

This is extremely important.

```rust
let numbers = vec![10, 20, 30];

for number in numbers {
    println!("{}", number);
}
```

Output:

```text
10
20
30
```

You will use this pattern constantly when working with collections.

---

# 11. Looping without taking ownership

This connects to **borrowing**.

```rust
let numbers = vec![10, 20, 30];

for number in &numbers {
    println!("{}", number);
}

println!("{:?}", numbers);
```

The `&numbers` means we're borrowing the vector rather than consuming it.

This is an important Rust concept.

Compare:

```rust
for number in numbers
```

with:

```rust
for number in &numbers
```

The first takes ownership of the vector during iteration.

The second borrows it.

---

# 12. Finding the length and looping

A common pattern:

```rust
let students = vec!["John", "Mary", "David"];

for student in &students {
    println!("Student: {}", student);
}

println!("Total students: {}", students.len());
```

Output:

```text
Student: John
Student: Mary
Student: David
Total students: 3
```

---

# 13. `Vec<String>` example

You'll use vectors with structs and strings frequently in backend development.

```rust
let mut students: Vec<String> = Vec::new();

students.push(String::from("John"));
students.push(String::from("Mary"));
students.push(String::from("David"));

for student in &students {
    println!("{}", student);
}
```

---

# 14. `Vec` containing structs

This is particularly important for your Week 3 project.

```rust
struct Student {
    name: String,
    age: u32,
}

fn main() {
    let mut students = Vec::new();

    students.push(Student {
        name: String::from("John"),
        age: 20,
    });

    students.push(Student {
        name: String::from("Mary"),
        age: 22,
    });

    for student in &students {
        println!("{} - {}", student.name, student.age);
    }
}
```

This is much closer to what you will eventually do in backend applications.

For example, a database query might return:

```text
Vec<User>
```

meaning:

> A collection of `User` structs.

---

# 15. The methods you should know for Week 3

No need trying to memorize dozens of methods.

Focus on these:

| Method       | Purpose                   |
| ------------ | ------------------------- |
| `Vec::new()` | Create empty vector       |
| `vec![]`     | Create vector with values |
| `push()`     | Add item                  |
| `pop()`      | Remove last item          |
| `remove()`   | Remove item by index      |
| `len()`      | Get number of items       |
| `is_empty()` | Check if empty            |
| `get()`      | Safely access item        |
| `[]`         | Access item by index      |
| `iter()`     | Iterate over items        |

You should also understand:

```rust
&vector
```

for borrowing.

---

# Your Week 3 `Vec` target

Before moving on, you should be able to write this without copying:

```rust
struct Expense {
    description: String,
    amount: f64,
}

fn main() {
    let mut expenses = Vec::new();

    expenses.push(Expense {
        description: String::from("Food"),
        amount: 5000.0,
    });

    expenses.push(Expense {
        description: String::from("Transport"),
        amount: 2000.0,
    });

    for expense in &expenses {
        println!("{}: ₦{}", expense.description, expense.amount);
    }

    println!("Total expenses: {}", expenses.len());
}
```

If you understand **why `Vec` is mutable, how `push()` works, how indexing works, what `get()` returns, how `pop()` works, and why we use `&expenses` in the loop**, you've learned the important part of `Vec` for Week 3.

### One mental model to remember

```text
Vec<T>
  │
  ├── create → Vec::new() / vec![]
  ├── add → push()
  ├── access → [index] / get()
  ├── modify → [index] = value
  ├── remove → remove() / pop()
  ├── count → len()
  └── iterate → for item in &vector
```
