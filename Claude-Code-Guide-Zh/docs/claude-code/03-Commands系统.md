# Claude Code Commands 系统指南

> 本章时长：4-6 小时 | 难度：⭐⭐ | 必学度：⭐⭐

---

## 📑 本章目录

1. [什么是 Commands](#1-什么是-commands)
2. [内置命令](#2-内置命令)
3. [自定义 Slash 命令](#3-自定义-slash-命令)
4. [命令参数传递](#4-命令参数传递)
5. [工作流组合](#5-工作流组合)
6. [命令管理](#6-命令管理)
7. [实战案例](#7-实战案例)
8. [常见问题](#8-常见问题)

---

## 1. 什么是 Commands

Commands（命令）是 Claude Code 中的快捷方式，让你用 `/命令` 的形式快速触发特定操作。

### 1.1 命令类型

| 类型 | 说明 | 示例 |
|-----|------|------|
| **Slash 命令** | 以 `/` 开头的快捷操作 | `/review`、`/test` |
| **内置命令** | Claude Code 自带的核心命令 | `/quit`、`/approve` |
| **自定义命令** | 用户自己定义的命令 | `/my-command` |

### 1.2 命令 vs 提示词

```
命令：/review          → 快速触发代码审查
提示词：审查这个代码   → 自然语言表达
```

命令是**结构化**的，提示词是**自然语言**的。命令更精确，提示词更灵活。

---

## 2. 内置命令

Claude Code 自带一批内置命令，涵盖常用操作。

### 2.1 会话控制

| 命令 | 简写 | 说明 |
|-----|-----|------|
| `/quit` | `/q` | 退出当前会话 |
| `/resume` | `/r` | 恢复被中断的任务 |
| `/clear` | | 清空对话历史 |

### 2.2 操作控制

| 命令 | 简写 | 说明 |
|-----|-----|------|
| `/approve` | `/a` | 批准待执行的操作 |
| `/reject` | `/x` | 拒绝待执行的操作 |
| `/retry` | | 重试当前操作 |

### 2.3 信息查询

| 命令 | 简写 | 说明 |
|-----|-----|------|
| `/cost` | `/c` | 查看 Token 消耗 |
| `/help` | `/h` | 显示帮助 |
| `/status` | `/s` | 查看当前状态 |

### 2.4 任务管理

| 命令 | 简写 | 说明 |
|-----|-----|------|
| `/tasks` | | 列出所有任务 |
| `/task <name>` | | 执行指定任务 |
| `/task new <name>` | | 创建新任务 |

---

## 3. 自定义 Slash 命令

你可以创建自己的命令，自动化常用操作。

### 3.1 命令文件位置

自定义命令保存在：

```bash
~/.claude/commands/
```

### 3.2 创建自定义命令

**命令结构：**
```
/command-name.md
```

**文件内容：**
```markdown
# 命令描述

执行特定操作的提示词模板

## 参数
- $ARG1: 第一个参数
- $ARG2: 第二个参数

## 示例
/test "my-feature"
```

### 3.3 示例：代码审查命令

创建文件 `~/.claude/commands/review.md`：

```markdown
# 代码审查

对指定代码进行全面的代码审查

## 提示词

请对以下代码进行全面的代码审查，包括：
1. 代码质量评分（1-10）
2. 潜在问题
3. 优化建议
4. 安全风险

需要审查的代码：
$CODE

## 参数
- CODE: 要审查的代码或文件路径
```

**使用方式：**
```
/review src/auth.py
```

### 3.4 示例：单元测试命令

创建文件 `~/.claude/commands/test.md`：

```markdown
# 生成单元测试

为指定的代码文件生成单元测试

## 提示词

请为以下代码生成 Python unittest 单元测试：
1. 覆盖所有公开函数
2. 包含边界情况测试
3. 使用 mock 模拟依赖

代码：
$FILEPATH
```

### 3.5 示例：代码转换命令

创建文件 `~/.claude/commands/convert.md`：

```markdown
# 代码转换

将代码从一种语言/风格转换为另一种

## 提示词

将以下代码从 $FROM 转换为 $TO：

$CODE

要求：
- 保持功能一致
- 遵循目标语言的最佳实践
- 添加必要的注释
```

**使用：**
```
/convert "Python" "JavaScript" "def add(a,b): return a+b"
```

---

## 4. 命令参数传递

### 4.1 位置参数

```markdown
# 格式
$ARG1 $ARG2 $ARG3

# 使用
/命令 参数1 参数2 参数3
```

### 4.2 命名参数

```markdown
# 格式
--input <value>
--output <value>

# 使用
/命令 --input file.py --output result.py
```

### 4.3 混合参数

```markdown
# 格式
$FILE --option <value>

# 使用
/命令 auth.py --verbose
```

### 4.4 默认值

```markdown
# 设置默认值
$FILE:src/app.py

# 使用时不带参数则使用默认值
/命令           # 使用 src/app.py
/命令 other.py  # 使用 other.py
```

---

## 5. 工作流组合

### 5.1 什么是工作流

工作流 = 多个命令/操作按顺序执行，完成一个完整任务。

### 5.2 命令组合

在提示词中组合多个命令：

```markdown
# 完整开发流程

## 提示词

请执行以下步骤：

1. 审查代码：/review $FILE
2. 生成测试：/test $FILE  
3. 运行测试：运行 pytest
4. 生成文档：/doc $FILE

## 参数
- FILE: 要处理的文件路径
```

### 5.3 条件执行

```markdown
# 条件分支

## 提示词

$ACTION 的值决定执行什么：

- "review": 执行代码审查
- "test": 生成并运行测试
- "doc": 生成文档
- "all": 执行全部操作
```

### 5.4 循环执行

```markdown

# 批量处理

## 提示词

遍历以下文件列表，为每个文件执行处理：

$FILES

每个文件执行：
1. 代码审查
2. 格式化
3. 运行测试
```

---

## 6. 命令管理

### 6.1 查看所有命令

```bash
# 列出自定义命令
claude commands list

# 查看命令详情
claude commands show review
```

### 6.2 编辑命令

```bash
# 编辑命令
claude commands edit review

# 会在默认编辑器中打开
```

### 6.3 删除命令

```bash
# 删除命令
claude commands delete review
```

### 6.4 导入/导出命令

```bash
# 导出自定义命令
claude commands export > commands.json

# 导入命令
claude commands import commands.json
```

### 6.5 命令目录结构

```
~/.claude/
├── commands/
│   ├── review.md      # 代码审查命令
│   ├── test.md        # 测试命令
│   ├── doc.md         # 文档命令
│   └── ...
└── settings.json
```

---

## 7. 实战案例

### 7.1 案例：PR 审查流程

创建 `~/.claude/commands/pr-review.md`：

```markdown
# PR 审查

自动化 Pull Request 审查流程

## 提示词

请执行以下 PR 审查流程：

1. **获取变更**：查看 $BRANCH 与 main 的差异
2. **代码审查**：
   - 检查代码风格
   - 识别潜在 bug
   - 评估安全性
3. **测试覆盖**：确认是否有对应测试
4. **文档检查**：确认是否更新了文档
5. **总结**：给出 approve / request changes / comment 的建议

## 参数
- BRANCH: PR 的分支名（默认：当前分支）
```

### 7.2 案例：重构任务

创建 `~/.claude/commands/refactor.md`：

```markdown
# 代码重构

按照重构目标重构代码

## 提示词

请对 `$FILEPATH` 进行以下重构：

1. **提取函数**：将长函数拆分为小函数
2. **移除重复**：识别并消除重复代码
3. **命名优化**：确保变量/函数命名清晰
4. **添加注释**：为复杂逻辑添加说明

重构后确保：
- 所有现有测试通过
- 不引入新的警告

## 参数
- FILEPATH: 要重构的文件路径
- MODE: 重构模式（默认：safe，可选 aggressive）
```

### 7.3 案例：快速启动项目

创建 `~/.claude/commands/scaffold.md`：

```markdown
# 项目脚手架

快速搭建项目结构

## 提示词

请根据以下规格创建项目结构：

- 框架：$FRAMEWORK
- 语言：$LANGUAGE
- 项目名：$NAME

创建：
1. 基础目录结构
2. 配置文件
3. 入口文件
4. 基础模块
5. README

## 参数
- FRAMEWORK: 框架名称（react/vue/express/fastapi...）
- LANGUAGE: 编程语言（typescript/python/go...）
- NAME: 项目名称
```

---

## 8. 常见问题

### Q1: 自定义命令不生效？

**检查：**
1. 文件是否在 `~/.claude/commands/` 目录
2. 文件名是否以 `.md` 结尾
3. 格式是否正确

### Q2: 怎么查看命令帮助？

```bash
# 查看所有命令
claude --help

# 查看特定命令
claude commands show <command-name>
```

### Q3: 命令可以嵌套吗？

可以，在提示词中使用其他命令即可。

### Q4: 命令有数量限制吗？

没有明确限制，但建议控制在合理范围内（10-20个）。

### Q5: 可以分享命令给其他人吗？

可以，导出为 JSON 文件后分享：

```bash
claude commands export > my-commands.json
```

---

## ✅ 本章小结

本章学习了：

- [x] Commands 是什么及三种类型
- [x] 内置命令（会话控制、操作控制、信息查询）
- [x] 创建自定义 Slash 命令
- [x] 命令参数传递（位置、命名、默认值）
- [x] 工作流组合（顺序、条件、循环）
- [x] 命令管理（查看、编辑、删除、导入导出）
- [x] 实战案例

---

## 🔗 下章预告

**[第四章：MCP 集成](./04-MCP集成.md)**

我们将学习：
- 什么是 MCP
- MCP 服务器安装与配置
- 常用 MCP 服务推荐
- 自定义 MCP 开发

---

*⭐ 如果这个教程对你有帮助，欢迎 Star 支持！*
