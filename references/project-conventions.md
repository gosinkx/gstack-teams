# gstack Agent Teams — 项目公共规范模板

> 使用方法：将本文件内容复制到项目根目录的 `CODEBUDDY.md`，
> 按项目实际情况填写 [占位符] 部分。
> 所有 Agent Teams 成员在启动时会自动读取此文件。

---

# [项目名称] — Agent Teams 项目规范

本文件为所有 Agent Teams 成员提供统一的项目上下文和行为规范。
基于 [gstack](https://github.com/garrytan/gstack) 角色体系，适配 WorkBuddy Agent Teams。

---

## 核心原则（所有成员必须遵守）

### 完整性原则（Boil the Lake）
> AI 辅助编码使得完整实现的边际成本接近于零。
> **倾向于选择完整实现，而非走捷径。**

- 所有边缘情况都应处理
- 测试覆盖应全面
- 错误处理应完整
- 不接受"差不多能用"的方案

### 零静默失败原则
每个失败模式都必须对系统、团队或用户可见。禁止静默吞掉异常。

### 证据优先原则
所有结论必须基于实际代码、日志或截图，禁止凭感觉断言。

### 最小 diff 原则
只修改与任务直接相关的代码，避免范围蔓延。

---

## 成员间协作规范

1. **任务交接**：完成阶段性任务后，将产物写入约定文件（见下方路径约定），并通过 `@Lead` 通知 Leader
2. **问题上报**：遇到阻塞或超出职责范围的问题，立即 `@Lead` 而不是自行决策
3. **成果文件路径约定**：
   - 设计文档 → `docs/designs/`
   - 测试报告 → `docs/qa-reports/`
   - 架构图 → `docs/architecture/`
   - TODO 跟踪 → `TODOS.md`
   - 回顾报告 → `.context/retros/`

---

## 项目约定（按实际项目填写）

- **项目语言**: [例：TypeScript / Python / Dart]
- **框架**: [例：React / FastAPI / Flutter]
- **测试框架**: [例：Jest / pytest / flutter test]
- **基础分支**: `main`
- **版本规范**: [例：Semantic Versioning / 日期版本]
- **代码风格**: [例：ESLint + Prettier / Black / dart format]
- **提交规范**: [例：Conventional Commits / 自由格式]

---

## 文件目录规范

```
项目根目录/
├── CODEBUDDY.md          ← 本文件（所有成员读取）
├── TODOS.md              ← 各角色维护的待办清单
├── CHANGELOG.md          ← 由 release-engineer 维护
├── docs/
│   ├── designs/          ← product-partner、ceo-reviewer 输出的设计文档
│   ├── qa-reports/       ← qa-lead 输出的测试报告
│   └── architecture/     ← eng-reviewer 输出的架构图
└── .context/
    └── retros/           ← eng-manager 输出的周回顾报告
```
