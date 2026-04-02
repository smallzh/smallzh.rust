# 测试

## 0x01 测试概述

Rust 有强大的测试框架，支持：

- 单元测试（Unit Tests）
- 集成测试（Integration Tests）
- 文档测试（Documentation Tests）
- 基准测试（Benchmark Tests）

### 测试命令

```bash
# 运行所有测试
cargo test

# 运行特定测试
cargo test test_name

# 运行集成测试
cargo test --test integration_test

# 运行文档测试
cargo test --doc

# 显示测试输出
cargo test -- --nocapture
```

## 0x02 单元测试

### 基本单元测试

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
        assert_eq!(add(-1, 1), 0);
        assert_eq!(add(0, 0), 0);
    }
}
```

### 测试私有函数

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn private_helper(x: i32) -> i32 {
    x * 2
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_private_helper() {
        assert_eq!(private_helper(5), 10);
    }
}
```

### 断言宏

```rust
#[test]
fn test_assertions() {
    let a = 5;
    let b = 10;
    
    // assert! - 检查布尔条件
    assert!(a < b);
    
    // assert_eq! - 检查相等
    assert_eq!(a + b, 15);
    
    // assert_ne! - 检查不相等
    assert_ne!(a, b);
    
    // 带消息的断言
    assert!(a < b, "a ({}) should be less than b ({})", a, b);
}
```

## 0x03 测试组织

### 测试模块

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn subtract(a: i32, b: i32) -> i32 {
    a - b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    mod add_tests {
        use super::*;
        
        #[test]
        fn test_positive_numbers() {
            assert_eq!(add(2, 3), 5);
        }
        
        #[test]
        fn test_negative_numbers() {
            assert_eq!(add(-2, -3), -5);
        }
    }
    
    mod subtract_tests {
        use super::*;
        
        #[test]
        fn test_subtraction() {
            assert_eq!(subtract(5, 3), 2);
        }
    }
}
```

### 测试辅助函数

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    fn setup() -> (i32, i32) {
        (2, 3)
    }
    
    #[test]
    fn test_add_with_setup() {
        let (a, b) = setup();
        assert_eq!(add(a, b), 5);
    }
}
```

## 0x04 测试属性

### #[test] 属性

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}
```

### #[should_panic] 属性

```rust
fn divide(a: f64, b: f64) -> f64 {
    if b == 0.0 {
        panic!("Division by zero");
    }
    a / b
}

#[test]
#[should_panic]
fn test_divide_by_zero() {
    divide(10.0, 0.0);
}

#[test]
#[should_panic(expected = "Division by zero")]
fn test_divide_by_zero_with_message() {
    divide(10.0, 0.0);
}
```

### #[ignore] 属性

```rust
#[test]
#[ignore]
fn expensive_test() {
    // 这个测试被忽略
    // 运行：cargo test -- --ignored
}
```

## 0x05 Result 和 Option 测试

### 使用 Result 的测试

```rust
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Division by zero".to_string())
    } else {
        Ok(a / b)
    }
}

#[test]
fn test_divide_success() -> Result<(), String> {
    let result = divide(10.0, 2.0)?;
    assert_eq!(result, 5.0);
    Ok(())
}

#[test]
fn test_divide_failure() {
    let result = divide(10.0, 0.0);
    assert!(result.is_err());
}
```

## 0x06 集成测试

### 创建集成测试文件

```
my_project/
├── src/
│   └── lib.rs
└── tests/
    ├── integration_test.rs
    └── common/
        └── mod.rs
```

### 集成测试示例

```rust
// tests/integration_test.rs
use my_project::add;

#[test]
fn test_add() {
    assert_eq!(add(2, 3), 5);
}

#[test]
fn test_add_multiple() {
    assert_eq!(add(1, 2), 3);
    assert_eq!(add(0, 0), 0);
    assert_eq!(add(-1, 1), 0);
}
```

### 测试辅助模块

```rust
// tests/common/mod.rs
pub fn setup() {
    // 初始化代码
    println!("Setting up tests");
}

pub fn teardown() {
    // 清理代码
    println!("Tearing down tests");
}
```

```rust
// tests/integration_test.rs
mod common;

#[test]
fn test_with_common_setup() {
    common::setup();
    // 测试代码
    common::teardown();
}
```

## 0x07 文档测试

### 基本文档测试

```rust
/// Adds two numbers together.
///
/// # Examples
///
/// ```
/// let result = my_project::add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

### 隐藏行

```rust
/// # Examples
///
/// ```
/// # use my_project::add;
/// let result = add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

### 错误处理测试

```rust
/// # Examples
///
/// ```
/// use my_project::divide;
///
/// let result = divide(10.0, 2.0);
/// assert!(result.is_ok());
/// assert_eq!(result.unwrap(), 5.0);
/// ```
pub fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Division by zero".to_string())
    } else {
        Ok(a / b)
    }
}
```

## 0x08 基准测试

### 使用 criterion

```toml
# Cargo.toml
[dev-dependencies]
criterion = "0.3"

[[bench]]
name = "my_benchmark"
harness = false
```

```rust
// benches/my_benchmark.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use my_project::add;

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("add 2 + 3", |b| b.iter(|| add(black_box(2), black_box(3))));
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
```

运行基准测试：
```bash
cargo bench
```

## 0x09 模拟和测试替身

### 使用 mockall

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
    fn save_user(&mut self, id: u32, name: &str);
}

struct UserService<D: Database> {
    db: D,
}

impl<D: Database> UserService<D> {
    fn new(db: D) -> Self {
        UserService { db }
    }
    
    fn get_user_name(&self, id: u32) -> Option<String> {
        self.db.get_user(id)
    }
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
        
        let service = UserService::new(mock_db);
        let name = service.get_user_name(1);
        
        assert_eq!(name, Some("Alice".to_string()));
    }
}
```

## 0x10 测试最佳实践

### 1. 测试命名约定

```rust
#[test]
fn test_function_name_scenario_expected_result() {
    // 测试代码
}

// 示例
#[test]
fn test_add_positive_numbers_returns_sum() {
    assert_eq!(add(2, 3), 5);
}

#[test]
fn test_divide_by_zero_returns_error() {
    let result = divide(10.0, 0.0);
    assert!(result.is_err());
}
```

### 2. 测试结构（AAA 模式）

```rust
#[test]
fn test_example() {
    // Arrange - 准备测试数据
    let a = 2;
    let b = 3;
    
    // Act - 执行被测试的操作
    let result = add(a, b);
    
    // Assert - 验证结果
    assert_eq!(result, 5);
}
```

### 3. 边界条件测试

```rust
#[test]
fn test_edge_cases() {
    // 最小值
    assert_eq!(add(i32::MIN, 0), i32::MIN);
    
    // 最大值
    assert_eq!(add(i32::MAX, 0), i32::MAX);
    
    // 零值
    assert_eq!(add(0, 0), 0);
    
    // 负数
    assert_eq!(add(-1, -1), -2);
}
```

### 4. 测试覆盖率

使用 `cargo-tarpaulin` 检查测试覆盖率：

```bash
cargo install cargo-tarpaulin
cargo tarpaulin --out Html
```

## 0x11 实际应用示例

### 示例 1：用户服务测试

```rust
// src/user.rs
pub struct User {
    pub id: u32,
    pub name: String,
    pub email: String,
}

pub struct UserService {
    users: Vec<User>,
}

impl UserService {
    pub fn new() -> Self {
        UserService { users: Vec::new() }
    }
    
    pub fn add_user(&mut self, user: User) -> Result<(), String> {
        if self.users.iter().any(|u| u.id == user.id) {
            return Err("User already exists".to_string());
        }
        
        if self.users.iter().any(|u| u.email == user.email) {
            return Err("Email already exists".to_string());
        }
        
        self.users.push(user);
        Ok(())
    }
    
    pub fn get_user(&self, id: u32) -> Option<&User> {
        self.users.iter().find(|u| u.id == id)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_add_user() {
        let mut service = UserService::new();
        
        let user = User {
            id: 1,
            name: "Alice".to_string(),
            email: "alice@example.com".to_string(),
        };
        
        assert!(service.add_user(user).is_ok());
        assert!(service.get_user(1).is_some());
    }
    
    #[test]
    fn test_add_duplicate_user() {
        let mut service = UserService::new();
        
        let user1 = User {
            id: 1,
            name: "Alice".to_string(),
            email: "alice@example.com".to_string(),
        };
        
        let user2 = User {
            id: 1,
            name: "Bob".to_string(),
            email: "bob@example.com".to_string(),
        };
        
        assert!(service.add_user(user1).is_ok());
        assert!(service.add_user(user2).is_err());
    }
}
```

## 参考

- [Rust 程序设计语言 - 测试](https://doc.rust-lang.org/book/ch11-00-testing.html)
- [Rust by Example - 测试](https://doc.rust-lang.org/rust-by-example/testing.html)
- [criterion 文档](https://docs.rs/criterion/)
- [mockall 文档](https://docs.rs/mockall/)
- [cargo-tarpaulin 文档](https://docs.rs/cargo-tarpaulin/)