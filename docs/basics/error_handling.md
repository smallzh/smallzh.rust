# 错误处理

## 0x01 Rust 错误处理概述

Rust 将错误分为两大类：
- **可恢复错误**（Recoverable）：使用 `Result<T, E>` 处理
- **不可恢复错误**（Unrecoverable）：使用 `panic!` 宏处理

### panic! 与不可恢复错误

```rust
fn main() {
    // 直接调用 panic!
    panic!("crash and burn");
}
```

### 使用 panic! 的场景

```rust
fn main() {
    let v = vec![1, 2, 3];
    
    // 访问越界会 panic
    v[99]; // thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99'
}
```

### 设置 panic 时的行为

```bash
# 设置 panic 时的回溯
RUST_BACKTRACE=1 cargo run
```

## 0x02 Result 枚举

### 定义

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### 基本使用

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
    
    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

### 匹配不同的错误

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");
    
    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

## 0x03 错误处理快捷方式

### unwrap

```rust
use std::fs::File;

fn main() {
    // unwrap：成功返回值，失败 panic
    let f = File::open("hello.txt").unwrap();
}
```

### expect

```rust
use std::fs::File;

fn main() {
    // expect：自定义错误信息
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

### unwrap_or

```rust
fn main() {
    let x: Result<i32, &str> = Err("error");
    let y: Result<i32, &str> = Ok(10);
    
    let x_unwrapped = x.unwrap_or(0); // 0
    let y_unwrapped = y.unwrap_or(0); // 10
    
    println!("x_unwrapped: {}", x_unwrapped);
    println!("y_unwrapped: {}", y_unwrapped);
}
```

### unwrap_or_else

```rust
fn main() {
    let x: Result<i32, &str> = Err("error");
    let y: Result<i32, &str> = Ok(10);
    
    let x_unwrapped = x.unwrap_or_else(|e| {
        println!("Error: {}", e);
        0
    });
    
    let y_unwrapped = y.unwrap_or_else(|_| 0);
    
    println!("x_unwrapped: {}", x_unwrapped);
    println!("y_unwrapped: {}", y_unwrapped);
}
```

## 0x04 传播错误

### 使用 match

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");
    
    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    
    let mut s = String::new();
    
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

### 使用 ? 运算符

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}

// 更简洁的写法
fn read_username_from_file_short() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

### ? 运算符的链式调用

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

## 0x05 自定义错误类型

### 定义自定义错误

```rust
use std::fmt;
use std::io;

#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(std::num::ParseIntError),
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::Io(err) => write!(f, "IO error: {}", err),
            AppError::Parse(err) => write!(f, "Parse error: {}", err),
        }
    }
}

impl From<io::Error> for AppError {
    fn from(err: io::Error) -> AppError {
        AppError::Io(err)
    }
}

impl From<std::num::ParseIntError> for AppError {
    fn from(err: std::num::ParseIntError) -> AppError {
        AppError::Parse(err)
    }
}
```

### 使用自定义错误

```rust
use std::fs;
use std::io;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(ParseIntError),
}

impl From<io::Error> for AppError {
    fn from(err: io::Error) -> AppError {
        AppError::Io(err)
    }
}

impl From<ParseIntError> for AppError {
    fn from(err: ParseIntError) -> AppError {
        AppError::Parse(err)
    }
}

fn read_and_parse() -> Result<i32, AppError> {
    let content = fs::read_to_string("config.txt")?; // 自动转换
    let number: i32 = content.trim().parse()?; // 自动转换
    Ok(number)
}

fn main() {
    match read_and_parse() {
        Ok(n) => println!("Number: {}", n),
        Err(e) => println!("Error: {:?}", e),
    }
}
```

## 0x06 使用 thiserror 库

### 添加依赖

```toml
# Cargo.toml
[dependencies]
thiserror = "1.0"
```

### 定义错误类型

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum DataStoreError {
    #[error("data store disconnected")]
    Disconnect(#[from] io::Error),
    #[error("the data for key `{0}` is not available")]
    Redaction(String),
    #[error("invalid header (expected {expected:?}, got {found:?})")]
    InvalidHeader {
        expected: String,
        found: String,
    },
    #[error("unknown data store error")]
    Unknown,
}
```

## 0x07 使用 anyhow 库

### 添加依赖

```toml
# Cargo.toml
[dependencies]
anyhow = "1.0"
```

### 基本使用

```rust
use anyhow::{Result, Context};

fn main() -> Result<()> {
    let content = std::fs::read_to_string("config.txt")
        .context("Failed to read config file")?;
    
    println!("Content: {}", content);
    Ok(())
}
```

### 链式错误处理

```rust
use anyhow::{Result, Context, anyhow};

fn parse_config(content: &str) -> Result<i32> {
    let number: i32 = content.trim().parse()
        .context("Failed to parse number")?;
    
    if number < 0 {
        return Err(anyhow!("Number must be positive"));
    }
    
    Ok(number)
}

fn main() -> Result<()> {
    let content = std::fs::read_to_string("config.txt")
        .context("Failed to read config file")?;
    
    let number = parse_config(&content)?;
    println!("Number: {}", number);
    Ok(())
}
```

## 0x08 多错误处理

### 使用枚举

```rust
use std::io;
use std::num::ParseIntError;

#[derive(Debug)]
enum CliError {
    IoError(io::Error),
    ParseError(ParseIntError),
}

impl From<io::Error> for CliError {
    fn from(err: io::Error) -> CliError {
        CliError::IoError(err)
    }
}

impl From<ParseIntError> for CliError {
    fn from(err: ParseIntError) -> CliError {
        CliError::ParseError(err)
    }
}

fn main() -> Result<(), CliError> {
    let content = std::fs::read_to_string("config.txt")?;
    let number: i32 = content.trim().parse()?;
    println!("Number: {}", number);
    Ok(())
}
```

## 0x09 错误处理最佳实践

### 1. 使用 Result 而不是 panic

```rust
// 好的做法
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Division by zero".to_string())
    } else {
        Ok(a / b)
    }
}

// 不好的做法
fn divide_bad(a: f64, b: f64) -> f64 {
    if b == 0.0 {
        panic!("Division by zero");
    }
    a / b
}
```

### 2. 提供有意义的错误信息

```rust
use std::fs;

fn read_config() -> Result<String, String> {
    fs::read_to_string("config.toml")
        .map_err(|e| format!("Failed to read config file: {}", e))
}
```

### 3. 使用上下文

```rust
use anyhow::{Result, Context};

fn load_user(id: u32) -> Result<User> {
    let config = load_config()
        .context("Failed to load configuration")?;
    
    let user = fetch_user(id, &config)
        .context(format!("Failed to fetch user {}", id))?;
    
    Ok(user)
}
```

### 4. 错误转换

```rust
use std::io;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(ParseIntError),
}

impl From<io::Error> for AppError {
    fn from(err: io::Error) -> AppError {
        AppError::Io(err)
    }
}

impl From<ParseIntError> for AppError {
    fn from(err: ParseIntError) -> AppError {
        AppError::Parse(err)
    }
}

fn read_number() -> Result<i32, AppError> {
    let content = std::fs::read_to_string("number.txt")?; // 自动转换
    let number = content.trim().parse()?; // 自动转换
    Ok(number)
}
```

## 0x10 实际应用示例

### 示例 1：配置文件加载

```rust
use std::fs;
use std::io;
use std::num::ParseIntError;
use std::collections::HashMap;

#[derive(Debug)]
enum ConfigError {
    IoError(io::Error),
    ParseError(ParseIntError),
    MissingField(String),
}

impl From<io::Error> for ConfigError {
    fn from(err: io::Error) -> ConfigError {
        ConfigError::IoError(err)
    }
}

impl From<ParseIntError> for ConfigError {
    fn from(err: ParseIntError) -> ConfigError {
        ConfigError::ParseError(err)
    }
}

struct Config {
    port: u16,
    host: String,
    debug: bool,
}

impl Config {
    fn load(path: &str) -> Result<Config, ConfigError> {
        let content = fs::read_to_string(path)?;
        let mut map = HashMap::new();
        
        for line in content.lines() {
            if let Some((key, value)) = line.split_once('=') {
                map.insert(key.trim(), value.trim());
            }
        }
        
        let port = map.get("port")
            .ok_or_else(|| ConfigError::MissingField("port".to_string()))?
            .parse()?;
        
        let host = map.get("host")
            .ok_or_else(|| ConfigError::MissingField("host".to_string()))?
            .to_string();
        
        let debug = map.get("debug")
            .map(|s| s == &"true")
            .unwrap_or(false);
        
        Ok(Config { port, host, debug })
    }
}

fn main() {
    match Config::load("config.txt") {
        Ok(config) => println!("Config: {:?}", config),
        Err(e) => println!("Error loading config: {:?}", e),
    }
}
```

### 示例 2：数据库操作

```rust
use std::io;
use std::fmt;

#[derive(Debug)]
enum DbError {
    ConnectionError(io::Error),
    QueryError(String),
    NotFound(String),
}

impl fmt::Display for DbError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            DbError::ConnectionError(e) => write!(f, "Connection error: {}", e),
            DbError::QueryError(msg) => write!(f, "Query error: {}", msg),
            DbError::NotFound(id) => write!(f, "Record not found: {}", id),
        }
    }
}

impl From<io::Error> for DbError {
    fn from(err: io::Error) -> DbError {
        DbError::ConnectionError(err)
    }
}

struct User {
    id: u32,
    name: String,
}

fn find_user(id: u32) -> Result<User, DbError> {
    if id == 0 {
        return Err(DbError::NotFound(format!("User with id {}", id)));
    }
    
    Ok(User {
        id,
        name: format!("User {}", id),
    })
}

fn main() {
    match find_user(1) {
        Ok(user) => println!("Found user: {} ({})", user.name, user.id),
        Err(e) => println!("Error: {}", e),
    }
    
    match find_user(0) {
        Ok(user) => println!("Found user: {} ({})", user.name, user.id),
        Err(e) => println!("Error: {}", e),
    }
}
```

## 参考

- [Rust 程序设计语言 - 错误处理](https://doc.rust-lang.org/book/ch09-00-error-handling.html)
- [Rust by Example - 错误处理](https://doc.rust-lang.org/rust-by-example/error.html)
- [Rust 错误处理指南](https://blog.burntsushi.net/err0r/)
- [thiserror 文档](https://docs.rs/thiserror)
- [anyhow 文档](https://docs.rs/anyhow)