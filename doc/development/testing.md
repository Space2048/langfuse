# 测试指南

本文档详细介绍了 Langfuse 项目的测试策略、工具使用和最佳实践。

## 📋 测试概览

### 测试金字塔
```
        E2E 测试 (5-10%)
          / \
         /   \
        /     \
  集成测试 (15-20%)  集成测试 (15-20%)
      \       /
       \     /
        \   /
      单元测试 (60-70%)
```

### 测试目标
- **可靠性**: 确保功能按预期工作
- **可维护性**: 便于重构和修改
- **文档性**: 作为代码行为的文档
- **质量**: 减少生产环境缺陷

## 🛠️ 测试工具栈

### 核心工具
- **Jest**: 单元和集成测试框架
- **Playwright**: E2E 测试框架
- **React Testing Library**: React 组件测试
- **MSW**: API 模拟
- **Zod**: 数据验证测试

### 辅助工具
- **jest.config.js**: Jest 配置
- **playwright.config.ts**: Playwright 配置
- **@testing-library/jest-dom**: DOM 断言扩展
- **@testing-library/user-event**: 用户交互模拟

## 🧪 单元测试

### 测试文件结构
```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   └── index.ts
│   └── __tests__/          # 可选：共享测试工具
├── utils/
│   ├── dateFormatter.ts
│   └── dateFormatter.test.ts
└── services/
    ├── api.ts
    └── api.test.ts
```

### 测试编写规范

#### 1. 工具函数测试
```typescript
// utils/dateFormatter.test.ts
import { formatDate, parseDate } from './dateFormatter';

describe('dateFormatter', () => {
  describe('formatDate', () => {
    it('should format date to YYYY-MM-DD', () => {
      const date = new Date('2024-01-15');
      expect(formatDate(date)).toBe('2024-01-15');
    });

    it('should handle invalid date', () => {
      expect(() => formatDate(new Date('invalid'))).toThrow('Invalid date');
    });
  });

  describe('parseDate', () => {
    it('should parse YYYY-MM-DD string to Date', () => {
      const result = parseDate('2024-01-15');
      expect(result.getFullYear()).toBe(2024);
      expect(result.getMonth()).toBe(0); // 0-indexed
      expect(result.getDate()).toBe(15);
    });

    it('should return null for invalid format', () => {
      expect(parseDate('15-01-2024')).toBeNull();
    });
  });
});
```

#### 2. React 组件测试
```typescript
// components/Button/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Button from './Button';

describe('Button', () => {
  const defaultProps = {
    onClick: jest.fn(),
    children: 'Click me',
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should render button with text', () => {
    render(<Button {...defaultProps} />);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('should call onClick when clicked', async () => {
    const user = userEvent.setup();
    render(<Button {...defaultProps} />);
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(defaultProps.onClick).toHaveBeenCalledTimes(1);
  });

  it('should be disabled when disabled prop is true', () => {
    render(<Button {...defaultProps} disabled />);
    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('should show loading state', () => {
    render(<Button {...defaultProps} loading />);
    expect(screen.getByRole('button')).toHaveAttribute('aria-busy', 'true');
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });

  describe('accessibility', () => {
    it('should have proper aria-label when provided', () => {
      render(<Button {...defaultProps} aria-label="Submit form" />);
      expect(screen.getByRole('button')).toHaveAttribute('aria-label', 'Submit form');
    });
  });
});
```

#### 3. Hook 测试
```typescript
// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('should initialize with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('should increment count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('should decrement count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  it('should reset count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(0);
  });
});
```

## 🔗 集成测试

### API 集成测试
```typescript
// __tests__/integration/api.test.ts
import { setupServer } from 'msw/node';
import { rest } from 'msw';
import { fetchUser, createUser } from '@/services/api';

const server = setupServer(
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    return res(
      ctx.json({
        id,
        name: 'John Doe',
        email: 'john@example.com',
      })
    );
  }),
  
  rest.post('/api/users', async (req, res, ctx) => {
    const userData = await req.json();
    return res(
      ctx.status(201),
      ctx.json({
        id: 'user-123',
        ...userData,
        createdAt: new Date().toISOString(),
      })
    );
  })
);

describe('API Integration', () => {
  beforeAll(() => server.listen());
  afterEach(() => server.resetHandlers());
  afterAll(() => server.close());

  it('should fetch user by id', async () => {
    const user = await fetchUser('user-123');
    
    expect(user).toEqual({
      id: 'user-123',
      name: 'John Doe',
      email: 'john@example.com',
    });
  });

  it('should create new user', async () => {
    const newUser = {
      name: 'Jane Doe',
      email: 'jane@example.com',
    };
    
    const createdUser = await createUser(newUser);
    
    expect(createdUser).toMatchObject({
      id: expect.any(String),
      ...newUser,
      createdAt: expect.any(String),
    });
  });

  it('should handle API errors', async () => {
    server.use(
      rest.get('/api/users/:id', (req, res, ctx) => {
        return res(ctx.status(404));
      })
    );
    
    await expect(fetchUser('non-existent')).rejects.toThrow('User not found');
  });
});
```

### 数据库集成测试
```typescript
// __tests__/integration/database.test.ts
import { PrismaClient } from '@prisma/client';
import { createUser, findUserByEmail } from '@/repositories/userRepository';

const prisma = new PrismaClient();

describe('Database Integration', () => {
  beforeAll(async () => {
    await prisma.$connect();
  });

  afterAll(async () => {
    await prisma.$disconnect();
  });

  beforeEach(async () => {
    // 清理测试数据
    await prisma.user.deleteMany();
  });

  it('should create and find user', async () => {
    const userData = {
      email: 'test@example.com',
      name: 'Test User',
      passwordHash: 'hashed-password',
    };

    // 创建用户
    const createdUser = await createUser(userData);
    expect(createdUser.email).toBe(userData.email);
    expect(createdUser.id).toBeDefined();

    // 查找用户
    const foundUser = await findUserByEmail(userData.email);
    expect(foundUser).not.toBeNull();
    expect(foundUser?.email).toBe(userData.email);
  });

  it('should handle duplicate email', async () => {
    const userData = {
      email: 'duplicate@example.com',
      name: 'User 1',
      passwordHash: 'hash1',
    };

    await createUser(userData);
    
    // 尝试创建重复用户
    await expect(
      createUser({ ...userData, name: 'User 2', passwordHash: 'hash2' })
    ).rejects.toThrow();
  });
});
```

## 🌐 E2E 测试

### Playwright 测试
```typescript
// e2e/tests/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should login successfully', async ({ page }) => {
    // 导航到登录页
    await page.click('text=Sign In');
    
    // 填写表单
    await page.fill('input[name="email"]', 'demo@langfuse.com');
    await page.fill('input[name="password"]', 'password');
    
    // 提交表单
    await page.click('button[type="submit"]');
    
    // 验证登录成功
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('text=Welcome back')).toBeVisible();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.click('text=Sign In');
    
    await page.fill('input[name="email"]', 'wrong@example.com');
    await page.fill('input[name="password"]', 'wrongpassword');
    await page.click('button[type="submit"]');
    
    // 验证错误消息
    await expect(page.locator('text=Invalid email or password')).toBeVisible();
  });

  test('should logout successfully', async ({ page }) => {
    // 先登录
    await page.click('text=Sign In');
    await page.fill('input[name="email"]', 'demo@langfuse.com');
    await page.fill('input[name="password"]', 'password');
    await page.click('button[type="submit"]');
    
    // 登出
    await page.click('[data-testid="user-menu"]');
    await page.click('text=Sign Out');
    
    // 验证登出成功
    await expect(page).toHaveURL('/');
    await expect(page.locator('text=Sign In')).toBeVisible();
  });
});

// e2e/tests/tracing.spec.ts
test.describe('Tracing', () => {
  test('should create new trace', async ({ page }) => {
    await page.goto('/traces');
    
    // 点击新建按钮
    await page.click('text=New Trace');
    
    // 填写追踪信息
    await page.fill('input[name="name"]', 'Test Trace');
    await page.fill('textarea[name="description"]', 'This is a test trace');
    
    // 提交
    await page.click('button[type="submit"]');
    
    // 验证创建成功
    await expect(page.locator('text=Test Trace')).toBeVisible();
    await expect(page.locator('text=Trace created successfully')).toBeVisible();
  });

  test('should filter traces', async ({ page }) => {
    await page.goto('/traces');
    
    // 应用过滤器
    await page.fill('input[placeholder="Search traces..."]', 'test');
    await page.click('button[type="submit"]');
    
    // 验证过滤结果
    const traceItems = page.locator('[data-testid="trace-item"]');
    await expect(traceItems).toHaveCount(1);
  });
});
```

### 测试数据管理
```typescript
// e2e/fixtures/testData.ts
export const testUsers = {
  admin: {
    email: 'admin@example.com',
    password: 'admin123',
    role: 'admin' as const,
  },
  user: {
    email: 'user@example.com',
    password: 'user123',
    role: 'user' as const,
  },
  demo: {
    email: 'demo@langfuse.com',
    password: 'password',
    role: 'user' as const,
  },
};

export const testTraces = [
  {
    name: 'API Request Trace',
    description: 'Trace for API request processing',
    metadata: { method: 'GET', endpoint: '/api/users' },
  },
  {
    name: 'Error Trace',
    description: 'Trace for error handling',
    metadata: { error: 'Timeout', retryCount: 3 },
  },
];
```

## 🧩 测试最佳实践

### 1. 测试隔离
```typescript
// ✅ 每个测试独立运行
describe('UserService', () => {
  let userService: UserService;
  let mockRepository: jest.Mocked<UserRepository>;
  
  beforeEach(() => {
    mockRepository = {
      findById: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
    };
    userService = new UserService(mockRepository);
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
});
```

### 2. 测试描述清晰
```typescript
// ✅ 描述行为而非实现
it('should return user when valid id is provided', () => {});
it('should throw error when user not found', () => {});
it('should update lastLogin timestamp', () => {});

// ❌ 避免
it('should work', () => {});
it('test 1', () => {});
```

### 3. 使用恰当的断言
```typescript
// ✅ 具体断言
expect(user.name).toBe('John Doe');
expect(users).toHaveLength(3);
expect(response).toMatchObject({ status: 'success' });
expect(button).toBeDisabled();
expect(console.error).toHaveBeenCalledWith('Error message');

// ❌ 避免模糊断言
expect(result).toBeTruthy(); // 太模糊
```

### 4. 模拟外部依赖
```typescript
// ✅ 正确模拟
jest.mock('@/services/api', () => ({
  fetchUser: jest.fn().mockResolvedValue({ id: '123', name: 'John' }),
}));

// ✅ 使用 MSW 模拟 API
const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json([{ id: '1', name: 'John' }]));
  })
);
```

### 5. 测试边缘情况
```typescript
describe('edge cases', () => {
  it('should handle empty array', () => {});
  it('should handle null input', () => {});
  it('should handle very large numbers', () => {});
  it('should handle special characters', () => {});
  it('should handle concurrent requests', () => {});
});
```

## 📊 测试覆盖率

### 覆盖率目标
- **语句覆盖率**: > 80%
- **分支覆盖率**: > 70%
- **函数覆盖率**: > 85%
- **行覆盖率**: > 80%

### 覆盖率报告
```bash
# 生成覆盖率报告
pnpm run test:coverage

# 查看 HTML 报告
open coverage/lcov-report/index.html
```

### 忽略文件
```json
// jest.config.js
module.exports = {
  coveragePathIgnorePatterns: [
    '/node_modules/',
    '/dist/',
    '/coverage/',
    '__tests__',
    '.test.',
    '.spec.',
    '.mock.',
  ],
};
```

## 🚀 测试命令

### 常用命令
```bash
# 运行所有测试
pnpm run test

# 运行特定测试文件
pnpm run test -- src/components/Button/Button.test.tsx

# 运行匹配模式的测试
pnpm run test -- --testNamePattern="authentication"

# 监视模式（开发时）
pnpm run test:watch

# 覆盖率报告
pnpm run test:coverage

# E2E 测试
pnpm run test:e2e

# 并行测试
pnpm run test:parallel
```

### CI/CD 集成
```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '24'
      - run: pnpm install
      - run: pnpm run test
      - run: pnpm run test:e2e
      - run: pnpm run test:coverage
```

## 🔍 调试测试

### Jest 调试
```bash
# 调试特定测试
node --inspect-brk node_modules/.bin/jest --runInBand --testNamePattern="my test"

# 然后打开 chrome://inspect
```

### Playwright 调试
```bash
# 调试模式
npx playwright test --debug

# UI 模式
npx playwright test --ui

# 追踪模式
npx playwright test --trace on
```

### VS Code 调试配置
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Jest Current File",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": ["${relativeFile}", "--runInBand"],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

## 📚 测试资源

### 学习资源
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Playwright Documentation](https://playwright.dev/docs/intro)
- [Testing JavaScript](https://testingjavascript.com/)

### 工具文档
- [MSW - Mock Service Worker](https://mswjs.io/docs/)
- [jest-dom Custom Matchers](https://github.com/testing-library/jest-dom)
- [user-event Library](https://testing-library.com/docs/user-event/intro)

### 最佳实践
- [Testing Implementation Details](https://kentcdodds.com/blog/testing-implementation-details)
- [Common Testing Mistakes](https://kentcdodds.com/blog/common-testing-mistakes)
- [Write Tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests)

---

**测试检查清单**：
```bash
# 编写测试前
[ ] 理解需求和行为
[ ] 确定测试类型（单元/集成/E2E）
[ ] 设计测试用例

# 编写测试时
[ ] 测试描述清晰
[ ] 测试独立运行
[ ] 使用恰当的断言
[ ] 模拟外部依赖
[ ] 测试边缘情况

# 测试完成后
[ ] 所有测试通过
[ ] 覆盖率达标
[ ] 代码审查通过
[ ] 文档更新

# 维护测试
[ ] 定期更新测试
[ ] 重构时更新测试
[ ] 删除过时测试
[ ] 优化测试性能
```

遵循这些测试指南将确保代码质量、减少缺陷并提高开发效率。🚀