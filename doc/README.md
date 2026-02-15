# Langfuse 项目文档

欢迎来到 Langfuse 项目文档！Langfuse 是一个开源的 LLM 工程平台，帮助团队协作开发、监控、评估和调试 AI 应用程序。

## 📚 文档导航

### 项目概述
- [项目介绍](./overview/README.md) - Langfuse 的核心功能和价值主张
- [核心特性](./overview/features.md) - 详细的功能说明
- [技术架构](./architecture/README.md) - 系统架构和技术栈

### 开发指南
- [环境搭建](./development/setup.md) - 本地开发环境配置
- [开发工作流](./development/workflow.md) - 开发流程和最佳实践
- [代码规范](./development/code-style.md) - 编码标准和约定
- [测试指南](./development/testing.md) - 测试策略和执行

### 架构设计
- [系统架构](./architecture/system-architecture.md) - 整体架构设计
- [数据库设计](./architecture/database.md) - 数据库架构和设计
- [API 设计](./architecture/api-design.md) - API 架构和规范
- [部署架构](./architecture/deployment.md) - 部署架构和选项

### 部署文档
- [本地部署](./deployment/local.md) - Docker Compose 本地部署
- [生产部署](./deployment/production.md) - 生产环境部署指南
- [Kubernetes 部署](./deployment/kubernetes.md) - Helm Chart 部署
- [云平台部署](./deployment/cloud.md) - AWS/Azure/GCP 部署

### API 文档
- [API 概览](./api/overview.md) - API 设计和认证
- [公共 API](./api/public-api.md) - 公共 API 端点文档
- [管理 API](./api/admin-api.md) - 管理 API 端点文档
- [SDK 集成](./api/sdk-integration.md) - SDK 使用指南

### 运维指南
- [监控告警](./operations/monitoring.md) - 系统监控和告警配置
- [故障排查](./operations/troubleshooting.md) - 常见问题排查
- [性能优化](./operations/performance.md) - 性能调优指南
- [安全配置](./operations/security.md) - 安全最佳实践

### 贡献指南
- [贡献流程](./contributing/process.md) - 如何贡献代码
- [代码审查](./contributing/code-review.md) - 代码审查标准
- [发布流程](./contributing/release.md) - 版本发布流程

## 🚀 快速开始

### 本地开发
```bash
# 克隆项目
git clone https://github.com/langfuse/langfuse.git
cd langfuse

# 安装依赖
pnpm i

# 启动开发环境
pnpm run dev
```

### 快速部署
```bash
# 使用 Docker Compose 部署
docker compose up
```

## 📞 支持与帮助

- **GitHub Issues**: [报告问题](https://github.com/langfuse/langfuse/issues)
- **Discord**: [加入社区](https://langfuse.com/discord)
- **文档网站**: [官方文档](https://langfuse.com/docs)

## 📄 许可证

Langfuse 采用 MIT 许可证。详见 [LICENSE](../LICENSE) 文件。