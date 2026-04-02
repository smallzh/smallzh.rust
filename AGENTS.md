# AGENTS.md - smallzh.rust 项目指南

## 项目概述

这是一个记录 rust 关键知识的文档项目，使用 MkDocs 构建。项目目标是记录rust的理解和学习笔记，涵盖从基础语法到常用库的各个方面。



## 技术栈

| 组件 | 版本/说明 |
|------|----------|
| Python | >= 3.13 |
| MkDocs | >= 1.6.1 |
| 主题 | mkdocs-smzhbook-theme >= 1.1 |
| 包管理器 | uv |

## 项目结构

```
smallzh.rust/
├── docs/                    # 文档源文件目录
│   ├── _assets/            # 静态资源（图片等）
│   ├── index.md            # 首页
├── mkdocs.yml              # MkDocs 配置文件
├── pyproject.toml          # 项目配置
└── uv.lock                 # 依赖锁定文件
```

## 常用命令

```shell
# 安装依赖
uv sync

# 启动本地开发服务器（实时预览）
uv run mkdocs serve

# 构建静态站点
uv run mkdocs build

# 部署到 GitHub Pages（如配置）
uv run mkdocs gh-deploy
```

## 内容写作风格

### 文档结构约定

1. **标题**: 使用 `#` 作为一级标题，标题采用疑问句或陈述句形式
   - 示例: `# 支持哪些数据类型？`、`# 字符串`

2. **代码示例**: 使用代码块展示示例，标注语言类型
   ```rust
   # 示例代码
   ```

3. **引用**: 使用 `>` 标注引用内容，常用于引用官方文档或外部资料

4. **图片**: 存放在 `docs/_assets/` 目录下，使用相对路径引用
   ```markdown
   ![描述](./_assets/img/filename.jpg)
   ```
5. **二级标题**： 使用`## 0x01` 这种格式
	- 示例: `## 0x02 代码的详细内容`

### 内容组织原则

- **简洁明了**: 直接说明知识点，避免冗长解释
- **实用导向**: 侧重于实际使用场景和常用操作
- **示例优先**: 用代码示例说明概念
- **参考来源**: 在文末列出参考链接

## 添加新文档

1. 在 `docs/` 目录下创建新的 `.md` 文件
2. 在 `mkdocs.yml` 的 `nav` 部分添加导航项：
   ```yaml
   nav:
     - 新主题: new_topic.md
   ```
3. 遵循现有文档的写作风格

## 配置说明

### mkdocs.yml

- `site_name`: 站点名称 - "smallzh's python"
- `theme.name`: 使用 "smzhbook" 主题
- `nav`: 文档导航结构
- `plugins`: 当前为空数组

### pyproject.toml

- 项目名称: smallzh-rust
- Python 版本要求: >= 3.13
- 依赖: mkdocs, mkdocs-smzhbook-theme

## 注意事项

1. **Python 版本**: 开发环境需要 3.13+
2. **包管理**: 使用 `uv` 而非 `pip` 进行依赖管理
3. **主题**: 使用自定义的 `smzhbook` 主题
4. **中文内容**: 文档使用中文编写
5. **文件命名**: 使用小写字母和下划线命名文档文件

## 扩展指南

### 添加新的 rust 库文档

1. 创建 `docs/new_library.md`
2. 按照以下结构组织内容：
   ```markdown
   # 库名称
   
   ## 简介
   ## 安装
   ## 基本用法
   ## 常用功能
   ## 示例
   ## 参考
   ```
3. 在 `mkdocs.yml` 中添加导航

### 修改主题或样式

主题配置在 `mkdocs.yml` 的 `theme` 部分。如需修改样式，可能需要：
- 查看 mkdocs-smzhbook-theme 文档
- 添加自定义 CSS（如主题支持）
