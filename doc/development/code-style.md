# 代码规范指南

本文档定义了 Langfuse 项目的代码编写规范，确保代码的一致性、可读性和可维护性。

## 📋 规范概览

### 核心原则
1. **一致性**: 团队统一风格
2. **可读性**: 代码即文档
3. **可维护性**: 易于修改和扩展
4. **安全性**: 避免常见漏洞
5. **性能**: 高效且可扩展

### 工具支持
- **ESLint**: 代码质量检查
- **Prettier**: 代码格式化
- **TypeScript**: 类型安全
- **Husky**: Git 钩子
- **lint-staged**: 暂存文件检查

## 🎨 代码格式化

### 基础规则
- **缩进**: 2个空格（非制表符）
- **行宽**: 最大 100 字符
- **引号**: 单引号（字符串），反引号（模板字符串）
- **分号**: 必须使用
- **逗号**: 尾随逗号（多行时）

### 配置文件
```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false
}
```

## 📝 命名规范

### 通用规则
- **清晰表达意图**: 名称应准确描述用途
- **避免缩写**: 除非是广泛接受的缩写
- **一致性**: 相同概念使用相同命名

### 具体规范

#### 1. 变量和常量
```typescript
// ✅ 正确
const userCount = 10;
const MAX_RETRIES = 3;
const isLoading = false;
const apiEndpoint = '/api/v1/users';

// ❌ 避免
const cnt = 10;          // 缩写
const maxretries = 3;    // 小写常量
const is_loading = false; // 下划线
```

#### 2. 函数和方法
```typescript
// ✅ 正确
function calculateTotalPrice() {}
async function fetchUserData() {}
const handleSubmit = () => {};
const isValidEmail = (email: string) => {};

// ❌ 避免
function calc() {}           // 缩写
async function get() {}      // 过于通用
const submitHandler = () => {}; // 不一致
```

#### 3. 类和接口
```typescript
// ✅ 正确
class UserRepository {}
interface ApiResponse {}
type UserPreferences = {};

// ❌ 避免
class userRepo {}        // 小写开头
interface api_response {} // 下划线
type prefs = {};         // 缩写
```

#### 4. 文件和目录
```bash
# ✅ 正确
src/components/UserProfile.tsx
src/utils/dateFormatter.ts
src/types/api.d.ts

# ❌ 避免
src/components/userProfile.tsx  # 不一致大小写
src/utils/date-formatter.ts     # 连字符（除非必要）
src/types/Api.d.ts              # 不一致扩展名
```

## 🏗️ 代码结构

### 文件组织
```
src/
├── components/     # React 组件
│   ├── common/     # 通用组件
│   ├── layout/     # 布局组件
│   └── features/   # 功能组件
├── hooks/          # 自定义 Hook
├── utils/          # 工具函数
├── types/          # TypeScript 类型
├── constants/      # 常量定义
├── services/       # API 服务
├── store/          # 状态管理
└── styles/         # 样式文件
```

### 导入顺序
```typescript
// 1. 外部依赖
import React from 'react';
import { useRouter } from 'next/router';

// 2. 内部模块
import { User } from '@/types';
import { formatDate } from '@/utils/date';

// 3. 相对路径
import { Header } from './Header';
import styles from './styles.module.css';

// 4. 类型导入
import type { ApiResponse } from '@/types/api';
```

### 导出规范
```typescript
// 命名导出（推荐）
export const CONSTANT_VALUE = 'value';
export function helperFunction() {}
export class UtilityClass {}

// 默认导出（组件、页面）
export default function Component() {}

// 批量导出
export { ComponentA, ComponentB } from './components';
```

## 🔧 TypeScript 规范

### 类型定义
```typescript
// ✅ 明确类型
interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
}

// ✅ 使用类型别名
type UserRole = 'admin' | 'user' | 'guest';

// ✅ 泛型约束
function findItem<T extends { id: string }>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

// ❌ 避免 any
function processData(data: any) {} // 避免
function processData(data: unknown) {} // 使用 unknown
```

### 严格模式
确保 `tsconfig.json` 启用：
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}
```

## ⚛️ React 规范

### 组件定义
```typescript
// ✅ 函数组件（推荐）
interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
}

export default function UserCard({ user, onEdit }: UserCardProps) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      {onEdit && <button onClick={() => onEdit(user)}>Edit</button>}
    </div>
  );
}

// ❌ 避免类组件（除非必要）
```

### Hook 使用
```typescript
// ✅ 正确使用
import { useState, useEffect, useCallback } from 'react';

function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  
  const fetchUsers = useCallback(async () => {
    setLoading(true);
    try {
      const data = await api.getUsers();
      setUsers(data);
    } finally {
      setLoading(false);
    }
  }, []);
  
  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);
  
  // ...
}
```

### Props 设计
```typescript
// ✅ 明确且简洁
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}

// ✅ 使用默认值
const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'medium',
  disabled = false,
  onClick,
  children
}) => {
  // ...
};
```

## 🧪 测试规范

### 测试文件命名
```bash
# 单元测试
Component.tsx
Component.test.tsx  # 或 Component.spec.tsx

# 集成测试
Component.integration.test.tsx

# E2E 测试
Component.e2e.test.tsx
```

### 测试结构
```typescript
describe('ComponentName', () => {
  // 前置条件
  beforeEach(() => {
    // 设置测试环境
  });
  
  afterEach(() => {
    // 清理测试环境
  });
  
  // 测试用例
  it('should render correctly', () => {
    // 断言
  });
  
  it('should handle user interaction', () => {
    // 模拟用户操作
  });
  
  describe('when data is loading', () => {
    it('should show loading state', () => {
      // 特定场景测试
    });
  });
});
```

## 🔒 安全规范

### 输入验证
```typescript
// ✅ 使用 Zod 验证
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  role: z.enum(['admin', 'user']),
});

// ✅ 清理用户输入
function sanitizeInput(input: string): string {
  return input.trim().replace(/[<>]/g, '');
}
```

### 敏感数据处理
```typescript
// ✅ 避免日志敏感信息
console.log('User logged in:', { userId: user.id }); // ✅
console.log('User logged in:', user); // ❌ 可能包含敏感数据

// ✅ 使用环境变量
const apiKey = process.env.API_KEY; // ✅
const apiKey = 'hardcoded-key'; // ❌
```

## 🚀 性能规范

### 渲染优化
```typescript
// ✅ 使用 React.memo
const MemoizedComponent = React.memo(Component);

// ✅ 使用 useMemo 和 useCallback
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

// ✅ 代码分割
const LazyComponent = React.lazy(() => import('./LazyComponent'));
```

### 网络请求
```typescript
// ✅ 请求去重
import { useQuery } from '@tanstack/react-query';

// ✅ 错误重试
const { data } = useQuery({
  queryKey: ['user', userId],
  queryFn: fetchUser,
  retry: 3,
  retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
});
```

## 📚 文档规范

### 代码注释
```typescript
/**
 * 计算用户的总订单金额
 * 
 * @param userId - 用户ID
 * @param currency - 货币类型，默认为 'USD'
 * @returns 总金额，如果用户不存在则返回 0
 * 
 * @example
 * const total = calculateUserTotal('user-123');
 * console.log(total); // 150.75
 */
function calculateUserTotal(userId: string, currency: string = 'USD'): number {
  // 实现...
}
```

### README 文件
每个目录应包含 README.md：
```markdown
# 目录名称

## 用途
说明此目录的用途和包含的内容。

## 文件结构
```
directory/
├── File1.ts
├── File2.ts
└── README.md
```

## 使用示例
提供使用示例。

## 注意事项
列出重要注意事项。
```

## 🔍 代码审查要点

### 必须检查项
- [ ] 代码遵循命名规范
- [ ] 类型定义完整且准确
- [ ] 错误处理完善
- [ ] 测试覆盖充分
- [ ] 文档更新及时
- [ ] 性能影响评估
- [ ] 安全考虑周全

### 建议检查项
- [ ] 代码可读性高
- [ ] 函数职责单一
- [ ] 重复代码已重构
- [ ] 依赖关系合理
- [ ] 向后兼容性考虑

## 🛠️ 工具配置

### ESLint 配置
```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'next/core-web-vitals',
    'plugin:@typescript-eslint/recommended',
    'prettier'
  ],
  rules: {
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/explicit-function-return-type': 'off',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn'
  }
};
```

### Git 钩子
```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "pre-push": "npm run test"
    }
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

## 📊 质量指标

### 代码质量
- **测试覆盖率**: > 80%
- **类型覆盖率**: > 95%
- **重复代码**: < 3%
- **圈复杂度**: < 10

### 审查效率
- **PR 大小**: 建议 < 500 行
- **审查时间**: 目标 < 24 小时
- **反馈质量**: 具体且有建设性

## 🎯 最佳实践总结

### 立即应用
1. 使用 TypeScript 严格模式
2. 遵循命名规范
3. 编写有意义的注释
4. 添加适当的测试

### 持续改进
1. 定期回顾代码规范
2. 分享最佳实践
3. 参与代码审查
4. 学习新技术和模式

### 团队协作
1. 尊重现有代码风格
2. 提供建设性反馈
3. 及时沟通问题
4. 共同维护代码质量

## 📚 相关资源

### 内部文档
- [开发工作流](./workflow.md)
- [测试指南](./testing.md)
- [架构设计](../architecture/README.md)

### 外部资源
- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
- [React Best Practices](https://reactjs.org/docs/faq-structure.html)

### 工具文档
- [ESLint Rules](https://eslint.org/docs/rules/)
- [Prettier Options](https://prettier.io/docs/en/options.html)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)

---

**代码规范检查清单**：
```bash
# 提交前检查
[ ] 运行 ESLint: pnpm run lint
[ ] 运行 Prettier: pnpm run format:check
[ ] 运行类型检查: pnpm run typecheck
[ ] 运行测试: pnpm run test

# 代码审查检查
[ ] 命名是否符合规范
[ ] 类型定义是否完整
[ ] 错误处理是否完善
[ ] 测试是否充分
[ ] 文档是否更新
```

遵循这些规范将确保代码质量、团队协作效率和项目的长期可维护性。🚀