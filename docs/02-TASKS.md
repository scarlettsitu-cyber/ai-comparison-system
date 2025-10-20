# 开发任务清单

> **本文档使用Markdown Checkbox追踪所有开发任务**
> **每完成一个任务就勾选对应的checkbox**

---

## 📊 整体进度

- **当前Phase**: Phase 0 - 项目初始化
- **完成度**: 15% (3/20)
- **预计完成时间**: 2周（2025-10-27）

---

## Phase 0: 项目初始化（2025-10-13）

### 0.1 项目结构创建
- [x] 创建项目目录 `ai-comparison-system`
- [x] 创建基础目录结构（docs/src/tests/config）
- [x] 初始化Git仓库

### 0.2 核心文档编写
- [x] 创建CLAUDE.md（项目说明书）
- [x] 创建docs/01-DESIGN.md（设计文档）
- [ ] 创建docs/02-TASKS.md（本文档）⏳
- [ ] 创建docs/03-WORKFLOW.md（执行规范）
- [ ] 创建docs/04-CHANGELOG.md（变更记录）
- [ ] 创建docs/05-CONTEXT.md（会话上下文）

### 0.3 开发环境配置
- [ ] 创建package.json
- [ ] 创建tsconfig.json
- [ ] 创建.gitignore
- [ ] 创建README.md
- [ ] 安装依赖（TypeScript、chrome-devtools-mcp等）

### 0.4 首次提交
- [ ] Git add所有文件
- [ ] Git commit（"chore: 项目初始化"）
- [ ] （可选）推送到GitHub

---

## Phase 1: MVP核心闭环（Week 1: 2025-10-21 ~ 2025-10-27）

**MVP目标**：登录豆包 → 提交问题 → 获取回复 → 写入数据库（串行模式）

### 1.1 核心接口定义
- [ ] 创建`src/core/interfaces/IAIProvider.ts`
- [ ] 创建`src/core/interfaces/QueryResult.ts`
- [ ] 创建`src/core/interfaces/SessionData.ts`
- [ ] 创建`src/core/types/index.ts`

### 1.2 豆包Provider开发（从test1迁移）
- [ ] 拷贝DoubaoLoginManager.ts到`src/providers/doubao/DoubaoLogin.ts`
- [ ] 调整import路径（chrome-devtools-mcp从npm安装）
- [ ] 实现IAIProvider接口
- [ ] 整合DoubaoLoginDetector逻辑
- [ ] 实现DoubaoQuery.ts（查询逻辑）
- [ ] 创建DoubaoProvider.ts（总入口）
- [ ] 编写3个单元测试
  - [ ] 测试登录流程
  - [ ] 测试查询功能
  - [ ] 测试会话恢复
- [ ] 运行测试，确保全部通过

### 1.3 数据库服务
- [ ] 创建`src/storage/DatabaseService.ts`
- [ ] 实现SQLite初始化
- [ ] 创建数据表（queries, results, sessions）
- [ ] 实现查询结果保存方法
- [ ] 实现会话保存方法
- [ ] 编写DatabaseService单元测试

### 1.4 QueryService串行编排
- [ ] 创建`src/services/QueryService.ts`
- [ ] 实现单条查询编排（单AI、单问题）
- [ ] 实现串行执行模式（for循环逐个执行）
- [ ] 集成DatabaseService（自动保存结果）
- [ ] 编写QueryService单元测试

### 1.5 TUI最小化界面（5个必需界面）
- [ ] 安装blessed、cli-table3、chalk、ora、boxen
- [ ] 创建`src/tui/index.ts`（TUI入口）
- [ ] 创建`src/tui/TUIManager.ts`（核心管理器）
- [ ] **Level 0: 主界面**（基础菜单：AI管理、开始任务、退出）
- [ ] **Level 1-A: AI应用管理**（显示豆包状态、登录）
- [ ] **Level 1-B: 任务管理**（简化版：选AI、输入问题、确认执行）
- [ ] **Level 2-B: 执行中（写库模式）**（进度条 + 💾写库标识）
- [ ] **Level 3-B: 完成总结**（显示结果、失败任务）
- [ ] 实现键盘事件监听（↑↓选择、Enter确认、Q退出）
- [ ] 手动测试完整流程

### 1.6 端到端测试
- [ ] 编写端到端测试：豆包登录流程
- [ ] 编写端到端测试：单条查询流程（登录→查询→写库）
- [ ] 编写集成测试：豆包Provider + DatabaseService
- [ ] 运行所有测试，确保100%通过
- [ ] ✅ **MVP闭环验收**：用户手动测试完整流程

---

## Phase 2: 多AI支持（Week 2）

**前置条件**：Phase 1 MVP闭环验收通过 + 手动测试Kimi和DeepSeek登录流程

### 2.1 Kimi Provider开发
- [ ] ⚠️ **用户测试Kimi登录流程**（https://kimi.moonshot.cn/）
  - [ ] 分析登录页面结构
  - [ ] 识别登录选择器
  - [ ] 测试验证码流程
  - [ ] 确认可用后继续
- [ ] 实现KimiLogin.ts
- [ ] 实现KimiQuery.ts
- [ ] 创建KimiProvider.ts
- [ ] 编写3个单元测试
- [ ] 运行测试，确保全部通过
- [ ] ✅ **用户验收**：Kimi完整流程可用

### 2.2 DeepSeek Provider开发
- [ ] ⚠️ **用户测试DeepSeek登录流程**（https://chat.deepseek.com/）
  - [ ] 分析登录页面结构
  - [ ] 识别登录选择器
  - [ ] 测试验证码流程
  - [ ] 确认可用后继续
- [ ] 实现DeepSeekLogin.ts
- [ ] 实现DeepSeekQuery.ts
- [ ] 创建DeepSeekProvider.ts
- [ ] 编写3个单元测试
- [ ] 运行测试，确保全部通过
- [ ] ✅ **用户验收**：DeepSeek完整流程可用

### 2.3 BaseProvider抽象类（可选优化）
- [ ] 评估3个Provider的共同代码
- [ ] 如有必要，创建`src/providers/base/BaseProvider.ts`
- [ ] 实现通用登录流程框架
- [ ] 实现会话管理通用方法
- [ ] 重构3个Provider继承BaseProvider

### 2.4 多AI集成测试
- [ ] 编写集成测试：Kimi Provider + DatabaseService
- [ ] 编写集成测试：DeepSeek Provider + DatabaseService
- [ ] 编写端到端测试：3个AI串行查询流程
- [ ] 运行所有测试，确保100%通过

---

## Phase 3: TUI完整界面（Week 3）

**目标**：实现所有6个TUI界面 + 公共组件库 + 双输出模式

### 3.1 TUI演示模式界面（Level 2-A、3-A）
- [ ] **Level 2-A: 执行中（演示模式）**
  - [ ] 创建`src/tui/screens/ExecutionDemoScreen.ts`
  - [ ] 实现AI×问题进度矩阵（cli-table3）
  - [ ] 实时更新状态图标（⏳→✓/✗）
  - [ ] 实现日志滚动区域（blessed.log）
  - [ ] 显示执行时间和剩余时间
  - [ ] 自动跳转到结果展示界面
- [ ] **Level 3-A: 结果展示（演示模式）**
  - [ ] 创建`src/tui/screens/ResultDemoScreen.ts`
  - [ ] 实现逐个展示AI回复（← → 切换问题）
  - [ ] 实现并排对比视图（V切换）
  - [ ] 实现手动保存功能（S键）
  - [ ] 实现复制全部结果（C键）
  - [ ] 实现导出功能（CSV/JSON）

### 3.2 任务管理界面完整版（Level 1-B）
- [ ] 升级`src/tui/screens/TaskManagementScreen.ts`
- [ ] 实现步骤1：AI多选（Space勾选、Enter确认）
- [ ] 实现步骤2：问题输入（多行文本框、或CSV导入）
- [ ] 实现步骤3：执行模式选择（串行/自动）⚠️ 并行模式暂不可选
- [ ] 实现步骤4：输出模式选择（演示模式/写库模式）⭐
- [ ] 实现Tab切换焦点区域
- [ ] 手动测试所有步骤

### 3.3 TUI公共组件库
- [ ] 创建`src/tui/components/Header.ts`（统一顶栏）
- [ ] 创建`src/tui/components/Footer.ts`（统一底栏）
- [ ] 创建`src/tui/components/Table.ts`（通用表格）
- [ ] 创建`src/tui/components/ProgressBar.ts`（进度条）
- [ ] 创建`src/tui/components/StatusIcon.ts`（状态图标）
- [ ] 创建`src/tui/components/ErrorDialog.ts`（错误弹窗）
- [ ] 编写组件单元测试

### 3.4 错误处理完善
- [ ] 实现统一错误处理中间件
- [ ] 添加3种错误类型弹窗（登录错误、查询错误、资源不足）
- [ ] 实现网络超时处理
- [ ] 实现验证码超时处理
- [ ] 添加错误日志记录
- [ ] 实现重试机制

### 3.5 文档与测试
- [ ] 更新README.md（添加TUI使用示例）
- [ ] 创建API文档（docs/API.md）
- [ ] 创建TUI开发指南（docs/TUI-DEVELOPMENT.md）
- [ ] 补充E2E测试覆盖
- [ ] 确保代码覆盖率≥70%

---

## Phase 4: 并行执行模式（Week 4）

**前置条件**：Phase 3完成 + 系统资源评估器开发完成

**⚠️ 重要**：三种模式需要逐个开发，每个都要用户确认效果

### 4.1 系统资源评估器
- [ ] 创建`src/services/ResourceEvaluator.ts`
- [ ] 实现内存检测（os.freemem()）
- [ ] 实现CPU检测（os.cpus().length）
- [ ] 实现推荐并发数计算公式
- [ ] 实现推荐模式选择逻辑
- [ ] 返回推荐结果对象
- [ ] 编写单元测试

### 4.2 串行模式优化（保底方案）
- [ ] 创建`src/services/executors/SerialQueryExecutor.ts`
- [ ] 从QueryService抽取串行逻辑
- [ ] 实现统一的Executor接口
- [ ] 测试：3AI × 3问题 ≈ 45秒
- [ ] 测试：内存占用 ≤ 500MB
- [ ] ✅ **用户确认执行效果**

### 4.3 方案A：多AI并行
- [ ] 研究chrome-devtools-mcp的`--isolated`参数
- [ ] 创建`src/services/executors/MultiAIParallelExecutor.ts`
- [ ] 实现多MCP实例启动（每个AI一个实例）
- [ ] 实现端口分配策略
- [ ] 实现Promise.all并行执行
- [ ] 测试：3AI × 3问题 ≈ 15秒
- [ ] 测试：内存占用 ≤ 1.5GB
- [ ] ✅ **用户确认"同时打开3个浏览器窗口"的效果**

### 4.4 方案B：单AI多问题并行
- [ ] 创建`src/services/executors/SingleAIMultiQuestionExecutor.ts`
- [ ] 实现Puppeteer多Page并行（browser.newPage()）
- [ ] 实现Promise.all并行查询
- [ ] 实现Page关闭清理
- [ ] 测试：1AI × 10问题 ≈ 5秒
- [ ] 测试：内存占用 ≤ 1.5GB
- [ ] ✅ **用户确认"同时打开10个标签页"的效果**

### 4.5 执行模式集成
- [ ] 更新TaskManagementScreen支持3种模式选择
- [ ] 集成ResourceEvaluator显示推荐
- [ ] 实现自动模式选择
- [ ] 更新ExecutionScreen支持不同模式显示
- [ ] 端到端测试3种模式

---

## Phase 5: 扩展功能（后续规划）

### 5.1 添加更多AI平台
- [ ] 文心一言（https://yiyan.baidu.com/）
  - [ ] 研究登录流程
  - [ ] 实现Provider
  - [ ] 编写测试
- [ ] 通义千问（https://www.tongyi.com/）
  - [ ] 研究登录流程
  - [ ] 实现Provider
  - [ ] 编写测试
- [ ] 元宝（https://yuanbao.tencent.com/chat/）
  - [ ] 研究登录流程
  - [ ] 实现Provider
  - [ ] 编写测试

### 5.2 导出功能
- [ ] 实现CSV导出（`src/utils/exporters/CSVExporter.ts`）
- [ ] 实现JSON导出（`src/utils/exporters/JSONExporter.ts`）
- [ ] 实现Markdown导出（`src/utils/exporters/MarkdownExporter.ts`）
- [ ] 实现PDF导出（可选，使用puppeteer）
- [ ] 创建`export`命令

### 5.3 高级筛选和搜索
- [ ] 实现全文搜索（SQLite FTS5）
- [ ] 实现高级筛选（多条件组合）
- [ ] 实现结果排序（按时间/评分/Provider）
- [ ] 实现结果分页

### 5.4 监控和日志
- [ ] 实现日志系统（Winston）
- [ ] 添加性能监控
- [ ] 添加错误统计
- [ ] 创建`logs`命令（查看日志）

### 5.5 插件系统（可选）
- [ ] 设计插件接口
- [ ] 实现插件加载器
- [ ] 创建示例插件
- [ ] 编写插件开发文档

---

## 技术债务与优化

### 待重构项目
- [ ] 无（新项目）

### 性能优化
- [ ] 无（Phase 2完成后评估）

### 安全增强
- [ ] 实现Cookie加密存储
- [ ] 实现输入验证和清洗
- [ ] 添加SQL注入防护

---

## 已弃用任务

*（目前无）*

---

## 📝 任务更新日志

### 2025-10-18 22:15 - TASKS.md重组（YAGNI原则）
- ✅ 完成文档架构优化（CLAUDE.md瘦身、DECISIONS.md、DEV-STANDARDS.md）
- ✅ 完成TUI产品设计文档（10个文件）
- ✅ 重组TASKS.md为MVP优先
  - **Phase 1**: 精简为6个子任务（豆包MVP闭环）
  - **Phase 2**: Kimi/DeepSeek Provider（需先测试）
  - **Phase 3**: TUI完整界面 + 组件库
  - **Phase 4**: 并行执行模式
  - **Phase 5**: 扩展功能
- ⏳ 创建docs/ARCHITECTURE.md
- ⏳ 更新docs/05-CONTEXT.md
- ⏳ Git提交推送

**会话**: 2025-10-18-004
**用户**: @situruimin
**下次继续**: 创建ARCHITECTURE.md、更新CONTEXT.md、Git推送

---

### 2025-10-18 20:50 - TUI架构确定
- ✅ 完成TUI设计确认（6个界面、3种执行模式、双输出模式）
- ✅ 更新CLAUDE.md（技术栈、决策记录）
- ✅ 更新docs/01-DESIGN.md（架构、并行方案、TUI详细设计）
- ✅ 更新docs/02-TASKS.md（TUI任务替换CLI任务）
- ✅ 更新docs/04-CHANGELOG.md（变更记录）
- ✅ 更新docs/05-CONTEXT.md（会话上下文）
- ✅ 创建docs/06-TUI-DESIGN/（11个详细设计文档）

**会话**: 2025-10-18-003
**用户**: @situruimin
**下次继续**: 文档优化、TASKS重组

---

### 2025-10-13 16:30 - 项目初始化
- ✅ 创建项目目录结构
- ✅ 初始化Git仓库
- ✅ 创建CLAUDE.md
- ✅ 创建docs/01-DESIGN.md
- ⏳ 创建docs/02-TASKS.md（当前）

**会话**: 2025-10-13-001
**用户**: @situruimin
**下次继续**: 完成剩余核心文档（03-WORKFLOW, 04-CHANGELOG, 05-CONTEXT）

---

## 📊 统计信息

### Phase 0进度（项目初始化）
- 总任务数: 16
- 已完成: 3
- 进行中: 13
- 完成度: 80%（文档阶段）

### Phase 1预估（MVP核心闭环）
- 总任务数: ~35
- 预计耗时: 5天（每天8小时）
- 关键路径: 豆包Provider → DatabaseService → QueryService → TUI基础界面
- ✅ 纯MVP：只有豆包、串行模式、写库模式

### Phase 2预估（多AI支持）
- 总任务数: ~20
- 预计耗时: 3天
- 关键路径: Kimi Provider → DeepSeek Provider
- ⚠️ 前置条件：手动测试Kimi/DeepSeek登录流程

### Phase 3预估（TUI完整界面）
- 总任务数: ~40
- 预计耗时: 4天
- 关键路径: 演示模式界面 → 组件库 → 错误处理

### Phase 4预估（并行执行模式）
- 总任务数: ~25
- 预计耗时: 3天
- 关键路径: 资源评估器 → 3种模式逐个开发
- ⚠️ 每种模式都需要用户验收

---

## 🎯 当前焦点

**当前Phase**: Phase 0 - 项目初始化（文档阶段）
**完成度**: 80% (13/16)

**本次会话剩余任务（Phase 0.2）**:
- ✅ CLAUDE.md瘦身（421→199行）
- ✅ 创建DECISIONS.md、DEV-STANDARDS.md
- ✅ 创建3个子目录CLAUDE.md
- ✅ 创建10个TUI产品设计文档
- ✅ 重组TASKS.md为MVP优先
- ⏳ 创建docs/ARCHITECTURE.md
- ⏳ 更新docs/05-CONTEXT.md
- ⏳ Git提交并推送到GitHub

**下次会话任务（Phase 0.3）**:
- 创建package.json、tsconfig.json、.gitignore
- 安装依赖（chrome-devtools-mcp、better-sqlite3、blessed等）
- 验证环境（TypeScript编译、测试依赖）
- Git提交环境配置

---

## 💡 提醒事项

### 对Claude的提醒
- ✅ 每完成一个checkbox就更新本文档
- ✅ 每完成一个子任务就Git commit
- ✅ 每次会话结束前更新"任务更新日志"
- ✅ 遇到问题立即记录到CONTEXT.md
- ⚠️ **MVP优先原则**：Phase 1只做豆包、串行、写库

### 对用户的提醒
- 📌 每完成一个Phase就review代码质量
- 📌 每次新增任务记得更新本文档
- 📌 定期备份数据库文件
- 📌 **Phase 2前必须手动测试Kimi和DeepSeek登录**

---

**Last Updated**: 2025-10-18 22:15
**Current Task**: [0.2] 重组TASKS.md（已完成）
**Next Task**: [0.2] 创建docs/ARCHITECTURE.md
