# Web框架

## 0x01 Web框架概述

Rust 生态系统中有多个优秀的 Web 框架：

- **Actix-web**：高性能、类型安全的 Web 框架
- **Rocket**：注重易用性和安全性的框架
- **Axum**：基于 tokio 的模块化框架
- **Warp**：轻量级、可组合的 Web 框架

## 0x02 Actix-web

### 基本应用

```toml
# Cargo.toml
[dependencies]
actix-web = "4.0"
actix-rt = "2.0"
serde = { version = "1.0", features = ["derive"] }
```

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

async fn index() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

async fn greet(name: web::Path<String>) -> impl Responder {
    format!("Hello {}!", name)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
            .route("/greet/{name}", web::get().to(greet))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 处理 JSON

```rust
use actix_web::{web, App, HttpResponse, HttpServer};
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct CreateUser {
    username: String,
    email: String,
}

#[derive(Serialize)]
struct User {
    id: u32,
    username: String,
    email: String,
}

async fn create_user(user: web::Json<CreateUser>) -> HttpResponse {
    let new_user = User {
        id: 1,
        username: user.username.clone(),
        email: user.email.clone(),
    };
    
    HttpResponse::Created().json(new_user)
}

async fn get_users() -> HttpResponse {
    let users = vec![
        User {
            id: 1,
            username: "alice".to_string(),
            email: "alice@example.com".to_string(),
        },
        User {
            id: 2,
            username: "bob".to_string(),
            email: "bob@example.com".to_string(),
        },
    ];
    
    HttpResponse::Ok().json(users)
}
```

### 状态管理

```rust
use actix_web::{web, App, HttpServer, HttpResponse};
use std::sync::Mutex;

struct AppState {
    app_name: String,
    counter: Mutex<i32>,
}

async fn index(data: web::Data<AppState>) -> String {
    let app_name = &data.app_name;
    let mut counter = data.counter.lock().unwrap();
    *counter += 1;
    
    format!("{}: {} visits!", app_name, counter)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let data = web::Data::new(AppState {
        app_name: String::from("Actix-web"),
        counter: Mutex::new(0),
    });
    
    HttpServer::new(move || {
        App::new()
            .app_data(data.clone())
            .route("/", web::get().to(index))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 中间件

```rust
use actix_web::middleware::{Logger, Compress};
use actix_web::{web, App, HttpServer};

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap(Logger::default())
            .wrap(Compress::default())
            .route("/", web::get().to(|| async { "Hello world!" }))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## 0x03 Rocket

### 基本应用

```toml
# Cargo.toml
[dependencies]
rocket = "0.5"
serde = { version = "1.0", features = ["derive"] }
```

```rust
#[macro_use] extern crate rocket;

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

#[get("/hello/<name>")]
fn hello(name: &str) -> String {
    format!("Hello, {}!", name)
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![index, hello])
}
```

### JSON 处理

```rust
use rocket::serde::{Deserialize, Serialize, json::Json};

#[derive(Deserialize)]
#[serde(crate = "rocket::serde")]
struct CreateUser {
    username: String,
    email: String,
}

#[derive(Serialize)]
#[serde(crate = "rocket::serde")]
struct User {
    id: u32,
    username: String,
    email: String,
}

#[post("/users", format = "json", data = "<user>")]
fn create_user(user: Json<CreateUser>) -> Json<User> {
    let new_user = User {
        id: 1,
        username: user.username.clone(),
        email: user.email.clone(),
    };
    
    Json(new_user)
}

#[get("/users")]
fn get_users() -> Json<Vec<User>> {
    let users = vec![
        User {
            id: 1,
            username: "alice".to_string(),
            email: "alice@example.com".to_string(),
        },
        User {
            id: 2,
            username: "bob".to_string(),
            email: "bob@example.com".to_string(),
        },
    ];
    
    Json(users)
}
```

### 状态管理

```rust
use rocket::State;
use std::sync::atomic::{AtomicUsize, Ordering};

struct HitCount {
    count: AtomicUsize,
}

#[get("/")]
fn index(hit_count: &State<HitCount>) -> String {
    let count = hit_count.count.fetch_add(1, Ordering::Relaxed);
    format!("This page has been visited {} times", count)
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .manage(HitCount {
            count: AtomicUsize::new(0),
        })
        .mount("/", routes![index])
}
```

## 0x04 Axum

### 基本应用

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
    id: u32,
    name: String,
    email: String,
}

async fn index() -> &'static str {
    "Hello, World!"
}

async fn create_user(Json(user): Json<User>) -> (StatusCode, Json<User>) {
    (StatusCode::CREATED, Json(user))
}

async fn get_users() -> Json<Vec<User>> {
    let users = vec![
        User {
            id: 1,
            name: "Alice".to_string(),
            email: "alice@example.com".to_string(),
        },
        User {
            id: 2,
            name: "Bob".to_string(),
            email: "bob@example.com".to_string(),
        },
    ];
    
    Json(users)
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(index))
        .route("/users", get(get_users).post(create_user));
    
    axum::Server::bind(&"127.0.0.1:3000".parse().unwrap())
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

### 提取器

```rust
use axum::{
    extract::{Path, Query, Json},
    routing::get,
    Router,
};
use serde::Deserialize;

#[derive(Deserialize)]
struct Pagination {
    page: Option<u32>,
    limit: Option<u32>,
}

async fn get_user(Path(id): Path<u32>) -> String {
    format!("User ID: {}", id)
}

async fn list_users(Query(pagination): Query<Pagination>) -> String {
    format!(
        "Page: {}, Limit: {}",
        pagination.page.unwrap_or(1),
        pagination.limit.unwrap_or(10)
    )
}

async fn create_user(Json(user): Json<User>) -> Json<User> {
    Json(user)
}
```

### 中间件

```rust
use axum::{
    middleware::{self, Next},
    extract::Request,
    response::Response,
    routing::get,
    Router,
};
use std::time::Instant;

async fn logging_middleware(request: Request, next: Next) -> Response {
    let start = Instant::now();
    let method = request.method().clone();
    let uri = request.uri().clone();
    
    let response = next.run(request).await;
    
    let duration = start.elapsed();
    println!("{} {} - {} - {:?}", method, uri, response.status(), duration);
    
    response
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(|| async { "Hello, World!" }))
        .layer(middleware::from_fn(logging_middleware));
    
    axum::Server::bind(&"127.0.0.1:3000".parse().unwrap())
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

## 0x05 Warp

### 基本应用

```toml
# Cargo.toml
[dependencies]
warp = "0.3"
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
```

```rust
use warp::Filter;

#[tokio::main]
async fn main() {
    let hello = warp::path!("hello" / String)
        .map(|name| format!("Hello, {}!", name));
    
    let routes = hello;
    
    warp::serve(routes)
        .run(([127, 0, 0, 1], 3030))
        .await;
}
```

### JSON 处理

```rust
use warp::Filter;
use serde::{Deserialize, Serialize};

#[derive(Deserialize, Serialize)]
struct User {
    id: u32,
    name: String,
    email: String,
}

#[tokio::main]
async fn main() {
    let create_user = warp::post()
        .and(warp::path("users"))
        .and(warp::body::json())
        .map(|user: User| {
            warp::reply::json(&user)
        });
    
    let get_users = warp::get()
        .and(warp::path("users"))
        .map(|| {
            let users = vec![
                User {
                    id: 1,
                    name: "Alice".to_string(),
                    email: "alice@example.com".to_string(),
                },
                User {
                    id: 2,
                    name: "Bob".to_string(),
                    email: "bob@example.com".to_string(),
                },
            ];
            warp::reply::json(&users)
        });
    
    let routes = create_user.or(get_users);
    
    warp::serve(routes)
        .run(([127, 0, 0, 1], 3030))
        .await;
}
```

## 0x06 数据库集成

### 使用 SQLx

```toml
# Cargo.toml
[dependencies]
sqlx = { version = "0.6", features = ["runtime-tokio-rustls", "sqlite"] }
```

```rust
use actix_web::{web, App, HttpServer, HttpResponse};
use sqlx::sqlite::SqlitePool;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, sqlx::FromRow)]
struct User {
    id: i64,
    name: String,
    email: String,
}

async fn get_users(pool: web::Data<SqlitePool>) -> HttpResponse {
    match sqlx::query_as::<_, User>("SELECT id, name, email FROM users")
        .fetch_all(pool.get_ref())
        .await
    {
        Ok(users) => HttpResponse::Ok().json(users),
        Err(e) => HttpResponse::InternalServerError().body(format!("Error: {}", e)),
    }
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let pool = SqlitePool::connect("sqlite:users.db").await.unwrap();
    
    HttpServer::new(move || {
        App::new()
            .app_data(web::Data::new(pool.clone()))
            .route("/users", web::get().to(get_users))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## 0x07 模板引擎

### 使用 Tera

```toml
# Cargo.toml
[dependencies]
actix-web = "4.0"
actix-files = "0.6"
tera = "1.0"
```

```rust
use actix_web::{web, App, HttpResponse, HttpServer};
use tera::{Tera, Context};

async fn index(tmpl: web::Data<Tera>) -> HttpResponse {
    let mut context = Context::new();
    context.insert("title", "Home");
    context.insert("name", "World");
    
    let rendered = tmpl.render("index.html", &context).unwrap();
    HttpResponse::Ok().body(rendered)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let tera = Tera::new("templates/**/*").unwrap();
    
    HttpServer::new(move || {
        App::new()
            .app_data(web::Data::new(tera.clone()))
            .service(actix_files::Files::new("/static", "./static"))
            .route("/", web::get().to(index))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## 0x08 认证与授权

### JWT 认证

```toml
# Cargo.toml
[dependencies]
actix-web = "4.0"
jsonwebtoken = "8.0"
serde = { version = "1.0", features = ["derive"] }
```

```rust
use actix_web::{web, App, HttpRequest, HttpResponse, HttpServer};
use jsonwebtoken::{encode, decode, Header, Validation, EncodingKey, DecodingKey};
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct Claims {
    sub: String,
    exp: usize,
}

async fn login() -> HttpResponse {
    let claims = Claims {
        sub: "user@example.com".to_string(),
        exp: 10000000000,
    };
    
    let token = encode(
        &Header::default(),
        &claims,
        &EncodingKey::from_secret("secret".as_ref()),
    ).unwrap();
    
    HttpResponse::Ok().json(serde_json::json!({ "token": token }))
}

async fn protected(req: HttpRequest) -> HttpResponse {
    let token = req.headers()
        .get("Authorization")
        .and_then(|v| v.to_str().ok())
        .and_then(|v| v.strip_prefix("Bearer "));
    
    match token {
        Some(token) => {
            match decode::<Claims>(
                token,
                &DecodingKey::from_secret("secret".as_ref()),
                &Validation::default(),
            ) {
                Ok(token_data) => {
                    HttpResponse::Ok().body(format!("Welcome, {}", token_data.claims.sub))
                }
                Err(e) => {
                    HttpResponse::Unauthorized().body(format!("Invalid token: {}", e))
                }
            }
        }
        None => {
            HttpResponse::Unauthorized().body("Missing token")
        }
    }
}
```

## 0x09 测试

### 使用 actix-web 测试

```rust
use actix_web::{test, web, App};
use serde_json::json;

#[actix_web::test]
async fn test_index() {
    let app = test::init_service(
        App::new().route("/", web::get().to(|| async { "Hello, world!" }))
    ).await;
    
    let req = test::TestRequest::get().uri("/").to_request();
    let resp = test::call_service(&app, req).await;
    
    assert!(resp.status().is_success());
    
    let body = test::read_body(resp).await;
    assert_eq!(body, "Hello, world!");
}

#[actix_web::test]
async fn test_create_user() {
    let app = test::init_service(
        App::new().route("/users", web::post().to(create_user))
    ).await;
    
    let req = test::TestRequest::post()
        .uri("/users")
        .set_json(json!({
            "username": "alice",
            "email": "alice@example.com"
        }))
        .to_request();
    
    let resp = test::call_service(&app, req).await;
    assert!(resp.status().is_success());
}
```

## 参考

- [actix-web 文档](https://actix.rs/)
- [Rocket 文档](https://rocket.rs/)
- [axum 文档](https://docs.rs/axum/)
- [warp 文档](https://docs.rs/warp/)
- [tera 文档](https://docs.rs/tera/)
- [jsonwebtoken 文档](https://docs.rs/jsonwebtoken/)