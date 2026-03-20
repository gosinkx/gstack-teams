---
name: gstack-teams
description: >
  基于 gstack（github.com/garrytan/gstack）的 WorkBuddy Agent Teams 多角色调度 skill。
  当用户需要以下操作时触发：创建多角色开发团队、按阶段调度 agent、规划新功能（产品审查/架构审查）、
  代码审查、调试 bug、设计审查、QA 测试、发布流程、文档同步、工程回顾分析。
  触发词示例：创建开发团队、调用 gstack 角色、规划期团队、发布期团队、多角色协作、
  帮我做 code review（多角色）、帮我跑完整发布流程、启动 agent teams。
---

# gstack-teams Skill

## 概述

本 skill 将 [gstack](https://github.com/garrytan/gstack) 的 9 个专业角色适配为 WorkBuddy Agent Teams，
让 Leader 能够按开发阶段（规划 → 开发 → 发布）按需召唤对应角色，形成完整的虚拟工程团队。

**9 个角色一览：**

| 角色名 | 职责 | 阶段 |
|--------|------|------|
| Product Partner | 产品方向验证，YC Office Hours 风格 | 规划期 |
| CEO Reviewer | 战略审查，11 维度深度评审 | 规划期 |
| Eng Reviewer | 架构规划，锁定技术方案 | 规划期 |
| Staff Engineer | 代码审查，双阶段关键/信息性 | 开发期 |
| Debugger | 根因调试，铁律不找根因不修复 | 开发期 |
| Design Reviewer | 设计质量审查，AI 痕迹检测 | 开发期 |
| QA Lead | 系统化测试，健康评分 | 发布期 |
| Release Engineer | 全自动发布流程 | 发布期 |
| Doc Engineer | 发布后文档同步 | 发布期 |
| Eng Manager | 工程效能回顾分析 | 任意时间 |

---

## 核心工作流程

### 一、识别用户意图 → 选择对应阶段

收到用户请求后，判断属于哪个开发阶段：

- **规划期**：用户有新功能想法、需要产品验证或架构设计 → 启动规划团队
- **开发期**：用户写完代码需要 review、有 bug 需要调查、有 UI 问题 → 启动开发评审团队
- **发布期**：用户准备发布、需要测试 + 推送 + 文档 → 启动发布团队
- **回顾**：用户想看工程效能数据 → 启动 eng-manager

### 二、初始化 Agent Teams

**参考文件：** `references/roles.md` — 包含所有角色的完整 Prompt

按照以下步骤初始化团队：

1. 在 WorkBuddy 主会话中，用自然语言请求创建对应角色的 Agent Teams 成员
2. 为每个成员粘贴 `references/roles.md` 中对应的角色 Prompt
3. 按阶段设定执行顺序（见下方各阶段调度指令）

### 三、各阶段调度指令

#### 📋 阶段一：规划期（新功能开始前）

```
帮我创建规划团队，包含：
- product-partner：[粘贴 roles.md 中 Product Partner 的 Prompt]
- ceo-reviewer：[粘贴 CEO Reviewer 的 Prompt，模式默认 SELECTIVE EXPANSION]
- eng-reviewer：[粘贴 Eng Reviewer 的 Prompt]

任务：[描述新功能需求]

执行顺序：
1. product-partner 先运行，输出设计文档到 docs/designs/
2. 完成后 @Lead，再启动 ceo-reviewer
3. 完成后 @Lead，再启动 eng-reviewer
4. 汇总三方意见，给出"是否开始编码"的决策
```

#### 💻 阶段二：开发期（代码开发中）

```
帮我创建开发评审团队：
- staff-engineer：[粘贴 Staff Engineer 的 Prompt]
- debugger（待命）：[粘贴 Debugger 的 Prompt]
（如有前端变更，追加）
- design-reviewer：[粘贴 Design Reviewer 的 Prompt]

任务：审查当前分支变更

执行：staff-engineer 与 design-reviewer 并行；发现 bug 时召唤 debugger
```

#### 🚀 阶段三：发布期（准备上线）

```
帮我创建发布团队：
- qa-lead：[粘贴 QA Lead 的 Prompt，模式：Standard]
- release-engineer：[粘贴 Release Engineer 的 Prompt]
- doc-engineer：[粘贴 Doc Engineer 的 Prompt]

执行顺序：
1. qa-lead 完整测试 → 健康评分达标
2. release-engineer 执行发布 → 推送 PR
3. doc-engineer 同步文档
```

#### 📊 额外：工程回顾（任意时间）

```
启动 eng-manager，时间范围：[7d/14d/30d]，执行本周工程效能分析
[粘贴 Eng Manager 的 Prompt]
```

### 四、项目公共规范

**参考文件：** `references/project-conventions.md` — 复制到项目根目录的 CODEBUDDY.md，
供所有成员自动读取，统一行为规范（完整性原则、零静默失败、证据优先、最小 diff）。

### 五、角色依赖关系

```
新功能需求
    │
[product-partner] → 设计文档
    │
[ceo-reviewer] ──→ 战略审查
    │
[eng-reviewer] ──→ 架构规划
    │（开始编码）
[staff-engineer] ─┐
[design-reviewer] ─┤ 并行审查
[debugger] (待命) ─┘
    │（代码就绪）
[qa-lead] ───────→ 质量测试（必须通过）
    │
[release-engineer] → 自动发布
    │
[doc-engineer] ──→ 文档同步
    │
[eng-manager] ───→ 每周回顾（可选）
```

---

## 重要注意事项

- **不要一次启动所有角色**，按阶段分批，避免上下文混乱
- **每次只有一个角色主动执行**（明确并行的场景除外）
- **成员间通过文件传递产物**（设计文档、测试报告等），不依赖实时内存共享
- **发布流程不可跳过 qa-lead**，健康分未达标禁止触发 release-engineer
- **关键决策必须回到 Leader（主会话）**，人来做最终判断

---

## 参考文件

- `references/roles.md` — 所有角色的完整 Prompt（初始化成员时直接复制粘贴）
- `references/project-conventions.md` — 复制到项目根目录的公共规范模板
