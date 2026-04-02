# 生命周期

## 0x01 什么是生命周期？

生命周期（Lifetimes）是 Rust 中引用有效性的概念。每个引用都有一个生命周期，即该引用保持有效的作用域。大多数情况下，生命周期是隐式且可推断的。当引用的生命周期可能以多种方式相关时，我们需要手动标注生命周期。

### 悬垂引用问题

```rust
fn main() {
    let r;
    {
        let x = 5;
        r = &x; // 错误：`x` 不够长寿
    }
    println!("r: {}", r);
}
```

编译器会报错，因为 `x` 在内部作用域结束时被销毁，但 `r` 仍然尝试引用它。

## 0x02 生命周期注解语法

生命周期注解不会改变引用的存活时间，只是描述多个引用生命周期之间的关系。

### 语法

生命周期参数名称必须以撇号（`'`）开头，通常使用小写字母，且通常很短：

```rust
&i32        // 引用
&'a i32     // 带有生命周期的引用
&'a mut i32 // 带有生命周期的可变引用
```

### 函数中的生命周期注解

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";
    
    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

## 0x03 生命周期规则

### 三条生命周期规则

1. **每个引用参数都有自己的生命周期参数**
2. **如果只有一个输入生命周期参数，该生命周期被赋予所有输出生命周期参数**
3. **如果方法有多个输入生命周期参数，但其中一个是 `&self` 或 `&mut self`，则 `self` 的生命周期被赋予所有输出生命周期参数**

### 示例

```rust
// 规则 1：每个引用参数都有自己的生命周期参数
fn first_word(s: &str) -> &str {  // 隐式生命周期
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}

// 显式生命周期
fn first_word_explicit<'a>(s: &'a str) -> &'a str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}
```

## 0x04 结构体中的生命周期

### 定义

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
    println!("First sentence: {}", i.part);
}
```

### 方法中的生命周期

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
    
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
    
    println!("Level: {}", i.level());
    println!("Part: {}", i.announce_and_return_part("Hello"));
}
```

## 0x05 静态生命周期

### `'static` 生命周期

`'static` 生命周期表示引用在程序的整个运行期间都有效：

```rust
fn main() {
    let s: &'static str = "I have a static lifetime.";
    println!("{}", s);
}
```

### 何时使用 `'static`

1. **字符串字面量**：所有字符串字面量都有 `'static` 生命周期
2. **全局常量**：需要在整个程序中有效的数据
3. **线程间共享数据**：使用 `Arc` 等智能指针时

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let data = Arc::new(vec![1, 2, 3]);
    
    let data_clone = Arc::clone(&data);
    let handle = thread::spawn(move || {
        println!("Data: {:?}", data_clone);
    });
    
    handle.join().unwrap();
}
```

## 0x06 生命周期与泛型

### 生命周期作为泛型参数

```rust
fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: std::fmt::Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";
    
    let result = longest_with_an_announcement(
        string1.as_str(),
        string2,
        "Today is someone's birthday!",
    );
    println!("The longest string is {}", result);
}
```

## 0x07 常见生命周期模式

### 模式 1：一个输入生命周期

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}
```

### 模式 2：多个输入生命周期

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### 模式 3：通过方法推断生命周期

```rust
struct StringHolder {
    data: String,
}

impl StringHolder {
    fn get_ref(&self) -> &String {
        &self.data
    }
}

fn main() {
    let holder = StringHolder { data: String::from("hello") };
    let reference = holder.get_ref();
    println!("Reference: {}", reference);
}
```

## 0x08 生命周期省略规则

### 规则 1：每个引用参数获得自己的生命周期参数

```rust
fn first_word(s: &str) -> &str  // 等价于
fn first_word<'a>(s: &'a str) -> &'a str
```

### 规则 2：如果只有一个输入生命周期参数，该生命周期被赋予所有输出生命周期参数

```rust
fn longest(x: &str, y: &str) -> &str  // 等价于
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str
```

### 规则 3：如果存在 `&self` 或 `&mut self`，`self` 的生命周期被赋予所有输出生命周期参数

```rust
impl ImportantExcerpt {
    fn announce_and_return_part(&self, announcement: &str) -> &str  // 等价于
    fn announce_and_return_part<'a>(&'a self, announcement: &'a str) -> &'a str
}
```

## 0x09 生命周期与 trait

### 定义带生命周期的 trait

```rust
trait Display<'a> {
    fn display(&self) -> &'a str;
}

struct MyStruct {
    data: String,
}

impl<'a> Display<'a> for MyStruct {
    fn display(&self) -> &'a str {
        &self.data
    }
}

fn main() {
    let s = MyStruct { data: String::from("hello") };
    println!("{}", s.display());
}
```

### 生命周期约束

```rust
use std::fmt::Display;

fn display_with_lifetime<'a, T>(x: &'a T) -> &'a T
where
    T: Display,
{
    println!("{}", x);
    x
}

fn main() {
    let x = 42;
    let y = display_with_lifetime(&x);
    println!("y = {}", y);
}
```

## 0x10 实际应用示例

### 示例 1：解析器

```rust
struct Parser<'a> {
    input: &'a str,
}

impl<'a> Parser<'a> {
    fn new(input: &'a str) -> Self {
        Parser { input }
    }
    
    fn parse(&self) -> Result<&'a str, &'static str> {
        if self.input.is_empty() {
            Err("Empty input")
        } else {
            Ok(self.input)
        }
    }
}

fn main() {
    let input = "hello world";
    let parser = Parser::new(input);
    match parser.parse() {
        Ok(result) => println!("Parsed: {}", result),
        Err(e) => println!("Error: {}", e),
    }
}
```

### 示例 2：缓存结构

```rust
use std::collections::HashMap;

struct Cache<'a, K, V> {
    data: HashMap<&'a K, &'a V>,
}

impl<'a, K, V> Cache<'a, K, V>
where
    K: std::hash::Hash + Eq,
{
    fn new() -> Self {
        Cache { data: HashMap::new() }
    }
    
    fn get(&self, key: &K) -> Option<&'a V> {
        self.data.get(key).copied()
    }
    
    fn set(&mut self, key: &'a K, value: &'a V) {
        self.data.insert(key, value);
    }
}

fn main() {
    let key = String::from("key");
    let value = String::from("value");
    
    let mut cache = Cache::new();
    cache.set(&key, &value);
    
    if let Some(v) = cache.get(&key) {
        println!("Found: {}", v);
    }
}
```

## 参考

- [Rust 程序设计语言 - 生命周期](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)
- [Rust by Example - 生命周期](https://doc.rust-lang.org/rust-by-example/scope/lifetime.html)
- [Rust Reference - 生命周期](https://doc.rust-lang.org/reference/types/lifetime.html)
- [Rust Nomicon - 生命周期](https://doc.rust-lang.org/nomicon/lifetimes.html)