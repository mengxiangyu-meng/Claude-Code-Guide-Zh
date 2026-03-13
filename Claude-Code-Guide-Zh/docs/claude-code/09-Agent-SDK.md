# Claude Code Agent-SDK 指南

> 本章时长：6-8 小时 | 难度：⭐⭐⭐ | 必学度：⭐⭐

---

## 📑 本章目录

1. [什么是 Agent-SDK](#1-什么是-agent-sdk)
2. [SDK 安装与环境](#2-sdk-安装与环境)
3. [核心概念](#3-核心概念)
4. [核心 API 详解](#4-核心-api-详解)
5. [开发 AI Agent](#5-开发-ai-agent)
6. [DAG 任务编排](#6-dag-任务编排)
7. [上下文管理](#7-上下文管理)
8. [实战案例](#8-实战案例)
9. [常见问题](#9-常见问题)

---

## 1. 什么是 Agent-SDK

**Agent-SDK** 是 Claude Code 的软件开发工具包，让你可以通过编程方式创建和控制 AI Agent。

### 1.1 Agent-SDK 的定义

```
Agent-SDK = 编程接口 + 智能体 + 工作流

让开发者可以：
- 编写代码创建 AI Agent
- 自定义 Agent 行为
- 编排复杂任务
- 集成到现有系统
```

### 1.2 Agent-SDK vs CLI

| 特性 | CLI | SDK |
|-----|-----|-----|
| 使用方式 | 命令行 | 编程 |
| 灵活性 | 固定功能 | 完全自定义 |
| 集成能力 | 有限 | 无限 |
| 适用用户 | 最终用户 | 开发者 |

### 1.3 Agent-SDK 能做什么

```
✅ 创建自定义 AI Agent
✅ 编程控制任务执行
✅ 构建复杂工作流
✅ 集成到现有应用
✅ 构建 AI 产品
```

---

## 2. SDK 安装与环境

### 2.1 安装 SDK

```bash
# 使用 npm
npm install @anthropic-ai/claude-sdk

# 使用 yarn
yarn add @anthropic-ai/claude-sdk

# 使用 pnpm
pnpm add @anthropic-ai/claude-sdk
```

### 2.2 环境要求

| 要求 | 最低版本 |
|-----|---------|
| Node.js | 18.0.0 |
| TypeScript | 4.7+ |

### 2.3 初始化

```typescript
import { Claude } from '@anthropic-ai/claude-sdk';

const claude = new Claude({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-sonnet-4-20250514'
});
```

### 2.4 配置选项

```typescript
const claude = new Claude({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-sonnet-4-20250514',
  maxTokens: 4096,
  temperature: 0.7,
  system: '你是一个专业的开发者...',
  tools: ['read', 'write', 'edit', 'bash']
});
```

---

## 3. 核心概念

### 3.1 Agent（智能体）

```typescript
import { Agent } from '@anthropic-ai/claude-sdk';

const agent = new Agent({
  name: 'my-agent',
  description: '我的自定义 Agent',
  systemPrompt: '你是一个...',
  tools: ['read', 'write', 'bash']
});
```

### 3.2 Task（任务）

```typescript
const task = agent.createTask({
  name: '代码审查',
  description: '审查代码质量问题',
  input: {
    files: ['src/**/*.ts']
  }
});
```

### 3.3 Workflow（工作流）

```typescript
const workflow = new Workflow({
  name: 'CI/CD 流程',
  steps: [
    { task: 'lint', runAfter: [] },
    { task: 'test', runAfter: ['lint'] },
    { task: 'build', runAfter: ['test'] },
    { task: 'deploy', runAfter: ['build'] }
  ]
});
```

### 3.4 Context（上下文）

```typescript
// 添加上下文
agent.addContext({
  type: 'file',
  content: await fs.readFile('README.md')
});

// 清除上下文
agent.clearContext();
```

---

## 4. 核心 API 详解

### 4.1 创建 Agent

```typescript
// 基本创建
const agent = await claude.createAgent({
  name: 'coder',
  description: '编程助手',
  systemPrompt: '你是一个专业的程序员，擅长...'
});
```

### 4.2 运行任务

```typescript
// 同步执行
const result = await agent.run('写一个 Python HTTP 服务器');

// 异步执行
const taskId = await agent.runAsync('复杂任务');

// 获取结果
const result = await agent.getResult(taskId);
```

### 4.3 工具调用

```typescript
// 注册自定义工具
agent.registerTool({
  name: 'my_tool',
  description: '我的工具',
  inputSchema: {
    type: 'object',
    properties: {
      param: { type: 'string' }
    }
  },
  handler: async (input) => {
    // 处理逻辑
    return { result: 'success' };
  }
});
```

### 4.4 事件监听

```typescript
// 监听任务开始
agent.on('task:start', (task) => {
  console.log('任务开始:', task.name);
});

// 监听工具调用
agent.on('tool:use', (tool) => {
  console.log('使用工具:', tool.name);
});

// 监听任务完成
agent.on('task:complete', (result) => {
  console.log('任务完成:', result);
});
```

### 4.5 错误处理

```typescript
try {
  const result = await agent.run('任务');
} catch (error) {
  if (error instanceof ClaudeError) {
    console.error('Claude 错误:', error.message);
  } else {
    console.error('其他错误:', error);
  }
}
```

---

## 5. 开发 AI Agent

### 5.1 基础 Agent

```typescript
import { Claude } from '@anthropic-ai/claude-sdk';

class MyAgent {
  private claude: Claude;

  constructor(apiKey: string) {
    this.claude = new Claude({ apiKey });
  }

  async chat(message: string) {
    const response = await this.claude.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 4096,
      messages: [{ role: 'user', content: message }]
    });
    return response.content[0].text;
  }
}

// 使用
const agent = new MyAgent(process.env.API_KEY);
const response = await agent.chat('你好');
console.log(response);
```

### 5.2 带工具的 Agent

```typescript
class ToolAgent {
  private claude: Claude;

  constructor(apiKey: string) {
    this.claude = new Claude({
      apiKey,
      tools: [
        {
          name: 'read_file',
          description: '读取文件',
          input_schema: {
            type: 'object',
            properties: {
              path: { type: 'string', description: '文件路径' }
            },
            required: ['path']
          }
        }
      ]
    });
  }

  async execute(command: string) {
    // 执行逻辑
  }
}
```

### 5.3 多 Agent 协作

```typescript
// 创建多个 Agent
const reviewer = claude.createAgent({
  name: 'reviewer',
  systemPrompt: '你是代码审查员...'
});

const coder = claude.createAgent({
  name: 'coder',
  systemPrompt: '你是程序员...'
});

// 协作
async function codeReviewAndFix(code: string) {
  // 1. 审查
  const issues = await reviewer.run(`审查代码: ${code}`);
  
  // 2. 修复
  const fixed = await coder.run(`修复这些问题: ${issues}`);
  
  return fixed;
}
```

---

## 6. DAG 任务编排

### 6.1 什么是 DAG

DAG = 有向无环图（Directed Acyclic Graph）

```
     A
    / \
   B   C
    \ /
     D
```

### 6.2 DAG 任务配置

```typescript
import { DAG } from '@anthropic-ai/claude-sdk';

const dag = new DAG();

// 定义任务
dag.addTask({
  id: 'fetch',
  agent: 'fetcher',
  runAfter: []
});

dag.addTask({
  id: 'process',
  agent: 'processor',
  runAfter: ['fetch']
});

dag.addTask({
  id: 'save',
  agent: 'saver',
  runAfter: ['process']
});

// 并行任务
dag.addTask({
  id: 'notify',
  agent: 'notifier',
  runAfter: ['save']
});

dag.addTask({
  id: 'log',
  agent: 'logger',
  runAfter: ['save']
});
```

### 6.3 执行 DAG

```typescript
// 执行
const result = await dag.execute({
  input: '数据'
});

// 监听执行
dag.on('task:start', (task) => {
  console.log(`开始: ${task.id}`);
});

dag.on('task:complete', (task, result) => {
  console.log(`完成: ${task.id}`);
});
```

### 6.4 条件分支

```typescript
dag.addTask({
  id: 'check',
  agent: 'checker',
  runAfter: [],
  condition: (result) => result.passed
});

dag.addTask({
  id: 'fix',
  agent: 'fixer',
  runAfter: ['check'],
  condition: (result) => !result.passed
});
```

---

## 7. 上下文管理

### 7.1 上下文类型

| 类型 | 说明 |
|-----|------|
| `file` | 文件内容 |
| `text` | 文本 |
| `memory` | 记忆数据 |
| `tool_result` | 工具结果 |

### 7.2 添加上下文

```typescript
// 添加文件
agent.addContext({
  type: 'file',
  name: 'README',
  content: await fs.readFile('README.md')
});

// 添加文本
agent.addContext({
  type: 'text',
  name: 'instruction',
  content: '请按照这个风格...'
});
```

### 7.3 上下文模板

```typescript
const template = agent.createContextTemplate({
  name: '代码审查模板',
  variables: ['files', 'rules'],
  render: (vars) => `
请审查以下代码：
${vars.files.join('\n')}

审查规则：
${vars.rules.join('\n')}
  `
});

// 使用
agent.addContext(template.render({
  files: ['a.ts', 'b.ts'],
  rules: ['无 console.log', '类型严格']
}));
```

### 7.4 上下文压缩

```typescript
// 自动压缩
agent.setContextStrategy('auto', {
  maxTokens: 100000
});

// 手动压缩
agent.compressContext();
```

---

## 8. 实战案例

### 8.1 案例：自动代码审查系统

```typescript
import { Claude, Agent } from '@anthropic-ai/claude-sdk';

class CodeReviewer {
  private agent: Agent;

  constructor(apiKey: string) {
    this.agent = new Claude({ apiKey, tools: ['read', 'grep'] });
  }

  async review(prUrl: string) {
    // 1. 获取 PR 变更
    const changes = await this.getPRChanges(prUrl);
    
    // 2. 并行审查
    const results = await Promise.all([
      this.agent.run(`审查安全性: ${changes}`),
      this.agent.run(`审查性能: ${changes}`),
      this.agent.run(`审查风格: ${changes}`)
    ]);
    
    // 3. 汇总结果
    return this.summarize(results);
  }

  private async getPRChanges(url: string) {
    // 获取 PR 变更逻辑
  }

  private summarize(results: any[]) {
    // 汇总逻辑
  }
}
```

### 8.2 案例：智能客服 Agent

```typescript
class CustomerServiceAgent {
  private agent: Claude;
  private tools: Map<string, Function>;

  constructor(apiKey: string) {
    this.agent = new Claude({
      apiKey,
      systemPrompt: '你是客服，礼貌、专业...',
      tools: this.registerTools()
    });
  }

  private registerTools() {
    return [
      {
        name: 'query_order',
        description: '查询订单',
        input_schema: { /* ... */ },
        handler: async (input) => await this.queryOrder(input)
      },
      {
        name: 'refund',
        description: '退款',
        input_schema: { /* ... */ },
        handler: async (input) => await this.refund(input)
      }
    ];
  }

  async handle(message: string) {
    return await this.agent.chat(message);
  }
}
```

### 8.3 案例：数据处理 Pipeline

```typescript
class DataPipeline {
  private dag: DAG;

  constructor() {
    this.dag = new DAG();
    this.setup();
  }

  private setup() {
    this.dag.addTask({ id: 'extract', agent: 'extractor' });
    this.dag.addTask({ id: 'transform', agent: 'transformer', runAfter: ['extract'] });
    this.dag.addTask({ id: 'validate', agent: 'validator', runAfter: ['transform'] });
    this.dag.addTask({ id: 'load', agent: 'loader', runAfter: ['validate'] });
  }

  async run(input: any) {
    return await this.dag.execute({ data: input });
  }
}
```

---

## 9. 常见问题

### Q1: Agent-SDK 和 CLI 哪个好？

| 场景 | 推荐 |
|-----|------|
| 日常使用 | CLI |
| 二次开发 | SDK |
| 产品集成 | SDK |
| 自动化脚本 | 都可以 |

### Q2: 如何调试 Agent？

```typescript
agent.on('*', (event) => {
  console.log('Event:', event.type, event.data);
});
```

### Q3: Agent 数量有限制吗？

没有硬性限制，但建议：
- 同一任务：1-3 个 Agent
- 复杂流程：按功能模块划分

### Q4: 如何处理 API 限流？

```typescript
const claude = new Claude({
  apiKey,
  maxRetries: 3,
  retryDelay: 1000
});
```

### Q5: 可以自定义模型吗？

可以，配置 `model` 参数：

```typescript
const claude = new Claude({
  model: 'claude-opus-4-20250514'
});
```

### Q6: 如何部署 Agent？

```typescript
// 打包
npm run build

// 部署到云函数
// AWS Lambda / Vercel / Cloudflare Workers
```

---

## ✅ 本章小结

本章学习了：

- [x] Agent-SDK 是什么及核心概念
- [x] SDK 安装与环境配置
- [x] 核心 API（创建 Agent、运行任务、工具调用、事件监听）
- [x] 开发 AI Agent（基础、带工具、多 Agent 协作）
- [x] DAG 任务编排
- [x] 上下文管理
- [x] 实战案例（代码审查、客服、数据处理）

---

## 🔗 下章预告

**[第十章：综合实战](./10-综合实战.md)**

我们将学习：
- 团队协作最佳实践
- CI/CD 集成
- 项目案例实战

---

*⭐ 如果这个教程对你有帮助，欢迎 Star 支持！*
