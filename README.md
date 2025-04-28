## 🦀 Personal Rust Language Guide

---

### 1. Introduction

**Brief History and Philosophy:**
Rust is a systems programming language that originated at Mozilla Research in 2010, led by Graydon Hoare. Rust was designed to provide memory safety without using garbage collection, emphasizing performance, concurrency, and safety. Its core philosophy revolves around preventing common bugs like null pointer dereferencing, data races, and buffer overflows.

Rust is widely used for low-level systems programming, including operating systems, embedded systems, and performance-critical services.

**Compilation and Execution Example:**
```bash
# Compile Rust program
rustc main.rs

# Run the executable
./main
```

**Installing Rust and Setting Up in VS Code:**
1. **Install Rust:**
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```
   - This installs `rustc`, `cargo`, and `rustup`.

2. **Install VS Code:**
   - Download and install from [Visual Studio Code](https://code.visualstudio.com/).

3. **Install Rust extension in VS Code:**
   - Open VS Code and go to the Extensions view (`Ctrl+Shift+X`).
   - Search for "rust-analyzer" and install it. This provides IDE support (autocompletion, inline errors, etc.).

4. **Install LLDB for Debugging:**
   - **Linux:**
     ```bash
     sudo apt-get install lldb
     ```
   - **macOS:**
     ```bash
     brew install llvm
     ```
   - **Windows:**
     - Install LLVM from [LLVM releases](https://releases.llvm.org/).
     - Ensure `lldb.exe` is in your system PATH.

5. **Configure Debugging in VS Code:**
   - Install the "CodeLLDB" extension (also from the Extensions view).
   - Create a `.vscode/launch.json` configuration:
     ```json
     {
         "version": "0.2.0",
         "configurations": [
             {
                 "type": "lldb",
                 "request": "launch",
                 "name": "Debug Rust",
                 "program": "${workspaceFolder}/target/debug/<your_program>",
                 "args": [],
                 "cwd": "${workspaceFolder}"
             }
         ]
     }
     ```
   - Replace `<your_program>` with your compiled binary name.

**Cargo: The Rust Package Manager and Build System:**
- **What is Cargo?**
  - Cargo is Rust's official package manager and build system. It handles project creation, dependency management, building, running, and testing.
  - Cargo simplifies Rust development by managing the compilation process and ensuring that all dependencies are fetched and built correctly.

- **Installing Cargo:**
  - Cargo is installed automatically when you install Rust using `rustup`.
  - Verify installation:
    ```bash
    cargo --version
    ```

- **Common Cargo Commands:**
  ```bash
  cargo new my_project       # Create a new Rust project
  cargo build                # Compile the current project
  cargo run                  # Build and run the project
  cargo test                 # Run tests
  cargo check                # Check code without producing binaries
  cargo update               # Update dependencies
  ```

- **Cargo.toml:**
  - Each Cargo project has a `Cargo.toml` file that defines metadata and dependencies.
  ```toml
  [package]
  name = "my_project"
  version = "0.1.0"
  edition = "2021"

  [dependencies]
  serde = "1.0"
  ```

- **Why Cargo is Important:**
  - Automates project setup, building, and dependency resolution.
  - Ensures consistent builds across environments.
  - Integrates with crates.io, the Rust package registry, making it easy to share and use libraries.

---

### Common Rust Frameworks and Ecosystem

Rust's ecosystem provides various frameworks that cater to different application domains, from web development to system programming and data processing. Below are some widely adopted frameworks, when to use them, and real-world use cases:

**1. Web Development Frameworks:**
- **Actix-Web:**
  - **Use Case:** High-performance, asynchronous web servers and APIs.
  - **Why Choose:** Known for speed and scalability. Best suited for services requiring low-latency and high-throughput.
  - **Companies Using:** 
    - **Microsoft:** Actix powers parts of Azure IoT infrastructure for handling HTTP requests efficiently.

- **Rocket:**
  - **Use Case:** Rapid web application development with an emphasis on ease of use and safety.
  - **Why Choose:** Best for quick development cycles where ergonomics and type safety are prioritized.
  - **Companies Using:**
    - **Snapview (Germany):** Builds secure web platforms for financial services.

**Understanding Tokio and Web Frameworks:**
- **What does this mean?**
  - **Tokio** allows web frameworks like **Actix-Web** and **Hyper** to run asynchronous operations (such as handling HTTP requests) efficiently and at scale.
  - **Tokio** is the **foundation** upon which many async frameworks and tools in Rust are built.

- **Actual Web Frameworks:**
  - **Actix-Web** is the leading web framework for **performance**, using its own runtime (not directly dependent on Tokio).
  - **Rocket** is a popular framework focusing on **ease of use** and **safety**, now supporting Tokio for async capabilities.

- **Summary:**
  - **Tokio** = asynchronous runtime (concurrency infrastructure).
  - **Actix-Web**, **Rocket** = web frameworks.

- **When to Choose:**
  - **Actix-Web:** When extreme performance is required (low latency, high throughput).
  - **Rocket:** When productivity and ergonomics are the priority (rapid development, small to medium APIs).

**Performance Comparisons with Other Languages:**
- **Actix-Web** has consistently ranked among the top performers in **TechEmpower benchmarks**, often outperforming frameworks in other languages:
  - **Rust (Actix-Web)** > **Go (Gin)** > **Node.js (Express)** > **Python (FastAPI)** for raw HTTP throughput.
  - For real-world scenarios, performance depends on I/O patterns and workloads, but Rust generally delivers lower latency and higher throughput due to its zero-cost abstractions and efficient memory model.**
- **Actix-Web** vs **Rocket:** Choose Actix for performance-critical systems; Rocket for ergonomics and developer experience.
- **Tokio** for async needs, or **RTIC** for embedded real-time systems.
- **Bevy** if you need to build interactive 2D/3D games.
- **Polars** when working with large data analytics tasks in Rust.

---

### Integrating Databases, Message Brokers, and Docker with Rust

**1. Databases:**

- **PostgreSQL:**
  - Use the `tokio-postgres` or `diesel` crates for interacting with PostgreSQL.
  - Example with `tokio-postgres`:
    ```toml
    [dependencies]
    tokio = { version = "1", features = ["full"] }
    tokio-postgres = "0.7"
    ```
    ```rust
    use tokio_postgres::{NoTls, Error};

    #[tokio::main]
    async fn main() -> Result<(), Error> {
        let (client, connection) = tokio_postgres::connect("host=localhost user=postgres password=secret dbname=mydb", NoTls).await?;
        tokio::spawn(async move {
            if let Err(e) = connection.await {
                eprintln!("connection error: {}", e);
            }
        });
        let rows = client.query("SELECT * FROM users", &[]).await?;
        Ok(())
    }
    ```

- **MongoDB:**
  - Use the `mongodb` crate.
  ```toml
  [dependencies]
  mongodb = { version = "2.0", features = ["tokio-runtime"] }
  ```
  ```rust
  use mongodb::{Client, options::ClientOptions};

  #[tokio::main]
  async fn main() -> mongodb::error::Result<()> {
      let client_uri = "mongodb://localhost:27017";
      let options = ClientOptions::parse(client_uri).await?;
      let client = Client::with_options(options)?;
      Ok(())
  }
  ```

- **Redis:**
  - Use the `redis` crate.
  ```toml
  [dependencies]
  redis = "0.23"
  ```
  ```rust
  use redis::Commands;

  fn main() -> redis::RedisResult<()> {
      let client = redis::Client::open("redis://127.0.0.1/")?;
      let mut con = client.get_connection()?;
      let _: () = con.set("my_key", 42)?;
      let val: i32 = con.get("my_key")?;
      println!("Got value: {}", val);
      Ok(())
  }
  ```

**2. Message Brokers:**

- **RabbitMQ:**
  - Use the `lapin` crate (AMQP client).
  ```toml
  [dependencies]
  lapin = { version = "2.0", features = ["full"] }
  tokio = { version = "1", features = ["full"] }
  ```
  ```rust
  use lapin::{options::*, types::FieldTable, Connection, ConnectionProperties};

  #[tokio::main]
  async fn main() {
      let conn = Connection::connect("amqp://127.0.0.1:5672/%2f", ConnectionProperties::default()).await.unwrap();
      println!("Connected to RabbitMQ");
  }
  ```

- **Kafka:**
  - Use the `rdkafka` crate (Rust wrapper for librdkafka).
  ```toml
  [dependencies]
  rdkafka = { version = "0.29", features = ["tokio"] }
  ```
  ```rust
  use rdkafka::consumer::{Consumer, StreamConsumer};
  use rdkafka::config::ClientConfig;

  #[tokio::main]
  async fn main() {
      let consumer: StreamConsumer = ClientConfig::new()
          .set("group.id", "example")
          .set("bootstrap.servers", "localhost:9092")
          .create()
          .expect("Consumer creation failed");
      println!("Kafka consumer ready");
  }
  ```

**Native Support:**
- Rust doesn't have native integration for databases or message brokers but offers high-quality crates.
- **Best Practices:**
  - Use connection pools (e.g., `bb8` or `deadpool`) for database connections.
  - Handle retries, backoffs, and error handling robustly.
  - Prefer async runtimes like Tokio when dealing with I/O-heavy systems.

**3. Docker Integration:**

- **Dockerizing Rust Apps:**
  - Example Dockerfile:
  ```Dockerfile
  FROM rust:1.73 as builder
  WORKDIR /app
  COPY . .
  RUN cargo build --release

  FROM debian:buster-slim
  WORKDIR /app
  COPY --from=builder /app/target/release/my_app ./
  CMD ["./my_app"]
  ```

- **Common Docker Commands for Rust:**
  ```bash
  docker build -t my_rust_app .
  docker run -p 8080:8080 my_rust_app
  ```
  - Adjust `CMD` and ports according to your Rust application's configuration.

---

### 2. Data Types and Variables

**Primitive Types:**
- **Integers:**
  - Signed: `i8`, `i16`, `i32`, `i64`, `i128`, `isize`
  - Unsigned: `u8`, `u16`, `u32`, `u64`, `u128`, `usize`
- **Floating-point:** `f32`, `f64`
- **Boolean:** `bool`
- **Character:** `char`

**Choosing Signed vs Unsigned Integers and Their Sizes:**

- **Signed Integers (`i*`):**
  - Can represent both positive and negative numbers.
  - Example: `i32` ranges from -2,147,483,648 to 2,147,483,647.
  - Use when negative values are possible or expected.

- **Unsigned Integers (`u*`):**
  - Represent only non-negative numbers.
  - Example: `u32` ranges from 0 to 4,294,967,295.
  - Use for cases like indexing, sizes, counts where negative values are invalid.

- **Choosing Size (`8`, `16`, `32`, `64`, `128`, `isize`, `usize`):**
  - Choose based on the **range of values** you expect:
    - `u8` (0 to 255) for byte-level operations (e.g., image processing).
    - `u32` or `i32` (default for most integer calculations).
    - `u64` or `i64` for large datasets, file sizes, or timestamps.
  - Use **`isize`** and **`usize`** when dealing with memory sizes, indexing collections:
    - They adjust based on the architecture (e.g., 32-bit or 64-bit).

**Memory Size Table:**
| Type    | Size (bits) | Range                                        |
|---------|-------------|----------------------------------------------|
| `i8`    | 8           | -128 to 127                                  |
| `u8`    | 8           | 0 to 255                                     |
| `i16`   | 16          | -32,768 to 32,767                            |
| `u16`   | 16          | 0 to 65,535                                  |
| `i32`   | 32          | -2,147,483,648 to 2,147,483,647              |
| `u32`   | 32          | 0 to 4,294,967,295                           |
| `i64`   | 64          | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 |
| `u64`   | 64          | 0 to 18,446,744,073,709,551,615              |
| `i128`  | 128         | Extremely large range (signed)               |
| `u128`  | 128         | Extremely large range (unsigned)             |
| `isize` | Arch-dependent | Signed, matches pointer size (32 or 64 bits) |
| `usize` | Arch-dependent | Unsigned, matches pointer size (32 or 64 bits) |

**When to Define Sizes:**
- **Use smaller sizes (`u8`, `i16`)** for memory-constrained environments (e.g., embedded systems).
- **Use default sizes (`i32`, `u32`)** for general-purpose arithmetic.
- **Use larger sizes (`i64`, `u64`)** when dealing with large numeric values (e.g., big datasets, file handling).
- **Always prefer `usize` and `isize`** for collection indexing and memory-related operations, as they adapt to the system's architecture.


- Integers: `i8`, `i16`, `i32`, `i64`, `i128`, `isize` (signed), `u8`, `u16`, `u32`, `u64`, `u128`, `usize` (unsigned)
- Floating-point: `f32`, `f64`
- Boolean: `bool`
- Character: `char`

**Constants and Variables:**
```rust
const MAX_POINTS: u32 = 100_000; // Constant, immutable
let mut x: i32 = 5; // Mutable variable
let y = 10; // Immutable variable with type inference
```

**Enums:**
```rust
enum Direction {
    North,
    East,
    South,
    West,
}

let dir = Direction::North;
```

---

### 3. Operators

**Arithmetic Operators:** `+`, `-`, `*`, `/`, `%`
```rust
let sum = 5 + 3; // 8
let product = 4 * 2; // 8
```

**Relational Operators:** `==`, `!=`, `<`, `>`, `<=`, `>=`
```rust
let is_equal = 5 == 5; // true
```

**Logical Operators:** `&&`, `||`, `!`
```rust
let and = true && false; // false
```

**Bitwise Operators:** `&`, `|`, `^`, `<<`, `>>`
```rust
let bitwise_and = 0b1100 & 0b1010; // 0b1000
```

---

### 4. Input/Output (I/O)

**Standard I/O:**
```rust
use std::io;

let mut input = String::new();
io::stdin().read_line(&mut input).expect("Failed to read line");
println!("You entered: {}", input);
```

**File I/O:**
```rust
use std::fs::File;
use std::io::{self, Read, Write};

// Write to a file
let mut file = File::create("output.txt").expect("Could not create file");
file.write_all(b"Hello, Rust!").expect("Write failed");

// Read from a file
let mut file = File::open("output.txt").expect("Could not open file");
let mut content = String::new();
file.read_to_string(&mut content).expect("Read failed");
println!("File content: {}", content);
```

---

### 5. Control Flow

**Conditionals:**
```rust
let number = 5;
if number < 10 {
    println!("Less than 10");
} else {
    println!("10 or more");
}
```

**Loops:**
```rust
// For loop
for i in 0..5 {
    println!("i = {}", i);
}

// While loop
let mut n = 0;
while n < 5 {
    println!("n = {}", n);
    n += 1;
}

// Loop with break
let mut count = 0;
loop {
    if count == 3 {
        break;
    }
    println!("count = {}", count);
    count += 1;
}
```

---

### 6. Functions/Methods

**Function Declaration and Usage:**
```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

let result = add(5, 3);
println!("Result: {}", result);
```

**Pass-by-Value and Pass-by-Reference:**
```rust
// Pass-by-value
fn increment(mut num: i32) {
    num += 1;
    println!("Inside increment: {}", num);
}

// Pass-by-reference
fn increment_ref(num: &mut i32) {
    *num += 1;
}

let mut x = 5;
increment(x);
println!("After pass-by-value: {}", x);
increment_ref(&mut x);
println!("After pass-by-reference: {}", x);
```

---

### 7. Advanced Structures

**Arrays and Vectors:**
```rust
let arr = [1, 2, 3, 4, 5];
let vec = vec![1, 2, 3, 4, 5];
```

**Matrices:**
```rust
let matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];
```

**Structs:**
```rust
struct Point {
    x: i32,
    y: i32,
}

let p = Point { x: 1, y: 2 };
```

**References:**
```rust
let s = String::from("hello");
let r = &s; // Immutable reference
```

---

### 8. Memory Management

**Stack vs Heap: Detailed Overview**
- **Stack:** Fast, fixed-size memory. Used for primitives and references.
- **Heap:** Dynamically allocated memory for complex or variable-sized data (e.g., `Vec`, `Box`).

**Dynamic Memory Allocation Example:**
```rust
let boxed = Box::new(5); // Allocated on the heap
```

---

**When to Use Stack vs Heap:**

- **Use Stack:**
  - For small, fixed-size data known at compile time (e.g., integers, small structs).
  - When function-local data doesn't need to outlive the function's scope.
  - Pros: Faster allocation/deallocation due to LIFO nature, cache-friendly.

- **Use Heap:**
  - For dynamic-sized data (e.g., vectors, strings, large objects).
  - When the size of data isn't known at compile time or data must live beyond the function scope.
  - Pros: Allows flexible memory usage, supports complex data structures (trees, graphs).

**Calculating Memory Size in Rust:**

Rust provides ways to calculate and control memory allocation through the `std::mem` module.

```rust
use std::mem;

let x: i32 = 5;
println!("Size of x: {} bytes", mem::size_of_val(&x));

let vec = vec![1, 2, 3];
println!("Size of vec pointer on stack: {} bytes", mem::size_of_val(&vec));
println!("Size of vec data on heap: {} bytes", vec.len() * mem::size_of::<i32>());
```

**Estimating Heap Allocations:**

- Heap-allocated structures (e.g., `Box`, `Vec`, `String`) contain pointers on the stack and the actual data on the heap.
- Example for `Vec<T>`:
  - Stack size: 24 bytes on 64-bit systems (pointer + length + capacity).
  - Heap size: Number of elements * size of `T`.

**Memory Optimization Tips:**
- Pre-allocate memory when the size is known:
  ```rust
  let mut vec = Vec::with_capacity(1000); // Avoids frequent reallocations
  ```
- Use `Box<T>` for large structs to move them to the heap:
  ```rust
  struct LargeData { data: [u8; 1024] }
  let boxed = Box::new(LargeData { data: [0; 1024] });
  ```
- Use `Rc<T>` or `Arc<T>` for shared ownership with reference counting.
  - **Rc:** Single-threaded environments.
  - **Arc:** Multi-threaded environments.

### 9. Macros

```rust
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
}

say_hello!();
```

---

### 10. Practical Examples

**Hello World:**
```rust
fn main() {
    println!("Hello, World!");
}
```

**Simple Calculator:**
```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn main() {
    let result = add(2, 3);
    println!("Sum: {}", result);
}
```

**File Reader:**
```rust
use std::fs::File;
use std::io::{self, Read};

fn main() -> io::Result<()> {
    let mut file = File::open("output.txt")?;
    let mut content = String::new();
    file.read_to_string(&mut content)?;
    println!("Content: {}", content);
    Ok(())
}
```
