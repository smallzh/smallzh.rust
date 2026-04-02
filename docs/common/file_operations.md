# 文件操作

## 0x01 文件读取

### 读取整个文件

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let contents = fs::read_to_string("hello.txt")?;
    println!("File content:\n{}", contents);
    
    Ok(())
}
```

### 读取二进制文件

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let bytes = fs::read("image.png")?;
    println!("File size: {} bytes", bytes.len());
    
    Ok(())
}
```

### 逐行读取

```rust
use std::fs::File;
use std::io::{self, BufRead};
use std::path::Path;

fn main() -> io::Result<()> {
    let file = File::open("hello.txt")?;
    let reader = io::BufReader::new(file);
    
    for line in reader.lines() {
        println!("{}", line?);
    }
    
    Ok(())
}
```

### 使用 BufRead 特征

```rust
use std::fs::File;
use std::io::{self, BufRead, BufReader};

fn main() -> io::Result<()> {
    let file = File::open("hello.txt")?;
    let mut reader = BufReader::new(file);
    
    let mut line = String::new();
    while reader.read_line(&mut line)? > 0 {
        print!("{}", line);
        line.clear();
    }
    
    Ok(())
}
```

## 0x02 文件写入

### 写入字符串

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::write("output.txt", "Hello, world!")?;
    println!("File written successfully");
    
    Ok(())
}
```

### 追加写入

```rust
use std::fs::OpenOptions;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let mut file = OpenOptions::new()
        .append(true)
        .create(true)
        .open("output.txt")?;
    
    file.write_all(b"\nThis line was appended")?;
    
    Ok(())
}
```

### 写入二进制数据

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let data = vec![0x48, 0x65, 0x6c, 0x6c, 0x6f]; // "Hello"
    fs::write("output.bin", data)?;
    
    Ok(())
}
```

## 0x03 目录操作

### 创建目录

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // 创建单个目录
    fs::create_dir("new_dir")?;
    
    // 递归创建目录
    fs::create_dir_all("path/to/nested/dir")?;
    
    Ok(())
}
```

### 读取目录

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    for entry in fs::read_dir(".")? {
        let entry = entry?;
        let path = entry.path();
        
        if path.is_file() {
            println!("File: {}", path.display());
        } else if path.is_dir() {
            println!("Dir: {}", path.display());
        }
    }
    
    Ok(())
}
```

### 递归遍历目录

```rust
use std::fs;
use std::path::Path;

fn visit_dirs(dir: &Path) -> std::io::Result<()> {
    if dir.is_dir() {
        for entry in fs::read_dir(dir)? {
            let entry = entry?;
            let path = entry.path();
            
            if path.is_dir() {
                visit_dirs(&path)?;
            } else {
                println!("{}", path.display());
            }
        }
    }
    
    Ok(())
}

fn main() -> std::io::Result<()> {
    visit_dirs(Path::new("."))
}
```

## 0x04 文件元数据

### 获取文件信息

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let metadata = fs::metadata("hello.txt")?;
    
    println!("File size: {} bytes", metadata.len());
    println!("Is file: {}", metadata.is_file());
    println!("Is directory: {}", metadata.is_dir());
    println!("Modified: {:?}", metadata.modified()?);
    
    Ok(())
}
```

### 符号链接

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // 创建符号链接
    #[cfg(unix)]
    fs::symlink("original.txt", "link.txt")?;
    
    #[cfg(windows)]
    fs::symlink_file("original.txt", "link.txt")?;
    
    // 读取符号链接
    let target = fs::read_link("link.txt")?;
    println!("Target: {}", target.display());
    
    Ok(())
}
```

## 0x05 文件权限

### Unix 权限

```rust
use std::fs;
use std::os::unix::fs::PermissionsExt;

fn main() -> std::io::Result<()> {
    let metadata = fs::metadata("hello.txt")?;
    let permissions = metadata.permissions();
    
    println!("Permissions: {:o}", permissions.mode());
    
    // 设置权限
    let mut permissions = fs::metadata("hello.txt")?.permissions();
    permissions.set_mode(0o644);
    fs::set_permissions("hello.txt", permissions)?;
    
    Ok(())
}
```

## 0x06 临时文件

### 使用 tempfile 库

```toml
# Cargo.toml
[dependencies]
tempfile = "3.0"
```

```rust
use tempfile::NamedTempFile;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let mut temp_file = NamedTempFile::new()?;
    
    writeln!(temp_file, "Hello, world!")?;
    
    println!("Temp file path: {:?}", temp_file.path());
    
    // 文件会在 temp_file 离开作用域时自动删除
    
    Ok(())
}
```

## 0x07 文件锁

### 使用 file-lock 库

```toml
# Cargo.toml
[dependencies]
file-lock = "1.0"
```

```rust
use file_lock::FileLock;
use std::fs::OpenOptions;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let file = OpenOptions::new()
        .write(true)
        .create(true)
        .open("locked.txt")?;
    
    let mut lock = FileLock::new(file);
    
    // 获取锁
    lock.lock()?;
    
    // 写入数据
    lock.file.write_all(b"Locked data")?;
    
    // 锁会在 lock 离开作用域时自动释放
    
    Ok(())
}
```

## 0x08 文件监控

### 使用 notify 库

```toml
# Cargo.toml
[dependencies]
notify = "5.0"
```

```rust
use notify::{Watcher, RecursiveMode, watcher};
use std::sync::mpsc::channel;
use std::time::Duration;

fn main() -> notify::Result<()> {
    let (tx, rx) = channel();
    
    let mut watcher = watcher(tx, Duration::from_secs(1))?;
    watcher.watch(".", RecursiveMode::Recursive)?;
    
    loop {
        match rx.recv() {
            Ok(event) => println!("{:?}", event),
            Err(e) => println!("watch error: {:?}", e),
        }
    }
}
```

## 0x09 压缩文件

### 使用 flate2 库

```toml
# Cargo.toml
[dependencies]
flate2 = "1.0"
```

```rust
use flate2::write::GzEncoder;
use flate2::read::GzDecoder;
use flate2::Compression;
use std::io::prelude::*;

fn main() -> std::io::Result<()> {
    // 压缩
    let mut encoder = GzEncoder::new(Vec::new(), Compression::default());
    encoder.write_all(b"Hello, world!")?;
    let compressed = encoder.finish()?;
    
    println!("Compressed size: {} bytes", compressed.len());
    
    // 解压
    let mut decoder = GzDecoder::new(&compressed[..]);
    let mut decompressed = String::new();
    decoder.read_to_string(&mut decompressed)?;
    
    println!("Decompressed: {}", decompressed);
    
    Ok(())
}
```

## 0x10 实际应用示例

### 示例 1：配置文件管理

```rust
use serde::{Deserialize, Serialize};
use std::fs;
use std::path::Path;

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    database_url: String,
    port: u16,
    debug: bool,
}

impl Config {
    fn load(path: &str) -> Result<Config, Box<dyn std::error::Error>> {
        if Path::new(path).exists() {
            let content = fs::read_to_string(path)?;
            let config: Config = toml::from_str(&content)?;
            Ok(config)
        } else {
            let config = Config {
                database_url: "sqlite:data.db".to_string(),
                port: 8080,
                debug: false,
            };
            config.save(path)?;
            Ok(config)
        }
    }
    
    fn save(&self, path: &str) -> Result<(), Box<dyn std::error::Error>> {
        let content = toml::to_string(self)?;
        fs::write(path, content)?;
        Ok(())
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = Config::load("config.toml")?;
    println!("Config: {:?}", config);
    
    Ok(())
}
```

### 示例 2：日志文件管理

```rust
use chrono::Local;
use std::fs;
use std::io::Write;

struct Logger {
    log_dir: String,
}

impl Logger {
    fn new(log_dir: &str) -> std::io::Result<Logger> {
        fs::create_dir_all(log_dir)?;
        Ok(Logger {
            log_dir: log_dir.to_string(),
        })
    }
    
    fn log(&self, level: &str, message: &str) -> std::io::Result<()> {
        let timestamp = Local::now().format("%Y-%m-%d %H:%M:%S");
        let log_message = format!("[{}] {}: {}", timestamp, level, message);
        
        let date = Local::now().format("%Y-%m-%d");
        let filename = format!("{}/{}.log", self.log_dir, date);
        
        let mut file = fs::OpenOptions::new()
            .create(true)
            .append(true)
            .open(filename)?;
        
        writeln!(file, "{}", log_message)?;
        
        println!("{}", log_message);
        
        Ok(())
    }
    
    fn info(&self, message: &str) -> std::io::Result<()> {
        self.log("INFO", message)
    }
    
    fn error(&self, message: &str) -> std::io::Result<()> {
        self.log("ERROR", message)
    }
}

fn main() -> std::io::Result<()> {
    let logger = Logger::new("logs")?;
    
    logger.info("Application started")?;
    logger.error("Something went wrong")?;
    
    Ok(())
}
```

### 示例 3：批量文件处理

```rust
use std::fs;
use std::path::Path;

fn process_files(dir: &str, extension: &str) -> std::io::Result<Vec<String>> {
    let mut results = Vec::new();
    
    for entry in fs::read_dir(dir)? {
        let entry = entry?;
        let path = entry.path();
        
        if path.is_file() {
            if let Some(ext) = path.extension() {
                if ext == extension {
                    let content = fs::read_to_string(&path)?;
                    let processed = content.to_uppercase();
                    
                    let output_path = path.with_extension(format!("{}.processed", extension));
                    fs::write(&output_path, &processed)?;
                    
                    results.push(output_path.display().to_string());
                }
            }
        }
    }
    
    Ok(results)
}

fn main() -> std::io::Result<()> {
    let processed = process_files(".", "txt")?;
    println!("Processed files:");
    for file in processed {
        println!("  {}", file);
    }
    
    Ok(())
}
```

## 参考

- [Rust 标准库 - std::fs](https://doc.rust-lang.org/std/fs/)
- [Rust 标准库 - std::io](https://doc.rust-lang.org/std/io/)
- [Rust by Example - 文件 I/O](https://doc.rust-lang.org/rust-by-example/std_misc/file.html)
- [tempfile 文档](https://docs.rs/tempfile/)
- [notify 文档](https://docs.rs/notify/)