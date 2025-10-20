# TUI开发规范

> **本文件在Claude读取src/tui/目录文件时自动加载**

---

## TUI框架

使用blessed创建终端界面，支持：
- 窗口布局（box、list、table）
- 键盘导航（↑↓←→、Enter、Space、Tab、Q）
- 实时刷新（进度条、日志滚动）

---

## 目录结构

```
src/tui/
├── index.ts               # TUI入口
├── TUIManager.ts          # 核心管理器
├── screens/               # 6个界面
│   ├── MainScreen.ts
│   ├── AIManagementScreen.ts
│   ├── TaskManagementScreen.ts
│   ├── ExecutionDemoScreen.ts
│   ├── ExecutionDBScreen.ts
│   ├── ResultDemoScreen.ts
│   └── ResultDBScreen.ts
└── components/            # 公共组件
    ├── Header.ts
    ├── Footer.ts
    └── Table.ts
```

---

## 开发规范

### 1. 界面布局
- 统一使用三段式：Header + Content + Footer
- Header显示标题和状态
- Footer显示快捷键提示

### 2. 键盘处理
- 全局快捷键：Q退出、Esc返回、↑↓选择
- 上下文快捷键：在Footer中显示

### 3. 错误处理
- 使用boxen显示错误弹窗
- 3秒后自动关闭或按任意键关闭

### 4. 参考设计
- 查看产品设计：`@docs/06-TUI-DESIGN/`

---

**Last Updated**: 2025-10-18 21:40
