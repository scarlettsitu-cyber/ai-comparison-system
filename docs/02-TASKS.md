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

## Phase 1: MVP基础功能（Week 1: 2025-10-14 ~ 2025-10-20）

### 1.1 核心接口定义
- [ ] 创建`src/core/interfaces/IAIProvider.ts`
- [ ] 创建`src/core/interfaces/QueryResult.ts`
- [ ] 创建`src/core/interfaces/SessionData.ts`
- [ ] 创建`src/core/types/index.ts`

### 1.2 BaseProvider抽象类
- [ ] 创建`src/providers/base/BaseProvider.ts`
- [ ] 实现通用登录流程框架
- [ ] 实现会话管理通用方法
- [ ] 编写BaseProvider单元测试

### 1.3 豆包Provider开发（从test1迁移）
- [ ] 拷贝DoubaoLoginManager.ts到`src/providers/doubao/DoubaoLogin.ts`
- [ ] 调整import路径
- [ ] 实现IAIProvider接口
- [ ] 整合DoubaoLoginDetector逻辑
- [ ] 实现DoubaoQuery.ts（查询逻辑）
- [ ] 创建DoubaoProvider.ts（总入口）
- [ ] 编写3个单元测试
  - [ ] 测试登录流程
  - [ ] 测试查询功能
  - [ ] 测试会话恢复
- [ ] 运行测试，确保全部通过

### 1.4 会话管理服务（从test1迁移）
- [ ] 拷贝SessionStorage.ts到`src/services/session/`
- [ ] 拷贝LoginSessionManager.ts并重命名为SessionService.ts
- [ ] 调整import路径
- [ ] 适配新的项目结构
- [ ] 编写SessionService单元测试

### 1.5 Kimi Provider开发
- [ ] 研究Kimi登录流程（https://kimi.moonshot.cn/）
  - [ ] 分析登录页面结构
  - [ ] 识别登录选择器
  - [ ] 测试验证码流程
- [ ] 实现KimiLogin.ts
- [ ] 实现KimiQuery.ts
- [ ] 创建KimiProvider.ts
- [ ] 编写3个单元测试
- [ ] 运行测试，确保全部通过

### 1.6 DeepSeek Provider开发
- [ ] 研究DeepSeek登录流程（https://chat.deepseek.com/）
  - [ ] 分析登录页面结构
  - [ ] 识别登录选择器
  - [ ] 测试验证码流程
- [ ] 实现DeepSeekLogin.ts
- [ ] 实现DeepSeekQuery.ts
- [ ] 创建DeepSeekProvider.ts
- [ ] 编写3个单元测试
- [ ] 运行测试，确保全部通过

### 1.7 数据库服务
- [ ] 安装better-sqlite3依赖
- [ ] 创建`src/storage/DatabaseService.ts`
- [ ] 实现SQLite初始化
- [ ] 创建所有数据表（Schema见DESIGN.md）
- [ ] 实现查询结果保存方法
- [ ] 实现引用链接保存方法
- [ ] 实现会话保存方法
- [ ] 编写DatabaseService单元测试

### 1.8 QueryService编排服务
- [ ] 创建`src/services/QueryService.ts`
- [ ] 实现单条查询编排
- [ ] 实现串行执行模式
- [ ] 集成DatabaseService（保存结果）
- [ ] 编写QueryService单元测试

### 1.9 CLI命令框架
- [ ] 安装Commander、Inquirer、Chalk、Ora
- [ ] 创建`src/cli/index.ts`（CLI入口）
- [ ] 实现`login`命令（`src/cli/commands/login.ts`）
- [ ] 实现`query`命令（`src/cli/commands/query.ts`）
- [ ] 实现`status`命令（`src/cli/commands/status.ts`）
- [ ] 添加命令帮助文档
- [ ] 手动测试所有命令

### 1.10 集成测试
- [ ] 编写端到端测试：完整登录流程
- [ ] 编写端到端测试：单条查询流程
- [ ] 编写集成测试：豆包Provider + DatabaseService
- [ ] 编写集成测试：Kimi Provider + DatabaseService
- [ ] 运行所有测试，确保100%通过

---

## Phase 2: 高级功能（Week 2: 2025-10-21 ~ 2025-10-27）

### 2.1 批量查询支持
- [ ] 实现CSV文件解析（`src/utils/CSVParser.ts`）
- [ ] 实现批量查询编排（QueryService扩展）
- [ ] 实现批量任务表管理
- [ ] 创建`batch`命令（`src/cli/commands/batch.ts`）
- [ ] 添加进度条显示（Ora集成）
- [ ] 编写批量查询测试

### 2.2 并行执行模式
- [ ] 研究chrome-devtools-mcp的--isolated参数
- [ ] 实现ParallelQueryExecutor（`src/services/ParallelQueryExecutor.ts`）
- [ ] 实现多MCP实例管理
- [ ] 实现端口分配策略
- [ ] 添加并行模式性能测试
  - [ ] 测试：3个AI并行查询 vs 串行查询
  - [ ] 验证：并行模式速度提升≥2倍
  - [ ] 验证：内存占用≤1.5GB

### 2.3 结果对比展示
- [ ] 创建ComparisonService（`src/services/ComparisonService.ts`）
- [ ] 实现并排对比展示（cli-table3）
- [ ] 实现结果差异高亮
- [ ] 实现结果评分系统（可选）
- [ ] 创建`compare`命令（`src/cli/commands/compare.ts`）

### 2.4 UI组件优化
- [ ] 拷贝test1的DoubaoCliUI.ts到`src/cli/components/UI.ts`
- [ ] 去豆包化，改为通用UI组件
- [ ] 实现统一的进度显示器
- [ ] 实现统一的错误提示
- [ ] 实现统一的成功提示

### 2.5 会话管理增强
- [ ] 实现会话自动过期清理
- [ ] 实现`session`命令（`src/cli/commands/session.ts`）
  - [ ] session list - 列出所有会话
  - [ ] session cleanup - 清理过期会话
  - [ ] session delete - 删除指定会话
- [ ] 实现会话验证功能
- [ ] 添加会话恢复失败重试机制

### 2.6 配置管理
- [ ] 创建AI配置文件（config/ai-providers/*.json）
  - [ ] doubao.json
  - [ ] kimi.json
  - [ ] deepseek.json
- [ ] 实现配置加载器（`src/core/ConfigLoader.ts`）
- [ ] 实现`config`命令（`src/cli/commands/config.ts`）
  - [ ] config list - 列出所有配置
  - [ ] config add - 添加新AI
  - [ ] config edit - 编辑配置

### 2.7 历史记录查询
- [ ] 实现历史记录查询（DatabaseService扩展）
- [ ] 创建`history`命令（`src/cli/commands/history.ts`）
  - [ ] history - 查看最近10条记录
  - [ ] history --date - 按日期筛选
  - [ ] history --provider - 按AI筛选
  - [ ] history --question - 按问题关键词搜索

### 2.8 错误处理完善
- [ ] 实现统一错误处理中间件
- [ ] 添加所有Provider的错误重试机制
- [ ] 实现网络超时处理
- [ ] 实现验证码超时处理
- [ ] 添加错误日志记录

### 2.9 性能优化
- [ ] 实现浏览器实例复用
- [ ] 实现数据库连接池
- [ ] 添加Provider配置缓存
- [ ] 优化SQL查询（添加索引）
- [ ] 进行性能基准测试

### 2.10 文档与测试
- [ ] 更新README.md（添加使用示例）
- [ ] 创建API文档（docs/API.md）
- [ ] 创建开发指南（docs/DEVELOPMENT.md）
- [ ] 补充E2E测试覆盖
- [ ] 确保代码覆盖率≥70%

---

## Phase 3: 扩展功能（后续规划）

### 3.1 添加更多AI平台
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

### 3.2 导出功能
- [ ] 实现CSV导出（`src/utils/exporters/CSVExporter.ts`）
- [ ] 实现JSON导出（`src/utils/exporters/JSONExporter.ts`）
- [ ] 实现Markdown导出（`src/utils/exporters/MarkdownExporter.ts`）
- [ ] 实现PDF导出（可选，使用puppeteer）
- [ ] 创建`export`命令

### 3.3 高级筛选和搜索
- [ ] 实现全文搜索（SQLite FTS5）
- [ ] 实现高级筛选（多条件组合）
- [ ] 实现结果排序（按时间/评分/Provider）
- [ ] 实现结果分页

### 3.4 监控和日志
- [ ] 实现日志系统（Winston）
- [ ] 添加性能监控
- [ ] 添加错误统计
- [ ] 创建`logs`命令（查看日志）

### 3.5 插件系统（可选）
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

### Phase 0进度
- 总任务数: 20
- 已完成: 3
- 进行中: 1
- 待开始: 16
- 完成度: 15%

### Phase 1预估
- 总任务数: 60+
- 预计耗时: 5天（每天8小时）
- 关键路径: 豆包Provider → Kimi Provider → CLI命令

### Phase 2预估
- 总任务数: 40+
- 预计耗时: 3天
- 关键路径: 批量查询 → 并行执行 → 结果对比

---

## 🎯 当前焦点

**本周目标（2025-10-13 ~ 2025-10-20）**:
1. 完成核心文档编写（Phase 0）
2. 完成豆包Provider迁移（Phase 1.3）
3. 完成Kimi Provider开发（Phase 1.5）
4. 完成基础CLI命令（Phase 1.9）

**今日目标（2025-10-13）**:
- ✅ 创建项目结构
- ✅ 创建CLAUDE.md和DESIGN.md
- ⏳ 完成所有核心文档
- 📋 用户审阅文档
- 📋 开始Phase 1开发

---

## 💡 提醒事项

### 对Claude的提醒
- ✅ 每完成一个checkbox就更新本文档
- ✅ 每完成一个子任务就Git commit
- ✅ 每次会话结束前更新"任务更新日志"
- ✅ 遇到问题立即记录到CONTEXT.md

### 对用户的提醒
- 📌 每完成一个Phase就review代码质量
- 📌 每次新增任务记得更新本文档
- 📌 定期备份数据库文件

---

**Last Updated**: 2025-10-13 16:40
**Current Task**: [0.2] 创建docs/02-TASKS.md
**Next Task**: [0.2] 创建docs/03-WORKFLOW.md
