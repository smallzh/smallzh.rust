# 宏基础

## 0x01 宏概述

Rust 的宏系统允许你编写可以生成代码的代码。宏在编译时展开，可以减少重复代码，创建领域特定语言（DSL）。

### 宏与函数的区别

- 宏在编译时展开，函数在运行时调用
- 宏可以接受不同数量和类型的参数
- 宏可以生成结构、函数、trait 等

## 0x02 声明式宏 macro_rules!

### 基本语法

```rust
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
}

fn main() {
    say_hello!();
}
```

### 带参数的宏

```rust
macro_rules! create_function {
    ($func_name:ident) => {
        fn $func_name() {
            println!("You called {:?}()", stringify!($func_name));
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

### 模式匹配

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

### 多模式宏

```rust
macro_rules! test {
    ($left:expr; and $right:expr) => {
        println!("{:?} and {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left && $right);
    };
    ($left:expr; or $right:expr) => {
        println!("{:?} or {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left || $right);
    };
}

fn main() {
    test!(1i32 + 1 == 2i32; and 2i32 * 2 == 4i32);
    test!(true; or false);
}
```

## 0x03 宏规则匹配

### 表达式 ($e:expr)

```rust
macro_rules! print_expr {
    ($e:expr) => {
        println!("{} = {}", stringify!($e), $e);
    };
}

fn main() {
    print_expr!(1 + 2);
    print_expr!(3 * 4);
    print_expr!("hello");
}
```

### 标识符 ($i:ident)

```rust
macro_rules! create_variable {
    ($name:ident) => {
        let $name = 42;
    };
}

fn main() {
    create_variable!(x);
    println!("x = {}", x);
}
```

### 类型 ($t:ty)

```rust
macro_rules! create_vec {
    ($t:ty) => {
        Vec::<$t>::new()
    };
}

fn main() {
    let v: Vec<i32> = create_vec!(i32);
    println!("v = {:?}", v);
}
```

### 语句 ($s:stmt)

```rust
macro_rules! execute {
    ($s:stmt) => {
        $s
    };
}

fn main() {
    execute!(println!("Hello"));
}
```

### 块 ($b:block)

```rust
macro_rules! with_logging {
    ($b:block) => {
        println!("Before");
        $b
        println!("After");
    };
}

fn main() {
    with_logging!({
        println!("During");
    });
}
```

### 模式 ($p:pat)

```rust
macro_rules! match_pattern {
    ($p:pat) => {
        let x = 5;
        match x {
            $p => println!("Matched"),
            _ => println!("Not matched"),
        }
    };
}

fn main() {
    match_pattern!(5);
}
```

### 路径 ($path:path)

```rust
macro_rules! call_function {
    ($path:path) => {
        $path();
    };
}

fn hello() {
    println!("Hello!");
}

fn main() {
    call_function!(hello);
}
```

## 0x04 重复模式

### 重复语法

```rust
macro_rules! create_vec {
    ($($x:expr),*) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

fn main() {
    let v = create_vec![1, 2, 3];
    println!("v = {:?}", v);
}
```

### 重复分隔符

```rust
macro_rules! create_vec {
    ($($x:expr),*) => {
        vec![$($x),*]
    };
    ($($x:expr;)*) => {
        vec![$($x),*]
    };
    ($($x:expr),+ $(,)?) => {
        vec![$($x),*]
    };
}

fn main() {
    let v1 = create_vec![1, 2, 3];
    let v2 = create_vec![1; 2; 3];
    let v3 = create_vec![1, 2, 3,]; // 尾逗号
    
    println!("v1 = {:?}", v1);
    println!("v2 = {:?}", v2);
    println!("v3 = {:?}", v3);
}
```

## 0x05 递归宏

### 递归展开

```rust
macro_rules! recursive {
    () => {};
    ($head:expr) => {
        println!("{}", $head);
    };
    ($head:expr, $($tail:expr),*) => {
        println!("{}", $head);
        recursive!($($tail),*);
    };
}

fn main() {
    recursive!(1, 2, 3, 4, 5);
}
```

## 0x06 卫生宏

### 变量卫生

```rust
macro_rules! using_a {
    ($e:expr) => {
        {
            let a = 42;
            $e
        }
    };
}

fn main() {
    let a = 13;
    let b = using_a!(a + a); // 这里的 a 是外部的 a，不是宏内部的 a
    println!("b = {}", b); // 输出 26
}
```

### 生成标识符

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

## 0x07 过程宏

### 函数式过程宏

```rust
// 在 Cargo.toml 中添加:
// [lib]
// proc-macro = true

use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro]
pub fn make_answer(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

### 派生宏

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(AnswerFn)]
pub fn derive_answer_fn(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

### 属性宏

```rust
use proc_macro::TokenStream;

#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
    // attr 是属性参数，item 是被装饰的项
    item
}
```

## 0x08 常用宏示例

### 日志宏

```rust
macro_rules! log {
    ($level:expr, $($arg:tt)*) => {
        println!("[{}] {}", $level, format!($($arg)*));
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

### 测试宏

```rust
macro_rules! assert_eq_verbose {
    ($left:expr, $right:expr) => {
        let left_val = $left;
        let right_val = $right;
        
        if left_val != right_val {
            panic!("assertion failed: `(left == right)` \
                   \n  left: `{:?}`,\
                   \n right: `{:?}`", left_val, right_val);
        }
    };
}

fn main() {
    assert_eq_verbose!(1 + 1, 2);
    // assert_eq_verbose!(1 + 1, 3); // 会 panic
}
```

### 构建器宏

```rust
macro_rules! builder {
    ($name:ident { $($field:ident: $ty:ty),* }) => {
        struct $name {
            $($field: $ty),*
        }
        
        impl $name {
            fn new() -> Self {
                $name {
                    $($field: Default::default()),*
                }
            }
            
            $(
                fn $field(mut self, $field: $ty) -> Self {
                    self.$field = $field;
                    self
                }
            )*
        }
    };
}

builder!(Person {
    name: String,
    age: u32,
    email: String
});

fn main() {
    let person = Person::new()
        .name("Alice".to_string())
        .age(30)
        .email("alice@example.com".to_string());
    
    println!("Name: {}", person.name);
}
```

## 0x09 调试宏

### 展开宏查看

```bash
# 查看宏展开
cargo expand

# 使用 rustc 直接查看
rustc -Z unstable-options --pretty=expanded file.rs
```

### 调试技巧

```rust
macro_rules! debug_macro {
    ($($arg:tt)*) => {
        println!("Macro input: {}", stringify!($($arg)*));
        println!("Macro expansion: {}", $($arg)*);
    };
}

fn main() {
    debug_macro!(1 + 2);
}
```

## 0x10 最佳实践

### 1. 使用宏减少重复

```rust
// 不好：重复代码
fn add_i32(a: i32, b: i32) -> i32 { a + b }
fn add_i64(a: i64, b: i64) -> i64 { a + b }
fn add_f32(a: f32, b: f32) -> f32 { a + b }

// 好：使用宏
macro_rules! add {
    ($t:ty) => {
        fn add(a: $t, b: $t) -> $t {
            a + b
        }
    };
}

add!(i32);
add!(i64);
add!(f32);
```

### 2. 提供清晰的错误信息

```rust
macro_rules! ensure {
    ($condition:expr, $msg:expr) => {
        if !$condition {
            return Err($msg.into());
        }
    };
}

fn divide(a: f64, b: f64) -> Result<f64, String> {
    ensure!(b != 0.0, "Division by zero");
    Ok(a / b)
}
```

### 3. 遵循命名约定

```rust
// 使用 snake_case 命名宏
macro_rules! my_macro {
    () => {};
}

// 派生宏使用 PascalCase
#[derive(MyDerive)]
struct MyStruct;
```

## 参考

- [Rust 程序设计语言 - 宏](https://doc.rust-lang.org/book/ch19-06-macros.html)
- [Rust by Example - 宏](https://doc.rust-lang.org/rust-by-example/macros.html)
- [The Little Book of Rust Macros](https://veykril.github.io/tlborm/)
- [Rust Reference - 宏](https://doc.rust-lang.org/reference/macros.html)