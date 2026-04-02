# 序列化与反序列化

## 0x01 序列化概述

序列化是将数据结构或对象状态转换为可以存储或传输的格式的过程。反序列化则是相反的过程。

### 常用格式

- JSON - 轻量级数据交换格式
- YAML - 人类可读的数据序列化格式
- TOML - 配置文件格式
- MessagePack - 高效的二进制格式
- CSV - 表格数据格式

## 0x02 JSON 处理

### 使用 serde_json

```toml
# Cargo.toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u32,
    email: String,
}

fn main() -> Result<(), serde_json::Error> {
    let person = Person {
        name: "Alice".to_string(),
        age: 30,
        email: "alice@example.com".to_string(),
    };
    
    // 序列化为 JSON 字符串
    let json = serde_json::to_string(&person)?;
    println!("JSON: {}", json);
    
    // 序列化为格式化的 JSON
    let json_pretty = serde_json::to_string_pretty(&person)?;
    println!("Pretty JSON:\n{}", json_pretty);
    
    // 从 JSON 字符串反序列化
    let person2: Person = serde_json::from_str(&json)?;
    println!("Person: {:?}", person2);
    
    Ok(())
}
```

### 处理 JSON 值

```rust
use serde_json::{Value, json};

fn main() -> Result<(), serde_json::Error> {
    // 创建 JSON 值
    let value = json!({
        "name": "Alice",
        "age": 30,
        "address": {
            "city": "Beijing",
            "country": "China"
        },
        "hobbies": ["reading", "coding", "gaming"]
    });
    
    println!("JSON: {}", value);
    
    // 访问字段
    if let Some(name) = value.get("name") {
        println!("Name: {}", name);
    }
    
    if let Some(city) = value["address"]["city"].as_str() {
        println!("City: {}", city);
    }
    
    // 遍历数组
    if let Some(hobbies) = value["hobbies"].as_array() {
        for hobby in hobbies {
            println!("Hobby: {}", hobby);
        }
    }
    
    Ok(())
}
```

### 处理动态 JSON

```rust
use serde_json::Value;

fn main() -> Result<(), serde_json::Error> {
    let json_str = r#"
    {
        "name": "Alice",
        "age": 30,
        "active": true,
        "scores": [95, 87, 92],
        "address": null
    }
    "#;
    
    let value: Value = serde_json::from_str(json_str)?;
    
    // 检查类型
    match &value["name"] {
        Value::String(s) => println!("Name is string: {}", s),
        _ => println!("Name is not a string"),
    }
    
    match &value["active"] {
        Value::Bool(b) => println!("Active is bool: {}", b),
        _ => println!("Active is not a bool"),
    }
    
    // 获取嵌套值
    if let Some(score) = value["scores"].get(0) {
        println!("First score: {}", score);
    }
    
    Ok(())
}
```

## 0x03 YAML 处理

### 使用 serde_yaml

```toml
# Cargo.toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_yaml = "0.9"
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    database: Database,
    server: Server,
}

#[derive(Serialize, Deserialize, Debug)]
struct Database {
    host: String,
    port: u16,
    username: String,
    password: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct Server {
    host: String,
    port: u16,
    workers: u32,
}

fn main() -> Result<(), serde_yaml::Error> {
    let yaml_str = r#"
database:
  host: localhost
  port: 5432
  username: admin
  password: secret
server:
  host: 0.0.0.0
  port: 8080
  workers: 4
"#;
    
    let config: Config = serde_yaml::from_str(yaml_str)?;
    println!("Config: {:?}", config);
    
    // 序列化为 YAML
    let yaml = serde_yaml::to_string(&config)?;
    println!("YAML:\n{}", yaml);
    
    Ok(())
}
```

## 0x04 TOML 处理

### 使用 toml 库

```toml
# Cargo.toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
toml = "0.5"
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    title: String,
    database: Database,
    servers: Vec<Server>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Database {
    host: String,
    port: u16,
}

#[derive(Serialize, Deserialize, Debug)]
struct Server {
    name: String,
    host: String,
    port: u16,
}

fn main() -> Result<(), toml::de::Error> {
    let toml_str = r#"
title = "My Application"

[database]
host = "localhost"
port = 5432

[[servers]]
name = "alpha"
host = "127.0.0.1"
port = 8080

[[servers]]
name = "beta"
host = "127.0.0.1"
port = 8081
"#;
    
    let config: Config = toml::from_str(toml_str)?;
    println!("Config: {:?}", config);
    
    // 序列化为 TOML
    let toml = toml::to_string(&config).unwrap();
    println!("TOML:\n{}", toml);
    
    Ok(())
}
```

## 0x05 CSV 处理

### 使用 csv 库

```toml
# Cargo.toml
[dependencies]
csv = "1.1"
serde = { version = "1.0", features = ["derive"] }
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Record {
    name: String,
    age: u32,
    city: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 读取 CSV
    let mut rdr = csv::Reader::from_reader(std::io::stdin());
    
    for result in rdr.deserialize() {
        let record: Record = result?;
        println!("{:?}", record);
    }
    
    // 写入 CSV
    let mut wtr = csv::Writer::from_writer(std::io::stdout());
    
    wtr.write_record(&["name", "age", "city"])?;
    wtr.serialize(("Alice", 30, "Beijing"))?;
    wtr.serialize(("Bob", 25, "Shanghai"))?;
    
    wtr.flush()?;
    
    Ok(())
}
```

## 0x06 MessagePack 处理

### 使用 rmp-serde

```toml
# Cargo.toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
rmp-serde = "1.0"
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Data {
    id: u32,
    name: String,
    values: Vec<f64>,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let data = Data {
        id: 1,
        name: "test".to_string(),
        values: vec![1.0, 2.0, 3.0],
    };
    
    // 序列化为 MessagePack
    let bytes = rmp_serde::to_vec(&data)?;
    println!("Encoded: {:?}", bytes);
    
    // 反序列化
    let decoded: Data = rmp_serde::from_slice(&bytes)?;
    println!("Decoded: {:?}", decoded);
    
    Ok(())
}
```

## 0x07 自定义序列化

### 使用 serde 属性

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    #[serde(rename = "user_name")]
    name: String,
    
    #[serde(skip_serializing_if = "Option::is_none")]
    email: Option<String>,
    
    #[serde(default)]
    age: u32,
    
    #[serde(with = "date_format")]
    created_at: chrono::NaiveDate,
}

mod date_format {
    use chrono::NaiveDate;
    use serde::{self, Deserialize, Serializer, Deserializer};
    
    const FORMAT: &str = "%Y-%m-%d";
    
    pub fn serialize<S>(date: &NaiveDate, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let s = date.format(FORMAT).to_string();
        serializer.serialize_str(&s)
    }
    
    pub fn deserialize<'de, D>(deserializer: D) -> Result<NaiveDate, D::Error>
    where
        D: Deserializer<'de>,
    {
        let s = String::deserialize(deserializer)?;
        NaiveDate::parse_from_str(&s, FORMAT).map_err(serde::de::Error::custom)
    }
}
```

### 自定义枚举序列化

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "type", content = "data")]
enum Message {
    Text(String),
    Number(i32),
    Complex { x: f64, y: f64 },
}

fn main() -> Result<(), serde_json::Error> {
    let messages = vec![
        Message::Text("hello".to_string()),
        Message::Number(42),
        Message::Complex { x: 1.0, y: 2.0 },
    ];
    
    let json = serde_json::to_string_pretty(&messages)?;
    println!("JSON:\n{}", json);
    
    let decoded: Vec<Message> = serde_json::from_str(&json)?;
    println!("Decoded: {:?}", decoded);
    
    Ok(())
}
```

## 0x08 流式处理

### 处理大型 JSON 数组

```rust
use serde::{Deserialize, Serialize};
use std::io::{BufRead, BufReader};

#[derive(Serialize, Deserialize, Debug)]
struct Item {
    id: u32,
    name: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let file = std::fs::File::open("large.json")?;
    let reader = BufReader::new(file);
    
    let mut stream = serde_json::Deserializer::from_reader(reader).into_iter::<Vec<Item>>();
    
    while let Some(Ok(items)) = stream.next() {
        for item in items {
            println!("Item: {:?}", item);
        }
    }
    
    Ok(())
}
```

## 0x09 性能优化

### 使用 simd-json

```toml
# Cargo.toml
[dependencies]
simd-json = "0.6"
serde = { version = "1.0", features = ["derive"] }
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Data {
    id: u32,
    name: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut json = r#"{"id": 1, "name": "test"}"#.to_string();
    
    // 使用 simd-json 解析
    let data: Data = simd_json::from_str(&mut json)?;
    println!("Parsed: {:?}", data);
    
    Ok(())
}
```

## 0x10 实际应用示例

### 配置文件管理

```rust
use serde::{Deserialize, Serialize};
use std::path::Path;

#[derive(Serialize, Deserialize, Debug)]
struct AppConfig {
    database: DatabaseConfig,
    server: ServerConfig,
    features: FeatureConfig,
}

#[derive(Serialize, Deserialize, Debug)]
struct DatabaseConfig {
    url: String,
    pool_size: u32,
    timeout: u64,
}

#[derive(Serialize, Deserialize, Debug)]
struct ServerConfig {
    host: String,
    port: u16,
    workers: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct FeatureConfig {
    enable_logging: bool,
    enable_cache: bool,
    max_connections: u32,
}

impl AppConfig {
    fn load(path: &str) -> Result<Self, Box<dyn std::error::Error>> {
        if Path::new(path).exists() {
            let content = std::fs::read_to_string(path)?;
            let config: AppConfig = toml::from_str(&content)?;
            Ok(config)
        } else {
            let config = AppConfig::default();
            config.save(path)?;
            Ok(config)
        }
    }
    
    fn save(&self, path: &str) -> Result<(), Box<dyn std::error::Error>> {
        let content = toml::to_string(self)?;
        std::fs::write(path, content)?;
        Ok(())
    }
}

impl Default for AppConfig {
    fn default() -> Self {
        AppConfig {
            database: DatabaseConfig {
                url: "sqlite:data.db".to_string(),
                pool_size: 10,
                timeout: 30,
            },
            server: ServerConfig {
                host: "127.0.0.1".to_string(),
                port: 8080,
                workers: 4,
            },
            features: FeatureConfig {
                enable_logging: true,
                enable_cache: true,
                max_connections: 100,
            },
        }
    }
}
```

### API 响应处理

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct ApiResponse<T> {
    success: bool,
    data: Option<T>,
    error: Option<String>,
    timestamp: u64,
}

#[derive(Serialize, Deserialize, Debug)]
struct User {
    id: u32,
    name: String,
    email: String,
}

fn main() -> Result<(), serde_json::Error> {
    // 成功响应
    let success_response = ApiResponse {
        success: true,
        data: Some(User {
            id: 1,
            name: "Alice".to_string(),
            email: "alice@example.com".to_string(),
        }),
        error: None,
        timestamp: 1640995200,
    };
    
    let json = serde_json::to_string_pretty(&success_response)?;
    println!("Success response:\n{}", json);
    
    // 错误响应
    let error_response: ApiResponse<User> = ApiResponse {
        success: false,
        data: None,
        error: Some("User not found".to_string()),
        timestamp: 1640995200,
    };
    
    let json = serde_json::to_string_pretty(&error_response)?;
    println!("Error response:\n{}", json);
    
    Ok(())
}
```

## 参考

- [serde 文档](https://serde.rs/)
- [serde_json 文档](https://docs.rs/serde_json/)
- [serde_yaml 文档](https://docs.rs/serde_yaml/)
- [toml 文档](https://docs.rs/toml/)
- [csv 文档](https://docs.rs/csv/)
- [rmp-serde 文档](https://docs.rs/rmp-serde/)