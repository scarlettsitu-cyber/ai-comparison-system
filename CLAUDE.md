# CLAUDE.md - 项目指南（必读）

> **本文件是Claude Code的项目说明书，每次打开新窗口会自动读取**

## 🚨 重要提醒

每次打开新窗口，必须先读取以下文档：
1. **本文件（CLAUDE.md）** - 自动读取
2. **docs/05-CONTEXT.md** - 上次会话断点
3. **docs/02-TASKS.md** - 当前任务状态
4. **docs/03-WORKFLOW.md** - 执行规范（必须遵守）

---

## 📋 项目概述

**项目名称**: ai-comparison-system
**项目目标**: CLI工具，支持向多个AI平台（豆包、Kimi、DeepSeek等）同时提问并对比结果
**技术栈**: TypeScript + Puppeteer + chrome-devtools-mcp + SQLite + Commander.js
**开发周期**: 2周MVP

### 核心功能
- ✅ 多AI平台会话管理（登录、保持、恢复）
- ✅ 单条查询支持（同时向多个AI提问）
- ✅ 批量查询支持（从CSV导入）
- ✅ 结果对比展示（并排显示）
- ✅ 数据库持久化（SQLite）
- ✅ 并行执行（可选，使用--isolated模式）

---

## 🎯 当前开发阶段

**Phase**: Phase 0 - 项目初始化
**进度**: 正在创建核心文档

✅ 已完成：
- 项目目录结构创建
- Git仓库初始化

⏳ 进行中：
- 核心文档编写

📋 待开发：
- Phase 1: MVP基础功能（2周）
- Phase 2: 高级功能（后续）

详见：**docs/02-TASKS.md**

---

## 📂 代码结构

```
ai-comparison-system/
├── docs/                       # 📚 核心文档目录
│   ├── 01-DESIGN.md           # 架构设计文档
│   ├── 02-TASKS.md            # 任务清单（checkbox追踪）
│   ├── 03-WORKFLOW.md         # ⭐ 执行规范（必读）
│   ├── 04-CHANGELOG.md        # 变更记录
│   └── 05-CONTEXT.md          # 会话上下文（断点记录）
│
├── src/
│   ├── core/
│   │   └── interfaces/
│   │       └── IAIProvider.ts     # ⭐ 所有Provider必须实现此接口
│   │
│   ├── providers/              # AI平台适配器
│   │   ├── base/
│   │   │   └── BaseProvider.ts    # 抽象基类（通用逻辑）
│   │   ├── doubao/                # 豆包 (从test1迁移)
│   │   │   ├── DoubaoProvider.ts
│   │   │   ├── DoubaoLogin.ts
│   │   │   └── DoubaoQuery.ts
│   │   ├── kimi/                  # Kimi (待开发)
│   │   └── deepseek/              # DeepSeek (待开发)
│   │
│   ├── services/               # 业务逻辑层
│   │   ├── session/
│   │   │   ├── SessionService.ts  # 会话管理
│   │   │   └── SessionStorage.ts  # 会话存储
│   │   ├── QueryService.ts        # 查询编排
│   │   └── DatabaseService.ts     # 数据持久化
│   │
│   ├── cli/                    # CLI命令
│   │   └── commands/
│   │       ├── login.ts
│   │       ├── query.ts
│   │       ├── batch.ts
│   │       └── status.ts
│   │
│   └── storage/                # 数据层
│       └── DatabaseService.ts
│
├── tests/
│   ├── unit/                   # 单元测试
│   ├── integration/            # 集成测试
│   └── e2e/                    # 端到端测试
│
├── config/
│   └── ai-providers/           # AI平台配置文件
│       ├── doubao.json
│       ├── kimi.json
│       └── deepseek.json
│
├── CLAUDE.md                   # ⭐ 本文件
├── README.md                   # 项目说明
├── package.json                # 依赖管理
├── tsconfig.json               # TypeScript配置
└── .gitignore                  # Git忽略规则
```

---

## 🔑 关键决策记录

### [决策 #001] 2025-10-13 - 适配器模式支持多AI
**问题**：如何支持多个AI平台？
**方案**：每个AI实现统一的`IAIProvider`接口
**原因**：解耦、易扩展、测试友好
**影响**：所有Provider必须实现login()、query()、logout()方法

### [决策 #002] 2025-10-13 - SQLite数据库
**问题**：如何存储查询结果？
**方案**：SQLite本地数据库
**原因**：轻量、无需额外服务、跨平台、便于备份
**影响**：结果存储在本地.db文件中

### [决策 #003] 2025-10-13 - 并行执行方案
**问题**：如何实现真正的并行查询？
**方案**：使用`--isolated`启动多个chrome-devtools-mcp服务器实例
**原因**：chrome-devtools-mcp默认使用Mutex串行执行，isolated模式可并行
**影响**：并行模式内存消耗增加（每个实例~500MB）

### [决策 #004] 2025-10-13 - 项目独立安装chrome-devtools-mcp
**问题**：是否复用test1的chrome-devtools-mcp？
**方案**：新项目通过npm独立安装标准版chrome-devtools-mcp
**原因**：test1的版本已被修改（混入业务代码），新项目需要干净的架构
**影响**：test1的豆包代码需要拷贝到新项目并调整import路径

### [决策 #005] 2025-10-13 - 文档驱动的跨窗口协作
**问题**：如何避免多个Claude窗口导致项目失控？
**方案**：6个核心文档 + 三大Checklist + 严格的执行规范
**原因**：保证一致性、可追溯、质量可控
**影响**：每次开发前后都要执行规范的checklist操作

---

## 📜 开发规范（必须遵守）

### TypeScript规范
- ✅ 启用严格模式（`strict: true`）
- ✅ 所有函数必须有类型注解
- ✅ 禁止使用`any`（允许`unknown`）
- ✅ 接口命名使用`I`前缀（如`IAIProvider`）

### 测试规范
- ✅ 代码覆盖率≥70%
- ✅ 关键路径100%覆盖
- ✅ 每个Provider至少3个单元测试
- ✅ 测试命名格式：`describe('ComponentName', () => { it('should do something', ...) })`

### Commit规范
- ✅ 格式：`<type>(<scope>): <subject>`
- ✅ 每完成一个Task checkbox就commit
- ✅ 详细的commit body（包含测试、影响、下一步）
- ✅ Type类型：feat, fix, docs, refactor, test, chore

示例：
```bash
git commit -m "feat(doubao): 完成登录功能

- 实现手机号验证码登录
- 添加会话状态检测
- 集成到Provider接口

测试: 3个单元测试通过
TASKS: 完成 [1.2] 豆包Provider开发
NEXT: 开始Kimi Provider开发"
```

### 文档规范
- ✅ 代码变更必须同步更新文档
- ✅ 弃用代码必须标记`@deprecated`
- ✅ 重大变更必须记录到`CHANGELOG.md`
- ✅ 每次会话结束更新`CONTEXT.md`

---

## 🔄 执行流程（Checklist）

详见：**docs/03-WORKFLOW.md**

### 快速检查清单

#### ✅ 新窗口启动时
- [ ] Claude自动读取CLAUDE.md
- [ ] 手动检查`docs/05-CONTEXT.md`（上次断点）
- [ ] 手动检查`docs/02-TASKS.md`（当前任务）
- [ ] 运行`npm test`确认测试通过
- [ ] 运行`git status`确认工作区干净
- [ ] 向用户确认："上次进度到[XXX]，下一步是[YYY]，准备开始了吗？"

#### ✅ 完成任务时
- [ ] 运行`npm run typecheck`（TypeScript检查）
- [ ] 运行`npm run format`（代码格式化）
- [ ] 运行`npm test`（所有测试通过）
- [ ] 更新`docs/02-TASKS.md`（勾选checkbox）
- [ ] 更新`docs/05-CONTEXT.md`（记录当前状态）
- [ ] Git commit（规范的message）
- [ ] 向用户汇报："✅ 已完成[X.X]，📝已更新文档，💾已提交Git"

#### ✅ 结束窗口前
- [ ] 所有修改已保存
- [ ] 所有代码已提交到Git
- [ ] 更新`docs/05-CONTEXT.md`（明确下次从哪里继续）
- [ ] 运行`npm run build`（确保能编译）
- [ ] 运行`npm test`（确保所有测试通过）
- [ ] 运行`git status`（确保工作区干净）
- [ ] 向用户确认："本次会话已结束，进度已保存，可以安全关闭窗口"

---

## 🚫 禁止事项

- ❌ 不读文档就开始开发
- ❌ 不经用户同意修改核心架构
- ❌ 跳过测试直接提交代码
- ❌ 不更新文档就提交代码
- ❌ 删除代码不记录到CHANGELOG
- ❌ 累积多个任务一次性commit
- ❌ 使用`any`类型（除非有充分理由）
- ❌ 添加未经测试的代码

---

## 📞 遇到问题？

按以下顺序排查：

1. **检查文档**
   - `docs/05-CONTEXT.md` - 可能已记录解决方案
   - `docs/04-CHANGELOG.md` - 类似变更的历史

2. **搜索Git历史**
   ```bash
   git log --grep="关键词"
   git log --all --grep="kimi" --oneline
   ```

3. **检查test1项目**
   - 豆包的实现可以作为参考
   - 路径：`../test1/chrome-devtools-mcp/src/doubao/`

4. **询问用户**
   - 明确说明问题和尝试过的方案
   - 提供2-3个备选方案供用户选择

---

## 🎯 开发里程碑

### Phase 1: MVP基础功能（Week 1）
- [ ] 1.1 项目初始化
- [ ] 1.2 豆包Provider开发（从test1迁移）
- [ ] 1.3 Kimi Provider开发
- [ ] 1.4 DeepSeek Provider开发
- [ ] 1.5 CLI命令开发
- [ ] 1.6 数据库集成

### Phase 2: 高级功能（Week 2）
- [ ] 2.1 批量查询支持
- [ ] 2.2 并行执行模式
- [ ] 2.3 结果对比展示
- [ ] 2.4 性能优化
- [ ] 2.5 错误处理完善

### Phase 3: 扩展与优化（后续）
- [ ] 3.1 添加更多AI平台（文心一言、通义千问、元宝）
- [ ] 3.2 导出功能（CSV、JSON、Markdown）
- [ ] 3.3 高级筛选和搜索
- [ ] 3.4 监控和日志系统

---

## 📚 关键文档索引

- **docs/01-DESIGN.md** - 架构设计和技术方案
- **docs/02-TASKS.md** - 任务清单（checkbox追踪）
- **docs/03-WORKFLOW.md** - ⭐ 执行规范（必读）
- **docs/04-CHANGELOG.md** - 变更记录
- **docs/05-CONTEXT.md** - 会话上下文（断点记录）

---

## 📊 质量指标

### 代码质量
- TypeScript编译: 0 errors
- ESLint: 0 errors, ≤5 warnings
- 代码覆盖率: ≥70%
- 单元测试通过率: 100%

### 性能指标
- 单次查询响应时间: ≤5秒
- 批量查询（10个问题）: ≤30秒
- 内存占用（单AI）: ≤500MB
- 数据库查询: ≤100ms

### 文档完整性
- 核心文档同步率: 100%
- 代码注释覆盖率: ≥50%
- API文档完整性: 100%

---

## 🔧 常用命令

```bash
# 开发
npm run dev              # 启动开发服务器
npm run build            # 编译TypeScript
npm run typecheck        # 类型检查

# 测试
npm test                 # 运行所有测试
npm run test:unit        # 运行单元测试
npm run test:integration # 运行集成测试
npm run test:coverage    # 生成覆盖率报告

# 代码质量
npm run format           # 格式化代码
npm run lint             # ESLint检查
npm run check-unused     # 检测未使用代码

# Git
git status               # 查看状态
git log --oneline -10    # 查看最近10次提交
git diff                 # 查看修改
```

---

## 🎓 学习资源

### 项目参考
- **test1项目**: `../test1/chrome-devtools-mcp/`
  - 豆包登录实现：`src/doubao/DoubaoLoginManager.ts`
  - 会话管理：`src/session/LoginSessionManager.ts`
  - 查询测试：`doubao-query-test.mjs`

### 技术文档
- chrome-devtools-mcp: https://github.com/modelcontextprotocol/servers
- Puppeteer: https://pptr.dev/
- Commander.js: https://github.com/tj/commander.js
- Inquirer.js: https://github.com/SBoudrias/Inquirer.js

---

**Last Updated**: 2025-10-13 16:25
**Current Phase**: Phase 0 - 项目初始化
**Next Milestone**: 完成核心文档编写，开始Phase 1开发
