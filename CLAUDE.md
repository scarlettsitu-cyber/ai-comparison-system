# CLAUDE.md - 项目指南

> **本文件是Claude Code的项目说明书，每次打开新窗口会自动读取**

---

## 📋 项目概述

**项目名称**: ai-comparison-system
**项目目标**: TUI工具，支持向多个AI平台（豆包、Kimi、DeepSeek等）同时提问并对比结果
**技术栈**: TypeScript + Puppeteer + chrome-devtools-mcp + SQLite + blessed + cli-table3
**开发周期**: 2周MVP

### 核心功能
- ✅ 多AI平台会话管理（登录、保持、恢复）
- ✅ TUI交互界面（6个终端界面，键盘导航）
- ✅ 单条/批量查询支持
- ✅ 三种执行模式（串行、方案A并行、方案B并行）
- ✅ 双输出模式（演示模式、写库模式）
- ✅ 系统资源自动评估

---

## 🎯 当前阶段

**Phase**: Phase 0 - 项目初始化（文档完成阶段）
**进度**: 90%
**下一步**: 环境配置→Phase 1开发

详见：`@docs/02-TASKS.md`

---

## 📂 项目结构

```
ai-comparison-system/
├── docs/                  # 📚 核心文档
│   ├── 01-DESIGN.md       # 架构设计
│   ├── 02-TASKS.md        # 任务清单
│   ├── 03-WORKFLOW.md     # 执行规范
│   ├── 05-CONTEXT.md      # 会话上下文（断点记录）
│   ├── DECISIONS.md       # 技术决策
│   ├── DEV-STANDARDS.md   # 开发规范
│   ├── ARCHITECTURE.md    # 目录架构
│   └── 06-TUI-DESIGN/     # TUI界面详细设计
│
├── src/
│   ├── core/interfaces/   # 核心接口
│   ├── providers/         # AI平台适配器（查看 @src/providers/CLAUDE.md）
│   ├── services/          # 业务逻辑（查看 @src/services/CLAUDE.md）
│   ├── tui/               # TUI界面（查看 @src/tui/CLAUDE.md）
│   └── storage/           # 数据持久化
│
├── tests/                 # 测试
└── config/                # 配置文件
```

---

## 🔑 关键决策

完整决策记录详见：`@docs/DECISIONS.md`

- **#001**: 适配器模式支持多AI（`IAIProvider`接口）
- **#002**: 使用SQLite数据库
- **#003**: 采用TUI界面（blessed框架）
- **#004**: 演示模式 vs 写库模式
- **#005**: 三种并行模式统一设计
- **#006**: 项目独立安装chrome-devtools-mcp
- **#007**: 文档驱动的跨窗口协作

---

## 📜 开发规范速查

完整规范详见：`@docs/DEV-STANDARDS.md`

### TypeScript
- ✅ 严格模式、禁用`any`、接口`I`前缀

### 测试
- ✅ 覆盖率≥70%、每个Provider至少3个测试

### Commit
- ✅ 格式：`<type>(<scope>): <subject>`
- ✅ 每完成checkbox就commit

### 文档
- ✅ 代码变更同步更新文档
- ✅ 重大变更记录`CHANGELOG.md`

---

## 🔄 执行流程

详细Checklist详见：`@docs/03-WORKFLOW.md`

### 快速检查
**新窗口启动时**：
1. Claude自动读取本文件
2. 手动检查`docs/05-CONTEXT.md`（上次断点）
3. 手动检查`docs/02-TASKS.md`（当前任务）
4. 向用户确认："上次进度到[XXX]，下一步[YYY]，开始吗？"

**完成任务时**：
1. 更新`docs/02-TASKS.md`（勾选checkbox）
2. 更新`docs/05-CONTEXT.md`（记录状态）
3. Git commit
4. 向用户汇报："✅已完成[X.X]，📝已更新文档，💾已提交Git"

---

## 🚫 禁止事项

- ❌ 不读文档就开发
- ❌ 不经用户同意修改核心架构
- ❌ 跳过测试
- ❌ 使用`any`类型
- ❌ 不更新文档

---

## 🔧 常用命令

```bash
# TUI应用
npm run start            # 启动TUI界面

# 开发
npm run dev              # 开发模式
npm run build            # 编译TypeScript
npm run typecheck        # 类型检查

# 测试
npm test                 # 运行所有测试
npm run test:coverage    # 覆盖率报告

# 代码质量
npm run format           # 格式化
npm run lint             # ESLint检查
```

---

## 📚 核心文档索引

### 必读文档（启动时）
- **本文件（CLAUDE.md）** - 项目总览
- **docs/05-CONTEXT.md** - 上次会话断点
- **docs/02-TASKS.md** - 当前任务状态

### 设计文档（开发时按需读取）
- **@docs/01-DESIGN.md** - 架构设计和技术方案
- **@docs/ARCHITECTURE.md** - 目录结构和模块划分
- **@docs/06-TUI-DESIGN/** - TUI界面详细设计（11个文件）

### 规范文档（开发时按需读取）
- **@docs/03-WORKFLOW.md** - 执行规范和Checklist
- **@docs/DEV-STANDARDS.md** - 代码规范
- **@docs/DECISIONS.md** - 技术决策记录
- **@docs/04-CHANGELOG.md** - 变更历史

### 模块文档（进入对应目录时自动加载）
- **@src/providers/CLAUDE.md** - Provider开发规范
- **@src/tui/CLAUDE.md** - TUI组件开发规范
- **@src/services/CLAUDE.md** - Service层规范

---

## 🎓 学习资源

### 项目参考
- **test1项目**: `../test1/chrome-devtools-mcp/`
  - 豆包登录实现：`src/doubao/DoubaoLoginManager.ts`
  - 会话管理：`src/session/LoginSessionManager.ts`

### 技术文档
- chrome-devtools-mcp: https://github.com/modelcontextprotocol/servers
- Puppeteer: https://pptr.dev/
- blessed (TUI框架): https://github.com/chjj/blessed
- cli-table3: https://github.com/cli-table/cli-table3

---

## 💡 上下文管理最佳实践

**避免"上下文腐烂"**：
- ✅ 本文件控制在150行、5KB以内
- ✅ 详细文档使用`@`语法延迟加载
- ✅ 子目录使用独立CLAUDE.md（按需加载）
- ✅ 定期执行`/compact`压缩上下文
- ✅ 切换功能时执行`/clear`重置

---

**Last Updated**: 2025-10-18 21:35
**Current Phase**: Phase 0 - 项目初始化（90%）
**Next Milestone**: 环境配置 → Phase 1 MVP开发
