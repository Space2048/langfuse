# 开发环境搭建指南

本文档将指导您完成 Langfuse 开发环境的完整搭建过程。

## 🎯 环境要求

### 基础要求
- **操作系统**: Windows 10/11, macOS 10.15+, Linux (Ubuntu 20.04+)
- **内存**: 最低 8GB，推荐 16GB
- **磁盘空间**: 最低 10GB 可用空间

### 软件要求
- **Node.js**: 版本 24 (详见 `.nvmrc`)
- **包管理器**: pnpm v9.5.0
- **Docker**: Docker Desktop 或 Docker Engine
- **Git**: 版本控制工具
- **代码编辑器**: VS Code 推荐

## 🚀 快速开始

### 1. 克隆仓库
```bash
# 克隆主仓库
git clone https://github.com/langfuse/langfuse.git
cd langfuse

# 或使用 SSH
git clone git@github.com:langfuse/langfuse.git
cd langfuse
```

### 2. 安装 Node.js
```bash
# 使用 nvm (推荐)
nvm install 24
nvm use 24

# 或直接下载 Node.js 24
# 从 https://nodejs.org/ 下载安装
```

### 3. 安装 pnpm
```bash
# 使用 npm 安装 pnpm
npm install -g pnpm@9.5.0

# 验证安装
pnpm --version
# 应该显示 9.5.0
```

### 4. 安装 Docker
- **Windows/macOS**: 下载 [Docker Desktop](https://www.docker.com/products/docker-desktop)
- **Linux**: 使用包管理器安装 Docker Engine

验证 Docker 安装：
```bash
docker --version
docker compose version
```

## 🔧 完整环境配置

### 1. 安装项目依赖
```bash
# 在项目根目录执行
pnpm i
```

### 2. 配置环境变量
```bash
# 复制开发环境配置文件
cp .env.dev.example .env

# 编辑 .env 文件，根据需要修改配置
# 主要配置项：
# - 数据库连接信息
# - Redis 配置
# - ClickHouse 配置
# - 认证密钥
```

### 3. 启动基础设施服务
```bash
# 启动所有 Docker 服务
pnpm run infra:dev:up

# 服务包括：
# - PostgreSQL (端口 5432)
# - ClickHouse (端口 8123, 9000)
# - Redis (端口 6379)
# - MinIO (端口 9090, 9091)
```

### 4. 初始化数据库
```bash
# 生成 Prisma 客户端
pnpm run db:generate

# 运行数据库迁移
pnpm run db:migrate

# 重置数据库（开发环境）
pnpm run db:reset

# 种子示例数据
pnpm run db:seed:examples
```

### 5. 启动开发服务器
```bash
# 启动所有服务（Web + Worker）
pnpm run dev

# 或分别启动
pnpm run dev:web     # 只启动 Web 应用 (localhost:3000)
pnpm run dev:worker  # 只启动 Worker 服务
```

## 🎮 开发工作流

### 一键初始化脚本
```bash
# 完整初始化（会重置数据库和 node_modules）
pnpm run dx

# 快速初始化（强制重置）
pnpm run dx-f

# 跳过基础设施初始化
pnpm run dx:skip-infra
```

### 常用开发命令
```bash
# 类型检查
pnpm run typecheck
pnpm run tc  # 简写

# 代码检查
pnpm run lint
pnpm run lint:fix  # 自动修复

# 代码格式化
pnpm run format
pnpm run format:check  # 检查格式

# 运行测试
pnpm run test          # 异步测试
pnpm run test-sync     # 同步测试
pnpm run test-client   # 客户端测试
```

## 🐳 Docker 开发环境

### 使用 Dev Container
项目包含完整的 Dev Container 配置，支持：

1. **GitHub Codespaces**: 点击仓库的 "Code" → "Open with Codespaces"
2. **VS Code Dev Containers**: 安装 Remote - Containers 扩展

### Docker 服务管理
```bash
# 查看服务状态
docker compose -f ./docker-compose.dev.yml ps

# 停止服务
pnpm run infra:dev:down

# 清理服务（删除数据卷）
pnpm run infra:dev:prune

# 重启单个服务
docker compose -f ./docker-compose.dev.yml restart postgres
```

## 🔍 环境验证

### 验证服务运行
```bash
# 检查 Web 应用
curl http://localhost:3000/api/health

# 检查 PostgreSQL
docker exec -it langfuse-postgres-1 psql -U postgres -c "SELECT version();"

# 检查 Redis
docker exec -it langfuse-redis-1 redis-cli ping

# 检查 ClickHouse
curl http://localhost:8123/ping
```

### 验证开发环境
1. **访问 Web 界面**: http://localhost:3000
2. **默认登录账号**:
   - 邮箱: `demo@langfuse.com`
   - 密码: `password`
3. **检查功能**:
   - 创建新项目
   - 查看追踪数据
   - 测试提示管理

## 🛠️ 开发工具配置

### VS Code 推荐配置
1. **扩展推荐**:
   - ESLint
   - Prettier
   - TypeScript and JavaScript Language Features
   - Prisma
   - Docker
   - GitLens

2. **工作区设置**:
```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "typescript.preferences.importModuleSpecifier": "non-relative"
}
```

### 浏览器开发工具
- **React Developer Tools**: React 组件调试
- **Redux DevTools**: 状态管理调试
- **Network 面板**: API 请求监控
- **Console 面板**: 日志和错误查看

## 🔧 故障排除

### 常见问题

#### 1. 端口冲突
```bash
# 检查端口占用
netstat -ano | findstr :3000  # Windows
lsof -i :3000                 # macOS/Linux

# 修改端口（在 .env 文件中）
PORT=3001
```

#### 2. 数据库连接失败
```bash
# 检查 Docker 服务
docker compose -f ./docker-compose.dev.yml ps

# 重启数据库服务
docker compose -f ./docker-compose.dev.yml restart postgres

# 重置数据库
pnpm --filter=shared run db:reset
```

#### 3. 依赖安装失败
```bash
# 清理 node_modules 重新安装
rm -rf node_modules
rm -rf web/node_modules
rm -rf worker/node_modules
rm -rf packages/*/node_modules
pnpm i
```

#### 4. 内存不足
- 增加 Docker 内存分配（Docker Desktop 设置）
- 关闭不必要的服务
- 使用 `pnpm run dev:web` 只启动 Web 应用

### 获取帮助
- **查看日志**: `docker compose -f ./docker-compose.dev.yml logs`
- **社区支持**: [Langfuse Discord](https://langfuse.com/discord)
- **GitHub Issues**: 报告具体问题

## 📚 下一步

### 开始开发
1. **熟悉代码结构**: 查看 `CLAUDE.md` 了解项目结构
2. **运行示例**: 尝试修改示例代码
3. **编写测试**: 为新功能添加测试

### 深入学习
- 阅读 [开发工作流](./workflow.md) 了解开发流程
- 查看 [代码规范](./code-style.md) 遵循编码标准
- 学习 [测试指南](./testing.md) 编写高质量测试

### 参与贡献
- 查看 [贡献指南](../contributing/process.md)
- 选择 [Good First Issue](https://github.com/langfuse/langfuse/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)
- 提交 Pull Request

---

**环境状态检查**：
```bash
# 运行环境检查脚本
node -v          # 应该显示 v24.x.x
pnpm --version   # 应该显示 9.5.0
docker --version # 应该显示 Docker 版本
docker compose version # 应该显示 Compose 版本

# 验证服务运行
curl -f http://localhost:3000/api/health || echo "Web 服务未运行"
```

如果所有检查通过，您的开发环境已准备就绪！🎉