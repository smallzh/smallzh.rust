# 网络编程

## 0x01 网络编程概述

Rust 提供了强大的网络编程能力，包括：

- TCP/UDP 套接字
- HTTP 客户端和服务器
- WebSocket
- 异步网络 I/O

### 常用库

- `std::net` - 基本网络功能
- `tokio` - 异步运行时
- `reqwest` - HTTP 客户端
- `hyper` - HTTP 库
- `actix-web` - Web 框架
- `tungstenite` - WebSocket

## 0x02 TCP 编程

### TCP 客户端

```rust
use std::net::TcpStream;
use std::io::{Read, Write};

fn main() -> std::io::Result<()> {
    let mut stream = TcpStream::connect("127.0.0.1:8080")?;
    
    // 发送数据
    stream.write_all(b"Hello, server!")?;
    
    // 接收响应
    let mut buffer = [0; 1024];
    let bytes_read = stream.read(&mut buffer)?;
    
    println!("Received: {}", String::from_utf8_lossy(&buffer[..bytes_read]));
    
    Ok(())
}
```

### TCP 服务器

```rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};
use std::thread;

fn handle_client(mut stream: TcpStream) -> std::io::Result<()> {
    let mut buffer = [0; 1024];
    
    loop {
        let bytes_read = stream.read(&mut buffer)?;
        if bytes_read == 0 {
            break;
        }
        
        println!("Received: {}", String::from_utf8_lossy(&buffer[..bytes_read]));
        
        // 回显数据
        stream.write_all(&buffer[..bytes_read])?;
    }
    
    Ok(())
}

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Server listening on 127.0.0.1:8080");
    
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                thread::spawn(|| {
                    if let Err(e) = handle_client(stream) {
                        println!("Error: {}", e);
                    }
                });
            }
            Err(e) => {
                println!("Error: {}", e);
            }
        }
    }
    
    Ok(())
}
```

### 异步 TCP 服务器

```rust
use tokio::net::TcpListener;
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Server listening on 127.0.0.1:8080");
    
    loop {
        let (mut socket, addr) = listener.accept().await?;
        println!("New connection from: {}", addr);
        
        tokio::spawn(async move {
            let mut buffer = [0; 1024];
            
            loop {
                let bytes_read = match socket.read(&mut buffer).await {
                    Ok(0) => break,
                    Ok(n) => n,
                    Err(e) => {
                        println!("Error reading: {}", e);
                        break;
                    }
                };
                
                println!("Received: {}", String::from_utf8_lossy(&buffer[..bytes_read]));
                
                if let Err(e) = socket.write_all(&buffer[..bytes_read]).await {
                    println!("Error writing: {}", e);
                    break;
                }
            }
        });
    }
}
```

## 0x03 UDP 编程

### UDP 客户端

```rust
use std::net::UdpSocket;

fn main() -> std::io::Result<()> {
    let socket = UdpSocket::bind("127.0.0.1:0")?;
    
    // 发送数据
    socket.send_to(b"Hello, server!", "127.0.0.1:8080")?;
    
    // 接收响应
    let mut buffer = [0; 1024];
    let (bytes_read, src_addr) = socket.recv_from(&mut buffer)?;
    
    println!("Received from {}: {}", src_addr, String::from_utf8_lossy(&buffer[..bytes_read]));
    
    Ok(())
}
```

### UDP 服务器

```rust
use std::net::UdpSocket;

fn main() -> std::io::Result<()> {
    let socket = UdpSocket::bind("127.0.0.1:8080")?;
    println!("UDP server listening on 127.0.0.1:8080");
    
    let mut buffer = [0; 1024];
    
    loop {
        let (bytes_read, src_addr) = socket.recv_from(&mut buffer)?;
        println!("Received from {}: {}", src_addr, String::from_utf8_lossy(&buffer[..bytes_read]));
        
        // 回显数据
        socket.send_to(&buffer[..bytes_read], src_addr)?;
    }
}
```

## 0x04 HTTP 客户端

### 使用 reqwest

```toml
# Cargo.toml
[dependencies]
reqwest = { version = "0.11", features = ["json"] }
tokio = { version = "1.0", features = ["full"] }
```

```rust
use reqwest::Error;

#[tokio::main]
async fn main() -> Result<(), Error> {
    // GET 请求
    let body = reqwest::get("https://www.rust-lang.org")
        .await?
        .text()
        .await?;
    
    println!("Body:\n{}", body);
    
    // POST 请求
    let client = reqwest::Client::new();
    let res = client.post("https://httpbin.org/post")
        .body("the exact body that is sent")
        .send()
        .await?;
    
    println!("Status: {}", res.status());
    
    Ok(())
}
```

### JSON 请求和响应

```rust
use reqwest::Error;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String,
    age: u32,
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    let client = reqwest::Client::new();
    
    // 发送 JSON
    let user = User {
        name: "Alice".to_string(),
        age: 30,
    };
    
    let res = client.post("https://httpbin.org/post")
        .json(&user)
        .send()
        .await?;
    
    println!("Status: {}", res.status());
    
    // 接收 JSON
    let user: User = reqwest::get("https://api.example.com/user")
        .await?
        .json()
        .await?;
    
    println!("User: {:?}", user);
    
    Ok(())
}
```

### 设置请求头

```rust
use reqwest::header;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = reqwest::Client::new();
    
    let res = client.get("https://api.example.com/data")
        .header(header::AUTHORIZATION, "Bearer token123")
        .header(header::CONTENT_TYPE, "application/json")
        .header(header::USER_AGENT, "MyApp/1.0")
        .send()
        .await?;
    
    println!("Status: {}", res.status());
    
    Ok(())
}
```

## 0x05 HTTP 服务器

### 使用 actix-web

```toml
# Cargo.toml
[dependencies]
actix-web = "4.0"
actix-rt = "2.0"
serde = { version = "1.0", features = ["derive"] }
```

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
    name: String,
    age: u32,
}

async fn index() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

async fn get_user() -> impl Responder {
    let user = User {
        name: "Alice".to_string(),
        age: 30,
    };
    HttpResponse::Ok().json(user)
}

async fn create_user(user: web::Json<User>) -> impl Responder {
    println!("Creating user: {:?}", user);
    HttpResponse::Created().json(user.into_inner())
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
            .route("/user", web::get().to(get_user))
            .route("/user", web::post().to(create_user))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 使用 axum

```toml
# Cargo.toml
[dependencies]
axum = "0.6"
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
```

```rust
use axum::{
    routing::{get, post},
    http::StatusCode,
    Json, Router,
};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
    name: String,
    age: u32,
}

async fn get_user() -> Json<User> {
    let user = User {
        name: "Alice".to_string(),
        age: 30,
    };
    Json(user)
}

async fn create_user(Json(user): Json<User>) -> StatusCode {
    println!("Creating user: {:?}", user);
    StatusCode::CREATED
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(|| async { "Hello, World!" }))
        .route("/user", get(get_user))
        .route("/user", post(create_user));
    
    axum::Server::bind(&"127.0.0.1:3000".parse().unwrap())
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

## 0x06 WebSocket

### 使用 tokio-tungstenite

```toml
# Cargo.toml
[dependencies]
tokio-tungstenite = "0.18"
tokio = { version = "1.0", features = ["full"] }
futures-util = "0.3"
```

```rust
use tokio_tungstenite::{connect_async, tungstenite::protocol::Message};
use futures_util::{SinkExt, StreamExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let (ws_stream, _) = connect_async("ws://127.0.0.1:8080/ws").await?;
    println!("Connected to WebSocket server");
    
    let (mut write, mut read) = ws_stream.split();
    
    // 发送消息
    write.send(Message::Text("Hello!".to_string())).await?;
    
    // 接收消息
    while let Some(msg) = read.next().await {
        match msg? {
            Message::Text(text) => println!("Received: {}", text),
            Message::Binary(bin) => println!("Received binary: {:?}", bin),
            _ => {}
        }
    }
    
    Ok(())
}
```

## 0x07 DNS 解析

### 使用 trust-dns

```toml
# Cargo.toml
[dependencies]
trust-dns-resolver = "0.22"
```

```rust
use trust_dns_resolver::config::*;
use trust_dns_resolver::Resolver;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let resolver = Resolver::new(ResolverConfig::default(), ResolverOpts::default())?;
    
    let response = resolver.lookup_ip("www.example.com")?;
    
    for address in response.iter() {
        println!("{}", address);
    }
    
    Ok(())
}
```

## 0x08 网络工具

### 端口扫描器

```rust
use std::net::TcpStream;
use std::time::Duration;

fn scan_port(host: &str, port: u16) -> bool {
    let addr = format!("{}:{}", host, port);
    match TcpStream::connect_timeout(&addr.parse().unwrap(), Duration::from_millis(100)) {
        Ok(_) => true,
        Err(_) => false,
    }
}

fn main() {
    let host = "127.0.0.1";
    let ports = [22, 80, 443, 3306, 5432, 8080];
    
    println!("Scanning {}...", host);
    
    for port in ports {
        if scan_port(host, port) {
            println!("Port {} is open", port);
        } else {
            println!("Port {} is closed", port);
        }
    }
}
```

### HTTP 状态码检查

```rust
use reqwest::StatusCode;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let urls = vec![
        "https://www.google.com",
        "https://www.github.com",
        "https://www.invalid-url-12345.com",
    ];
    
    for url in urls {
        match reqwest::get(url).await {
            Ok(response) => {
                println!("{}: {}", url, response.status());
            }
            Err(e) => {
                println!("{}: Error - {}", url, e);
            }
        }
    }
    
    Ok(())
}
```

## 0x09 安全连接

### HTTPS 客户端

```rust
use reqwest::Client;
use reqwest::Certificate;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    // 加载自定义 CA 证书
    let ca_cert = std::fs::read("ca-cert.pem").unwrap();
    let cert = Certificate::from_pem(&ca_cert).unwrap();
    
    let client = Client::builder()
        .add_root_certificate(cert)
        .build()?;
    
    let res = client.get("https://my-internal-server.com")
        .send()
        .await?;
    
    println!("Status: {}", res.status());
    
    Ok(())
}
```

### TLS 服务器

```rust
use tokio::net::TcpListener;
use tokio_rustls::TlsAcceptor;
use tokio_rustls::rustls::ServerConfig;
use std::sync::Arc;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let certs = load_certs("cert.pem")?;
    let key = load_private_key("key.pem")?;
    
    let config = ServerConfig::builder()
        .with_safe_defaults()
        .with_no_client_auth()
        .with_single_cert(certs, key)?;
    
    let acceptor = TlsAcceptor::from(Arc::new(config));
    let listener = TcpListener::bind("127.0.0.1:443").await?;
    
    loop {
        let (stream, _) = listener.accept().await?;
        let acceptor = acceptor.clone();
        
        tokio::spawn(async move {
            let mut stream = acceptor.accept(stream).await.unwrap();
            // 处理 TLS 连接
        });
    }
}
```

## 参考

- [Rust 标准库 - std::net](https://doc.rust-lang.org/std/net/)
- [tokio 文档](https://docs.rs/tokio/)
- [reqwest 文档](https://docs.rs/reqwest/)
- [actix-web 文档](https://actix.rs/)
- [axum 文档](https://docs.rs/axum/)
- [tungstenite 文档](https://docs.rs/tungstenite/)