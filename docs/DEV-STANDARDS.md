# 开发规范

> **本文档定义项目的代码规范和开发流程**
> **从CLAUDE.md迁移而来，避免主文件过大**

---

## TypeScript规范

### 基本原则
- ✅ 启用严格模式（`strict: true`）
- ✅ 所有函数必须有类型注解
- ✅ 禁止使用`any`（允许`unknown`）
- ✅ 接口命名使用`I`前缀（如`IAIProvider`）

### 示例
```typescript
// ✅ 正确
interface IAIProvider {
  login(): Promise<boolean>;
  query(question: string): Promise<string>;
}

// ❌ 错误
interface AIProvider {  // 缺少I前缀
  login(): any;  // 使用了any
}
```

---

## 测试规范

### 覆盖率要求
- ✅ 代码覆盖率≥70%
- ✅ 关键路径100%覆盖
- ✅ 每个Provider至少3个单元测试

### 测试命名
- ✅ 格式：`describe('ComponentName', () => { it('should do something', ...) })`

### 示例
```typescript
describe('DoubaoProvider', () => {
  it('should login successfully with valid credentials', async () => {
    // 测试代码
  });

  it('should handle network timeout', async () => {
    // 测试代码
  });

  it('should maintain session after login', async () => {
    // 测试代码
  });
});
```

---

## Commit规范

### 格式
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type类型
- `feat`: 新功能
- `fix`: 修复bug
- `docs`: 文档修改
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建/工具相关

### 示例
```bash
git commit -m "feat(doubao): 完成登录功能

- 实现手机号验证码登录
- 添加会话状态检测
- 集成到Provider接口

测试: 3个单元测试通过
TASKS: 完成 [1.2] 豆包Provider开发
NEXT: 开始Kimi Provider开发"
```

### 注意事项
- ✅ 每完成一个Task checkbox就commit
- ✅ 详细的commit body（包含测试、影响、下一步）
- ✅ 重大变更必须记录到`CHANGELOG.md`

---

## 文档规范

### 基本原则
- ✅ 代码变更必须同步更新文档
- ✅ 弃用代码必须标记`@deprecated`
- ✅ 重大变更必须记录到`CHANGELOG.md`
- ✅ 每次会话结束更新`CONTEXT.md`

### 示例
```typescript
/**
 * @deprecated 使用 newMethod() 代替
 * @see {@link newMethod}
 */
function oldMethod() {
  // ...
}
```

---

## 代码审查Checklist

开发完成后，自查以下项：

### TypeScript
- [ ] 无`any`类型
- [ ] 所有函数有类型注解
- [ ] 接口命名符合规范

### 测试
- [ ] 单元测试通过
- [ ] 覆盖率≥70%
- [ ] 关键路径100%覆盖

### 文档
- [ ] 更新相关文档
- [ ] 更新CHANGELOG.md
- [ ] 更新CONTEXT.md

### Git
- [ ] Commit message符合规范
- [ ] 代码已提交

---

**Last Updated**: 2025-10-18 21:30
**Status**: 活跃使用中
