# 异步编程

## 0x01 异步编程概述

Rust 的异步编程基于 `async/await` 语法和 `Future` trait。异步编程允许在等待 I/O 操作时执行其他任务，提高程序的并发性能。

### 核心概念

- **Future**：表示一个可能尚未完成的计算
- **async/await**：定义和等待异步函数的语法糖
- **Executor**：运行 Future 的运行时
- **Task**：可以调度执行的工作单元

## 0x02 基本异步编程

### 异步函数

```rust
async fn hello() {
    println!("Hello, world!");
}

#[tokio::main]
async fn main() {
    hello().await;
}
```

### 返回值的异步函数

```rust
async fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[tokio::main]
async fn main() {
    let result = add(2, 3).await;
    println!("2 + 3 = {}", result);
}
```

### 异步闭包

```rust
use std::future::Future;

fn make_future() -> impl Future<Output = i32> {
    async {
        // 异步计算
        42
    }
}

#[tokio::main]
async fn main() {
    let result = make_future().await;
    println!("Result: {}", result);
}
```

## 0x03 异步运行时

### 使用 tokio

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1.0", features = ["full"] }
```

```rust
use tokio::time::{sleep, Duration};

async fn task1() {
    println!("Task 1 start");
    sleep(Duration::from_secs(1)).await;
    println!("Task 1 end");
}

async fn task2() {
    println!("Task 2 start");
    sleep(Duration::from_secs(2)).await;
    println!("Task 2 end");
}

#[tokio::main]
async fn main() {
    // 顺序执行
    task1().await;
    task2().await;
    
    // 并发执行
    tokio::join!(task1(), task2());
}
```

### 使用 async-std

```toml
# Cargo.toml
[dependencies]
async-std = { version = "1.0", features = ["attributes"] }
```

```rust
use async_std::task;
use std::time::Duration;

async fn hello() {
    println!("Hello");
    task::sleep(Duration::from_secs(1)).await;
    println!("World");
}

#[async_std::main]
async fn main() {
    hello().await;
}
```

## 0x04 并发执行

### join! 宏

```rust
use tokio::time::{sleep, Duration};

async fn fetch_data(id: u32) -> String {
    sleep(Duration::from_secs(1)).await;
    format!("Data from task {}", id)
}

#[tokio::main]
async fn main() {
    let (result1, result2, result3) = tokio::join!(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    );
    
    println!("Results: {}, {}, {}", result1, result2, result3);
}
```

### select! 宏

```rust
use tokio::time::{sleep, Duration};

async fn task1() -> String {
    sleep(Duration::from_secs(2)).await;
    "Task 1 done".to_string()
}

async fn task2() -> String {
    sleep(Duration::from_secs(1)).await;
    "Task 2 done".to_string()
}

#[tokio::main]
async fn main() {
    tokio::select! {
        result1 = task1() => {
            println!("{}", result1);
        }
        result2 = task2() => {
            println!("{}", result2);
        }
    }
}
```

## 0x05 异步流

### 使用 Stream

```rust
use tokio::stream::{self, StreamExt};
use tokio::time::{sleep, Duration};

async fn number_stream() -> impl stream::Stream<Item = i32> {
    stream::iter(vec![1, 2, 3, 4, 5])
}

#[tokio::main]
async fn main() {
    let mut stream = number_stream();
    
    while let Some(number) = stream.next().await {
        println!("Received: {}", number);
    }
}
```

### 异步迭代器

```rust
use futures::stream::{self, StreamExt};

#[tokio::main]
async fn main() {
    let stream = stream::iter(1..=10)
        .filter(|x| futures::future::ready(x % 2 == 0))
        .map(|x| x * x);
    
    let results: Vec<i32> = stream.collect().await;
    println!("Results: {:?}", results);
}
```

## 0x06 异步通道

### tokio 通道

```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel(32);
    
    let tx1 = tx.clone();
    tokio::spawn(async move {
        tx1.send("hello from task 1").await.unwrap();
    });
    
    tokio::spawn(async move {
        tx.send("hello from task 2").await.unwrap();
    });
    
    while let Some(message) = rx.recv().await {
        println!("Received: {}", message);
    }
}
```

### 广播通道

```rust
use tokio::sync::broadcast;

#[tokio::main]
async fn main() {
    let (tx, _rx) = broadcast::channel(16);
    
    let mut rx1 = tx.subscribe();
    let mut rx2 = tx.subscribe();
    
    tokio::spawn(async move {
        tx.send("hello").await.unwrap();
    });
    
    tokio::spawn(async move {
        println!("rx1 received: {}", rx1.recv().await.unwrap());
    });
    
    println!("rx2 received: {}", rx2.recv().await.unwrap());
}
```

## 0x07 异步锁

### 异步互斥锁

```rust
use tokio::sync::Mutex;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let data = Arc::new(Mutex::new(0));
    
    let mut handles = vec![];
    
    for _ in 0..10 {
        let data = Arc::clone(&data);
        let handle = tokio::spawn(async move {
            let mut num = data.lock().await;
            *num += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.await.unwrap();
    }
    
    println!("Result: {}", *data.lock().await);
}
```

### 异步读写锁

```rust
use tokio::sync::RwLock;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let lock = Arc::new(RwLock::new(5));
    
    // 多个读取者
    let mut read_handles = vec![];
    for _ in 0..5 {
        let lock = Arc::clone(&lock);
        let handle = tokio::spawn(async move {
            let num = lock.read().await;
            println!("Read: {}", *num);
        });
        read_handles.push(handle);
    }
    
    // 一个写入者
    let write_handle = tokio::spawn(async move {
        let mut num = lock.write().await;
        *num += 1;
        println!("Write: {}", *num);
    });
    
    for handle in read_handles {
        handle.await.unwrap();
    }
    write_handle.await.unwrap();
}
```

## 0x08 异步任务

### 生成任务

```rust
use tokio::time::{sleep, Duration};

async fn background_task(id: u32) {
    println!("Task {} started", id);
    sleep(Duration::from_secs(1)).await;
    println!("Task {} completed", id);
}

#[tokio::main]
async fn main() {
    let mut handles = vec![];
    
    for i in 0..5 {
        let handle = tokio::spawn(background_task(i));
        handles.push(handle);
    }
    
    for handle in handles {
        handle.await.unwrap();
    }
}
```

### 任务取消

```rust
use tokio::time::{sleep, Duration};

async fn long_running_task() {
    loop {
        println!("Task running...");
        sleep(Duration::from_secs(1)).await;
    }
}

#[tokio::main]
async fn main() {
    let handle = tokio::spawn(long_running_task());
    
    sleep(Duration::from_secs(3)).await;
    handle.abort();
    
    match handle.await {
        Ok(_) => println!("Task completed"),
        Err(e) if e.is_cancelled() => println!("Task was cancelled"),
        Err(e) => println!("Task failed: {}", e),
    }
}
```

## 0x09 异步 I/O

### 异步文件读写

```rust
use tokio::fs;
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 异步写入文件
    let mut file = fs::File::create("hello.txt").await?;
    file.write_all(b"Hello, async world!").await?;
    
    // 异步读取文件
    let mut file = fs::File::open("hello.txt").await?;
    let mut contents = String::new();
    file.read_to_string(&mut contents).await?;
    
    println!("File contents: {}", contents);
    
    Ok(())
}
```

### 异步网络 I/O

```rust
use tokio::net::TcpStream;
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut stream = TcpStream::connect("127.0.0.1:8080").await?;
    
    stream.write_all(b"Hello from async client!").await?;
    
    let mut buffer = [0; 1024];
    let n = stream.read(&mut buffer).await?;
    
    println!("Received: {}", String::from_utf8_lossy(&buffer[..n]));
    
    Ok(())
}
```

## 0x10 错误处理

### 异步错误处理

```rust
use tokio::fs;
use std::io;

async fn read_file(path: &str) -> Result<String, io::Error> {
    let contents = fs::read_to_string(path).await?;
    Ok(contents)
}

#[tokio::main]
async fn main() {
    match read_file("hello.txt").await {
        Ok(contents) => println!("File contents: {}", contents),
        Err(e) => println!("Error reading file: {}", e),
    }
}
```

### 使用 anyhow

```rust
use anyhow::{Result, Context};
use tokio::fs;

async fn process_file(path: &str) -> Result<String> {
    let contents = fs::read_to_string(path)
        .await
        .context("Failed to read file")?;
    
    Ok(contents.to_uppercase())
}

#[tokio::main]
async fn main() -> Result<()> {
    let result = process_file("hello.txt").await?;
    println!("Processed: {}", result);
    
    Ok(())
}
```

## 0x11 性能优化

### 避免阻塞操作

```rust
use tokio::task;

async fn blocking_operation() {
    // 在阻塞线程池中运行阻塞操作
    task::spawn_blocking(|| {
        // 阻塞代码
        std::thread::sleep(std::time::Duration::from_secs(1));
        "blocking result"
    }).await.unwrap();
}

#[tokio::main]
async fn main() {
    blocking_operation().await;
}
```

### 使用缓冲区

```rust
use tokio::io::{AsyncBufReadExt, BufReader};
use tokio::fs::File;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let file = File::open("large_file.txt").await?;
    let reader = BufReader::new(file);
    
    let mut lines = reader.lines();
    
    while let Some(line) = lines.next_line().await? {
        println!("Line: {}", line);
    }
    
    Ok(())
}
```

## 0x12 实际应用示例

### 异步 Web 服务器

```rust
use axum::{
    routing::get,
    Router,
    Json,
};
use serde::{Deserialize, Serialize};
use std::sync::Arc;
use tokio::sync::Mutex;

#[derive(Serialize, Deserialize, Clone)]
struct User {
    id: u32,
    name: String,
}

struct AppState {
    users: Vec<User>,
}

async fn get_users(state: Arc<Mutex<AppState>>) -> Json<Vec<User>> {
    let state = state.lock().await;
    Json(state.users.clone())
}

async fn add_user(
    state: Arc<Mutex<AppState>>,
    Json(user): Json<User>,
) -> Json<User> {
    let mut state = state.lock().await;
    state.users.push(user.clone());
    Json(user)
}

#[tokio::main]
async fn main() {
    let state = Arc::new(Mutex::new(AppState {
        users: Vec::new(),
    }));
    
    let app = Router::new()
        .route("/users", get(get_users).post(add_user))
        .layer(axum::Extension(state));
    
    axum::Server::bind(&"127.0.0.1:3000".parse().unwrap())
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

## 参考

- [Rust 异步编程](https://rust-lang.github.io/async-book/)
- [tokio 文档](https://docs.rs/tokio/)
- [async-std 文档](https://docs.rs/async-std/)
- [futures 文档](https://docs.rs/futures/)
- [Rust by Example - 异步](https://doc.rust-lang.org/rust-by-example/async.html)