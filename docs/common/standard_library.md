# 常用标准库

## 0x01 标准库概述

Rust 标准库提供了丰富的功能，包括：

- 基本类型和操作
- 集合类型
- 错误处理
- 并发编程
- I/O 操作
- 网络编程
- 时间处理

### 导入标准库

```rust
// 标准库自动导入，无需显式使用 use
fn main() {
    println!("Hello, world!");
}
```

## 0x02 常用模块

### std::io - 输入输出

```rust
use std::io;

fn main() {
    println!("Enter your name:");
    
    let mut name = String::new();
    io::stdin().read_line(&mut name).expect("Failed to read line");
    
    println!("Hello, {}!", name.trim());
}
```

### std::fs - 文件系统

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // 读取文件
    let contents = fs::read_to_string("hello.txt")?;
    println!("File content: {}", contents);
    
    // 写入文件
    fs::write("output.txt", "Hello, world!")?;
    
    // 创建目录
    fs::create_dir_all("new_dir")?;
    
    Ok(())
}
```

### std::path - 路径处理

```rust
use std::path::Path;

fn main() {
    let path = Path::new("/tmp/file.txt");
    
    println!("Path: {}", path.display());
    println!("Exists: {}", path.exists());
    println!("Is file: {}", path.is_file());
    println!("Is dir: {}", path.is_dir());
    
    if let Some(ext) = path.extension() {
        println!("Extension: {}", ext.to_str().unwrap());
    }
    
    if let Some(name) = path.file_name() {
        println!("File name: {}", name.to_str().unwrap());
    }
}
```

## 0x03 时间处理

### std::time

```rust
use std::time::{Duration, Instant};

fn main() {
    // 测量时间
    let start = Instant::now();
    
    // 执行一些操作
    for _ in 0..1000000 {}
    
    let duration = start.elapsed();
    println!("Time elapsed: {:?}", duration);
    
    // 创建 Duration
    let five_seconds = Duration::from_secs(5);
    let ten_millis = Duration::from_millis(10);
    
    println!("Five seconds: {:?}", five_seconds);
    println!("Ten millis: {:?}", ten_millis);
}
```

### SystemTime

```rust
use std::time::SystemTime;

fn main() {
    let now = SystemTime::now();
    println!("Now: {:?}", now);
    
    // 计算时间差
    let duration = now.elapsed().unwrap();
    println!("Elapsed: {:?}", duration);
}
```

## 0x04 随机数生成

### 使用 rand 库

```toml
# Cargo.toml
[dependencies]
rand = "0.8"
```

```rust
use rand::Rng;

fn main() {
    let mut rng = rand::thread_rng();
    
    // 生成随机数
    let n: u32 = rng.gen();
    println!("Random number: {}", n);
    
    // 范围随机数
    let n: u32 = rng.gen_range(1..=100);
    println!("Random number (1-100): {}", n);
    
    // 浮点数
    let f: f64 = rng.gen();
    println!("Random float: {}", f);
}
```

## 0x05 正则表达式

### 使用 regex 库

```toml
# Cargo.toml
[dependencies]
regex = "1.0"
```

```rust
use regex::Regex;

fn main() {
    let re = Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap();
    
    assert!(re.is_match("2021-01-01"));
    assert!(!re.is_match("01-01-2021"));
    
    // 捕获组
    let re = Regex::new(r"(\d{4})-(\d{2})-(\d{2})").unwrap();
    let caps = re.captures("2021-01-01").unwrap();
    
    println!("Year: {}", &caps[1]);
    println!("Month: {}", &caps[2]);
    println!("Day: {}", &caps[3]);
    
    // 替换
    let result = re.replace_all("2021-01-01", "$2/$3/$1");
    println!("Replaced: {}", result);
}
```

## 0x06 序列化

### 使用 serde

```toml
# Cargo.toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u32,
    email: String,
}

fn main() {
    let person = Person {
        name: "Alice".to_string(),
        age: 30,
        email: "alice@example.com".to_string(),
    };
    
    // 序列化为 JSON
    let json = serde_json::to_string(&person).unwrap();
    println!("JSON: {}", json);
    
    // 从 JSON 反序列化
    let person2: Person = serde_json::from_str(&json).unwrap();
    println!("Person: {:?}", person2);
}
```

## 0x07 日志

### 使用 log 和 env_logger

```toml
# Cargo.toml
[dependencies]
log = "0.4"
env_logger = "0.9"
```

```rust
use log::{info, warn, error, debug};

fn main() {
    env_logger::init();
    
    info!("This is an info message");
    warn!("This is a warning");
    error!("This is an error");
    debug!("This is a debug message");
}
```

运行时设置日志级别：
```bash
RUST_LOG=info cargo run
```

## 0x08 命令行参数

### std::env

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    
    println!("Program: {}", args[0]);
    
    if args.len() > 1 {
        println!("Arguments:");
        for (i, arg) in args.iter().enumerate().skip(1) {
            println!("  {}: {}", i, arg);
        }
    }
    
    // 环境变量
    match env::var("HOME") {
        Ok(home) => println!("Home directory: {}", home),
        Err(e) => println!("Couldn't read HOME: {}", e),
    }
}
```

### 使用 clap 库

```toml
# Cargo.toml
[dependencies]
clap = { version = "3.0", features = ["derive"] }
```

```rust
use clap::Parser;

#[derive(Parser, Debug)]
#[clap(author, version, about)]
struct Args {
    /// Name of the person to greet
    #[clap(short, long)]
    name: String,

    /// Number of times to greet
    #[clap(short, long, default_value_t = 1)]
    count: u8,
}

fn main() {
    let args = Args::parse();
    
    for _ in 0..args.count {
        println!("Hello, {}!", args.name);
    }
}
```

## 0x09 网络请求

### 使用 reqwest

```toml
# Cargo.toml
[dependencies]
reqwest = { version = "0.11", features = ["json"] }
tokio = { version = "1.0", features = ["full"] }
```

```rust
use reqwest::Error;

#[tokio::main]
async fn main() -> Result<(), Error> {
    let body = reqwest::get("https://www.rust-lang.org")
        .await?
        .text()
        .await?;
    
    println!("body = {:?}", body);
    
    Ok(())
}
```

## 0x10 异步运行时

### 使用 tokio

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1.0", features = ["full"] }
```

```rust
use tokio::time::{sleep, Duration};

async fn say_hello() {
    println!("Hello");
    sleep(Duration::from_secs(1)).await;
    println!("World");
}

#[tokio::main]
async fn main() {
    say_hello().await;
}
```

## 0x11 数据库

### 使用 sqlx

```toml
# Cargo.toml
[dependencies]
sqlx = { version = "0.6", features = ["runtime-tokio-rustls", "sqlite"] }
```

```rust
use sqlx::sqlite::SqlitePool;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = SqlitePool::connect("sqlite:data.db").await?;
    
    sqlx::query(
        "CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)"
    ).execute(&pool).await?;
    
    sqlx::query("INSERT INTO users (name) VALUES (?)")
        .bind("Alice")
        .execute(&pool)
        .await?;
    
    let rows = sqlx::query!("SELECT id, name FROM users")
        .fetch_all(&pool)
        .await?;
    
    for row in rows {
        println!("User: {} ({})", row.name, row.id);
    }
    
    Ok(())
}
```

## 0x12 测试

### 基本测试

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
    
    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }
    
    #[test]
    #[should_panic]
    fn test_panic() {
        panic!("This should panic");
    }
    
    #[test]
    #[ignore]
    fn expensive_test() {
        // 这个测试被忽略
    }
}

fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

运行测试：
```bash
cargo test
cargo test test_add
cargo test -- --ignored
```

## 参考

- [Rust 标准库文档](https://doc.rust-lang.org/std/)
- [Rust by Example - 标准库](https://doc.rust-lang.org/rust-by-example/std.html)
- [Rust 异步编程](https://rust-lang.github.io/async-book/)
- [serde 文档](https://serde.rs/)
- [clap 文档](https://docs.rs/clap/)