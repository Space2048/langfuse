# 使用指南

本指南将帮助您快速上手并充分利用 Langfuse 的各项功能。

## 目录

1. [快速开始](#快速开始)
2. [项目设置](#项目设置)
3. [追踪 LLM 应用](#追踪-llm-应用)
4. [观察点管理](#观察点管理)
5. [提示管理](#提示管理)
6. [评估系统](#评估系统)
7. [指标与分析](#指标与分析)
8. [高级功能](#高级功能)
9. [最佳实践](#最佳实践)

## 快速开始

### 1. 创建项目

1. 登录 Langfuse 后，点击「New Project」按钮
2. 输入项目名称和描述
3. 选择项目类型（如「应用开发」、「研究」等）
4. 点击「Create Project」完成创建

### 2. 获取 API 密钥

1. 进入项目设置
2. 点击「API Keys」选项卡
3. 点击「Generate New Key」
4. 复制生成的 `PUBLIC_KEY` 和 `SECRET_KEY`
5. 在您的应用中配置这些密钥

### 3. 安装 SDK

根据您的编程语言，选择对应的 SDK：

```bash
# Node.js
npm install langfuse

# Python
pip install langfuse

# Go
go get github.com/langfuse/langfuse-go

# Rust
cargo add langfuse
```

### 4. 基本集成

以 Node.js 为例：

```javascript
import { Langfuse } from "langfuse";

// 初始化客户端
const langfuse = new Langfuse({
  publicKey: process.env.LANGFUSE_PUBLIC_KEY,
  secretKey: process.env.LANGFUSE_SECRET_KEY,
  baseUrl: process.env.LANGFUSE_BASE_URL, // 自托管时需要设置
});

// 追踪会话
async function runChat() {
  const trace = langfuse.trace({
    name: "user-chat",
    userId: "user-123",
    metadata: { userType: "premium" },
  });

  try {
    // 追踪提示
    const prompt = trace.prompt({
      name: "chat-prompt",
      input: "Hello, how can I help you today?",
      metadata: { model: "gpt-4" },
    });

    // 模拟 LLM 调用
    const response = await llm.chat.completions.create({
      model: "gpt-4",
      messages: [{ role: "user", content: "Hello, how can I help you today?" }],
    });

    // 记录 LLM 响应
    prompt.completion({
      output: response.choices[0].message.content,
      metadata: { tokenUsage: response.usage },
    });

    // 记录评分
    trace.score({
      name: "user-satisfaction",
      value: 4.5,
      comment: "Helpful response",
    });
  } catch (error) {
    trace.error(error);
  } finally {
    await trace.close();
  }
}
```

## 项目设置

### 项目配置

在项目设置中，您可以配置以下内容：

- **基本信息**：项目名称、描述、头像
- **API 密钥**：管理项目的 API 密钥
- **集成**：连接第三方服务
- **权限**：管理项目成员和权限
- **环境变量**：配置项目特定的环境变量
- **Webhook**：设置事件通知

### 团队协作

1. **邀请成员**：在项目设置中，点击「Members」选项卡，输入邮箱邀请成员
2. **设置权限**：为不同成员设置不同角色（管理员、编辑者、查看者）
3. **创建团队**：将相关项目和成员组织成团队

## 追踪 LLM 应用

### 什么是追踪？

追踪（Trace）是 Langfuse 的核心概念，用于记录 LLM 应用的完整执行流程。一个追踪可以包含多个观察点（Observations），如提示、完成、函数调用等。

### 创建追踪

```javascript
// 创建基本追踪
const trace = langfuse.trace({
  name: "document-summarization",
  userId: "user-456",
});

// 创建带有元数据的追踪
const traceWithMetadata = langfuse.trace({
  name: "document-summarization",
  userId: "user-456",
  metadata: {
    documentType: "research-paper",
    documentLength: 10000,
  },
});
```

### 追踪嵌套

对于复杂的应用，您可以创建嵌套的追踪：

```javascript
const parentTrace = langfuse.trace({ name: "multi-step-workflow" });

// 创建子追踪
const childTrace1 = parentTrace.trace({ name: "step-1" });
const childTrace2 = parentTrace.trace({ name: "step-2" });

// 完成所有追踪
await childTrace1.close();
await childTrace2.close();
await parentTrace.close();
```

## 观察点管理

### 观察点类型

Langfuse 支持多种观察点类型：

- **Prompt**：提示及其响应
- **Completion**：LLM 生成的响应
- **Function Call**：函数调用
- **Event**：自定义事件
- **Score**：评估分数

### 创建观察点

```javascript
const trace = langfuse.trace({ name: "example-trace" });

// 创建提示观察点
const prompt = trace.prompt({
  name: "translation-prompt",
  input: "Translate 'Hello' to Spanish",
  metadata: { model: "gpt-3.5-turbo" },
});

// 创建函数调用观察点
const functionCall = trace.functionCall({
  name: "fetch-weather",
  input: { city: "New York" },
});

// 记录函数结果
functionCall.result({
  output: { temperature: 25, condition: "sunny" },
});

// 创建自定义事件
const customEvent = trace.event({
  name: "user-interaction",
  metadata: { action: "button-click", element: "submit-btn" },
});
```

## 提示管理

### 创建和管理提示

1. 登录 Langfuse 平台，点击「Prompts」选项卡
2. 点击「New Prompt」按钮
3. 输入提示名称、内容和版本
4. 保存提示

### 使用版本控制

Langfuse 提供完整的提示版本控制：

1. 编辑提示内容
2. 点击「Save as New Version」
3. 输入版本描述
4. 保存新版本

### 从 SDK 使用提示

```javascript
// 从 Langfuse 获取提示
const prompt = await langfuse.prompts.get("translation-prompt", { version: "1.0.0" });

// 使用提示内容
const response = await llm.chat.completions.create({
  model: "gpt-3.5-turbo",
  messages: [{ role: "user", content: prompt.content }],
});

// 记录使用情况
await prompt.log({
  input: { text: "Hello" },
  output: { translation: response.choices[0].message.content },
  metadata: { model: "gpt-3.5-turbo" },
});
```

## 评估系统

### 创建评估

1. 点击「Evaluations」选项卡
2. 点击「New Evaluation」按钮
3. 输入评估名称、描述和评估类型
4. 配置评估指标
5. 保存评估

### 手动评估

```javascript
const trace = langfuse.trace({ name: "example-trace" });

// 添加评分
await trace.score({
  name: "relevance",
  value: 4.0,
  comment: "Response is mostly relevant",
  metadata: { evaluator: "human" },
});
```

### 自动化评估

```javascript
const trace = langfuse.trace({ name: "example-trace" });

// 自动化评分（例如使用另一个 LLM 进行评估）
const evaluationResult = await evaluateResponse(relevancePrompt, userQuery, llmResponse);

await trace.score({
  name: "relevance",
  value: evaluationResult.score,
  comment: evaluationResult.reasoning,
  metadata: { evaluator: "automated", model: "gpt-4" },
});
```

### 查看评估结果

在 Langfuse 平台上，您可以：

1. 查看单个评估的详细结果
2. 比较不同版本的性能
3. 生成评估报告
4. 导出评估数据

## 指标与分析

### 查看指标

在「Metrics」选项卡中，您可以查看：

- **使用情况**：API 调用次数、令牌使用量
- **性能**：响应时间、延迟分布
- **质量**：评估分数、满意度评分
- **成本**：API 调用成本、资源消耗

### 创建自定义指标

1. 点击「Custom Metrics」按钮
2. 输入指标名称和描述
3. 选择指标类型（计数、平均值、总和等）
4. 配置过滤条件
5. 保存指标

### 分析追踪数据

```javascript
// 查询追踪数据
const traces = await langfuse.traces.query({
  filters: {
    name: "document-summarization",
    dateRange: { start: "2023-01-01", end: "2023-01-31" },
  },
  sort: { field: "createdAt", direction: "desc" },
  limit: 100,
});

// 分析数据
traces.forEach((trace) => {
  console.log(`Trace: ${trace.name}, Duration: ${trace.duration}ms`);
});
```

## 高级功能

### 批量操作

```javascript
// 批量记录追踪
const traces = [
  { name: "trace-1", userId: "user-1" },
  { name: "trace-2", userId: "user-2" },
];

await Promise.all(
  traces.map(async (traceData) => {
    const trace = langfuse.trace(traceData);
    // ... 添加观察点 ...
    await trace.close();
  })
);
```

### 异步追踪

对于需要长时间运行的任务，可以使用异步追踪：

```javascript
// 开始异步追踪
const trace = langfuse.trace({
  name: "long-running-task",
  userId: "user-123",
  async: true,
});

// 在后续请求中继续追踪
const existingTrace = langfuse.trace({ id: trace.id });

// 添加观察点
existingTrace.event({ name: "task-progress", metadata: { progress: 50 } });

// 完成追踪
await existingTrace.close();
```

### 导出数据

```javascript
// 导出追踪数据
const exportData = await langfuse.export({
  dateRange: { start: "2023-01-01", end: "2023-01-31" },
  format: "json",
});

// 保存到文件
fs.writeFileSync("langfuse-export.json", JSON.stringify(exportData, null, 2));
```

## 最佳实践

### 1. 一致的命名规范

为追踪和观察点使用一致的命名规范：

- 使用描述性名称：`customer-support-chat` 而不是 `trace-123`
- 使用小写和连字符：`document-summarization` 而不是 `DocumentSummarization`
- 保持命名简洁但信息丰富

### 2. 合理使用元数据

利用元数据来丰富追踪信息：

- 包含用户信息：`userId`, `userType`
- 包含应用上下文：`featureName`, `environment`
- 包含 LLM 相关信息：`model`, `temperature`, `maxTokens`

### 3. 适当的粒度

- 为每个用户会话创建一个顶级追踪
- 为每个主要步骤创建子追踪
- 为每个 LLM 调用和关键函数调用创建观察点

### 4. 错误处理

确保正确处理错误：

```javascript
try {
  const trace = langfuse.trace({ name: "example-trace" });
  // ... 执行操作 ...
  await trace.close();
} catch (error) {
  console.error("Error:", error);
  // 确保追踪被关闭
  if (trace) {
    trace.error(error);
    await trace.close();
  }
}
```

### 5. 性能考虑

- 批量处理：在高流量场景下，考虑批量记录追踪
- 异步处理：使用异步方法避免阻塞主应用流程
- 采样：对于高流量应用，可以考虑采样记录追踪

## 下一步

- 查看 [API 文档](../api/README.md) 了解更多高级功能
- 探索 [集成指南](../integrations/README.md) 了解如何与其他工具集成
- 查看 [故障排除](../troubleshooting/README.md) 解决常见问题

如果您有任何疑问或需要帮助，请访问 [社区论坛](https://github.com/langfuse/langfuse/discussions) 或联系我们的 [支持团队](mailto:support@langfuse.com)。