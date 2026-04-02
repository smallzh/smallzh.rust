# 数据库操作

## 0x01 数据库概述

Rust 生态系统中有多个数据库操作库：

- **SQLx**：异步 SQL 工具包，支持多种数据库
- **Diesel**：类型安全的 ORM 和查询构建器
- **SeaORM**：基于 Diesel 的现代 ORM
- **rusqlite**：SQLite 绑定
- **tokio-postgres**：异步 PostgreSQL 客户端

## 0x02 SQLx

### 基本设置

```toml
# Cargo.toml
[dependencies]
sqlx = { version = "0.6", features = ["runtime-tokio-rustls", "sqlite"] }
tokio = { version = "1.0", features = ["full"] }
```

### SQLite 连接

```rust
use sqlx::sqlite::SqlitePool;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = SqlitePool::connect("sqlite:data.db").await?;
    
    // 创建表
    sqlx::query(
        "CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT NOT NULL
        )"
    ).execute(&pool).await?;
    
    println!("Database initialized");
    
    Ok(())
}
```

### 插入数据

```rust
use sqlx::sqlite::SqlitePool;

#[derive(Debug)]
struct User {
    id: i64,
    name: String,
    email: String,
}

async fn insert_user(pool: &SqlitePool, name: &str, email: &str) -> Result<i64, sqlx::Error> {
    let result = sqlx::query(
        "INSERT INTO users (name, email) VALUES (?, ?)"
    )
    .bind(name)
    .bind(email)
    .execute(pool)
    .await?;
    
    Ok(result.last_insert_rowid())
}

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = SqlitePool::connect("sqlite:data.db").await?;
    
    let user_id = insert_user(&pool, "Alice", "alice@example.com").await?;
    println!("Inserted user with ID: {}", user_id);
    
    Ok(())
}
```

### 查询数据

```rust
use sqlx::sqlite::SqlitePool;

async fn get_users(pool: &SqlitePool) -> Result<Vec<User>, sqlx::Error> {
    let users = sqlx::query_as::<_, User>(
        "SELECT id, name, email FROM users"
    )
    .fetch_all(pool)
    .await?;
    
    Ok(users)
}

async fn get_user_by_id(pool: &SqlitePool, id: i64) -> Result<Option<User>, sqlx::Error> {
    let user = sqlx::query_as::<_, User>(
        "SELECT id, name, email FROM users WHERE id = ?"
    )
    .bind(id)
    .fetch_optional(pool)
    .await?;
    
    Ok(user)
}
```

### 使用查询宏

```rust
use sqlx::sqlite::SqlitePool;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = SqlitePool::connect("sqlite:data.db").await?;
    
    // 使用 query! 宏进行编译时检查
    let users = sqlx::query!("SELECT id, name, email FROM users")
        .fetch_all(&pool)
        .await?;
    
    for user in users {
        println!("User: {} ({})", user.name, user.email);
    }
    
    Ok(())
}
```

## 0x03 Diesel

### 基本设置

```toml
# Cargo.toml
[dependencies]
diesel = { version = "1.4", features = ["sqlite"] }
dotenv = "0.15"
```

```bash
# 安装 diesel CLI
cargo install diesel_cli --no-default-features --features sqlite

# 初始化
diesel setup
```

### 定义模型

```rust
// src/schema.rs
table! {
    users (id) {
        id -> Integer,
        name -> Text,
        email -> Text,
    }
}

// src/models.rs
use super::schema::users;

#[derive(Queryable)]
pub struct User {
    pub id: i32,
    pub name: String,
    pub email: String,
}

#[derive(Insertable)]
#[table_name = "users"]
pub struct NewUser<'a> {
    pub name: &'a str,
    pub email: &'a str,
}
```

### 数据库操作

```rust
use diesel::prelude::*;
use diesel::sqlite::SqliteConnection;
use dotenv::dotenv;
use std::env;

fn establish_connection() -> SqliteConnection {
    dotenv().ok();
    
    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    
    SqliteConnection::establish(&database_url)
        .expect(&format!("Error connecting to {}", database_url))
}

fn create_user(conn: &SqliteConnection, name: &str, email: &str) -> User {
    use super::schema::users;
    
    let new_user = NewUser { name, email };
    
    diesel::insert_into(users::table)
        .values(&new_user)
        .execute(conn)
        .expect("Error inserting user");
    
    users::table.order(users::id.desc()).first(conn).unwrap()
}

fn get_users(conn: &SqliteConnection) -> Vec<User> {
    use super::schema::users::dsl::*;
    
    users.load::<User>(conn).expect("Error loading users")
}
```

## 0x04 SeaORM

### 基本设置

```toml
# Cargo.toml
[dependencies]
sea-orm = { version = "0.10", features = ["sqlx-sqlite", "runtime-tokio-rustls"] }
tokio = { version = "1.0", features = ["full"] }
```

### 定义实体

```rust
use sea_orm::entity::prelude::*;

#[derive(Clone, Debug, PartialEq, DeriveEntityModel)]
#[sea_orm(table_name = "users")]
pub struct Model {
    #[sea_orm(primary_key)]
    pub id: i32,
    pub name: String,
    pub email: String,
}

#[derive(Copy, Clone, Debug, EnumIter, DeriveRelation)]
pub enum Relation {}

impl ActiveModelBehavior for ActiveModel {}
```

### 数据库操作

```rust
use sea_orm::{Database, EntityTrait, Set};

#[tokio::main]
async fn main() -> Result<(), sea_orm::DbErr> {
    let db = Database::connect("sqlite:data.db").await?;
    
    // 插入数据
    let new_user = users::ActiveModel {
        name: Set("Alice".to_string()),
        email: Set("alice@example.com".to_string()),
        ..Default::default()
    };
    
    let result = users::Entity::insert(new_user).exec(&db).await?;
    println!("Inserted user with ID: {}", result.last_insert_id);
    
    // 查询数据
    let users = users::Entity::find().all(&db).await?;
    for user in users {
        println!("User: {} ({})", user.name, user.email);
    }
    
    Ok(())
}
```

## 0x05 PostgreSQL

### 使用 tokio-postgres

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1.0", features = ["full"] }
tokio-postgres = "0.7"
```

```rust
use tokio_postgres::{NoTls, Error};

#[tokio::main]
async fn main() -> Result<(), Error> {
    let (client, connection) =
        tokio_postgres::connect("host=localhost user=postgres", NoTls).await?;
    
    tokio::spawn(async move {
        if let Err(e) = connection.await {
            eprintln!("connection error: {}", e);
        }
    });
    
    // 创建表
    client.execute(
        "CREATE TABLE IF NOT EXISTS users (
            id SERIAL PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT NOT NULL
        )",
        &[],
    ).await?;
    
    // 插入数据
    client.execute(
        "INSERT INTO users (name, email) VALUES ($1, $2)",
        &[&"Alice", &"alice@example.com"],
    ).await?;
    
    // 查询数据
    let rows = client.query("SELECT id, name, email FROM users", &[]).await?;
    
    for row in rows {
        let id: i32 = row.get(0);
        let name: &str = row.get(1);
        let email: &str = row.get(2);
        
        println!("User: {} - {} ({})", id, name, email);
    }
    
    Ok(())
}
```

### 使用 SQLx 连接 PostgreSQL

```toml
# Cargo.toml
[dependencies]
sqlx = { version = "0.6", features = ["runtime-tokio-rustls", "postgres"] }
```

```rust
use sqlx::postgres::PgPool;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = PgPool::connect("postgres://postgres:password@localhost/testdb").await?;
    
    // 创建表
    sqlx::query(
        "CREATE TABLE IF NOT EXISTS users (
            id SERIAL PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT NOT NULL
        )"
    ).execute(&pool).await?;
    
    // 插入数据
    let row: (i32,) = sqlx::query_as(
        "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id"
    )
    .bind("Alice")
    .bind("alice@example.com")
    .fetch_one(&pool)
    .await?;
    
    println!("Inserted user with ID: {}", row.0);
    
    // 查询数据
    let users: Vec<(i32, String, String)> = sqlx::query_as(
        "SELECT id, name, email FROM users"
    )
    .fetch_all(&pool)
    .await?;
    
    for (id, name, email) in users {
        println!("User: {} - {} ({})", id, name, email);
    }
    
    Ok(())
}
```

## 0x06 Redis

### 使用 redis 库

```toml
# Cargo.toml
[dependencies]
redis = "0.22"
tokio = { version = "1.0", features = ["full"] }
```

```rust
use redis::Commands;

#[tokio::main]
async fn main() -> redis::RedisResult<()> {
    let client = redis::Client::open("redis://127.0.0.1/")?;
    let mut con = client.get_connection()?;
    
    // 设置键值
    con.set("key", "value")?;
    
    // 获取值
    let value: String = con.get("key")?;
    println!("Value: {}", value);
    
    // 设置带过期时间的键
    con.set_ex("temp_key", "temp_value", 60)?;
    
    // 删除键
    con.del("key")?;
    
    Ok(())
}
```

### 异步 Redis

```rust
use redis::AsyncCommands;

#[tokio::main]
async fn main() -> redis::RedisResult<()> {
    let client = redis::Client::open("redis://127.0.0.1/")?;
    let mut con = client.get_async_connection().await?;
    
    // 设置键值
    con.set("key", "value").await?;
    
    // 获取值
    let value: String = con.get("key").await?;
    println!("Value: {}", value);
    
    // 列表操作
    con.lpush("my_list", "item1").await?;
    con.lpush("my_list", "item2").await?;
    
    let items: Vec<String> = con.lrange("my_list", 0, -1).await?;
    println!("List items: {:?}", items);
    
    Ok(())
}
```

## 0x07 连接池

### SQLx 连接池

```rust
use sqlx::sqlite::SqlitePoolOptions;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = SqlitePoolOptions::new()
        .max_connections(5)
        .connect("sqlite:data.db")
        .await?;
    
    // 使用连接池
    let users = sqlx::query!("SELECT * FROM users")
        .fetch_all(&pool)
        .await?;
    
    println!("Found {} users", users.len());
    
    Ok(())
}
```

### 自定义连接池

```rust
use std::sync::Arc;
use tokio::sync::Mutex;

struct ConnectionPool<T> {
    connections: Arc<Mutex<Vec<T>>>,
    max_size: usize,
}

impl<T> ConnectionPool<T> {
    fn new(max_size: usize) -> Self {
        ConnectionPool {
            connections: Arc::new(Mutex::new(Vec::new())),
            max_size,
        }
    }
    
    async fn get_connection(&self) -> Option<T> {
        let mut connections = self.connections.lock().await;
        connections.pop()
    }
    
    async fn return_connection(&self, conn: T) {
        let mut connections = self.connections.lock().await;
        if connections.len() < self.max_size {
            connections.push(conn);
        }
    }
}
```

## 0x08 事务处理

### SQLx 事务

```rust
use sqlx::sqlite::SqlitePool;

async fn transfer_money(
    pool: &SqlitePool,
    from_id: i64,
    to_id: i64,
    amount: f64,
) -> Result<(), sqlx::Error> {
    let mut tx = pool.begin().await?;
    
    // 扣款
    sqlx::query("UPDATE accounts SET balance = balance - ? WHERE id = ?")
        .bind(amount)
        .bind(from_id)
        .execute(&mut tx)
        .await?;
    
    // 加款
    sqlx::query("UPDATE accounts SET balance = balance + ? WHERE id = ?")
        .bind(amount)
        .bind(to_id)
        .execute(&mut tx)
        .await?;
    
    tx.commit().await?;
    
    Ok(())
}
```

## 0x09 迁移管理

### SQLx 迁移

```sql
-- migrations/20230101000000_create_users.sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- migrations/20230101000001_add_age.sql
ALTER TABLE users ADD COLUMN age INTEGER;
```

```rust
use sqlx::sqlite::SqlitePool;
use sqlx::migrate::Migrator;

static MIGRATOR: Migrator = sqlx::migrate!();

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = SqlitePool::connect("sqlite:data.db").await?;
    
    MIGRATOR.run(&pool).await?;
    
    println!("Migrations completed");
    
    Ok(())
}
```

## 0x10 实际应用示例

### 用户管理系统

```rust
use sqlx::sqlite::SqlitePool;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
struct User {
    id: i64,
    name: String,
    email: String,
    age: i32,
}

#[derive(Debug, Deserialize)]
struct CreateUser {
    name: String,
    email: String,
    age: i32,
}

struct UserRepository {
    pool: SqlitePool,
}

impl UserRepository {
    async fn new(database_url: &str) -> Result<Self, sqlx::Error> {
        let pool = SqlitePool::connect(database_url).await?;
        
        // 创建表
        sqlx::query(
            "CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT NOT NULL UNIQUE,
                age INTEGER NOT NULL
            )"
        ).execute(&pool).await?;
        
        Ok(UserRepository { pool })
    }
    
    async fn create_user(&self, user: CreateUser) -> Result<User, sqlx::Error> {
        let result = sqlx::query(
            "INSERT INTO users (name, email, age) VALUES (?, ?, ?)"
        )
        .bind(&user.name)
        .bind(&user.email)
        .bind(user.age)
        .execute(&self.pool)
        .await?;
        
        Ok(User {
            id: result.last_insert_rowid(),
            name: user.name,
            email: user.email,
            age: user.age,
        })
    }
    
    async fn get_user(&self, id: i64) -> Result<Option<User>, sqlx::Error> {
        sqlx::query_as::<_, User>(
            "SELECT id, name, email, age FROM users WHERE id = ?"
        )
        .bind(id)
        .fetch_optional(&self.pool)
        .await
    }
    
    async fn list_users(&self) -> Result<Vec<User>, sqlx::Error> {
        sqlx::query_as::<_, User>(
            "SELECT id, name, email, age FROM users ORDER BY id"
        )
        .fetch_all(&self.pool)
        .await
    }
    
    async fn update_user(&self, id: i64, user: CreateUser) -> Result<Option<User>, sqlx::Error> {
        let result = sqlx::query(
            "UPDATE users SET name = ?, email = ?, age = ? WHERE id = ?"
        )
        .bind(&user.name)
        .bind(&user.email)
        .bind(user.age)
        .bind(id)
        .execute(&self.pool)
        .await?;
        
        if result.rows_affected() > 0 {
            self.get_user(id).await
        } else {
            Ok(None)
        }
    }
    
    async fn delete_user(&self, id: i64) -> Result<bool, sqlx::Error> {
        let result = sqlx::query("DELETE FROM users WHERE id = ?")
            .bind(id)
            .execute(&self.pool)
            .await?;
        
        Ok(result.rows_affected() > 0)
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let repo = UserRepository::new("sqlite:users.db").await?;
    
    // 创建用户
    let user = repo.create_user(CreateUser {
        name: "Alice".to_string(),
        email: "alice@example.com".to_string(),
        age: 30,
    }).await?;
    
    println!("Created user: {:?}", user);
    
    // 列出所有用户
    let users = repo.list_users().await?;
    println!("All users: {:?}", users);
    
    Ok(())
}
```

## 参考

- [SQLx 文档](https://docs.rs/sqlx/)
- [Diesel 文档](https://docs.rs/diesel/)
- [SeaORM 文档](https://docs.rs/sea-orm/)
- [rusqlite 文档](https://docs.rs/rusqlite/)
- [redis 文档](https://docs.rs/redis/)