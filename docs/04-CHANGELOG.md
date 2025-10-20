# 变更记录

> **本文档记录所有计划变更、架构调整和重大决策**
> **用于追溯历史和避免重复错误**

---

## 变更记录格式

每条变更记录包含：
- **变更编号**: #001, #002...
- **变更日期**: YYYY-MM-DD
- **变更类型**: 计划变更 / 架构调整 / 代码删除 / 重构
- **原计划/问题**: 描述原来的计划或遇到的问题
- **新方案**: 描述调整后的方案
- **影响范围**: 列出受影响的文件和模块
- **批准人**: 谁批准了这次变更（通常是用户）

---

## 变更历史

### [变更 #001] 2025-10-13 - 项目初始化
**类型**: 计划变更
**原计划**: 修改test1项目
**问题**: test1项目已混入豆包业务代码，修改会破坏现有功能
**新方案**: 创建新项目ai-comparison-system，从test1迁移可复用代码
**影响范围**:
- 新建项目目录
- 制定完整的文档体系
- 设计清晰的架构
**批准人**: 用户（会话2025-10-13-001）

---

### [变更 #002] 2025-10-18 - Web GUI → TUI架构调整
**类型**: 架构调整
**原计划**: 使用CLI命令行交互（Commander.js + Inquirer.js）
**问题**: 用户历史需求提到"界面、按钮、表单"，澄清后确认需要TUI（Terminal User Interface）
**新方案**: 使用blessed框架创建6个TUI界面，提供类似GUI的终端交互体验
**影响范围**:
- 删除：Commander.js、Inquirer.js依赖
- 新增：blessed、cli-table3、boxen依赖
- 新增：src/tui/目录（界面、组件）
- 修改：CLAUDE.md（技术栈、决策#002）
- 修改：docs/01-DESIGN.md（架构图、Section 7完全重写）
- 修改：docs/02-TASKS.md（Phase 1.9、Phase 2全部重写）
- 新增：docs/06-TUI-DESIGN/（11个详细设计文档）
**批准人**: 用户（会话2025-10-18-003）

---

### [变更 #003] 2025-10-18 - 演示模式作为任务管理选项
**类型**: 计划变更
**原计划**: 所有查询结果自动写入数据库
**问题**: 用户希望有"演示模式"在测试时不写库，只展示结果
**新方案**: 在Level 1-B任务管理界面增加"步骤4: 输出模式选择"
- 演示模式（Demo Mode）：不写库，缓存在内存，Level 3-A展示结果
- 写库模式（Database Mode）：立即写库，Level 3-B展示统计和失败处理
**影响范围**:
- 新增：src/tui/screens/ExecutionDemoScreen.ts（Level 2-A）
- 新增：src/tui/screens/ExecutionDBScreen.ts（Level 2-B）
- 新增：src/tui/screens/ResultDemoScreen.ts（Level 3-A）
- 新增：src/tui/screens/ResultDBScreen.ts（Level 3-B）
- 修改：TaskManagementScreen.ts（增加步骤4）
- 修改：QueryService.ts（根据mode参数决定是否写库）
**批准人**: 用户（会话2025-10-18-003）

---

### [变更 #004] 2025-10-18 - 三种并行模式统一设计
**类型**: 架构调整
**原计划**: 只有"串行"和"并行"两种模式
**问题**: 用户确认需要三种并行方案都支持，且每种都要逐个开发并确认效果
**新方案**: 统一设计四种执行模式（3种并行+1种自动选择）
- 串行模式：保底方案，~500MB内存，最慢但最稳定
- 方案A（多AI并行）：chrome-devtools-mcp的`--isolated`启动多实例，~1.5GB内存
- 方案B（单AI多问题并行）：Puppeteer多Page并行，~1.5GB内存，单AI场景最快
- 自动选择模式：ResourceEvaluator根据系统资源自动推荐
**影响范围**:
- 新增：src/services/ResourceEvaluator.ts（资源评估器）
- 新增：src/services/executors/SerialQueryExecutor.ts（串行）
- 新增：src/services/executors/MultiAIParallelExecutor.ts（方案A）
- 新增：src/services/executors/SingleAIMultiQuestionExecutor.ts（方案B）
- 修改：docs/01-DESIGN.md（Section 5完全重写，增加详细算法和性能对比表）
- 修改：docs/02-TASKS.md（Phase 2.9拆分为2.9.1/2.9.2/2.9.3）
**批准人**: 用户（会话2025-10-18-003）

---

## 变更示例模板

### [变更 #XXX] YYYY-MM-DD - 变更标题
**类型**: 计划变更 / 架构调整 / 代码删除 / 重构
**原计划**: XXX
**问题**: XXX
**新方案**: XXX
**影响范围**:
- 修改：xxx.ts
- 新增：yyy.ts
- 删除：zzz.ts
**批准人**: 用户（会话YYYY-MM-DD-XXX）

---

## 删除记录

### [删除 #XXX] YYYY-MM-DD - 删除标题
**删除文件/代码**: 路径
**删除原因**: 原因说明
**替代方案**: 新的实现路径
**最后使用**: Phase X.Y（已完成迁移）
**验证方式**: 如何确认无影响（如运行测试）

---

## 重构记录

### [重构 #XXX] YYYY-MM-DD - 重构标题
**重构模块**: 模块名
**重构原因**: 触发重构的条件（如代码重复率>20%）
**重构方法**: 技术方案（如抽象基类）
**影响范围**: 受影响的文件列表
**测试**: 测试通过情况
**性能影响**: 性能对比（如响应时间从5s降到3s）

---

## 统计信息

- 总变更数: 4
- 计划变更: 2（#001, #003）
- 架构调整: 2（#002, #004）
- 代码删除: 0
- 重构: 0

---

**Last Updated**: 2025-10-18 20:55
**Status**: 活跃使用中
