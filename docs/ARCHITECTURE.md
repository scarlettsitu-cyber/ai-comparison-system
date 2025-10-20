# 项目架构文档

> **MVP架构说明 + 延后模块标记**

---

## 📂 MVP目录结构（Phase 1）

```
ai-comparison-system/
├── docs/                           # 📚 文档目录
│   ├── CLAUDE.md                   # ⭐ Claude Code项目说明书（精简版）
│   ├── 01-DESIGN.md                # 架构设计和技术方案
│   ├── 02-TASKS.md                 # 任务清单（MVP优先）
│   ├── 03-WORKFLOW.md              # 执行规范
│   ├── 04-CHANGELOG.md             # 变更记录
│   ├── 05-CONTEXT.md               # 会话上下文（断点记录）
│   ├── DECISIONS.md                # 技术决策记录
│   ├── DEV-STANDARDS.md            # 开发规范
│   ├── ARCHITECTURE.md             # 本文档
│   └── 06-TUI-DESIGN/              # TUI界面详细设计（11个文件）
│
├── src/
│   ├── core/                       # ✅ Phase 1: 核心接口
│   │   ├── interfaces/
│   │   │   ├── IAIProvider.ts      # AI Provider统一接口
│   │   │   ├── QueryResult.ts      # 查询结果接口
│   │   │   └── SessionData.ts      # 会话数据接口
│   │   └── types/
│   │       └── index.ts            # 类型定义
│   │
│   ├── providers/                  # ✅ Phase 1: 豆包Provider
│   │   ├── CLAUDE.md               # Provider开发规范
│   │   ├── doubao/                 # 豆包（从test1迁移）
│   │   │   ├── DoubaoProvider.ts
│   │   │   ├── DoubaoLogin.ts
│   │   │   └── DoubaoQuery.ts
│   │   │
│   │   ├── kimi/                   # ❌ Phase 2: Kimi（延后）
│   │   │   └── .gitkeep
│   │   │
│   │   ├── deepseek/               # ❌ Phase 2: DeepSeek（延后）
│   │   │   └── .gitkeep
│   │   │
│   │   └── base/                   # ❌ Phase 2: 抽象基类（可选优化）
│   │       └── .gitkeep
│   │
│   ├── services/                   # ✅ Phase 1: 业务逻辑层
│   │   ├── CLAUDE.md               # Service层规范
│   │   ├── QueryService.ts         # 查询编排（仅串行模式）
│   │   │
│   │   ├── executors/              # ❌ Phase 4: 并行执行模式（延后）
│   │   │   └── .gitkeep
│   │   │
│   │   └── ResourceEvaluator.ts    # ❌ Phase 4: 资源评估器（延后）
│   │
│   ├── storage/                    # ✅ Phase 1: 数据持久化
│   │   └── DatabaseService.ts      # SQLite服务
│   │
│   └── tui/                        # ✅ Phase 1: TUI界面（5个必需界面）
│       ├── CLAUDE.md               # TUI开发规范
│       ├── index.ts                # TUI入口
│       ├── TUIManager.ts           # 核心管理器
│       │
│       ├── screens/                # 界面
│       │   ├── MainScreen.ts           # Level 0: 主界面
│       │   ├── AIManagementScreen.ts   # Level 1-A: AI管理
│       │   ├── TaskManagementScreen.ts # Level 1-B: 任务管理（简化版）
│       │   ├── ExecutionDBScreen.ts    # Level 2-B: 执行中（写库模式）
│       │   ├── ResultDBScreen.ts       # Level 3-B: 完成总结
│       │   │
│       │   ├── ExecutionDemoScreen.ts  # ❌ Phase 3: 演示模式（延后）
│       │   └── ResultDemoScreen.ts     # ❌ Phase 3: 结果展示（延后）
│       │
│       └── components/             # ❌ Phase 3: 公共组件库（延后）
│           └── .gitkeep
│
├── tests/                          # ✅ Phase 1: 测试
│   ├── unit/                       # 单元测试
│   ├── integration/                # 集成测试
│   └── e2e/                        # 端到端测试
│
├── config/                         # ❌ Phase 5: 配置文件（延后）
│   └── .gitkeep
│
├── package.json                    # ⏳ Phase 0.3: 依赖管理
├── tsconfig.json                   # ⏳ Phase 0.3: TypeScript配置
├── .gitignore                      # ⏳ Phase 0.3: Git忽略规则
└── README.md                       # ✅ Phase 0: 项目说明

图例：
✅ Phase 1 MVP核心功能
⏳ Phase 0.3 环境配置（下次会话）
❌ 延后到Phase 2/3/4/5
```

---

## 🏗️ 模块职责划分

### 1. core/ - 核心接口层

**职责**：定义所有模块的接口和类型
**关键文件**：
- `IAIProvider.ts`: 所有Provider必须实现的接口
  ```typescript
  interface IAIProvider {
    login(): Promise<boolean>;
    query(question: string): Promise<string>;
    logout(): Promise<void>;
    getStatus(): Promise<SessionStatus>;
  }
  ```
- `QueryResult.ts`: 查询结果数据结构
- `SessionData.ts`: 会话数据结构

**依赖**：无（最底层）

---

### 2. providers/ - AI平台适配器

**职责**：实现各AI平台的登录、查询、会话管理
**MVP范围**：仅实现豆包Provider（从test1迁移）

#### 豆包Provider模块（Phase 1）
- **DoubaoLogin.ts**: 登录逻辑（手机号验证码）
- **DoubaoQuery.ts**: 查询逻辑（发送问题、接收回复）
- **DoubaoProvider.ts**: 总入口（实现IAIProvider接口）

**依赖**：
- `chrome-devtools-mcp`（通过Puppeteer控制浏览器）
- `core/interfaces/IAIProvider`

**延后模块**：
- Kimi Provider（Phase 2）- 需先手动测试登录流程
- DeepSeek Provider（Phase 2）- 需先手动测试登录流程
- BaseProvider抽象类（Phase 2）- 可选优化

---

### 3. storage/ - 数据持久化层

**职责**：SQLite数据库的初始化、查询、保存
**关键文件**：
- `DatabaseService.ts`: 数据库服务（单例）

**数据表**（MVP最小化）:
```sql
-- 查询记录表
CREATE TABLE queries (
  id INTEGER PRIMARY KEY,
  question TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 查询结果表
CREATE TABLE results (
  id INTEGER PRIMARY KEY,
  query_id INTEGER NOT NULL,
  provider TEXT NOT NULL,  -- "doubao" | "kimi" | "deepseek"
  answer TEXT NOT NULL,
  response_time REAL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (query_id) REFERENCES queries(id)
);

-- 会话记录表
CREATE TABLE sessions (
  id INTEGER PRIMARY KEY,
  provider TEXT NOT NULL,
  cookie_data TEXT,
  expires_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**依赖**：
- `better-sqlite3`

---

### 4. services/ - 业务逻辑层

**职责**：编排查询流程、管理执行模式

#### QueryService（Phase 1）
**MVP范围**：仅实现串行执行模式
```typescript
class QueryService {
  // 单条查询（单AI、单问题）
  async querySingle(provider: IAIProvider, question: string): Promise<QueryResult>

  // 批量查询（多AI、多问题，串行执行）
  async queryBatch(providers: IAIProvider[], questions: string[]): Promise<QueryResult[]>
}
```

**执行流程**（串行模式）:
1. 遍历providers列表
2. 遍历questions列表
3. 调用provider.query()
4. 结果自动写入DatabaseService
5. 失败记录到failed_tasks表

**延后模块**：
- `executors/` 目录（Phase 4）
  - SerialQueryExecutor.ts（从QueryService抽取）
  - MultiAIParallelExecutor.ts（多AI并行）
  - SingleAIMultiQuestionExecutor.ts（单AI多问题并行）
- `ResourceEvaluator.ts`（Phase 4）- 系统资源评估器

---

### 5. tui/ - TUI界面层

**职责**：终端交互界面、用户输入、结果展示

#### MVP界面（Phase 1）- 仅5个必需界面

| 界面 | 文件 | 功能 | 状态 |
|------|------|------|------|
| **Level 0** | MainScreen.ts | 主菜单（AI管理、开始任务、退出） | ✅ Phase 1 |
| **Level 1-A** | AIManagementScreen.ts | AI列表、登录操作 | ✅ Phase 1 |
| **Level 1-B** | TaskManagementScreen.ts | 选AI、输入问题、确认执行（简化版） | ✅ Phase 1 |
| **Level 2-B** | ExecutionDBScreen.ts | 执行中（进度条 + 💾写库标识） | ✅ Phase 1 |
| **Level 3-B** | ResultDBScreen.ts | 完成总结（显示结果、失败任务） | ✅ Phase 1 |

**简化版TaskManagement**：
- 步骤1：选择AI（豆包）
- 步骤2：输入问题（单问题或多问题）
- 步骤3：确认执行（固定为串行模式 + 写库模式）

**延后界面**（Phase 3）：
- ExecutionDemoScreen.ts（演示模式执行）
- ResultDemoScreen.ts（演示模式结果展示）
- components/（公共组件库：Header、Footer、Table、ProgressBar、StatusIcon、ErrorDialog）

**依赖**：
- `blessed`（TUI框架）
- `cli-table3`（表格渲染）
- `chalk`（颜色）
- `ora`（进度条）
- `boxen`（边框）

---

## 🔄 依赖关系图

```
┌──────────────────────────────────────────────────────┐
│                      TUI Layer                       │
│  (MainScreen, AIManagementScreen, TaskManagement...)  │
└───────────────┬──────────────────────────────────────┘
                │
                │ 调用
                ↓
┌───────────────────────────────────────────────────────┐
│                  Services Layer                       │
│            (QueryService - 串行模式)                   │
└───────────┬───────────────────┬───────────────────────┘
            │                   │
            │ 调用               │ 写库
            ↓                   ↓
┌───────────────────┐    ┌──────────────────┐
│  Providers Layer  │    │  Storage Layer   │
│  (DoubaoProvider) │    │ (DatabaseService)│
└─────────┬─────────┘    └──────────────────┘
          │
          │ 实现接口
          ↓
┌───────────────────┐
│    Core Layer     │
│  (IAIProvider)    │
└───────────────────┘
```

---

## 🚀 启动流程（MVP）

1. **TUI启动**（`npm run start`）
   - `src/tui/index.ts` → `TUIManager.ts`
   - 显示MainScreen（主菜单）

2. **AI管理**（选项1）
   - 跳转到AIManagementScreen
   - 显示豆包状态（未登录/已登录）
   - L键触发登录 → DoubaoProvider.login()
   - 登录成功 → 保存会话到DatabaseService

3. **开始任务**（选项2）
   - 跳转到TaskManagementScreen
   - 步骤1：选择豆包（默认勾选）
   - 步骤2：输入问题（单问题或多问题CSV导入）
   - 步骤3：确认执行

4. **执行查询**
   - 跳转到ExecutionDBScreen
   - 调用QueryService.queryBatch()
   - 实时更新进度条和💾写库标识
   - 完成后自动跳转到ResultDBScreen

5. **查看结果**
   - ResultDBScreen显示：
     - 完成统计（成功/失败/总耗时）
     - 失败任务列表
     - R键重试所有失败任务
     - Enter键重试选中任务

---

## 🔑 关键设计模式

### 1. 适配器模式（Adapter Pattern）
**应用**：`providers/` 目录
- 每个AI平台实现统一的`IAIProvider`接口
- 解耦AI平台差异，上层无需关心具体实现

### 2. 单例模式（Singleton Pattern）
**应用**：`DatabaseService`、`TUIManager`
- 全局唯一实例，避免重复初始化

### 3. 策略模式（Strategy Pattern）
**应用**：执行模式（Phase 4延后）
- 串行模式、多AI并行、单AI多问题并行作为不同策略
- MVP阶段仅实现串行策略

### 4. 观察者模式（Observer Pattern）
**应用**：TUI实时更新
- QueryService执行时发出进度事件
- ExecutionScreen监听并更新进度条

---

## ⚠️ 延后功能清单

### Phase 2延后
- ❌ Kimi Provider
- ❌ DeepSeek Provider
- ❌ BaseProvider抽象类（可选）

### Phase 3延后
- ❌ 演示模式界面（ExecutionDemoScreen、ResultDemoScreen）
- ❌ 完整版TaskManagement（执行模式选择、输出模式选择）
- ❌ TUI公共组件库（6个组件）
- ❌ 错误处理完善（3种错误类型弹窗）

### Phase 4延后
- ❌ 系统资源评估器
- ❌ 多AI并行执行
- ❌ 单AI多问题并行执行
- ❌ 自动模式选择

### Phase 5延后
- ❌ 更多AI平台（文心一言、通义千问、元宝）
- ❌ 导出功能（CSV、JSON、Markdown、PDF）
- ❌ 高级筛选和搜索
- ❌ 监控和日志系统
- ❌ 插件系统

---

## 📏 代码规模预估（MVP）

| 模块 | 文件数 | 代码行数（估算） |
|------|--------|----------------|
| core/ | 4 | ~150行 |
| providers/doubao/ | 3 | ~500行（从test1迁移） |
| storage/ | 1 | ~200行 |
| services/ | 1 | ~150行 |
| tui/ | 7 | ~800行 |
| tests/ | 10+ | ~500行 |
| **总计** | **26+** | **~2300行** |

---

## 📚 延伸阅读

- **@docs/01-DESIGN.md** - 完整架构设计和技术方案
- **@docs/02-TASKS.md** - 任务清单（MVP优先）
- **@docs/06-TUI-DESIGN/** - TUI界面详细设计（11个文件）
- **@src/providers/CLAUDE.md** - Provider开发规范
- **@src/tui/CLAUDE.md** - TUI开发规范
- **@src/services/CLAUDE.md** - Service层规范

---

**Last Updated**: 2025-10-18 22:20
**Phase**: Phase 0.2 - 项目初始化（文档阶段）
**Next**: 更新CONTEXT.md → Git提交推送
