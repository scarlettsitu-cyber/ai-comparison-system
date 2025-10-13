# AI Comparison System

> **多AI平台对比查询工具 - 一次提问，多个AI同时响应**

[![TypeScript](https://img.shields.io/badge/TypeScript-5.3-blue)](https://www.typescriptlang.org/)
[![Node](https://img.shields.io/badge/Node.js-20%2B-green)](https://nodejs.org/)
[![License](https://img.shields.io/badge/license-MIT-brightgreen)](LICENSE)

---

## 📋 项目简介

**ai-comparison-system** 是一个命令行工具，用于同时向多个AI平台（豆包、Kimi、DeepSeek等）提问并对比结果，帮助用户快速获取不同AI的观点和答案。

### 核心功能

- ✅ **多AI平台支持**: 豆包、Kimi、DeepSeek、文心一言、通义千问、元宝
- ✅ **智能会话管理**: 自动登录保持，无需重复登录
- ✅ **单条/批量查询**: 支持单次提问或CSV批量导入
- ✅ **并行执行模式**: 同时查询多个AI，速度提升3倍
- ✅ **结果对比展示**: 并排显示，快速发现差异
- ✅ **数据持久化**: SQLite本地存储，支持历史回溯
- ✅ **引用来源追踪**: 记录AI回答的引用链接

---

## 🚀 快速开始

### 系统要求

- Node.js 20.0+
- npm 9.0+
- Chrome浏览器

### 安装

```bash
# 克隆项目
git clone https://github.com/situruimin/ai-comparison-system.git
cd ai-comparison-system

# 安装依赖
npm install

# 编译项目
npm run build
```

### 基本使用

```bash
# 1. 登录AI平台
ai-compare login doubao
ai-compare login kimi

# 2. 单条查询
ai-compare query "深圳宠物医院推荐" --apps doubao,kimi

# 3. 批量查询
ai-compare batch --input questions.csv --apps all

# 4. 查看状态
ai-compare status

# 5. 查看历史
ai-compare history
```

---

## 📖 使用示例

### 示例1：单条查询

```bash
$ ai-compare query "Python数据分析库对比" --apps doubao,kimi

🔄 正在查询...
  ✓ 豆包 [2.3s]
  ✓ Kimi [1.8s]

╔═══════════════════════════════════════════╗
║ 豆包                                      ║
╠═══════════════════════════════════════════╣
║ 主流Python数据分析库对比：                ║
║ 1. Pandas - 数据处理和清洗               ║
║ 2. NumPy - 科学计算                      ║
║ 3. Matplotlib - 数据可视化               ║
║                                           ║
║ 📚 引用来源:                              ║
║ [1] https://pandas.pydata.org/           ║
╚═══════════════════════════════════════════╝

╔═══════════════════════════════════════════╗
║ Kimi                                      ║
╠═══════════════════════════════════════════╣
║ 为您推荐以下Python数据分析库：            ║
║ • Pandas - 最常用的数据分析库            ║
║ • Polars - 新一代高性能库                ║
║ • Dask - 大规模并行计算                  ║
╚═══════════════════════════════════════════╝

💾 结果已保存到数据库 (request_id: req_20251013_001)
```

### 示例2：批量查询

```bash
$ ai-compare batch --input questions.csv --apps all

📂 读取问题列表: questions.csv
   共10个问题

执行模式: ⚡ 并行执行
预计耗时: ~15秒

🔄 正在处理...
  [████████████████████] 100% (10/10)

✅ 批量查询完成！
   成功: 10个 | 失败: 0个
   总耗时: 14.2秒
   平均每个问题: 1.4秒
```

### 示例3：查看历史记录

```bash
$ ai-compare history --date 2025-10-13

📋 历史查询记录（2025-10-13）

1. 14:30 - "深圳宠物医院推荐"
   AI: 豆包、Kimi
   状态: ✅ 成功

2. 15:00 - "Python数据分析库对比"
   AI: 豆包、Kimi、DeepSeek
   状态: ✅ 成功

3. 16:20 - "React vs Vue性能分析"
   AI: DeepSeek
   状态: ✅ 成功
```

---

## 🏗️ 架构设计

### 系统架构

```
┌─────────────────────────────────────────┐
│        CLI Interface Layer              │
│  (Commander + Inquirer + Chalk)         │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│         Service Layer                   │
│  QueryService + SessionService          │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│       AI Providers Layer                │
│  Doubao | Kimi | DeepSeek | ...         │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│    Infrastructure Layer                 │
│  Browser + Database + Utils             │
└─────────────────────────────────────────┘
```

### 核心技术

- **TypeScript 5.3**: 类型安全的编程语言
- **Puppeteer**: 浏览器自动化
- **chrome-devtools-mcp**: Model Context Protocol服务器
- **SQLite**: 轻量级数据库
- **Commander.js**: CLI框架
- **Inquirer.js**: 交互式提示

---

## 📂 项目结构

```
ai-comparison-system/
├── docs/                      # 📚 文档目录
│   ├── CLAUDE.md             # Claude Code项目指南
│   ├── 01-DESIGN.md          # 架构设计文档
│   ├── 02-TASKS.md           # 任务清单
│   ├── 03-WORKFLOW.md        # 开发规范
│   ├── 04-CHANGELOG.md       # 变更记录
│   └── 05-CONTEXT.md         # 会话上下文
│
├── src/
│   ├── core/                 # 核心接口和类型
│   ├── providers/            # AI平台适配器
│   ├── services/             # 业务逻辑层
│   ├── cli/                  # CLI命令
│   └── storage/              # 数据存储
│
├── tests/
│   ├── unit/                 # 单元测试
│   ├── integration/          # 集成测试
│   └── e2e/                  # 端到端测试
│
├── config/                   # 配置文件
├── CLAUDE.md                 # ⭐ Claude必读
├── package.json              # 依赖配置
├── tsconfig.json             # TypeScript配置
└── README.md                 # 本文件
```

---

## 🔧 开发指南

### 安装开发依赖

```bash
npm install
```

### 运行测试

```bash
# 运行所有测试
npm test

# 运行单元测试
npm run test:unit

# 运行集成测试
npm run test:integration

# 生成覆盖率报告
npm run test:coverage
```

### 代码质量检查

```bash
# TypeScript类型检查
npm run typecheck

# ESLint检查
npm run lint

# 代码格式化
npm run format

# 检查未使用代码
npm run check-unused
```

### 添加新AI平台

1. 创建Provider目录：`src/providers/new-ai/`
2. 实现IAIProvider接口
3. 编写单元测试
4. 添加配置文件：`config/ai-providers/new-ai.json`
5. 更新文档

详见：[docs/01-DESIGN.md](docs/01-DESIGN.md)

---

## 📊 开发进度

### Phase 0: 项目初始化 ✅
- [x] 创建项目结构
- [x] 编写核心文档
- [x] 配置开发环境

### Phase 1: MVP基础功能（进行中）
- [ ] 豆包Provider开发
- [ ] Kimi Provider开发
- [ ] DeepSeek Provider开发
- [ ] CLI命令框架
- [ ] 数据库集成

### Phase 2: 高级功能（计划中）
- [ ] 批量查询支持
- [ ] 并行执行模式
- [ ] 结果对比展示
- [ ] 性能优化

详见：[docs/02-TASKS.md](docs/02-TASKS.md)

---

## 🤝 贡献指南

欢迎提交Issue和Pull Request！

### 开发流程

1. Fork本项目
2. 创建功能分支：`git checkout -b feature/new-feature`
3. 提交修改：`git commit -m "feat: add new feature"`
4. 推送分支：`git push origin feature/new-feature`
5. 创建Pull Request

### Commit规范

使用Angular Commit Message格式：

```
<type>(<scope>): <subject>

<body>

<footer>
```

Type类型：
- `feat`: 新功能
- `fix`: Bug修复
- `docs`: 文档更新
- `test`: 测试相关
- `refactor`: 代码重构
- `chore`: 构建/工具配置

---

## 📄 许可证

本项目采用 [MIT License](LICENSE)

---

## 📞 联系方式

- **作者**: situruimin
- **邮箱**: situruimin@example.com
- **GitHub**: https://github.com/situruimin/ai-comparison-system
- **Issues**: https://github.com/situruimin/ai-comparison-system/issues

---

## 🙏 致谢

- [chrome-devtools-mcp](https://github.com/modelcontextprotocol/servers) - MCP服务器
- [Puppeteer](https://pptr.dev/) - 浏览器自动化
- [Commander.js](https://github.com/tj/commander.js) - CLI框架
- [Inquirer.js](https://github.com/SBoudrias/Inquirer.js) - 交互式提示

---

**注意**: 本项目仅供学习和研究使用，请遵守各AI平台的服务条款和使用协议。

---

**Last Updated**: 2025-10-13
**Current Version**: 0.1.0（开发中）
**License**: MIT
