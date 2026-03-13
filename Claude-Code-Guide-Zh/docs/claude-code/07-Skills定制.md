# Claude Code Skills 定制指南

> 本章时长：6-8 小时 | 难度：⭐⭐ | 必学度：⭐⭐

---

## 📑 本章目录

1. [什么是 Skills](#1-什么是-skills)
2. [Skills vs Commands vs Hooks](#2-skills-vs-commands-vs-hooks)
3. [内置 Skills](#3-内置-skills)
4. [自定义 Skill 开发](#4-自定义-skill-开发)
5. [Skill 结构详解](#5-skill-结构详解)
6. [Skill 参数与变量](#6-skill-参数与变量)
7. [实战案例](#7-实战案例)
8. [常见问题](#8-常见问题)

---

## 1. 什么是 Skills

**Skills** 是 Claude Code 的可复用功能包，让你可以打包一组提示词、命令和配置，形成标准化的能力模块。

### 1.1 Skill 的定义

Skill = **功能包** = 提示词 + 配置 + 工具

```
Skill
├── SKILL.md          # 技能定义
├── prompts/          # 提示词模板
├── scripts/          # 辅助脚本
└── config.json       # 配置
```

### 1.2 Skill 的特性

- 📦 **可复用** - 一次定义，多次使用
- 🔧 **可配置** - 参数化设计，灵活定制
- 📤 **可分享** - 导出给其他人使用
- 🔄 **可组合** - 多个 Skill 组合使用

### 1.3 Skill 能做什么

```
✅ 标准化工作流程
✅ 团队知识共享
✅ 快速启动项目
✅ 复杂任务封装
✅ 自动化流水线
```

---

## 2. Skills vs Commands vs Hooks

三者都是扩展 Claude Code 的方式，但适用场景不同。

### 2.1 对比表

| 特性 | Skills | Commands | Hooks |
|-----|--------|----------|-------|
| **本质** | 功能包 | 快捷命令 | 自动化钩子 |
| **触发** | 手动调用 | `/命令` | 自动触发 |
| **复杂度** | 高 | 低 | 中 |
| **复用性** | 高 | 中 | 低 |
| **适用场景** | 复杂流程 | 简单操作 | 自动化 |

### 2.2 选择指南

| 场景 | 推荐 |
|-----|------|
| 快速执行某个操作 | Commands |
| 自动化工作流 | Hooks |
| 复杂、可配置的功能 | Skills |

### 2.3 组合使用

```
Skills + Commands + Hooks = 完整解决方案

Skill       → 定义完整功能
  ├── Command  → 快捷入口
  └── Hook    → 自动化
```

---

## 3. 内置 Skills

Claude Code 自带一批内置 Skills，开箱即用。

### 3.1 常用内置 Skills

| Skill | 功能 | 使用场景 |
|-------|------|---------|
| `skill-creator` | 创建新 Skill | 开发新技能 |
| `read` | 文件读取 | 代码审查 |
| `write` | 文件写入 | 代码生成 |
| `edit` | 文件编辑 | 代码修改 |
| `bash` | 命令执行 | 终端操作 |
| `grep` | 代码搜索 | 搜索代码 |

### 3.2 查看 Skills 列表

```bash
# 列出所有 Skills
npx claude skills list

# 查看特定 Skill
npx claude skills show skill-name
```

### 3.3 使用内置 Skill

```bash
# 直接使用
claude "用 read skill 读取文件"

# 指定 Skill
@skill:read file.py
```

---

## 4. 自定义 Skill 开发

### 4.1 Skill 文件结构

```
my-skill/
├── SKILL.md          # 必须：技能定义
├── prompts/          # 提示词模板
│   ├── default.md
│   └── custom.md
├── scripts/          # 辅助脚本
│   └── helper.sh
└── config.json       # 配置（可选）
```

### 4.2 SKILL.md 模板

```markdown
# SKILL.md - 技能名称

## 描述
这个技能用于...

## 功能
- 功能 1
- 功能 2

## 参数
| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| param1 | string | 是 | 参数1说明 |
| param2 | boolean | 否 | 参数2说明 |

## 使用方法

### 基本用法
\`\`\`bash
claude @skill:param1=value1
\`\`\`

### 高级用法
...


## 示例

### 示例 1
输入：
\`\`\`
...
\`\`\`

输出：
\`\`\`
...
\`\`\`

## 注意事项
- 注意点 1
- 注意点 2

## 更新日志
- 2026-01-01: 初始版本
```

### 4.3 最小 Skill 示例

**目录结构：**
```
hello-skill/
└── SKILL.md
```

**SKILL.md：**
```markdown
# hello-skill

## 描述
简单的打招呼技能

## 使用方法
直接调用即可：
@hello-skill
或
/hello
```

---

## 5. Skill 结构详解

### 5.1 必填字段

| 字段 | 说明 | 示例 |
|-----|------|------|
| `#` | 技能名称 | `# my-skill` |
| `## 描述` | 功能描述 | `## 描述 这是一个...` |
| `## 使用方法` | 使用说明 | `## 使用方法 ...` |

### 5.2 可选字段

| 字段 | 说明 |
|-----|------|
| `## 参数` | 参数定义 |
| `## 示例` | 使用示例 |
| `## 注意事项` | 注意事项 |
| `## 更新日志` | 版本记录 |

### 5.3 配置项

在 `config.json` 中配置：

```json
{
  "name": "my-skill",
  "version": "1.0.0",
  "description": "技能描述",
  "parameters": {
    "param1": {
      "type": "string",
      "required": true,
      "default": "default value"
    }
  },
  "env": {
    "API_KEY": "required"
  }
}
```

---

## 6. Skill 参数与变量

### 6.1 参数定义

```markdown
## 参数
| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| name | string | 是 | 用户名称 |
| age | number | 否 | 年龄，默认 18 |
| options | object | 否 | 选项配置 |
```

### 6.2 参数使用

```markdown
## 使用方法

你好，$NAME！

你的年龄是 $AGE 岁。
```

### 6.3 内置变量

| 变量 | 说明 |
|-----|------|
| `$CLAUDE_DIR` | Claude Code 目录 |
| `$WORKING_DIR` | 当前工作目录 |
| `$SESSION_ID` | 会话 ID |
| `$TIMESTAMP` | 当前时间戳 |

### 6.4 条件逻辑

```markdown
#if ($AGE >= 18)
你可以使用完整功能。
#else
你只能使用受限功能。
#end
```

### 6.5 循环

```markdown
#for ($item in $items)
- $item.name: $item.value
#end
```

---

## 7. 实战案例

### 7.1 案例：API 客户端 Skill

**目录结构：**
```
api-client/
├── SKILL.md
└── config.json
```

**SKILL.md：**
```markdown
# api-client

## 描述
通用的 API 客户端技能，支持 REST API 调用

## 参数
| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| method | string | 是 | HTTP 方法 |
| url | string | 是 | 请求 URL |
| headers | object | 否 | 请求头 |
| body | object | 否 | 请求体 |

## 使用方法

### GET 请求
@api-client method=get url=https://api.example.com/users

### POST 请求
@api-client method=post url=https://api.example.com/users body={"name":"test"}

## 示例

### 示例 1：获取用户列表
输入：
\`\`\`
method: GET
url: https://api.example.com/users
\`\`\`

输出：
\`\`\`
[
  {"id": 1, "name": "Alice"},
  {"id": 2, "name": "Bob"}
]
\`\`\`
```

### 7.2 案例：项目脚手架 Skill

**目录结构：**
```
project-scaffold/
├── SKILL.md
└── templates/
    ├── react/
    ├── vue/
    └── express/
```

**SKILL.md：**
```markdown
# project-scaffold

## 描述
快速生成项目脚手架

## 参数
| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| framework | string | 是 | 框架名称 |
| name | string | 是 | 项目名称 |
| typescript | boolean | 否 | 是否使用 TypeScript |

## 使用方法

### React 项目
@project-scaffold framework=react name=my-app

### Vue + TypeScript
@project-scaffold framework=vue name=my-app typescript=true

## 模板
- react: React + Vite
- vue: Vue 3 + Vite
- express: Express.js
- next: Next.js
```

### 7.3 案例：代码审查 Skill

**目录结构：**
```
code-review/
├── SKILL.md
└── prompts/
    ├── security.md
    ├── performance.md
    └── style.md
```

**SKILL.md：**
```markdown
# code-review

## 描述
全面的代码审查技能，支持多维度审查

## 参数
| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| target | string | 是 | 审查目标（文件/目录） |
| dimensions | string[] | 否 | 审查维度 |
| strictness | string | 否 | 严格程度（normal/strict） |

## 审查维度
- security: 安全审查
- performance: 性能审查
- style: 代码风格
- test: 测试覆盖
- docs: 文档完整性

## 使用方法

### 全面审查
@code-review target=src/

### 指定维度
@code-review target=src/ dimensions=security,performance

### 严格模式
@code-review target=src/ strictness=strict
```

### 7.4 案例：数据处理 Skill

**SKILL.md：**
```markdown
# data-processor

## 描述
通用数据处理技能，支持 JSON/CSV/Excel

## 参数
| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| input | string | 是 | 输入文件路径 |
| output | string | 是 | 输出文件路径 |
| operation | string | 是 | 操作类型 |
| options | object | 否 | 操作选项 |

## 操作类型
- transform: 数据转换
- filter: 数据过滤
- aggregate: 数据聚合
- validate: 数据验证
- export: 格式转换

## 使用示例

### JSON 转 CSV
@data-processor input=data.json output=data.csv operation=export format=csv

### 数据过滤
@data-processor input=data.json output=filtered.json operation=filter condition="age > 18"

### 数据聚合
@data-processor input=sales.json output=summary.json operation=aggregate groupBy=category
```

---

## 8. 常见问题

### Q1: Skill 和 Subagent 有什么区别？

| 特性 | Skill | Subagent |
|-----|-------|----------|
| 本质 | 功能包 | 代理 |
| 复杂度 | 中 | 高 |
| 适用 | 标准化流程 | 对话式任务 |
| 触发 | @skill | @agent |

### Q2: Skill 放在哪里？

用户自定义 Skills 放在：
```
~/.claude/skills/
```

### Q3: 可以导入别人的 Skill 吗？

可以，复制到 `~/.claude/skills/` 目录即可。

### Q4: Skill 支持版本管理吗？

支持，在 `config.json` 中指定版本号：

```json
{
  "version": "1.0.0"
}
```

### Q5: Skill 有数量限制吗？

没有硬性限制。

### Q6: 怎么分享 Skill？

```bash
# 导出 Skill
claude skills export my-skill > my-skill.zip

# 导入 Skill
claude skills import my-skill.zip
```

---

## ✅ 本章小结

本章学习了：

- [x] Skills 是什么及核心概念
- [x] Skills vs Commands vs Hooks 选择指南
- [x] 内置 Skills
- [x] 自定义 Skill 开发
- [x] Skill 结构详解（必填/可选字段）
- [x] 参数与变量、条件逻辑、循环
- [x] 实战案例（API 客户端、脚手架、审查、数据处理）

---

## 🔗 下章预告

**[第八章：Plugins 生态](./08-Plugins生态.md)**

我们将学习：
- 什么是 Plugins
- Marketplace 插件推荐
- 插件安装与配置
- 自定义插件开发

---

*⭐ 如果这个教程对你有帮助，欢迎 Star 支持！*
