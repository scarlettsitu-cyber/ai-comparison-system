# 架构设计文档

> **本文档描述ai-comparison-system的整体架构、技术选型和设计决策**

## 📋 文档信息

- **版本**: v1.1
- **创建日期**: 2025-10-13
- **最后更新**: 2025-10-18
- **负责人**: Claude Code + situruimin

---

## 1. 系统概述

### 1.1 项目定位

**ai-comparison-system**是一个TUI（Terminal User Interface）工具，用于同时向多个AI平台（豆包、Kimi、DeepSeek等）提问并对比结果，帮助用户快速获取不同AI的观点和答案。

**界面特点**：
- 在终端内运行，无需浏览器
- 使用ASCII字符绘制界面（框、表格、进度条）
- 支持键盘导航（上下键选择、空格多选、Tab切换）
- 6个精心设计的交互界面，覆盖完整工作流

### 1.2 核心价值

- 🚀 **效率提升**: 一次提问，多个AI同时响应
- 🖥️ **用户友好**: TUI界面，终端内完成所有交互，无需浏览器
- 🔍 **结果对比**: 并排展示，快速发现差异
- 💾 **数据积累**: 持久化存储，支持历史回溯
- 🤖 **智能管理**: 自动会话管理，无需重复登录
- ⚡ **灵活并发**: 三种执行模式，自动推荐最优方案
- 🎯 **双输出模式**: 演示模式测试、写库模式生产

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
│                      TUI Interface Layer                        │
│              (blessed + cli-table3 + chalk + ora)               │
│   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│   │ Level 0  │ │Level 1-A │ │Level 1-B │ │Level 2-3 │          │
│   │  主界面   │ │ AI管理   │ │ 任务管理  │ │ 执行结果 │          │
│   └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
└─────────────────────┬───────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                    Service Layer                                │
│  ┌─────────────────┐ ┌──────────────┐ ┌────────────────────┐   │
│  │  QueryService   │ │SessionService│ │ ComparisonService  │   │
│  │  (查询编排)      │ │(会话管理)     │ │   (结果对比)        │   │
│  │  - 串行模式      │ │              │ │                    │   │
│  │  - 并行方案A/B   │ │              │ │                    │   │
│  │  - 自动选择      │ │              │ │                    │   │
│  └─────────────────┘ └──────────────┘ └────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  ResourceEvaluator (系统资源评估)                        │  │
│  │  - 内存检测  - CPU检测  - 并发数计算  - 自动推荐        │  │
│  └──────────────────────────────────────────────────────────┘  │
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

#### **Interface Layer (TUI)**
- **职责**: 用户交互、界面渲染、结果展示
- **技术**: blessed（TUI框架）、cli-table3（表格）、chalk（颜色）、ora（进度条）
- **关键组件**:
  - `screens/MainScreen.ts` - Level 0: 主界面
  - `screens/AIManagementScreen.ts` - Level 1-A: AI应用管理
  - `screens/TaskManagementScreen.ts` - Level 1-B: 任务管理
  - `screens/ExecutionScreen.ts` - Level 2: 执行中（演示/写库）
  - `screens/ResultScreen.ts` - Level 3: 结果展示
  - `components/` - 公共TUI组件（Header、Footer、Table等）

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

## 5. 执行模式设计

### 5.1 三种执行模式总览

为了适应不同的系统资源和性能需求，系统提供三种执行模式供用户选择：

| 模式 | 使用场景 | 速度 | 内存消耗 | 实现方式 |
|------|---------|------|---------|---------|
| **串行模式** | 系统资源受限 | ⭐ | 低 (~500MB) | 逐个执行，保底方案 |
| **方案A: 多AI并行** | 多AI少问题 | ⭐⭐⭐ | 中高 (~1.5GB) | 多个MCP实例 |
| **方案B: 单AI多问题** | 单AI多问题 | ⭐⭐⭐⭐⭐ | 中 (~1.5GB) | Puppeteer多Page |
| **自动选择** | 默认推荐 | 自适应 | 自适应 | 系统资源评估 |

**设计原则**：
- ✅ 串行模式作为保底，保证所有环境都能运行
- ✅ 两种并行方案逐个开发，每个都需用户确认效果
- ✅ 自动选择模式根据系统资源智能推荐
- ✅ 用户可手动覆盖系统推荐

---

### 5.2 模式1：串行模式（保底方案）

#### 执行流程
```
问题1 → 豆包 → 完成 → 写库
     → Kimi  → 完成 → 写库
     → DeepSeek → 完成 → 写库
问题2 → 豆包 → 完成 → 写库
     → Kimi  → 完成 → 写库
     ...
```

#### 实现代码
```typescript
export class SerialQueryExecutor {
  async execute(questions: string[], providers: IAIProvider[]): Promise<QueryResult[]> {
    const results: QueryResult[] = [];

    for (const question of questions) {
      for (const provider of providers) {
        try {
          const result = await provider.query(question);
          results.push(result);
          await this.databaseService.save(result); // 立即写库
        } catch (error) {
          this.logError(provider.id, question, error);
        }
      }
    }

    return results;
  }
}
```

#### 性能指标
- **3个AI × 3个问题 = 9次查询**
- **耗时**: ~45秒 (每次查询5秒)
- **内存**: ~500MB (单个浏览器实例)
- **优势**: 资源消耗低，稳定性高
- **劣势**: 速度最慢

---

### 5.3 模式2：方案A - 多AI应用并行

#### 技术挑战
chrome-devtools-mcp默认使用Mutex强制串行：
```typescript
// chrome-devtools-mcp/src/main.ts
const toolMutex = new Mutex();
await toolMutex.acquire();  // 获取锁，串行执行
```

#### 解决方案
使用`--isolated`参数启动多个独立的MCP实例：

```typescript
export class MultiAIParallelExecutor {
  private mcpInstances: Map<string, MCPInstance> = new Map();

  async initialize(providers: string[]) {
    // 为每个Provider启动独立的MCP服务器
    for (const providerId of providers) {
      const instance = await this.launchMCPServer(providerId);
      this.mcpInstances.set(providerId, instance);
    }
  }

  private async launchMCPServer(providerId: string): Promise<MCPInstance> {
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

  async executeParallel(questions: string[], providers: string[]): Promise<QueryResult[]> {
    const results: QueryResult[] = [];

    for (const question of questions) {
      // 每个问题，多个AI并行执行
      const promises = providers.map(async (providerId) => {
        const instance = this.mcpInstances.get(providerId);
        return await this.queryWithInstance(instance, question);
      });

      const batchResults = await Promise.all(promises);
      results.push(...batchResults);
    }

    return results;
  }
}
```

#### 性能指标
- **3个AI × 3个问题 = 9次查询**
- **耗时**: ~15秒 (3个AI并行，每轮5秒)
- **内存**: ~1.5GB (3个MCP实例)
- **优势**: 多AI场景提速3倍
- **劣势**: 内存消耗高，多问题场景提速有限

#### ⚠️ 开发注意事项
- 需要用户确认"同时打开3个浏览器窗口"的效果
- TUI界面实时显示"正在启动豆包实例..."
- 测试内存占用是否在预期范围内

---

### 5.4 模式3：方案B - 单AI多问题并行

#### 技术原理
使用Puppeteer的多Page功能，在同一个浏览器实例中并行执行：

```typescript
export class SingleAIMultiQuestionExecutor {
  async execute(questions: string[], provider: IAIProvider): Promise<QueryResult[]> {
    const browser = await provider.getBrowser();

    // 为每个问题创建独立的Page
    const promises = questions.map(async (question) => {
      const page = await browser.newPage();
      try {
        const result = await this.queryWithPage(page, provider, question);
        return result;
      } finally {
        await page.close();
      }
    });

    return await Promise.all(promises);
  }

  private async queryWithPage(
    page: Page,
    provider: IAIProvider,
    question: string
  ): Promise<QueryResult> {
    // 导航到AI聊天页面
    await page.goto(provider.chatUrl);

    // 输入问题
    await page.type('#chat-input', question);
    await page.click('#send-btn');

    // 等待响应
    await page.waitForSelector('.response-text');
    const answer = await page.$eval('.response-text', el => el.textContent);

    return {
      requestId: generateRequestId(),
      providerId: provider.id,
      question,
      answer,
      executionTime: Date.now() - startTime,
      status: 'success',
      createdAt: new Date()
    };
  }
}
```

#### 性能指标
- **10个问题同时执行（豆包）**
- **耗时**: ~5秒 (所有问题并行)
- **内存**: ~1.5GB (单浏览器10个Page)
- **优势**: 单AI多问题场景提速10倍！
- **劣势**: 并发数受CPU限制

#### ⚠️ 开发注意事项
- 需要用户确认"同时打开10个标签页"的效果
- TUI界面实时显示"Page 1/10执行中..."
- 测试浏览器崩溃风险（建议最多20并发）

---

### 5.5 模式4：自动选择（智能推荐）

#### 系统资源评估算法

```typescript
export class ResourceEvaluator {
  /**
   * 计算推荐的并发数
   */
  async recommendConcurrency(): Promise<RecommendationResult> {
    const freeMemory = os.freemem(); // 可用内存（字节）
    const totalCPU = os.cpus().length; // CPU核心数

    // 公式：推荐并发数 = min(内存限制, CPU限制, 问题总数)
    const memoryLimit = Math.floor((freeMemory - 2 * 1024 * 1024 * 1024) / (150 * 1024 * 1024));
    // 每个Page约150MB，预留2GB给系统

    const cpuLimit = totalCPU * 2; // CPU核心数 × 2

    const recommendedConcurrency = Math.max(1, Math.min(memoryLimit, cpuLimit));

    return {
      freeMemory: (freeMemory / 1024 / 1024 / 1024).toFixed(2) + 'GB',
      cpuCores: totalCPU,
      recommendedConcurrency,
      recommendedMode: this.selectMode(recommendedConcurrency)
    };
  }

  /**
   * 根据并发数选择模式
   */
  private selectMode(concurrency: number): ExecutionMode {
    if (concurrency <= 1) {
      return 'serial'; // 串行模式
    } else if (concurrency >= 10) {
      return 'single-ai-multi-question'; // 方案B
    } else {
      return 'multi-ai-parallel'; // 方案A
    }
  }
}
```

#### 推荐策略

| 可用内存 | CPU核心 | 推荐模式 | 推荐并发数 |
|---------|--------|---------|-----------|
| < 4GB | 任意 | 串行模式 | 1 |
| 4-8GB | 2-4核 | 方案A | 3个AI并行 |
| > 8GB | 4核+ | 方案B | 10问题并发 |
| > 16GB | 8核+ | 方案B | 20问题并发 |

#### TUI界面展示
```
┌──────────────────────────────────────┐
│ 系统资源评估：                        │
│ ├─ 可用内存: 6.5GB / 16GB           │
│ ├─ CPU核心: 4核                     │
│ ├─ 推荐并发: 10个问题并发            │
│ └─ 推荐模式: 方案B (单AI多问题并行)  │
└──────────────────────────────────────┘
```

---

### 5.6 性能对比总表

| 场景 | 串行模式 | 方案A | 方案B | 最优选择 |
|------|---------|-------|-------|---------|
| 3AI × 3问题 | 45秒 | 15秒 | 15秒 | 方案A/B |
| 3AI × 10问题 | 150秒 | 50秒 | 30秒 | 方案B ⭐ |
| 5AI × 10问题 | 250秒 | 50秒 | 50秒 | 方案A/B |
| 1AI × 20问题 | 100秒 | 100秒 | 5秒 | 方案B ⭐⭐⭐ |

**结论**：
- 多AI少问题 → 方案A
- 单AI多问题 → 方案B（速度提升最显著）
- 资源受限 → 串行模式

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

## 7. TUI界面设计总览

### 7.1 界面层级关系

系统共有6个TUI界面，按层级组织：

```
Level 0: 主界面（启动后第一屏）
   ├─ 1. AI应用管理 → Level 1-A
   ├─ 2. 开始任务 → Level 1-B
   ├─ 3. 历史记录
   └─ 4. 退出

Level 1-A: AI应用管理
   ├─ 查看AI列表（状态、登录、到期）
   ├─ 登录/更新操作
   └─ [开始任务→] → Level 1-B

Level 1-B: 任务管理
   ├─ 步骤1: 选择AI（多选）
   ├─ 步骤2: 输入问题（多行）
   ├─ 步骤3: 执行模式（串行/方案A/方案B/自动）
   ├─ 步骤4: 输出模式（演示/写库）⭐
   └─ [开始执行] →
        ├─ 演示模式 → Level 2-A → Level 3-A
        └─ 写库模式 → Level 2-B → Level 3-B

Level 2-A: 执行中（演示模式）
   ├─ 实时进度条（AI×问题矩阵）
   ├─ 实时日志滚动
   └─ 自动跳转 → Level 3-A

Level 2-B: 执行中（写库模式）
   ├─ 实时进度条 + 数据库写入标识（💾）
   ├─ 写入统计
   └─ 自动跳转 → Level 3-B

Level 3-A: 结果展示（演示模式）
   ├─ 逐个展示AI回复（← → 切换问题）
   ├─ 并排对比视图
   └─ 手动保存/导出/返回

Level 3-B: 完成总结（写库模式）
   ├─ 完成统计（成功/失败/耗时）
   ├─ 失败任务列表 + 重试
   └─ 查看详情/新任务/返回
```

**详细设计请参阅**: `docs/06-TUI-DESIGN/` 文件夹（包含11个详细设计文档）

---

### 7.2 核心技术栈

| 技术 | 用途 | 示例 |
|------|------|------|
| **blessed** | TUI框架 | 创建窗口、布局、组件 |
| **cli-table3** | 表格展示 | AI列表、进度矩阵 |
| **chalk** | 文本颜色 | 绿色✓、红色✗、黄色⚠️ |
| **ora** | 进度动画 | Spinner、进度条 |
| **boxen** | 边框绘制 | 弹窗、警告框 |

---

### 7.3 键盘交互规范

#### 全局快捷键（所有界面通用）
- **Q**: 快速返回主界面
- **Ctrl+C**: 取消执行/退出程序
- **↑ ↓**: 上下选择
- **Enter**: 确认/下一步

#### 上下文快捷键（特定界面）

**Level 1-B: 任务管理**
- **Tab**: 切换焦点区域（4个步骤间切换）
- **Space**: 多选勾选/取消
- **→**: 开始执行
- **←**: 返回上一步

**Level 3-A: 结果展示（演示模式）**
- **← →**: 切换问题
- **S**: 保存当前结果到数据库
- **C**: 复制全部结果
- **V**: 切换并排对比视图

**Level 3-B: 失败任务处理**
- **R**: 重试所有失败任务
- **↑ ↓**: 选择单个失败任务
- **Enter**: 单独处理选中任务

---

### 7.4 演示模式 vs 写库模式对比

| 特性 | 演示模式 | 写库模式 |
|------|---------|---------|
| **触发位置** | Level 1-B步骤4 | Level 1-B步骤4 |
| **执行界面** | Level 2-A | Level 2-B |
| **数据写入** | ❌ 不写库（缓存在内存） | ✅ 立即写库（每完成一个） |
| **进度显示** | 状态图标 | 状态图标 + 💾写库标识 |
| **结果展示** | Level 3-A逐个展示 | Level 3-B完成总结 |
| **失败处理** | 显示错误，不保存 | 记录失败任务，可重试 |
| **手动保存** | ✅ 支持手动保存 | ❌ 已自动保存 |
| **导出功能** | ✅ 导出CSV/JSON | ✅ 导出CSV/JSON |
| **使用场景** | 测试、演示、快速查看 | 生产、积累数据 |

---

### 7.5 错误处理和提示设计

#### 错误类型

**1. 登录错误**
```
┌──────────────────────────────────────┐
│ ⚠️  登录失败                          │
├──────────────────────────────────────┤
│ AI: 豆包                              │
│ 错误: 验证码过期，请重新发送          │
│                                      │
│ [重试] [跳过] [返回]                  │
└──────────────────────────────────────┘
```

**2. 查询错误**
```
┌──────────────────────────────────────┐
│ ✗ 查询失败                            │
├──────────────────────────────────────┤
│ AI: Kimi                              │
│ 问题: Python数据分析入门              │
│ 错误: 等待响应超时 (30秒)            │
│ 日志: /logs/error_20251018_001.log  │
│                                      │
│ [重试] [查看日志] [跳过]              │
└──────────────────────────────────────┘
```

**3. 系统资源不足**
```
┌──────────────────────────────────────┐
│ ⚠️  系统资源不足                      │
├──────────────────────────────────────┤
│ 可用内存: 1.5GB                       │
│ 推荐并发: 1个（串行模式）             │
│                                      │
│ 您选择了"方案B: 10问题并发"           │
│ 可能导致系统卡顿或崩溃                │
│                                      │
│ [使用推荐] [强制执行] [返回]          │
└──────────────────────────────────────┘
```

---

### 7.6 TUI界面示例（Level 0: 主界面）

```
┌──────────────────────────────────────────────────────────────────┐
│                  AI比较系统 v1.0                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  系统状态:                                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 已登录AI: 2/4  (豆包 ✓、Kimi ✓、DeepSeek ✗、文心一言 ✗)   │ │
│  │ 内存可用: 6.5GB / 16GB                                     │ │
│  │ 推荐并发: 方案B (单AI多问题并行10个)                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  请选择操作:                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  ❯ 1. AI应用管理          (登录、更新、删除AI)             │ │
│  │    2. 开始任务            (需要至少1个AI已登录)             │ │
│  │    3. 历史记录            (查看过往查询)                   │ │
│  │    4. 系统设置            (配置、日志)                     │ │
│  │    5. 退出                                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ⚠️  提示: 只有2个AI已登录，建议先进入"AI应用管理"登录更多AI    │
│                                                                  │
│  ↑↓ 选择  │  Enter 确认  │  Q 退出                              │
└──────────────────────────────────────────────────────────────────┘
```

**更多界面设计**: 请查阅 `docs/06-TUI-DESIGN/01-level-0-main.md` 等文件

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
| blessed | 0.1.x | TUI框架 | 成熟的终端UI库，支持窗口/布局/组件 |
| cli-table3 | 0.6.x | 表格渲染 | 终端表格展示，样式丰富 |
| Chalk | 5.x | 终端颜色 | 易用，跨平台 |
| Ora | 7.x | 进度动画 | 美观的Spinner |
| boxen | 7.x | 边框绘制 | 终端弹窗、警告框 |
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
- blessed文档: https://github.com/chjj/blessed
- cli-table3: https://github.com/cli-table/cli-table3

---

**Last Updated**: 2025-10-18 20:45
**Version**: v1.1
**Status**: TUI架构设计完成，待开发
