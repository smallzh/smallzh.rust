# 所有权与借用

## 0x01 所有权规则

Rust 的所有权系统是其核心特性，它在编译时保证内存安全。所有权有三条基本规则：

1. **每个值都有一个变量，这个变量是该值的所有者**
2. **每个值同时只能有一个所有者**
3. **当所有者离开作用域，值将被丢弃**

### 变量作用域

```rust
fn main() {
    {                      // s 在这里无效，尚未声明
        let s = "hello";   // s 从此处开始有效
        println!("{}", s); // 可以使用 s
    }                      // 作用域结束，s 不再有效
}
```

### 内存与分配

```rust
fn main() {
    let s1 = String::from("hello"); // 在堆上分配内存
    let s2 = s1;                    // s1 的所有权转移到 s2
    
    // println!("{}", s1);          // 错误：s1 已无效
    println!("{}", s2);             // 正确
}
```

## 0x02 移动（Move）

### 字符串的移动

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // s1 被移动到 s2，s1 无效
    
    // println!("{}", s1); // 编译错误
    println!("{}", s2);
}
```

### 函数传参的移动

```rust
fn main() {
    let s = String::from("hello");
    takes_ownership(s);        // s 的所有权转移到函数
    // println!("{}", s);      // 错误：s 已无效
    
    let x = 5;
    makes_copy(x);             // x 是 i32，实现了 Copy trait
    println!("{}", x);         // 正确：x 仍然有效
}

fn takes_ownership(some_string: String) {
    println!("{}", some_string);
} // some_string 离开作用域，内存被释放

fn makes_copy(some_integer: i32) {
    println!("{}", some_integer);
} // some_integer 离开作用域，没有特殊操作
```

### 返回值的所有权

```rust
fn main() {
    let s1 = gives_ownership();         // 获取返回值的所有权
    let s2 = String::from("hello");
    let s3 = takes_and_gives_back(s2);  // s2 被移动，s3 获取所有权
    
    println!("s1 = {}, s3 = {}", s1, s3);
}

fn gives_ownership() -> String {
    let some_string = String::from("yours");
    some_string  // 返回并转移所有权
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string  // 返回并转移所有权
}
```

## 0x03 引用与借用

### 引用（References）

引用允许你使用值但不获取所有权：

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // 传递引用
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
} // s 离开作用域，但因为它不拥有值，所以不会释放内存
```

### 可变引用

可变引用允许你修改引用的值：

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
    println!("{}", s); // 输出 "hello, world"
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

### 引用规则

1. **在任意给定时间，要么只能有一个可变引用，要么只能有多个不可变引用**
2. **引用必须总是有效的**

```rust
fn main() {
    let mut s = String::from("hello");
    
    let r1 = &s; // 没问题
    let r2 = &s; // 没问题
    println!("{} and {}", r1, r2);
    // 此位置之后 r1 和 r2 不再使用
    
    let r3 = &mut s; // 没问题
    println!("{}", r3);
    
    // 下面的代码会编译错误
    // let r1 = &s;
    // let r2 = &mut s; // 错误：不能同时有可变引用和不可变引用
    // println!("{}, {}", r1, r2);
}
```

### 悬垂引用

Rust 编译器确保引用永远不会变成悬垂引用：

```rust
fn main() {
    // let reference_to_nothing = dangle();
    let reference_to_something = no_dangle();
    println!("{}", reference_to_something);
}

// 下面的代码会编译错误
// fn dangle() -> &String {
//     let s = String::from("hello");
//     &s // 返回字符串 s 的引用
// } // s 离开作用域并被释放，引用无效

fn no_dangle() -> String {
    let s = String::from("hello");
    s // 返回所有权
}
```

## 0x04 切片类型

### 字符串切片

```rust
fn main() {
    let s = String::from("hello world");
    
    let hello = &s[0..5];
    let world = &s[6..11];
    
    println!("hello = {}, world = {}", hello, world);
    
    // 简写形式
    let s = String::from("hello");
    let slice = &s[..2];      // 从开始到索引 2
    let slice = &s[3..];      // 从索引 3 到结束
    let slice = &s[..];       // 整个字符串
    
    println!("slice = {}", slice);
}
```

### 字符串字面量就是切片

```rust
fn main() {
    let s: &str = "Hello, world!"; // 字符串字面量是 &str 类型
    println!("{}", s);
}
```

### 函数中的字符串切片

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    
    &s[..]
}

fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);
    println!("The first word is: {}", word);
    
    // s.clear(); // 错误：不能在存在不可变引用时修改字符串
}
```

### 数组切片

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
    let slice = &a[1..3];
    
    println!("slice = {:?}", slice); // [2, 3]
}
```

## 0x05 所有权与函数

### 函数参数的所有权

```rust
fn main() {
    let s = String::from("hello");
    takes_ownership(s);  // s 的所有权转移到函数
    // println!("{}", s); // 错误：s 已无效
    
    let x = 5;
    makes_copy(x);       // x 是 i32，复制到函数
    println!("x is still valid: {}", x);
}

fn takes_ownership(some_string: String) {
    println!("{}", some_string);
}

fn makes_copy(some_integer: i32) {
    println!("{}", some_integer);
}
```

### 函数返回值的所有权

```rust
fn main() {
    let s1 = gives_ownership();
    let s2 = String::from("hello");
    let s3 = takes_and_gives_back(s2);
    
    println!("s1 = {}, s3 = {}", s1, s3);
}

fn gives_ownership() -> String {
    String::from("yours")
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string
}
```

### 引用作为参数

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

## 0x06 Copy 与 Clone

### Copy trait

如果一个类型实现了 `Copy` trait，那么一个旧的变量在赋值后仍然可用：

```rust
fn main() {
    let x = 5;
    let y = x;  // x 被复制
    println!("x = {}, y = {}", x, y); // 两个都有效
}
```

实现了 `Copy` 的类型：
- 所有整数类型
- 布尔类型
- 所有浮点类型
- 字符类型
- 元组（如果所有元素都实现了 `Copy`）

### Clone trait

对于堆上分配的数据，可以使用 `clone` 进行深拷贝：

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone(); // 深拷贝
    
    println!("s1 = {}, s2 = {}", s1, s2);
}
```

## 0x07 所有权模式

### 结构体的所有权

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

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
    
    // user1.username 已被移动到 user2
    // println!("{}", user1.username); // 错误
    println!("{}", user2.username);
}
```

### 枚举的所有权

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::Write(String::from("hello"));
    
    match msg {
        Message::Write(text) => println!("Text: {}", text),
        _ => println!("Other message"),
    }
}
```

### 集合的所有权

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v2 = v1; // v1 被移动
    
    // println!("{:?}", v1); // 错误
    println!("{:?}", v2);
    
    // 使用引用避免移动
    let v1 = vec![1, 2, 3];
    let v2 = &v1;
    println!("{:?} and {:?}", v1, v2);
}
```

## 0x08 生命周期

### 生命周期注解语法

```rust
fn main() {
    let string1 = String::from("abcd");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### 结构体中的生命周期

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

## 参考

- [Rust 程序设计语言 - 所有权](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)
- [Rust 程序设计语言 - 引用与借用](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)
- [Rust by Example - 所有权](https://doc.rust-lang.org/rust-by-example/scope/ownership.html)
- [Rust by Example - 借用](https://doc.rust-lang.org/rust-by-example/scope/borrow.html)