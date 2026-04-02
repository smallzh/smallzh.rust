# 泛型与特征

## 0x01 泛型基础

### 泛型函数

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];
    
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];
    
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
    
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
```

### 使用泛型重写

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    
    for item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("The largest number is {}", result);
    
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

## 0x02 泛型结构体

### 定义泛型结构体

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
    
    println!("integer: ({}, {})", integer.x, integer.y);
    println!("float: ({}, {})", float.x, float.y);
}
```

### 多个泛型参数

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
    
    println!("both_integer: ({}, {})", both_integer.x, both_integer.y);
    println!("both_float: ({}, {})", both_float.x, both_float.y);
    println!("integer_and_float: ({}, {})", integer_and_float.x, integer_and_float.y);
}
```

## 0x03 泛型枚举

### Option 和 Result

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}

fn main() {
    let some_number = Some(5);
    let some_string = Some("a string");
    
    let absent_number: Option<i32> = None;
    
    println!("some_number: {:?}", some_number);
    println!("some_string: {:?}", some_string);
    println!("absent_number: {:?}", absent_number);
}
```

## 0x04 泛型方法

### 在结构体上实现泛型方法

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };
    println!("p.x = {}", p.x());
}
```

### 为特定类型实现方法

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

fn main() {
    let p = Point { x: 5.0, y: 10.0 };
    println!("distance: {}", p.distance_from_origin());
}
```

### 泛型方法中的泛型参数

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };
    
    let p3 = p1.mixup(p2);
    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

## 0x05 特征（Traits）

### 定义特征

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

### 实现特征

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    };
    
    println!("1 new tweet: {}", tweet.summarize());
}
```

### 默认实现

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

impl Summary for NewsArticle {}
```

## 0x06 特征作为参数

### 使用 impl Trait 语法

```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

### 特征约束语法

```rust
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

### 多个特征约束

```rust
use std::fmt::Display;

pub fn notify(item: impl Summary + Display) {
    println!("Breaking news! {}", item.summarize());
}

// 或者使用 where 子句
pub fn notify<T>(item: T)
where
    T: Summary + Display,
{
    println!("Breaking news! {}", item.summarize());
}
```

## 0x07 返回实现了特征的类型

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

## 0x08 使用特征约束有条件地实现方法

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

## 0x09 特征对象

### 使用 trait 对象实现多态

```rust
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        println!("Drawing button: {}", self.label);
    }
}

pub struct SelectBox {
    pub width: u32,
    pub height: u32,
    pub options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        println!("Drawing select box with options: {:?}", self.options);
    }
}

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };
    
    screen.run();
}
```

### 特征对象的限制

```rust
// 不能在特征对象上使用泛型方法
trait Cloneable {
    fn clone(&self) -> Self; // 返回 Self 的方法不能用于特征对象
}

// 可以使用的设计
trait Printable {
    fn print(&self);
}

fn print_all(items: Vec<Box<dyn Printable>>) {
    for item in items {
        item.print();
    }
}
```

## 0x10 高级特征

### 关联类型

```rust
pub trait Iterator {
    type Item;
    
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;
    
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}

fn main() {
    let mut counter = Counter::new();
    
    while let Some(value) = counter.next() {
        println!("Value: {}", value);
    }
}
```

### 默认泛型类型参数

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;
    
    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

### 完全限定语法

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    
    person.fly(); // 调用 Human 自身的 fly 方法
    
    Pilot::fly(&person); // 调用 Pilot trait 的 fly 方法
    Wizard::fly(&person); // 调用 Wizard trait 的 fly 方法
}
```

### 父特征

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}

struct Point {
    x: i32,
    y: i32,
}

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

impl OutlinePrint for Point {}

fn main() {
    let p = Point { x: 1, y: 3 };
    p.outline_print();
}
```

## 参考

- [Rust 程序设计语言 - 泛型](https://doc.rust-lang.org/book/ch10-00-generics.html)
- [Rust 程序设计语言 - 特征](https://doc.rust-lang.org/book/ch10-02-traits.html)
- [Rust by Example - 泛型](https://doc.rust-lang.org/rust-by-example/generics.html)
- [Rust by Example - 特征](https://doc.rust-lang.org/rust-by-example/trait.html)