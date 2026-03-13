# Claude Code MCP 集成指南

> 本章时长：4-6 小时 | 难度：⭐⭐ | 必学度：⭐⭐⭐

---

## 📑 本章目录

1. [什么是 MCP](#1-什么是-mcp)
2. [MCP 服务器架构](#2-mcp-服务器架构)
3. [内置 MCP 服务器](#3-内置-mcp-服务器)
4. [安装和使用 MCP 服务器](#4-安装和使用-mcp-服务器)
5. [常用 MCP 服务推荐](#5-常用-mcp-服务推荐)
6. [自定义 MCP 开发](#6-自定义-mcp-开发)
7. [MCP 工具懒加载](#7-mcp-工具懒加载)
8. [远程 MCP 服务器](#8-远程-mcp-服务器)
9. [常见问题](#9-常见问题)

---

## 1. 什么是 MCP

**MCP（Model Context Protocol）** 是 Claude Code 强大的扩展系统，让你轻松接入各种外部工具和服务。

### 1.1 MCP 的定义

MCP 是一个开放标准协议，用于：
- 🔌 **扩展工具** - 让 Claude 调用外部 API
- 🗄️ **数据访问** - 连接数据库、文件系统
- 🌐 **服务集成** - 接入 GitHub、Slack、Notion 等

### 1.2 MCP vs 传统集成

| 特性 | 传统集成 | MCP |
|-----|---------|-----|
| 接入方式 | API 定制开发 | 标准化协议 |
| 工具数量 | 有限 | 可扩展数百个 |
| 配置难度 | 高 | 低 |
| 维护成本 | 高 | 低 |

### 1.3 MCP 能做什么

```
Claude + MCP = 超级助手

✅ 读写文件、搜索代码
✅ 操作 GitHub、GitLab
✅ 查询数据库
✅ 调用 Slack、Discord
✅ 搜索网页、获取天气
✅ ...几乎任何外部服务
```

---

## 2. MCP 服务器架构

### 2.1 架构概览

```
┌─────────────────┐
│   Claude Code   │
│                 │
│  ┌───────────┐  │
│  │ MCP Client│  │
│  └─────┬─────┘  │
└────────┼────────┘
         │ STDIO / HTTP
         ▼
┌─────────────────┐
│  MCP Server     │
│                 │
│  - 文件系统     │
│  - GitHub       │
│  - 数据库       │
│  - ...         │
└─────────────────┘
```

### 2.2 通信方式

| 方式 | 说明 | 适用场景 |
|-----|------|---------|
| **STDIO** | 本地进程通信 | 本地工具（文件系统、Git） |
| **HTTP** | 网络请求 | 远程服务（GitHub API） |
| **SSE** | 服务器推送 | 实时更新 |

### 2.3 核心概念

| 概念 | 说明 |
|-----|------|
| **MCP Server** | 提供工具的服务程序 |
| **MCP Client** | Claude Code 中的客户端 |
| **Tool** | 可调用的具体工具 |
| **Resource** | 可访问的数据资源 |
| **Prompt** | 预定义的提示词模板 |

---

## 3. 内置 MCP 服务器

Claude Code 自带一批常用的 MCP 服务器，无需额外安装。

### 3.1 文件系统服务器

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/directory"]
    }
  }
}
```

**功能：**
- 读取/写入文件
- 列出目录
- 搜索文件
- 创建/删除文件

### 3.2 Git 服务器

```json
{
  "mcpServers": {
    "git": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-git"]
    }
  }
}
```

**功能：**
- 查看 git 状态
- 查看提交历史
- 创建提交
- 创建分支

### 3.3 Puppeteer 服务器

```json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

**功能：**
- 浏览器自动化
- 网页截图
- 网页内容抓取

---

## 4. 安装和使用 MCP 服务器

### 4.1 配置文件位置

MCP 服务器配置在 `~/.claude/settings.json`：

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "package-name"]
    }
  }
}
```

### 4.2 安装步骤

**步骤 1：编辑配置文件**

```bash
# 打开配置文件
claude config edit
```

**步骤 2：添加 MCP 服务器**

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    },
    "filesystem": {
      "command": "npx", 
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  }
}
```

**步骤 3：重启 Claude Code**

```bash
# 退出并重新启动
claude
```

### 4.3 验证安装

```bash
# 查看已连接的 MCP 服务器
claude mcp list

# 测试 MCP 工具
claude "列出当前目录的文件"
```

---

## 5. 常用 MCP 服务推荐

### 5.1 开发工具类

| 服务器 | 功能 | 安装命令 |
|-------|------|---------|
| **GitHub** | GitHub API 操作 | `@modelcontextprotocol/server-github` |
| **GitLab** | GitLab API 操作 | `@modelcontextprotocol/server-gitlab` |
| **PostgreSQL** | 数据库查询 | `@modelcontextprotocol/server-postgres` |
| **SQLite** | 本地数据库 | `@modelcontextprotocol/server-sqlite` |
| **Brave Search** | 网页搜索 | `@modelcontextprotocol/server-brave-search` |

### 5.2 办公协作类

| 服务器 | 功能 | 安装命令 |
|-------|------|---------|
| **Slack** | Slack 消息 | `@modelcontextprotocol/server-slack` |
| **Google Maps** | 地图服务 | `@modelcontextprotocol/server-google-maps` |
| **Notion** | Notion 集成 | 自定义开发 |

### 5.3 系统工具类

| 服务器 | 功能 | 安装命令 |
|-------|------|---------|
| **Puppeteer** | 浏览器自动化 | `@modelcontextprotocol/server-puppeteer` |
| **Fetch** | HTTP 请求 | `@modelcontextprotocol/server-fetch` |
| **Memory** | 知识图谱 | `@modelcontextprotocol/server-memory` |

### 5.4 示例：安装 GitHub MCP

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your-token"
      }
    }
  }
}
```

**可用工具：**
- `github_get_repo` - 获取仓库信息
- `github_list_issues` - 列出 Issue
- `github_create_issue` - 创建 Issue
- `github_search_code` - 搜索代码
- `github_list_prs` - 列出 PR

---

## 6. 自定义 MCP 开发

### 6.1 创建自定义 MCP 服务器

**基本结构：**

```javascript
// server.js
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new Server({
  name: 'my-custom-server',
  version: '1.0.0'
}, {
  capabilities: {
    tools: {}
  }
});

// 定义工具
server.setRequestHandler('tools/list', async () => {
  return {
    tools: [{
      name: 'my_tool',
      description: '我的自定义工具',
      inputSchema: {
        type: 'object',
        properties: {
          param: { type: 'string', description: '参数描述' }
        },
        required: ['param']
      }
    }]
  };
});

// 处理工具调用
server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;
  
  if (name === 'my_tool') {
    // 执行操作
    return { content: [{ type: 'text', text: '结果' }] };
  }
});

// 启动服务器
const transport = new StdioServerTransport();
await server.connect(transport);
```

### 6.2 打包为 npm 包

```json
{
  "name": "my-mcp-server",
  "version": "1.0.0",
  "bin": {
    "my-mcp": "./server.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0"
  }
}
```

### 6.3 配置使用

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/path/to/server.js"]
    }
  }
}
```

---

## 7. MCP 工具懒加载

> v2.1.52+ 新功能

### 7.1 什么是工具懒加载

默认情况下，Claude Code 会加载所有 MCP 工具。懒加载可以：
- ⚡ **减少上下文** - 最多减少 95%
- 🚀 **提升速度** - 只加载需要的工具
- 💰 **节省成本** - Token 用量大幅降低

### 7.2 启用懒加载

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "lazyToolTrigger": true
    }
  }
}
```

### 7.3 触发懒加载

```bash
# 通过注释触发
# @github list_issues owner=xxx repo=yyy
claude "列出 owner/xxx 的 issues"

# 显式指定
claude "使用 github 工具列出 issues"
```

---

## 8. 远程 MCP 服务器

### 8.1 为什么用远程 MCP

- 🔒 安全隔离
- 🌐 跨网络访问
- 💻 资源受限环境
- 📦 集中管理

### 8.2 HTTP 传输配置

```json
{
  "mcpServers": {
    "remote-github": {
      "url": "https://mcp.example.com/github",
      "transport": "http"
    }
  }
}
```

### 8.3 自托管 MCP

```bash
# 启动 MCP 服务器（远程）
npx -y @modelcontextprotocol/server-github \
  --port 3000 \
  --auth my-secret-token
```

**客户端配置：**

```json
{
  "mcpServers": {
    "github": {
      "url": "http://your-server:3000/mcp",
      "headers": {
        "Authorization": "Bearer my-secret-token"
      }
    }
  }
}
```

---

## 9. 常见问题

### Q1: MCP 工具不显示？

**检查步骤：**
1. 配置文件格式是否正确
2. 服务器是否正确安装（`npx -y package-name`）
3. 重启 Claude Code

```bash
# 调试模式
claude --debug "你的请求"
```

### Q2: 安装 MCP 失败？

**常见原因：**
- Node.js 版本过低 → 升级 Node.js
- 网络问题 → 使用国内镜像
- 权限问题 → 检查 npm 权限

### Q3: Token 需要什么权限？

GitHub MCP 需要：
- `repo` - 仓库操作
- `read:user` - 用户信息

### Q4: 如何查看 MCP 日志？

```bash
# 查看 Claude Code 日志
cat ~/.claude/logs/claude.log

# 搜索 MCP 相关
grep -i mcp ~/.claude/logs/claude.log
```

### Q5: MCP 有数量限制吗？

没有硬性限制，但建议：
- 常用 MCP: 5-10 个
- 偶尔使用: 按需加载

### Q6: 懒加载不生效？

确保：
1. Claude Code 版本 >= 2.1.52
2. 配置了 `lazyToolTrigger: true`
3. 使用正确的触发方式

---

## ✅ 本章小结

本章学习了：

- [x] MCP 是什么及核心概念
- [x] MCP 服务器架构（STDIO/HTTP）
- [x] 内置 MCP 服务器
- [x] 安装和配置 MCP 服务器
- [x] 常用 MCP 服务推荐（GitHub、Slack、数据库等）
- [x] 自定义 MCP 开发
- [x] MCP 工具懒加载（v2.1.52+）
- [x] 远程 MCP 服务器

---

## 🔗 下章预告

**[第五章：Hooks 系统](./05-Hooks系统.md)**

我们将学习：
- 什么是 Hooks
- 8 种 Hook 类型详解
- 自定义 Hook 开发
- 自动化工作流

---

*⭐ 如果这个教程对你有帮助，欢迎 Star 支持！*
