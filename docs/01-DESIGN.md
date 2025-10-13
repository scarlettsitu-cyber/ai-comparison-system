# 架构设计文档

> **本文档描述ai-comparison-system的整体架构、技术选型和设计决策**

## 📋 文档信息

- **版本**: v1.0
- **创建日期**: 2025-10-13
- **最后更新**: 2025-10-13
- **负责人**: Claude Code + situruimin

---

## 1. 系统概述

### 1.1 项目定位

**ai-comparison-system**是一个CLI工具，用于同时向多个AI平台（豆包、Kimi、DeepSeek等）提问并对比结果，帮助用户快速获取不同AI的观点和答案。

### 1.2 核心价值

- 🚀 **效率提升**: 一次提问，多个AI同时响应
- 🔍 **结果对比**: 并排展示，快速发现差异
- 💾 **数据积累**: 持久化存储，支持历史回溯
- 🤖 **智能管理**: 自动会话管理，无需重复登录

### 1.3 目标用户

- 研究人员：需要对比多个AI的回答质量
- 开发者：测试不同AI的技术能力
- 内容创作者：寻找最佳创作灵感
- 普通用户：获取更全面的信息

---

## 2. 系统架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                      CLI Interface Layer                        │
│          (Commander.js + Inquirer.js + Chalk + Ora)             │
└─────────────────────┬───────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                    Service Layer                                │
│  ┌─────────────────┐ ┌──────────────┐ ┌────────────────────┐   │
│  │  QueryService   │ │SessionService│ │ ComparisonService  │   │
│  │  (查询编排)      │ │(会话管理)     │ │   (结果对比)        │   │
│  └─────────────────┘ └──────────────┘ └────────────────────┘   │
└─────────────────────┬───────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                  AI Providers Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │BaseProvider  │  │  BaseProvider│  │  BaseProvider│          │
│  │(抽象基类)     │  │              │  │              │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                 │                 │                   │
│  ┌──────▼───────┐  ┌──────▼───────┐  ┌──────▼───────┐          │
│  │  Doubao      │  │    Kimi      │  │  DeepSeek    │          │
│  │  Provider    │  │   Provider   │  │   Provider   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────┬───────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│              Infrastructure Layer                               │
│  ┌─────────────────────┐      ┌──────────────────────┐         │
│  │   BrowserManager    │      │  DatabaseService     │         │
│  │   (Puppeteer +      │      │     (SQLite)         │         │
│  │ chrome-devtools-mcp)│      │                      │         │
│  └─────────────────────┘      └──────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 分层说明

#### **Interface Layer (CLI)**
- **职责**: 用户交互、命令解析、结果展示
- **技术**: Commander.js（命令框架）、Inquirer.js（交互提示）
- **关键组件**:
  - `commands/login.ts` - 登录命令
  - `commands/query.ts` - 查询命令
  - `commands/batch.ts` - 批量查询命令
  - `commands/status.ts` - 状态查询命令

#### **Service Layer (业务逻辑)**
- **职责**: 业务编排、流程控制、数据处理
- **关键组件**:
  - `QueryService` - 查询编排（串行/并行）
  - `SessionService` - 会话管理（保存/恢复/验证）
  - `ComparisonService` - 结果对比（格式化/对比分析）

#### **Providers Layer (AI适配器)**
- **职责**: 封装AI平台特定逻辑
- **设计模式**: 适配器模式 + 模板方法模式
- **关键接口**:
  ```typescript
  interface IAIProvider {
    login(): Promise<boolean>;
    query(question: string): Promise<QueryResult>;
    logout(): Promise<void>;
    isLoggedIn(): Promise<boolean>;
  }
  ```

#### **Infrastructure Layer (基础设施)**
- **职责**: 提供底层能力（浏览器、数据库、工具）
- **关键组件**:
  - `BrowserManager` - 浏览器管理（启动/关闭/复用）
  - `DatabaseService` - 数据库操作（CRUD）
  - `FileUtils` - 文件工具（读写/导入导出）

---

## 3. 核心接口设计

### 3.1 IAIProvider 接口

```typescript
/**
 * AI提供商统一接口
 * 所有AI平台必须实现此接口
 */
export interface IAIProvider {
  /**
   * 提供商标识
   */
  readonly id: string;

  /**
   * 提供商名称
   */
  readonly name: string;

  /**
   * 登录URL
   */
  readonly loginUrl: string;

  /**
   * 执行登录
   * @returns 登录是否成功
   */
  login(): Promise<boolean>;

  /**
   * 检查登录状态
   * @returns 是否已登录
   */
  isLoggedIn(): Promise<boolean>;

  /**
   * 执行查询
   * @param question 查询问题
   * @returns 查询结果
   */
  query(question: string): Promise<QueryResult>;

  /**
   * 退出登录
   */
  logout(): Promise<void>;

  /**
   * 恢复会话
   * @param sessionData 会话数据
   */
  restoreSession(sessionData: SessionData): Promise<boolean>;

  /**
   * 获取会话数据
   */
  getSessionData(): Promise<SessionData>;
}
```

### 3.2 QueryResult 接口

```typescript
/**
 * 查询结果
 */
export interface QueryResult {
  /**
   * 请求ID（唯一标识）
   */
  requestId: string;

  /**
   * AI提供商ID
   */
  providerId: string;

  /**
   * 问题
   */
  question: string;

  /**
   * 回答文本
   */
  answer: string;

  /**
   * 引用链接
   */
  references: Reference[];

  /**
   * 执行时间（毫秒）
   */
  executionTime: number;

  /**
   * 状态
   */
  status: 'success' | 'failed' | 'timeout';

  /**
   * 错误信息（如果失败）
   */
  error?: string;

  /**
   * 创建时间
   */
  createdAt: Date;
}

/**
 * 引用链接
 */
export interface Reference {
  url: string;
  title?: string;
  snippet?: string;
}
```

### 3.3 SessionData 接口

```typescript
/**
 * 会话数据
 */
export interface SessionData {
  sessionId: string;
  providerId: string;
  userId?: string;
  username?: string;
  cookies: Cookie[];
  localStorage: Record<string, string>;
  sessionStorage: Record<string, string>;
  userAgent: string;
  createdAt: Date;
  lastAccessAt: Date;
  expiresAt: Date;
  isActive: boolean;
}
```

---

## 4. 数据库设计

### 4.1 SQLite Schema

```sql
-- AI应用表
CREATE TABLE ai_applications (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    provider_id TEXT NOT NULL UNIQUE,  -- 'doubao', 'kimi', 'deepseek'
    name TEXT NOT NULL,                -- '豆包', 'Kimi', 'DeepSeek'
    login_url TEXT NOT NULL,
    query_url TEXT NOT NULL,
    is_active BOOLEAN DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 分类表
CREATE TABLE categories (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,         -- '技术', '生活', '娱乐'
    description TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 查询记录表（核心表）
CREATE TABLE queries (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    request_id TEXT NOT NULL UNIQUE,   -- 同一批次查询共享此ID
    ai_application_id INTEGER NOT NULL,
    category_id INTEGER,
    question TEXT NOT NULL,
    result_text TEXT,                  -- AI的回答
    status TEXT NOT NULL DEFAULT 'pending',  -- 'pending', 'success', 'failed', 'timeout'
    execution_mode TEXT NOT NULL,      -- 'single', 'batch', 'parallel'
    execution_time_ms INTEGER,         -- 执行耗时（毫秒）
    error_message TEXT,                -- 错误信息（如果失败）
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ai_application_id) REFERENCES ai_applications(id),
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

-- 引用来源表
CREATE TABLE references (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    query_id INTEGER NOT NULL,
    url TEXT NOT NULL,
    title TEXT,
    snippet TEXT,                      -- 摘要
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (query_id) REFERENCES queries(id) ON DELETE CASCADE
);

-- 会话表
CREATE TABLE sessions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    session_id TEXT NOT NULL UNIQUE,
    provider_id TEXT NOT NULL,
    username TEXT,
    user_id TEXT,
    session_data TEXT NOT NULL,        -- JSON格式的会话数据
    user_agent TEXT,
    is_active BOOLEAN DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_access_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    expires_at DATETIME NOT NULL,
    FOREIGN KEY (provider_id) REFERENCES ai_applications(provider_id)
);

-- 批量任务表
CREATE TABLE batch_tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    task_id TEXT NOT NULL UNIQUE,
    request_id TEXT NOT NULL,          -- 关联queries表
    total_questions INTEGER NOT NULL,
    completed_questions INTEGER DEFAULT 0,
    failed_questions INTEGER DEFAULT 0,
    status TEXT NOT NULL DEFAULT 'pending',  -- 'pending', 'running', 'completed', 'failed'
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    completed_at DATETIME
);

-- 索引优化
CREATE INDEX idx_queries_request_id ON queries(request_id);
CREATE INDEX idx_queries_created_at ON queries(created_at);
CREATE INDEX idx_queries_status ON queries(status);
CREATE INDEX idx_sessions_provider_id ON sessions(provider_id);
CREATE INDEX idx_sessions_is_active ON sessions(is_active);
CREATE INDEX idx_references_query_id ON references(query_id);
```

### 4.2 数据流图

```
用户提问 → CLI命令解析 → QueryService编排
                            ↓
              ┌─────────────┴─────────────┐
              │                           │
         Provider1.query()          Provider2.query()
              ↓                           ↓
         QueryResult1               QueryResult2
              │                           │
              └─────────────┬─────────────┘
                            ↓
                    DatabaseService.save()
                            ↓
                      SQLite Database
```

---

## 5. 并行执行方案

### 5.1 技术挑战

**问题**: chrome-devtools-mcp默认使用Mutex强制工具顺序执行，无法并行操作多个浏览器

**代码证据**:
```typescript
// chrome-devtools-mcp/src/main.ts
const toolMutex = new Mutex();
await toolMutex.acquire();  // 获取锁，串行执行
```

### 5.2 解决方案

使用`--isolated`参数启动多个独立的MCP服务器实例，每个实例管理独立的浏览器：

```typescript
/**
 * 并行查询实现
 */
export class ParallelQueryExecutor {
  private mcpInstances: Map<string, MCPInstance> = new Map();

  async initialize(providers: string[]) {
    // 为每个Provider启动独立的MCP服务器
    for (const providerId of providers) {
      const instance = await this.launchMCPServer(providerId);
      this.mcpInstances.set(providerId, instance);
    }
  }

  private async launchMCPServer(providerId: string): Promise<MCPInstance> {
    // 启动独立的MCP服务器进程
    const process = spawn('node', [
      'node_modules/chrome-devtools-mcp/build/index.js',
      '--isolated',  // ← 关键参数
      `--user-data-dir=/tmp/mcp-${providerId}`
    ]);

    return {
      providerId,
      process,
      port: await this.allocatePort()
    };
  }

  async executeParallel(question: string, providers: string[]) {
    // 并行执行查询
    const promises = providers.map(providerId => {
      const instance = this.mcpInstances.get(providerId);
      return this.queryWithInstance(instance, question);
    });

    return await Promise.all(promises);
  }
}
```

### 5.3 性能对比

| 模式 | 3个AI查询耗时 | 内存占用 | 优势 | 劣势 |
|------|--------------|---------|------|------|
| **串行模式** | ~15秒 | ~500MB | 资源消耗低 | 速度慢 |
| **并行模式** | ~5秒 | ~1.5GB | 速度快3倍 | 资源消耗高 |

---

## 6. AI Provider实现策略

### 6.1 BaseProvider抽象类

```typescript
/**
 * Provider基类，封装通用逻辑
 */
export abstract class BaseProvider implements IAIProvider {
  protected browser: Browser;
  protected page: Page;

  abstract readonly id: string;
  abstract readonly name: string;
  abstract readonly loginUrl: string;

  /**
   * 模板方法：登录流程框架
   */
  async login(): Promise<boolean> {
    await this.navigateToLoginPage();
    await this.fillLoginForm();
    await this.submitLogin();
    return await this.verifyLoginSuccess();
  }

  // 钩子方法，由子类实现
  protected abstract fillLoginForm(): Promise<void>;
  protected abstract verifyLoginSuccess(): Promise<boolean>;

  // 通用方法，所有Provider复用
  protected async navigateToLoginPage(): Promise<void> {
    await this.page.goto(this.loginUrl, { waitUntil: 'networkidle2' });
  }

  protected async submitLogin(): Promise<void> {
    // 通用的提交逻辑
  }
}
```

### 6.2 豆包Provider示例

```typescript
/**
 * 豆包Provider实现
 * 从test1项目迁移并适配
 */
export class DoubaoProvider extends BaseProvider {
  readonly id = 'doubao';
  readonly name = '豆包';
  readonly loginUrl = 'https://www.doubao.com';

  protected async fillLoginForm(): Promise<void> {
    // 输入手机号
    await this.page.type('#phone-input', config.phoneNumber);

    // 点击发送验证码
    await this.page.click('#send-code-btn');

    // 等待用户手动输入验证码
    await this.waitForUserInput('#code-input');
  }

  protected async verifyLoginSuccess(): Promise<boolean> {
    // 检查localStorage标识
    const hasLogin = await this.page.evaluate(() => {
      return localStorage.getItem('flow_web_has_login') === 'true';
    });

    return hasLogin;
  }

  async query(question: string): Promise<QueryResult> {
    // 导航到聊天页面
    await this.page.goto('https://www.doubao.com/chat');

    // 输入问题
    await this.page.type('#chat-input', question);

    // 发送
    await this.page.click('#send-btn');

    // 等待响应
    await this.page.waitForSelector('.response-text');

    // 提取结果
    const answer = await this.page.$eval('.response-text', el => el.textContent);

    return {
      requestId: generateRequestId(),
      providerId: this.id,
      question,
      answer,
      references: await this.extractReferences(),
      executionTime: Date.now() - startTime,
      status: 'success',
      createdAt: new Date()
    };
  }
}
```

---

## 7. CLI命令设计

### 7.1 命令列表

```bash
# 登录命令
ai-compare login <provider>
ai-compare login doubao
ai-compare login all              # 登录所有配置的AI

# 查询命令
ai-compare query "<question>" [options]
ai-compare query "深圳宠物医院推荐" --apps doubao,kimi
ai-compare query "Python数据分析" --apps all --mode test

# 批量查询
ai-compare batch --input <file> [options]
ai-compare batch --input questions.csv --apps doubao,kimi
ai-compare batch --input questions.csv --apps all --parallel

# 状态查询
ai-compare status                 # 查看所有AI登录状态
ai-compare status doubao          # 查看特定AI状态

# 会话管理
ai-compare session list           # 列出所有会话
ai-compare session cleanup        # 清理过期会话
ai-compare logout <provider>      # 退出登录

# 历史记录
ai-compare history [options]      # 查看历史查询
ai-compare history --date 2025-10-13
ai-compare history --provider doubao

# 配置管理
ai-compare config list            # 列出所有配置
ai-compare config add <provider>  # 添加新AI
```

### 7.2 交互式流程示例

```bash
$ ai-compare query "深圳宠物医院推荐"

🤔 检测到未指定AI平台，请选择：
? 选择AI平台 (空格选择，回车确认)
  ◉ 豆包 (已登录)
  ◉ Kimi (已登录)
  ◯ DeepSeek (未登录)

? 是否并行执行？(Y/n) Y

🔄 正在查询...
  ✓ 豆包 [2.3s]
  ✓ Kimi [1.8s]

╔═══════════════════════════════════════════╗
║ 豆包                                      ║
╠═══════════════════════════════════════════╣
║ 推荐以下几家深圳宠物医院：                ║
║ 1. 深圳瑞派宠物医院（福田区）            ║
║ ...                                       ║
╚═══════════════════════════════════════════╝

? 是否保存到数据库？(Y/n) Y

✅ 结果已保存 (request_id: req_20251013_001)
```

---

## 8. 从test1迁移策略

### 8.1 迁移清单

| 源文件 | 目标文件 | 迁移方式 | 调整内容 |
|--------|---------|---------|---------|
| `DoubaoLoginManager.ts` (405行) | `providers/doubao/DoubaoLogin.ts` | 直接拷贝 | 修改import路径，实现IAIProvider |
| `DoubaoLoginDetector.ts` (150行) | 整合到DoubaoLogin | 合并 | 作为DoubaoLogin的内部方法 |
| `LoginSessionManager.ts` (300行) | `services/session/SessionService.ts` | 直接拷贝 | 重命名类名，改import路径 |
| `SessionStorage.ts` (200行) | `services/session/SessionStorage.ts` | 直接拷贝 | 几乎不变 |
| `DoubaoCliUI.ts` (500行) | `cli/components/UI.ts` | 提取通用部分 | 去豆包化，改为通用组件 |
| `doubao-query-test.mjs` (150行) | `providers/doubao/DoubaoQuery.ts` | 参考重写 | 改为类方法，TypeScript化 |
| `doubao-batch-query.mjs` (100行) | `services/QueryService.ts` | 参考重写 | 改为通用批量执行器 |

### 8.2 迁移步骤

```bash
# 1. 拷贝登录相关代码
cp ../test1/chrome-devtools-mcp/src/doubao/DoubaoLoginManager.ts \
   ./src/providers/doubao/DoubaoLogin.ts

# 2. 拷贝会话管理代码
cp ../test1/chrome-devtools-mcp/src/session/*.ts \
   ./src/services/session/

# 3. 调整import路径（手动）
# 原来：import { LoginSessionManager } from '../session/LoginSessionManager.js';
# 改为：import { SessionService } from '../../services/session/SessionService.js';

# 4. 实现IAIProvider接口
# 在DoubaoProvider.ts中整合Login和Query逻辑
```

---

## 9. 测试策略

### 9.1 测试金字塔

```
        /\
       /E2E\          15% - 端到端测试
      /------\
     /Integration\    30% - 集成测试
    /------------\
   /  Unit Tests  \   50% - 单元测试
  /----------------\
        + 5% 手动测试
```

### 9.2 测试覆盖目标

- **单元测试**: 覆盖率≥70%，关键路径100%
- **集成测试**: 每个Provider至少3个场景
- **E2E测试**: 核心用户流程全覆盖

### 9.3 关键测试场景

```typescript
// Provider单元测试
describe('DoubaoProvider', () => {
  it('should login successfully with valid credentials', async () => {
    // ...
  });

  it('should return query result with references', async () => {
    // ...
  });

  it('should restore session from storage', async () => {
    // ...
  });
});

// 并行执行测试
describe('ParallelQueryExecutor', () => {
  it('should execute 3 queries in parallel faster than serial', async () => {
    const parallel = await executor.executeParallel(question, ['doubao', 'kimi', 'deepseek']);
    expect(parallel.totalTime).toBeLessThan(serialTime * 0.5);
  });
});

// 数据库测试
describe('DatabaseService', () => {
  it('should save query result with references', async () => {
    // ...
  });
});
```

---

## 10. 安全考虑

### 10.1 会话数据安全

- ✅ Cookie加密存储（使用Node.js crypto模块）
- ✅ 本地存储，不上传云端
- ✅ 会话自动过期（默认14天）
- ✅ 敏感信息脱敏（日志中隐藏token）

### 10.2 输入验证

```typescript
// 防止注入攻击
function sanitizeInput(input: string): string {
  return input
    .replace(/<script[^>]*>.*?<\/script>/gi, '')
    .replace(/[<>'"]/g, '')
    .trim();
}
```

---

## 11. 性能优化

### 11.1 优化策略

1. **浏览器复用**: 同一Provider复用浏览器实例
2. **连接池**: 维护数据库连接池，避免频繁创建
3. **缓存机制**: 缓存Provider配置、会话状态
4. **懒加载**: 按需加载Provider模块

### 11.2 性能指标

- 单次查询响应时间: ≤5秒
- 批量查询（10个问题）: ≤30秒
- 内存占用（单AI）: ≤500MB
- 数据库查询: ≤100ms

---

## 12. 可扩展性设计

### 12.1 新增AI平台步骤

```bash
# 1. 创建Provider目录
mkdir src/providers/new-ai

# 2. 实现Provider类
# src/providers/new-ai/NewAIProvider.ts
export class NewAIProvider extends BaseProvider {
  readonly id = 'new-ai';
  readonly name = '新AI';
  readonly loginUrl = 'https://new-ai.com';

  // 实现抽象方法
  protected async fillLoginForm() { ... }
  protected async verifyLoginSuccess() { ... }
  async query(question: string) { ... }
}

# 3. 注册到Provider工厂
// src/providers/ProviderFactory.ts
providers.set('new-ai', NewAIProvider);

# 4. 添加配置文件
# config/ai-providers/new-ai.json
{
  "id": "new-ai",
  "name": "新AI",
  "loginUrl": "https://new-ai.com",
  "selectors": { ... }
}

# 5. 编写测试
# tests/unit/providers/new-ai.test.ts
```

**耗时估算**: 2-4小时（熟悉后可缩短到1小时）

---

## 13. 技术选型总结

| 技术 | 版本 | 用途 | 选择原因 |
|------|------|------|---------|
| TypeScript | 5.x | 编程语言 | 类型安全，工程化强 |
| Node.js | 20+ | 运行时 | LTS版本，生态成熟 |
| Puppeteer | latest | 浏览器自动化 | 官方支持，功能强大 |
| chrome-devtools-mcp | latest | MCP服务器 | 统一接口，易于扩展 |
| SQLite | 3.x | 数据库 | 轻量，本地部署 |
| Commander.js | 11.x | CLI框架 | 社区标准，API简洁 |
| Inquirer.js | 9.x | 交互提示 | 丰富的交互组件 |
| Chalk | 5.x | 终端颜色 | 易用，跨平台 |
| Ora | 7.x | 进度动画 | 美观的Spinner |
| Jest | 29.x | 测试框架 | 功能完整，快照测试 |

---

## 14. 未来规划

### Phase 3: 扩展功能
- [ ] 添加更多AI平台（文心一言、通义千问、元宝）
- [ ] 导出功能（CSV、JSON、Markdown、PDF）
- [ ] 高级筛选和搜索
- [ ] 结果评分系统
- [ ] 监控和日志系统

### Phase 4: 优化与生态
- [ ] Web UI（基于Electron）
- [ ] 插件系统（自定义Provider）
- [ ] 云端同步（可选）
- [ ] API接口（供第三方调用）

---

## 附录

### A. 术语表

- **Provider**: AI提供商，如豆包、Kimi
- **Session**: 登录会话，包含Cookie、LocalStorage等
- **Request ID**: 同一批次查询的唯一标识
- **Query**: 向AI提问的单次操作
- **Batch Query**: 批量提问操作
- **Parallel Mode**: 并行执行模式

### B. 参考资料

- chrome-devtools-mcp文档: https://github.com/modelcontextprotocol/servers
- Puppeteer API: https://pptr.dev/
- SQLite文档: https://www.sqlite.org/docs.html
- Commander.js: https://github.com/tj/commander.js

---

**Last Updated**: 2025-10-13 16:30
**Version**: v1.0
**Status**: 设计完成，待开发
