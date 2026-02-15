# 集成指南

Langfuse 提供了多种集成方式，可以与您现有的开发工具链、LLM 平台和监控系统无缝协作。本指南将介绍如何设置和使用这些集成。

## 目录

1. [SDK 集成](#sdk-集成)
2. [LLM 平台集成](#llm-平台集成)
3. [框架集成](#框架集成)
4. [开发工具集成](#开发工具集成)
5. [监控与分析集成](#监控与分析集成)
6. [数据仓库集成](#数据仓库集成)
7. [自定义集成](#自定义集成)
8. [集成最佳实践](#集成最佳实践)

## SDK 集成

### Node.js SDK

#### 安装

```bash
npm install langfuse
```

#### 基本使用

```javascript
import { Langfuse } from "langfuse";

const langfuse = new Langfuse({
  publicKey: process.env.LANGFUSE_PUBLIC_KEY,
  secretKey: process.env.LANGFUSE_SECRET_KEY,
  baseUrl: process.env.LANGFUSE_BASE_URL, // 自托管时需要设置
});

// 创建追踪
const trace = langfuse.trace({
  name: "user-chat",
  userId: "user-123",
});

// 记录提示和响应
const prompt = trace.prompt({
  name: "chat-prompt",
  input: "Hello, how can I help you today?",
});

// 记录 LLM 响应
prompt.completion({
  output: "I'm here to assist you with any questions or tasks you have.",
  metadata: { tokenUsage: { promptTokens: 10, completionTokens: 15, totalTokens: 25 } },
});

// 关闭追踪
await trace.close();
```

### Python SDK

#### 安装

```bash
pip install langfuse
```

#### 基本使用

```python
from langfuse import Langfuse

langfuse = Langfuse(
    public_key=os.environ["LANGFUSE_PUBLIC_KEY"],
    secret_key=os.environ["LANGFUSE_SECRET_KEY"],
    base_url=os.environ["LANGFUSE_BASE_URL"]  # 自托管时需要设置
)

# 创建追踪
trace = langfuse.trace(
    name="user-chat",
    user_id="user-123"
)

# 记录提示和响应
prompt = trace.prompt(
    name="chat-prompt",
    input="Hello, how can I help you today?"
)

# 记录 LLM 响应
prompt.completion(
    output="I'm here to assist you with any questions or tasks you have.",
    metadata={"tokenUsage": {"promptTokens": 10, "completionTokens": 15, "totalTokens": 25}}
)

# 关闭追踪
trace.close()
```

### Go SDK

#### 安装

```bash
go get github.com/langfuse/langfuse-go
```

#### 基本使用

```go
import "github.com/langfuse/langfuse-go"

func main() {
    client := langfuse.NewClient(
        langfuse.WithPublicAPIKey("your-public-key"),
        langfuse.WithSecretAPIKey("your-secret-key"),
        langfuse.WithBaseURL("https://cloud.langfuse.com"), // 自托管时需要设置
    )

    // 创建追踪
    trace := client.Trace(
        langfuse.WithTraceName("user-chat"),
        langfuse.WithTraceUserID("user-123"),
    )

    // 记录提示和响应
    prompt := trace.Prompt(
        langfuse.WithPromptName("chat-prompt"),
        langfuse.WithPromptInput("Hello, how can I help you today?"),
    )

    // 记录 LLM 响应
    prompt.Completion(
        langfuse.WithCompletionOutput("I'm here to assist you with any questions or tasks you have."),
        langfuse.WithCompletionMetadata(map[string]interface{}{
            "tokenUsage": map[string]interface{}{
                "promptTokens":  10,
                "completionTokens": 15,
                "totalTokens":  25,
            },
        }),
    )

    // 关闭追踪
    trace.Close()
}
```

## LLM 平台集成

### OpenAI

#### 安装

```bash
npm install langfuse-openai
```

#### 基本使用

```javascript
import { LangfuseOpenAI } from "langfuse-openai";
import OpenAI from "openai";

const openai = new LangfuseOpenAI({
  openai: new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  }),
  langfuse: new Langfuse({
    publicKey: process.env.LANGFUSE_PUBLIC_KEY,
    secretKey: process.env.LANGFUSE_SECRET_KEY,
  }),
  // 可选配置
  traceName: "openai-chat",
  userId: "user-123",
});

// 正常使用 OpenAI SDK，自动追踪
const response = await openai.chat.completions.create({
  model: "gpt-3.5-turbo",
  messages: [{ role: "user", content: "Hello, how are you?" }],
});
```

### Anthropic

#### 安装

```bash
npm install langfuse-anthropic
```

#### 基本使用

```javascript
import { LangfuseAnthropic } from "langfuse-anthropic";
import Anthropic from "@anthropic-ai/sdk";

const anthropic = new LangfuseAnthropic({
  anthropic: new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  }),
  langfuse: new Langfuse({
    publicKey: process.env.LANGFUSE_PUBLIC_KEY,
    secretKey: process.env.LANGFUSE_SECRET_KEY,
  }),
});

// 正常使用 Anthropic SDK，自动追踪
const response = await anthropic.messages.create({
  model: "claude-3-opus-20240229",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, how are you?" }],
});
```

### Google AI (Gemini)

#### 安装

```bash
npm install langfuse-google-ai
```

#### 基本使用

```javascript
import { LangfuseGoogleAI } from "langfuse-google-ai";
import { GoogleGenerativeAI } from "@google/generative-ai";

const genAI = new LangfuseGoogleAI({
  googleAI: new GoogleGenerativeAI(process.env.GOOGLE_AI_API_KEY),
  langfuse: new Langfuse({
    publicKey: process.env.LANGFUSE_PUBLIC_KEY,
    secretKey: process.env.LANGFUSE_SECRET_KEY,
  }),
});

// 正常使用 Google AI SDK，自动追踪
const model = genAI.getGenerativeModel({ model: "gemini-1.5-pro" });
const result = await model.generateContent("Hello, how are you?");
```

## 框架集成

### Next.js

#### 安装

```bash
npm install langfuse langfuse-nextjs
```

#### 基本使用

1. **创建配置文件** (`lib/langfuse.ts`):

```javascript
import { Langfuse } from "langfuse";

export const langfuse = new Langfuse({
  publicKey: process.env.LANGFUSE_PUBLIC_KEY,
  secretKey: process.env.LANGFUSE_SECRET_KEY,
  baseUrl: process.env.LANGFUSE_BASE_URL,
});
```

2. **在 API 路由中使用**:

```javascript
import { langfuse } from "../../lib/langfuse";

export default async function handler(req, res) {
  const trace = langfuse.trace({
    name: "api-request",
    userId: req.body.userId,
  });

  try {
    // 处理请求...
    res.status(200).json({ message: "Success" });
  } catch (error) {
    trace.error(error);
    res.status(500).json({ error: "Internal Server Error" });
  } finally {
    await trace.close();
  }
}
```

3. **在页面组件中使用**:

```javascript
'use client';
import { LangfuseClient } from "langfuse";

const langfuseClient = new LangfuseClient({
  publicKey: process.env.NEXT_PUBLIC_LANGFUSE_PUBLIC_KEY,
  baseUrl: process.env.NEXT_PUBLIC_LANGFUSE_BASE_URL,
});

export default function Home() {
  const handleClick = async () => {
    const trace = langfuseClient.trace({
      name: "user-interaction",
      userId: "user-123",
    });

    try {
      // 处理交互...
    } catch (error) {
      trace.error(error);
    } finally {
      await trace.close();
    }
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

### Flask

#### 安装

```bash
pip install langfuse langfuse-flask
```

#### 基本使用

```python
from flask import Flask, request
from langfuse import Langfuse
from langfuse.flask import LangfuseFlask

app = Flask(__name__)

# 初始化 Langfuse
langfuse = Langfuse(
    public_key=os.environ["LANGFUSE_PUBLIC_KEY"],
    secret_key=os.environ["LANGFUSE_SECRET_KEY"]
)

# 初始化 Flask 扩展
langfuse_flask = LangfuseFlask(app, langfuse)

@app.route('/api/chat', methods=['POST'])
def chat():
    # 自动创建追踪
    trace = request.langfuse_trace
    
    # 记录提示
    prompt = trace.prompt(
        name="chat-prompt",
        input=request.json.get("message")
    )
    
    # 处理请求...
    response = generate_response(request.json.get("message"))
    
    # 记录响应
    prompt.completion(output=response)
    
    return {"response": response}
```

## 开发工具集成

### VS Code 扩展

Langfuse 提供了 VS Code 扩展，方便在开发过程中直接访问和管理提示。

#### 安装

1. 在 VS Code 扩展市场中搜索 "Langfuse"
2. 点击 "Install" 安装扩展
3. 点击 "Login" 并使用您的 Langfuse 账号登录

#### 功能

- 直接在 VS Code 中查看和编辑提示
- 版本控制和历史记录
- 一键测试提示
- 与代码编辑器无缝集成

### GitHub Actions 集成

使用 GitHub Actions 自动化部署和集成 Langfuse。

#### 示例配置

```yaml
name: Langfuse Deployment

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_HUB_USERNAME }}
        password: ${{ secrets.DOCKER_HUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: yourusername/langfuse:latest
    
    - name: Deploy to Langfuse Cloud
      uses: langfuse/deploy-action@v1
      with:
        langfuse-api-key: ${{ secrets.LANGFUSE_API_KEY }}
        environment: production
```

## 监控与分析集成

### Datadog

将 Langfuse 指标导出到 Datadog 进行监控和分析。

#### 配置

1. 在 Langfuse 项目设置中，点击 "Integrations" 选项卡
2. 选择 "Datadog"
3. 输入您的 Datadog API 密钥和应用密钥
4. 选择要导出的指标
5. 点击 "Save"

### Prometheus & Grafana

设置 Prometheus 从 Langfuse 收集指标，并使用 Grafana 进行可视化。

#### 配置 Prometheus

```yaml
scrape_configs:
  - job_name: 'langfuse'
    static_configs:
      - targets: ['langfuse:3000']
    metrics_path: '/api/public/metrics'
    bearer_token: 'your-langfuse-api-key'
```

#### 配置 Grafana

1. 在 Grafana 中添加 Prometheus 数据源
2. 导入 Langfuse 预设仪表板（可在 Langfuse 文档中下载）
3. 自定义仪表板以满足您的需求

## 数据仓库集成

### BigQuery

将 Langfuse 数据导出到 BigQuery 进行高级分析。

#### 配置

1. 在 Langfuse 项目设置中，点击 "Integrations" 选项卡
2. 选择 "BigQuery"
3. 输入您的 Google Cloud 项目 ID
4. 上传服务账号密钥
5. 配置导出频率和数据范围
6. 点击 "Save"

### Snowflake

将 Langfuse 数据导出到 Snowflake 进行数据仓库分析。

#### 配置

1. 在 Langfuse 项目设置中，点击 "Integrations" 选项卡
2. 选择 "Snowflake"
3. 输入您的 Snowflake 连接信息
4. 配置导出频率和数据范围
5. 点击 "Save"

## 自定义集成

### Webhook

使用 Webhook 接收 Langfuse 事件通知。

#### 配置

1. 在 Langfuse 项目设置中，点击 "Webhook" 选项卡
2. 输入您的 Webhook URL
3. 选择要接收的事件类型
4. 点击 "Save"

#### 事件示例

```json
{
  "event": "trace.completed",
  "data": {
    "id": "trace-123",
    "name": "user-chat",
    "userId": "user-456",
    "duration": 1200,
    "createdAt": "2023-01-01T00:00:00Z",
    "metadata": {
      "userType": "premium"
    }
  }
}
```

### API 集成

使用 Langfuse API 构建自定义集成。

#### 示例：使用 API 获取追踪数据

```bash
curl -X GET "https://cloud.langfuse.com/api/public/traces" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"filters":{"name":"user-chat"},"limit":10}'
```

## 集成最佳实践

### 1. 分层集成

- **核心层**：在应用核心代码中集成 SDK，确保所有 LLM 调用都被追踪
- **框架层**：利用框架扩展（如 Next.js、Flask 集成）简化集成
- **工具层**：在开发工具中集成，提高开发效率
- **监控层**：与监控系统集成，实现统一的可观测性

### 2. 一致的命名规范

- 为追踪和观察点使用一致的命名规范
- 确保元数据键名在整个应用中保持一致
- 使用有意义的名称，便于后续分析

### 3. 数据隐私

- 避免记录敏感数据（如密码、信用卡信息）
- 使用数据掩码或过滤技术保护用户隐私
- 遵循相关的数据保护法规（如 GDPR、CCPA）

### 4. 性能优化

- 在高流量场景下，考虑批量处理追踪数据
- 使用异步方法避免阻塞主应用流程
- 对非常高频的操作使用采样策略

### 5. 错误处理

- 确保集成代码具有良好的错误处理机制
- 避免集成错误影响主应用功能
- 记录集成错误，便于排查问题

## 下一步

- 查看 [API 文档](../api/README.md) 了解更多高级集成选项
- 探索 [使用指南](../usage/README.md) 了解如何充分利用 Langfuse 功能
- 查看 [故障排除](../troubleshooting/README.md) 解决常见集成问题

如果您有任何疑问或需要帮助，请访问 [社区论坛](https://github.com/langfuse/langfuse/discussions) 或联系我们的 [支持团队](mailto:support@langfuse.com)。