# Claude Code Subagent 子代理指南

> 本章时长：1-2 小时 | 难度：⭐⭐ | 必学度：⭐⭐

---

## 📑 本章目录

1. [什么是 Subagent](#1-什么是-subagent)
2. [内置子代理](#2-内置子代理)
3. [子代理配置](#3-子代理配置)
4. [多代理协作模式](#4-多代理协作模式)
5. [Task 工具与编排](#5-task-工具与编排)
6. [自定义子代理](#6-自定义子代理)
7. [实战案例](#7-实战案例)
8. [常见问题](#8-常见问题)

---

## 1. 什么是 Subagent

**Subagent（子代理）** 是 Claude Code 内置的多代理系统，让一个主 Claude 指挥多个专业子代理同时工作。

### 1.1 子代理的定义

```
主 Claude ──┬── 子代理 A（代码审查）
            ├── 子代理 B（测试生成）
            ├── 子代理 C（文档编写）
            └── 子代理 D（安全扫描）
```

### 1.2 Subagent vs 单代理

| 特性 | 单代理 | 多代理（Subagent） |
|-----|-------|-----------------|
| 任务处理 | 串行 | 并行 |
| 专业性 | 全能 | 专业分工 |
| 上下文 | 单一 | 隔离 |
| 协作 | 无 | 有 |

### 1.3 子代理的优势

- ⚡ **并行处理** - 多个任务同时执行
- 🎯 **专业分工** - 每个子代理专注特定领域
- 🔒 **上下文隔离** - 子代理间互不干扰
- 📦 **可复用** - 一次配置，多次使用

---

## 2. 内置子代理

Claude Code 自带 100+ 专家子代理，开箱即用。

### 2.1 子代理分类

| 类别 | 数量 | 示例 |
|-----|-----|------|
| **代码审查** | 15+ | review, security, performance |
| **测试** | 10+ | unit-test, integration-test |
| **文档** | 10+ | readme, api-doc, comments |
| **重构** | 10+ | migrate, optimize, simplify |
| **调试** | 10+ | debug, fix, reproduce |
| **其他** | 50+ | summarize, explain, translate |

### 2.2 常用子代理

| 子代理 | 功能 | 使用场景 |
|-------|------|---------|
| `review` | 代码审查 | 审查 PR/代码变更 |
| `security` | 安全扫描 | 检测漏洞 |
| `unit-test` | 单元测试 | 生成测试用例 |
| `readme` | 文档生成 | 生成 README |
| `fix` | Bug 修复 | 修复已知问题 |
| `migrate` | 代码迁移 | 升级框架/语言 |
| `summarize` | 总结摘要 | 总结代码/文档 |

### 2.3 子代理列表查询

```bash
# 列出所有子代理
claude subagents list

# 查看特定子代理
claude subagents show review
```

---

## 3. 子代理配置

### 3.1 基本使用

```bash
# 使用子代理
claude @review

# 指定文件
claude @review src/auth.py

# 指定问题
claude @security "检查登录漏洞"
```

### 3.2 配置子代理

在 `~/.claude/settings.json` 中配置：

```json
{
  "subagents": {
    "custom-reviewer": {
      "description": "自定义代码审查",
      "systemPrompt": "你是一个严格的代码审查员...",
      "enabled": true
    }
  }
}
```

### 3.3 子代理参数

| 参数 | 说明 | 类型 |
|-----|------|------|
| `name` | 子代理名称 | string |
| `description` | 功能描述 | string |
| `systemPrompt` | 系统提示词 | string |
| `enabled` | 是否启用 | boolean |
| `tools` | 允许的工具 | string[] |

---

## 4. 多代理协作模式

### 4.1 串行模式

任务按顺序执行，每个完成后执行下一个：

```
A → B → C → D
```

**适用场景：**
- 有依赖关系的任务
- 每步需要上一步结果

**示例：**
```
@fix bug1 → @test → @review
```

### 4.2 并行模式

多个任务同时执行：

```
A
B
C  → 结果汇总
D
```

**适用场景：**
- 独立任务
- 需要加速处理

**示例：**
```
@review + @test + @security 同时运行
```

### 4.3 分支模式

根据条件选择不同路径：

```
      ┌─ A
主 ───┼─ B
      └─ C
```

**适用场景：**
- 条件分支
- 错误处理

### 4.4 层级模式

多级代理，层层递进：

```
主 → 组长 → 组员
```

**适用场景：**
- 复杂任务分解
- 评审机制

---

## 5. Task 工具与编排

> v2.1.49+ 新功能

### 5.1 什么是 Task 工具

Task 是 Claude Code 内置的任务编排工具，简化子代理调用：

```bash
# 基础用法
/Task "代码审查" @review

# 带参数
/Task "修复 bug" @fix issue=123
```

### 5.2 DAG 依赖

支持任务依赖图（Directed Acyclic Graph）：

```javascript
{
  "tasks": [
    {
      "id": "review",
      "subagent": "review",
      "runAfter": []
    },
    {
      "id": "test", 
      "subagent": "unit-test",
      "runAfter": ["review"]  // 等待 review 完成
    },
    {
      "id": "merge",
      "command": "git merge",
      "runAfter": ["test"]    // 等待 test 完成
    }
  ]
}
```

### 5.3 上下文注入

子代理可以继承主代理的上下文：

```bash
# 注入当前项目上下文
@review --context current

# 注入特定文件
@review src/main.py
```

### 5.4 任务状态查询

```bash
# 查看任务状态
claude tasks status

# 查看任务结果
claude tasks result task-id
```

---

## 6. 自定义子代理

### 6.1 创建子代理

在 `~/.claude/subagents/` 目录创建配置文件：

**文件：** `~/.claude/subagents/my-agent.md`

```markdown
# name: my-agent
# description: 我的自定义子代理

你是一个专业的 [角色名称]，擅长 [领域]。

## 能力
- 能力 1
- 能力 2

## 输出格式
请按以下格式输出：
1. ...
2. ...
```

### 6.2 子代理模板

```markdown
# name: code-auditor
# description: 代码审计员

你是一个专业的代码审计员，擅长发现代码中的问题。

## 审计维度
1. 安全性 - 是否有安全漏洞
2. 性能 - 是否有性能问题
3. 可维护性 - 是否易于维护
4. 测试覆盖 - 是否有充分测试

## 输出格式
请按以下格式输出审计报告：

### 发现的问题
| 严重程度 | 位置 | 问题描述 | 建议 |
|---------|------|---------|------|
| 高 | xxx | xxx | xxx |

### 总体评分
/10
```

### 6.3 使用自定义子代理

```bash
# 调用自定义子代理
claude @code-auditor src/
```

---

## 7. 实战案例

### 7.1 案例：完整代码审查流程

```bash
# 1. 并行执行多个审查维度
@review src/           # 代码审查
@security src/         # 安全审查
@performance src/      # 性能审查

# 2. 汇总结果
/summarize 审查结果

# 3. 生成报告
@readme --type review
```

### 7.2 案例：Bug 修复工作流

```bash
# 1. 理解问题
@explain bug.md

# 2. 定位问题
@debug "复现步骤..."

# 3. 修复问题
@fix "问题描述"

# 4. 验证修复
@unit-test "修复的代码"

# 5. 代码审查
@review "修复代码"
```

### 7.3 案例：多语言迁移

```bash
# 1. 分析原代码
@analyze src/

# 2. 迁移代码
@migrate "Python" "Go"

# 3. 生成测试
@unit-test "Go 代码"

# 4. 验证功能
@integration-test

# 5. 更新文档
@readme --type migration
```

### 7.4 案例：自动化测试覆盖

```bash
# 为整个项目生成测试
@unit-test --all

# 运行测试
claude "npm test"

# 查看覆盖率
@coverage-report
```

---

## 8. 常见问题

### Q1: 子代理和 Task 有什么区别？

| 特性 | 子代理 | Task |
|-----|-------|------|
| 定义 | 长期配置 | 临时任务 |
| 持久性 | 保存配置 | 一次性 |
| 灵活性 | 较低 | 高 |
| 适用场景 | 常用角色 | 临时需求 |

### Q2: 子代理数量有限制吗？

没有硬性限制，但建议：
- 常用子代理：5-10 个
- 临时子代理：按需创建

### Q3: 子代理之间能共享上下文吗？

可以，通过主代理中转：

```
子代理 A → 主代理 → 子代理 B
```

### Q4: 子代理可以调用子代理吗？

可以，但建议不超过 2 层：

```
主 → 子代理 A → 子代理 A.1（最多）
```

### Q5: Task 的 DAG 是什么？

DAG = 有向无环图，描述任务依赖关系：

```
A → B → C
↑   ↘
D ──┘  (不允许循环)
```

### Q6: 如何选择串行还是并行？

| 场景 | 模式 |
|-----|------|
| 有依赖 | 串行 |
| 独立任务 | 并行 |
| 需要汇总 | 分支+汇总 |
| 复杂任务 | 层级 |

---

## ✅ 本章小结

本章学习了：

- [x] Subagent 是什么及核心概念
- [x] 100+ 内置子代理（审查、测试、文档等）
- [x] 子代理配置与使用
- [x] 多代理协作模式（串行、并行、分支、层级）
- [x] Task 工具与 DAG 编排（v2.1.49+）
- [x] 自定义子代理开发
- [x] 实战案例

---

## 🔗 下章预告

**[第七章：Skills 定制](./07-Skills定制.md)**

我们将学习：
- 什么是 Skills
- 内置 Skills
- 自定义 Skill 开发
- Skills vs Commands vs Hooks

---

*⭐ 如果这个教程对你有帮助，欢迎 Star 支持！*
