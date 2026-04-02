# 模式匹配

## 0x01 模式匹配基础

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
        Coin::Penny => 1,
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

### match 的穷尽性

```rust
fn main() {
    let dice_roll = 9;
    
    match dice_roll {
        3 => println!("Three"),
        7 => println!("Seven"),
        _ => println!("Other"), // 必须处理所有情况
    }
}
```

## 0x02 模式类型

### 字面量模式

```rust
fn main() {
    let number = 13;
    
    match number {
        1 => println!("One"),
        2 | 3 | 5 | 7 | 11 | 13 => println!("Prime"),
        13..=19 => println!("Teen"),
        _ => println!("Other"),
    }
}
```

### 命名变量模式

```rust
fn main() {
    let x = Some(5);
    let y = 10;
    
    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y), // 这里的 y 是新的变量
        _ => println!("Default, x = {:?}", x),
    }
    
    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

### 多重模式

```rust
fn main() {
    let x = 1;
    
    match x {
        1 | 2 => println!("One or two"),
        3 => println!("Three"),
        _ => println!("Anything"),
    }
}
```

### 范围模式

```rust
fn main() {
    let x = 5;
    
    match x {
        1..=5 => println!("One through five"),
        _ => println!("Something else"),
    }
    
    let c = 'c';
    
    match c {
        'a'..='j' => println!("Early ASCII letter"),
        'k'..='z' => println!("Late ASCII letter"),
        _ => println!("Something else"),
    }
}
```

## 0x03 解构模式

### 解构结构体

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };
    
    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
    
    // 使用别名
    match p {
        Point { x: x_val, y: y_val } => {
            println!("x: {}, y: {}", x_val, y_val);
        }
    }
}
```

### 解构枚举

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);
    
    match msg {
        Message::Quit => {
            println!("Quit");
        }
        Message::Move { x, y } => {
            println!("Move to x: {}, y: {}", x, y);
        }
        Message::Write(text) => {
            println!("Text message: {}", text);
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change the color to red {}, green {}, and blue {}", r, g, b);
        }
    }
}
```

### 解构元组

```rust
fn main() {
    let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
    
    println!("Feet: {}, Inches: {}", feet, inches);
    println!("x: {}, y: {}", x, y);
}
```

### 解构数组

```rust
fn main() {
    let arr = [1, 2, 3, 4, 5];
    
    match arr {
        [1, _, _, _, _] => println!("Starts with one"),
        [first, .., last] => println!("First: {}, Last: {}", first, last),
        _ => println!("Other"),
    }
    
    // 使用 .. 忽略多个值
    let arr = [1, 2, 3, 4, 5];
    match arr {
        [first, middle @ .., last] => {
            println!("First: {}", first);
            println!("Middle: {:?}", middle);
            println!("Last: {}", last);
        }
    }
}
```

## 0x04 模式守卫

### 使用 if 在模式中添加条件

```rust
fn main() {
    let num = Some(4);
    
    match num {
        Some(x) if x < 5 => println!("Less than five: {}", x),
        Some(x) => println!("{}", x),
        None => (),
    }
}
```

### 模式守卫中访问外部变量

```rust
fn main() {
    let x = Some(5);
    let y = 10;
    
    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {:?}", n),
        _ => println!("Default, x = {:?}", x),
    }
    
    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

## 0x05 @ 绑定

### 使用 @ 创建变量

```rust
enum Message {
    Hello { id: i32 },
}

fn main() {
    let msg = Message::Hello { id: 5 };
    
    match msg {
        Message::Hello { id: id_variable @ 3..=7 } => {
            println!("Found an id in range: {}", id_variable)
        }
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => {
            println!("Found some other id: {}", id)
        }
    }
}
```

### 在解构中使用 @

```rust
fn main() {
    let x = 5;
    
    match x {
        e @ 1..=5 => println!("Got a range element: {}", e),
        _ => println!("Anything"),
    }
}
```

## 0x06 忽略模式

### 忽略整个值 _

```rust
fn main() {
    let _x = 5; // 忽略未使用的变量
    let y = 10;
    
    let s = Some(String::from("Hello"));
    
    if let Some(_) = s {
        println!("found a string");
    }
    
    println!("{:?}", s); // s 仍然有效
}
```

### 忽略部分值

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);
    
    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {}, {}, {}", first, third, fifth)
        }
    }
}
```

### 忽略剩余值 ..

```rust
fn main() {
    struct Point {
        x: i32,
        y: i32,
        z: i32,
    }
    
    let origin = Point { x: 0, y: 0, z: 0 };
    
    match origin {
        Point { x, .. } => println!("x is {}", x),
    }
    
    let numbers = (2, 4, 8, 16, 32);
    
    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        }
    }
}
```

## 0x07 匹配引用

### 解引用

```rust
fn main() {
    let robot_name = Some(String::from("Bors"));
    
    match robot_name {
        Some(name) => println!("Found: {}", name),
        None => println!("No name found"),
    }
    
    // robot_name 在这里被移动了
    // println!("{:?}", robot_name); // 错误
}
```

### 使用 ref 和 ref mut

```rust
fn main() {
    let robot_name = Some(String::from("Bors"));
    
    match robot_name {
        Some(ref name) => println!("Found: {}", name),
        None => println!("No name found"),
    }
    
    println!("{:?}", robot_name); // 正确
    
    let mut robot_name = Some(String::from("Bors"));
    
    match robot_name {
        Some(ref mut name) => *name = String::from("Another name"),
        None => (),
    }
    
    println!("{:?}", robot_name);
}
```

## 0x08 if let 和 while let

### if let

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();
    
    if let Some(color) = favorite_color {
        println!("Using your favorite color, {}, as the background", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

### while let

```rust
fn main() {
    let mut stack = Vec::new();
    
    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
}
```

## 0x09 for 循环中的模式

```rust
fn main() {
    let v = vec!['a', 'b', 'c'];
    
    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
    
    // 解构元组
    let points = vec![(1, 2), (3, 4), (5, 6)];
    for (x, y) in points {
        println!("Point: ({}, {})", x, y);
    }
}
```

## 0x10 let 语句中的模式

```rust
fn main() {
    let (x, y, z) = (1, 2, 3);
    println!("x: {}, y: {}, z: {}", x, y, z);
    
    // 解构结构体
    struct Point {
        x: i32,
        y: i32,
    }
    
    let point = Point { x: 0, y: 7 };
    let Point { x: a, y: b } = point;
    println!("a: {}, b: {}", a, b);
    
    // 使用模式守卫
    let x = 5;
    match x {
        n if n < 0 => println!("Negative"),
        n if n > 0 => println!("Positive"),
        _ => println!("Zero"),
    }
}
```

## 0x11 函数参数中的模式

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
    
    // 闭包中的模式
    let closure = |&(x, y): &(i32, i32)| {
        println!("Closure location: ({}, {})", x, y);
    };
    
    closure(&point);
}
```

## 参考

- [Rust 程序设计语言 - 模式匹配](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html)
- [Rust by Example - 模式匹配](https://doc.rust-lang.org/rust-by-example/flow_control/match.html)
- [Rust Reference - 模式](https://doc.rust-lang.org/reference/patterns.html)
- [Rust by Example - 解构](https://doc.rust-lang.org/rust-by-example/flow_control/match/destructuring.html)