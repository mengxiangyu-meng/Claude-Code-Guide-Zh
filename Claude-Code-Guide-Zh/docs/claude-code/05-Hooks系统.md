# Claude Code Hooks 系统指南

> 本章时长：4-6 小时 | 难度：⭐⭐ | 必学度：⭐⭐⭐

---

## 📑 本章目录

1. [什么是 Hooks](#1-什么是-hooks)
2. [Hook 类型详解](#2-hook-类型详解)
3. [内置 Hook](#3-内置-hook)
4. [自定义 Hook 开发](#4-自定义-hook-开发)
5. [Hook 实战案例](#5-hook-实战案例)
6. [Git Worktree 集成](#6-git-worktree-集成)
7. [自动化工作流](#7-自动化工作流)
8. [常见问题](#8-常见问题)

---

## 1. 什么是 Hooks

**Hooks** 是 Claude Code 的自动化钩子系统，让你在特定时机自动执行自定义操作。

### 1.1 Hook 的定义

Hooks = **钩子** + **脚本**

- 当某个事件发生时（触发条件）
- 自动执行预设的脚本（自定义操作）

### 1.2 Hook vs MCP

| 特性 | Hooks | MCP |
|-----|-------|-----|
| 触发时机 | 特定事件自动触发 | 手动调用工具 |
| 用途 | 自动化、工作流 | 扩展功能 |
| 编程语言 | 任意脚本 | JavaScript/TypeScript |
| 场景 | 预处理、后处理、监控 | API 调用、数据查询 |

### 1.3 Hook 能做什么

```
✅ 自动预处理提示词
✅ 任务完成后发送通知
✅ 自动格式化代码
✅ 代码提交前检查
✅ 敏感信息自动脱敏
✅ 生成变更日志
✅ ...任何自动化任务
```

---

## 2. Hook 类型详解

Claude Code 支持 8 种 Hook 类型，涵盖开发全流程。

### 2.1 PreToolUse Hook

**触发时机：** 工具执行前

```javascript
// 示例：拒绝删除文件的危险操作
{
  "matcher": ".*",
  "hooks": [
    {
      "type": "preToolUse",
      "matcher": "ToolName",
      "hooks": [
        {
          "type": "command",
          "command": ["echo", "Tool is about to run"]
        }
      ]
    }
  ]
}
```

**使用场景：**
- 记录操作日志
- 验证操作合法性
- 阻止危险操作

### 2.2 PostToolUse Hook

**触发时机：** 工具执行后

```javascript
{
  "type": "postToolUse",
  "matcher": "Bash",
  "hooks": [
    {
      "type": "command",
      "command": ["echo", "Bash command completed"]
    }
  ]
}
```

**使用场景：**
- 代码格式化
- 运行测试
- 自动提交

### 2.3 Notification Hook

**触发时机：** 任务完成或出错时

```javascript
{
  "type": "notification",
  "matcher": ".*",
  "hooks": [
    {
      "type": "command",
      "command": ["notify-send", "Claude Code", "Task completed"]
    }
  ]
}
```

**使用场景：**
- 任务完成通知
- 错误告警
- 进度提醒

### 2.4 Stop Hook

**触发时机：** 对话被中断时

```javascript
{
  "type": "stop",
  "hooks": [
    {
      "type": "command",
      "command": ["echo", "Session stopped"]
    }
  ]
}
```

**使用场景：**
- 保存工作进度
- 清理临时文件
- 记录中断点

### 2.5 Start Hook

**触发时机：** 新会话开始时

```javascript
{
  "type": "start",
  "hooks": [
    {
      "type": "command",
      "command": ["echo", "New session started"]
    }
  ]
}
```

**使用场景：**
- 初始化环境
- 显示欢迎信息
- 加载项目配置

### 2.6 Resume Hook

**触发时机：** 会话恢复时

```javascript
{
  "type": "resume",
  "hooks": [
    {
      "type": "command",
      "command": ["echo", "Session resumed"]
    }
  ]
}
```

**使用场景：**
- 恢复工作状态
- 加载上下文
- 检查中断点

### 2.7 Message Hook

**触发时机：** 收到用户消息时

```javascript
{
  "type": "message",
  "matcher": ".*",
  "hooks": [
    {
      "type": "command",
      "command": ["echo", "New message received"]
    }
  ]
}
```

**使用场景：**
- 消息日志
- 敏感词过滤
- 自动回复

### 2.8 Error Hook

**触发时机：** 发生错误时

```javascript
{
  "type": "error",
  "hooks": [
    {
      "type": "command",
      "command": ["echo", "Error occurred: $ERROR"]
    }
  ]
}
```

**使用场景：**
- 错误记录
- 自动重试
- 告警通知

---

## 3. 内置 Hook

Claude Code 自带一批内置 Hook，开箱即用。

### 3.1 默认 Hook 配置

```json
{
  "hooks": {
    "preToolUse": [],
    "postToolUse": [],
    "notification": [],
    "stop": [],
    "start": [],
    "resume": [],
    "message": [],
    "error": []
  }
}
```

### 3.2 可用的内置 Hook

| Hook | 说明 | 默认行为 |
|-----|------|---------|
| `preToolUse` | 工具执行前 | 无 |
| `postToolUse` | 工具执行后 | 无 |
| `notification` | 通知 | 系统通知 |
| `stop` | 会话停止 | 保存状态 |
| `start` | 会话开始 | 加载上下文 |

---

## 4. 自定义 Hook 开发

### 4.1 Hook 文件位置

```
~/.claude/hooks/
├── pre_tool_use.sh
├── post_tool_use.sh
├── notification.sh
└── ...
```

### 4.2 Hook 脚本模板

```bash
#!/bin/bash
# Hook 名称：pre_tool_use
# 触发时机：工具执行前

# 环境变量
# $CLAUDE_TOOL_NAME - 工具名称
# $CLAUDE_TOOL_INPUT - 工具输入（JSON）

echo "Tool: $CLAUDE_TOOL_NAME"
echo "Input: $CLAUDE_TOOL_INPUT"

# 返回值
# 0 - 继续执行
# 非 0 - 阻止执行
exit 0
```

### 4.3 环境变量参考

| 变量 | 说明 |
|-----|------|
| `CLAUDE_TOOL_NAME` | 工具名称 |
| `CLAUDE_TOOL_INPUT` | 工具输入（JSON） |
| `CLAUDE_TOOL_OUTPUT` | 工具输出（JSON） |
| `CLAUDE_SESSION_ID` | 会话 ID |
| `CLAUDE_WORKING_DIR` | 工作目录 |

### 4.4 配置 Hook

在 `~/.claude/settings.json` 中配置：

```json
{
  "hooks": {
    "preToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": ["~/.claude/hooks/pre_tool_use.sh"]
          }
        ]
      }
    ]
  }
}
```

---

## 5. Hook 实战案例

### 5.1 案例：自动代码格式化

**文件：** `~/.claude/hooks/post_tool_use.sh`

```bash
#!/bin/bash

# 仅在 Write/Edit 工具后运行
if [[ "$CLAUDE_TOOL_NAME" == "Write" || "$CLAUDE_TOOL_NAME" == "Edit" ]]; then
  # 获取文件路径
  FILE_PATH=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.file_path')
  
  # 根据文件类型格式化
  case "$FILE_PATH" in
    *.js|*.ts)
      npx prettier --write "$FILE_PATH"
      ;;
    *.py)
      black "$FILE_PATH"
      ;;
    *.go)
      gofmt -w "$FILE_PATH"
      ;;
  esac
fi

exit 0
```

### 5.2 案例：Git 提交自动生成变更日志

**文件：** `~/.claude/hooks/post_tool_use.sh`

```bash
#!/bin/bash

if [[ "$CLAUDE_TOOL_NAME" == "Bash" ]]; then
  COMMAND=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.command')
  
  # 检测 git 提交
  if [[ "$COMMAND" == *"git commit"* ]]; then
    # 生成变更日志
    git log --oneline -1 > ~/.claude/changelog.txt
    echo "✅ 自动生成变更日志"
  fi
fi

exit 0
```

### 5.3 案例：任务完成通知

**文件：** `~/.claude/settings.json`

```json
{
  "hooks": {
    "notification": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": [
              "osascript", 
              "-e", 
              "display notification \"Task Done!\" with title \"Claude Code\""
            ]
          }
        ]
      }
    ]
  }
}
```

### 5.4 案例：敏感信息检查

**文件：** `~/.claude/hooks/pre_tool_use.sh`

```bash
#!/bin/bash

if [[ "$CLAUDE_TOOL_NAME" == "Write" || "$CLAUDE_TOOL_NAME" == "Edit" ]]; then
  CONTENT=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.content')
  
  # 检查敏感信息
  if echo "$CONTENT" | grep -qiE "(api[_-]?key|password|secret|token)"; then
    echo "⚠️ 检测到敏感信息，请确认是否继续！"
    # 可以在这里阻止操作
    # exit 1
  fi
fi

exit 0
```

---

## 6. Git Worktree 集成

> v2.1.49+ 新功能

### 6.1 什么是 Git Worktree

Git Worktree 允许你在**不同目录**同时工作于同一仓库的多个分支。

### 6.2 Worktree Hook

Claude Code 提供了专门的 Worktree Hook：

| Hook | 说明 |
|-----|------|
| `WorktreeCreate` | 创建 Worktree 前 |
| `WorktreeRemove` | 删除 Worktree 前 |

### 6.3 使用 Worktree 模式

```bash
# 创建新分支的 Worktree
claude -w feature-branch

# 指定分支和目录
claude -w feature-branch:/path/to/worktree
```

### 6.4 Worktree Hook 示例

```json
{
  "hooks": {
    "WorktreeCreate": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": ["echo", "Creating worktree for $CLAUDE_BRANCH"]
          }
        ]
      }
    ]
  }
}
```

### 6.5 隔离任务的好处

- 🔒 **分支隔离** - 避免意外修改主分支
- 🚀 **并行开发** - 同时处理多个任务
- 🧹 **干净环境** - 每次都是全新工作区

---

## 7. 自动化工作流

### 7.1 完整工作流示例

结合多种 Hook 实现完整自动化：

```json
{
  "hooks": {
    "start": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": ["echo", "🚀 开始新任务"]
          }
        ]
      }
    ],
    "postToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": ["~/.claude/hooks/format.sh"]
          }
        ]
      },
      {
        "matcher": "Bash.*git commit",
        "hooks": [
          {
            "type": "command",
            "command": ["~/.claude/hooks/changelog.sh"]
          }
        ]
      }
    ],
    "notification": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": ["notify-send", "Claude Code", "任务完成"]
          }
        ]
      }
    ]
  }
}
```

### 7.2 条件判断

```bash
#!/bin/bash
# 复杂的条件逻辑

case "$CLAUDE_TOOL_NAME" in
  "Write")
    # 处理写入
    ;;
  "Edit")
    # 处理编辑
    ;;
  "Bash")
    # 处理命令
    ;;
esac
```

### 7.3 错误处理

```bash
#!/bin/bash

set -e

# 你的逻辑
if [[ $? -ne 0 ]]; then
  echo "❌ Hook 执行失败"
  # 发送错误通知
  exit 1
fi

exit 0
```

---

## 8. 常见问题

### Q1: Hook 不生效？

**检查步骤：**
1. 脚本是否有执行权限：`chmod +x ~/.claude/hooks/*.sh`
2. 配置文件格式是否正确
3. 重启 Claude Code

### Q2: 怎么调试 Hook？

```bash
# 在脚本中添加调试
set -x
echo "DEBUG: $CLAUDE_TOOL_NAME"
```

### Q3: Hook 能阻止操作吗？

可以，在 PreToolUse Hook 中返回非 0 退出码：

```bash
#!/bin/bash
# 阻止危险操作
if [[ "$CLAUDE_TOOL_NAME" == "rm" ]]; then
  echo "❌ 禁止使用 rm 命令"
  exit 1
fi
```

### Q4: Hook 有执行顺序吗？

有，按配置文件中的顺序执行。

### Q5: 可以禁用某个 Hook 吗？

可以，在配置中注释掉或删除：

```json
{
  "hooks": {
    // "preToolUse": [...]
  }
}
```

### Q6: Worktree 和普通模式哪个好？

| 场景 | 推荐模式 |
|-----|---------|
| 简单任务 | 普通模式 |
| 多分支并行 | Worktree 模式 |
| 实验性改动 | Worktree 模式 |
| 生产环境操作 | 普通模式 |

---

## ✅ 本章小结

本章学习了：

- [x] Hooks 是什么及 8 种类型
- [x] PreToolUse / PostToolUse / Notification / Stop 等
- [x] 内置 Hook
- [x] 自定义 Hook 开发
- [x] 实战案例：格式化、变更日志、通知、敏感检查
- [x] Git Worktree 集成（v2.1.49+）
- [x] 自动化工作流

---

## 🔗 下章预告

**[第六章：Subagent 子代理](./06-Subagent子代理.md)**

我们将学习：
- 什么是 Subagent
- 内置子代理列表
- 子代理配置
- 多代理协作模式

---

*⭐ 如果这个教程对你有帮助，欢迎 Star 支持！*
