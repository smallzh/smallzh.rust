# 集合类型

## 0x01 动态数组 Vec<T>

### 创建向量

```rust
fn main() {
    // 创建空向量
    let v: Vec<i32> = Vec::new();
    
    // 使用初始值创建向量
    let v = vec![1, 2, 3];
    
    // 使用宏创建
    let v = vec![0; 10]; // 10 个 0
    
    println!("v = {:?}", v);
}
```

### 更新向量

```rust
fn main() {
    let mut v = Vec::new();
    
    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
    
    println!("v = {:?}", v);
}
```

### 读取向量元素

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    
    // 使用索引
    let third: &i32 = &v[2];
    println!("The third element is {}", third);
    
    // 使用 get 方法
    match v.get(2) {
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }
    
    // 越界访问
    // let does_not_exist = &v[100]; // panic
    let does_not_exist = v.get(100); // None
}
```

### 遍历向量

```rust
fn main() {
    let v = vec![100, 32, 57];
    
    // 不可变引用
    for i in &v {
        println!("{}", i);
    }
    
    // 可变引用
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
    
    println!("v = {:?}", v);
}
```

### 使用枚举存储不同类型

```rust
#[derive(Debug)]
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

fn main() {
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
    
    println!("row = {:?}", row);
}
```

## 0x02 字符串

### 创建字符串

```rust
fn main() {
    // 创建空字符串
    let mut s = String::new();
    
    // 使用 to_string 方法
    let data = "initial contents";
    let s = data.to_string();
    
    // 使用 String::from
    let s = String::from("initial contents");
    
    println!("s = {}", s);
}
```

### 更新字符串

```rust
fn main() {
    let mut s = String::from("foo");
    
    // push_str
    s.push_str("bar");
    println!("s = {}", s);
    
    // push
    let mut s = String::from("lo");
    s.push('l');
    println!("s = {}", s);
    
    // 使用 + 运算符
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // s1 被移动
    println!("s3 = {}", s3);
    
    // 使用 format! 宏
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");
    let s = format!("{}-{}-{}", s1, s2, s3);
    println!("s = {}", s);
}
```

### 索引字符串

```rust
fn main() {
    let s1 = String::from("hello");
    // let h = s1[0]; // 错误：Rust 不支持字符串索引
    
    // 使用 bytes
    for b in "नमस्ते".bytes() {
        println!("{}", b);
    }
    
    // 使用 chars
    for c in "नमस्ते".chars() {
        println!("{}", c);
    }
}
```

### 字符串切片

```rust
fn main() {
    let hello = "Здравствуйте";
    let s = &hello[0..4]; // "Зд"
    println!("s = {}", s);
    
    // 注意：切片边界必须在字符边界上
    // let s = &hello[0..1]; // panic
}
```

## 0x03 哈希映射 HashMap<K, V>

### 创建哈希映射

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    
    println!("scores = {:?}", scores);
}
```

### 访问哈希映射中的值

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    
    let team_name = String::from("Blue");
    let score = scores.get(&team_name);
    
    match score {
        Some(s) => println!("Score: {}", s),
        None => println!("Team not found"),
    }
    
    // 遍历
    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
}
```

### 哈希映射与所有权

```rust
use std::collections::HashMap;

fn main() {
    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");
    
    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name 和 field_value 被移动
    
    // println!("{}", field_name); // 错误
}
```

### 更新哈希映射

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25); // 覆盖
    
    println!("{:?}", scores);
    
    // 只在键没有值时插入
    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::em("Blue")).or_insert(50);
    
    println!("{:?}", scores);
}
```

### 基于旧值更新

```rust
use std::collections::HashMap;

fn main() {
    let text = "hello world wonderful world";
    
    let mut map = HashMap::new();
    
    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }
    
    println!("{:?}", map);
}
```

## 0x04 其他集合类型

### VecDeque<T> 双端队列

```rust
use std::collections::VecDeque;

fn main() {
    let mut buf = VecDeque::new();
    
    buf.push_front(1);
    buf.push_back(2);
    buf.push_front(3);
    
    println!("buf = {:?}", buf); // [3, 1, 2]
    
    let front = buf.pop_front();
    let back = buf.pop_back();
    
    println!("front = {:?}, back = {:?}", front, back);
}
```

### LinkedList<T> 双向链表

```rust
use std::collections::LinkedList;

fn main() {
    let mut list = LinkedList::new();
    
    list.push_back(1);
    list.push_back(2);
    list.push_back(3);
    
    println!("list = {:?}", list);
    
    for i in list {
        println!("i = {}", i);
    }
}
```

### BTreeMap<K, V> 有序映射

```rust
use std::collections::BTreeMap;

fn main() {
    let mut map = BTreeMap::new();
    
    map.insert("c", 3);
    map.insert("a", 1);
    map.insert("b", 2);
    
    for (key, value) in &map {
        println!("{}: {}", key, value);
    } // a: 1, b: 2, c: 3 (有序)
}
```

### BTreeSet<T> 有序集合

```rust
use std::collections::BTreeSet;

fn main() {
    let mut set = BTreeSet::new();
    
    set.insert(3);
    set.insert(1);
    set.insert(2);
    set.insert(1); // 重复值被忽略
    
    for i in &set {
        println!("{}", i);
    } // 1, 2, 3 (有序)
}
```

### HashSet<T> 哈希集合

```rust
use std::collections::HashSet;

fn main() {
    let mut set = HashSet::new();
    
    set.insert(1);
    set.insert(2);
    set.insert(3);
    set.insert(1); // 重复值被忽略
    
    println!("set = {:?}", set);
    
    // 集合操作
    let mut a = HashSet::new();
    a.insert(1);
    a.insert(2);
    
    let mut b = HashSet::new();
    b.insert(2);
    b.insert(3);
    
    let union: HashSet<_> = a.union(&b).collect();
    let intersection: HashSet<_> = a.intersection(&b).collect();
    let difference: HashSet<_> = a.difference(&b).collect();
    
    println!("union = {:?}", union);
    println!("intersection = {:?}", intersection);
    println!("difference = {:?}", difference);
}
```

## 0x05 集合性能比较

### 选择指南

| 集合类型 | 适用场景 | 时间复杂度 |
|----------|----------|------------|
| Vec<T> | 随机访问、末尾操作 | O(1) 随机访问 |
| VecDeque<T> | 两端插入删除 | O(1) 两端操作 |
| LinkedList<T> | 频繁插入删除 | O(1) 插入删除 |
| HashMap<K, V> | 快速键值查找 | O(1) 平均查找 |
| BTreeMap<K, V> | 有序键值存储 | O(log n) 查找 |
| HashSet<T> | 去重、集合运算 | O(1) 平均查找 |
| BTreeSet<T> | 有序去重 | O(log n) 查找 |

## 0x06 迭代器与集合

### 收集器

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];
    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
    
    println!("v2 = {:?}", v2);
    
    // 使用 collect 转换为不同集合类型
    use std::collections::HashSet;
    
    let v = vec![1, 2, 3, 2, 1];
    let set: HashSet<_> = v.into_iter().collect();
    
    println!("set = {:?}", set);
}
```

### 过滤器

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    
    let even: Vec<_> = v.iter().filter(|x| *x % 2 == 0).collect();
    
    println!("even = {:?}", even);
}
```

## 0x07 排序

### 基本排序

```rust
fn main() {
    let mut v = vec![3, 1, 4, 1, 5, 9, 2, 6];
    
    v.sort();
    println!("sorted = {:?}", v);
    
    v.reverse();
    println!("reversed = {:?}", v);
}
```

### 自定义排序

```rust
fn main() {
    let mut v = vec![3, 1, 4, 1, 5, 9, 2, 6];
    
    v.sort_by(|a, b| b.cmp(a)); // 降序
    
    println!("sorted = {:?}", v);
    
    // 按条件排序
    let mut v = vec!["hello", "world", "a", "bc"];
    v.sort_by_key(|s| s.len());
    
    println!("sorted = {:?}", v);
}
```

## 0x08 实际应用示例

### 示例 1：单词频率统计

```rust
use std::collections::HashMap;

fn word_frequency(text: &str) -> HashMap<&str, usize> {
    let mut frequency = HashMap::new();
    
    for word in text.split_whitespace() {
        let count = frequency.entry(word).or_insert(0);
        *count += 1;
    }
    
    frequency
}

fn main() {
    let text = "hello world hello rust world";
    let frequency = word_frequency(text);
    
    for (word, count) in &frequency {
        println!("{}: {}", word, count);
    }
}
```

### 示例 2：学生成绩管理

```rust
use std::collections::HashMap;

struct Student {
    name: String,
    grades: Vec<f64>,
}

impl Student {
    fn new(name: &str) -> Student {
        Student {
            name: name.to_string(),
            grades: Vec::new(),
        }
    }
    
    fn add_grade(&mut self, grade: f64) {
        self.grades.push(grade);
    }
    
    fn average(&self) -> Option<f64> {
        if self.grades.is_empty() {
            None
        } else {
            Some(self.grades.iter().sum::<f64>() / self.grades.len() as f64)
        }
    }
}

fn main() {
    let mut students = HashMap::new();
    
    let mut alice = Student::new("Alice");
    alice.add_grade(85.0);
    alice.add_grade(92.0);
    alice.add_grade(78.0);
    
    let mut bob = Student::new("Bob");
    bob.add_grade(90.0);
    bob.add_grade(88.0);
    
    students.insert("Alice", alice);
    students.insert("Bob", bob);
    
    for (name, student) in &students {
        match student.average() {
            Some(avg) => println!("{}: {:.2}", name, avg),
            None => println!("{}: No grades", name),
        }
    }
}
```

## 参考

- [Rust 程序设计语言 - 集合](https://doc.rust-lang.org/book/ch08-00-common-collections.html)
- [Rust by Example - 向量](https://doc.rust-lang.org/rust-by-example/std/vec.html)
- [Rust by Example - 字符串](https://doc.rust-lang.org/rust-by-example/std/str.html)
- [Rust by Example - 哈希映射](https://doc.rust-lang.org/rust-by-example/std/hash.html)