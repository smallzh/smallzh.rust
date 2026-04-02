# Rust 简介与安装

## 0x01 什么是 Rust？

Rust 是一种系统编程语言，专注于三个目标：安全性、并发性和性能。它由 Mozilla 研究院于 2010 年首次公开发布，旨在解决 C++ 等语言中常见的内存安全问题，同时保持高性能。

### Rust 的设计哲学

1. **内存安全**：通过所有权（Ownership）、借用（Borrowing）和生命周期（Lifetimes）系统，在编译时防止内存错误
2. **零成本抽象**：高级抽象（如迭代器、闭包）不会带来运行时性能损失
3. **并发安全**：编译器防止数据竞争，使并发编程更安全
4. **实用性**：注重实际开发体验，提供优秀的工具链

### Rust 与其他语言的比较

| 特性 | Rust | C++ | Go | Python |
|------|------|-----|----|--------|
| 内存安全 | 编译时保证 | 手动管理 | 垃圾回收 | 垃圾回收 |
| 性能 | 接近 C++ | 最高 | 良好 | 较低 |
| 并发安全 | 编译时保证 | 需要手动管理 | 内置支持 | 受 GIL 限制 |
| 学习曲线 | 陡峭 | 陡峭 | 平缓 | 平缓 |
| 生态系统 | 快速成长 | 成熟 | 成熟 | 非常成熟 |

## 0x02 安装 Rust

### 使用 rustup 安装（推荐）

`rustup` 是 Rust 的官方安装工具，可以管理多个 Rust 版本和工具链。

#### Linux/macOS

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### Windows

下载并运行 [rustup-init.exe](https://win.rustup.rs/)。

### 验证安装

安装完成后，打开终端运行以下命令：

```bash
rustc --version
cargo --version
```

如果显示版本信息，说明安装成功。

### 更新 Rust

```bash
rustup update
```

### 卸载 Rust

```bash
rustup self uninstall
```

## 0x03 开发环境配置

### 编辑器配置

#### VS Code（推荐）

1. 安装 [Visual Studio Code](https://code.visualstudio.com/)
2. 安装 `rust-analyzer` 扩展
3. 安装 `Even Better TOML` 扩展（用于 Cargo.toml 语法高亮）

#### JetBrains IDE

1. 安装 IntelliJ IDEA 或 CLion
2. 安装 Rust 插件

#### Vim/Neovim

1. 安装 `rust.vim` 插件
2. 配置 `rust-analyzer` 语言服务器

### Cargo 配置

Cargo 是 Rust 的包管理器和构建工具。配置文件位于 `~/.cargo/config.toml`。

#### 国内镜像配置（可选）

```toml
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```

## 0x04 第一个 Rust 程序

### 创建新项目

```bash
cargo new hello_rust
cd hello_rust
```

### 项目结构

```
hello_rust/
├── Cargo.toml    # 项目配置文件
└── src/
    └── main.rs   # 源代码文件
```

### 代码示例

编辑 `src/main.rs`：

```rust
fn main() {
    println!("Hello, world!");
    
    // 变量和基本类型
    let x = 5;
    let y: i32 = 10;
    let sum = x + y;
    
    println!("x = {}, y = {}, sum = {}", x, y, sum);
    
    // 字符串
    let greeting = String::from("Hello");
    let name = "Rust";
    let message = format!("{}, {}!", greeting, name);
    
    println!("{}", message);
    
    // 条件语句
    let number = 7;
    if number > 0 {
        println!("{} is positive", number);
    } else if number < 0 {
        println!("{} is negative", number);
    } else {
        println!("{} is zero", number);
    }
    
    // 循环
    for i in 0..5 {
        println!("Iteration: {}", i);
    }
    
    // 函数调用
    let result = add(3, 4);
    println!("3 + 4 = {}", result);
}

fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

### 编译和运行

```bash
# 编译并运行
cargo run

# 只编译
cargo build

# 发布版本编译
cargo build --release
```

## 0x05 Cargo 常用命令

### 项目管理

```bash
# 创建新项目（二进制）
cargo new project_name

# 创建新项目（库）
cargo new --lib library_name

# 初始化当前目录为 Cargo 项目
cargo init
```

### 构建和运行

```bash
# 构建项目
cargo build

# 运行项目
cargo run

# 检查代码（不生成可执行文件）
cargo check

# 构建发布版本
cargo build --release
```

### 测试和文档

```bash
# 运行测试
cargo test

# 生成文档
cargo doc

# 生成并打开文档
cargo doc --open
```

### 依赖管理

```bash
# 添加依赖（需要 cargo-edit 扩展）
cargo add serde

# 更新依赖
cargo update

# 清理构建缓存
cargo clean
```

## 0x06 学习资源

### 官方资源

- [The Rust Programming Language](https://doc.rust-lang.org/book/) - 官方教程
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/) - 通过示例学习
- [Rust Reference](https://doc.rust-lang.org/reference/) - 语言参考
- [Standard Library API](https://doc.rust-lang.org/std/) - 标准库文档

### 社区资源

- [Rust 中文社区](https://rustcc.cn/)
- [Rust 知识库](https://rust-lang.github.io/async-book/)
- [Rust 异步编程](https://rust-lang.github.io/async-book/)

### 实践平台

- [Rust Playground](https://play.rust-lang.org/) - 在线编译器
- [Exercism Rust Track](https://exercism.org/tracks/rust) - 编程练习
- [Rustlings](https://github.com/rust-lang/rustlings) - 小练习

## 参考

- [Rust 官方网站](https://www.rust-lang.org/)
- [Rust 安装指南](https://www.rust-lang.org/tools/install)
- [Cargo 手册](https://doc.rust-lang.org/cargo/)
- [rust-analyzer 手册](https://rust-analyzer.github.io/)