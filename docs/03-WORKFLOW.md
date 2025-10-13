# 开发执行规范

> **本文档定义Claude Code窗口的标准操作流程，必须严格遵守**
> **这是保证跨窗口协作一致性的核心文档**

---

## 📖 总则

### 规范目的
- ✅ 保证多个Claude窗口行为一致
- ✅ 防止代码质量下降
- ✅ 确保任务追踪完整
- ✅ 避免Git历史混乱

### 适用范围
- ✅ 所有Claude Code窗口
- ✅ 所有开发任务
- ✅ 所有代码提交

### 违规后果
- ❌ 代码可能被回滚
- ❌ 需要额外时间修复
- ❌ 项目进度受影响

---

## 🚀 Phase 1: 新窗口启动规范

> **每次打开新Claude窗口必须执行此流程**

### ✅ Checklist（必须全部完成）

#### Step 1: 自动读取（0分钟）
- [ ] **CLAUDE.md会自动读取**（无需操作）
  - Claude Code会自动加载此文件
  - 理解项目概述、技术栈、当前阶段

#### Step 2: 手动检查文档（2分钟）
```bash
# 必读文档（按优先级）
1. docs/05-CONTEXT.md  ← ⭐ 最重要：上次会话断点
2. docs/02-TASKS.md    ← 当前任务状态
3. docs/04-CHANGELOG.md ← 最近变更（如果有）
```

- [ ] **读取CONTEXT.md**
  - 找到"最新状态"部分
  - 确认"上次会话完成内容"
  - 确认"当前正在进行"
  - 确认"下次会话待办"

- [ ] **读取TASKS.md**
  - 找到当前Phase
  - 确认哪些任务已完成（checked）
  - 确认哪些任务进行中（⏳标记）
  - 确认下一个待办任务

- [ ] **（可选）读取CHANGELOG.md**
  - 如果最近有重大变更
  - 了解计划调整原因

#### Step 3: 确认环境状态（3分钟）
```bash
# 执行以下命令检查环境
cd /Users/situruimin/project/ai-comparison-system

# 1. 检查Git状态
git status
# 预期：工作区干净（nothing to commit, working tree clean）

# 2. 检查依赖完整性（如果已安装）
[ -d node_modules ] && npm list --depth=0 || echo "Dependencies not installed yet"

# 3. 运行测试（如果有测试）
[ -f package.json ] && npm test || echo "Tests not set up yet"

# 4. 检查最近5次提交
git log --oneline -5
```

- [ ] **检查Git状态**
  - ✅ 工作区干净：可以开始新任务
  - ❌ 有未提交修改：先review后提交

- [ ] **检查依赖**
  - ✅ node_modules存在：正常
  - ❌ node_modules缺失：运行`npm install`

- [ ] **检查测试**
  - ✅ 所有测试通过：正常
  - ❌ 测试失败：先修复后继续

#### Step 4: 向用户确认（1分钟）
```
用户你好！我已完成启动检查：

📋 项目状态：
- 当前Phase: Phase X - XXX
- 进度: XX% (X/XX tasks)

📖 上次进度：
- 已完成：[任务X.X] XXX
- 进行中：[任务X.Y] XXX

✅ 环境检查：
- Git状态：✅ 工作区干净
- 依赖安装：✅ 完整
- 测试状态：✅ 全部通过

📋 下一步任务：
[X.Z] XXX

准备开始了吗？还是需要先做其他事情？
```

- [ ] **等待用户确认**
  - 用户说"开始"/"继续"：执行下一步任务
  - 用户说"先XXX"：按用户要求调整

---

## 🔨 Phase 2: 任务执行规范

> **执行开发任务时必须遵守此流程**

### 任务开始前

#### ✅ Checklist
- [ ] **在TASKS.md中找到对应任务**
  - 确认任务编号（如[1.3]）
  - 确认任务描述
  - 确认任务的子任务列表

- [ ] **创建分支（可选，重大功能时）**
  ```bash
  git checkout -b feature/task-1-3-doubao-provider
  ```

- [ ] **向用户确认任务理解**
  ```
  我将开始 [1.3] 豆包Provider开发

  计划步骤：
  1. 从test1拷贝DoubaoLoginManager.ts
  2. 调整import路径
  3. 实现IAIProvider接口
  4. 编写3个单元测试

  预计耗时：2小时

  是否同意此计划？
  ```

### 任务执行中

#### ✅ Checklist
- [ ] **小步提交**
  - 每完成一个子任务就commit
  - 不要累积多个子任务一次性commit

  ```bash
  # 好的示例
  git commit -m "feat(doubao): 拷贝登录代码并调整import路径"
  git commit -m "feat(doubao): 实现IAIProvider接口"
  git commit -m "test(doubao): 添加登录流程测试"

  # 不好的示例（累积太多）
  git commit -m "feat(doubao): 完成所有开发和测试"
  ```

- [ ] **遵循代码规范**
  - TypeScript严格模式
  - 所有函数有类型注解
  - 禁止使用`any`
  - ESLint无error

- [ ] **遵循测试规范**
  - 关键功能必须有测试
  - 测试必须通过
  - 覆盖率≥70%

- [ ] **遇到问题立即记录**
  - 在CONTEXT.md的"当前正在进行"中记录问题
  - 记录尝试过的解决方案
  - 如果无法解决，向用户说明

### 任务完成后

#### ✅ Checklist（必须全部完成）

#### Step 1: 代码质量检查（5分钟）
```bash
# 1. TypeScript类型检查
npm run typecheck
# 预期：0 errors

# 2. 代码格式化
npm run format
# 自动格式化所有代码

# 3. ESLint检查
npm run lint
# 预期：0 errors, ≤5 warnings

# 4. 运行测试
npm test
# 预期：所有测试通过

# 5. 检查覆盖率（如果配置了）
npm run test:coverage
# 预期：≥70%
```

- [ ] **所有检查必须通过**
  - ❌ 如果有error：立即修复
  - ❌ 如果测试失败：立即修复
  - ✅ 全部通过：继续下一步

#### Step 2: 更新文档（3分钟）
- [ ] **更新TASKS.md**
  ```markdown
  # 将对应的checkbox勾选
  - [x] 拷贝DoubaoLoginManager.ts
  - [x] 调整import路径
  - [x] 实现IAIProvider接口
  ```

- [ ] **更新CONTEXT.md**
  ```markdown
  ### 上次会话完成内容
  - ✅ [1.3] 豆包Provider开发
    - 从test1迁移登录代码
    - 实现IAIProvider接口
    - 添加3个单元测试（全部通过）

  ### 当前正在进行
  - ⏳ [1.4] 会话管理服务迁移

  ### 下次会话待办
  1. 完成SessionService.ts的重命名
  2. 调整所有import路径
  3. 运行集成测试
  ```

- [ ] **（如果有计划变更）更新CHANGELOG.md**
  ```markdown
  ## [变更 #001] 2025-10-13

  **原计划**：豆包Provider一次性完成
  **问题**：LoginDetector逻辑复杂，需要更多时间
  **新方案**：拆分为两个任务，先完成Login再完成Query
  **批准人**：用户（会话2025-10-13-002）
  ```

#### Step 3: Git提交（2分钟）
```bash
# 1. 查看修改
git status
git diff

# 2. 添加文件
git add .

# 3. 提交（使用规范的message）
git commit -m "feat(doubao): 完成豆包Provider基础功能

- 从test1迁移DoubaoLoginManager代码
- 实现IAIProvider接口的login()方法
- 实现query()方法的基础框架
- 添加3个单元测试，全部通过

测试:
- ✅ DoubaoProvider.login()
- ✅ DoubaoProvider.query()
- ✅ DoubaoProvider.restoreSession()

覆盖率: 75%
TASKS: 完成 [1.3] 豆包Provider开发
NEXT: [1.4] 会话管理服务迁移"
```

#### Step 4: 向用户汇报（1分钟）
```
✅ 任务完成汇报：

📋 任务：[1.3] 豆包Provider开发
📝 代码变更：
  - 新增：src/providers/doubao/DoubaoLogin.ts (405行)
  - 新增：src/providers/doubao/DoubaoQuery.ts (150行)
  - 新增：src/providers/doubao/DoubaoProvider.ts (100行)
  - 新增：tests/unit/providers/doubao.test.ts (200行)

✅ 质量检查：
  - TypeScript: ✅ 0 errors
  - ESLint: ✅ 0 errors, 2 warnings
  - Tests: ✅ 8/8 passed
  - Coverage: ✅ 75%

💾 Git提交：
  - Commit: a1b2c3d
  - Message: "feat(doubao): 完成豆包Provider基础功能"

📊 进度更新：
  - Phase 1: 20% → 25% (+5%)
  - 已完成任务：4/20

⏭️ 下个任务：
  [1.4] 会话管理服务迁移（预计1小时）

是否继续，还是先休息？
```

---

## 🚪 Phase 3: 窗口结束规范

> **关闭Claude窗口前必须执行此流程**

### ✅ Checklist（必须全部完成）

#### Step 1: 保存所有进度（2分钟）
- [ ] **检查未保存的文件**
  ```bash
  git status
  ```
  - ❌ 有未保存修改：立即保存
  - ❌ 有未跟踪文件：确认是否需要提交

- [ ] **检查TODO和临时注释**
  ```bash
  grep -r "TODO" src/
  grep -r "FIXME" src/
  grep -r "console.log" src/
  ```
  - ❌ 有TODO：记录到CONTEXT.md
  - ❌ 有console.log：删除或改为logger

- [ ] **提交所有代码**
  ```bash
  git add .
  git commit -m "WIP: 当前进度保存点

  完成：
  - XXX

  未完成：
  - YYY（下次继续）

  问题：
  - ZZZ（需要解决）"
  ```

#### Step 2: 更新CONTEXT.md（3分钟）
```markdown
# docs/05-CONTEXT.md

## 最新状态（2025-10-13 18:00）

### 上次会话完成内容
- ✅ [1.3] 豆包Provider开发
  - 迁移登录代码
  - 实现IAIProvider接口
  - 添加3个单元测试

### 当前正在进行
- ⏳ [1.4] 会话管理服务迁移
  - 📂 文件位置：`src/services/session/SessionService.ts`
  - 🐛 遇到问题：import路径需要调整
  - 💡 解决思路：使用相对路径，避免循环依赖
  - 📊 进度：50%（已重命名，未调整路径）

### 下次会话待办
1. 【优先】完成SessionService的import路径调整
2. 运行SessionService的单元测试
3. 集成到DoubaoProvider中
4. 提交代码并更新TASKS.md

### 重要提醒
- SessionService依赖SessionStorage，需要先确保SessionStorage正常
- 测试文件在tests/unit/services/session.test.ts
- 参考test1的实现：../test1/chrome-devtools-mcp/src/session/

### 本次会话统计
- 工作时长：2小时
- 完成任务：1个 ([1.3])
- 代码行数：+855行
- 测试通过率：100%
- Git提交数：3次
```

- [ ] **确认记录完整**
  - ✅ 完成内容清晰
  - ✅ 当前进度明确
  - ✅ 下次任务具体
  - ✅ 问题和思路都记录

#### Step 3: 运行完整检查（5分钟）
```bash
# 1. 确保能编译
npm run build

# 2. 确保所有测试通过
npm test

# 3. 确认Git工作区干净
git status
# 预期：nothing to commit, working tree clean

# 4. 查看最近提交
git log --oneline -3
```

- [ ] **所有检查必须通过**
  - ❌ 编译失败：立即修复
  - ❌ 测试失败：立即修复或回滚
  - ❌ 工作区不干净：提交或暂存

#### Step 4: 向用户确认（1分钟）
```
🏁 本次会话即将结束

📊 会话总结：
- 工作时长：2小时
- 完成任务：[1.3] 豆包Provider开发
- Git提交：3次
- 代码变更：+855行 / -0行
- 测试通过：8/8

💾 进度保存：
- ✅ 所有代码已提交
- ✅ CONTEXT.md已更新
- ✅ TASKS.md已更新
- ✅ 工作区干净

✅ 环境检查：
- ✅ 编译通过
- ✅ 测试通过（100%）
- ✅ Git状态正常

⏭️ 下次继续：
[1.4] 会话管理服务迁移
（预计从SessionService的import路径调整开始）

可以安全关闭窗口了。
```

---

## 📋 场景化Checklist速查

### 场景1：开始一个新任务
```
☐ 读取CONTEXT.md确认上次进度
☐ 在TASKS.md找到对应任务
☐ 向用户确认任务理解
☐ （可选）创建feature分支
☐ 开始开发...
```

### 场景2：完成一个子任务
```
☐ 运行npm test确保测试通过
☐ 运行npm run typecheck确保无类型错误
☐ Git commit（小步提交）
☐ 更新TASKS.md（勾选checkbox）
```

### 场景3：遇到阻塞问题
```
☐ 在CONTEXT.md记录问题和尝试方案
☐ 向用户说明问题并提供2-3个备选方案
☐ 等待用户决策
☐ 执行用户选择的方案
☐ 在CHANGELOG.md记录变更（如果改变计划）
```

### 场景4：需要删除代码
```
☐ 标记@deprecated（如果是弃用）
☐ 使用Grep工具搜索所有引用
☐ 逐个迁移引用到新代码
☐ 运行所有测试确保无破坏
☐ 删除代码
☐ 在CHANGELOG.md记录删除原因
```

### 场景5：需要重构
```
☐ 向用户说明重构原因和预期收益
☐ 获得用户批准
☐ 创建refactor分支
☐ 小步重构，每步都运行测试
☐ 合并到main分支
☐ 在CHANGELOG.md记录重构
```

---

## 🚫 禁止事项（严格执行）

### 代码相关
- ❌ **绝不**跳过测试直接提交
- ❌ **绝不**使用`any`类型（除非有充分理由并注释说明）
- ❌ **绝不**提交包含`console.log`的代码（改用logger）
- ❌ **绝不**提交包含临时文件的代码（.DS_Store, .vscode等）

### 文档相关
- ❌ **绝不**不更新文档就提交代码
- ❌ **绝不**不记录就删除代码
- ❌ **绝不**不说明就修改核心架构

### Git相关
- ❌ **绝不**累积多个任务一次性commit
- ❌ **绝不**使用无意义的commit message（如"update"）
- ❌ **绝不**在工作区不干净时关闭窗口

### 流程相关
- ❌ **绝不**不读CONTEXT就开始开发
- ❌ **绝不**不检查环境就运行代码
- ❌ **绝不**不向用户确认就做重大变更

---

## ✅ 质量门禁

### 每次提交前必须满足
```
✅ TypeScript编译: 0 errors
✅ ESLint检查: 0 errors, ≤5 warnings
✅ 单元测试: 100% passed
✅ 代码覆盖率: ≥70%
✅ 无遗留console.log
✅ 文档已同步更新
```

### 每完成一个Phase必须满足
```
✅ 所有任务checkbox已勾选
✅ 代码覆盖率≥70%
✅ 集成测试全部通过
✅ 文档完整且最新
✅ 无技术债务遗留
✅ 用户验收通过
```

---

## 📊 执行效果评估

### 每周review
```
✅ Git提交是否规范？
✅ TASKS.md是否实时更新？
✅ CONTEXT.md是否准确反映进度？
✅ 代码质量是否达标？
✅ 测试覆盖率是否≥70%？
```

### 发现问题如何处理
```
1. 在CHANGELOG.md记录问题
2. 制定改进计划
3. 更新WORKFLOW.md（本文档）
4. 通知所有后续Claude窗口
```

---

## 💡 最佳实践

### 1. 频繁小提交 > 大批量提交
```bash
# 好
git commit -m "feat(doubao): 添加登录方法"
git commit -m "feat(doubao): 添加查询方法"
git commit -m "test(doubao): 添加登录测试"

# 不好
git commit -m "feat(doubao): 完成所有功能"
```

### 2. 详细的commit message > 简短的message
```bash
# 好
git commit -m "feat(doubao): 完成登录功能

- 实现手机号验证码登录
- 添加会话状态检测
- 集成到Provider接口

测试: 3个单元测试通过
TASKS: 完成 [1.3.1]
NEXT: 开始query()实现"

# 不好
git commit -m "update doubao"
```

### 3. 实时更新文档 > 批量更新
```
# 好：每完成一个子任务就更新TASKS.md
- [x] 子任务1
- [x] 子任务2
- [ ] 子任务3

# 不好：等所有任务完成再一次性更新
```

### 4. 主动记录问题 > 事后回忆
```
# 在CONTEXT.md立即记录：
🐛 遇到问题：Kimi的验证码选择器不稳定
💡 尝试方案1：使用固定选择器（失败）
💡 尝试方案2：使用wait_for工具（成功）
```

---

## 🎯 检查清单总结

### 新窗口启动（必做）
```
☑ 读CONTEXT.md（断点）
☑ 读TASKS.md（任务）
☑ 检查环境（git/npm/test）
☑ 向用户确认
```

### 任务完成（必做）
```
☑ 跑测试
☑ 更新TASKS.md
☑ 更新CONTEXT.md
☑ Git commit
☑ 向用户汇报
```

### 窗口结束（必做）
```
☑ 保存进度
☑ 更新CONTEXT.md
☑ 完整检查（build/test/git）
☑ 向用户确认
```

---

## 📞 遇到问题？

### 常见问题快速查询

**Q1: 忘记上次做到哪里了？**
A: 读`docs/05-CONTEXT.md`的"最新状态"部分

**Q2: 不知道下一步做什么？**
A: 读`docs/02-TASKS.md`找到第一个未勾选的checkbox

**Q3: Git工作区不干净怎么办？**
A: 先review修改（`git diff`），决定提交还是回滚

**Q4: 测试失败怎么办？**
A: 立即修复，不允许带着失败测试继续开发

**Q5: 发现之前的代码有问题？**
A: 在CONTEXT.md记录，向用户说明，获批准后修复

---

**Last Updated**: 2025-10-13 16:50
**Version**: v1.0
**Status**: 正式生效

**重要提醒**: 本文档是项目质量的保障，必须严格遵守！
