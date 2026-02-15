# 贡献流程指南

本文档详细介绍了如何为 Langfuse 项目做出贡献，包括代码贡献、文档改进、问题报告和社区参与。

## 🤝 贡献概览

### 贡献类型
| 贡献类型 | 适合人群 | 所需技能 | 时间投入 |
|---------|---------|----------|----------|
| **代码贡献** | 开发者 | 编程、Git | 中等 |
| **文档改进** | 技术写作者 | 写作、Markdown | 低 |
| **问题报告** | 所有用户 | 观察、描述 | 低 |
| **功能建议** | 资深用户 | 产品思维 | 低 |
| **代码审查** | 核心贡献者 | 代码评审 | 中等 |
| **社区支持** | 热心用户 | 沟通、技术 | 可变 |

### 行为准则
所有贡献者必须遵守我们的 [行为准则](CODE_OF_CONDUCT.md)，确保：
- 尊重所有社区成员
- 提供建设性反馈
- 保持专业和友善
- 包容不同背景和观点

## 🐛 报告问题

### 问题模板
```markdown
## 问题描述
清晰描述遇到的问题。

## 重现步骤
1. 第一步
2. 第二步
3. 第三步

## 期望行为
描述期望发生什么。

## 实际行为
描述实际发生了什么。

## 环境信息
- **Langfuse 版本**: [例如: 3.153.0]
- **部署方式**: [例如: Docker Compose, Kubernetes]
- **操作系统**: [例如: Ubuntu 22.04]
- **浏览器**: [例如: Chrome 120]
- **Node.js 版本**: [例如: 20.10.0]

## 附加信息
- 错误日志
- 截图
- 相关配置
```

### 问题分类
| 标签 | 说明 | 优先级 |
|------|------|--------|
| `bug` | 功能异常或错误 | 高 |
| `enhancement` | 功能改进建议 | 中 |
| `feature-request` | 新功能请求 | 中 |
| `documentation` | 文档问题 | 低 |
| `question` | 使用问题 | 低 |
| `good-first-issue` | 适合新贡献者 | 低 |

### 报告前检查
- [ ] 搜索现有问题，避免重复
- [ ] 检查文档和常见问题
- [ ] 尝试最新版本
- [ ] 提供完整的环境信息
- [ ] 包含可重现的示例

## 💡 提出功能建议

### 建议模板
```markdown
## 问题/机会
描述要解决的问题或机会。

## 建议方案
详细描述建议的功能或改进。

## 替代方案
描述考虑过的其他方案。

## 影响范围
- 受影响的组件
- 需要修改的 API
- 向后兼容性考虑

## 附加信息
- 用户场景示例
- 技术实现思路
- 相关参考资料
```

### 评估标准
1. **价值**: 解决多少用户问题
2. **复杂度**: 实现难度和资源需求
3. **一致性**: 是否符合项目愿景
4. **维护性**: 长期维护成本

## 🔧 代码贡献

### 准备工作
1. **熟悉项目**
   ```bash
   # 阅读文档
   # 运行示例
   # 了解代码结构
   ```

2. **设置开发环境**
   ```bash
   # 克隆仓库
   git clone https://github.com/langfuse/langfuse.git
   cd langfuse
   
   # 安装依赖
   pnpm i
   
   # 启动开发环境
   pnpm run dx
   ```

3. **选择任务**
   ```bash
   # 查看 Good First Issues
   # 选择感兴趣的问题
   # 在 Issue 中留言表示认领
   ```

### 开发流程

#### 1. 创建分支
```bash
# 同步主分支
git checkout develop
git pull origin develop

# 创建功能分支
git checkout -b feature/your-feature-name
# 或
git checkout -b fix/issue-number-description
```

#### 2. 实现功能
```bash
# 编写代码
# 遵循代码规范
# 添加必要的测试

# 运行测试
pnpm run test
pnpm run lint
pnpm run typecheck

# 提交代码
git add .
git commit -m "feat: add new feature"
```

#### 3. 保持同步
```bash
# 定期同步主分支
git fetch origin
git rebase origin/develop

# 解决冲突（如果有）
git rebase --continue
```

#### 4. 代码质量
```bash
# 运行完整检查
pnpm run test:all
pnpm run lint:fix
pnpm run format

# 检查测试覆盖率
pnpm run test:coverage
```

### 提交规范
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
git commit -m "feat(auth): add OAuth2 support"
git commit -m "fix(api): resolve memory leak in trace processing"
git commit -m "docs: update deployment guide"
```

### 创建 Pull Request

#### PR 模板
```markdown
## 变更描述
清晰描述本次 PR 的变更内容。

## 相关 Issue
链接到相关的 Issue，例如: Fixes #123

## 测试覆盖
- [ ] 添加了单元测试
- [ ] 添加了集成测试
- [ ] 添加了 E2E 测试
- [ ] 所有测试通过

## 检查清单
- [ ] 代码遵循项目规范
- [ ] 更新了相关文档
- [ ] 添加了变更日志条目
- [ ] 考虑了向后兼容性
- [ ] 性能影响已评估
- [ ] 安全考虑已处理

## 截图/录屏
如果适用，添加 UI 变更的截图。

## 附加信息
任何其他需要说明的信息。
```

#### PR 要求
- **标题**: 遵循 Conventional Commits 规范
- **描述**: 详细说明变更内容和原因
- **范围**: 每个 PR 专注于一个功能或修复
- **大小**: 建议不超过 500 行代码
- **测试**: 必须包含相关测试
- **文档**: 必须更新相关文档

## 📖 文档贡献

### 文档类型
| 文档类型 | 位置 | 贡献方式 |
|---------|------|----------|
| **API 文档** | `/docs/api` | 更新 OpenAPI 规范 |
| **用户指南** | `/docs/guides` | 编写教程和示例 |
| **开发文档** | `/docs/development` | 更新开发指南 |
| **部署文档** | `/docs/deployment` | 添加部署方案 |
| **架构文档** | `/docs/architecture` | 更新架构说明 |

### 文档标准
1. **准确性**: 信息必须准确且最新
2. **清晰性**: 语言简洁明了
3. **完整性**: 覆盖所有重要方面
4. **一致性**: 遵循文档风格指南
5. **可访问性**: 考虑不同背景的读者

### 文档编写流程
```bash
# 1. 创建文档分支
git checkout -b docs/topic-name

# 2. 编写文档
# 使用 Markdown 格式
# 添加适当的标题和结构
# 包含代码示例和截图

# 3. 预览文档
# 使用本地 Markdown 预览器
# 检查链接和格式

# 4. 提交 PR
git add .
git commit -m "docs: add guide for feature X"
git push origin docs/topic-name
```

## 🔍 代码审查

### 审查者指南
1. **及时响应**: 24 小时内初步响应
2. **建设性反馈**: 提供具体改进建议
3. **尊重贡献者**: 感谢所有贡献
4. **关注重点**: 代码质量、安全性、性能
5. **解释原因**: 说明为什么需要修改

### 审查清单
- [ ] 代码遵循项目规范
- [ ] 功能按预期工作
- [ ] 测试覆盖充分
- [ ] 文档更新完整
- [ ] 性能影响可接受
- [ ] 安全考虑周全
- [ ] 向后兼容性保持
- [ ] 代码可读性高

### 审查流程
```bash
# 1. 检查 CI 状态
# 确保所有检查通过

# 2. 本地测试
git checkout feature-branch
pnpm i
pnpm run test
pnpm run build

# 3. 代码审查
# 使用 GitHub Review 功能
# 添加具体评论
# 请求必要的修改

# 4. 批准合并
# 所有问题解决后批准
# 使用 Squash and Merge
```

## 🚀 发布流程

### 版本管理
使用 [Semantic Versioning](https://semver.org/):
- **主版本号**: 不兼容的 API 变更
- **次版本号**: 向后兼容的功能性新增
- **修订号**: 向后兼容的问题修正

### 发布步骤
1. **功能冻结**: 停止新功能开发
2. **测试阶段**: 全面测试和修复
3. **预发布**: 发布候选版本
4. **正式发布**: 发布稳定版本
5. **维护阶段**: 修复和更新

### 变更日志
每个版本必须更新 `CHANGELOG.md`:
```markdown
## [3.154.0] - 2024-01-15

### Added
- 新增 OAuth2 认证支持
- 添加批量追踪导入功能

### Changed
- 优化数据库查询性能
- 更新 API 响应格式

### Fixed
- 修复内存泄漏问题
- 修复时区处理错误

### Deprecated
- 废弃旧版 API 端点
```

## 🏆 成为核心贡献者

### 贡献等级
| 等级 | 要求 | 权限 |
|------|------|------|
| **贡献者** | 1+ 个合并的 PR | 提交 PR |
| **活跃贡献者** | 5+ 个重要 PR | 代码审查 |
| **核心贡献者** | 持续贡献，项目理解深 | 合并权限 |
| **维护者** | 项目领导，长期承诺 | 发布权限 |

### 成长路径
1. **开始**: 解决 Good First Issues
2. **深入**: 参与功能开发
3. **领导**: 主导功能模块
4. **指导**: 帮助新贡献者
5. **维护**: 参与项目维护

### 核心贡献者责任
- 定期参与代码审查
- 帮助新贡献者入门
- 参与技术决策讨论
- 维护代码质量和一致性
- 协助问题排查和修复

## 🌍 社区参与

### 参与方式
1. **Discord 社区**
   - 回答问题
   - 分享经验
   - 参与讨论

2. **GitHub 讨论**
   - 提出建议
   - 分享用例
   - 寻求帮助

3. **博客和教程**
   - 编写教程
   - 分享案例研究
   - 录制视频演示

4. **会议和活动**
   - 参加社区会议
   - 分享项目经验
   - 组织本地活动

### 社区准则
- 保持尊重和友善
- 提供有价值的帮助
- 分享知识和经验
- 鼓励多样性和包容性
- 遵守社区行为准则

## 🛠️ 工具和资源

### 开发工具
- **IDE**: VS Code 推荐配置
- **Git**: GitHub Desktop 或命令行
- **Docker**: 容器化开发环境
- **测试工具**: Jest, Playwright
- **代码质量**: ESLint, Prettier

### 学习资源
- [项目架构文档](../architecture/README.md)
- [开发环境指南](../development/setup.md)
- [代码规范指南](../development/code-style.md)
- [测试指南](../development/testing.md)
- [API 文档](../api/README.md)

### 沟通渠道
- **问题报告**: GitHub Issues
- **功能讨论**: GitHub Discussions
- **实时交流**: Discord
- **安全报告**: security@langfuse.com
- **一般咨询**: support@langfuse.com

## 📊 贡献统计

### 贡献者认可
- **贡献者列表**: 在 README 中列出
- **特别感谢**: 突出重要贡献
- **社区奖项**: 定期表彰优秀贡献者
- **职业发展**: 提供推荐信和证明

### 贡献追踪
```bash
# 查看个人贡献
git log --author="your-email@example.com" --oneline

# 查看项目贡献统计
git shortlog -sn --all

# 生成贡献报告
git log --since="1 month ago" --pretty=format:"%h - %an, %ar : %s"
```

## 🎯 最佳实践

### 对于新贡献者
1. **从小处着手**: 从 Good First Issues 开始
2. **寻求帮助**: 不要犹豫提问
3. **学习代码**: 理解现有代码结构
4. **遵循规范**: 遵守项目约定
5. **保持耐心**: 贡献需要时间和学习

### 对于经验贡献者
1. **指导新人**: 帮助新贡献者入门
2. **代码审查**: 提供建设性反馈
3. **文档维护**: 保持文档更新
4. **质量把关**: 确保代码质量
5. **社区建设**: 促进积极社区文化

### 对于维护者
1. **明确期望**: 清晰定义贡献流程
2. **及时响应**: 快速处理 PR 和 Issues
3. **公平评估**: 客观评价所有贡献
4. **持续改进**: 优化贡献流程
5. **社区培育**: 培养健康社区生态

## 🚨 紧急贡献

### 安全漏洞
1. **私下报告**: 发送到 security@langfuse.com
2. **不要公开**: 避免在公开渠道讨论
3. **快速响应**: 安全团队会尽快处理
4. **协调发布**: 协调修复和披露时间

### 严重问题
1. **标记优先级**: 使用 `critical` 标签
2. **快速修复**: 优先处理关键问题
3. **热修复发布**: 发布紧急修复版本
4. **通知用户**: 通过适当渠道通知

## 📚 相关文档

### 内部文档
- [行为准则](CODE_OF_CONDUCT.md)
- [开发工作流](../development/workflow.md)
- [部署指南](../deployment/README.md)
- [架构设计](../architecture/README.md)

### 外部资源
- [GitHub 贡献指南](https://docs.github.com/en/communities)
- [开源贡献最佳实践](https://opensource.guide/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)

### 工具文档
- [Git 文档](https://git-scm.com/doc)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [VS Code 开发容器](https://code.visualstudio.com/docs/remote/containers)

---

**贡献检查清单**：
```bash
# 开始前
[ ] 阅读行为准则
[ ] 设置开发环境
[ ] 理解项目结构
[ ] 选择合适任务

# 开发中
[ ] 遵循代码规范
[ ] 编写充分测试
[ ] 保持代码整洁
[ ] 及时提交代码

# 提交前
[ ] 运行所有测试
[ ] 更新相关文档
[ ] 检查代码质量
[ ] 准备 PR 描述

# PR 审查中
[ ] 及时响应反馈
[ ] 礼貌沟通交流
[ ] 认真处理修改
[ ] 感谢审查意见
```

感谢您对 Langfuse 项目的贡献！您的参与使这个项目变得更好。🚀

## 🙏 致谢

### 特别感谢
- 所有代码贡献者
- 文档编写和维护者
- 问题报告和测试者
- 社区支持和推广者
- 项目维护和领导者

### 如何致谢
- 在发布说明中提及
- 在文档中列出贡献者
- 在社交媒体上分享
- 在社区会议中表彰
- 提供贡献者福利

### 长期贡献
我们重视长期贡献者，并提供：
- 项目决策参与权
- 核心维护者权限
- 职业发展支持
- 社区领导机会
- 特别认可和奖励

---

**欢迎贡献！** 无论您是第一次参与开源项目，还是经验丰富的贡献者，我们都期待您的参与。让我们一起构建更好的 LLM 工程平台！🎉