# 闭包与迭代器

## 0x01 闭包基础

### 闭包定义

闭包是可以捕获其环境的匿名函数：

```rust
fn main() {
    // 基本闭包
    let closure = || println!("Hello from closure!");
    closure();
    
    // 带参数的闭包
    let add = |a: i32, b: i32| -> i32 { a + b };
    println!("add(2, 3) = {}", add(2, 3));
    
    // 类型推断
    let add = |a, b| a + b;
    println!("add(2, 3) = {}", add(2, 3));
}
```

### 闭包捕获环境

```rust
fn main() {
    let x = 4;
    
    // 不可变借用
    let equal_to_x = |z| z == x;
    let y = 4;
    assert!(equal_to_x(y));
    
    // 可变借用
    let mut x = 4;
    let mut change_x = |z| x = z;
    change_x(5);
    println!("x = {}", x);
    
    // 获取所有权
    let x = vec![1, 2, 3];
    let equal_to_x = move |z| z == x;
    // println!("x = {:?}", x); // 错误：x 被移动
    let y = vec![1, 2, 3];
    assert!(equal_to_x(y));
}
```

## 0x02 闭包作为参数

### Fn trait

```rust
fn apply<F>(f: F)
where
    F: Fn(),
{
    f();
}

fn main() {
    let message = "Hello";
    let closure = || println!("{}", message);
    apply(closure);
}
```

### FnMut trait

```rust
fn apply_mut<F>(mut f: F)
where
    F: FnMut(),
{
    f();
}

fn main() {
    let mut count = 0;
    let mut closure = || {
        count += 1;
        println!("count = {}", count);
    };
    
    apply_mut(&mut closure);
    apply_mut(&mut closure);
}
```

### FnOnce trait

```rust
fn apply_once<F>(f: F)
where
    F: FnOnce(),
{
    f();
}

fn main() {
    let x = vec![1, 2, 3];
    let closure = move || {
        println!("x = {:?}", x);
    };
    
    apply_once(closure);
    // apply_once(closure); // 错误：闭包已被消费
}
```

## 0x03 闭包作为返回值

```rust
fn create_adder(x: i32) -> impl Fn(i32) -> i32 {
    move |y| x + y
}

fn main() {
    let add_5 = create_adder(5);
    println!("add_5(3) = {}", add_5(3));
    println!("add_5(10) = {}", add_5(10));
}
```

## 0x04 迭代器基础

### 创建迭代器

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    
    // 创建迭代器
    let v1_iter = v1.iter();
    
    // 使用迭代器
    for val in v1_iter {
        println!("Got: {}", val);
    }
    
    // 消费迭代器
    let v1 = vec![1, 2, 3];
    let total: i32 = v1.iter().sum();
    println!("total = {}", total);
}
```

### 迭代器方法

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];
    
    // map
    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
    println!("v2 = {:?}", v2);
    
    // filter
    let v3: Vec<_> = v1.iter().filter(|x| *x % 2 == 0).collect();
    println!("v3 = {:?}", v3);
    
    // enumerate
    let v1 = vec![10, 20, 30];
    for (i, val) in v1.iter().enumerate() {
        println!("index: {}, value: {}", i, val);
    }
}
```

## 0x05 消费者适配器

### sum

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let total: i32 = v1.iter().sum();
    println!("total = {}", total);
}
```

### collect

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v2: Vec<_> = v1.iter().map(|x| x * 2).collect();
    println!("v2 = {:?}", v2);
    
    // 转换为其他集合
    use std::collections::HashSet;
    let set: HashSet<_> = v1.iter().map(|x| x * 2).collect();
    println!("set = {:?}", set);
}
```

### fold

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    
    let sum = v.iter().fold(0, |acc, x| acc + x);
    println!("sum = {}", sum);
    
    let product = v.iter().fold(1, |acc, x| acc * x);
    println!("product = {}", product);
}
```

### for_each

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    
    v.iter().for_each(|x| {
        println!("x = {}", x);
    });
}
```

## 0x06 迭代器适配器

### map

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];
    
    let v2: Vec<_> = v1.iter().map(|x| x * 2).collect();
    println!("v2 = {:?}", v2);
    
    // 链式调用
    let v3: Vec<_> = v1.iter()
        .map(|x| x * 2)
        .map(|x| x + 1)
        .collect();
    println!("v3 = {:?}", v3);
}
```

### filter

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    
    let even: Vec<_> = v.iter().filter(|x| *x % 2 == 0).collect();
    println!("even = {:?}", even);
    
    // 复杂条件
    let large_even: Vec<_> = v.iter()
        .filter(|x| *x % 2 == 0)
        .filter(|x| *x > &2)
        .collect();
    println!("large_even = {:?}", large_even);
}
```

### flat_map

```rust
fn main() {
    let v = vec![vec![1, 2], vec![3, 4], vec![5, 6]];
    
    let flat: Vec<_> = v.iter()
        .flat_map(|inner| inner.iter())
        .collect();
    println!("flat = {:?}", flat);
    
    // 字符串分割
    let sentences = vec!["hello world", "rust is great"];
    let words: Vec<_> = sentences.iter()
        .flat_map(|s| s.split_whitespace())
        .collect();
    println!("words = {:?}", words);
}
```

### take 和 skip

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    
    let first_three: Vec<_> = v.iter().take(3).collect();
    println!("first_three = {:?}", first_three);
    
    let skip_two: Vec<_> = v.iter().skip(2).collect();
    println!("skip_two = {:?}", skip_two);
}
```

### zip

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v2 = vec!['a', 'b', 'c'];
    
    let zipped: Vec<_> = v1.iter().zip(v2.iter()).collect();
    println!("zipped = {:?}", zipped);
    
    // 解构
    for (num, ch) in v1.iter().zip(v2.iter()) {
        println!("{}: {}", num, ch);
    }
}
```

## 0x07 自定义迭代器

### 实现 Iterator trait

```rust
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
        println!("value = {}", value);
    }
    
    // 使用迭代器方法
    let sum: u32 = Counter::new()
        .zip(Counter::new().skip(1))
        .map(|(a, b)| a * b)
        .filter(|x| x % 3 == 0)
        .sum();
    
    println!("sum = {}", sum);
}
```

## 0x08 性能考虑

### 零成本抽象

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    
    // 迭代器链：编译器优化为高效循环
    let sum: i32 = v.iter()
        .map(|x| x * 2)
        .filter(|x| x % 3 == 0)
        .sum();
    
    println!("sum = {}", sum);
    
    // 等价于手写循环
    let mut sum = 0;
    for x in &v {
        let doubled = x * 2;
        if doubled % 3 == 0 {
            sum += doubled;
        }
    }
    println!("sum = {}", sum);
}
```

## 0x09 实际应用示例

### 示例 1：数据处理管道

```rust
#[derive(Debug)]
struct Record {
    name: String,
    age: u32,
    city: String,
}

fn main() {
    let records = vec![
        Record { name: "Alice".to_string(), age: 25, city: "Beijing".to_string() },
        Record { name: "Bob".to_string(), age: 30, city: "Shanghai".to_string() },
        Record { name: "Charlie".to_string(), age: 35, city: "Beijing".to_string() },
        Record { name: "Diana".to_string(), age: 28, city: "Shanghai".to_string() },
    ];
    
    // 统计北京用户的平均年龄
    let beijing_avg_age: f64 = records.iter()
        .filter(|r| r.city == "Beijing")
        .map(|r| r.age as f64)
        .sum::<f64>() / records.iter().filter(|r| r.city == "Beijing").count() as f64;
    
    println!("Beijing average age: {:.2}", beijing_avg_age);
    
    // 获取年龄大于 30 的用户名
    let older_names: Vec<_> = records.iter()
        .filter(|r| r.age > 30)
        .map(|r| &r.name)
        .collect();
    
    println!("Users over 30: {:?}", older_names);
}
```

### 示例 2：文件处理

```rust
use std::fs;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let content = fs::read_to_string("data.txt")?;
    
    // 统计单词频率
    let word_freq = content.split_whitespace()
        .map(|word| word.to_lowercase())
        .fold(std::collections::HashMap::new(), |mut map, word| {
            *map.entry(word).or_insert(0) += 1;
            map
        });
    
    // 获取频率最高的 5 个单词
    let mut freq_vec: Vec<_> = word_freq.into_iter().collect();
    freq_vec.sort_by(|a, b| b.1.cmp(&a.1));
    
    let top_5: Vec<_> = freq_vec.iter().take(5).collect();
    
    println!("Top 5 words:");
    for (word, count) in top_5 {
        println!("{}: {}", word, count);
    }
    
    Ok(())
}
```

### 示例 3：并行处理

```rust
use std::thread;

fn main() {
    let data = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // 分块处理
    let chunk_size = 3;
    let handles: Vec<_> = data.chunks(chunk_size)
        .map(|chunk| {
            let chunk = chunk.to_vec();
            thread::spawn(move || {
                chunk.iter().sum::<i32>()
            })
        })
        .collect();
    
    let total: i32 = handles.into_iter()
        .map(|h| h.join().unwrap())
        .sum();
    
    println!("Total: {}", total);
}
```

## 0x10 常见模式

### 链式调用

```rust
fn main() {
    let result = (1..=10)
        .filter(|x| x % 2 == 0)
        .map(|x| x * x)
        .fold(0, |acc, x| acc + x);
    
    println!("result = {}", result);
}
```

### 条件处理

```rust
fn main() {
    let data = vec![Some(1), None, Some(3), None, Some(5)];
    
    let sum: i32 = data.iter()
        .filter_map(|x| *x)
        .sum();
    
    println!("sum = {}", sum);
    
    // 处理错误
    let results: Vec<Result<i32, &str>> = vec![Ok(1), Err("error"), Ok(3)];
    
    let successes: Vec<_> = results.iter()
        .filter_map(|r| r.ok())
        .collect();
    
    println!("successes = {:?}", successes);
}
```

## 参考

- [Rust 程序设计语言 - 闭包](https://doc.rust-lang.org/book/ch13-01-closures.html)
- [Rust 程序设计语言 - 迭代器](https://doc.rust-lang.org/book/ch13-02-iterators.html)
- [Rust by Example - 闭包](https://doc.rust-lang.org/rust-by-example/fn/closures.html)
- [Rust by Example - 迭代器](https://doc.rust-lang.org/rust-by-example/trait/iter.html)