# Service层开发规范

> **本文件在Claude读取src/services/目录文件时自动加载**

---

## Service层职责

- 业务逻辑编排
- Provider调用
- 数据库操作
- 资源评估

---

## 目录结构

```
src/services/
├── QueryService.ts           # 查询编排
├── SessionService.ts         # 会话管理
├── DatabaseService.ts        # 数据持久化
├── ResourceEvaluator.ts      # 系统资源评估
└── executors/                # 执行器
    ├── SerialQueryExecutor.ts          # 串行模式
    ├── MultiAIParallelExecutor.ts      # 方案A
    └── SingleAIMultiQuestionExecutor.ts # 方案B
```

---

## 开发规范

### 1. 禁止直接操作UI
- Service层不依赖blessed
- 使用回调返回进度信息

### 2. 异步处理
- 所有方法返回`Promise`
- 使用`async/await`语法

### 3. 错误处理
- 统一错误类型：`ServiceError`
- 使用try-catch包裹所有Provider调用

### 4. 测试要求
- 每个Service至少5个单元测试
- 模拟Provider和Database

---

**Last Updated**: 2025-10-18 21:40
