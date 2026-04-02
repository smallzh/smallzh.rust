# 智能指针

## 0x01 智能指针概述

智能指针是表现得像指针的数据结构，但拥有额外的元数据和功能。与引用不同，智能指针拥有它们指向的数据。

### 常用智能指针

- `Box<T>`：在堆上分配值
- `Rc<T>`：引用计数智能指针
- `Arc<T>`：原子引用计数智能指针
- `RefCell<T>`：内部可变性模式

## 0x02 Box<T>

### 在堆上存储数据

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
    
    // 递归类型
    #[derive(Debug)]
    enum List {
        Cons(i32, Box<List>),
        Nil,
    }
    
    use List::{Cons, Nil};
    
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
    println!("{:?}", list);
}
```

### Box 的使用场景

```rust
fn main() {
    // 1. 当有一个在编译时未知大小的类型，但又需要在上下文中使用该类型时
    let large_data = Box::new([0; 1000000]);
    
    // 2. 当有大量数据并希望确保数据不会被复制时
    let data = Box::new(vec![1, 2, 3, 4, 5]);
    
    // 3. 当希望拥有一个值并只关心它是否实现了特定 trait 而不是具体类型时
    let printable: Box<dyn std::fmt::Display> = Box::new(42);
    println!("{}", printable);
}
```

## 0x03 Rc<T> 引用计数

### 共享所有权

```rust
use std::rc::Rc;

fn main() {
    let a = Rc::new(5);
    let b = Rc::clone(&a);
    let c = Rc::clone(&a);
    
    println!("a = {}, b = {}, c = {}", a, b, c);
    println!("Reference count: {}", Rc::strong_count(&a));
}
```

### 共享列表

```rust
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

fn main() {
    use List::{Cons, Nil};
    
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
    
    println!("a = {:?}", a);
    println!("b = {:?}", b);
    println!("c = {:?}", c);
}
```

## 0x04 RefCell<T> 内部可变性

### 违反借用规则的情况

```rust
use std::cell::RefCell;

fn main() {
    let data = RefCell::new(5);
    
    // 不可变借用
    let borrow1 = data.borrow();
    println!("borrow1 = {}", borrow1);
    
    // 可变借用
    let mut borrow2 = data.borrow_mut();
    *borrow2 += 1;
    
    println!("borrow2 = {}", borrow2);
}
```

### 运行时借用检查

```rust
use std::cell::RefCell;

fn main() {
    let data = RefCell::new(vec![1, 2, 3]);
    
    // 这会在运行时 panic
    let mut borrow1 = data.borrow_mut();
    let mut borrow2 = data.borrow_mut(); // panic!
}
```

## 0x05 Rc<RefCell<T>> 组合

### 共享可变数据

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

fn main() {
    use List::{Cons, Nil};
    
    let value = Rc::new(RefCell::new(5));
    
    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));
    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));
    
    *value.borrow_mut() += 10;
    
    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

## 0x06 Arc<T> 原子引用计数

### 线程安全的引用计数

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let data = Arc::new(vec![1, 2, 3, 4, 5]);
    
    let mut handles = vec![];
    
    for i in 0..5 {
        let data = Arc::clone(&data);
        let handle = thread::spawn(move || {
            println!("Thread {} sees: {:?}", i, data);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

## 0x07 Mutex<T> 互斥锁

### 线程间共享可变数据

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", *counter.lock().unwrap());
}
```

## 0x08 Drop trait

### 自定义清理逻辑

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    
    println!("CustomSmartPointers created.");
    // drop 会在这里自动调用
}
```

### 手动提前 drop

```rust
fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    println!("CustomSmartPointer created.");
    
    drop(c); // 手动 drop
    
    println!("CustomSmartPointer dropped before the end of main.");
}
```

## 0x09 Deref trait

### 自定义解引用行为

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;
    
    fn deref(&self) -> &T {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);
    
    assert_eq!(5, x);
    assert_eq!(5, *y);
    
    // 解引用强制多态
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}

fn hello(name: &str) {
    println!("Hello, {}!", name);
}
```

## 0x10 智能指针选择指南

### 选择表

| 场景 | 推荐智能指针 |
|------|-------------|
| 堆分配 | `Box<T>` |
| 共享所有权（单线程） | `Rc<T>` |
| 共享所有权（多线程） | `Arc<T>` |
| 内部可变性 | `RefCell<T>` |
| 线程间共享可变数据 | `Arc<Mutex<T>>` |
| 引用计数 + 内部可变性 | `Rc<RefCell<T>>` |

## 0x11 实际应用示例

### 示例 1：图结构

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    edges: Vec<Rc<RefCell<Node>>>,
}

impl Node {
    fn new(value: i32) -> Rc<RefCell<Node>> {
        Rc::new(RefCell::new(Node {
            value,
            edges: Vec::new(),
        }))
    }
    
    fn add_edge(from: &Rc<RefCell<Node>>, to: &Rc<RefCell<Node>>) {
        from.borrow_mut().edges.push(Rc::clone(to));
    }
}

fn main() {
    let node1 = Node::new(1);
    let node2 = Node::new(2);
    let node3 = Node::new(3);
    
    Node::add_edge(&node1, &node2);
    Node::add_edge(&node1, &node3);
    Node::add_edge(&node2, &node3);
    
    println!("Graph structure:");
    println!("Node 1: {:?}", node1);
    println!("Node 2: {:?}", node2);
    println!("Node 3: {:?}", node3);
}
```

### 示例 2：缓存系统

```rust
use std::collections::HashMap;
use std::rc::Rc;
use std::cell::RefCell;

struct Cache {
    data: RefCell<HashMap<String, Rc<dyn std::any::Any>>>,
}

impl Cache {
    fn new() -> Cache {
        Cache {
            data: RefCell::new(HashMap::new()),
        }
    }
    
    fn get<T: 'static + Clone>(&self, key: &str) -> Option<T> {
        self.data.borrow().get(key)
            .and_then(|v| v.downcast_ref::<T>())
            .cloned()
    }
    
    fn set<T: 'static + Clone>(&self, key: String, value: T) {
        self.data.borrow_mut().insert(key, Rc::new(value));
    }
}

fn main() {
    let cache = Cache::new();
    
    cache.set("user_id".to_string(), 42);
    cache.set("username".to_string(), "Alice".to_string());
    
    if let Some(id) = cache.get::<i32>("user_id") {
        println!("User ID: {}", id);
    }
    
    if let Some(name) = cache.get::<String>("username") {
        println!("Username: {}", name);
    }
}
```

## 参考

- [Rust 程序设计语言 - 智能指针](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html)
- [Rust by Example - Box](https://doc.rust-lang.org/rust-by-example/std/box.html)
- [Rust by Example - Rc](https://doc.rust-lang.org/rust-by-example/std/rc.html)
- [Rust by Example - RefCell](https://doc.rust-lang.org/rust-by-example/std/cell.html)