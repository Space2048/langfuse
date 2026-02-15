# 开发工作流指南

本文档详细介绍了 Langfuse 项目的开发工作流程，包括代码管理、分支策略、代码审查和发布流程。

## 📋 工作流概览

### 开发周期
```
构思 → 开发 → 测试 → 审查 → 合并 → 部署
```

### 工具链
- **版本控制**: Git + GitHub
- **项目管理**: GitHub Issues + Projects
- **CI/CD**: GitHub Actions
- **代码质量**: ESLint + Prettier + TypeScript
- **测试**: Jest + Playwright
- **部署**: Docker + Kubernetes

## 🌿 Git 分支策略

### 分支类型

#### 1. 主分支
- **`main`**: 生产就绪代码，受保护分支
- **`develop`**: 开发集成分支，功能合并目标

#### 2. 支持分支
- **功能分支**: `feature/*`
- **修复分支**: `fix/*`
- **发布分支**: `release/*`
- **热修复分支**: `hotfix/*`

### 分支命名规范
```bash
# 功能分支
feature/add-oauth-support
feature/1234-improve-tracing-ui

# 修复分支
fix/5678-memory-leak
fix/typo-in-docs

# 发布分支
release/v2.1.0

# 热修复分支
hotfix/critical-security-issue
```

## 🚀 开发流程

### 1. 准备阶段

#### 选择 Issue
```bash
# 查看可用的 Issue
# 优先选择带有以下标签的：
# - good-first-issue
# - help-wanted
# - bug
```

#### 创建分支
```bash
# 从 develop 分支创建新分支
git checkout develop
git pull origin develop
git checkout -b feature/your-feature-name

# 或使用 GitHub CLI
gh issue develop 1234 --name feature/issue-1234
```

### 2. 开发阶段

#### 代码编写
```bash
# 启动开发服务器
pnpm run dev

# 运行相关测试
pnpm run test -- --testPathPattern=your-feature

# 代码检查
pnpm run lint
pnpm run typecheck
```

#### 提交规范
使用 [Conventional Commits](https://www.conventionalcommits.org/) 规范：

```bash
# 提交类型
feat:     新功能
fix:      修复bug
docs:     文档更新
style:    代码格式（不影响功能）
refactor: 代码重构
test:     测试相关
chore:    构建过程或辅助工具
perf:     性能优化
ci:       CI/CD 相关
build:    构建系统
revert:   回滚提交

# 示例
git commit -m "feat: add OAuth2 authentication support"
git commit -m "fix: resolve memory leak in trace processing"
git commit -m "docs: update API documentation for v2"
```

### 3. 测试阶段

#### 本地测试
```bash
# 运行所有测试
pnpm run test

# 运行特定测试
pnpm run test -- --testNamePattern="authentication"

# 运行 E2E 测试
pnpm run test:e2e

# 测试覆盖率
pnpm run test:coverage
```

#### 测试要求
- 新功能必须包含单元测试
- API 变更必须包含集成测试
- UI 变更必须包含 E2E 测试
- 测试覆盖率不低于 80%

### 4. 代码审查阶段

#### 创建 Pull Request
```bash
# 推送分支
git push origin feature/your-feature-name

# 创建 PR
gh pr create --title "feat: add new feature" --body "## Description\n\n详细描述..."
```

#### PR 模板要求
每个 PR 必须包含：
1. **标题**: 遵循 Conventional Commits
2. **描述**: 功能说明、变更原因
3. **测试**: 测试覆盖情况
4. **检查清单**: 完成的项目
5. **相关 Issue**: 链接到 Issue

#### 审查流程
1. **自动检查**: CI 运行测试和 lint
2. **人工审查**: 至少 1 名核心成员批准
3. **变更请求**: 根据反馈修改代码
4. **重新审查**: 修改后重新请求审查

### 5. 合并阶段

#### 合并前检查
```bash
# 确保分支最新
git checkout develop
git pull origin develop
git checkout feature/your-feature-name
git rebase develop

# 解决冲突（如果有）
git rebase --continue

# 运行最终测试
pnpm run test
pnpm run build
```

#### 合并策略
```bash
# 使用 squash merge（推荐）
git merge --squash feature/your-feature-name
git commit -m "feat: add new feature (#123)"

# 或使用 GitHub UI 的 "Squash and merge"
```

## 🔄 持续集成流程

### GitHub Actions 工作流

#### 1. PR 检查工作流
触发条件：`pull_request`
检查项目：
- 代码格式检查
- 类型检查
- 单元测试
- 集成测试
- 构建检查

#### 2. 主分支工作流
触发条件：`push` 到 `main` 或 `develop`
执行项目：
- 完整测试套件
- Docker 镜像构建
- 部署到测试环境

#### 3. 发布工作流
触发条件：`release` 事件
执行项目：
- 版本号更新
- 变更日志生成
- Docker 镜像推送
- 部署到生产环境

## 📦 发布流程

### 版本管理
使用 [Semantic Versioning](https://semver.org/)：
- **主版本号**: 不兼容的 API 变更
- **次版本号**: 向后兼容的功能性新增
- **修订号**: 向后兼容的问题修正

### 发布周期
1. **功能冻结**: 停止新功能开发
2. **测试阶段**: 全面测试
3. **预发布**: 发布候选版本
4. **正式发布**: 发布稳定版本
5. **维护阶段**: 修复和更新

### 发布命令
```bash
# 使用 release-it 工具
pnpm run release

# 或手动发布
npm version patch  # 修订号
npm version minor  # 次版本号
npm version major  # 主版本号
```

## 🧪 测试策略

### 测试金字塔
```
        E2E 测试
          / \
         /   \
        /     \
  集成测试   集成测试
      \       /
       \     /
        \   /
      单元测试
```

### 测试类型

#### 1. 单元测试
```bash
# 运行单元测试
pnpm run test:unit

# 测试位置
packages/*/src/__tests__/
web/src/__tests__/
worker/src/__tests__/
```

#### 2. 集成测试
```bash
# 运行集成测试
pnpm run test:integration

# 测试位置
packages/*/src/__tests__/integration/
```

#### 3. E2E 测试
```bash
# 运行 E2E 测试
pnpm run test:e2e

# 测试位置
e2e/tests/
```

### 测试最佳实践
1. **隔离性**: 测试之间不依赖
2. **确定性**: 相同输入产生相同输出
3. **快速性**: 测试运行速度快
4. **可读性**: 测试名称清晰表达意图
5. **覆盖率**: 关键路径必须覆盖

## 🔍 代码质量

### 代码检查
```bash
# 运行所有检查
pnpm run lint
pnpm run typecheck
pnpm run format:check

# 自动修复
pnpm run lint:fix
pnpm run format
```

### 代码审查清单
- [ ] 代码遵循项目规范
- [ ] 添加了适当的测试
- [ ] 更新了相关文档
- [ ] 考虑了向后兼容性
- [ ] 性能影响评估
- [ ] 安全考虑
- [ ] 错误处理完善

## 🚨 紧急修复流程

### 热修复流程
1. **创建热修复分支**: `hotfix/critical-issue`
2. **快速修复**: 最小化变更
3. **紧急测试**: 重点测试修复部分
4. **快速审查**: 核心成员快速审查
5. **合并到 main**: 直接合并
6. **部署**: 立即部署到生产
7. **同步到 develop**: 后续同步

### 回滚流程
```bash
# 识别问题提交
git log --oneline -10

# 创建回滚提交
git revert <commit-hash>

# 或重置到安全状态
git reset --hard <safe-commit>
```

## 📊 监控和反馈

### 开发指标
- **构建时间**: 监控构建性能
- **测试时间**: 优化测试速度
- **代码覆盖率**: 保持高质量
- **PR 合并时间**: 优化审查流程

### 反馈循环
1. **用户反馈**: GitHub Issues, Discord
2. **性能监控**: 生产环境监控
3. **错误追踪**: Sentry 集成
4. **使用分析**: 功能使用统计

## 🎯 最佳实践总结

### 开发效率
- 使用 VS Code 开发容器
- 利用 Turbo 的缓存机制
- 编写可重用的组件和工具

### 代码质量
- 遵循 TypeScript 严格模式
- 使用 ESLint 和 Prettier
- 编写自解释的代码

### 团队协作
- 及时沟通进度和问题
- 积极参与代码审查
- 分享知识和经验

### 持续改进
- 定期回顾开发流程
- 收集团队反馈
- 优化工具和流程

## 📚 相关资源

### 内部文档
- [代码规范](./code-style.md)
- [测试指南](./testing.md)
- [部署指南](../deployment/README.md)

### 外部资源
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)

### 工具文档
- [Turbo Repo](https://turbo.build/repo)
- [pnpm](https://pnpm.io/)
- [Jest](https://jestjs.io/)
- [Playwright](https://playwright.dev/)

---

**工作流检查清单**：
```bash
# 开发前
[ ] 选择 Issue 并分配
[ ] 创建功能分支
[ ] 理解需求和技术方案

# 开发中
[ ] 编写代码和测试
[ ] 运行本地检查
[ ] 提交符合规范的提交

# 提交前
[ ] 更新相关文档
[ ] 确保测试通过
[ ] 代码审查自查

# 合并后
[ ] 删除功能分支
[ ] 更新 Issue 状态
[ ] 验证部署结果
```

遵循这个工作流将确保代码质量、团队协作效率和项目的可持续发展。🚀