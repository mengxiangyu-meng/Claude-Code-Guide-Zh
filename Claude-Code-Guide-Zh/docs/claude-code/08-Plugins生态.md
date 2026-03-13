# Claude Code Plugins 生态指南

> 本章时长：4-6 小时 | 难度：⭐⭐ | 必学度：⭐

---

## 📑 本章目录

1. [什么是 Plugins](#1-什么是-plugins)
2. [Plugins vs Skills vs MCP](#2-plugins-vs-skills-vs-mcp)
3. [Marketplace 插件推荐](#3-marketplace-插件推荐)
4. [插件安装与配置](#4-插件安装与配置)
5. [插件开发基础](#5-插件开发基础)
6. [实战案例](#6-实战案例)
7. [常见问题](#7-常见问题)

---

## 1. 什么是 Plugins

**Plugins（插件）** 是 Claude Code 的扩展生态，让你可以安装第三方功能增强 Claude Code。

### 1.1 Plugin 的定义

Plugin = **第三方扩展** = npm 包 + 配置文件

```
Claude Code + Plugins = 增强版 Claude
```

### 1.2 Plugin 的特性

- 🔌 **即插即用** - 一键安装，快速启用
- 🌐 **生态丰富** - 社区贡献的各类插件
- 🔒 **安全可控** - 权限可控，可随时禁用
- ♻️ **可更新** - 版本管理，及时更新

### 1.3 Plugin 能做什么

```
✅ 代码质量检测
✅ CI/CD 集成
✅ 云服务接入
✅ 团队协作工具
✅ 高级搜索
✅ 可视化工具
```

---

## 2. Plugins vs Skills vs MCP

三者都是扩展方式，但定位不同。

### 2.1 对比表

| 特性 | Plugins | Skills | MCP |
|-----|---------|--------|-----|
| **本质** | 第三方 npm 包 | 自定义功能包 | 外部服务协议 |
| **来源** | 社区/官方 | 用户自定义 | 第三方服务 |
| **开发方式** | TypeScript | Markdown | 多种语言 |
| **复杂度** | 中高 | 低中 | 中 |
| **适用场景** | 通用功能 | 定制流程 | API 集成 |

### 2.2 选择指南

| 需求 | 推荐 |
|-----|------|
| 接入第三方服务/工具 | MCP |
| 标准化工作流程 | Skills |
| 增强基础功能 | Plugins |

### 2.3 组合使用

```
Plugins + Skills + MCP = 完整生态

Plugin  → 基础能力增强
Skill   → 定制工作流
MCP     → 外部服务集成
```

---

## 3. Marketplace 插件推荐

### 3.1 代码质量类

| 插件 | 功能 | 安装命令 |
|-----|------|---------|
| **ESLint** | 代码规范检查 | `npm install @claude/plugin-eslint` |
| **Prettier** | 代码格式化 | `npm install @claude/plugin-prettier` |
| **SonarQube** | 代码质量分析 | `npm install @claude/plugin-sonar` |
| **TypeScript** | TS 类型检查 | 内置 |

### 3.2 CI/CD 类

| 插件 | 功能 | 安装命令 |
|-----|------|---------|
| **GitHub Actions** | CI/CD 集成 | `npm install @claude/plugin-github-actions` |
| **GitLab CI** | CI/CD 集成 | `npm install @claude/plugin-gitlab-ci` |
| **Jenkins** | CI/CD 集成 | `npm install @claude/plugin-jenkins` |
| **Docker** | 容器支持 | `npm install @claude/plugin-docker` |

### 3.3 云服务类

| 插件 | 功能 | 安装命令 |
|-----|------|---------|
| **AWS** | AWS 服务 | `npm install @claude/plugin-aws` |
| **Vercel** | 部署平台 | `npm install @claude/plugin-vercel` |
| **Netlify** | 部署平台 | `npm install @claude/plugin-netlify` |
| **Firebase** | Google 云服务 | `npm install @claude/plugin-firebase` |

### 3.4 团队协作类

| 插件 | 功能 | 安装命令 |
|-----|------|---------|
| **Slack** | 团队沟通 | `npm install @claude/plugin-slack` |
| **Discord** | 社区沟通 | `npm install @claude/plugin-discord` |
| **Notion** | 知识库 | `npm install @claude/plugin-notion` |
| **Linear** | 项目管理 | `npm install @claude/plugin-linear` |

### 3.5 开发工具类

| 插件 | 功能 | 安装命令 |
|-----|------|---------|
| **Database** | 数据库客户端 | `npm install @claude/plugin-database` |
| **API Client** | API 测试 | `npm install @claude/plugin-api-client` |
| **GraphQL** | GraphQL 支持 | `npm install @claude/plugin-graphql` |
| **Terraform** | 基础设施 | `npm install @claude/plugin-terraform` |

---

## 4. 插件安装与配置

### 4.1 安装插件

```bash
# 方式一：npm 安装
npm install @claude/plugin-name

# 方式二：Claude Code 命令安装
claude plugins install @claude/plugin-name

# 方式三：手动配置
claude config add plugin @claude/plugin-name
```

### 4.2 配置插件

在 `~/.claude/settings.json` 中配置：

```json
{
  "plugins": {
    "eslint": {
      "enabled": true,
      "options": {
        "fix": true,
        "extensions": [".js", ".ts"]
      }
    },
    "prettier": {
      "enabled": true,
      "options": {
        "semi": true,
        "singleQuote": true
      }
    }
  }
}
```

### 4.3 启用/禁用插件

```bash
# 禁用插件
claude plugins disable eslint

# 启用插件
claude plugins enable eslint

# 查看已安装插件
claude plugins list
```

### 4.4 更新插件

```bash
# 更新所有插件
claude plugins update

# 更新指定插件
claude plugins update @claude/plugin-name
```

---

## 5. 插件开发基础

### 5.1 插件结构

```typescript
// src/index.ts
import { ClaudePlugin } from '@claude/sdk';

export default {
  name: 'my-plugin',
  version: '1.0.0',

  // 插件初始化
  init(context) {
    // 初始化逻辑
  },

  // 注册工具
  tools() {
    return [
      {
        name: 'my_tool',
        description: '我的自定义工具',
        inputSchema: {
          type: 'object',
          properties: {
            param: { type: 'string' }
          }
        }
      }
    ];
  },

  // 注册 Hook
  hooks() {
    return {
      onStart: () => { /* ... */ },
      onToolUse: (tool) => { /* ... */ }
    };
  }
} as ClaudePlugin;
```

### 5.2 插件配置

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "main": "dist/index.js",
  "claude": {
    "plugins": ["my-plugin"]
  }
}
```

### 5.3 打包发布

```bash
# 打包
npm run build

# 发布到 npm
npm publish

# 本地测试
npm link
claude plugins enable my-plugin
```

---

## 6. 实战案例

### 6.1 案例：ESLint + Prettier 工作流

**安装：**
```bash
npm install @claude/plugin-eslint @claude/plugin-prettier
```

**配置：**
```json
{
  "plugins": {
    "eslint": {
      "enabled": true,
      "options": {
        "fix": true,
        "extensions": [".js", ".ts", ".jsx", ".tsx"]
      }
    },
    "prettier": {
      "enabled": true,
      "options": {
        "semi": true,
        "singleQuote": true,
        "trailingComma": "es5"
      }
    }
  }
}
```

**使用：**
```bash
# 写代码后自动格式化
# 自动修复 ESLint 错误
```

### 6.2 案例：GitHub Actions CI

**安装：**
```bash
npm install @claude/plugin-github-actions
```

**配置：**
```json
{
  "plugins": {
    "github-actions": {
      "enabled": true,
      "workflows": [".github/workflows/"],
      "autoTrigger": ["push", "pull_request"]
    }
  }
}
```

**功能：**
- 自动创建 PR
- 查看 CI 状态
- 触发工作流

### 6.3 案例：AWS 部署

**安装：**
```bash
npm install @claude/plugin-aws
```

**配置：**
```json
{
  "plugins": {
    "aws": {
      "enabled": true,
      "region": "us-east-1",
      "services": ["s3", "lambda", "ec2"]
    }
  }
}
```

**使用：**
```bash
# 部署到 S3
@plugin:aws deploy s3 bucket=my-bucket

# 调用 Lambda
@plugin:aws invoke function=my-function
```

---

## 7. 常见问题

### Q1: 插件安全吗？

大部分插件是安全的，但建议：
- 查看插件源码
- 检查插件权限
- 关注插件更新

### Q2: 插件冲突怎么办？

```bash
# 禁用可能有冲突的插件
claude plugins disable plugin-a
```

### Q3: 插件不兼容？

检查 Claude Code 版本：
```bash
claude --version
```

### Q4: 如何开发自己的插件？

参考上面的"插件开发基础"部分。

### Q5: 插件在哪里找？

- npm 搜索 `@claude/plugin-`
- GitHub 搜索 `claude-plugin`
- 官方文档

### Q6: 插件影响性能吗？

会有轻微影响，建议：
- 只安装需要的插件
- 禁用不常用的插件

---

## ✅ 本章小结

本章学习了：

- [x] Plugins 是什么及核心概念
- [x] Plugins vs Skills vs MCP 选择指南
- [x] Marketplace 插件推荐（代码质量、CI/CD、云服务、团队协作）
- [x] 插件安装与配置
- [x] 插件开发基础
- [x] 实战案例

---

## 🔗 下章预告

**[第九章：Agent-SDK](./09-Agent-SDK.md)**

我们将学习：
- 什么是 Agent-SDK
- SDK 安装与环境
- 核心 API 详解
- 开发 AI Agent

---

*⭐ 如果这个教程对你有帮助，欢迎 Star 支持！*
