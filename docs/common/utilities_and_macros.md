# 常用工具与宏

## 0x01 工具概述

Rust 生态系统中有许多实用的工具和宏，可以提高开发效率：

- **开发工具**：cargo-expand、cargo-watch、cargo-edit
- **调试工具**：dbg!、tracing
- **性能分析**：flamegraph、perf
- **代码质量**：clippy、rustfmt

## 0x02 Cargo 扩展

### cargo-expand

查看宏展开：

```bash
# 安装
cargo install cargo-expand

# 使用
cargo expand
cargo expand module_name
```

### cargo-watch

自动重新编译：

```bash
# 安装
cargo install cargo-watch

# 使用
cargo watch
cargo watch -x run
cargo watch -x test
cargo watch -c -x run
```

### cargo-edit

管理依赖：

```bash
# 安装
cargo install cargo-edit

# 添加依赖
cargo add serde
cargo add tokio --features full

# 移除依赖
cargo remove serde

# 升级依赖
cargo upgrade
```

### cargo-tree

查看依赖树：

```bash
# 安装
cargo install cargo-tree

# 使用
cargo tree
cargo tree -d  # 显示重复依赖
```

## 0x03 调试工具

### dbg! 宏

```rust
fn main() {
    let x = 5;
    let y = dbg!(x * 2) + 1;
    
    dbg!(y);
    
    // 调试表达式
    let result = dbg!(some_function(3));
}

fn some_function(n: i32) -> i32 {
    dbg!(n * 2)
}
```

### tracing 日志

```toml
# Cargo.toml
[dependencies]
tracing = "0.1"
tracing-subscriber = "0.3"
```

```rust
use tracing::{info, warn, error, debug, instrument};

#[instrument]
fn fibonacci(n: u32) -> u32 {
    debug!("Computing fibonacci({})", n);
    
    if n <= 1 {
        return n;
    }
    
    let result = fibonacci(n - 1) + fibonacci(n - 2);
    info!("fibonacci({}) = {}", n, result);
    
    result
}

fn main() {
    tracing_subscriber::fmt::init();
    
    let result = fibonacci(10);
    println!("Result: {}", result);
}
```

### pretty_env_logger

```toml
# Cargo.toml
[dependencies]
log = "0.4"
pretty_env_logger = "0.5"
```

```rust
use log::{info, warn, error, debug};

fn main() {
    pretty_env_logger::init();
    
    info!("This is an info message");
    warn!("This is a warning");
    error!("This is an error");
    debug!("This is a debug message");
}
```

## 0x04 性能分析

### flamegraph

```bash
# 安装
cargo install flamegraph

# 生成火焰图
cargo flamegraph
cargo flamegraph --bin my_binary
```

### criterion 基准测试

```toml
# Cargo.toml
[dev-dependencies]
criterion = { version = "0.3", features = ["html_reports"] }

[[bench]]
name = "my_benchmark"
harness = false
```

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use my_crate::fibonacci;

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("fibonacci 20", |b| b.iter(|| fibonacci(black_box(20))));
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
```

### 使用 divan

```toml
# Cargo.toml
[dev-dependencies]
divan = "0.1"
```

```rust
fn main() {
    divan::main();
}

#[divan::bench]
fn fibonacci_20() -> u32 {
    fibonacci(20)
}

#[divan::bench(args = [10, 20, 30])]
fn fibonacci_n(n: u32) -> u32 {
    fibonacci(n)
}
```

## 0x05 代码质量

### clippy

```bash
# 运行 clippy
cargo clippy

# 自动修复
cargo clippy --fix

# 严格模式
cargo clippy -- -W clippy::pedantic
```

### rustfmt

```bash
# 格式化代码
cargo fmt

# 检查格式
cargo fmt -- --check

# 自定义配置
# rustfmt.toml
max_width = 100
tab_spaces = 4
edition = "2021"
```

### rust-analyzer

配置 VS Code 的 `settings.json`：

```json
{
    "rust-analyzer.checkOnSave.command": "clippy",
    "rust-analyzer.inlayHints.typeHints.enable": true,
    "rust-analyzer.inlayHints.parameterHints.enable": true,
    "rust-analyzer.completion.addCallParenthesis": true
}
```

## 0x06 常用宏

### println! 宏族

```rust
fn main() {
    println!("Hello, world!");
    print!("No newline");
    eprintln!("Error message");
    
    // 格式化输出
    println!("Name: {}, Age: {}", "Alice", 30);
    println!("{0} is {1} years old, {0} lives in {2}", "Alice", 30, "Beijing");
    println!("{name} is {age} years old", name="Alice", age=30);
    
    // 格式化数字
    println!("{:b}", 42);      // 二进制
    println!("{:o}", 42);      // 八进制
    println!("{:x}", 42);      // 十六进制
    println!("{:08}", 42);     // 填充零
}
```

### format! 宏

```rust
fn main() {
    let name = "Alice";
    let age = 30;
    
    let message = format!("{} is {} years old", name, age);
    println!("{}", message);
    
    // 构建字符串
    let mut s = String::new();
    s.push_str(&format!("Hello, {}!", name));
    println!("{}", s);
}
```

### vec! 宏

```rust
fn main() {
    let v1 = vec![1, 2, 3, 4, 5];
    let v2 = vec![0; 10];  // 10 个 0
    
    println!("v1: {:?}", v1);
    println!("v2: {:?}", v2);
}
```

### assert! 宏族

```rust
fn main() {
    let x = 5;
    let y = 10;
    
    assert!(x < y);
    assert_eq!(x + y, 15);
    assert_ne!(x, y);
    
    // 带消息
    assert!(x < y, "x ({}) should be less than y ({})", x, y);
}
```

### todo! 宏

```rust
fn not_implemented_yet() -> i32 {
    todo!("Implement this function later")
}

fn main() {
    // 会 panic 并显示消息
    // not_implemented_yet();
}
```

### cfg! 宏

```rust
fn main() {
    if cfg!(target_os = "windows") {
        println!("Running on Windows");
    } else if cfg!(target_os = "linux") {
        println!("Running on Linux");
    }
    
    // 条件编译
    #[cfg(debug_assertions)]
    println!("Debug mode");
    
    #[cfg(not(debug_assertions))]
    println!("Release mode");
}
```

## 0x07 自定义宏

### 简单声明式宏

```rust
macro_rules! create_function {
    ($name:ident) => {
        fn $name() {
            println!("Function {:?} called", stringify!($name));
        }
    };
}

create_function!(foo);
create_function!(bar);

fn main() {
    foo();
    bar();
}
```

### 带条件的宏

```rust
macro_rules! calculate {
    (eval $e:expr) => {
        println!("{} = {}", stringify!($e), $e);
    };
}

fn main() {
    calculate!(eval 1 + 2);
    calculate!(eval 3 * 4);
    calculate!(eval 5 - 6);
}
```

### 日志宏

```rust
macro_rules! log {
    ($level:expr, $($arg:tt)*) => {
        println!("[{}] {}: {}", $level, module_path!(), format!($($arg)*));
    };
}

macro_rules! info {
    ($($arg:tt)*) => {
        log!("INFO", $($arg)*);
    };
}

macro_rules! error {
    ($($arg:tt)*) => {
        log!("ERROR", $($arg)*);
    };
}

fn main() {
    info!("Application started");
    error!("Something went wrong: {}", "file not found");
}
```

## 0x08 实用工具

### dotenvy

```toml
# Cargo.toml
[dependencies]
dotenvy = "0.15"
```

```rust
use dotenvy::dotenv;
use std::env;

fn main() {
    dotenv().ok();
    
    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    
    println!("Database URL: {}", database_url);
}
```

### directories

```toml
# Cargo.toml
[dependencies]
directories = "5.0"
```

```rust
use directories::ProjectDirs;

fn main() {
    if let Some(proj_dirs) = ProjectDirs::from("com", "example", "myapp") {
        let config_dir = proj_dirs.config_dir();
        let data_dir = proj_dirs.data_dir();
        let cache_dir = proj_dirs.cache_dir();
        
        println!("Config: {}", config_dir.display());
        println!("Data: {}", data_dir.display());
        println!("Cache: {}", cache_dir.display());
    }
}
```

### chrono

```toml
# Cargo.toml
[dependencies]
chrono = "0.4"
```

```rust
use chrono::{Utc, Local, DateTime, TimeZone};

fn main() {
    let now = Utc::now();
    println!("UTC: {}", now);
    
    let local = Local::now();
    println!("Local: {}", local);
    
    // 格式化
    let formatted = now.format("%Y-%m-%d %H:%M:%S");
    println!("Formatted: {}", formatted);
    
    // 解析
    let parsed = Utc.datetime_from_str("2023-01-01 12:00:00", "%Y-%m-%d %H:%M:%S");
    println!("Parsed: {:?}", parsed);
}
```

### uuid

```toml
# Cargo.toml
[dependencies]
uuid = { version = "1.0", features = ["v4"] }
```

```rust
use uuid::Uuid;

fn main() {
    let id = Uuid::new_v4();
    println!("UUID: {}", id);
    
    let id_str = id.to_string();
    println!("UUID String: {}", id_str);
}
```

## 0x09 测试工具

### proptest

```toml
# Cargo.toml
[dev-dependencies]
proptest = "1.0"
```

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_add_commutative(a in 0..1000i32, b in 0..1000i32) {
        assert_eq!(a + b, b + a);
    }
    
    #[test]
    fn test_add_associative(
        a in 0..1000i32,
        b in 0..1000i32,
        c in 0..1000i32
    ) {
        assert_eq!((a + b) + c, a + (b + c));
    }
}
```

### mockall

```toml
# Cargo.toml
[dev-dependencies]
mockall = "0.11"
```

```rust
use mockall::automock;

#[automock]
trait Database {
    fn get_user(&self, id: u32) -> Option<String>;
}

fn get_user_name(db: &dyn Database, id: u32) -> String {
    db.get_user(id).unwrap_or_else(|| "Unknown".to_string())
}

#[cfg(test)]
mod tests {
    use super::*;
    use mockall::predicate::*;
    
    #[test]
    fn test_get_user_name() {
        let mut mock_db = MockDatabase::new();
        
        mock_db.expect_get_user()
            .with(eq(1))
            .times(1)
            .returning(|_| Some("Alice".to_string()));
        
        let name = get_user_name(&mock_db, 1);
        assert_eq!(name, "Alice");
    }
}
```

## 0x10 实际应用示例

### 配置管理工具

```rust
use serde::{Deserialize, Serialize};
use std::path::Path;
use directories::ProjectDirs;

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    database_url: String,
    port: u16,
    debug: bool,
}

impl Config {
    fn load() -> Result<Self, Box<dyn std::error::Error>> {
        // 首先尝试从环境变量加载
        if let Ok(url) = std::env::var("DATABASE_URL") {
            return Ok(Config {
                database_url: url,
                port: std::env::var("PORT")
                    .unwrap_or_else(|_| "8080".to_string())
                    .parse()?,
                debug: std::env::var("DEBUG")
                    .unwrap_or_else(|_| "false".to_string())
                    .parse()?,
            });
        }
        
        // 然后尝试从配置文件加载
        if let Some(proj_dirs) = ProjectDirs::from("com", "example", "myapp") {
            let config_path = proj_dirs.config_dir().join("config.toml");
            
            if config_path.exists() {
                let content = std::fs::read_to_string(config_path)?;
                let config: Config = toml::from_str(&content)?;
                return Ok(config);
            }
        }
        
        // 最后使用默认配置
        Ok(Config::default())
    }
    
    fn save(&self) -> Result<(), Box<dyn std::error::Error>> {
        if let Some(proj_dirs) = ProjectDirs::from("com", "example", "myapp") {
            let config_dir = proj_dirs.config_dir();
            std::fs::create_dir_all(config_dir)?;
            
            let config_path = config_dir.join("config.toml");
            let content = toml::to_string(self)?;
            std::fs::write(config_path, content)?;
        }
        
        Ok(())
    }
}

impl Default for Config {
    fn default() -> Self {
        Config {
            database_url: "sqlite:data.db".to_string(),
            port: 8080,
            debug: false,
        }
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    dotenvy::dotenv().ok();
    
    let config = Config::load()?;
    println!("Config: {:?}", config);
    
    config.save()?;
    
    Ok(())
}
```

## 参考

- [cargo-expand 文档](https://github.com/dtolnay/cargo-expand)
- [tracing 文档](https://docs.rs/tracing/)
- [clippy 文档](https://rust-lang.github.io/rust-clippy/)
- [criterion 文档](https://docs.rs/criterion/)
- [chrono 文档](https://docs.rs/chrono/)
- [directories 文档](https://docs.rs/directories/)
- [uuid 文档](https://docs.rs/uuid/)