# Provider开发规范

> **本文件在Claude读取src/providers/目录文件时自动加载**

---

## Provider接口要求

所有AI Provider必须实现 `IAIProvider` 接口：

```typescript
interface IAIProvider {
  login(): Promise<boolean>;
  query(question: string): Promise<string>;
  logout(): Promise<void>;
  getStatus(): Promise<SessionStatus>;
}
```

---

## 目录结构

```
src/providers/
├── base/
│   └── BaseProvider.ts        # 抽象基类（通用逻辑）
├── doubao/                    # 豆包
│   ├── DoubaoProvider.ts
│   ├── DoubaoLogin.ts
│   └── DoubaoQuery.ts
├── kimi/                      # Kimi
└── deepseek/                  # DeepSeek
```

---

## 开发规范

### 1. 继承BaseProvider
```typescript
export class DoubaoProvider extends BaseProvider implements IAIProvider {
  // ...
}
```

### 2. 测试要求
- 至少3个单元测试：login、query、session recovery
- 使用`jest.mock`模拟网络请求

### 3. 参考实现
- 查看test1项目：`../test1/chrome-devtools-mcp/src/doubao/`

---

**Last Updated**: 2025-10-18 21:40
