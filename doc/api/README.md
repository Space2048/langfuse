# API 文档

本文档详细介绍了 Langfuse 的 REST API 接口，包括认证、资源操作、错误处理和最佳实践。

## 📋 API 概览

### 基础信息
- **API 版本**: v2
- **基础 URL**: `https://your-domain.com/api/public`
- **内容类型**: `application/json`
- **认证方式**: Bearer Token, API Key
- **速率限制**: 1000 请求/分钟/用户

### 状态码
| 状态码 | 说明 | 常见场景 |
|--------|------|----------|
| `200` | 成功 | GET, PUT, PATCH 成功 |
| `201` | 创建成功 | POST 创建资源 |
| `204` | 无内容 | DELETE 成功 |
| `400` | 请求错误 | 参数验证失败 |
| `401` | 未授权 | 缺少或无效的认证 |
| `403` | 禁止访问 | 权限不足 |
| `404` | 未找到 | 资源不存在 |
| `429` | 请求过多 | 超过速率限制 |
| `500` | 服务器错误 | 内部服务器错误 |

## 🔐 认证和授权

### API Key 认证
```bash
# 在请求头中添加 API Key
curl -X GET \
  -H "Authorization: Bearer YOUR_API_KEY" \
  https://your-domain.com/api/public/traces
```

### 获取 API Key
1. 登录 Langfuse 控制台
2. 进入项目设置 → API Keys
3. 点击 "Create API Key"
4. 复制生成的密钥（只显示一次）

### 权限范围
```typescript
// API Key 权限
interface ApiKeyPermissions {
  // 追踪相关权限
  traces: {
    read: boolean;
    create: boolean;
    update: boolean;
    delete: boolean;
  };
  
  // 评估相关权限
  evaluations: {
    read: boolean;
    create: boolean;
    update: boolean;
  };
  
  // 提示相关权限
  prompts: {
    read: boolean;
    create: boolean;
    update: boolean;
  };
  
  // 项目相关权限
  projects: {
    read: boolean;
    update: boolean;
  };
}
```

## 📊 追踪 API

### 创建追踪
```http
POST /api/public/traces
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "name": "User Authentication",
  "input": {
    "username": "john@example.com",
    "action": "login"
  },
  "metadata": {
    "environment": "production",
    "version": "1.2.3"
  },
  "sessionId": "session-123",
  "userId": "user-456",
  "tags": ["auth", "login"]
}
```

**响应**:
```json
{
  "id": "trace-789",
  "name": "User Authentication",
  "timestamp": "2024-01-15T10:30:00Z",
  "input": {
    "username": "john@example.com",
    "action": "login"
  },
  "metadata": {
    "environment": "production",
    "version": "1.2.3"
  },
  "sessionId": "session-123",
  "userId": "user-456",
  "tags": ["auth", "login"],
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z"
}
```

### 获取追踪列表
```http
GET /api/public/traces
Authorization: Bearer YOUR_API_KEY

# 查询参数
?page=1                    # 页码，默认 1
?limit=50                  # 每页数量，默认 50，最大 1000
?sortBy=timestamp          # 排序字段：timestamp, createdAt, name
?sortOrder=desc            # 排序顺序：asc, desc
?userId=user-456           # 按用户ID过滤
?sessionId=session-123     # 按会话ID过滤
?name=Authentication       # 按名称搜索（模糊匹配）
?tags=auth,login           # 按标签过滤（逗号分隔）
?startDate=2024-01-01      # 开始日期（包含）
?endDate=2024-01-31        # 结束日期（包含）
```

**响应**:
```json
{
  "data": [
    {
      "id": "trace-789",
      "name": "User Authentication",
      "timestamp": "2024-01-15T10:30:00Z",
      "userId": "user-456",
      "sessionId": "session-123",
      "tags": ["auth", "login"],
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 125,
    "totalPages": 3,
    "hasNext": true,
    "hasPrev": false
  }
}
```

### 获取单个追踪
```http
GET /api/public/traces/{traceId}
Authorization: Bearer YOUR_API_KEY
```

**响应**:
```json
{
  "id": "trace-789",
  "name": "User Authentication",
  "timestamp": "2024-01-15T10:30:00Z",
  "input": {
    "username": "john@example.com",
    "action": "login"
  },
  "output": {
    "success": true,
    "userId": "user-456",
    "sessionToken": "token-abc"
  },
  "metadata": {
    "environment": "production",
    "version": "1.2.3",
    "duration": 125.5,
    "totalTokens": 1500
  },
  "sessionId": "session-123",
  "userId": "user-456",
  "tags": ["auth", "login"],
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z",
  "observations": [
    {
      "id": "obs-001",
      "type": "span",
      "name": "Validate Credentials",
      "startTime": "2024-01-15T10:30:00Z",
      "endTime": "2024-01-15T10:30:02Z",
      "duration": 2.0,
      "metadata": {
        "validationMethod": "jwt"
      }
    }
  ],
  "scores": [
    {
      "id": "score-001",
      "name": "accuracy",
      "value": 0.95,
      "source": "auto",
      "comment": "High accuracy"
    }
  ]
}
```

### 更新追踪
```http
PATCH /api/public/traces/{traceId}
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "name": "Updated Trace Name",
  "output": {
    "success": true,
    "message": "Authentication successful"
  },
  "metadata": {
    "duration": 150.2,
    "totalTokens": 1800
  },
  "tags": ["auth", "login", "success"]
}
```

### 删除追踪
```http
DELETE /api/public/traces/{traceId}
Authorization: Bearer YOUR_API_KEY
```

## 🔍 观察点 API

### 创建观察点（Span）
```http
POST /api/public/observations
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "traceId": "trace-789",
  "type": "span",
  "name": "Database Query",
  "startTime": "2024-01-15T10:30:00Z",
  "endTime": "2024-01-15T10:30:01Z",
  "input": {
    "query": "SELECT * FROM users WHERE email = ?",
    "params": ["john@example.com"]
  },
  "output": {
    "rowCount": 1,
    "duration": 1.0
  },
  "metadata": {
    "database": "postgres",
    "table": "users"
  },
  "level": "DEFAULT",
  "parentObservationId": "obs-001",
  "version": "1.0.0"
}
```

### 创建观察点（Generation）
```http
POST /api/public/observations
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "traceId": "trace-789",
  "type": "generation",
  "name": "LLM Completion",
  "startTime": "2024-01-15T10:30:00Z",
  "endTime": "2024-01-15T10:30:05Z",
  "model": "gpt-4",
  "modelParameters": {
    "temperature": 0.7,
    "maxTokens": 1000
  },
  "input": {
    "messages": [
      {
        "role": "user",
        "content": "Explain quantum computing in simple terms"
      }
    ]
  },
  "output": {
    "message": {
      "role": "assistant",
      "content": "Quantum computing uses qubits..."
    }
  },
  "usage": {
    "input": 150,
    "output": 850,
    "total": 1000,
    "unit": "TOKENS"
  },
  "metadata": {
    "provider": "openai",
    "cost": 0.02
  }
}
```

### 批量创建观察点
```http
POST /api/public/observations/batch
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "observations": [
    {
      "traceId": "trace-789",
      "type": "span",
      "name": "Step 1",
      "startTime": "2024-01-15T10:30:00Z",
      "endTime": "2024-01-15T10:30:01Z"
    },
    {
      "traceId": "trace-789",
      "type": "span",
      "name": "Step 2",
      "startTime": "2024-01-15T10:30:01Z",
      "endTime": "2024-01-15T10:30:02Z",
      "parentObservationId": "obs-001"
    }
  ]
}
```

### 获取观察点
```http
GET /api/public/observations/{observationId}
Authorization: Bearer YOUR_API_KEY
```

**响应**:
```json
{
  "id": "obs-001",
  "traceId": "trace-789",
  "type": "span",
  "name": "Database Query",
  "startTime": "2024-01-15T10:30:00Z",
  "endTime": "2024-01-15T10:30:01Z",
  "duration": 1.0,
  "input": {
    "query": "SELECT * FROM users WHERE email = ?",
    "params": ["john@example.com"]
  },
  "output": {
    "rowCount": 1,
    "duration": 1.0
  },
  "metadata": {
    "database": "postgres",
    "table": "users"
  },
  "level": "DEFAULT",
  "parentObservationId": null,
  "version": "1.0.0",
  "createdAt": "2024-01-15T10:30:01Z",
  "updatedAt": "2024-01-15T10:30:01Z"
}
```

## 🎯 评估 API

### 创建评估
```http
POST /api/public/evaluations
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "traceId": "trace-789",
  "name": "accuracy",
  "value": 0.95,
  "source": "auto",
  "comment": "High accuracy based on ground truth",
  "metadata": {
    "evaluator": "rule-based",
    "threshold": 0.8
  }
}
```

### 批量创建评估
```http
POST /api/public/evaluations/batch
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "evaluations": [
    {
      "traceId": "trace-789",
      "name": "accuracy",
      "value": 0.95
    },
    {
      "traceId": "trace-789",
      "name": "relevance",
      "value": 0.88
    }
  ]
}
```

### 获取评估
```http
GET /api/public/evaluations
Authorization: Bearer YOUR_API_KEY

# 查询参数
?traceId=trace-789         # 按追踪ID过滤
?name=accuracy             # 按评估名称过滤
?startDate=2024-01-01      # 开始日期
?endDate=2024-01-31        # 结束日期
?page=1                    # 页码
?limit=50                  # 每页数量
```

## 📝 提示 API

### 创建提示
```http
POST /api/public/prompts
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "name": "customer_support",
  "prompt": "You are a helpful customer support assistant. Respond to the user's question: {{question}}",
  "config": {
    "model": "gpt-4",
    "temperature": 0.7,
    "maxTokens": 500
  },
  "labels": ["support", "general"],
  "version": "1.0.0",
  "isActive": true
}
```

### 获取提示
```http
GET /api/public/prompts/{promptName}
Authorization: Bearer YOUR_API_KEY

# 查询参数
?version=1.0.0             # 指定版本（可选）
?label=support             # 按标签过滤（可选）
```

**响应**:
```json
{
  "id": "prompt-123",
  "name": "customer_support",
  "prompt": "You are a helpful customer support assistant. Respond to the user's question: {{question}}",
  "config": {
    "model": "gpt-4",
    "temperature": 0.7,
    "maxTokens": 500
  },
  "labels": ["support", "general"],
  "version": "1.0.0",
  "isActive": true,
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z"
}
```

### 获取提示版本列表
```http
GET /api/public/prompts/{promptName}/versions
Authorization: Bearer YOUR_API_KEY
```

## 📈 指标和统计 API

### 获取追踪统计
```http
GET /api/public/metrics/traces
Authorization: Bearer YOUR_API_KEY

# 查询参数
?startDate=2024-01-01      # 开始日期
?endDate=2024-01-31        # 结束日期
?granularity=day           # 粒度：hour, day, week, month
?userId=user-456           # 按用户过滤（可选）
?sessionId=session-123     # 按会话过滤（可选）
```

**响应**:
```json
{
  "totalTraces": 1250,
  "totalObservations": 8500,
  "totalTokens": 1250000,
  "averageDuration": 45.2,
  "timeSeries": [
    {
      "date": "2024-01-01",
      "traces": 42,
      "observations": 280,
      "tokens": 42000,
      "averageDuration": 43.5
    },
    {
      "date": "2024-01-02",
      "traces": 38,
      "observations": 255,
      "tokens": 38250,
      "averageDuration": 46.8
    }
  ]
}
```

### 获取评估统计
```http
GET /api/public/metrics/evaluations
Authorization: Bearer YOUR_API_KEY

# 查询参数
?startDate=2024-01-01
?endDate=2024-01-31
?name=accuracy             # 评估名称（可选）
```

**响应**:
```json
{
  "totalEvaluations": 850,
  "averageScore": 0.87,
  "scoreDistribution": {
    "0-0.2": 15,
    "0.2-0.4": 42,
    "0.4-0.6": 128,
    "0.6-0.8": 285,
    "0.8-1.0": 380
  },
  "timeSeries": [
    {
      "date": "2024-01-01",
      "count": 28,
      "averageScore": 0.85
    },
    {
      "date": "2024-01-02",
      "count": 32,
      "averageScore": 0.88
    }
  ]
}
```

## 🔄 Webhook API

### 创建 Webhook
```http
POST /api/public/webhooks
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "name": "Slack Notifications",
  "url": "https://hooks.slack.com/services/...",
  "events": ["trace.created", "evaluation.created"],
  "secret": "your_webhook_secret",
  "isActive": true,
  "metadata": {
    "channel": "#alerts",
    "team": "engineering"
  }
}
```

### Webhook 事件类型
```typescript
type WebhookEvent =
  | 'trace.created'
  | 'trace.updated'
  | 'observation.created'
  | 'observation.updated'
  | 'evaluation.created'
  | 'evaluation.updated'
  | 'prompt.created'
  | 'prompt.updated'
  | 'error.occurred';
```

### Webhook 请求示例
```json
{
  "event": "trace.created",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "id": "trace-789",
    "name": "User Authentication",
    "userId": "user-456",
    "sessionId": "session-123",
    "createdAt": "2024-01-15T10:30:00Z"
  },
  "signature": "sha256=..."
}
```

## 🛡️ 错误处理

### 错误响应格式
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input parameters",
    "details": [
      {
        "field": "name",
        "message": "Name is required"
      },
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req-123456789"
  }
}
```

### 常见错误码
| 错误码 | HTTP 状态码 | 说明 |
|--------|-------------|------|
| `VALIDATION_ERROR` | 400 | 请求参数验证失败 |
| `AUTHENTICATION_ERROR` | 401 | 认证失败 |
| `AUTHORIZATION_ERROR` | 403 | 权限不足 |
| `RESOURCE_NOT_FOUND` | 404 | 资源不存在 |
| `RATE_LIMIT_EXCEEDED` | 429 | 超过速率限制 |
| `INTERNAL_SERVER_ERROR` | 500 | 服务器内部错误 |
| `SERVICE_UNAVAILABLE` | 503 | 服务暂时不可用 |

## 🚀 最佳实践

### 1. 请求重试
```typescript
async function makeRequestWithRetry(
  url: string,
  options: RequestInit,
  maxRetries = 3
) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      
      if (response.status === 429) {
        // 速率限制，等待后重试
        const retryAfter = response.headers.get('Retry-After');
        const waitTime = retryAfter ? parseInt(retryAfter) * 1000 : 1000 * attempt;
        await new Promise(resolve => setTimeout(resolve, waitTime));
        continue;
      }
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return await response.json();
    } catch (error) {
      if (attempt === maxRetries) {
        throw error;
      }
      
      // 指数退避
      const waitTime = 1000 * Math.pow(2, attempt - 1);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
}
```

### 2. 批量操作
```typescript
// 批量创建追踪
async function batchCreateTraces(traces: Trace[]) {
  const BATCH_SIZE = 100;
  const results = [];
  
  for (let i = 0; i < traces.length; i += BATCH_SIZE) {
    const batch = traces.slice(i, i + BATCH_SIZE);
    const response = await fetch('/api/public/traces/batch', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey}`
      },
      body: JSON.stringify({ traces: batch })
    });
    
    if (!response.ok) {
      throw new Error(`Batch ${i / BATCH_SIZE + 1} failed`);
    }
    
    const batchResult = await response.json();
    results.push(...batchResult.data);
  }
  
  return results;
}
```

### 3. 监控和告警
```typescript
// 监控 API 健康状态
async function monitorApiHealth() {
  const healthCheck = await fetch('/api/public/health');
  
  if (!healthCheck.ok) {
    // 发送告警
    await sendAlert({
      severity: 'critical',
      message: 'Langfuse API is down',
      timestamp: new Date().toISOString()
    });
    
    return false;
  }
  
  const healthData = await healthCheck.json();
  
  // 检查关键指标
  if (healthData.database.status !== 'healthy') {
    await sendAlert({
      severity: 'warning',
      message: 'Database connectivity issues',
      timestamp: new Date().toISOString()
    });
  }
  
  return true;
}
```

### 4. 数据验证
```typescript
// 使用 Zod 验证 API 响应
import { z } from 'zod';

const TraceSchema = z.object({
  id: z.string(),
  name: z.string(),
  timestamp: z.string().datetime(),
  userId: z.string().optional(),
  sessionId: z.string().optional(),
  tags: z.array(z.string()),
  createdAt: z.string().datetime(),
  updatedAt: z.string().datetime()
});

async function fetchTraces() {
  const response = await fetch('/api/public/traces');
  const data = await response.json();
  
  // 验证响应数据
  const validatedData = TraceSchema.array().parse(data.data);
  
  return validatedData;
}
```

## 📚 SDK 和客户端库

### 官方 SDK
- **JavaScript/TypeScript**: `@langfuse/node`
- **Python**: `langfuse`
- **Go**: `github.com/langfuse/langfuse-go`
- **Java**: `com.langfuse:langfuse-java`

### JavaScript SDK 示例
```javascript
import { Langfuse } from '@langfuse/node';

const langfuse = new Langfuse({
  publicKey: 'pk-lf-...',
  secretKey: 'sk-lf-...',
  baseUrl: 'https://your-domain.com'
});

// 创建追踪
const trace = await langfuse.trace({
  name: 'User Authentication',
  userId: 'user-456',
  sessionId: 'session-123',
  metadata: { environment: 'production' }
});

// 创建观察点
const span = trace.span({
  name: 'Database Query',
  input: { query: 'SELECT * FROM users' },
  metadata: { database: 'postgres' }
});

// 完成观察点
span.end({ output: { rowCount: 1 } });

// 创建评估
trace.score({
  name: 'accuracy',
  value: 0.95
});
```

### Python SDK 示例
```python
from langfuse import Langfuse

langfuse = Langfuse(
    public_key="pk-lf-...",
    secret_key="sk-lf-...",
    host="https://your-domain.com"
)

# 创建追踪
trace = langfuse.trace(
    name="User Authentication",
    user_id="user-456",
    session_id="session-123",
    metadata={"environment": "production"}
)

# 创建观察点
span = trace.span(
    name="Database Query",
    input={"query": "SELECT * FROM users"},
    metadata={"database": "postgres"}
)

# 完成观察点
span.end(output={"rowCount": 1})

# 创建评估
trace.score(
    name="accuracy",
    value=0.95
)
```

## 🔍 调试和故障排除

### 启用调试日志
```bash
# 设置环境变量
export LANGFUSE_DEBUG=true
export NODE_DEBUG=http,net

# 或通过 SDK 配置
const langfuse = new Langfuse({
  debug: true,
  // ... 其他配置
});
```

### 检查请求和响应
```typescript
// 拦截和记录请求
const originalFetch = global.fetch;
global.fetch = async function(...args) {
  console.log('Request:', args);
  const response = await originalFetch.apply(this, args);
  console.log('Response status:', response.status);
  console.log('Response headers:', Object.fromEntries(response.headers.entries()));
  
  const clone = response.clone();
  const body = await clone.text();
  console.log('Response body:', body);
  
  return response;
};
```

### 常见问题解决
1. **认证失败**: 检查 API Key 是否有效，是否有必要的权限
2. **速率限制**: 实现指数退避重试机制
3. **网络问题**: 检查网络连接，代理配置
4. **数据格式错误**: 验证请求体是否符合 API 规范
5. **服务器错误**: 检查 Langfuse 服务状态，查看服务器日志

## 📞 支持和资源

### 获取帮助
- **文档**: https://langfuse.com/docs
- **API 参考**: https://langfuse.com/docs/api
- **社区**: https://langfuse.com/discord
- **GitHub**: https://github.com/langfuse/langfuse
- **支持邮箱**: support@langfuse.com

### 更新日志
- **API 变更**: 查看 GitHub Releases
- **SDK 更新**: 查看各 SDK 包的更新日志
- **功能公告**: 关注 Langfuse Blog

### 反馈和贡献
- **报告问题**: GitHub Issues
- **功能请求**: GitHub Discussions
- **贡献代码**: 查看 CONTRIBUTING.md

---

**API 健康检查**:
```bash
# 检查 API 状态
curl -f https://your-domain.com/api/public/health || echo "API 不可用"

# 检查认证
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://your-domain.com/api/public/traces?limit=1 || echo "认证失败"

# 检查速率限制
curl -I -H "Authorization: Bearer YOUR_API_KEY" \
  https://your-domain.com/api/public/traces
# 查看 X-RateLimit-* 头信息
```

如果所有检查通过，您的 API 集成已准备就绪！🚀