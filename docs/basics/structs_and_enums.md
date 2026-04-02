# 结构体与枚举

## 0x01 结构体基础

### 定义结构体

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

### 创建结构体实例

```rust
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    
    println!("Username: {}", user1.username);
    println!("Email: {}", user1.email);
}
```

### 可变结构体

```rust
fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    
    user1.email = String::from("anotheremail@example.com");
    println!("Email: {}", user1.email);
}
```

### 结构体更新语法

```rust
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1  // 结构体更新语法
    };
    
    println!("Username: {}", user2.username);
}
```

## 0x02 结构体方法

### 定义方法

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };
    let rect3 = Rectangle { width: 60, height: 45 };
    
    println!("The area of the rectangle is {} square pixels.", rect1.area());
    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

### 关联函数

```rust
impl Rectangle {
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
    
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

fn main() {
    let rect1 = Rectangle::new(30, 50);
    let square = Rectangle::square(25);
    
    println!("Rectangle: {}x{}", rect1.width, rect1.height);
    println!("Square: {}x{}", square.width, square.height);
}
```

### 多个 impl 块

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn perimeter(&self) -> u32 {
        2 * (self.width + self.height)
    }
}

fn main() {
    let rect = Rectangle { width: 30, height: 50 };
    println!("Area: {}", rect.area());
    println!("Perimeter: {}", rect.perimeter());
}
```

## 0x03 元组结构体

### 定义元组结构体

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
    
    println!("Black: ({}, {}, {})", black.0, black.1, black.2);
    println!("Origin: ({}, {}, {})", origin.0, origin.1, origin.2);
}
```

## 0x04 单元结构体

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
    // 用于标记或占位
}
```

## 0x05 枚举基础

### 定义枚举

```rust
enum IpAddrKind {
    V4,
    V6,
}

fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
    
    route(four);
    route(six);
}

fn route(ip_kind: IpAddrKind) {
    match ip_kind {
        IpAddrKind::V4 => println!("IPv4"),
        IpAddrKind::V6 => println!("IPv6"),
    }
}
```

### 枚举携带数据

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

fn main() {
    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));
    
    match home {
        IpAddr::V4(a, b, c, d) => println!("{}.{}.{}.{}", a, b, c, d),
        IpAddr::V6(addr) => println!("{}", addr),
    }
}
```

### 复杂枚举

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        match self {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => println!("Move to ({}, {})", x, y),
            Message::Write(text) => println!("Text message: {}", text),
            Message::ChangeColor(r, g, b) => println!("Change color to ({}, {}, {})", r, g, b),
        }
    }
}

fn main() {
    let m = Message::Write(String::from("hello"));
    m.call();
}
```

## 0x06 Option 枚举

### 定义

```rust
enum Option<T> {
    Some(T),
    None,
}
```

### 使用示例

```rust
fn main() {
    let some_number = Some(5);
    let some_string = Some("a string");
    let absent_number: Option<i32> = None;
    
    match some_number {
        Some(n) => println!("Number: {}", n),
        None => println!("No number"),
    }
    
    // 使用 if let
    if let Some(s) = some_string {
        println!("String: {}", s);
    }
}
```

## 0x07 模式匹配

### match 表达式

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn main() {
    let coin = Coin::Penny;
    println!("Value: {}", value_in_cents(coin));
}
```

### 匹配 Option

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

fn main() {
    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
    
    println!("five: {:?}", five);
    println!("six: {:?}", six);
    println!("none: {:?}", none);
}
```

### 通配模式

```rust
fn main() {
    let dice_roll = 9;
    
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other), // 通配模式
    }
    
    // 不关心值的通配模式
    match dice_roll {
        3 => add_fancy_hat(),
        7 => reroll(),
        _ => (), // 忽略其他值
    }
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}
fn reroll() {}
```

## 0x08 if let 语法

```rust
fn main() {
    let config_max = Some(3u8);
    
    // 使用 match
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }
    
    // 使用 if let
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
    
    // if let 与 else
    let coin = Coin::Quarter;
    if let Coin::Quarter = coin {
        println!("State quarter!");
    } else {
        println!("Not a state quarter");
    }
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}
```

## 0x09 结构体与枚举组合

### 状态模式

```rust
struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
    
    fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
    
    fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(self)
    }
    
    fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
    
    fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
    
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
    
    fn approve(self: Box<Self>) -> Box<dyn State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
    
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
    
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}

fn main() {
    let mut post = Post::new();
    
    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());
    
    post.request_review();
    assert_eq!("", post.content());
    
    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

## 0x10 派生 trait

### 自动派生

```rust
#[derive(Debug, PartialEq, Clone)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = rect1.clone();
    
    println!("rect1: {:?}", rect1);
    println!("rect2: {:?}", rect2);
    println!("rect1 == rect2: {}", rect1 == rect2);
}
```

### 常用派生 trait

- `Debug`：用于调试输出
- `PartialEq` 和 `Eq`：相等比较
- `PartialOrd` 和 `Ord`：排序比较
- `Clone` 和 `Copy`：复制
- `Hash`：哈希计算
- `Default`：默认值
- `Display`：格式化输出

## 参考

- [Rust 程序设计语言 - 结构体](https://doc.rust-lang.org/book/ch05-01-structs.html)
- [Rust 程序设计语言 - 枚举](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html)
- [Rust by Example - 结构体](https://doc.rust-lang.org/rust-by-example/custom_types/structs.html)
- [Rust by Example - 枚举](https://doc.rust-lang.org/rust-by-example/custom_types/enum.html)