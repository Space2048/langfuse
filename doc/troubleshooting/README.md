# 故障排除指南

本指南旨在帮助您解决在使用、开发或部署 Langfuse 过程中可能遇到的常见问题。

## 目录

1. [安装与配置问题](#安装与配置问题)
2. [开发环境问题](#开发环境问题)
3. [部署问题](#部署问题)
4. [API 与集成问题](#api-与集成问题)
5. [性能问题](#性能问题)
6. [数据与存储问题](#数据与存储问题)
7. [安全问题](#安全问题)
8. [日志与调试](#日志与调试)
9. [常见错误码](#常见错误码)
10. [获取支持](#获取支持)

## 安装与配置问题

### Node.js 版本不兼容

**问题**：安装依赖时出现错误，提示 Node.js 版本不兼容。

**解决方案**：
1. 检查您的 Node.js 版本：`node -v`
2. Langfuse 要求 Node.js 24 或更高版本
3. 如果版本过低，可以使用 nvm 安装正确的版本：
   ```bash
   nvm install 24
   nvm use 24
   ```
4. 重新安装依赖：`pnpm install`

### 依赖安装失败

**问题**：`pnpm install` 失败，出现依赖冲突或网络问题。

**解决方案**：
1. 检查网络连接，确保可以访问 npm 注册表
2. 清除 pnpm 缓存：`pnpm store prune`
3. 删除 `node_modules` 目录和 `pnpm-lock.yaml` 文件
4. 重新运行：`pnpm install`
5. 如果问题仍然存在，尝试使用不同的网络环境或镜像源

### 环境变量配置错误

**问题**：应用无法启动，提示缺少必要的环境变量。

**解决方案**：
1. 复制示例环境变量文件：`cp .env.example .env`
2. 根据您的环境配置所有必要的环境变量
3. 确保 `.env` 文件位于项目根目录
4. 重启应用

**关键环境变量**：
- `DATABASE_URL`: PostgreSQL 数据库连接字符串
- `CLICKHOUSE_URL`: ClickHouse 数据库连接字符串
- `REDIS_URL`: Redis 连接字符串
- `NEXTAUTH_SECRET`: NextAuth 密钥
- `NEXTAUTH_URL`: NextAuth URL

## 开发环境问题

### 开发服务器无法启动

**问题**：`pnpm dev` 命令失败，服务器无法启动。

**解决方案**：
1. 检查端口是否被占用：`lsof -i :3000`（Linux/Mac）或 `netstat -ano | findstr :3000`（Windows）
2. 确保数据库服务正在运行：PostgreSQL、ClickHouse、Redis
3. 运行数据库迁移：`pnpm db:migrate`
4. 检查日志输出，查找具体错误信息

### 数据库连接错误

**问题**：应用无法连接到数据库。

**解决方案**：
1. 检查数据库服务是否正在运行
2. 验证数据库连接字符串是否正确
3. 确保数据库用户具有正确的权限
4. 检查防火墙设置，确保允许应用连接到数据库

### 热重载不工作

**问题**：修改代码后，开发服务器没有自动重载。

**解决方案**：
1. 检查是否有语法错误或编译错误
2. 确保 `next.config.mjs` 中的配置正确
3. 尝试重启开发服务器
4. 检查文件系统权限，确保 Node.js 可以读取文件变更

## 部署问题

### Docker Compose 部署失败

**问题**：`docker-compose up` 命令失败，容器无法启动。

**解决方案**：
1. 检查 Docker 是否正在运行
2. 确保所有环境变量都已正确配置（特别是标记为 `# CHANGEME` 的变量）
3. 检查端口是否被占用
4. 查看容器日志：`docker-compose logs -f`
5. 尝试重新构建镜像：`docker-compose build`

### Kubernetes 部署问题

**问题**：Kubernetes 部署后，Pod 无法正常运行。

**解决方案**：
1. 检查 Pod 状态：`kubectl get pods`
2. 查看 Pod 日志：`kubectl logs <pod-name>`
3. 检查 Deployment 状态：`kubectl describe deployment <deployment-name>`
4. 确保所有必要的 ConfigMap 和 Secret 都已正确配置
5. 检查网络策略和服务配置

### 负载均衡器配置错误

**问题**：应用无法通过负载均衡器访问。

**解决方案**：
1. 检查负载均衡器配置是否正确
2. 确保健康检查路径返回 200 状态码
3. 检查防火墙设置，确保允许流量通过
4. 验证 SSL/TLS 证书是否有效

## API 与集成问题

### API 密钥无效

**问题**：使用 API 密钥调用 Langfuse API 时，收到 "Invalid API key" 错误。

**解决方案**：
1. 检查 API 密钥是否正确（区分大小写）
2. 确保 API 密钥没有过期或被撤销
3. 验证项目权限，确保密钥对请求的项目有访问权限
4. 检查 API 密钥的类型（公钥/密钥）是否与 API 端点匹配

### SDK 集成错误

**问题**：在应用中集成 Langfuse SDK 时出现错误。

**解决方案**：
1. 确保 SDK 版本与 Langfuse 服务器版本兼容
2. 检查 SDK 初始化配置是否正确
3. 验证 API 密钥和基础 URL 是否正确
4. 查看 SDK 日志，获取更详细的错误信息

### LLM 平台集成问题

**问题**：与 OpenAI、Anthropic 等 LLM 平台的集成不工作。

**解决方案**：
1. 检查 LLM 平台的 API 密钥是否正确
2. 确保 LLM 平台的 API 端点可以访问
3. 验证 LLM 平台的 API 限制是否已达到
4. 检查 Langfuse LLM 集成的配置是否正确

## 性能问题

### 应用响应缓慢

**问题**：Langfuse 应用响应缓慢或卡顿。

**解决方案**：
1. 检查服务器资源使用情况（CPU、内存、磁盘）
2. 优化数据库查询，添加必要的索引
3. 检查 Redis 连接，确保缓存正常工作
4. 增加服务器资源或使用负载均衡器扩展

### 数据库性能问题

**问题**：数据库查询缓慢，影响应用性能。

**解决方案**：
1. 监控数据库性能指标（查询时间、连接数、缓存命中率）
2. 优化慢查询，添加适当的索引
3. 考虑数据库分片或垂直扩展
4. 定期清理旧数据，维护数据库

### 高流量下的性能问题

**问题**：在高流量情况下，应用性能下降。

**解决方案**：
1. 使用负载均衡器分发流量
2. 增加应用服务器实例
3. 优化缓存策略，减少数据库查询
4. 考虑使用 CDN 加速静态资源

## 数据与存储问题

### 数据丢失

**问题**：部分数据丢失或无法查询。

**解决方案**：
1. 检查数据库备份策略，确保定期备份
2. 验证数据迁移过程是否正确
3. 检查应用日志，查找数据操作错误
4. 从最近的备份恢复数据（如果需要）

### 存储容量不足

**问题**：数据库或存储容量不足。

**解决方案**：
1. 监控存储使用情况，设置容量警报
2. 清理不需要的数据或归档旧数据
3. 扩展存储容量
4. 优化数据存储方式，减少存储占用

### 数据同步问题

**问题**：PostgreSQL 和 ClickHouse 数据不同步。

**解决方案**：
1. 检查数据同步服务是否正在运行
2. 查看同步日志，查找错误信息
3. 重新启动同步服务
4. 手动触发数据同步

## 安全问题

### 未授权访问

**问题**：检测到未授权访问尝试或数据泄露。

**解决方案**：
1. 立即撤销可疑的 API 密钥
2. 检查应用日志，识别攻击来源
3. 更新密码和密钥
4. 审查安全配置，确保遵循最佳实践
5. 考虑实施额外的安全措施（如 WAF、IP 限制）

### SSL/TLS 证书问题

**问题**：SSL/TLS 证书过期或无效，导致安全警告。

**解决方案**：
1. 检查证书有效期
2. 重新生成或更新证书
3. 确保证书链完整
4. 配置自动证书续订（如使用 Let's Encrypt）

### 敏感数据暴露

**问题**：敏感数据可能被意外暴露。

**解决方案**：
1. 审查代码，确保敏感数据不被记录或暴露
2. 实施数据掩码或加密
3. 配置适当的访问控制
4. 遵循数据最小化原则

## 日志与调试

### 查看应用日志

**开发环境**：
```bash
# 查看所有服务日志
pnpm dev

# 查看特定服务日志
pnpm dev:web  # Web 服务
pnpm dev:worker  # Worker 服务
```

**生产环境（Docker）**：
```bash
# 查看所有容器日志
docker-compose logs -f

# 查看特定容器日志
docker-compose logs -f langfuse-web
docker-compose logs -f langfuse-worker
docker-compose logs -f postgres
docker-compose logs -f clickhouse
docker-compose logs -f redis
```

**生产环境（Kubernetes）**：
```bash
# 查看 Pod 日志
kubectl logs <pod-name>

# 持续查看日志
kubectl logs -f <pod-name>

# 查看所有 Pod 日志
kubectl logs -f -l app=langfuse
```

### 启用调试日志

**开发环境**：
1. 在 `.env` 文件中设置：`DEBUG=true`
2. 重启开发服务器

**生产环境**：
1. 在环境变量中设置：`DEBUG=true`
2. 重启应用服务

### 使用调试工具

**Node.js 调试**：
```bash
# 使用 Node.js 调试器
pnpm dev:debug

# 使用 Chrome DevTools
pnpm dev:inspect
```

**数据库调试**：
```bash
# 连接到 PostgreSQL
docker-compose exec postgres psql -U langfuse -d langfuse

# 连接到 ClickHouse
docker-compose exec clickhouse clickhouse-client

# 连接到 Redis
docker-compose exec redis redis-cli
```

## 常见错误码

### 400 Bad Request

**原因**：请求参数错误或格式不正确。

**解决方案**：
1. 检查请求参数是否符合 API 文档要求
2. 验证请求体格式是否正确（如 JSON 格式）
3. 确保所有必填参数都已提供

### 401 Unauthorized

**原因**：未提供有效的认证信息。

**解决方案**：
1. 检查 API 密钥是否正确
2. 确保认证头格式正确
3. 验证 API 密钥是否有权限访问请求的资源

### 403 Forbidden

**原因**：认证成功，但没有足够的权限访问资源。

**解决方案**：
1. 检查用户或 API 密钥的权限设置
2. 确保请求的资源属于正确的项目
3. 联系项目管理员获取必要的权限

### 404 Not Found

**原因**：请求的资源不存在。

**解决方案**：
1. 检查 API 端点 URL 是否正确
2. 验证资源 ID 是否存在
3. 确保请求的资源属于正确的项目

### 429 Too Many Requests

**原因**：API 请求频率超过限制。

**解决方案**：
1. 减少请求频率，实现请求节流
2. 考虑使用批量操作减少请求次数
3. 联系支持团队增加 API 限制

### 500 Internal Server Error

**原因**：服务器内部错误。

**解决方案**：
1. 检查应用日志，查找具体错误信息
2. 验证服务器资源是否充足
3. 确保所有依赖服务都正常运行
4. 如果问题持续存在，联系支持团队

## 获取支持

### 社区支持

- **GitHub Discussions**：[Langfuse Community](https://github.com/langfuse/langfuse/discussions)
- **Discord**：[Langfuse Discord Server](https://discord.gg/your-discord-link)
- **Stack Overflow**：使用 `langfuse` 标签提问

### 企业支持

如果您使用的是 Langfuse 企业版，可以联系我们的企业支持团队：

- 邮件：[enterprise-support@langfuse.com](mailto:enterprise-support@langfuse.com)
- 电话：+1 (555) 123-4567
- 专属 Slack 频道（企业客户）

### 报告问题

如果您发现了 bug 或问题，可以在 GitHub 上提交 issue：

1. 访问 [Langfuse GitHub Repository](https://github.com/langfuse/langfuse)
2. 点击 "Issues" 选项卡
3. 点击 "New Issue"
4. 选择适当的模板
5. 填写详细的问题描述，包括：
   - 问题重现步骤
   - 预期行为
   - 实际行为
   - 错误日志
   - 环境信息（版本、操作系统等）

### 提供反馈

我们欢迎您的反馈和建议，帮助我们改进 Langfuse：

- 功能建议：[Feature Requests](https://github.com/langfuse/langfuse/discussions/categories/feature-requests)
- 用户反馈：[User Feedback](https://github.com/langfuse/langfuse/discussions/categories/user-feedback)
- 文档改进：[Documentation Issues](https://github.com/langfuse/langfuse/issues?q=is%3Aissue+is%3Aopen+label%3Adocumentation)

## 最佳实践

### 预防问题

1. **定期备份**：定期备份数据库和配置文件
2. **监控系统**：设置监控和警报，及时发现问题
3. **更新软件**：定期更新 Langfuse 和依赖软件到最新版本
4. **测试变更**：在测试环境中测试所有变更，然后再部署到生产环境
5. **文档化**：记录系统配置、架构和流程

### 问题解决流程

1. **识别问题**：准确定义问题，包括症状和影响
2. **收集信息**：收集日志、错误信息和环境数据
3. **分析问题**：根据收集的信息，分析可能的原因
4. **测试解决方案**：在测试环境中测试可能的解决方案
5. **实施修复**：在生产环境中实施经过测试的解决方案
6. **验证修复**：验证问题是否已解决
7. **记录问题**：记录问题和解决方案，以便将来参考

## 总结

通过遵循本指南，您应该能够解决在使用、开发或部署 Langfuse 过程中遇到的大多数常见问题。如果您遇到了本指南未涵盖的问题，或者问题无法解决，请不要犹豫，通过上述渠道联系我们的支持团队或社区。

我们致力于为您提供最好的产品和支持体验！🚀