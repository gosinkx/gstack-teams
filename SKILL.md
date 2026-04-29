---
name: gstack-teams
description: >
  基于 gstack（github.com/garrytan/gstack）的 WorkBuddy Agent Teams 多角色调度 skill。
  当用户需要以下操作时触发：创建多角色开发团队、按阶段调度 agent、规划新功能（产品审查/架构审查）、
  代码审查、调试 bug、设计审查、QA 测试、发布流程、文档同步、工程回顾分析。
  触发词示例：创建开发团队、调用 gstack 角色、规划期团队、发布期团队、多角色协作、
  帮我做 code review（多角色）、帮我跑完整发布流程、启动 agent teams。
---

# gstack_teams Skill v2.0

## 概述

本 skill 将 [gstack](https://github.com/garrytan/gstack) 的 14 个专业角色适配为 WorkBuddy Agent Teams，
让 Leader 能够按开发阶段（规划 → 开发 → 发布 → 部署后）按需召唤对应角色，形成完整的虚拟工程团队。

**14 个角色一览：**

| 角色名 | 职责 | 阶段 | 建议 max_turns |
|--------|------|------|----------------|
| Product Partner | 产品方向验证，YC Office Hours 风格 | 规划期 | 15 |
| CEO Reviewer | 战略审查，11 维度深度评审 | 规划期 | 20 |
| Eng Reviewer | 架构规划，锁定技术方案 | 规划期 | 15 |
| Autoplan | 自动化审查流水线，6条决策原则 | 规划期 | 25 |
| Staff Engineer | 代码审查，双阶段关键/信息性 | 开发期 | 15 |
| Debugger | 根因调试，铁律不找根因不修复 | 开发期 | 25 |
| Design Reviewer | 设计质量审查，AI 痕迹检测 | 开发期 | 10 |
| CSO | 首席安全官，OWASP + STRIDE 审计 | 发布期前 | 20 |
| QA Lead | 系统化测试，健康评分 | 发布期 | 30 |
| Ship (/ship) | 创建 PR，测试，推送 | 发布期 | 20 |
| Land-and-Deploy | 合并 PR，部署，上线验证 | 发布期 | 25 |
| Canary | 部署后可视监控，基线对比+控制台错误 | 发布期后 | 10 |
| Doc Engineer | 发布后文档同步 | 发布期 | 15 |
| Eng Manager | 工程效能回顾分析 | 任意时间 | 15 |

---

## ⚠️ 安全机制（防止假死，必须严格遵守）

> **每次使用本 skill 创建子 agent 时，必须遵守以下安全规则。违反这些规则是导致假死的直接原因。**

### 规则 1：强制设置 `max_turns`

每次调用 Agent 工具创建子 agent 时，**必须**设置 `max_turns` 参数（参考上表建议值）。
不设置 `max_turns` 的子 agent 可能无限循环直至假死。

```python
# ✅ 正确示例：
Agent(
    description="启动 QA 测试",
    prompt="...",
    max_turns=30   # ← 必须设置
)

# ❌ 错误示例（禁止）：
Agent(
    description="启动 QA 测试",
    prompt="..."
    # 没有 max_turns —— 子 agent 可能无限循环！
)
```

### 规则 2：终端命令强制设置 timeout

子 agent 执行 **所有** Bash / PowerShell 命令时，必须设置合理的 `timeout`（单位：毫秒）：

| 命令类型 | 建议 timeout | 说明 |
|----------|--------------|------|
| 文件操作（ls/cat/cp 等） | 30000 | 30秒，快速操作 |
| 包安装（npm/pip 等） | 300000 | 5分钟，有网络依赖 |
| 构建/编译 | 600000 | 10分钟，大型项目 |
| 测试套件 | 300000 | 5分钟，视测试量而定 |
| git 操作（本地） | 60000 | 1分钟 |
| git push/fetch（远程） | 120000 | 2分钟 |

```python
# ✅ 正确：
Bash(command="npm install", timeout=300000, description="安装依赖")

# ❌ 错误（禁止）：
Bash(command="npm install", description="安装依赖")
# 没有 timeout —— 网络慢时会无限等待！
```

### 规则 3：禁止交互式命令

子 agent **不得**执行任何可能进入交互模式的命令。必须预先加上非交互参数：

| 命令 | 安全写法 |
|------|----------|
| `npm install` | `npm install --yes` 或先设置 `CI=true` 环境变量 |
| `git push` | 确保已配置 credential helper，或使用 `git push --quiet` |
| `ssh` | `ssh -o BatchMode=yes` |
| Python 交互模式 | 使用 `python -c "..."` 或脚本文件 |
| 任何可能 prompt 的命令 | 预先加 `--yes` / `-y` / `--no-input` 等参数 |

### 规则 4：子 agent 必须定期上报状态

每个子 agent 在 prompt 中**必须**包含状态上报指令（见 `references/roles.md` 中每个角色的 `STATUS_REPORTING` 部分）。
上报频率：每完成 2-3 个工具调用，通过 `SendMessage(type="message")` 向 Lead 汇报一次进度。

### 规则 5：长时间任务使用 `run_in_background`

预计运行时间超过 5 分钟的任务（如完整测试套件、大型构建），**必须**使用 `run_in_background=true`，
主 agent 不阻塞等待，而是通过 `TaskOutput` 或轮询方式获取结果。

---

## 看门狗模式（Watchdog）

主 agent（Leader）在启动子 agent 后，应启动看门狗监控，**防止子 agent 假死后无限等待**。

### 看门狗流程

```
1. 启动子 agent（设置 max_turns）
   │
2. 主 agent 进入等待状态
   │
3. 【看门狗】每 60 秒检查一次子 agent 状态：
   │   - 如果收到子 agent 的状态上报 → 正常，继续等待
   │   - 如果超过 180 秒未收到任何消息 → 发送健康检查
   │   - 如果发送 2 次健康检查均无响应 → 判定为假死
   │
4. 【恢复】判定假死后：
   ├── 通过 SendMessage 发送 shutdown_request
   ├── 等待 30 秒确认关闭
   ├── 如果未关闭，记录失败原因
   └── 向用户报告：哪个 agent 假死、可能原因、建议操作
```

### 健康检查实现

主 agent 向子 agent 发送消息：
```
SendMessage(
    type="message",
    recipient="子agent名称",
    content="HEALTH_CHECK：请回复当前状态（正在执行第X步，最后操作为...）",
    summary="Health check"
)
```

子 agent 的 prompt 中已包含响应健康检查的指令（见 `references/roles.md`）。

---

## 核心工作流程

### 一、识别用户意图 → 选择对应阶段

收到用户请求后，判断属于哪个开发阶段：

- **规划期**：用户有新功能想法、需要产品验证或架构设计 → 启动规划团队（含可选 autoplan 自动审查流水线）
- **开发期**：用户写完代码需要 review、有 bug 需要调查、有 UI 问题 → 启动开发评审团队
- **发布期前**：用户需要安全审计 → 启动 CSO（首席安全官，OWASP + STRIDE）
- **发布期**：用户准备发布、需要测试 + 创建 PR → 启动发布团队（Ship）
- **部署期**：PR 已合并、需要部署上线 → 启动 Land-and-Deploy
- **发布期后**：部署完成、需要上线验证 → 启动 Canary（部署后可视监控）
- **回顾**：用户想看工程效能数据 → 启动 eng-manager

### 二、初始化 Agent Teams（安全模式）

**参考文件：** `references/roles.md` — 包含所有角色的完整 Prompt

按照以下步骤初始化团队（**必须**遵守安全机制）：

1. 在 WorkBuddy 主会话中，用 `Agent` 工具创建对应角色的 Agent
2. **必须设置 `max_turns`**（见上方角色表建议值）
3. 为每个 Agent 的 prompt 追加 `references/safety-rules.md` 中的安全规范
4. 按阶段设定执行顺序（见下方各阶段调度指令）

### 三、各阶段调度指令（含安全参数）

#### 📋 阶段一：规划期（新功能开始前）

**选项 A：手动审查（逐项确认）**
```
帮我创建规划团队，包含（每个都设置 max_turns）：
- product-partner：max_turns=15，prompt 见 roles.md
- ceo-reviewer：max_turns=20，模式默认 SELECTIVE EXPANSION
- eng-reviewer：max_turns=15

任务：[描述新功能需求]

执行顺序：
1. product-partner 先运行（max_turns=15）
2. 完成后 @Lead，再启动 ceo-reviewer（max_turns=20）
3. 完成后 @Lead，再启动 eng-reviewer（max_turns=15）
4. 汇总三方意见，给出"是否开始编码"的决策
```

**选项 B：自动审查流水线（autoplan，无中间确认）**
```
帮我创建自动审查流水线（设置 max_turns=25）：
- autoplan：按顺序自动执行 CEO Review → Design Review → Eng Review
- 使用 6 条决策原则自动决策中间问题
- 最终门控呈现推荐供用户确认

6 条决策原则：
1. 选择完整性 — 覆盖更多边界情况
2. 完整实现 — AI 辅助使完整实现的边际成本近零
3. 务实 — 选更简洁的修复
4. DRY 原则 — 拒绝重复
5. 显式优于聪明 — 10 行明显修复 > 200 行抽象
6. 行动偏向 — 合并 > 审查循环 > 陈旧推敲
```

#### 💻 阶段二：开发期（代码开发中）

```
帮我创建开发评审团队（每个都设置 max_turns）：
- staff-engineer：max_turns=15
- debugger（待命）：max_turns=25
- design-reviewer（如有前端变更）：max_turns=10

任务：审查当前分支变更

执行：staff-engineer 与 design-reviewer 并行（如有）；
     发现 bug 时召唤 debugger（max_turns=25）
```

#### 🛡️ 阶段三：发布期前（安全审计）

**必须在 QA 之前完成，高风险问题未修复禁止进入发布期**

```
帮我启动 CSO（首席安全官，max_turns=20）：
- 执行 15 个阶段 OWASP + STRIDE 安全审计
- 审计完成后给出门控结论：APPROVED / APPROVED_WITH_RISKS / BLOCKED

阻断条件（以下情况停止并 @Lead）：
- 发现高风险问题（OWASP Top 10 相关）→ 必须修复后才能发布
- 依赖项有已知 CVE 且严重等级为 High/Critical → 必须升级依赖
- 代码中检测到硬编码密钥 → 必须轮换密钥并清理提交历史

如为 APPROVED_WITH_RISKS，需在发布后跟进修复。
```

---

#### 🚀 阶段四：发布期（准备上线 + 合并部署）

**选项 A：分步执行（推荐，便于控制）**
```
帮我创建发布团队（每个都设置 max_turns）：
- qa-lead：max_turns=30，模式：Standard
- ship：max_turns=20
- land-and-deploy：max_turns=25
- doc-engineer：max_turns=15

执行顺序：
1. qa-lead 完整测试（max_turns=30）→ 健康评分达标（关键问题清零，高优 ≤ 2）
2. ship 执行发布流程（max_turns=20）→ 创建 PR，推送远程
3. land-and-deploy 合并+部署+验证（max_turns=25）→ 合并 PR → 部署 → 上线验证
4. doc-engineer 同步文档（max_turns=15）
5. 触发 Canary 监控（见阶段五）
```

**选项 B：Ship 与 Land-and-Deploy 合并调用（全自动）**
```
帮我执行完整发布流程（全自动，无中间确认）：
1. qa-lead（max_turns=30）→ 必须通过
2. ship（max_turns=20）→ 创建并推送 PR
3. land-and-deploy（max_turns=25）→ 合并 → 部署 → 验证
4. doc-engineer（max_turns=15）→ 同步文档
5. 自动触发 Canary 监控

阻塞条件：任一步骤失败 → 中止并 @Lead 报告
```

---

#### 📡 阶段五：发布期后（部署后监控）

```
部署完成且 Land-and-Deploy 验证通过后，启动 Canary（max_turns=10）：
- 建立/更新性能基线（如需要）
- 部署后实时对比（与基线对比）
- 控制台错误检测（首页 + 核心功能页）
- 用户行为异常检测（可选）

监控时长：
- 小型修复（Hotfix）：15 分钟
- 常规发布：30 分钟
- 大型功能发布：60 分钟

判定结论：
- 🟢 通过：无需操作
- 🟡 警告：建议调查，暂不回滚
- 🔴 异常：建议立即回滚

如为 🔴 异常，Canary 会自动通知 Lead 并建议回滚。
```

---

#### 📊 额外：工程回顾（任意时间）

```
启动 eng-manager（max_turns=15），时间范围：[7d/14d/30d]，执行本周工程效能分析
```

---

## 子 agent 假死恢复流程

当检测到子 agent 假死（超过 180 秒无响应 + 2 次健康检查失败）：

### 步骤 1：尝试优雅关闭

```
SendMessage(
    type="shutdown_request",
    recipient="假死的 agent 名称",
    content="检测到无响应，请确认当前状态。如正常运行请回复，否则将强制关闭。"
)
```

等待 30 秒。如果子 agent 响应并批准关闭 → 关闭成功，进入步骤 3。

### 步骤 2：强制关闭（如优雅关闭失败）

WorkBuddy 目前不支持强制终止子 agent。此时应：
1. 向用户报告：哪个 agent 假死、最后已知状态、可能原因
2. 建议用户手动重启会话
3. 记录到 `references/failure-log.md`（新建此文件记录所有假死事件）

### 步骤 3：任务恢复

根据子 agent 的最后已知状态（从之前的 `SendMessage` 记录中获取）：
1. 读取子 agent 已完成的输出文件（如 `docs/designs/` 下的设计文档）
2. 判断任务完成度
3. 如任务未完成，重新创建子 agent（设置相同的 `max_turns`），从断点继续
4. 在 prompt 中说明"请先读取已有文件，从中断处继续"

---

## 项目公共规范

**参考文件：** `references/project-conventions.md` — 复制到项目根目录的 CODEBUDDY.md，
供所有成员自动读取，统一行为规范（完整性原则、零静默失败、证据优先、最小 diff）。

**额外参考文件：** `references/safety-rules.md` — 所有角色的安全规范，初始化时追加到每个角色的 prompt 末尾。

---

## 角色依赖关系

```
新功能需求
    │
[product-partner] → 设计文档
    │
[ceo-reviewer] ──→ 战略审查
    │
[eng-reviewer] ──→ 架构规划
    │
[autoplan] ──────→ 自动审查流水线（可选，自动串联以上三项）
    │（开始编码）
[staff-engineer] ─┐
[design-reviewer] ─┤ 并行审查
[debugger] (待命) ─┘
    │（代码就绪）
[qa-lead] ───────→ 质量测试（必须通过，max_turns=30）
    │
[cso] ──────────→ 安全审计（必须在 Ship 之前完成）
    │
[ship] ─────────→ 创建 PR，推送远程（max_turns=20）
    │
[land-and-deploy] ─→ 合并 + 部署 + 验证（max_turns=25）
    │
[doc-engineer] ──→ 文档同步（max_turns=15）
    │
[canary] ────────→ 部署后监控（max_turns=10）
    │
[eng-manager] ───→ 每周回顾（可选，max_turns=15）
```

---

## 重要注意事项

- **不要一次启动所有角色**，按阶段分批，避免上下文混乱
- **每次只有一个角色主动执行**（明确并行的场景除外）
- **成员间通过文件传递产物**（设计文档、测试报告等），不依赖实时内存共享
- **发布流程不可跳过 qa-lead**，健康分未达标禁止触发 release-engineer
- **关键决策必须回到 Leader（主会话）**，人来做最终判断
- **所有 Agent 调用必须设置 max_turns**（见上方角色表）
- **所有终端命令必须设置 timeout**（见规则 2）
- **子 agent 假死后必须记录到 failure-log.md**，用于改进 skill

---

## 参考文件

- `references/roles.md` — 所有角色的完整 Prompt（初始化成员时直接复制粘贴）
  - **v2.0 新增**：每个角色包含 `STATUS_REPORTING` 和 `SAFETY RULES` 章节
  - **v2.1 新增角色**：Autoplan、CSO、Land-and-Deploy、Canary
- `references/project-conventions.md` — 复制到项目根目录的公共规范模板
- `references/safety-rules.md` — 所有角色共用的安全规范
- `references/failure-log.md` — 记录所有子 agent 假死事件
- `references/cso.md` — CSO（首席安全官）完整审计流程（15 个阶段）
- `references/autoplan.md` — Autoplan 自动审查流水线（6 条决策原则）
- `references/land-and-deploy.md` — Land-and-Deploy 合并+部署+验证流程
- `references/canary.md` — Canary 部署后监控流程（基线对比+控制台错误）

---

*v2.1 — 2026-04-29 — 同步 gstack 原版新特性，增加 CSO/Canary/Autoplan/Land-and-Deploy 四个角色，新增阶段三（发布期前）、阶段五（发布期后）*
*v2.0 — 2026-04-29 — 新增安全机制（7 条规则）、看门狗模式、假死恢复流程*

