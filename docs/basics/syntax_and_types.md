# 基本语法与数据类型

## 0x01 变量与可变性

### 变量绑定

在 Rust 中，使用 `let` 关键字创建变量绑定：

```rust
fn main() {
    let x = 5;
    let y: i32 = 10;
    let z = x + y;
    
    println!("x = {}, y = {}, z = {}", x, y, z);
}
```

### 可变性

默认情况下，变量是不可变的（immutable）。使用 `mut` 关键字使变量可变：

```rust
fn main() {
    let mut x = 5;
    println!("x = {}", x);
    
    x = 10; // 可以修改，因为 x 是 mut
    println!("x = {}", x);
    
    // 下面的代码会编译错误
    // let y = 5;
    // y = 10; // 错误：不能修改不可变变量
}
```

### 变量遮蔽（Shadowing）

可以使用相同的名字声明新的变量，新变量会遮蔽之前的变量：

```rust
fn main() {
    let x = 5;
    let x = x + 1; // 遮蔽
    let x = x * 2; // 再次遮蔽
    
    println!("x = {}", x); // 输出 12
    
    // 遮蔽可以改变类型
    let spaces = "   ";      // &str 类型
    let spaces = spaces.len(); // usize 类型
    
    println!("spaces = {}", spaces);
}
```

### 常量

使用 `const` 关键字声明常量，必须注明类型，只能被设置为常量表达式：

```rust
const MAX_POINTS: u32 = 100_000;
const PI: f64 = 3.141592653589793;

fn main() {
    println!("MAX_POINTS = {}", MAX_POINTS);
    println!("PI = {}", PI);
}
```

## 0x02 基本数据类型

### 标量类型

#### 整数类型

| 类型 | 大小 | 范围 |
|------|------|------|
| i8 | 8 位 | -128 到 127 |
| u8 | 8 位 | 0 到 255 |
| i16 | 16 位 | -32768 到 32767 |
| u16 | 16 位 | 0 到 65535 |
| i32 | 32 位 | -2^31 到 2^31-1 |
| u32 | 32 位 | 0 到 2^32-1 |
| i64 | 64 位 | -2^63 到 2^63-1 |
| u64 | 64 位 | 0 到 2^64-1 |
| i128 | 128 位 | -2^127 到 2^127-1 |
| u128 | 128 位 | 0 到 2^128-1 |
| isize | 指针大小 | 取决于架构 |
| usize | 指针大小 | 取决于架构 |

```rust
fn main() {
    let a: i32 = 98_222;    // 十进制
    let b: i32 = 0xff;      // 十六进制
    let c: i32 = 0o77;      // 八进制
    let d: i32 = 0b1111_0000; // 二进制
    let e: u8 = b'A';       // 字节（仅限 u8）
    
    println!("a = {}, b = {}, c = {}, d = {}, e = {}", a, b, c, d, e);
}
```

#### 浮点类型

```rust
fn main() {
    let x: f64 = 2.0;      // f64（默认）
    let y: f32 = 3.0;      // f32
    
    println!("x = {}, y = {}", x, y);
    
    // 数学运算
    let sum = 5.0 + 10.0;
    let difference = 95.5 - 4.3;
    let product = 4.0 * 30.0;
    let quotient = 56.7 / 32.2;
    let remainder = 43.0 % 5.0;
    
    println!("sum = {}, difference = {}, product = {}, quotient = {}, remainder = {}", 
             sum, difference, product, quotient, remainder);
}
```

#### 布尔类型

```rust
fn main() {
    let t = true;
    let f: bool = false; // 显式类型标注
    
    println!("t = {}, f = {}", t, f);
    
    // 布尔运算
    let and = true && false;
    let or = true || false;
    let not = !true;
    
    println!("and = {}, or = {}, not = {}", and, or, not);
}
```

#### 字符类型

```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ';       // Unicode 标量值
    let heart_eyed_cat = '😻';
    
    println!("c = {}, z = {}, heart_eyed_cat = {}", c, z, heart_eyed_cat);
    
    // 字符占 4 个字节
    let size = std::mem::size_of::<char>();
    println!("char 大小: {} 字节", size);
}
```

### 复合类型

#### 元组（Tuple）

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    
    // 解构
    let (x, y, z) = tup;
    println!("x = {}, y = {}, z = {}", x, y, z);
    
    // 使用点号访问
    let five_hundred = tup.0;
    let six_point_four = tup.1;
    let one = tup.2;
    
    println!("five_hundred = {}, six_point_four = {}, one = {}", 
             five_hundred, six_point_four, one);
    
    // 空元组（unit）
    let unit = ();
    println!("unit = {:?}", unit);
}
```

#### 数组（Array）

```rust
fn main() {
    let a = [1, 2, 3, 4, 5]; // 类型推断
    let b: [i32; 5] = [1, 2, 3, 4, 5]; // 显式类型
    let c = [3; 5]; // [3, 3, 3, 3, 3]
    
    println!("a = {:?}", a);
    println!("b = {:?}", b);
    println!("c = {:?}", c);
    
    // 访问数组元素
    let first = a[0];
    let second = a[1];
    
    println!("first = {}, second = {}", first, second);
    
    // 数组长度
    let length = a.len();
    println!("数组长度: {}", length);
    
    // 数组切片
    let slice = &a[1..3];
    println!("切片: {:?}", slice);
}
```

## 0x03 运算符

### 算术运算符

```rust
fn main() {
    let a = 10;
    let b = 3;
    
    println!("a + b = {}", a + b);   // 13
    println!("a - b = {}", a - b);   // 7
    println!("a * b = {}", a * b);   // 30
    println!("a / b = {}", a / b);   // 3
    println!("a % b = {}", a % b);   // 1
    
    // 整数除法会截断
    let c = 10.0;
    let d = 3.0;
    println!("c / d = {}", c / d);   // 3.333...
}
```

### 比较运算符

```rust
fn main() {
    let a = 10;
    let b = 20;
    
    println!("a == b: {}", a == b);  // false
    println!("a != b: {}", a != b);  // true
    println!("a < b: {}", a < b);    // true
    println!("a > b: {}", a > b);    // false
    println!("a <= b: {}", a <= b);  // true
    println!("a >= b: {}", a >= b);  // false
}
```

### 逻辑运算符

```rust
fn main() {
    let a = true;
    let b = false;
    
    println!("a && b: {}", a && b);  // false
    println!("a || b: {}", a || b);  // true
    println!("!a: {}", !a);          // false
    println!("!b: {}", !b);          // true
}
```

### 位运算符

```rust
fn main() {
    let a: u8 = 0b1100;  // 12
    let b: u8 = 0b1010;  // 10
    
    println!("a & b: {:08b}", a & b);   // 0b1000 (8)
    println!("a | b: {:08b}", a | b);   // 0b1110 (14)
    println!("a ^ b: {:08b}", a ^ b);   // 0b0110 (6)
    println!("!a: {:08b}", !a);         // 0b11110011
    println!("a << 2: {:08b}", a << 2); // 0b110000 (48)
    println!("a >> 2: {:08b}", a >> 2); // 0b0011 (3)
}
```

## 0x04 控制流

### if 表达式

```rust
fn main() {
    let number = 3;
    
    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
    
    // if 是表达式
    let condition = true;
    let value = if condition { 5 } else { 6 };
    println!("value = {}", value);
    
    // 多条件
    let number = 6;
    
    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

### 循环

#### loop 循环

```rust
fn main() {
    let mut counter = 0;
    
    let result = loop {
        counter += 1;
        
        if counter == 10 {
            break counter * 2; // 返回值
        }
    };
    
    println!("The result is {}", result); // 20
}
```

#### while 循环

```rust
fn main() {
    let mut number = 3;
    
    while number != 0 {
        println!("{}!", number);
        number -= 1;
    }
    
    println!("LIFTOFF!!!");
}
```

#### for 循环

```rust
fn main() {
    // 遍历数组
    let a = [10, 20, 30, 40, 50];
    
    for element in a.iter() {
        println!("the value is: {}", element);
    }
    
    // 范围循环
    for number in 1..4 {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
    
    // 反向范围
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

### match 表达式

```rust
fn main() {
    let number = 13;
    
    match number {
        1 => println!("One!"),
        2 | 3 | 5 | 7 | 11 | 13 => println!("This is a prime"),
        13..=19 => println!("A teen"),
        _ => println!("Ain't special"),
    }
    
    // match 是表达式
    let boolean = true;
    let binary = match boolean {
        false => 0,
        true => 1,
    };
    
    println!("binary = {}", binary);
}
```

## 0x05 函数

### 函数定义

```rust
fn main() {
    println!("Hello from main!");
    another_function();
    function_with_value(5);
    let result = function_with_return(3, 4);
    println!("3 + 4 = {}", result);
}

fn another_function() {
    println!("Hello from another_function!");
}

fn function_with_value(x: i32) {
    println!("The value of x is: {}", x);
}

fn function_with_return(x: i32, y: i32) -> i32 {
    x + y // 表达式作为返回值（没有分号）
}
```

### 语句与表达式

```rust
fn main() {
    // 语句：不返回值
    let y = 6; // 语句
    
    // 表达式：返回值
    let y = {
        let x = 3;
        x + 1 // 表达式（没有分号）
    };
    
    println!("The value of y is: {}", y);
}
```

### 函数指针

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);
    println!("The answer is: {}", answer); // 12
}
```

## 0x06 注释

### 行注释

```rust
// 这是行注释
fn main() {
    let x = 5; // 这也是行注释
    println!("x = {}", x);
}
```

### 文档注释

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, add_one(5));
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}

fn main() {
    println!("add_one(5) = {}", add_one(5));
}
```

## 0x07 打印输出

### println! 宏

```rust
fn main() {
    // 基本用法
    println!("Hello, world!");
    
    // 格式化参数
    println!("{} days", 31);
    
    // 位置参数
    println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob");
    
    // 命名参数
    println!("{subject} {verb} {object}",
             object="the lazy dog",
             subject="the quick brown fox",
             verb="jumps over");
    
    // 特殊格式
    println!("Base 10:               {}",   69420);
    println!("Base 2 (binary):       {:b}", 69420);
    println!("Base 8 (octal):        {:o}", 69420);
    println!("Base 16 (hexadecimal): {:x}", 69420);
    println!("Base 16 (hexadecimal): {:X}", 69420);
    
    // 宽度和对齐
    println!("{number:>5}", number=1);
    println!("{number:0>5}", number=1);
    println!("{number:0<5}", number=1);
    
    // 捕获环境变量
    let number: f64 = 1.0;
    let width: usize = 5;
    println!("{number:>width$}");
}
```

### print! 宏

```rust
fn main() {
    print!("This will not have a newline at the end");
    println!("This will have a newline at the end");
}
```

### eprintln! 宏

```rust
fn main() {
    eprintln!("This will be printed to stderr");
}
```

## 参考

- [Rust 程序设计语言 - 基本数据类型](https://doc.rust-lang.org/book/ch03-02-data-types.html)
- [Rust 程序设计语言 - 控制流](https://doc.rust-lang.org/book/ch03-05-control-flow.html)
- [Rust by Example - 原生类型](https://doc.rust-lang.org/rust-by-example/primitives.html)
- [Rust by Example - 流程控制](https://doc.rust-lang.org/rust-by-example/flow_control.html)