# 并发编程基础

## 0x01 并发概述

Rust 的并发模型基于所有权和类型系统，在编译时防止数据竞争。Rust 提供了多种并发编程方式：

- 线程（Threads）
- 消息传递（Message Passing）
- 共享状态（Shared State）
- 异步编程（Async/Await）

## 0x02 线程基础

### 创建线程

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });
    
    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
    
    handle.join().unwrap(); // 等待线程完成
}
```

### 线程与 move 闭包

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });
    
    // drop(v); // 错误：v 已被移动
    handle.join().unwrap();
}
```

## 0x03 消息传递通道

### 基本通道

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    
    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        // println!("val is {}", val); // 错误：val 已被发送
    });
    
    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

### 发送多个值

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });
    
    for received in rx {
        println!("Got: {}", received);
    }
}
```

### 多个发送者

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    let tx1 = mpsc::Sender::clone(&tx);
    
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("thread"),
        ];
        
        for val in vals {
            tx1.send(val).unwrap();
        }
    });
    
    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];
        
        for val in vals {
            tx.send(val).unwrap();
        }
    });
    
    for received in rx {
        println!("Got: {}", received);
    }
}
```

## 0x04 共享状态并发

### 互斥锁 Mutex

```rust
use std::sync::{Mutex, Arc};
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

### 读写锁 RwLock

```rust
use std::sync::{RwLock, Arc};
use std::thread;

fn main() {
    let lock = Arc::new(RwLock::new(5));
    
    // 多个读取者
    let mut read_handles = vec![];
    for _ in 0..5 {
        let lock = Arc::clone(&lock);
        let handle = thread::spawn(move || {
            let num = lock.read().unwrap();
            println!("Read: {}", *num);
        });
        read_handles.push(handle);
    }
    
    // 一个写入者
    let write_handle = thread::spawn(move || {
        let mut num = lock.write().unwrap();
        *num += 1;
        println!("Write: {}", *num);
    });
    
    for handle in read_handles {
        handle.join().unwrap();
    }
    write_handle.join().unwrap();
}
```

### 原子类型

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                counter.fetch_add(1, Ordering::SeqCst);
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", counter.load(Ordering::SeqCst));
}
```

## 0x05 线程同步

### 使用屏障 Barrier

```rust
use std::sync::{Arc, Barrier};
use std::thread;

fn main() {
    let mut handles = vec![];
    let barrier = Arc::new(Barrier::new(10));
    
    for i in 0..10 {
        let barrier = Arc::clone(&barrier);
        handles.push(thread::spawn(move || {
            println!("before wait {}", i);
            barrier.wait();
            println!("after wait {}", i);
        }));
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

### 条件变量 Condvar

```rust
use std::sync::{Arc, Mutex, Condvar};
use std::thread;

fn main() {
    let pair = Arc::new((Mutex::new(false), Condvar::new()));
    let pair2 = Arc::clone(&pair);
    
    thread::spawn(move || {
        let (lock, cvar) = &*pair2;
        let mut started = lock.lock().unwrap();
        *started = true;
        cvar.notify_one();
    });
    
    let (lock, cvar) = &*pair;
    let mut started = lock.lock().unwrap();
    while !*started {
        started = cvar.wait(started).unwrap();
    }
    
    println!("started changed");
}
```

## 0x06 Send 和 Sync trait

### Send trait

实现了 `Send` 的类型可以安全地在线程间传递所有权：

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    
    let handle = thread::spawn(move || {
        println!("v = {:?}", v);
    });
    
    handle.join().unwrap();
}
```

### Sync trait

实现了 `Sync` 的类型可以安全地在线程间引用：

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let data = Arc::new(vec![1, 2, 3]);
    
    let mut handles = vec![];
    for i in 0..3 {
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

## 0x07 线程池

### 简单线程池实现

```rust
use std::sync::{mpsc, Arc, Mutex};
use std::thread;

struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Message>,
}

impl ThreadPool {
    fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
        
        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));
        
        let mut workers = Vec::with_capacity(size);
        
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
        
        ThreadPool { workers, sender }
    }
    
    fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
        self.sender.send(Message::NewJob(job)).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        println!("Sending terminate message to all workers.");
        
        for _ in &self.workers {
            self.sender.send(Message::Terminate).unwrap();
        }
        
        println!("Shutting down all workers.");
        
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);
            
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Message>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv().unwrap();
            
            match message {
                Message::NewJob(job) => {
                    println!("Worker {} got a job; executing.", id);
                    job();
                }
                Message::Terminate => {
                    println!("Worker {} was told to terminate.", id);
                    break;
                }
            }
        });
        
        Worker {
            id,
            thread: Some(thread),
        }
    }
}

enum Message {
    NewJob(Job),
    Terminate,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

fn main() {
    let pool = ThreadPool::new(4);
    
    for i in 0..8 {
        pool.execute(move || {
            println!("Processing job {}", i);
            thread::sleep(std::time::Duration::from_millis(500));
        });
    }
    
    println!("All jobs submitted.");
    thread::sleep(std::time::Duration::from_secs(2));
}
```

## 0x08 并发模式

### 生产者-消费者模式

```rust
use std::sync::{mpsc, Arc, Mutex};
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    let rx = Arc::new(Mutex::new(rx));
    
    // 生产者线程
    let producer = thread::spawn(move || {
        for i in 0..10 {
            tx.send(i).unwrap();
            println!("Produced: {}", i);
            thread::sleep(Duration::from_millis(100));
        }
    });
    
    // 消费者线程
    let mut consumers = vec![];
    for id in 0..3 {
        let rx = Arc::clone(&rx);
        let consumer = thread::spawn(move || loop {
            let msg = rx.lock().unwrap().recv();
            match msg {
                Ok(value) => {
                    println!("Consumer {} consumed: {}", id, value);
                    thread::sleep(Duration::from_millis(200));
                }
                Err(_) => break,
            }
        });
        consumers.push(consumer);
    }
    
    producer.join().unwrap();
    drop(rx); // 关闭通道
    
    for consumer in consumers {
        consumer.join().unwrap();
    }
}
```

### 工作窃取模式

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::collections::VecDeque;

fn main() {
    let queue = Arc::new(Mutex::new(VecDeque::new()));
    
    // 添加任务
    {
        let mut q = queue.lock().unwrap();
        for i in 0..20 {
            q.push_back(i);
        }
    }
    
    let mut handles = vec![];
    for id in 0..4 {
        let queue = Arc::clone(&queue);
        let handle = thread::spawn(move || loop {
            let job = {
                let mut q = queue.lock().unwrap();
                q.pop_front()
            };
            
            match job {
                Some(task) => {
                    println!("Worker {} processing task {}", id, task);
                    thread::sleep(std::time::Duration::from_millis(100));
                }
                None => break,
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

## 0x09 性能考虑

### 避免锁竞争

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // 使用分片减少锁竞争
    let shards: Vec<_> = (0..4)
        .map(|_| Arc::new(Mutex::new(Vec::new())))
        .collect();
    
    let mut handles = vec![];
    
    for i in 0..8 {
        let shard = Arc::clone(&shards[i % 4]);
        let handle = thread::spawn(move || {
            let mut data = shard.lock().unwrap();
            data.push(i);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    for (i, shard) in shards.iter().enumerate() {
        println!("Shard {}: {:?}", i, shard.lock().unwrap());
    }
}
```

## 参考

- [Rust 程序设计语言 - 并发](https://doc.rust-lang.org/book/ch16-00-concurrency.html)
- [Rust by Example - 线程](https://doc.rust-lang.org/rust-by-example/std_misc/threads.html)
- [Rust 标准库 - std::sync](https://doc.rust-lang.org/std/sync/)
- [Rust 标准库 - std::thread](https://doc.rust-lang.org/std/thread/)